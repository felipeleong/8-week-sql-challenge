### 1.What is the total amount each customer spent at the restaurant?
````sql
SELECT 
	s.customer_id,
	sum(m.price) as total_amount
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 1;
````
#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

### 2.How many days has each customer visited the restaurant?
````sql
SELECT 
	customer_id,
	COUNT(DISTINCT order_date) as date
FROM dannys_diner.sales
GROUP BY 1
ORDER BY 1;
````
#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

### 3. What was the first item from the menu purchased by each customer?
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

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |





