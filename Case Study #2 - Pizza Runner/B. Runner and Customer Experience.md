### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
````sql
SELECT 
	EXTRACT(WEEK FROM registration_date) AS registration_week,
	COUNT(runner_id) AS runner_signup
FROM pizza_runner.runners
GROUP BY 1;
````
### Answer
| registration_week | runner_signup |
|------------------|---------------|
| 1                | 1            |
| 2               |   2         |
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

*we can conclude that the average time it takes to make an extra pizza is between 6 and 11 minutes.
Furthermore, the average time to make two pizzas is 18 minutes, therefore to make one pizza it should take on average 9 minutes and not 12 minutes.*

### 4.What was the average distance travelled for each customer?
````sql
SELECT
	co.customer_id,
	ROUND(AVG(distance::numeric),2) AS "Average distance"
FROM pizza_runner.runner_orders ro
JOIN pizza_runner.customer_orders co ON co.order_id = ro.order_id
WHERE duration IS NOT NULL
GROUP BY 1
ORDER BY 1;

````
### Answer:
| customer_id | Average distance |
|-------------|-----------------|
| 101         | 20.00           |
| 102         | 16.73           |
| 103         | 23.40           |
| 104         | 10.00           |
| 105         | 25.00           |

````sql
-------- FOR EACH RUNNER
SELECT
	runner_id,
	ROUND(AVG(distance::numeric),2) AS "Average distance"
FROM pizza_runner.runner_orders ro
WHERE duration IS NOT NULL
GROUP BY 1
ORDER BY 1;
````
### Answer:
| runner_id | Average distance |
|-----------|-----------------|
| 1         | 15.85           |
| 2         | 23.93           |
| 3         | 10.00           |

### 5. What was the difference between the longest and shortest delivery times for all orders?
````sql
SELECT 
	MAX(duration)::numeric - MIN(duration)::numeric AS difference
FROM pizza_runner.runner_orders;
````
### Answer: 
| difference |
|------------|
|     30     |

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
````sql
SELECT
	runner_id,
	co.order_id,
	ROUND(AVG(distance::numeric/duration::numeric)*60,2) AS "average speed"
FROM pizza_runner.customer_orders co 
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE distance IS NOT NULL OR duration IS NOT NULL
GROUP BY 1,2
ORDER BY 2;
````
### Answer: 
| runner_id | order_id | average speed |
|----------|----------|-------|
| 1        | 1        | 37.50 |
| 1        | 2        | 44.44 |
| 1        | 3        | 40.20 |
| 2        | 4        | 35.10 |
| 3        | 5        | 40.00 |
| 2        | 7        | 60.00 |
| 2        | 8        | 93.60 |
| 1        | 10       | 60.00 |

*It is time to check runner 2, as it is the one with the highest average speed, with 93 km, it could be a database error or that runner 2 was very late in delivery times.*

### 7.What is the successful delivery percentage for each runner?
````sql
WITH BASE AS (
	SELECT 
		runner_id,
		SUM(CASE WHEN distance IS NULL AND duration IS NULL THEN 0 ELSE 1 END) AS "Successful delivery" ,
		COUNT(order_id) AS "Total orders"
	FROM pizza_runner.runner_orders
	GROUP BY 1
)
SELECT
	runner_id,ROUND(("Successful delivery"::NUMERIC /  "Total orders"::NUMERIC) *100,0)
FROM BASE
ORDER BY 1;

---WITHOUT CTE
SELECT 
	runner_id,
	ROUND((SUM(CASE WHEN distance IS NULL AND duration IS NULL THEN 0 ELSE 1 END)/ COUNT(order_id)::NUMERIC)*100,0)
FROM pizza_runner.runner_orders
GROUP BY 1
ORDER BY 1
````
### Answer: 

*Runner 1 has 100% successful delivery.*

*Runner 2 has 75% successful delivery.*

*Runner 3 has 50% successful delivery*

| runner_id | Successful delivery |
|-----------|-------|
| 1         | 100   |
| 2         | 75    |
| 3         | 50    |


