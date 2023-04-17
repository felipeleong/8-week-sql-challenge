### 1. How many pizzas were ordered?
````sql
SELECT 
	COUNT(*) AS "number of pizzas ordered"
FROM pizza_runner.customer_orders;
````
### Answer:
| number of pizzas ordered |
| ----------------------- |
|            14           |

### 2. How many unique customer orders were made?
````sql
SELECT 
	COUNT(DISTINCT order_id) AS "Unique customer orders"
FROM pizza_runner.customer_orders;
````
### Answer:
| Unique customer orders |
|-----------------------|
|           10          |

### 3. How many successful orders were delivered by each runner?
````sql
SELECT 
	runner_id,
	COUNT(*) AS "successful orders"
FROM pizza_runner.runner_orders
WHERE distance IS NOT NULL OR distance::numeric != 0
GROUP BY 1
ORDER BY 1;
````
### Answer:
| runner_id | successful orders |
| --------- | ----------------- |
| 1         | 4                 |
| 2         | 3                 |
| 3         | 1                 |
