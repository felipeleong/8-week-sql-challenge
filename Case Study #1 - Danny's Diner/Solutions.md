### 1. What is the total amount each customer spent at the restaurant?
````sql
SELECT 
	s.customer_id,
	sum(m.price) as total_amount
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 1;
````
| customer_id | total_amount |
| --- | --- |
| A | 76 |
| B | 74 |
| C | 36 |

### 2. How many days has each customer visited the restaurant?
````sql
SELECT 
	customer_id,
	COUNT(DISTINCT order_date) as visit_count
FROM dannys_diner.sales
GROUP BY 1
ORDER BY 1;
````
#### Answer:
| customer_id | visit_count |
| --- | --- |
| A | 4 |
| B | 6 |
| C | 2 |


### 3. What was the first item from the menu purchased by each customer?
````sql
WITH BASE AS (
	SELECT
		s.customer_id,
		m.product_name,
		s.order_date,
		DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY s.order_date) as first_purchased
	FROM dannys_diner.sales s 
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	
)
SELECT 
	customer_id,product_name
FROM BASE
WHERE first_purchased = 1
GROUP BY customer_id, product_name
ORDER BY 1;
````
#### Answer:
| customer_id | product_name |
| --- | --- |
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT 
	m.product_name,
	COUNT(s.product_id) as count_of_product
FROM dannys_diner.sales s 
JOIN dannys_diner.menu m ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
````
#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


### 5. Which item was the most popular for each customer?
````sql
WITH BASE AS (
	SELECT 
		s.customer_id,
		m.product_name,
		COUNT(s.product_id) as count_of_product,
		DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id) DESC) as RANK
	FROM dannys_diner.sales s 
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	GROUP BY 1,2
)
SELECT customer_id,product_name,count_of_product
FROM BASE 
WHERE RANK = 1
````
#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

### 6. Which item was purchased first by the customer after they became a member?
````sql
WITH BASE AS (
	SELECT s.customer_id,m.product_name, p.join_date, s.order_date,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as first_item
	FROM dannys_diner.members p
	JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	WHERE s.order_date >= p.join_date
)
SELECT customer_id,product_name, join_date, order_date
FROM BASE 
WHERE first_item = 1 
ORDER BY join_date,order_date 
````
#### Answer:

|customer_id|product_name| join_date | 	 order_date|
|-----------|----------- |-----------|	-----------|
|A	    |	curry	 |2021-01-07 |	2021-01-07 |
|B	    |	sushi	 |2021-01-09 |	2021-01-11 |

### 7. Which item was purchased just before the customer became a member?
````sql
WITH BASE AS (
	SELECT s.customer_id,m.product_name, p.join_date, s.order_date,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) as first_item
	FROM dannys_diner.members p
	JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	WHERE s.order_date < p.join_date
)
SELECT customer_id,product_name, join_date, order_date
FROM BASE 
WHERE first_item = 1 
ORDER BY join_date,order_date 
````
### Answer:
|customer_id |product_name| join_date |	order_date|
|------------|------------|-----------|-----------|
|A	     |	sushi	  | 2021-01-07|	2021-01-01|
|A	     |	curry	  | 2021-01-07|	2021-01-01|
|B           |	sushi	  | 2021-01-09|	2021-01-04|

### 8. What is the total items and amount spent for each member before they became a member?
````sql
SELECT
	s.customer_id,
	SUM(m.price) as total_spend_amount,
	COUNT(DISTINCT s.product_id) AS total_items
FROM dannys_diner.members p
JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
JOIN dannys_diner.menu m ON m.product_id = s.product_id
WHERE s.order_date < p.join_date
GROUP BY 1
ORDER BY 1
````
### Answer:
| customer_id | total_items | total_spend_amount |
| --- | --- | --- |
| A | 2 | 25 |
| B | 2 | 40 |

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
````sql
WITH BASE AS (
	SELECT
		s.customer_id,
		CASE WHEN m.product_name = 'sushi' THEN m.price * 20  ELSE m.price *10 END AS points
	FROM dannys_diner.sales s 
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
)
SELECT 
	customer_id,SUM(points) as total_points
FROM BASE
GROUP BY 1
ORDER BY 1  
````
### Answer:
| customer_id | total_points |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
````sql
SELECT
	s.customer_id, 
	SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20 
	   WHEN s.order_date BETWEEN p.join_date AND  p.join_date + interval '6 days' THEN m.price * 20
	   ELSE m.price * 10 END) AS total_points
FROM dannys_diner.members p
JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
JOIN dannys_diner.menu m ON m.product_id = s.product_id
WHERE s.order_date < '2021-01-31'::date
GROUP BY 1
ORDER BY 1
````
### Answer:
| customer_id | total_points |
| --- | --- |
| A | 1370 |
| B | 820 |





