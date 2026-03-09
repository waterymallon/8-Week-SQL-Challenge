# Case Study #3 - Foodie Fi

## Table of Contents

* [Datasets & ERD](#datasets)
* [A. Customer Journey](#a)
* [B. Data Analysis Questions](#b)
* [C. Challenge Payment Question](#c)
* [D. Outside The Box Questions](#d)

## <a id = 'datasets'></a> Datasets & ERD

Please run the [schema query](./case3_schema.sql) to create the Datasets.

This case study provides only two tables: `plans`, `subscriptions`<br>
Note that `subscriptions` has more than 1000 rows

<img width="698" height="290" alt="image" src="https://github.com/user-attachments/assets/61ba3e9a-3444-4ae1-a2f5-7597da4891f7" />

<details>
  <summary> plans </summary>
| plan_id | plan_name     | price |
|---------|---------------|-------|
| 0       | trial         | 0     |
| 1       | basic monthly | 9.90  |
| 2       | pro monthly   | 19.90 |
| 3       | pro annual    | 199   |
| 4       | churn         | null  |
</details>

<details>
  <summary> subscriptions </summary>
| customer_id | plan_id | start_date |
|-------------|---------|------------|
| 1           | 0       | 2020-08-01 |
| 1           | 1       | 2020-08-08 |
| 2           | 0       | 2020-09-20 |
| 2           | 3       | 2020-09-27 |
| 11          | 0       | 2020-11-19 |
| 11          | 4       | 2020-11-26 |
| 13          | 0       | 2020-12-15 |
| 13          | 1       | 2020-12-22 |
| 13          | 2       | 2021-03-29 |
| 15          | 0       | 2020-03-17 |
| 15          | 2       | 2020-03-24 |
| 15          | 4       | 2020-04-29 |
| 16          | 0       | 2020-05-31 |
| 16          | 1       | 2020-06-07 |
| 16          | 3       | 2020-10-21 |
| 18          | 0       | 2020-07-06 |
| 18          | 2       | 2020-07-13 |
| 19          | 0       | 2020-06-22 |
| 19          | 2       | 2020-06-29 |
| 19          | 3       | 2020-08-29 |
  
</details>

## Case Study Questions

### <a id = 'a'></a> A. Customer Journey
Based off the 8 sample customers provided in the sample from the `subscriptions` table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```sql
select s.*, plan_name, price
from subscriptions s
inner join plans p
	on s.plan_id = p.plan_id
where customer_id in (1, 2, 11, 13, 15, 16, 18, 19)
```

| customer_id | plan_id | start_date | plan_name     | price  |
|---------------|---------|------------|---------------|--------|
| 1             | 0       | 2020-08-01 | trial         | 0.00   |
| 1             | 1       | 2020-08-08 | basic monthly | 9.90   |
| 2             | 0       | 2020-09-20 | trial         | 0.00   |
| 2             | 3       | 2020-09-27 | pro annual    | 199.00 |
| 11            | 0       | 2020-11-19 | trial         | 0.00   |
| 11            | 4       | 2020-11-26 | churn         | NULL   |
| 13            | 0       | 2020-12-15 | trial         | 0.00   |
| 13            | 1       | 2020-12-22 | basic monthly | 9.90   |
| 13            | 2       | 2021-03-29 | pro monthly   | 19.90  |
| 15            | 0       | 2020-03-17 | trial         | 0.00   |
| 15            | 2       | 2020-03-24 | pro monthly   | 19.90  |
| 15            | 4       | 2020-04-29 | churn         | NULL   |
| 16            | 0       | 2020-05-31 | trial         | 0.00   |
| 16            | 1       | 2020-06-07 | basic monthly | 9.90   |
| 16            | 3       | 2020-10-21 | pro annual    | 199.00 |
| 18            | 0       | 2020-07-06 | trial         | 0.00   |
| 18            | 2       | 2020-07-13 | pro monthly   | 19.90  |
| 19            | 0       | 2020-06-22 | trial         | 0.00   |
| 19            | 2       | 2020-06-29 | pro monthly   | 19.90  |
| 19            | 3       | 2020-08-29 | pro annual    | 199.00 |

- Filter customer_id, We only need to describe 1, 2, 11, 13, 15, 16, 18, 19
- Join with `plans` to check how each customer journey differs.
    - For example, customer 19 went on trial for a week starting on 2020-06-22. Started pro monthly subscription at the end of the trial, then subscribed pro annual on 2020-08-29

### <a id = 'b'></a> B. Data Analysis Questions

#### 1. How many customers has Foodie-Fi ever had?

```sql
select count(distinct customer_id) as customer_count
from subscriptions
```

| customer_count |
|------------------|
| 1000          |

#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
select month(start_date) as trial_month, count(*)
from subscriptions
where plan_id = 0
group by trial_month
order by trial_month
```

| trial_month | count(*) |
|---------------|----------|
| 1             | 88       |
| 2             | 68       |
| 3             | 94       |
| 4             | 81       |
| 5             | 88       |
| 6             | 79       |
| 7             | 89       |
| 8             | 88       |
| 9             | 87       |
| 10            | 79       |
| 11            | 75       |
| 12            | 84       |

- Filter by trial plans's id 0
- Group by month of `start_date`

#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

```sql
select s.plan_id, plan_name, count(*)
from subscriptions s
inner join plans p
	on s.plan_id = p.plan_id
where start_date >= '2021-01-01'
group by plan_id, plan_name
order by plan_id
```

| plan_id | plan_name     | count(*) |
|-----------|---------------|----------|
| 1         | basic monthly | 8        |
| 2         | pro monthly   | 60       |
| 3         | pro annual    | 63       |
| 4         | churn         | 71       |

- After the year 2020, which means start of the year 2021. Filter dates that is same or after 2021-01-01
- Group by plan_id and plan_name to show counts per plan.
- It seems trial plan did not happen at 2021

#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
select count(distinct customer_id) as churned_count,
(select count(distinct customer_id) from subscriptions) as total_count,
round(count(distinct customer_id) / (select count(distinct customer_id) from subscriptions) * 100, 1) as percentage
from subscriptions
where plan_id = 4
```

| churned_count | total_count | percentage |
|-----------------|-------------|------------|
| 307             | 1000        | 30.7       |

- Filter by plan_id 4 to get churned customers.
- Ensure customer_id is distinct to avoid double-counting.
- Used sub query for total customer count.

#### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
with cte as(
select customer_id, group_concat(plan_id)
from subscriptions
group by customer_id
having group_concat(plan_id order by start_date) = '0,4'
)

select count(distinct customer_id) as trial_churn_count,
(select count(distinct customer_id) from subscriptions) as total_count,
round((count(distinct customer_id) / (select count(distinct customer_id) from subscriptions)) * 100, 0) as percentage
from cte
```

- The plan has to be 0 (trial) to 4 (churn).
- First solution used `group_concat` to filter trial to churn plans.
  - Order by start_date to ensure chronological order of the customer's history.
- `group_concat` has max length limits, using this may not be the best solution.

```sql
with cte as(
select s.*, plan_name,
lead(plan_name) over (partition by customer_id order by start_date) as next_plan
from subscriptions s
inner join plans p
	on s.plan_id = p.plan_id
)

select count(*) as trial_churn_count,
(select count(distinct customer_id) from subscriptions) as total_count,
round((count(*) / (select count(distinct customer_id) from subscriptions)) * 100, 0) as percentage
from cte
where plan_name = 'trial' and next_plan = 'churn'
```

| trial_churn_count | total_count | percentage |
|---------------------|-------------|------------|
| 92                  | 1000        | 9          |

- Second solution used `lead()` to filter trial to churn plans.
  - Order by start_date to ensure chronological order of the customer's history.
- Joined `plans` to filter by plan_name

#### 6. What is the number and percentage of customer plans after their initial free trial?

```sql
with cte as (
select s.*, plan_name,
lead(plan_name) over (partition by customer_id order by start_date) as next_plan
from subscriptions s
inner join plans p
	on s.plan_id = p.plan_id
)

select next_plan, count(next_plan),
round(count(next_plan) * 100 / sum(count(next_plan)) over(), 1) as percentage
from cte
where plan_name = 'trial' and next_plan is not null
group by next_plan 
```

| next_plan   | count(next_plan) | percentage |
|---------------|------------------|------------|
| basic monthly | 546              | 54.6       |
| churn         | 92               | 9.2        |
| pro annual    | 37               | 3.7        |
| pro monthly   | 325              | 32.5       |

- Group by next_plan to get count and percentage per plan.
- Use `lead()` to identify the next plan immediately following the initial free trial.
- Use either a subquery or `sum(count(next_plan)) over()` to retrieve the grand total required for the percentage denominator.

#### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
-- 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
with cte as(
select s.*,
lead(start_date) over (partition by customer_id order by start_date) as end_date
from subscriptions s
where start_date <= '2020-12-31'
)

select plan_id, count(*),
round(count(*) * 100 / sum(count(*)) over(), 1) as percentage
from cte
where end_date >= '2020-12-31' or end_date is null
group by plan_id
order by plan_id
```
| plan_id | count(*) | percentage |
|-----------|----------|------------|
| 0         | 19       | 1.9        |
| 1         | 224      | 22.4       |
| 2         | 327      | 32.7       |
| 3         | 195      | 19.5       |
| 4         | 236      | 23.6       |

- Use `lead()` to identify next start_date.
- Filter all plans that starts before 2020-12-31 and ends (or does not end which is `null`) after 2020-12-31

#### 8. How many customers have upgraded to an annual plan in 2020?

#### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

#### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

#### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

### <a id = 'c'></a> C. Challenge Payment Question


### <a id = 'd'></a> D. Outside The Box Questions
