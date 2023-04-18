### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
````sql
SELECT 
	COUNT(*) AS "number of pizzas ordered"
FROM pizza_runner.customer_orders;
````
### Answer
| registration_week | runner_signup |
|------------------|---------------|
| 1                | 2             |
| 2               |   1           |
| 3                | 1             |
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
````sql
SELECT 
	ro.runner_id,
	ROUND(AVG(EXTRACT(MINUTE FROM AGE(pickup_time::timestamp,order_time::timestamp))),0) AS "average time in minutes"
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON ro.order_id = co.order_id
WHERE RO.pickup_time IS NOT NULL
GROUP BY 1
ORDER BY 1;
````
### Answer:
| runner_id | average time in minutes |
|-----------|------------------------|
|    1      |           15           |
|    2      |           23           |
|    3      |           10           |

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
````sql
WITH BASE AS (
SELECT 
	co.order_id,
	co.order_time,
	ro.pickup_time,
	EXTRACT(MINUTE FROM AGE(pickup_time::timestamp,order_time::timestamp)) AS "average time in minutes",
	COUNT(co.pizza_id) AS "Total volume of pizza"
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON ro.order_id = co.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY 1,2,3
)
SELECT
	"Total volume of pizza",
	ROUND(AVG("average time in minutes"),0) AS "AVG preparation time in minutes"
FROM BASE
GROUP BY 1
ORDER BY 1;
````
### Answer:
| Total volume of pizza | AVG preparation time in minutes |
|-----------------------|---------------------------------|
| 1                     | 12                              |
| 2                     | 18                              |
| 3                     | 29                              |

/*we can conclude that the average time it takes to make an extra pizza is between 6 and 11 minutes.
Furthermore, the average time to make two pizzas is 18 minutes, therefore to make one pizza it should take on average 9 minutes and not 12 minutes.*/



