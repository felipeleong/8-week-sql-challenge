*create a view with pizza_id, pizza_name, toppings name and id*

*this will help us to create a relationship with the data model.*
````sql
CREATE OR REPLACE VIEW pizza_runner.pizza_recepies_view
 AS
 WITH base AS (
         SELECT pn.pizza_name,
            	pn.pizza_id,
            	unnest(string_to_array(pr.toppings, ','::text))::integer AS topping_id
           FROM pizza_runner.pizza_recipes pr
           JOIN pizza_runner.pizza_names pn ON pr.pizza_id = pn.pizza_id
        ), 
	toppings AS (
         SELECT pizza_toppings.topping_id, pizza_toppings.topping_name
         FROM pizza_runner.pizza_toppings
        )
 SELECT b.pizza_id,b.pizza_name,t.topping_id,t.topping_name
 FROM base b
 JOIN pizza_runner.pizza_toppings t ON b.topping_id = t.topping_id
 ORDER BY b.pizza_id;

ALTER TABLE pizza_runner.pizza_recepies_view
    OWNER TO postgres;
````
### VIEW: 
````SQL
SELECT * FROM pizza_runner.pizza_recepies_view
````
Resultset: 
| pizza_id | pizza_name  | topping_id | topping_name   |
|---------|-------------|-----------|----------------|
| 1       | Meatlovers  | 2         | BBQ Sauce      |
| 1       | Meatlovers  | 8         | Pepperoni      |
| 1       | Meatlovers  | 4         | Cheese         |
| 1       | Meatlovers  | 10        | Salami         |
| 1       | Meatlovers  | 5         | Chicken        |
| 1       | Meatlovers  | 1         | Bacon          |
| 1       | Meatlovers  | 6         | Mushrooms      |
| 1       | Meatlovers  | 3         | Beef           |
| 2       | Vegetarian | 12        | Tomato Sauce   |
| 2       | Vegetarian | 4         | Cheese         |
| 2       | Vegetarian | 6         | Mushrooms      |
| 2       | Vegetarian | 7         | Onions         |
| 2       | Vegetarian | 9         | Peppers        |
| 2       | Vegetarian | 11        | Tomatoes       |

### 1. What are the standard ingredients for each pizza?

*With the view:*
````sql
SELECT 
	pizza_name,
	STRING_AGG(topping_name,', ') AS topping_name
FROM pizza_runner.pizza_recepies_view
GROUP BY 1;
````


Answer: 
| pizza_name |             topping_name              |
|------------|--------------------------------------|
| Meatlovers  | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami|
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes ,Tomato Sauce|

### 2.What was the most commonly added extra?

````sql
WITH customer_extra AS (
	SELECT UNNEST(string_to_array(extras,','))::integer AS extra_topping_id
	FROM pizza_runner.customer_orders co
)
,toppings AS (
	SELECT topping_id,topping_name
	FROM pizza_runner.pizza_toppings
)
SELECT
	ce.extra_topping_id,
	t.topping_name, 
	COUNT(*) AS "count of extra topping"
FROM customer_extra ce
JOIN toppings t ON t.topping_id = ce.extra_topping_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;
````
### Answer:
| extra_topping_id | topping_name | count of extra topping |
|-----------------|--------------|-----------------------|
| 1               | Bacon        | 4                     |

*The most commonly added extra is Bacon

### 3.What was the most common exclusion?
````SQL
WITH customer_exclusions AS (
	SELECT 
		*,
		string_to_table(exclusions,',')::INT AS exclusions_topping_id
	FROM pizza_runner.customer_orders
),
toppings AS (
	SELECT topping_id,topping_name
	FROM pizza_runner.pizza_toppings
)
SELECT 
	ce.exclusions_topping_id,
	t.topping_name,
	COUNT(*) AS "count of exclusions topping"
FROM customer_exclusions ce
JOIN toppings t ON t.topping_id = ce.exclusions_topping_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;
````
### Answer:
| exclusions_topping_id | topping_name | count of exclusions topping |
|----------------------|--------------|-----------------------------|
| 4                    | Cheese       | 4                           |

*The most commonly exclusion is Cheese*

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
*Meat Lovers
Meat Lovers - Exclude Beef
Meat Lovers - Extra Bacon
Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers*

````sql
CREATE OR REPLACE VIEW pizza_runner.pizza_all_toppings_view AS
SELECT 
	pizza_id,
	pizza_name,
	STRING_AGG(topping_name,', ') AS toppings
FROM pizza_runner.pizza_recepies_view
GROUP BY 1,2;
--------
WITH CTE_base AS (
	SELECT 
		co.order_id,co.pizza_id,co.exclusions,co.extras, prv.pizza_name
	FROM pizza_runner.customer_orders co
	JOIN pizza_runner.pizza_all_toppings_view prv USING(pizza_id)
),
cte_orders_without_e AS (
	SELECT cte.order_id,cte.pizza_id,cte.pizza_name,
		pizza_name AS order_item
	FROM CTE_BASE cte
	WHERE exclusions IS NULL AND extras IS NULL
),
cte_exclusions AS (
	SELECT cteb.order_id,cteb.pizza_id,cteb.pizza_name,
		CONCAT(cteb.pizza_name,' - Exclude ', pv.topping_name) AS order_item
	FROM cte_base cteb 
	JOIN pizza_runner.pizza_recepies_view pv ON cteb.pizza_id = pv.pizza_id AND pv.topping_id = cteb.exclusions::int
	WHERE exclusions IS NOT NULL AND extras IS NULL
),
cte_extras AS (
	SELECT 
		extras_t.order_id,extras_t.pizza_id,extras_t.pizza_name,
		CONCAT(extras_t.pizza_name,' - Extra ', pv.topping_name) AS order_item
	FROM (
		SELECT cteb.*
		FROM cte_base cteb 
		WHERE exclusions IS NULL AND extras IS NOT NULL) extras_t
	JOIN pizza_runner.pizza_recepies_view pv ON pv.topping_id = extras_t.extras::int
),
cte_exlusion_extras AS (
	SELECT 
		cteb.order_id,
		cteb.pizza_id,
		cteb.pizza_name,
		STRING_TO_TABLE(cteb.exclusions,', ')::INT AS exclusions,
		STRING_TO_TABLE(cteb.extras,', ')::INT AS extras
	FROM CTE_base cteb 
	WHERE exclusions IS NOT NULL AND extras IS NOT NULL
),
cte_exclusions_extras_only_exclusions AS (
	SELECT 
		ctee.order_id,ctee.pizza_id,ctee.pizza_name,
		STRING_AGG(pv.topping_name,', ') AS full_exclusions_toppings
	FROM cte_exlusion_extras ctee 
	JOIN pizza_runner.pizza_recepies_view pv ON ctee.pizza_id = pv.pizza_id AND ctee.exclusions = pv.topping_id
	GROUP BY ctee.order_id,ctee.pizza_id,ctee.pizza_name
),
cte_exclusions_extras_only_extras AS (
	SELECT 
		ctee.order_id,ctee.pizza_id,ctee.pizza_name,
		STRING_AGG(pv.topping_name,', ') AS full_extras_toppings
	FROM cte_exlusion_extras ctee 
	JOIN pizza_runner.pizza_recepies_view pv ON ctee.pizza_id = pv.pizza_id AND ctee.extras = pv.topping_id 
	GROUP BY ctee.order_id,ctee.pizza_id,ctee.pizza_name
),
final_cte_exclusisons_extras AS (
	SELECT 
		c1.order_id,c1.pizza_id,c1.pizza_name, 
		CONCAT(c1.pizza_name,' - Exclude ',full_exclusions_toppings, ' - Extra ',full_extras_toppings)
	FROM cte_exclusions_extras_only_exclusions c1
	JOIN cte_exclusions_extras_only_extras c2 USING(order_id)
),
final_union_orders AS (
	SELECT * FROM cte_extras
	UNION ALL
	SELECT * FROM cte_exclusions
	UNION ALL
	SELECT * FROM cte_orders_without_e
	UNION ALL 
	SELECT * FROM final_cte_exclusisons_extras
)
SELECT * FROM final_union_orders
ORDER BY order_id;


