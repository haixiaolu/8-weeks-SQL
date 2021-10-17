# 1. Central Location Statistics

- Location statistics are something like **mean, median, mode**

## 1.1 Arithmetic Mean or Average

$$\mu = \frac{\sum_{i=1}^N X_i}{N}$$

```sql
SELECT
	AVG(measure_value)
FROM health.user_logs;

```

## 1.2. The Median Concept

- In probability theory - the median is also known as the 50th percentile value. 

## 1.3. Ordered Set Aggregate Funtions

- Median Algorithm

  - Spot all N values from smallest to largest
  - Inspect the central values of the sorted set:
    - if N is odd:
      - the median is the value in the $\frac{N+1}{2}$th position
    - elis if N is even
      - the median is the average of values in the (N/2)th and 1 + (N/2)th positions

- Mode Algorithm

  - calculate the tally of values similar to a `GROUP BY` and `COUNT
    - The mode is the values the highest number of occurences

- To implement these 'algorithms' we need to use what we called Ordered Set Aggregate Functions in PostgreSQL

  ```sql
  WITH sample_data (example_values) AS(
  	VALUES
  	(82), (51, (144), (84), (120), (148), (148), (108), (160), (86))
  SELECT
  	AVG(example_values) AS mean_value,
  	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY example_values) AS median_value,
    MODE() WITHIN GROUP(ORDER BY example_values) AS mode_value
  FROM sample_data; 
  ```

# Spread of the Data

## 2.1. Min, Max & Range

- The minimum and maximum values of observations in a dataset help us identify the boundaries of where our data exists

- Range calculation to show the total possible limits of our data by simply calculating the difference between the max and min 

- `MIN` and `MAX` in SQL

  ```sql
  SELECT 
  	MIN(measure_value) AS minimum_value,
  	MAX(measure_value) AS maximum_value,
  	MAX(measure_value) - MIN(measure_value) AS range_value
  FROM health.user_logs
  WHERE measure = 'weight';
  ```

# Variance & Standard Deviation

- the variance is the sum of the (difference between each X value and the mean) squared, divided by N, the total number of values

  - $\sigma^2 = \frac{\sum_{i=1}^n (X_i - \mu)^2}{n}$

- Variance & Std in SQL

  ```sql
  WITH sample_data(example_values) AS
  	VALUES
  		(82), (51), (144), (84), (120), (148), (148), (108), (160), (86))
  SELECT 
  	ROUND(VARIANCE(example_values), 2) AS variance_value,
  	ROUND(STDDEV(example_values), 2) AS standard_dev_values,
  	ROUND(AVG(example_values), 2) AS mean_value,
  	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY example_values) AS median_value,
  	MODE() WITHIN GROUP(ORDER BY example_values) AS mode_value
  	
  FROM sample_data; 
  	
  ```

- Interpreting Spread

  - Spread for Normal Distribution
    - Empirical Rule
      - ![Screen Shot 2021-10-12 at 11.31.11 PM](/Users/haixiaolu/Library/Application Support/typora-user-images/Screen Shot 2021-10-12 at 11.31.11 PM.png)
      - <img src="/Users/haixiaolu/Library/Application Support/typora-user-images/Screen Shot 2021-10-12 at 11.30.42 PM.png" alt="Screen Shot 2021-10-12 at 11.30.42 PM" style="zoom:50%;" />

# Calculating All the Summary Statistics

```sql
SELECT 
	'weight' AS measure,
	ROUND(MIN(measure_value), 2) AS minimum_value,
	ROUND(MAX(measure_value), 2) AS Maximum_value,
	ROUND(AVG(measure_value), 2) AS mean_value,
	ROUND(
  	-- this function actually returns a float which is incompatible with ROUND
  	-- we use this cast function to convert the output type to Numeric)
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value) AS NUMERIC,2) AS median_value
   ROUND(
   	MODE() WITHIN GROUP(ORDER BY measure_value), 2) AS mode_value
    ROUND(STDDEV(measure_value), 2) AS standard_deviation
    ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight'; 
```

# Pearson Coefficients of Skewness

- *Coefficient1 :  ModelSkewness = $\frac{Mean - Mode}{StandardDeviation}$*

- *Coefficient2 : MedianSkewness = $\frac{Mean - Median}{StandardDeviation}$*

  ```sql
  WITH cte_blood_glucose_stats AS(
  	SELECT 
  		AVG(measure_value) AS mean_value,
  		PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
  		MODE() WITH GROUP (ORDER BY measure_value) AS mode_value
    	STDDEV(measure_value) AS stddev_value
    FROM health.user_logs
    WHERE measure = 'blood_glucose'
  )
  SELECT (mean_value - mode_value)/ stddev_value  AS pearson_corr_1,
  3 * (mean_value - median_value)/ stddev_value AS pearson_corr_2
  FROM cte_blood_glucose_stats
  ```

# Conclusion

- Central location statistics including: mean, median and mode

- Simple algorithm approach to calculate the median and mode

- Median is PERCENTILE_CONT(0.5 )- Ordered Set Aggregate Functions

- Spread statistics including: min, max, range, standard deviation and variance

  
