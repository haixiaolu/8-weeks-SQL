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

- A cross join creates a full combination of all the rows in both tables that are being joined. This type of join is also referred to as a cartesian join which is taken from the original mathematical transformation - the cartesian product of two sets!

- can be used quite effectively when you actually need all the combinations of 2 tables for specific purposes. 

  ```sql
  SELECT
  	names.iid as name_iid,
  	jobs.iid as job_iid,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names
  CROSS JOIN jobs;
  ```

- Cross joins also commonly seen with the following sysntax also without the `CROSS JOIN` and instead just using a comma to separate the tables to be cross joined

  ```sql
  SELECT
  	names.iid AS name_iid,
  	jobs.iid AS job_iid,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names, jobs;
  ```

  #### Alternative Cross Inner Join Syntax

  - Additionally - this is actually quite a common usage to `INNER JOIN` tables implicity using a `WHERE` filter instead of using the regular `INNER JOIN` and `ON` clauses. 
  - NOT recommend to implement joins in this way

  ```sql
  SELECT
  	names.iid as name_iid,
  	jobs.iid as job_iid,
  	names.first_name,
  	names.title,
  	jobs.occupation,
  	jobs.salary
  FROM names, jobs
  WHERE 
  	names.iid = jobs.iid;
  ```

  #### Combining Manual Input

  - Sometimes cross joins can also be used to create combinations of manual column values like in this following `CTE` example

  - `||` characters are called "pipes" and are used to concatenate or combine text strings together into a single text column. 

    ```sql
    WITH favorite_things (animal_name) AS (
    VALUES
    	('Purple Elephant'),
    	('Yellow Sea Cucumber'),
    	('Turquoise Gorilla'),
    	('Invisible Unicorn')
    )
    SELECT 
    	first_name || ' Likes ' || animal_name || 's!' AS text_output
    FROM names, favorite_things
    WHERE first_name = 'Danny';
    ```

# Advanced Joins

- `LEFT SEMI JOIN` or `WHERE EXISTS`
- `ANTI JOIN` or `WHERE NOT EXISTS`
- The main purpose of using these advanced joins is to **remove/exclude** rows from a table where specific column values based off a separate lookup table
- They also have the added benefit of helping eliminate some unexpected behavior when there are duplicates in the "target" table or in join operations.

## Dealing With Duplicates

- *Create a new table with duplicates*

  ```sql
  CREATE TEMP TABLE new_jobs AS
  WITH input_table (iid, occupation, salary) AS (
  	VALUES
  	(1, 'Cleaner', 'High'),
  	(1, 'Cleaner', 'Very High'),
  	(2, 'Janitor', 'Medium'),
  	(3, 'Monkey', 'Low'),
  	(3, 'Monkey', 'Very Low'),
  	(6, 'Plumber', 'Ultra'),
  	(7, 'Hero', 'Plus Ultra')
  )
  SELECT * FROM input_table;
  ```

  ​		<img src="/Users/haixiaolu/Desktop/Screen Shot 2021-10-24 at 1.45.26 PM.png" alt="Screen Shot 2021-10-24 at 1.45.26 PM" style="zoom:50%;" />

## Basic Joins With Duplicates

- Basic joins which we covered before will be forced to return every row per matching record. This is the case for both `LEFT JOIN` and `INNER JOIN`

- *Duplicate Inner Join*

  ```sql
  SELECT
  	names.iid,
  	names.first_name,
  	names.title,
  	new_jobs.occupation,
  	new_jobs.salary
  FROM names
  INNER JOIN new_jobs
  	ON names.iid = new_jobs.iid;
  ```

- *Duplicate Left Join*

  ```sql
  SELECT
  	names.iid,
  	names.first_name,
  	names.title,
  	new_jobs.occupation,
  	new_jobs.salary
  FROM names
  LEFT JOIN new_jobs
  	ON names.iid = new_jobs.iid;
  ```

## Left Semi Join

- A left semi join is actually really simiar to an `INNER JOIN` where it captures only the matching records between left and right **BUT** it differs in one very key way: it only returns records from the left table - no columns or rows from the right table are included in the output

- In PostgreSQL Standard SQL, this `LEFT SEMI JOIN` can be expressed using a `WHERE EXISTS` clause but in some other SQL flavours `LEFT SEMI JOIN` can be called directly also

