# Project Assumptions
* Only personal github account without any tie to a specific organization
* Upgrading to a paid plan requires explicit process and all the users start with a free account. 
# Logical Model

## Step 1: select the business process
1. Sign up for a free account
2. Convert from a free account to a paid account
3. Develop features that users like

## Step 2: declare the grain
|Business Process | Grain |
|---|---|
|Acquiring new users| user sign up with timestamp|
|Converting to a paid account| user changes from free to paid|
| Strategy for adding new features | market research, user survey, analyze converted/non-converted users |

## Step 3: identify dims
* Users
* Features
* Plans

## Step 4: identify facts
* fct_user_signup
* fct_plan
* fct_feature
* fct_event

# ER
![github-er](./github-er.png "Github ERD)"

# Business Questions
* How many new accounts signed up daily?
```sql
SELECT 	DATE_TRUNC(fct_user_signup.created_at, DAY) as signup_date,
		COUNT(user_id)
FROM fct_user_signup
GROUP BY 1
```
* How long does it take for an account to convert from a free account?

We can track the plan_type as a type 2 slowly changing dimension and calculate the differences between the valid_to and valid_from, group by the user_id and plan_type to find out how long it takes for an account to convert from a free account to a paid account. 

```sql
SELECT user_id, 
	   plan_type, 
	   COALESCE(valid_to, current_timestamp()) - valid_from  
FROM fct_plan
GROUP BY 1,2
```
* Which features do paid accounts tend to use more frequently than non-converted acccounts?

```sql
SELECT plan_type, count(*)
FROM 
	(SELECT UNNEST(features) FROM fct_feature
	LEFT JOIN dim_user ON dim_user.id = fct_feature.user_id
	LEFT JOIN fct_plan ON dim_user.id = fct_plan.user_id) t
GROUP BY 1
ORDER BY 2;
```