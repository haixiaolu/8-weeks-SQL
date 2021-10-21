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

