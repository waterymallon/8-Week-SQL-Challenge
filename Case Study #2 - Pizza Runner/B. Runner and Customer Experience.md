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

| runner_id | order_id | avg(speed) |
|-------------|----------|------------|
| 1           | 1        | 37.5       |
| 1           | 2        | 44.44      |
| 1           | 3        | 40.2       |
| 2           | 4        | 35.1       |
| 3           | 5        | 40         |
| 2           | 7        | 60         |
| 2           | 8        | 93.6       |
| 1           | 10       | 60         |

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


