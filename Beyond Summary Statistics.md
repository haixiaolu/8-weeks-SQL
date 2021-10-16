# SQL Implementation

## Order and Assign

1. Order all of the weight measurement values from smallest to largest
2. Split them into 100 equal buckets - and assign a number from 1 through to 100 for each bucket

We can actually achieve both of these algorithmic steps in a single bit of SQL functionality with **analytical function**

- Firstly the `OVER` and `ORDER BY` clauses in the following query help us re-order the dataset by the `measure_value` column

- Then the `NTILE` window function is used to perform the assignment of numbers 1 through 100 for each row in the records for each `measure_vaule`

  ```sql
  SELECT
  	measure_value,
  	NTILE(100) OVER(
      ORDER BY
      	measure_value
      ) AS percentile
   FROM health.user_logs
   WHERE measure = 'weight'
    	
  ```

## Bucket Calculations

3. For each bucket

   1. calculate the minimum value and the maximum value for the ceiling and floor values
   2. count how many records there are 

   We already split our data into 100 buckets - we can simply use `GROUP BY` on the `percentile` column from the previous table to calculate our `MIN` and `MAX` `measure_value` ranges for each bucket and also the `COUNT` of records for the `percentile_counts` field

   ```sql
   WITH percentile_value AS(
   	SELECT
   		measure_value,
   		NTILE(100) OVER(
       	ORDER BY 
       		measure_value
       )AS percentile
   	FROM health.user_logs
   	WHERE measure='weight'
   )
   SELECT
   	percentile,
   	MIN(measure_value) AS floor_value,
   	MAX(measure_value) AS ceiling_value,
   	COUNT(*) AS percentile_counts
   FROM percentile_values
   GROUP BY percentile
   ORDER BY percentile;
   
   ```

## Deep Dive Into 100th Percentil

### Window Funtions for Sorting Values

```sql
WITH percentile_value AS(
	SELECT
		measure_value,
		NTILE(100) OVER(
    	ORDER BY
    		measure_value
    ) AS percentile
	FROM health.user_logs
	WHERE measure='weight'
)
SELECT 
	measure_value,
	-- there are examples of window functions below
	ROW_NUMBER() OVER (ORDER BY measure_value DESC) AS row_number_order,
	RANK() OVER(ORDER BY measure_value DESC) AS rank_order,
	DENSE_RANK() OVER(ORDER BY measure_value DESC) AS dense_rank_order

FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;
```

### Large Outliers

