# Business Questions

## 1. How many unique users exist in the logs dataset?

```sql
SELECT
	COUNT(DISTINCT id)
FROM health.user_logs;
```



## 2. How many total measurements do we have per user on average?

```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS(
	SELECT
		id,
		COUNT(*) AS mesure_count,
		COUNT(DISTINCT measure) AS unique_measures
	FROM health.user_logs
	GROUP BY 1);
	
SELECT 
	ROUND(AVG(measure_count)) AS avgerage_value
FROM user_measure_count;
```



## 3. What about the median number of measurements per user?

```sql
SELECT 
	PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_count) AS median_value
	
FROM user_measure_count;
```



## 4. How many users have 3 or more measurements?

```sql
SELECT 
	COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
```



## 5. How many users have 1000 or more measurements?

```sql
SELECT 
	COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000;
```



### Looing at the logs data - what is the number and percentage of the active user base who:

## 6. Have logged blood glucose measurements?

```sql
SELECT 
	COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure='blood glucose'
```



## 7. Have at least 2 types of measurements?

```sql
SELECT 
	COUNT(*)
FROM user_measure_count
WHERE unique_measures >= 2;
```



## 8. Have all 3 measures - blood glucose, weight and blood pressure?

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures = 3;
```



### For users that have blood pressure measurements:

## 9. what is the median systolic/diastolic blood pressure values?

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure = 'blood_pressure';
```



