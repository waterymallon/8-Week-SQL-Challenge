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
|--------------|--------------|----------|
| 1            | Bacon        | 4        |
| 5            | Chicken      | 1        |
| 4            | Cheese       | 1        |

- 다중 값을 가진 customer_orders의 extras를 `JSON_TABLE()`을 사용해 분리해 줍니다.

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
|--------------|--------------|----------|
| 4            | Cheese       | 4        |
| 6            | Mushrooms    | 1        |
| 2            | BBQ Sauce    | 1        |

- 2번 문제와 완전히 동일하나, 분리할 속성을 exclusions로 바꿔줍니다.

---

#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
`Meat Lovers`<br>
`Meat Lovers - Exclude Beef`<br>
`Meat Lovers - Extra Bacon`<br>
`Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`<br>

- 해당 문제는 주문 요청에 따라 기본, 추가, 제외 옵션을 문자열로 표현하라고 요구합니다.
- 앞에 2, 3문제의 응용으로 cte로 extras와 exclusions를 분리한 cte를 만듭니다.

---


#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

---


#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
