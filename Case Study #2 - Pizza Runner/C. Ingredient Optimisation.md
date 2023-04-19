### 1. What are the standard ingredients for each pizza?
````SQL
WITH BASE AS (
SELECT 
	pizza_id, 
	UNNEST(string_to_array(toppings,','))::INT AS Topping_id
FROM pizza_runner.pizza_recipes pr
)
, toppings as (
	SELECT *
	FROM pizza_runner.pizza_toppings
)
SELECT b.pizza_id,STRING_AGG(t.topping_name,',') AS topping_name
FROM BASE b
JOIN pizza_runner.pizza_toppings t ON b.Topping_id = t.topping_id
GROUP BY 1
ORDER BY 1
````
Answer: 
| pizza_id |             topping_name              |
|---------|--------------------------------------|
|    1    | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami|
|    2    | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce|