- It quite a bit from a regular join operation with the `ON` clause

- The `WHERE` clause inside the surrounding `WHERE EXISTS` is the equivalent of the `ON` clause for a regular join syntax

  - *`WHERE EXISTS` Syntax*

    ```sql
    SELECT 
    	names.iid,
    	names.first_name
    FROM names
    WHERE EXISTS(
    	SELECT iid
    	FROM new_jobs
    	WHERE names.iid = new_jobs.iid
    );
    ```

    **The same query will run if I simply select `1` instead of the `iid` column**

    ```sql
    SELECT
    	names.iid,
    	names.first_name
    FROM names
    WHERE EXISTS (
    	SELECT 1
    	FROM new_jobs
    	WHERE names.iid = new_jobs.iid
    );
    
    ```

    **The key thing is that the query within the subquery part of the `WHERE EXISTS` does not actually return any data**

  - Some SQL practitioners will specifically use the `SELECT 1` in their `WHERE EXISTS` to make it clear that actually nothing is being returned

  ### `LEFT SEMI JOIN` Syntax

  - The following code will not run in PostgreSQL but might work in other flavours - it is the equivalent of the above `WHERE EXISTS` implementation

    ```sql
    SELECT
    	names.iid,
    	names.first_name
    FROM names
    LEFT SEMI JOIN new_jobs
    ON names.iid = new_jobs.iid;
    ```

## Alternative Left Semi Understanding

- Another way to think of this sort of join is in terms of a regular `LEFT JOIN` with a `DISTINCT` followed by a `WHERE` filter to remove all null values from the resulting joint dataset

- The following query is quite inefficient. **`DISTINCT` is the important point**

  ```sql
  SELECT DISTINCT
  	names.iid,
  	names.first_name
  FROM names
  LEFT JOIN new_jobs
  	ON names.iid = new_jobs.iid
  WHERE new_jobs.iid IS NOT NULL;
  ```

- This looks exactly the same as our previous `WHERE EXISTS` query

#### Left Semi Join Vs Inner Join

- Use an `INNER JOIN` to show this same `LEFT SEMI JOIN` operation, first with a `DISTINCT` and then without `DISTINCT`

  ```sql
  SELECT DISTINCT
  	names.iid,
  	names.first_name
  FROM names
  INNER JOIN new_jobs
  	ON names.iid = new_jobs.iid;
  ```

## Anti Join

- An `ANTI JOIN` is the opposite of a `LEFT SEMI JOIN` where only records from the left which do not appear on the right are returned - it is expressed as a `WHERE NOT EXISTS` clause in PostgreSQL but similar to the `LEFT SEMI JOIN` some SQL flavors support a direct `ANTI JOIN` syntax also

  ```sql
  SELECT 
  	names.iid,
  	names.first_name
  FROM names
  WHERE NOT EXISTS (
  	SELECT 1
  	FROM new_jobs
  	WHERE names.iid = new_jobs.iid
  );
  ```

  ### A Popular Alternative

  - Here is an alternative way to represent a similar `ANTI JOIN` using a `LEFT JOIN` with a null `WHERE` filter 

    ```sql
    SELECT
    	names.iid,
    	names.first_name
    FROM names
    LEFT JOIN new_jobs
    	ON names.iid = new_jobs.iid
    WHERE new_jobs.iid IS NULL;
    ```

  - This combination of `LEFT JOIN` and `IS NULL` implementation for anti joins are very popular in practive

## Joining on Null Values

