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