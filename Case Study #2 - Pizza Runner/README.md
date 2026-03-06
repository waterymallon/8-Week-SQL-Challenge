# Case Study #2 - Pizza Runner

<img width="50%" alt="image" src="https://github.com/user-attachments/assets/c409a6a4-a453-431d-9d71-56ede291a64d" />

## Table of Contents

* [Datasets & ERD](https://www.google.com/search?q=%23datasets)
* [Data Cleaning](https://www.google.com/search?q=%23cleaning)
* [A. Pizza Metrics](https://www.google.com/search?q=%23a)
* [B. Runner and Customer Experience](https://www.google.com/search?q=%23b)
* [C. Ingredient Optimisation](https://www.google.com/search?q=%23c)
* [D. Pricing and Ratings](https://www.google.com/search?q=%23d)
* [E. Bonus Questions](https://www.google.com/search?q=%23e)

## <a id="datasets"></a> Datasets & ERD

Please run the [schema query](https://www.google.com/search?q=https://github.com/waterymallon/8-Week-SQL-Challenge/blob/main/Case%2520Study%2520%25232%2520-%2520Pizza%2520Runner/case2_schema.sql) to create the Datasets.


A total of 6 tables are provided for this case.

<details>
<summary> customer_orders </summary>

| order_id | customer_id | pizza_id | exclusions | extras | order_time |
| --- | --- | --- | --- | --- | --- |
| 1 | 101 | 1 |  |  | 2020-01-01 18:05:02 |
| 2 | 101 | 1 |  |  | 2020-01-01 19:00:52 |
| 3 | 102 | 1 |  |  | 2020-01-02 23:51:23 |
| 3 | 102 | 2 |  | NULL | 2020-01-02 23:51:23 |
| 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46 |
| 4 | 103 | 1 | 4 |  | 2020-01-04 13:23:46 |
| 4 | 103 | 2 | 4 |  | 2020-01-04 13:23:46 |
| 5 | 104 | 1 | null | 1 | 2020-01-08 21:00:29 |
| 6 | 101 | 2 | null | null | 2020-01-08 21:03:13 |
| 7 | 105 | 2 | null | 1 | 2020-01-08 21:20:29 |
| 8 | 102 | 1 | null | null | 2020-01-09 23:54:33 |
| 9 | 103 | 1 | 4 | 1, 5 | 2020-01-10 11:22:59 |
| 10 | 104 | 1 | null | null | 2020-01-11 18:34:49 |
| 10 | 104 | 1 | 2, 6 | 1, 4 | 2020-01-11 18:34:49 |

</details>

<details>
<summary> pizza_names </summary>

| pizza_id | pizza_name |
| --- | --- |
| 1 | Meatlovers |
| 2 | Vegetarian |

</details>
<details>
<summary> pizza_recipes </summary>

| pizza_id | toppings |
| --- | --- |
| 1 | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2 | 4, 6, 7, 9, 11, 12 |

</details>

<details>
<summary> pizza_toppings </summary>

| topping_id | topping_name |
| --- | --- |
| 1 | Bacon |
| 2 | BBQ Sauce |
| 3 | Beef |
| 4 | Cheese |
| 5 | Chicken |
| 6 | Mushrooms |
| 7 | Onions |
| 8 | Pepperoni |
| 9 | Peppers |
| 10 | Salami |
| 11 | Tomatoes |
| 12 | Tomato Sauce |

</details>

<details>
<summary> runner_orders </summary>

| order_id | runner_id | pickup_time | distance | duration | cancellation |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 2020-01-01 18:15:34 | 20km | 32 minutes |  |
| 2 | 1 | 2020-01-01 19:10:54 | 20km | 27 minutes |  |
| 3 | 1 | 2020-01-03 00:12:37 | 13.4km | 20 mins | NULL |
| 4 | 2 | 2020-01-04 13:53:03 | 23.4 | 40 | NULL |
| 5 | 3 | 2020-01-08 21:10:57 | 10 | 15 | NULL |
| 6 | 3 | null | null | null | Restaurant Cancellation |
| 7 | 2 | 2020-01-08 21:30:45 | 25km | 25mins | null |
| 8 | 2 | 2020-01-10 00:15:02 | 23.4 km | 15 minute | null |
| 9 | 2 | null | null | null | Customer Cancellation |
| 10 | 1 | 2020-01-11 18:50:20 | 10km | 10minutes | null |

</details>

<details>
<summary> runners </summary>

| runner_id | registration_date |
| --- | --- |
| 1 | 2021-01-01 |
| 2 | 2021-01-03 |
| 3 | 2021-01-08 |
| 4 | 2021-01-15 |

</details>

<img width="690" height="566" alt="image" src="https://github.com/user-attachments/assets/d42c9a48-5082-45be-9461-d7d00b12fba3"/>


## <a id="cleaning"></a> Data Cleaning

Looking at the tables, instead of actual `NULL` values, columns contain the string value "null", empty spaces "", or unnecessary character values. Create a temporary table (`TEMPORARY TABLE`) after appropriately converting them using `CASE` statements.

### 1. customer_orders

Convert the string values "null" or empty spaces "" in the `exclusions` and `extras` columns to `NULL`.

```sql
create temporary table customer_orders_temp as
select order_id, customer_id, pizza_id,
case
    when exclusions = "" or exclusions = "null" then null
    else exclusions end as exclusions,
case 
    when extras = "" or extras = "null" then null
    else extras end as extras,
order_time
from customer_orders;


```

| order_id | customer_id | pizza_id | exclusions | extras | order_time |
| --- | --- | --- | --- | --- | --- |
| 1 | 101 | 1 | NULL | NULL | 2020-01-01 18:05:02 |
| 2 | 101 | 1 | NULL | NULL | 2020-01-01 19:00:52 |
| 3 | 102 | 1 | NULL | NULL | 2020-01-02 23:51:23 |
| 3 | 102 | 2 | NULL | NULL | 2020-01-02 23:51:23 |
| 4 | 103 | 1 | 4 | NULL | 2020-01-04 13:23:46 |
| 4 | 103 | 1 | 4 | NULL | 2020-01-04 13:23:46 |
| 4 | 103 | 2 | 4 | NULL | 2020-01-04 13:23:46 |
| 5 | 104 | 1 | NULL | 1 | 2020-01-08 21:00:29 |
| 6 | 101 | 2 | NULL | NULL | 2020-01-08 21:03:13 |
| 7 | 105 | 2 | NULL | 1 | 2020-01-08 21:20:29 |
| 8 | 102 | 1 | NULL | NULL | 2020-01-09 23:54:33 |
| 9 | 103 | 1 | 4 | 1, 5 | 2020-01-10 11:22:59 |
| 10 | 104 | 1 | NULL | NULL | 2020-01-11 18:34:49 |
| 10 | 104 | 1 | 2, 6 | 1, 4 | 2020-01-11 18:34:49 |

### 2. runner_orders

Handle noisy data in `pickup_time`, `distance`, `duration`, and `cancellation`.

* **duration**: Use `regexp_replace` to remove 'mins', 'minutes', 'minute' and remove spaces
* **distance**: Remove the 'km' characters

```sql
create temporary table runner_orders_temp
select order_id, runner_id,
case
    when pickup_time = "" or pickup_time = "null" then null
    else pickup_time
    end as pickup_time,
case
    when distance = "" or distance = "null" then null
    else replace(distance, "km", "")
    end as distance,
case
    when duration = "" or duration = "null" then null
    else replace(regexp_replace(duration, "mins|minutes|minute", ""), " ", "")
    end as duration,
case
    when cancellation = "" or cancellation = "null" then null
    else cancellation
    end as cancellation
from runner_orders;


```

| order_id | runner_id | pickup_time | distance | duration | cancellation |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 2020-01-01 18:15:34 | 20 | 32 | NULL |
| 2 | 1 | 2020-01-01 19:10:54 | 20 | 27 | NULL |
| 3 | 1 | 2020-01-03 00:12:37 | 13.4 | 20 | NULL |
| 4 | 2 | 2020-01-04 13:53:03 | 23.4 | 40 | NULL |
| 5 | 3 | 2020-01-08 21:10:57 | 10 | 15 | NULL |
| 6 | 3 | NULL | NULL | NULL | Restaurant Cancellation |
| 7 | 2 | 2020-01-08 21:30:45 | 25 | 25 | NULL |
| 8 | 2 | 2020-01-10 00:15:02 | 23.4 | 15 | NULL |
| 9 | 2 | NULL | NULL | NULL | Customer Cancellation |
| 10 | 1 | 2020-01-11 18:50:20 | 10 | 10 | NULL |

---

## Case Study Questions

### <a id ="a"></a> A. Pizza Metrics

#### 1. How many pizzas were ordered?

```sql
select count(*)
from customer_orders;


```

| count(*) |
| --- |
| 14 |

---

#### 2. How many unique customer orders were made?

```sql
select count(distinct order_id)
from customer_orders;


```

| count(distinct order_id) |
| --- |
| 10 |

* This is the number of unique orders. Even if one person orders multiple pizzas, it is counted as one `order_id`.

---

#### 3. How many successful orders were delivered by each runner?

```sql
select runner_id, count(*)
from runner_orders_temp
where cancellation is null
group by runner_id;


```

| runner_id | count(*) |
| --- | --- |
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |

* Group by `runner_id` for each runner.

---

#### 4. How many of each type of pizza was delivered?

```sql
select pizza_name, count(*) as delivered_counts
from customer_orders_temp c
inner join runner_orders_temp r 
    on c.order_id = r.order_id
    and cancellation is null
inner join pizza_names p 
    on c.pizza_id = p.pizza_id
group by pizza_name;


```

| pizza_name | count(*) |
| --- | --- |
| Meatlovers | 9 |
| Vegetarian | 3 |

* Group by `pizza_name` for each pizza.

---

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
select customer_id, pizza_name, count(*) as order_counts
from customer_orders_temp c
inner join pizza_names p 
    on c.pizza_id = p.pizza_id
group by customer_id, pizza_name
order by customer_id, pizza_name;


```

| customer_id | pizza_name | order_counts |
| --- | --- | --- |
| 101 | Meatlovers | 2 |
| 101 | Vegetarian | 1 |
| 102 | Meatlovers | 2 |
| 102 | Vegetarian | 1 |
| 103 | Meatlovers | 3 |
| 103 | Vegetarian | 1 |
| 104 | Meatlovers | 3 |
| 105 | Vegetarian | 1 |

* Since this is based on "ordered", delivery cancellations are not considered.

---

#### 6. What was the maximum number of pizzas delivered in a single order?

```sql
with cte as(
select c.order_id, count(*) as count
from customer_orders_temp c
inner join runner_orders_temp r 
    on c.order_id = r.order_id
    and cancellation is null
group by order_id
)

select max(count)
from cte;


```

| max(count) |
|--------------| 
| 3            |

---
#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
select customer_id,
sum(case 
        when exclusions is null and extras is null then 1
        else 0 end) as no_changes,
sum(case
        when exclusions is null and extras is null then 0
        else 1 end) as changes
from customer_orders_temp c
inner join runner_orders_temp r
    on c.order_id = r.order_id
    and cancellation is null
group by customer_id;


```

| customer_id | no_changes | changes | 
|---------------|------------|---------| 
| 101           | 2          | 0       | 
| 102           | 3          | 0       | 
| 103           | 0          | 3       | 
| 104           | 1          | 2       | 
| 105           | 0          | 1       |

#### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
select count(*) as delivered_counts
from customer_orders_temp c
inner join runner_orders_temp r
    on c.order_id = r.order_id
    and cancellation is null
where exclusions is not null and extras is not null;


```

| delivered_counts |
| --- |
| 1 |

---

#### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
select hour(order_time) as hour, count(*) as counts
from customer_orders_temp
group by hour
order by hour;


```

| hour | counts | 
|--------|--------| 
| 11     | 1      | 
| 13     | 3      | 
| 18     | 3      | 
| 19     | 1      | 
| 21     | 3      | 
| 23     | 3      |

#### 10. What was the volume of orders for each day of the week?

```sql
select dayname(order_time) as dayname, count(*) as ordered_counts
from customer_orders_temp
group by dayname(order_time)


```

| dayname | ordered_counts | 
|-----------|----------------| 
| Wednesday | 5              | 
| Thursday  | 3              | 
| Saturday  | 5              | 
| Friday    | 1              |

* You can check the name of the day directly using `dayname()` or get a numerical value (0~6) using `weekday()`.

### <a id="b"></a> B. Runner and Customer Experience

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
select floor(datediff(registration_date, '2021-01-01') / 7) + 1 as week_no, count(*)
from runners
group by floor(datediff(registration_date, '2021-01-01') / 7);


```

| week_no | count(*) |
| --- | --- |
| 1 | 2 |
| 2 | 1 |
| 3 | 1 |

* Note that the week starts on 2021-01-01. Checking the start date with `dayname()` shows it is a **Friday**. Therefore, a manual calculation is required instead of using `week()`.
* Calculate which week it is by dividing the difference between the registration date and the start date by 7 and rounding down.
* `week()` represents the year as the 0th to 53rd week based on Sunday (mode 0) or Monday (mode 1).

---

#### 2. Average time in minutes it took for each runner to arrive at Pizza Runner HQ to pickup the order?

```sql
with cte as (
select distinct c.order_id, runner_id,
timestampdiff(minute, order_time, pickup_time) as time
from customer_orders_temp c
inner join runner_orders_temp r
on c.order_id = r.order_id
)

select runner_id, avg(time)
from cte
group by runner_id;


```

| runner_id | avg(time) |
| --- | --- |
| 1 | 14.0000 |
| 2 | 19.6667 |
| 3 | 10.0000 |

* `distinct` is essential because there is only one pickup even if there are multiple pizzas in a single order.
* To calculate the minutes between order and pickup, you can use `timestampdiff()` or `minute(timediff())`.

---

#### 3. Relationship between the number of pizzas and preparation time?

```sql
with cte as (
select c.order_id, count(*) as pizza_no,
timestampdiff(minute, order_time, pickup_time) as prep_time
from customer_orders_temp c
inner join runner_orders_temp r on c.order_id = r.order_id
where cancellation is null
group by c.order_id, prep_time
)

select pizza_no, avg(prep_time)
from cte
group by pizza_no;


```

| pizza_no | avg(prep_time) |
| --- | --- |
| 1 | 12.0000 |
| 2 | 18.0000 |
| 3 | 29.0000 |

* The CTE from the previous question is used almost as is, but `distinct` is excluded because the **number of pizzas** is important.
* There is a positive correlation where preparation time increases as the number of pizzas increases.

---

#### 4. Average distance travelled for each customer?

```sql
with cte as(
select distinct customer_id, c.order_id, distance
from customer_orders_temp c
inner join runner_orders_temp r on c.order_id = r.order_id
where cancellation is null)

select customer_id, avg(distance)
from cte
group by customer_id;


```

| customer_id | avg(distance) |
| --- | --- |
| 101 | 20 |
| 102 | 18.4 |
| 103 | 23.4 |
| 104 | 10 |
| 105 | 25 |

* A CTE was created because distance information is required for each `order_id` and `customer_id`.
* If you query the average distance per customer directly without a CTE, cases where a customer ordered multiple times might be miscounted.

---

#### 5. Difference between the longest and shortest delivery times for all orders?

```sql
select max(duration) - min(duration) as diff
from runner_orders_temp;


```

| diff |
| --- |
| 30 |

---

#### 6. Average speed for each runner for each delivery?

```sql
with cte as(
select *, round(distance / (duration / 60), 2) as speed
from runner_orders_temp
where duration > 0
)

select runner_id, order_id, avg(speed)
from cte
group by runner_id, order_id


```

| runner_id | order_id | avg(speed) |
| --- | --- | --- |
| 1 | 1 | 37.5 |
| 1 | 2 | 44.44 |
| 1 | 3 | 40.2 |
| 2 | 4 | 35.1 |
| 3 | 5 | 40 |
| 2 | 7 | 60 |
| 2 | 8 | 93.6 |
| 1 | 10 | 60 |

* Speed can be calculated in km/hour. To do this, duration is used as a ratio based on 60 minutes.

---

#### 7. Successful delivery percentage for each runner?

```sql
select runner_id,
round((sum(case when cancellation is null then 1 else 0 end) / count(*)) * 100, 0) as success_percentage
from runner_orders_temp
group by runner_id;


```

| runner_id | success_percentage |
| --- | --- |
| 1 | 100 |
| 2 | 75 |
| 3 | 50 |

### <a id="c"></a> C. Ingredient Optimisation

#### 1. What are the standard ingredients for each pizza?

```sql
-- Use JSON_TABLE to convert comma-separated strings into rows
with cte as (
    select *
    from pizza_recipes
    inner join json_table(
        concat('[', pizza_recipes.toppings , ']'),
        '$[*]' columns(topping_id int path '$')
) as jt)

select pizza_id, group_concat(topping_name) as standard_toppings
from cte
inner join pizza_toppings pt
    on cte.topping_id = pt.topping_id
group by pizza_id


```

| pizza_id | standard_toppings |
| --- | --- |
| 1 | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 2 | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce |

* PostgreSQL allows splitting values by `,` easily using `REGEXP_SPLIT_TO_TABLE()`.
* However, since MySQL doesn't have that function, use `JSON_TABLE()` to separate multiple values.
* Used `group_concat()` aggregate function for readability.

#### 2. What was the most commonly added extra?

```sql
with cte as(
select *
from customer_orders_temp c
inner join json_table(
    concat('[', c.extras ,']'),
    '$[*]' columns(topping_id int path '$')
) as jt
)

select cte.topping_id, topping_name,  count(*)
from cte
inner join pizza_toppings pt
    on cte.topping_id = pt.topping_id
group by topping_id, topping_name
order by count(*) desc


```

| topping_id | topping_name | count(*) |
| --- | --- | --- |
| 1 | Bacon | 4 |
| 5 | Chicken | 1 |
| 4 | Cheese | 1 |

* Split multi-valued `extras` from `customer_orders` using `JSON_TABLE()`.

#### 3. What was the most common exclusion?

```sql
with cte as(
select *
from customer_orders_temp c
inner join json_table(
    concat('[', c.exclusions ,']'),
    '$[*]' columns(topping_id int path '$')
) as jt
)

select cte.topping_id, topping_name,  count(*)
from cte
inner join pizza_toppings pt
    on cte.topping_id = pt.topping_id
group by topping_id, topping_name
order by count(*) desc


```

| topping_id | topping_name | count(*) |
| --- | --- | --- |
| 4 | Cheese | 4 |
| 6 | Mushrooms | 1 |
| 2 | BBQ Sauce | 1 |

* Identical to question 2, but change the attribute to be split to `exclusions`.

---

#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:

`Meat Lovers`

`Meat Lovers - Exclude Beef`

`Meat Lovers - Extra Bacon`

`Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

* This question asks to represent standard, extra, and exclusion options as strings based on order requests.
* Using the logic from questions 2 and 3, create a CTE to separate `extras` and `exclusions`.

---

#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients

For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

---

#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

### <a id="d"></a> D. Pricing and Ratings

### <a id="e"></a> E. Bonus Questions

