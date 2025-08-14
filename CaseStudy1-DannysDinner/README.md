# 🍜 Danny's Dinner 🍜

## Overview

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

## Data Set

Danny’s Diner contains 3 tables:

- `sales` – customer_id, order_date, product_id  
- `menu` – product_id, product_name, price  
- `members` – customer_id, join_date  

These tables can be found [on the official challenge site](https://8weeksqlchallenge.com/case-study-1/).

## Questions and Solutions

### Q1. What is the total amount each customer spent at the restaurant?

To calculate each customer's total spending, I joined the "sales" and "menu" tables on "product_id", then aggregated the "price" column and grouped the results by "customer_id".

```sql
 SELECT sales.customer_id, SUM(menu.price) AS money_spent 
 FROM sales
 INNER JOIN menu
 	ON sales.product_id = menu.product_id
 GROUP BY customer_id 
 ORDER BY customer_id ASC;
```

### Q2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT(order_date)) AS No_of_Days
FROM sales
GROUP BY customer_id;
```

### Output:
| customer_id | product_name | 
| ----------- | ------------ |
| A			  |				4|
| B			  |				6|
| C			  |				2|


### Q3. What was the first item from the menu purchased by each customer?

```sql
WITH fp1 AS
(
SELECT 
	customer_id, 
	MIN(order_date) AS first_visit
FROM sales
GROUP BY customer_id
),
fp2 AS
(
SELECT 
	fp1.customer_id, 
	fp1.first_visit, 
	sales.product_id,
	ROW_NUMBER() OVER(PARTITION BY fp1.customer_id) AS rn
FROM fp1
JOIN sales
	ON fp1.customer_id = sales.customer_id
	AND fp1.first_visit = sales.order_date
)
SELECT 
	customer_id, 
	product_name
FROM fp2
JOIN menu
	ON fp2.product_id = menu.product_id
WHERE rn = 1
```

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?

### Q5. Which item was the most popular for each customer?

### Q6. Which item was purchased first by the customer after they became a member?

### Q7. Which item was purchased just before the customer became a member?

### Q8. What is the total items and amount spent for each member before they became a member?

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?





