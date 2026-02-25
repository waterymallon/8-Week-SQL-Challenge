# Case Study #1 - Danny's Diner
<img width="50%" alt="image" src="https://github.com/user-attachments/assets/d1d1e965-6d8b-431d-8aa7-338a217208ab" />

## 목차
- [Datasets & ERD](#datasets-&-erd)
- [Case Study Questions](#Case-Study-Questions)

***

## Datasets & ERD

Datasets를 생성하기 위해 [스키마 쿼리](https://github.com/waterymallon/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/case1_schema.sql)를 실행해주세요.<br>
해당 케이스는 3가지 테이블 sales, menu, members가 주어졌습니다.

<img width="699" height="440" alt="image" src="https://github.com/user-attachments/assets/7bbaf079-c193-4fa0-81a9-58602801831b" />

***

## Case Study Questions

### 1. What is the total amount each customer spent at the restaurant?
```sql
select customer_id, sum(price)
from sales s
inner join menu m on s.product_id = m.product_id
group by customer_id
```
<img width="212" height="91" alt="image" src="https://github.com/user-attachments/assets/4f6bd45d-3faa-4d4f-9401-cdf3391fdbe1" />

---

### 2. How many days has each customer visited the restaurant?
```sql
select customer_id, count(distinct order_date)
from sales
group by customer_id
```
<img width="303" height="111" alt="image" src="https://github.com/user-attachments/assets/c2733b62-19f6-4f75-a085-25f4bdf759d7" />

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
-- group by로 중복 제거
```
<img width="408" height="116" alt="image" src="https://github.com/user-attachments/assets/5af92e92-7f21-4bbb-a767-f8bfcc1bac6f" />

- 순위 함수() OVER (PARTITION BY 그룹기준 ORDER BY 정렬기준)
    - ROW_NUMBER() = 동점이여도 순서대로 (1, 2, 3, 4…)
    - RANK() = 동점자는 공동 순위, 다음 순위는 건너뜀 (1, 2, 2, 4…)
    - **DENSE_RANK()** = 동점자는 공동 순위, 다음 순위는 이어서 (1, 2, 2, 3…)
---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
select product_name, count(*)
from sales
inner join menu on sales.product_id = menu.product_id
group by product_name
order by count(*) desc
limit 1 -- 안해도 무관
```
<img width="216" height="95" alt="image" src="https://github.com/user-attachments/assets/b303b530-6e92-4311-948b-8552a82b56fa" />

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
<img width="396" height="148" alt="image" src="https://github.com/user-attachments/assets/b467b6db-1597-46da-a150-4516efad4b9c" />

- 실수 방지를 위해 count(*)도 함께 조회하는 습관을 길러두자
- group by customer_id, product_name으로 고객별로 인기있는 상품들의 count수를 조회했다.

---

### 6. Which item was purchased first by the customer after they became a member?
```sql
with cte as (
select sales.customer_id, order_date, product_id, join_date,
dense_rank() over (partition by customer_id order by order_date) as date_rank
from sales
inner join members on sales.customer_id = members.customer_id
where join_date < order_date
)

select customer_id, order_date, join_date, date_rank, product_name
from cte
inner join menu on cte.product_id = menu.product_id
where date_rank = 1
order by customer_id, date_rank
```
<img width="502" height="84" alt="image" src="https://github.com/user-attachments/assets/dace9ba6-3199-47c7-b8f1-dde8d419084b" />

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
<img width="323" height="77" alt="image" src="https://github.com/user-attachments/assets/43780ac3-1618-44af-a85b-a85a68b4a6be" />

- 멤버가 되기 직전의 첫 아이템이여도, where 조건으로 조인하기 전의 주문 날짜만 필터링 후 내림차순 랭크로 조회하면 된다<br>
이로 앞에 있는 다른 rank사용 문제들과 본질적으로 같은 문제다.

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
<img width="292" height="69" alt="image" src="https://github.com/user-attachments/assets/36f3c49a-c500-4aae-bb44-40517326b8ad" />

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
<img width="213" height="97" alt="image" src="https://github.com/user-attachments/assets/cbd42aa4-577d-4075-8a0b-58cdefb7fde6" />

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
and order_date >= join_date -- 포인트는 가입한 후에 집계한다
order by customer_id
)

select customer_id, sum(point)
from point_table
where order_date < '2021-02-01'
group by customer_id
```
<img width="210" height="75" alt="image" src="https://github.com/user-attachments/assets/b11242fe-b704-450f-9d8f-56877597c7f4" />


- 시간 연산은 + INTERVAL 1 YEAR/MONTH/DAY/HOUR/MINUTE/SECOND
- 포인트는 **가입 후** 계산되기에, 그 전에 날짜는 고려하지 않는다

### Join All The Things<br><br>Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)
```sql
select sales.customer_id, order_date, product_name, price, 
case when order_date < join_date then 'N'
when join_date is null then 'N'
else 'Y' end as member
from sales
inner join menu on sales.product_id = menu.product_id
left join members on sales.customer_id = members.customer_id
```
<img width="448" height="353" alt="image" src="https://github.com/user-attachments/assets/e55c673a-e23c-40b6-806a-547fa6bb8f00" />

-  주문 당시 고객 유무(Y/N) 확인이 주요 포인트다.

---

### Rank All The Things<br><br>Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
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
	partition by customer_id, member -- 파티션을 가입유무로도 나눠야 한다. 안그러면 'N'도 랭크 포함
	order by order_date
) end as 'rank'
from joined_table
```
<img width="495" height="352" alt="image" src="https://github.com/user-attachments/assets/a4ea232b-b8bb-4667-b095-0b09e0dacead" />

---
