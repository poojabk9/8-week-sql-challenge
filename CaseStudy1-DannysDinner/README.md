
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt=Image width=500 height=520>

## Overview
- [Problem Statement](#problem-statement)
- [Data Set](#data-set)
- [Questions and Solutions](#questions-and-solutions)

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
- Selected `customer_id` and applied `COUNT(DISTINCT order_date)` to calculate the number of unique days each customer visited.
- Ensured duplicate visits on the same day were counted only once per customer.
- Grouped results by `customer_id` to summarize visits per customer.
  
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

Approach:
- Created a CTE membership that joins the sales and members tables to identify purchases made after a customer’s membership join date.
- Applied DENSE_RANK() partitioned by customer_id and ordered by order_date to rank purchases for each member starting from their join date.
- Selected only the first ranked purchase (rnk = 1) for each customer.
- Joined with the menu table to retrieve product names.
- Displayed the results ordered by customer_id for clarity.

```sql
WITH members_first_order AS
(
	SELECT
		s.customer_id,
		join_date,
		order_date,
		product_id,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) AS rnk
	FROM sales s
	JOIN members m
		ON s.customer_id = m.customer_id
	WHERE order_date >= join_date
)
SELECT
	customer_id,
	product_name
FROM members_first_order m1
JOIN menu m2
	ON m1.product_id = m2.product_id
WHERE rnk = 1
ORDER BY customer_id
```

Output:
- `customer_id` -> Identifies the customer.
- `product_name` -> First product purchased after becoming a member.

| customer_id | product_name |
|-------------|--------------|
| A		      |	        curry|
| B		      |	        sushi|

***

### Q7. Which item was purchased just before the customer became a member?

Approach:
- Created a CTE `last_order` to capture each customer’s orders placed before their membership start date.
- Used the `DENSE_RANK()` function with ORDER BY `order_date` DESC inside the CTE to rank orders in reverse chronological order, so the most recent pre-membership purchase gets rank 1.
- Filtered only orders where `order_date < join_date` to ensure purchases were strictly before membership started.
- Selected rows with `last_order_rank = 1` to identify the last order before becoming a member.
- Joined with the menu table to fetch product names and ordered the results by `customer_id` for clarity.

```sql
WITH last_order AS
(
SELECT s.customer_id, join_date, order_date, product_id,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS last_order_rank
FROM sales s
JOIN members m
	ON s.customer_id = m.customer_id
WHERE order_date < join_date
)
SELECT customer_id, m.product_name
FROM last_order l
JOIN menu m
	ON l.product_id = m.product_id
WHERE last_order_rank = 1
ORDER BY customer_id
```

Output:
- customer_id -> Identifies the customer.
- product_name -> Product purchased just before becoming a member.
  
| customer_id | product_name |
|-------------|--------------|
| A		      |	        sushi|
| A		      |	        curry|
| B		      |	        sushi|

***

### Q8. What is the total items and amount spent for each member before they became a member?

Approach:
- Selected `customer_id` from the `sales` table.
- Joined the `members` table to filter orders placed before the `join_date`.
- Joined the `menu` table to bring in product prices.
- Counted `customer_id` to calculate the total number of items ordered before membership.
- Summed `price` to find the total amount spent.
- Grouped results by `customer_id` and ordered by `customer_id`.

```sql
SELECT 
	s.customer_id, 
	COUNT(s.customer_id) AS total_items, 
	SUM(m.price) AS amount_spent
FROM sales s
JOIN members mb
	ON s.customer_id = mb.customer_id AND s.order_date < mb.join_date
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
```

Output:
- `customer_id` -> Identifies the customer.
- `total_items` -> Number of products purchased before becoming a member.
- `amount_spent` -> Money spent before becoming a member.
  
| customer_id | total_items | amount_spent |
|-------------|-------------|--------------|
| A		      |	           2|            25|
| B		      |	           3|            40|

***

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Approach:
- Joined `sales` and `menu` tables to access purchase details and prices.
- Applied a CASE expression to calculate points: price * 20 for sushi, price * 10 for other items.
- Summed the points for each customer.
- Grouped by `customer_id` and ordered results.

```sql
SELECT 
	s.customer_id,
	SUM(
		CASE
			WHEN m.product_name = 'sushi' THEN (m.price * 20)
			ELSE (m.price * 10)
	END) AS points
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
```

Output:
- `customer_id` -> Identifies the customer.
- `points` -> Points accumulated by each customers.

| customer_id | points |
|-------------|--------|
| A		      |	    860|
| B		      |     940|
| C		      |	    360|
  
***

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


Approach:
- For the first 7 days after a customer’s join_date (including the join date itself), they earn 2× points on all items. After this period, the normal points rules apply: Sushi = 20 points per dollar and Other items = 10 points per dollar.
- Created a CTE `members_points` to calculate points for each order by joining the `sales`, `menu`, and `members` tables.
- Applied a `CASE` statement to handle the points logic: If `order_date` falls between `join_date` and `join_date + 6`, then all items earn double points `(price * 20)`. Otherwise, sushi earns `price * 20` and all other items earn `price * 10`.
- Filtered results to only include orders placed on or before `2021-01-31`.
- Aggregated the points using `SUM(points)` grouped by `customer_id` to get the final totals for each customer by the end of January.

```sql
WITH members_points AS 
(
SELECT s.customer_id, order_date, join_date, s.product_id, product_name, price,
CASE
	WHEN order_date BETWEEN join_date AND (join_date + 6) THEN (m.price * 20)
	ELSE CASE 
		WHEN m.product_name = 'sushi' THEN (m.price * 20)
		ELSE (m.price * 10)
	END
END AS points
FROM sales s
JOIN members mb
	ON s.customer_id = mb.customer_id
JOIN menu m
	ON s.product_id = m.product_id
WHERE order_date <= '2021-01-31'
)
SELECT customer_id, SUM(points) AS points
FROM members_points
GROUP BY customer_id
```

Output:
Output:
- `customer_id` -> Identifies the customer.
- `points` -> Points accumulated by members at the end of January.

| customer_id | points |
|-------------|--------|
| A		      |	   1370|
| B		      |     820|

