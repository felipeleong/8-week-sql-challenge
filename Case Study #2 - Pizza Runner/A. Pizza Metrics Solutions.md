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

### 4. How many of each type of pizza was delivered?
````sql
SELECT 
	pn.pizza_name,
	COUNT(*) AS "delivered_pizza_count"
FROM pizza_runner.customer_orders co
JOIN pizza_runner.pizza_names pn ON pn.pizza_id = co.pizza_id
JOIN pizza_runner.runner_orders ro ON ro.order_id = co.order_id
WHERE ro.distance IS NOT NULL OR ro.distance::numeric != 0
GROUP BY 1;
````
### Answer:
| pizza_name   | delivered_pizza_count |
|--------------|-----------------------|
| Meatlovers   | 9                     |
| Vegetarian   | 3                     |

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
````sql
SELECT
	co.customer_id,
	SUM(CASE WHEN pn.pizza_name = 'Vegetarian' THEN 1 ELSE 0 END)AS Vegetarian ,
	SUM(CASE WHEN pn.pizza_name = 'Meatlovers' THEN 1 ELSE 0 END)AS Meatlovers
FROM pizza_runner.customer_orders AS co
JOIN pizza_runner.pizza_names AS pn 
ON co.pizza_id = pn.pizza_id 
GROUP BY 1
ORDER BY 1;
````
### Answer:
| customer_id | vegetarian | meatlovers |
|-------------|------------|------------|
| 101         | 1          | 2          |
| 102         | 1          | 2          |
| 103         | 1          | 3          |
| 104         | 0          | 3          |
| 105         | 1          | 0          |

### 6. What was the maximum number of pizzas delivered in a single order?
````sql
WITH BASE AS (
	SELECT
		co.order_id,
		co.pizza_id,
		ROW_NUMBER() OVER(PARTITION BY co.order_id ORDER BY co.pizza_id) AS count_pizza
	FROM pizza_runner.customer_orders co
	JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
	WHERE ro.distance IS NOT NULL OR ro.distance::numeric != 0
)
SELECT order_id,count_pizza
FROM BASE
WHERE count_pizza = (SELECT MAX(count_pizza) FROM BASE);

--Other option
SELECT
	co.order_id,
	COUNT(co.pizza_id) as "Pizzas delivered in a single order"
FROM pizza_runner.runner_orders AS ro
JOIN pizza_runner.customer_orders AS co
ON ro.order_id = co.order_id
WHERE ro.distance IS NOT NULL OR ro.distance::numeric != 0
GROUP BY 1 
ORDER BY 2 DESC
LIMIT 1;

````
### Answer:
| order_id | count_pizza |
| -------- | ----------- |
| 4        | 3           |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
````sql
SELECT 	
	co.customer_id,
	SUM(CASE WHEN co.exclusions IS NOT NULL OR co.extras IS NOT NULL THEN 1 ELSE 0 END ) AS "delivered pizza with changes",
	SUM(CASE WHEN co.exclusions IS NULL AND co.extras IS NULL THEN 1 ELSE 0 END) AS "delivered pizza with no changes"
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE ro.distance IS NOT NULL OR ro.distance::numeric != 0
GROUP BY 1
ORDER BY 1;

````
### Answer:
| customer_id | delivered pizza with changes | delivered pizza with no changes |
|-------------|-------------------------------|---------------------------------|
| 101         | 0                             | 2                               |
| 102         | 0                             | 3                               |
| 103         | 3                             | 0                               |
| 104         | 2                             | 1                               |
| 105         | 1                             | 0                               |

### 8. How many pizzas were delivered that had both exclusions and extras?
````sql
SELECT 
	SUM(CASE WHEN co.exclusions IS NOT NULL AND co.extras IS NOT NULL THEN 1 ELSE 0 END)AS "pizzas delivered"
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE ro.distance IS NOT NULL OR ro.distance::numeric != 0; 
````
### Answer:
| Pizzas delivered |
|------------------|
|         1        |

### 9. What was the total volume of pizzas ordered for each hour of the day?
````sql
SELECT
	EXTRACT(HOUR FROM order_time) AS Hour,
	COUNT(pizza_id) AS "Total volume of pizza"
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 1;
````
### Answer:
| hour | Total volume of pizza |
|------|----------------------|
| 11   | 1                    |
| 13   | 3                    |
| 18   | 3                    |
| 19   | 1                    |
| 21   | 3                    |
| 23   | 3                    |
 
 ### 10. What was the volume of orders for each day of the week?
 ````sql
 SELECT
	TO_CHAR(order_time, 'Day') AS "Day of week",
	COUNT(order_id) AS "Total volume of orders"
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 2 desc;
 ````
### Answer:
| Day of week | Total volume of orders |
|------------|-----------------------|
| Saturday   | 5                     |
| Wednesday  | 5                     |
| Thursday   | 3                     |
| Friday     | 1                     |