- what would happen if we try to join tables which have `null` in the joining columns?

  ```sql
  DROP TABLE IF EXISTS null_names;
  CREATE TEMP TABLE null_names AS
  WITH input_data (iid, first_name, title) AS (
   VALUES
   (1,    'Kate',   'Datacated Visualizer'),
   (2,    'Eric',   'Captain SQL'),
   (3,    'Danny',  'Data Wizard Of Oz'),
   (4,    'Ben',    'Mad Scientist'),
   (5,    'Dave',   'Analytics Heretic'),
   (6,    'Ken',    'The YouTuber'),
   (null, 'Giorno', 'OG Data Gangster')
  )
  SELECT * FROM input_data;
  
  DROP TABLE IF EXISTS null_jobs;
  CREATE TEMP TABLE null_jobs AS
  WITH input_table (iid, occupation, salary) AS (
   VALUES
   (1,    'Cleaner',    'High'),
   (1,    'Cleaner',    'Very High'),
   (2,    'Janitor',    'Medium'),
   (3,    'Monkey',     'Low'),
   (3,    'Monkey',     'Very Low'),
   (6,    'Plumber',    'Ultra'),
   (7,    'Hero',       'Plus Ultra'),
   (null, 'Mastermind', 'Bank')
  )
  SELECT * FROM input_table;
  ```

  - *Inner Join*

    ```sql
    SELECT
    	names.iid,
    	names.first_name,
    	names.title,
    	jobs.occupation,
    	jobs.salary
    FROM null_names AS names
    INNER JOIN null_jobs AS jobs
    	ON names.iid = jobs.iid;
    ```

  - *Left Join*

    `null_names` as the base

    ```sql
    SELECT 
    	names.iid,
    	names.first_name,
    	names.title,
    	jobs.occupation,
    	jobs.salary
    FROM null_names AS names
    LEFT JOIN null_jobs AS jobs
    	ON names.iid = jobs.iid;
    ```

    `null_jobs` as the base

    ```sql
    SELECT
    	jobs.iid,
    	jobs.occupation,
    	jobs.salary,
    	names.first_name,
    	names.title
    FROM null_jobs AS jobs
    LEFT JOIN null_names AS names
    	ON jobs.iid = names.iid;
    ```

    **RESULT:** NO - it does not join (well not in the way that we were expecting anyway!)

## Left Semi Join

```sql
SELECT
	null_names.*
FROM null_names
WHERE EXISTS (
	SELECT iid
	FROM null_jobs
	WHERE null_names.iid = null_jobs.iid
);

```

​	**Result:** No - it does not join!

## Anti Join

```sql
SELECT
	null_names.*
FROM null_names
WHERE NOT EXISTS (
	SELECT iid
	FROM null_jobs
	WHERE null_names.iid = null_jobs.iid
);
```

​		**Result:** No - it does not join!

# IN and NOT IN

- `IN` and `NOT IN` clauses are very popular ways to filter datasets when used with a `WHERE` clause
- Logically an `IN` is essentially doing what a `WHERE EXISTS` is doing - and `NOT IN` is equivalent to an `ANTI JOIN` 

## IN instead of WHERE EXISTS

```sql
SELECT
	null_names.*
FROM null_names
WHERE null_names.iid IN(
	SELECT iid
	FROM null_jobs
);
```

# Set Operations

- There are 3 distinct set operations used to combine results from 2 or more `SELECT` statements
  - UNION (& UNION ALL)
  - INTERSECT
  - EXCEPT
- These 3 methods are pretty straightforward however as it matches directly with set notation, however the only condition is that all of the `SELECT` result sets must have columns with the same data types - the column names have no impact

## Union

- `UNION` will be the union between two sets or `SELECT` results

  ```sql
  SELECT * FROM names WHERE first_name = 'Danny'
  UNION
  SELECT * FROM names WHERE first_name = 'Kate';
  ```

## Union All

- **What is the difference between `UNION` and `UNION ALL`?**

  - `UNION` runs a `DISTINCT` on the output result whilst `UNION ALL` does not

  - This means that `UNION ALL` will be more performant as it does not run a relatively expensive `DISTINCT`

  - *POPULAR SQL INTERVIEW*

    ```sql
    -- USE UNION
    SELECT * FROM names where first_name = 'Danny'
    UNION
    SELECT * FROM names where first_name = 'Danny';
    ```

    ```sql
    -- USE UNION ALL
    SELECT * FROM names where first_name = 'Danny'
    UNION ALL
    SELECT * FROM names where first_name = 'Danny';
    ```

- A few things to consider when you need to combine the datasets in practice:

  - Do I have duplicates in my individual results set?
  - Do I have duplicates in my overall results set after `UNION ALL`?
  - Should I refactor the query to generate distinct values or use a `GROUP BY`

# Intersect

- Intersect is pretty straightforward in that only the records which exists in both tables will be returned - a little bit similar to an `INNER JOIN`

  ```sql
  SELECT * FROM names
  INTERSECT
  SELECT * FROM names where LEFT (first_name, 1) = 'K';
  ```

