### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
````sql
SELECT 
	COUNT(*) AS "number of pizzas ordered"
FROM pizza_runner.customer_orders;
````
