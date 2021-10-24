# Joins

## Sample Datasets

- `CREATE TEMP TABLE` statement combined with a CTE

  ```sql
  DROP TABLE IF EXISTS names;
  CREATE TEMP TABLE names AS
  WITH input_data (iid, first_name, title) AS (
  	VALUES
  	(1, 'Kate', 'Datacated Visualizer'),
  	(2, 'Eric', 'Captain SQL'),
  	(3, 'Danny', 'Data Wizard OF Oz'),
  	(4, 'Ben', 'Mad Scientist'),
  	(5, 'Dave', 'Analytics Heretic'),
  	(6, 'Ken', 'The YouTuber')
  )
  SELECT * FROM input_data;
  
  DROP TABLE IF EXISTS jobs;
  CREATE TEMP TABLE jobs AS(
  	VALUES
  	(1, 'Cleaner', 'High'),
  	(2, 'Janitor', 'Medium'),
  	(3, 'Monkey', 'Low'),
  	(6, 'Plumber', 'Ultra'),
  	(7, 'Hero', 'Plus Ultra')
  )
  SELECT * FROM input_data;
  ```

- **Inspecting the `names` table**

  ```sql
  SELECT * FROM names;
  ```

- **Inspecting the `jobs` table**

  ```sql
  SELECT * FROM jobs;
  ```

## Basic Table Joins

- `INNER JOIN` or `JOIN`
- `LEFT JOIN` or `LEFT OUTER JOIN`
- `FULL JOIN` or `FULL OUTER JOIN`

### The Inner Join

- An `INNER JOIN` or `JOIN` is used to get the intersection between two tables and return only the matching records. 

- Ex, we join `names` and `jobs` on the `idd` column to get first names and occupation columns in the same query

  ```sql
  SELECT
  	names.idd,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names
  INNER JOIN jobs
  ON names.idd = jobs.idd
  ```

  **NOTICE** how only 4 records remain once we perform the `INNER JOIN` - this is essentially performing an intersection of the two datasets which meets the condition inside the `ON` clause

  

  ### Inner Join & Join Difference

  - `INNER JOIN` and `JOIN` are actually the same thing and will execute exactly the same
  - However - for readability and explicitness - it is recommended to always be specific about which type of join you are using so I would actually suggest to avoid using `JOIN` by itself and to use `INNER JOIN` instead. 

  ### Table & Column References

  - If we don't list out all of the columns in the `SELECT` statement for the inner join

  - Then the output will have **two ON CONDITION (idd)**

    - ```sql
      SELECT *
      FROM names
      INNER JOIN jobs
      ON names.iid = jobs.iid;
      ```

    - ```sql
      SELECT
      	names.*,
      	jobs.*
      FROM names
      INNER JOIN jobs
      	ON names.iid = jobs.iid;
      ```

  ### No Table References

  - if no table reference in the query, `iid` instead of `names.iid`, it shows error: `column reference "id" is ambiguous`

### Left Join

- The `LEFT JOIN` or `LEFT OUTER JOIN` is used when you want to **keep all the rows or records from the *left* table and return any matching records from the right table**

- Note that these two terms are exactly the same but usually used as simply a `LEFT JOIN`

-  One of the fundamental concepts for this type of join is that the table on the left is also known as the "base" table where all of the rows are retained

- The "target" or right hand side table for the join will only return values when there is a match with records from the `base` table. When there is no match - the values for the specific "target" columns will be `null` for the non-matching rows. 

  #### Basic Left Join Implementation

  ```sql
  SELECT
  	names.iid,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names
  LEFT JOIN jobs
  	ON names.iid = jobs.iid;
  ```

  ​		**NOTE** how there are `null` values for the `occupation` and `salary` columns for Ben and Dave with respective `iid` values of 4, 5

  - `null` records for non-matching rows for both the `occupation` and `salary` columns which originate from the `jobs` table

### Left Join Table Order

- Swap the left and right tables, now jobs is on the left hand side as the "base" table and names is now the "target" table on the right?

  ```sql
  SELECT
  	jobs.iid,
  	jobs.occupation,
  	jobs.salary,
  	names.first_name,
  	names.title
  FROM jobs
  LEFT JOIN names
  	ON jobs.iid = names.iid;
  ```

  **Notice** how the rows of the resulting output has switched to match the new "base" `jobs` table when we swap the order of the tables as they appear in the left join. 

### Full Join or Full Outer Join

- Need to get the full combination of both tables to include all the rows and columns

- You might also want the same behavior as the previous left join but have both tables used as a "base" table where all the rows are kept from both tables. 

- This is where the **full join** or **full outer join** comes into play - they are the same thing. 

  ```sql
  SELECT
  	names.iid AS name_id,
  	jobs.iid AS job_id,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names
  FULL JOIN jobs
  	ON names.iid = jobs.iid;
  ```

  <img src="/Users/haixiaolu/Library/Application Support/typora-user-images/Screen Shot 2021-10-23 at 10.34.57 PM.png" alt="Screen Shot 2021-10-23 at 10.34.57 PM" style="zoom:50%;" />

- In the output above we can see how there are `null` values present in both the `name_id` and the `job_id` columns

- This behaviour mimics the same retention of rows in the `LEFT JOIN` we saw previously - but both sets of table rows are kept regardless whether they match or not with the adjoining table. 

### Table Aliases For Joins

- There aliases are exactly the same as when we used aliases to rename our columns but this time we can do it with entire tables(as well as subqueries and CTEs too!!!)

- A few different competing theories about how we properly name our table aliases

  #### First Letter Aliasing

  - Sometimes we use the first letter to alias tables, but when you have multiple tables taht start with the same letter as you cannot assign the same alias more than once. 

    ```sql
    SELECT
    	n.iid as name_iid,
    	j.iid as job_iid,
    	n.first_name,
    	n.title,
    	j.occupation,
    	j.salary
    FROM names AS n
    FULL JOIN jobs AS j
    	ON n.iid = j.iid;
    ```

  #### Logical Aliasing

  - Use logical names to alias tables to make it easier for the person reading the code to comprehend what's going on

    ```sql
    SELECT
    	sales.user_id,
    	names.first_name,
    	names.last_name
    FROM financial.customer_sales_daily AS sales
    INNER JOIN financial.customer_name_details AS names
    	ON sales.customer_id = names.customer_id
    ```

  #### Appearance Order Aliasing

  - This method is to just pop an arbitrary letter of `t` in front of each table (**from 1 to the number of table**)

    ```sql
    SELECT
    	t1.iid AS name_iid,
    	t2.iid AS job_iid,
    	t1.first_name,
    	t1.title,
    	t2.occupation,
    	t2.salary
    FROM names AS t1
    FULL JOIN jobs AS t2
    	ON t1.iid = t2.iid;
    ```

### Cross Join

