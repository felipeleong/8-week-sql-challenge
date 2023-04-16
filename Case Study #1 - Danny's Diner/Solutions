/*What is the total amount each customer spent at the restaurant?
select * from dannys_diner.members
select * from dannys_diner.sales
select * from dannys_diner.menu
*/
SELECT 
	s.customer_id,
	sum(m.price) as total_amount
FROM dannys_diner.sales s
JOIN dannys_diner.menu m ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 1;

/*How many days has each customer visited the restaurant?*/
SELECT 
	customer_id,
	COUNT(DISTINCT order_date) as date
FROM dannys_diner.sales
GROUP BY 1
ORDER BY 1;

/*What was the first item from the menu purchased by each customer?*/
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

/*What is the most purchased item on the menu and how many times was it purchased by all customers?*/
SELECT 
	m.product_name,
	COUNT(s.product_id) as count_of_product
FROM dannys_diner.sales s 
JOIN dannys_diner.menu m ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1


/*Which item was the most popular for each customer?*/
WITH BASE AS (
	SELECT 
		s.customer_id,
		m.product_name,
		COUNT(s.product_id) as count_of_product,
		ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY COUNT(s.product_id) DESC) as RANK
	FROM dannys_diner.sales s 
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	GROUP BY 1,2
)---SI SOLO SE QUIERE VER UN PRODUCTO USAMOS ROW_NUMBER, COMO AHI DICE ITEM Y NO ITEMS, SOLO PUSE UNO
SELECT customer_id,product_name,count_of_product
FROM BASE 
WHERE RANK = 1

/*Which item was purchased first by the customer after they became a member?*/
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

/*Which item was purchased just before the customer became a member?*/
WITH BASE AS (
	SELECT s.customer_id,m.product_name, p.join_date, s.order_date,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date desc) as first_item
	FROM dannys_diner.members p
	JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
	WHERE s.order_date < p.join_date
)
SELECT customer_id,product_name, join_date, order_date
FROM BASE 
WHERE first_item = 1 
ORDER BY join_date,order_date 

/*What is the total items and amount spent for each member before they became a member?*/

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

/* If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?*/
WITH BASE AS (
	SELECT
		s.customer_id,
		CASE WHEN m.product_name = 'sushi' THEN m.price * 20  ELSE m.price *10 END AS points
	FROM dannys_diner.sales s 
	JOIN dannys_diner.menu m ON m.product_id = s.product_id
)
SELECT 
	customer_id,SUM(points)
FROM BASE
GROUP BY 1
ORDER BY 1  

/*In the first week after a customer joins the program (including their join date) they earn 2x points on all items, 
not just sushi — h
how many points do customer A and B have at the end of January?*/

SELECT
	s.customer_id, 
	SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20 
	   WHEN s.order_date BETWEEN p.join_date AND  p.join_date + interval '6 days' THEN m.price * 20
	   ELSE m.price * 10 END) AS points
FROM dannys_diner.members p
JOIN dannys_diner.sales s ON s.customer_id = p.customer_id
JOIN dannys_diner.menu m ON m.product_id = s.product_id
WHERE s.order_date < '2021-01-31'::date
GROUP BY 1
ORDER BY 1


/*join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)*/
SELECT 
	s.customer_id, s.order_date,m.product_name,m.price,
	CASE WHEN s.order_date >= p.join_date THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales s 
LEFT JOIN dannys_diner.members p ON p.customer_id = s.customer_id
LEFT JOIN dannys_diner.menu m ON m.product_id = s.product_id
ORDER BY 1,2

/*Danny also requires further information about the ranking of customer products, 
but he purposely does not need the ranking for non-member purchases so he expects null 
ranking values for the records when customers are not yet part of the loyalty program.*/
WITH BASE AS (
SELECT 
	s.customer_id, s.order_date,m.product_name,m.price,p.join_date,
	CASE WHEN s.order_date >= p.join_date THEN 'Y' ELSE 'N' END AS member
FROM dannys_diner.sales s 
LEFT JOIN dannys_diner.members p ON p.customer_id = s.customer_id
LEFT JOIN dannys_diner.menu m ON m.product_id = s.product_id
)
SELECT *,
	CASE WHEN member = 'N' THEN NULL ELSE  
	RANK() OVER(PARTITION BY customer_id,member ORDER BY order_date) END AS ranking
FROM BASE



