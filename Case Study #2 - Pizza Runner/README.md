# Case Study #2 - Pizza Runner

## 목차

* [Datasets & ERD](#Datasets-&-ERD)
* [Data Cleaning](#Data-Cleaning)
* [A. Pizza Metrics](#A.-Pizza-Metrics)
* [B. Runner and Customer Experience](#B.-Runner-and-Customer-Experienc)
* [C. Ingredient Optimisation](C.-Ingredient-Optimisation)
* [D. Pricing and Ratings](D.-Pricing-and-Ratings)
* [E. Bonus Questions](#E.-Bonus-Questions)

---

## Datasets & ERD

Datasets를 생성하기 위해 [스키마 쿼리]()를 실행해주세요.
해당 케이스는 총 6개의 테이블이 주어졌습니다.



## Data Cleaning

테이블을 보면 컬럼마다 값이 `NULL`이 아니라 문자값 "null", 공백 "", 아니면 불필요한 문자값이 붙어있습니다. `CASE` 구문으로 적절히 변환해 준 후 임시 테이블(`TEMPORARY TABLE`)을 생성합니다.

### 1. customer_orders_temp

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

### 2. runner_orders_temp

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

---

## Case Study Questions

### A. Pizza Metrics

#### 1. How many pizzas were ordered?

```sql
select count(*)
from customer_orders;

```

---

#### 2. How many unique customer orders were made?

```sql
select count(distinct order_id)
from customer_orders;

```

- 유니크한 주문의 수입니다. 한 사람이 여러 피자를 주문해도 `order_id`는 하나로 집계됩니다.

---

#### 3. How many successful orders were delivered by each runner?

```sql
select runner_id, count(*)
from runner_orders_temp
where cancellation is null
group by runner_id;

```

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

---

#### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
select hour(order_time) as hour, count(*) as counts
from customer_orders_temp
group by hour
order by hour;

```

---

#### 10. What was the volume of orders for each day of the week?

```sql
select dayname(order_time) as weekday, count(*) as counts
from customer_orders_temp
group by dayname(order_time);

```

---

- `dayname()`을 통해 요일 명칭을 바로 확인하거나 `weekday()`로 수치화(0~6)할 수 있습니다.

---

### B. Runner and Customer Experience

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
select floor(datediff(registration_date, '2021-01-01') / 7) as week_no, count(*)
from runners
group by floor(datediff(registration_date, '2021-01-01') / 7);

```

- 2021-01-01은 금요일이므로 표준 `WEEK()` 함수 대신 기준일로부터의 일수 차이를 7로 나누어 계산합니다.

---

#### 2. Average time in minutes it took for each runner to arrive at Pizza Runner HQ to pickup the order?

```sql
with cte as (
select distinct c.order_id, runner_id,
timestampdiff(minute, order_time, pickup_time) as order_pickup
from customer_orders_temp c
inner join runner_orders_temp r
on c.order_id = r.order_id
)

select runner_id, avg(order_pickup)
from cte
group by runner_id;

```

- 한 주문에 피자가 여러 개여도 픽업은 한 번이므로 `distinct order_id`가 필수입니다.

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

---

#### 5. Difference between the longest and shortest delivery times for all orders?

```sql
select max(duration) - min(duration) as diff
from runner_orders_temp;

```

---

#### 6. Average speed for each runner for each delivery?

```sql
with cte as(
select runner_id, c.order_id, distance, duration,
round(distance / (duration/60), 2) as km_per_h
from customer_orders_temp c
inner join runner_orders_temp r on c.order_id = r.order_id
where distance > 0
group by order_id, runner_id, distance, duration
)

select runner_id, min(km_per_h), max(km_per_h), avg(km_per_h)
from cte
group by runner_id;

```

---

#### 7. Successful delivery percentage for each runner?

```sql
select runner_id,
round((sum(case when cancellation is null then 1 else 0 end) / count(*)) * 100, 0) as success_percentage
from runner_orders_temp
group by runner_id;

```

---

### C. Ingredient Optimisation

#### 1. What are the standard ingredients for each pizza?

```sql
-- JSON_TABLE을 활용해 쉼표로 구분된 문자열을 행으로 변환
with cte as (
    select pizza_id, jt.topping_id
    from pizza_recipes pr
    cross join json_table(
        concat('[', pr.toppings, ']'),
        '$[*]' columns(topping_id int path '$')
    ) as jt
)

select pizza_id, group_concat(topping_name) as standard_ingredients
from cte
inner join pizza_toppings pt on cte.topping_id = pt.topping_id
group by pizza_id;

```

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

### D. Pricing and Ratings


### E. Bonus Questions

