### 1. What are the standard ingredients for each pizza?
````SQL
WITH BASE AS (
SELECT 
	pn.pizza_name, 
	UNNEST(string_to_array(toppings,','))::INT AS Topping_id
FROM pizza_runner.pizza_recipes pr
JOIN pizza_runner.pizza_names pn ON pr.pizza_id = pn.pizza_id
)
, toppings as (
	SELECT *
	FROM pizza_runner.pizza_toppings
)
SELECT b.pizza_name,STRING_AGG(t.topping_name,',') AS topping_name
FROM BASE b
JOIN pizza_runner.pizza_toppings t ON b.Topping_id = t.topping_id
GROUP BY 1
ORDER BY 1
````
Answer: 
| pizza_name |             topping_name              |
|------------|--------------------------------------|
| Meatlovers  | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami|
| Vegetarian | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce|


