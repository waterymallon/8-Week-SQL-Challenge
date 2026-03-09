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