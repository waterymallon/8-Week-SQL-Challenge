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

```sql
with cte as(
select c.*, pizza_name, 
row_number() over (order by order_id) as row_no
from customer_orders_temp c
inner join pizza_names p
	on c.pizza_id = p.pizza_id
),

cte_exc as(
select row_no, group_concat(topping_name) as exclusions_name
from cte c
inner join json_table(
concat('[', c.exclusions ,']'),
'$[*]' columns(topping_id int path '$')
) as jt
inner join pizza_toppings pt
	on jt.topping_id = pt.topping_id
group by row_no
),

cte_ext as (
select row_no, group_concat(topping_name) as extras_name
from cte c
inner join json_table(
concat('[', c.extras ,']'),
'$[*]' columns(topping_id int path '$')
) as jt
inner join pizza_toppings pt
	on jt.topping_id = pt.topping_id
group by row_no
)

select cte.row_no, pizza_name, exclusions_name, extras_name,
concat(
	pizza_name,
    if(exclusions_name is not null, concat(" - Exclude ", exclusions_name), ""),
    if(extras_name is not null, concat(" - Extra ", extras_name), "")
	) as string_order
from cte
left join cte_exc
	on cte.row_no = cte_exc.row_no
left join cte_ext
	on cte.row_no = cte_ext.row_no
```

| row_no | exclusions_name     | extras_name   | pizza_name | string_order                                                  |
|--------|---------------------|---------------|------------|---------------------------------------------------------------|
| 1      |                     |               | Meatlovers | Meatlovers                                                    |
| 2      |                     |               | Meatlovers | Meatlovers                                                    |
| 3      |                     |               | Meatlovers | Meatlovers                                                    |
| 4      |                     |               | Vegetarian | Vegetarian                                                    |
| 5      | Cheese              |               | Meatlovers | Meatlovers - Exclude Cheese                                   |
| 6      | Cheese              |               | Meatlovers | Meatlovers - Exclude Cheese                                   |
| 7      | Cheese              |               | Vegetarian | Vegetarian - Exclude Cheese                                   |
| 8      |                     | Bacon         | Meatlovers | Meatlovers - Extra Bacon                                      |
| 9      |                     |               | Vegetarian | Vegetarian                                                    |
| 10     |                     | Bacon         | Vegetarian | Vegetarian - Extra Bacon                                      |
| 11     |                     |               | Meatlovers | Meatlovers                                                    |
| 12     | Cheese              | Chicken,Bacon | Meatlovers | Meatlovers - Exclude Cheese - Extra Chicken,Bacon             |
| 13     |                     |               | Meatlovers | Meatlovers                                                    |
| 14     | Mushrooms,BBQ Sauce | Cheese,Bacon  | Meatlovers | Meatlovers - Exclude Mushrooms,BBQ Sauce - Extra Cheese,Bacon |


* This question asks to represent standard, extra, and exclusion options as strings based on order requests.
- First CTE is used as base table after adding row number and pizza name. Row number is crucial to distinguish duplicate record in the table.
    - Row number is also needed for `group_concat` as grouping key
* Using the logic from questions 2 and 3, create a CTE to separate `extras` and `exclusions`.
* Left join base_table with two CTEs. Used `concat` and `IF` to format strings per record

---

#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients

For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

---

#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?