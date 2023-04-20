*create a view with the information of de pizza_id, pizza_name, toppings name and id*

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
GROUP BY 1
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
````
### Answer:
| extra_topping_id | topping_name | count of extra topping |
|-----------------|--------------|-----------------------|
| 1               | Bacon        | 4                     |
| 4               | Cheese       | 1                     |
| 5               | Chicken      | 1                     |

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
````
### Answer:
| exclusions_topping_id | topping_name | count of exclusions topping |
|----------------------|--------------|-----------------------------|
| 4                    | Cheese       | 4                           |
| 6                    | Mushrooms    | 1                           |
| 2                    | BBQ Sauce    | 1                           |

*The most commonly exclusion is Cheese*

