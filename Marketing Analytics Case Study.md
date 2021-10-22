# Key Business Requirements

## 1.1 Requirement #1



**Top 2 Categories** For each customer, we need to identify the top 2 categories for each customer based off their past rental history. 

## 1.2 Requirement #2

**Category Film Recommendations**  The marketing team also requested for the 3 most popular films for each customer's top 2 categories. BUT we cannot recommend a film which the customer has already viewed. If there are less than 3 films availabel - marketing is happy to show at least 1 film. 

*Any customer which do not have any film recommendations for either category must ve flagged out so the marketing team can exclude from the email campaign - high importance*

## 1.3 Requirement #3 & #4

**Individual Customer Insights** The number of films wathed by each customer in their top 2 categories is required as well as some specific insights.

For the 1st category, the marketing requires the following insights:

- How many total films have they watched in their top category
- How many more film has the customer watched compared to the average DVD Rental CO customer?
- How does the customer rank in terms of the top X% compared to all other customers in this film category?

For the second ranking category:

- How many total films has the customer watched in this category
- What proportion of each customer's total films watched does this count make

## 1.4 Requirement 5

**Favorite Actor Recommendations** Marketing has also requested top actor film recommendations where up to 3 more films are included in the recommendations list as well as the count of films by the top actor. 

Adviced to choose the actors in alphabetical order should there be any ties

In addition any films that have already been recommended in the top 2 categories must not be included as actor recommendations

If the customer doesn't have at least 1 fillm recommendation - they also need to be flagged with a separate actor exclusing flag. 



# Data Exploration

## ERD Diagrams Using *dbdiagram.io*

```sql
Table "rental"{
	"rental_id" integer [not null]
	"rental_date" timestamp [not null]
	"inventory_id" integer [not null]
	"customer_id" smallint [not null]
	"return_date" timestamp
	"staff_id" smallint [not null]
	"last_update" timestamp [not null, default: `now()`]
	
Indexes{
	inventory_id [type: btree, name: "idx_fk_inventory_id"]
	(rental_date, inventory_id, customer_id) [type: btree, unique, name: "idx_unq_rental_rental_date_inventory_id_customer_id"]
}
	}
	
Table "inventory" {
	"inventory_id" integer [not null, default: `nextval('inventory_inventory_id_seq'::regclass)`]
	"film_id" smallint [not null]
	"store_id" smallint [not null]
	"last_update" timestamp [not null, default:`now()`]
	
Indexes {
	(store_id, film_id) [type: btree, name: "idx_store_id_film_id"]
}
}

Table "film"{
	"film_id" integer [not null, default: `nextval('film_film_id_seq'::regclass)`]
	"title" "character varying(255)" [not null]
	"description" text
	"release_year" year
	"language_id" smallint [not null]
	"original_language_id" smallint
	"rental_duration" smallint [not null, default: 3]
	"rental_rate" "numeric(4, 2)"[not null, default: 4.99]
	"length" smallint
	"replacement_cost""numeric(5,2)"[not null, default:19.99]
	"rating" "character varying(5)"[default: "G"]
	"last_update" timestamp [not null, default: `now()`]
	"special_features" text
	"fulltext" tsvector [not null]
	
Indexes {
	fulltext [type: btree, name: "film_fulltext_idx"]
	language_id [type: btree, name: "idx_fk_language_id"]
	original_language_id [type:btree, name: "idx_fk_original_language_id"]
	title [type: btree, name: "idx_title"]
}
}

Table "film_category" {
  "film_id" smallint [not null]
  "category_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]
}

Table "category" {
  "category_id" integer [not null, default: `nextval('category_category_id_seq'::regclass)`]
  "name" "character varying(25)" [not null]
  "last_update" timestamp [not null, default: `now()`]
}

Table "film_actor" {
  "actor_id" smallint [not null]
  "film_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  film_id [type: btree, name: "idx_fk_film_id"]
}
}

Table "actor" {
  "actor_id" integer [not null, default: `nextval('actor_actor_id_seq'::regclass)`]
  "first_name" varchar(45) [not null]
  "last_name" varchar(45) [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  last_name [type: btree, name: "idx_actor_last_name"]
}
}

-- many to one relationship between rental & inventory
Ref: "rental"."inventory_id" > "inventory"."inventory_id"

-- many to one inventory to film
Ref: "inventory"."film_id" > "film"."film_id"

-- one to one relationship between film_category and film 
Ref: "film_category"."film_id" - "film"."film_id"

-- many to one relationship between film_category and category
Ref: "film_category"."category_id" > "category"."category_id"

-- one to many relationship between film ands film_actor
Ref: "film"."film_id" < "film_actor"."film_id"

-- many to one relationship between film_actor and actor
Ref: "film_actor"."actor_id" > "actor"."actor_id"

-- there is also an additional relationship, however we exclude it to reduce
-- any confusion!

-- many to many relationship between inventory and film_actor
-- however dbdiagram.io only lets you refer to each combination once...
-- so we only show one direction of relationship!
-- Ref: "inventory"."film_id" > "film_actor"."film_id"
```



## Embedded Interactive ERD

```markdown
<iframe height="400" width="100%"
src='https://dbdiagram.io/embed/5fe1cb6e9a6c525a03bbf839'>
</iframe>

```



<img src="/Users/haixiaolu/Library/Application Support/typora-user-images/Screen Shot 2021-10-21 at 8.37.58 PM.png" alt="Screen Shot 2021-10-21 at 8.37.58 PM" style="zoom:50%;" />



# Apporaching the Data Problem

- mentally walk through exactly what data points we need for our final outputs, then systematically work backwards. 
- We will follow this reverse-engineering strategy for the entire case study

## Defining The Target State

- A counter-intuitive approach is to look at the final end user/business output that we are required to generate
- In the marketing analytics problem - the actual end result will be a completed email template for each customer

## Top Categories Information

- `category_name`: The name of the top or second ranking category

- `rental_count`: How many total films have they watched in this category?

- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer

- `percentil`: How does the customer rank in terms of the top X% compared to all other customers in this film category?

- `category_percentage`: What proprotion of each customer's total films watched does this count make

  **The top category insight will use these inputs in the folowing text output:**

  *You've watched {`rental_count`}{`category_name`} films, that's{`average_comparison`} more than the Dvd Rental Co average and puts you in the top{`percentile`}% of {`category_name`} gurus!*

  **The second category insight text output uses the fields in a similar way**

  *You've watched {`rental_count`}{`category_name`} films making up {`category_percentage`}% of your entire viewing history!*
