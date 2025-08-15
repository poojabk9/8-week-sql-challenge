# 🍜 Danny's Dinner 🍜

## Overview

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

***

### Data Set

Danny’s Diner contains 3 tables:

- `sales` – customer_id, order_date, product_id  
- `menu` – product_id, product_name, price  
- `members` – customer_id, join_date  

These tables can be found [on the official challenge site](https://8weeksqlchallenge.com/case-study-1/).

***

### Questions and Solutions

#### Q1. What is the total amount each customer spent at the restaurant?

Approach:
- Joined the `sales` and `menu` tables on the `product_id` column.  
- Aggregated the `price` column using the `SUM()` function to calculate the total amount spent by each customer, and assigned the alias `money_spent`.  
- Grouped the results by `customer_id` and ordered them in ascending order.

```sql
 SELECT
	sales.customer_id,
	SUM(menu.price) AS money_spent 
 FROM sales
 INNER JOIN menu
 	ON sales.product_id = menu.product_id
 GROUP BY customer_id 
 ORDER BY customer_id ASC;
```

Output:
- `customer_id` -> Identifies the customer.
- `money_spent` -> Money spent by each customer.

| customer_id | money_spent | 
|-------------|-------------|
| A			  |			  76|
| B			  |			  74|
| C			  |			  36|

***

#### Q2. How many days has each customer visited the restaurant?

Approach:
- I selected `customer_id` and counted the distinct `order_date` values for each customer to find the total number of unique days they visited the restaurant.
- I used COUNT(DISTINCT ...) to ensure each day was counted only once per customer.
- Then, I grouped the results by `customer_id`.
  
```sql
SELECT
	customer_id,
	COUNT(DISTINCT(order_date)) AS no_of_Days
FROM sales
GROUP BY customer_id;
```

Output:
- `customer_id` -> Identifies the customer.
- `money_spent` -> Number of days each customer visited.

| customer_id | no_of_Days  |
|-------------|-------------|
| A			  |	 		   4|
| B			  |			   6|
| C			  |			   2|

***

#### Q3. What was the first item from the menu purchased by each customer?

Approach:
- Created a CTE `first_order` to identify the earliest `order_date` for each customer.
- Created a second CTE `first_product` to retrieve the products purchased on each customer’s first visit, assigning a rank based on the `order_date` using the `RANK()` window function.
- Joined `first_product` with the `menu` table to get the `product_name`.
- Filtered the results to include only rows where the rank equals `1`, and grouped by `customer_id` and `product_name`.

```sql
WITH first_order AS
(
	SELECT
		customer_id,
		MIN(order_date) AS first_visit
	FROM sales
	GROUP BY customer_id
),
first_product AS
(
	SELECT
		f.customer_id,
		first_visit,
		s.product_id,
		RANK() OVER(PARTITION BY f.customer_id ORDER BY s.order_date) AS rnk
	FROM first_order f
	JOIN sales s
		ON f.customer_id = s.customer_id
		AND f.first_visit = s.order_date
)
	SELECT
		f1.customer_id,
		m.product_name
	FROM first_product f1
	JOIN menu m
		ON f1.product_id = m.product_id
	WHERE rnk = 1
	GROUP BY f1.customer_id, m.product_name
```

Output:
- `customer_id` -> Identifies the customer.
- `product_name` -> First item/s ordered by the customer.

| customer_id | product_name |
|-------------|--------------|
| A			  |		    curry|
| A			  |		    sushi|
| B			  |		    curry|
| C			  |		    ramen|

***

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?

Approach 1: Simple Solution
- Join the `sales` table with the `menu` table on `product_id` to get the product names for each sale.
- Group the results by `product_name` to aggregate purchases for each item.
- Count the number of times each product was purchased using COUNT(*).
- Sort the results in descending order of purchase count so the most popular item is first.
- Limit the output to the top result using LIMIT 1 (assuming no need to handle ties).

```sql
SELECT 
	m.product_name, 
	COUNT(*) AS purchase_count
FROM sales s
JOIN menu m
 ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_count DESC
LIMIT 1
```

Approach 2: Handeling ties (In case there are more than one popular item)
- Aggregate purchases per product in a CTE (CTE1) by grouping sales on `product_id` and counting how many times each product was sold.
- Join the aggregated results with the `menu` table to get the product names.
- In a second CTE (CTE2), assign ranks to products using the RANK() window function, ordered by `purchase_count` descending.
- Filter to only include rows where `rnk = 1`, ensuring all top-ranked items appear in case of a tie.

```sql
WITH CTE1 AS
(
	SELECT
		product_id,
		COUNT(product_id) AS purchase_count
	FROM sales
	GROUP BY product_id
),
CTE2 As
(
	SELECT
		m.product_name,
		purchase_count, 
		RANK() OVER (ORDER BY purchase_count DESC) AS rnk
	FROM CTE c
	JOIN menu m
		ON c.product_id = m.product_id
)
SELECT
	product_name,
	purchase_count
FROM CTE2
Where rnk = 1
```

Output:
- `product_name` -> Name of the most purchased Item on the Menu.
- `purchase_count` -> Number of times the Item was purchased.

| product_name | purchase_count |
|--------------|----------------|
| ramen		   |		       8|

***

### Q5. Which item was the most popular for each customer?

Approach:
- Created a CTE `item_count` to calculate how many times each customer purchased each product by grouping `sales` table on `customer_id` and `product_id` and applying `COUNT(product_id)`.
- Created a second CTE `ranking_product_count` by joining `item_count` with the `menu` table on `product_id` to get the product names.
- Used the `RANK()` window function, partitioned by `customer_id` and ordered by `product_count` in descending order, to rank each product for every customer.
- Filtered the results to only include rows where `rnk = 1`, returning each customer’s most frequently purchased product(s), including ties.

```sql
WITH item_count AS
(
	SELECT 
		customer_id, 
		product_id, 
		COUNT(product_id) AS product_count
	FROM sales
	GROUP BY customer_id, product_id
),
ranking_product_count AS
(
	SELECT 
		customer_id, 
		i.product_id, 
		product_count,
		RANK() OVER(PARTITION BY customer_id ORDER BY product_count DESC) AS rnk, m.product_name
	FROM item_count i
	JOIN menu m
		ON i.product_id = m.product_id
)
SELECT 
	customer_id, 
	product_name, 
	product_count
FROM ranking_product_count
WHERE rnk = 1
```

Output:
- customer_id -> Identifies the customer.
- product_name -> Most purchased product.
- product_count -> Number of times the product was brought.
  
| customer_id | product_name | purchase_count |
|-------------|--------------|----------------|
| A      	  | ramen        |              3 |
| B      	  | curry        |              2 |
| B      	  | sushi        |              2 |
| B      	  | ramen        |              2 |
| C      	  | ramen        |              3 |

***

### Q6. Which item was purchased first by the customer after they became a member?

***

### Q7. Which item was purchased just before the customer became a member?

***

### Q8. What is the total items and amount spent for each member before they became a member?

***

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

***

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?