# Except

- Except is the equivalent of the `ANTI JOIN` in set notation

  ```sql
  SELECT * FROM names
  EXCEPT
  SELECT * FROM names where LEFT(first_name, 1) = 'K';
  ```

  - **NOTE** that `EXCEPT` has a default `DISTINCT` built in so it's doing the same deduplication as the `UNION`

# Multiple Combinations

- We can also combine these with 2+ queries like so - just be aware that all of these set operations will be executed in order from top to bottom

  ```sql
  SELECT * FROM names WHERE first_name = 'Danny'
  UNION
  SELECT * FROM names where LEFT(first_name, 1) = 'K'
  EXCEPT
  SELECT * FROM names where first_name = 'Kate';
  ```

# Common `UNION` Mistakes

#### Not-So-Silent Mistakes

- What happens when we have the wrong number of columns for a combination

  ```sql
  SELECT first_name FROM names WHERE first_name = 'Danny'
  UNION
  SELECT iid, first_name FROM names WHERE LEFT(first_name, 1) = 'K';
  ```

  - <img src="/Users/haixiaolu/Library/Application Support/typora-user-images/Screen Shot 2021-10-24 at 4.57.04 PM.png" alt="Screen Shot 2021-10-24 at 4.57.04 PM" style="zoom:50%;" />

#### Silent Mistakes

- What happens when we accidentally choose the wrong columns but their position and data types match?

  ```sql
  SELECT iid, first_name, title FROM names WHERE first_name = 'Danny'
  UNION
  SELECT iid, occupation, salary FROM jobs where iid = 2
  UNION
  SELECT * FROM names where first_name = 'Kate';
  ```

- Let's moving the second query to the first position

  ```sql
  SELECT iid, occupation, salary FROM jobs WHERE iid = 2
  UNION
  SELECT iid, first_name, title FROM names WHERE first_name = 'Danny'
  UNION
  SELECT * FROM names where first_name = 'Kate';
  ```

# Conclusion

- Use `LEFT JOIN` when you need to keep all rows on the left table but want to join on additional records from the right table
  - Be wary of which table is on the left vs right side of the `LEFT JOIN` as all left table records
- Use `INNER JOIN` when you need the intersection of both left and right tables and you also need to access columns from the right table
- Use `FULL OUTER JOIN` or `FULL JOIN` when you need to combine both left and right tables completely (similar to a `UNION ALL` operation without deduplication)
- Use `CROSS JOIN` when you need the cartesian product or the complete combinations of left and right tables
- Be careful of duplicate join keys from left and right tables when using `LEFT` and `INNER` and `CROSS` joins
- Use `LEFT SEMI JOIN` or `WHERE EXISTS` when you only need to keep records from the left table that exist in the right table
- Use `ANTI JOIN` when you need to remove exclude records from the left table based off the right table
- Use `UNION` to get the deduplicated or distinct combination of two tables or SQL query outputs - column positions and data types must align!
- Use `UNION` all to get the unduplicated combination of two tables or SQL query outputs - column positions and data types must align
- Use `INTERSECT` and `EXCEPT` for set operations between tables or SQL query outputs - column positions and data types must align
- =, >=, >, < or `BETWEEN` conditions can be used for the `ON` clause for table joins
- Use aliases and/or explicit references when joining tables and selecting columns to improve readability and comprehension of code, especially when joining multiple tables in one SQL statement

# Appendix

## Theta & Equi Joins

- Table joins require an `ON` clause that helps define the conditions where the table rows are matched in the final resulting output. 
- Equi or Equality joins have equivalence conditions on a single set of columns or multiple sets inside the `ON` clause. This is most commonly represented by having some column equal to another using the `=` operator after the `JOIN`, `INNER JOIN`, `LEFT JOIN`, `FULL JOIN`, `FULL OUTER JOIN`clause
- Theta joins are a more generic type join where the `ON` clause can consist of multiple types of comparison operators. 
- These can include the following types of comparisons and string operators:
  - strictly greater than or less than inequalities [`>`, `<`]
  - equal to and greater than or less than inequalities [`>=`, `<=`]
  - fuzzy string matching operators [case sensitive `LIKE` and case insensitive `ILIKE`]
  - ranges `BETWEEN` an small end range number/column and larger end range number/column
