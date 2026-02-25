### A. Pizza Metrics

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