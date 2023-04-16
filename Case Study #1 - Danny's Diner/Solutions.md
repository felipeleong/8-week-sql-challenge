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
