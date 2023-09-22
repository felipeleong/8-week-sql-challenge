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
WITH Without_extra_exclusions AS (
	SELECT 
		co.order_id,
		co.pizza_id,
		pn.pizza_name,
		null AS exclusions,
		null AS extras
	FROM pizza_runner.customer_orders co
	JOIN pizza_runner.pizza_names pn ON pn.pizza_id = co.pizza_id
	WHERE co.exclusions IS NULL AND extras IS  NULL
)
,exclusions_topping AS (
	SELECT 
		et.order_id,
		et.pizza_id,
		et.pizza_name,
		pt.topping_name AS exclusions,
		null AS extras
	FROM(
		SELECT
			co.order_id,
			co.pizza_id,
			string_to_table(co.exclusions,',')::integer AS exclusions,
			pn.pizza_name
		FROM pizza_runner.customer_orders co
		JOIN pizza_runner.pizza_names pn ON pn.pizza_id = co.pizza_id
	) et
	JOIN pizza_runner.pizza_toppings pt ON pt.topping_id = et.exclusions
)
,extras_topping AS (
SELECT 
		et.order_id,
		et.pizza_id,
		et.pizza_name,
		null AS exclusions,
		pt.topping_name AS extras
	FROM(
		SELECT
			co.order_id,
			co.pizza_id,
			string_to_table(co.extras,',')::integer AS extras,
			pn.pizza_name
		FROM pizza_runner.customer_orders co
		JOIN pizza_runner.pizza_names pn ON pn.pizza_id = co.pizza_id
	) et
	JOIN pizza_runner.pizza_toppings pt ON pt.topping_id = et.extras
)
, final_union AS (
	SELECT * FROM Without_extra_exclusions
	UNION ALL
	SELECT * FROM exclusions_topping
	UNION ALL
	SELECT * FROM extras_topping
)
SELECT *,
	CASE WHEN exclusions IS NULL AND extras IS NULL THEN pizza_name
		 WHEN exclusions IS NOT NULL THEN CONCAT(pizza_name,' - Exclude ',exclusions) 
		 ELSE CONCAT(pizza_name,' - Extra ',extras) END AS full_order
FROM final_union
ORDER BY order_id;

````
| order_id | pizza_id | pizza_name  | exclusions           | extras                  | full_order                          |
| -------- | -------- | -----------| --------------------|------------------------| -----------------------------------|
| 1        | 1        | Meatlovers |                      |                         | Meatlovers                          |
| 2        | 1        | Meatlovers |                      |                         | Meatlovers                          |
| 3        | 2        | Vegetarian |                      |                         | Vegetarian                         |
| 3        | 1        | Meatlovers |                      |                         | Meatlovers                          |
| 4        | 2        | Vegetarian | Cheese              |                         | Vegetarian - Exclude Cheese        |
| 4        | 1        | Meatlovers | Cheese              |                         | Meatlovers - Exclude Cheese        |
| 4        | 1        | Meatlovers | Cheese              |                         | Meatlovers - Exclude Cheese        |
| 5        | 1        | Meatlovers |                      | Bacon                   | Meatlovers - Extra Bacon           |
| 6        | 2        | Vegetarian |                      |                         | Vegetarian                         |
| 7        | 2        | Vegetarian |                      | Bacon                   | Vegetarian - Extra Bacon           |
| 8        | 1        | Meatlovers |                      |                         | Meatlovers                          |
| 9        | 1        | Meatlovers | Cheese              |                         | Meatlovers - Exclude Cheese        |
| 9        | 1        | Meatlovers |                      | Chicken, Bacon          | Meatlovers - Extra Chicken, Bacon |
| 10       | 1        | Meatlovers | BBQ Sauce, Mushrooms |                         | Meatlovers - Exclude BBQ Sauce     |
| 10       | 1        | Meatlovers | Mushrooms           |                         | Meatlovers - Exclude Mushrooms     |
| 10       | 1        | Meatlovers |                      |                         | Meatlovers                          |
| 10       | 1        | Meatlovers |                      | Cheese, Bacon           | Meatlovers - Extra Cheese, Bacon   |


