# Case Study #1 - Danny's Diner

<img width="50%" alt="image" src="https://github.com/user-attachments/assets/d1d1e965-6d8b-431d-8aa7-338a217208ab" />

## Table of Contents

* [Datasets & ERD](#datasets-%26-erd)
* [Case Study Questions](#Case-Study-Questions)

---

## Datasets & ERD

Please run the [Schema Query](https://www.google.com/search?q=https://github.com/waterymallon/8-Week-SQL-Challenge/blob/main/Case%2520Study%2520%25231%2520-%2520Danny%27s%2520Diner/case1_schema.sql) to create the datasets.


This case study provides three tables: `sales`, `menu`, and `members`.

<details>
<summary> sales </summary>

| customer_id | order_date | product_id |
| --- | --- | --- |
| A | 2021-01-01 | 1 |
| A | 2021-01-01 | 2 |
| A | 2021-01-07 | 2 |
| A | 2021-01-10 | 3 |
| A | 2021-01-11 | 3 |
| A | 2021-01-11 | 3 |
| B | 2021-01-01 | 2 |
| B | 2021-01-02 | 2 |
| B | 2021-01-04 | 1 |
| B | 2021-01-11 | 1 |
| B | 2021-01-16 | 3 |
| B | 2021-02-01 | 3 |
| C | 2021-01-01 | 3 |
| C | 2021-01-01 | 3 |
| C | 2021-01-07 | 3 |

</details>

<details>
<summary> menu </summary>

| product_id | product_name | price |
| --- | --- | --- |
| 1 | sushi | 10 |
| 2 | curry | 15 |
| 3 | ramen | 12 |

</details>

<details>
<summary> members </summary>

| customer_id | join_date |
| --- | --- |
| A | 2021-01-07 |
| B | 2021-01-09 |

</details>

<img width="699" height="440" alt="image" src="https://github.com/user-attachments/assets/7bbaf079-c193-4fa0-81a9-58602801831b" />

---

## Case Study Questions

### 1. What is the total amount each customer spent at the restaurant?

```sql
select customer_id, sum(price)
from sales s
inner join menu m on s.product_id = m.product_id
group by customer_id

```

| customer_id | sum(price) |
| --- | --- |
| A | 76 |
| B | 74 |
| C | 36 |

---

### 2. How many days has each customer visited the restaurant?

```sql
select customer_id, count(distinct order_date)
from sales
group by customer_id

```

| customer_id | count(distinct order_date) |
| --- | --- |
| A | 4 |
| B | 6 |
| C | 2 |

---

### 3. What was the first item from the menu purchased by each customer?

```sql
with cte as (select customer_id, order_date, sales.product_id, product_name,
dense_rank() over (partition by customer_id order by order_date) as rank_date
from sales
inner join menu on sales.product_id = menu.product_id)

select customer_id, order_date, product_name, rank_date
from cte
where rank_date = 1
group by customer_id, order_date, product_name, rank_date
-- Remove duplicates using GROUP BY

```

| customer_id | order_date | product_name | rank_date |
| --- | --- | --- | --- |
| A | 2021-01-01 | sushi | 1 |
| A | 2021-01-01 | curry | 1 |
| B | 2021-01-01 | curry | 1 |
| C | 2021-01-01 | ramen | 1 |

* Ranking Functions: `FUNCTION() OVER (PARTITION BY [grouping_criteria] ORDER BY [sorting_criteria])`
    * `ROW_NUMBER()` = Assigns a unique sequential number (1, 2, 3, 4…)
    * `RANK()` = Assigns the same rank to ties, skipping the next rank (1, 2, 2, 4…)
    * **`DENSE_RANK()`** = Assigns the same rank to ties, without skipping the next rank (1, 2, 2, 3…)


---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
select product_name, count(*)
from sales
inner join menu on sales.product_id = menu.product_id
group by product_name
order by count(*) desc
limit 1 -- Optional

```

| product_name | count(*) |
| --- | --- |
| ramen | 8 |

---

### 5. Which item was the most popular for each customer?

```sql
with cte as(
select customer_id, product_name, count(*),
dense_rank() over (partition by customer_id order by count(*) desc) as count_rank
from sales
inner join menu on sales.product_id = menu.product_id
group by customer_id, product_name
)

select *
from cte
where count_rank = 1

```

| customer_id | product_name | count(*) | count_rank |
| --- | --- | --- | --- |
| A | ramen | 3 | 1 |
| B | curry | 2 | 1 |
| B | sushi | 2 | 1 |
| B | ramen | 2 | 1 |
| C | ramen | 3 | 1 |

* It is good practice to include `COUNT(*)` in the results to avoid mistakes and verify counts.
* Grouped by `customer_id` and `product_name` to find the frequency of each item purchased per customer.

---

### 6. Which item was purchased first by the customer after they became a member?

```sql
with cte as (
select sales.customer_id, order_date, product_id, join_date,
dense_rank() over (partition by customer_id order by order_date) as date_rank
from sales
inner join members on sales.customer_id = members.customer_id
where join_date <= order_date
)

select customer_id, order_date, join_date, date_rank, product_name
from cte
inner join menu on cte.product_id = menu.product_id
where date_rank = 1
order by customer_id, date_rank

```

| customer_id | order_date | join_date | date_rank | product_name |
| --- | --- | --- | --- | --- |
| A | 2021-01-07 | 2021-01-07 | 1 | curry |
| B | 2021-01-11 | 2021-01-09 | 1 | sushi |

---

### 7. Which item was purchased just before the customer became a member?

```sql
with cte as (
select sales.customer_id, order_date, product_id, join_date,
row_number() over (partition by customer_id order by order_date desc) as date_rank
from sales
inner join members on sales.customer_id = members.customer_id
where order_date < join_date
)

select customer_id, product_name, date_rank
from cte
inner join menu on cte.product_id = menu.product_id
where date_rank = 1

```

| customer_id | product_name | date_rank |
| --- | --- | --- |
| A | sushi | 1 |
| B | sushi | 1 |

* To find the very last item purchased before membership, filter the order dates before joining and then rank them in descending order. This logic is essentially the same as other ranking problems.

---

### 8. What is the total items and amount spent for each member before they became a member?

```sql
select sales.customer_id, count(*), sum(price)
from sales
inner join members on sales.customer_id = members.customer_id
and order_date < join_date
inner join menu on sales.product_id = menu.product_id
group by customer_id

```

| customer_id | count(*) | sum(price) |
| --- | --- | --- |
| B | 3 | 40 |
| A | 2 | 25 |

---

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
with cte as(
select customer_id, product_name,
case when product_name = 'sushi' then price*20
else price*10 end as point
from sales
inner join menu on sales.product_id = menu.product_id
)

select customer_id, sum(point)
from cte
group by customer_id

```

| customer_id | sum(point) |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |

---

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
with point_table as(
select sales.customer_id, order_date, product_name, join_date, price, 
case when (join_date <= order_date and order_date <= join_date + interval 6 day) then price*20
when product_name = 'sushi' then price*20
else price*10 end as point
from sales
inner join menu on sales.product_id = menu.product_id
inner join members on sales.customer_id = members.customer_id
and order_date >= join_date -- Points are aggregated starting from the membership join date
order by customer_id
)

select customer_id, sum(point)
from point_table
where order_date < '2021-02-01'
group by customer_id

```

| customer_id | sum(point) |
| --- | --- |
| B | 320 |
| A | 1020 |

* Date arithmetic can be performed using `+ INTERVAL 1 YEAR/MONTH/DAY/HOUR/MINUTE/SECOND`.
* Since points are calculated **after joining**, dates prior to membership are excluded.

---

### Join All The Things

Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

```sql
select sales.customer_id, order_date, product_name, price, 
case when order_date < join_date then 'N'
when join_date is null then 'N'
else 'Y' end as member
from sales
inner join menu on sales.product_id = menu.product_id
left join members on sales.customer_id = members.customer_id

```

| customer_id | order_date | product_name | price | member |
| --- | --- | --- | --- | --- |
| A | 2021-01-01 | sushi | 10 | N |
| A | 2021-01-01 | curry | 15 | N |
| A | 2021-01-07 | curry | 15 | Y |
| A | 2021-01-10 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| B | 2021-01-01 | curry | 15 | N |
| B | 2021-01-02 | curry | 15 | N |
| B | 2021-01-04 | sushi | 10 | N |
| B | 2021-01-11 | sushi | 10 | Y |
| B | 2021-01-16 | ramen | 12 | Y |
| B | 2021-02-01 | ramen | 12 | Y |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-07 | ramen | 12 | N |

* The key objective is to determine the membership status (Y/N) at the specific time of each order.

---

### Rank All The Things

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
with joined_table as (
select sales.customer_id, order_date, product_name, price, 
case when order_date < join_date then 'N'
when join_date is null then 'N'
else 'Y' end as member
from sales
inner join menu on sales.product_id = menu.product_id
left join members on sales.customer_id = members.customer_id
)

select *,
case when member = 'N' then NULL
else dense_rank() over (
    partition by customer_id, member -- Partition by membership status as well to exclude 'N' from ranking
    order by order_date
) end as 'rank'
from joined_table

```

| customer_id | order_date | product_name | price | member | rank |
| --- | --- | --- | --- | --- | --- |
| A | 2021-01-01 | sushi | 10 | N | NULL |
| A | 2021-01-01 | curry | 15 | N | NULL |
| A | 2021-01-07 | curry | 15 | Y | 1 |
| A | 2021-01-10 | ramen | 12 | Y | 2 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| A | 2021-01-11 | ramen | 12 | Y | 3 |
| B | 2021-01-01 | curry | 15 | N | NULL |
| B | 2021-01-02 | curry | 15 | N | NULL |
| B | 2021-01-04 | sushi | 10 | N | NULL |
| B | 2021-01-11 | sushi | 10 | Y | 1 |
| B | 2021-01-16 | ramen | 12 | Y | 2 |
| B | 2021-02-01 | ramen | 12 | Y | 3 |
| C | 2021-01-01 | ramen | 12 | N | NULL |
| C | 2021-01-01 | ramen | 12 | N | NULL |
| C | 2021-01-07 | ramen | 12 | N | NULL |