# Case Study #2 - Pizza Runner
<img width="50%" alt="image" src="https://github.com/user-attachments/assets/c409a6a4-a453-431d-9d71-56ede291a64d" />


## 목차

* [Datasets & ERD](#datasets)
* [Data Cleaning](#cleaning)
* [A. Pizza Metrics](#a)
* [B. Runner and Customer Experience](#b)
* [C. Ingredient Optimisation](#c)
* [D. Pricing and Ratings](#d)
* [E. Bonus Questions](#e)

---

## <a id="datasets"></a> Datasets & ERD

Datasets를 생성하기 위해 [스키마 쿼리](https://github.com/waterymallon/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/case2_schema.sql)를 실행해주세요.<br>
해당 케이스는 총 6개의 테이블이 주어졌습니다.

<details>
<summary> customer_orders </summary>

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|------------|-------------|----------|------------|--------|---------------------|
| 1          | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2          | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3          | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3          | 102         | 2        |            | NULL   | 2020-01-02 23:51:23 |
| 4          | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4          | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4          | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5          | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6          | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7          | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8          | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9          | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10         | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10         | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

</details>

<details>
<summary> pizza_names </summary>

| pizza_id | pizza_name |
|------------|------------|
| 1          | Meatlovers |
| 2          | Vegetarian |

</details>
<details>
<summary> pizza_recipes </summary>

| pizza_id | toppings                |
|------------|-------------------------|
| 1          | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2          | 4, 6, 7, 9, 11, 12      |

</details>

<details>
<summary> pizza_toppings </summary>

| topping_id | topping_name |
|--------------|--------------|
| 1            | Bacon        |
| 2            | BBQ Sauce    |
| 3            | Beef         |
| 4            | Cheese       |
| 5            | Chicken      |
| 6            | Mushrooms    |
| 7            | Onions       |
| 8            | Pepperoni    |
| 9            | Peppers      |
| 10           | Salami       |
| 11           | Tomatoes     |
| 12           | Tomato Sauce |

</details>

<details>
<summary> runner_orders </summary>

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
|------------|-----------|---------------------|----------|------------|-------------------------|
| 1          | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2          | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3          | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    | NULL                    |
| 4          | 2         | 2020-01-04 13:53:03 | 23.4     | 40         | NULL                    |
| 5          | 3         | 2020-01-08 21:10:57 | 10       | 15         | NULL                    |
| 6          | 3         | null                | null     | null       | Restaurant Cancellation |
| 7          | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8          | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9          | 2         | null                | null     | null       | Customer Cancellation   |
| 10         | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |
</details>

<details>
<summary> runners </summary>

| runner_id | registration_date |
|-------------|-------------------|
| 1           | 2021-01-01        |
| 2           | 2021-01-03        |
| 3           | 2021-01-08        |
| 4           | 2021-01-15        |

</details>


<img width="690" height="566" alt="image" src="https://github.com/user-attachments/assets/d42c9a48-5082-45be-9461-d7d00b12fba3" />


## <a id="cleaning"></a> Data Cleaning

테이블을 보면 컬럼마다 값이 `NULL`이 아니라 문자값 "null", 공백 "", 아니면 불필요한 문자값이 붙어있습니다. `CASE` 구문으로 적절히 변환해 준 후 임시 테이블(`TEMPORARY TABLE`)을 생성합니다.

### 1. customer_orders

`exclusions`, `extras` 컬럼의 문자값 "null"이나 공백 ""을 `NULL`로 변환합니다.

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
| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
|------------|-------------|----------|------------|--------|---------------------|
| 1          | 101         | 1        | NULL       | NULL   | 2020-01-01 18:05:02 |
| 2          | 101         | 1        | NULL       | NULL   | 2020-01-01 19:00:52 |
| 3          | 102         | 1        | NULL       | NULL   | 2020-01-02 23:51:23 |
| 3          | 102         | 2        | NULL       | NULL   | 2020-01-02 23:51:23 |
| 4          | 103         | 1        | 4          | NULL   | 2020-01-04 13:23:46 |
| 4          | 103         | 1        | 4          | NULL   | 2020-01-04 13:23:46 |
| 4          | 103         | 2        | 4          | NULL   | 2020-01-04 13:23:46 |
| 5          | 104         | 1        | NULL       | 1      | 2020-01-08 21:00:29 |
| 6          | 101         | 2        | NULL       | NULL   | 2020-01-08 21:03:13 |
| 7          | 105         | 2        | NULL       | 1      | 2020-01-08 21:20:29 |
| 8          | 102         | 1        | NULL       | NULL   | 2020-01-09 23:54:33 |
| 9          | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10         | 104         | 1        | NULL       | NULL   | 2020-01-11 18:34:49 |
| 10         | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

### 2. runner_orders

`pickup_time`, `distance`, `duration`, `cancellation`의 노이즈 데이터를 처리합니다.

* **duration**: `regexp_replace`를 사용하여 'mins', 'minutes', 'minute' 제거 및 공백 제거
* **distance**: 'km' 문자 제거

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
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
|------------|-----------|---------------------|----------|----------|-------------------------|
| 1          | 1         | 2020-01-01 18:15:34 | 20       | 32       | NULL                    |
| 2          | 1         | 2020-01-01 19:10:54 | 20       | 27       | NULL                    |
| 3          | 1         | 2020-01-03 00:12:37 | 13.4     | 20       | NULL                    |
| 4          | 2         | 2020-01-04 13:53:03 | 23.4     | 40       | NULL                    |
| 5          | 3         | 2020-01-08 21:10:57 | 10       | 15       | NULL                    |
| 6          | 3         | NULL                | NULL     | NULL     | Restaurant Cancellation |
| 7          | 2         | 2020-01-08 21:30:45 | 25       | 25       | NULL                    |
| 8          | 2         | 2020-01-10 00:15:02 | 23.4     | 15       | NULL                    |
| 9          | 2         | NULL                | NULL     | NULL     | Customer Cancellation   |
| 10         | 1         | 2020-01-11 18:50:20 | 10       | 10       | NULL                    |

---

## Case Study Questions

### <a id="a"></a> A. Pizza Metrics

#### 1. How many pizzas were ordered?

```sql
select count(*)
from customer_orders;

```
| count(*) |
|------------|
| 14       |

---

#### 2. How many unique customer orders were made?

```sql
select count(distinct order_id)
from customer_orders;

```
| count(distinct order_id) |
|----------------------------|
| 10                         |


- 유니크한 주문의 수입니다. 한 사람이 여러 피자를 주문해도 `order_id`는 하나로 집계됩니다.

---

#### 3. How many successful orders were delivered by each runner?

```sql
select runner_id, count(*)
from runner_orders_temp
where cancellation is null
group by runner_id;

```
| runner_id | count(*) |
|-------------|----------|
| 1           | 4        |
| 2           | 3        |
| 3           | 1        |

- each runner로 `runner_id`별로 group by를 해줍니다.
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
|--------------|----------|
| Meatlovers   | 9        |
| Vegetarian   | 3        |
- each pizza로 `pizza_name`별로 group by를 해줍니다.
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
|---------------|------------|--------------|
| 101           | Meatlovers | 2            |
| 101           | Vegetarian | 1            |
| 102           | Meatlovers | 2            |
| 102           | Vegetarian | 1            |
| 103           | Meatlovers | 3            |
| 103           | Vegetarian | 1            |
| 104           | Meatlovers | 3            |
| 105           | Vegetarian | 1            |
- "ordered" 기준이므로 배달 취소 여부는 고려하지 않습니다.

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
| 3         |
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
---

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
|--------------------|
| 1                |

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
---

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
---

- `dayname()`을 통해 요일 명칭을 바로 확인하거나 `weekday()`로 수치화(0~6)할 수 있습니다.

---

### <a id="b"></a> B. Runner and Customer Experience

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
select floor(datediff(registration_date, '2021-01-01') / 7) + 1 as week_no, count(*)
from runners
group by floor(datediff(registration_date, '2021-01-01') / 7);

```

| week_no | count(*) |
|-----------|----------|
| 1         | 2        |
| 2         | 1        |
| 3         | 1        |

- 2021-01-01에 주가 시작되는 점에 유의해야 합니다. `dayname()`로 시작일을 확인하면 **금요일**임을 알 수 있습니다. 이로 `week()` 말고 직접 계산이 필요합니다.
- 시작일과 등록된 날짜의 차이를 7로 나눈 후 내림차하여 몇 번째 주인지 계산해 줍니다.
- `week()`는 일요일(mode 0) 혹은 월요일(mode 1)을 기준으로 1년을 0~53번째 주로 표현합니다.

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
|-------------|-----------|
| 1           | 14.0000   |
| 2           | 19.6667   |
| 3           | 10.0000   |

- 한 주문에 피자가 여러 개여도 픽업은 한 번이므로 `distinct`가 필수입니다.
- 주문에서 픽업까지 몇 분걸렸는지 구할 때는 `timestampdiff()` 혹은 `minute(timediff())`로 가능합니다.

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
|------------|----------------|
| 1          | 12.0000        |
| 2          | 18.0000        |
| 3          | 29.0000        |

- 직전 문제의 cte를 거의 그대로 쓰나, **number of pizzas**가 중요하여 `distinct`를 제외해 줍니다.
- 피자 개수가 많을수록 준비 시간이 늘어나는 양의 상관관계를 보입니다.

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
|---------------|---------------|
| 101           | 20            |
| 102           | 18.4          |
| 103           | 23.4          |
| 104           | 10            |
| 105           | 25            |


- order_id & customer_id별로 distance정보가 필요해 cte를 작성했습니다
- 만약 cte없이 바로 고객별로 평균 거리를 조회하면, 고객이 여러 번 주문한 경우가 빠질 수 있습니다.
---

#### 5. Difference between the longest and shortest delivery times for all orders?

```sql
select max(duration) - min(duration) as diff
from runner_orders_temp;

```

| diff |
|--------|
| 30   |

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

- 속도는 시속으로 km / hour로 계산하면 됩니다. 이를 위해 duration은 60분 기준의 비율을 사용합니다.
---

#### 7. Successful delivery percentage for each runner?

```sql
select runner_id,
round((sum(case when cancellation is null then 1 else 0 end) / count(*)) * 100, 0) as success_percentage
from runner_orders_temp
group by runner_id;

```

| runner_id | success_percentage |
|-------------|--------------------|
| 1           | 100                |
| 2           | 75                 |
| 3           | 50                 |

---

### <a id="c"></a> C. Ingredient Optimisation

#### 1. What are the standard ingredients for each pizza?

```sql
-- JSON_TABLE을 활용해 쉼표로 구분된 문자열을 행으로 변환
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
| pizza_id | standard_toppings                                              |
|------------|----------------------------------------------------------------|
| 1          | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| 2          | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |

- PostgreSQL은 `REGEXP_SPLIT_TO_TABLE()`을 사용해 편리하게 `,`별로 값을 나눌 수 있습니다.

    - 단 MySQL은 그런 함수는 없기에 `JSON_TABLE()`을 사용해 다중 값을 분리합시다.

- 가독성을 위해 집계함수 `group_concat()`를 사용했습니다.

#### 2. What was the most commonly added extra?

```sql
with cte as (
    select jt.extra_id
    from customer_orders_temp c
    cross join json_table(
        concat('[', c.extras, ']'),
        '$[*]' columns(extra_id int path '$')
    ) as jt
)

select extra_id, topping_name, count(*) as count
from cte
inner join pizza_toppings pt on cte.extra_id = pt.topping_id
group by extra_id, topping_name
order by count desc;

```

#### 3. What was the most common exclusion?

```sql
with cte as (
    select jt.topping_id
    from customer_orders_temp c
    cross join json_table(
        concat('[', c.exclusions, ']'),
        '$[*]' columns(topping_id int path '$')
    ) as jt
)

select topping_id, topping_name, count(*) as count
from cte
inner join pizza_toppings pt on cte.topping_id = pt.topping_id
group by topping_id, topping_name
order by count desc;

```

---

### <a id="d"></a> D. Pricing and Ratings


### <a id="e"></a> E. Bonus Questions

