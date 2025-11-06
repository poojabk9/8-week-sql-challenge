## A. Pizza Metrics 🍕

### Questions and Solutions

#### Q1. How many pizzas were ordered?

Approach:
- Each record in the `clean_customer_orders` table represents a single pizza order.
- Therefore, we can simply count the number of rows (or order_id values) in the table.
- The `COUNT()` aggregate function is used to return the total count of `order_id` entries, and the result is renamed as `no_of_pizza` for readability.

```sql
SELECT COUNT(order_id) AS no_of_pizza
FROM clean_customer_orders;
```

Output:
- `no_of_pizza` -> 14 is the total number of Pizza's ordered. 

| no_of_pizza |
|-------------|
|          14 |

***

#### Q2. How many unique customer orders were made?

Approach:
- Used the `COUNT(DISTINCT order_id)` function to count unique entries in the `order_id` column.
- This ensures that each order is counted only once, even if duplicate records exist.
- Applied the query on the `clean_customer_orders` table to return the total number of distinct orders placed by customers.

```sql
SELECT COUNT(DISTINCT(order_id)) AS unique_customer_orders
FROM clean_customer_orders;
```

Output:
- `unique_customer_orders` -> There are 10 unique orders.

| unique_customer_orders |
|------------------------|
|                     10 |

#### Q3. How many successful orders were delivered by each runner?

Approach:
- Filtered the dataset to include only records where the cancellation column is NULL, representing successful deliveries.
- Grouped the results by `runner_id` to analyze each runner individually.
- Used the `COUNT(runner_id)` function to calculate the total number of successful orders completed by each runner.

```sql
SELECT runner_id, COUNT(runner_id) AS successful_orders
FROM clean_runner_orders
WHERE cancellation is NULL
GROUP BY runner_id;
```

Output:
- `runner_id ` -> Lists unique runners id
- `successful_orders` -> Number of successful orders by each runner.

| runner_id | successful_orders |
|-----------|-------------------|
|         1 |                 4 |
|         2 |                 3 |
|         3 |                 1 |

***

#### Q4. How many of each type of pizza was delivered?

Approach:
- Joined the `clean_customer_orders`, `clean_runner_orders`, and `pizza_names` tables to combine order, delivery, and pizza details.
- Filtered the records to include only successful deliveries where cancellation is NULL.
- Grouped the data by `pizza_name` to categorize the results by pizza type.
- Used the `COUNT(pizza_id)` function to calculate how many of each pizza type was delivered.

```sql
SELECT p.pizza_name, COUNT(c.pizza_id) AS pizza_delivered
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
WHERE r.cancellation is NULL
GROUP BY p.pizza_name;
```

Output:
- `pizza_name` -> Name of the Pizza.
- `pizza_delivered` -> Number of time the pizza was delivered.

| pizza_name | pizza_delivered |
|------------|-----------------|
| Vegetarian |               3 |
| Meatlovers |               9 |

***

#### Q5. How many Vegetarian and Meatlovers were ordered by each customer?

Approach:
- Joined the `clean_customer_orders` table with `pizza_names` to link each order with its corresponding pizza type.
- Grouped the data by `customer_id` and `pizza_name` to categorize orders for each customer by pizza type.
- Used the `COUNT(pizza_name)` function to calculate how many Vegetarian and Meatlovers pizzas each customer ordered.
- Ordered the results by `customer_id` for clear and organized output.

```sql
SELECT c.customer_id, p.pizza_name, COUNT(p.pizza_name)
FROM clean_customer_orders c
JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```

Output:
- `customer_id` -> Customer ID
- `pizza_name` -> Name of the Pizza
- `no_of_pizza` -> Number of each type of pizza's ordered by individual customer.

| customer_id | pizza_name | no_of_pizza |
|-------------|------------|-------------|
|         101 |	Meatlovers |	       2 |
|         101 | Vegetarian |	       1 |
|         102 |	Meatlovers |	       2 |
|         102 |	Vegetarian |	       1 |
|         103 |	Meatlovers |           3 | 
|         103 |	Vegetarian | 	       1 |
|         104 |	Meatlovers |           3 |
|         105 |	Vegetarian |      	   1 |

***

#### Q6. What was the maximum number of pizzas delivered in a single order?

Approach:
-  Joined `clean_customer_orders` with `clean_runner_orders` to include only delivered orders by filtering out records with cancellations.
- Grouped the data by `order_id` and counted the number of pizzas in each order to determine how many pizzas were delivered per order.
- Used a Common Table Expression (CTE) `max_pizza` to store these counts for readability.
- Retrieved the maximum pizza count across all orders using the `MAX()` function to identify the maximum number of pizzas delivered in a single order.

```sql
WITH max_pizza AS
(
SELECT c.order_id, COUNT(c.order_id) AS count_order_id
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
WHERE cancellation is NULL
GROUP BY c.order_id
ORDER BY c.order_id
)
SELECT MAX(count_order_id) AS max_pizza_delivered
FROM max_pizza;
```

Output:
- `max_pizza_delivered` -> 3 is the maximum number of pizzas delivered in a single order

| max_pizza_delivered | 
|---------------------|
|                   3 |

***

#### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

Approach:
- Joined `clean_customer_orders` and `clean_runner_orders` on `order_id` to include only delivered pizzas (cancellation IS NULL).
- Used conditional aggregation to classify pizzas as having changes (when exclusions or extras are not NULL) or no changes (when both are NULL).
- Applied `SUM` with `CASE` statements to count pizzas in each category for every customer.
- Grouped results by `customer_id` to display the number of changed and unchanged pizzas per customer.

```sql
SELECT c.customer_id,
	   SUM(CASE WHEN exclusions is NULL AND extras is NULL THEN 1 ELSE 0 END) AS pizzas_without_changes,
	   SUM(CASE WHEN exclusions is NOT NULL OR extras is NOT NULL THEN 1 ELSE 0 END) AS pizzas_with_changes
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
WHERE r.cancellation is NULL
GROUP BY c.customer_id;
```

Output:
- `customer_id` -> Unique Customer ID
- `pizzas_without_changes` -> Number of pizza that were customized by each customer
- `pizzas_with_changes` -> Number of pizza that were not customised by the customer.

| customer_id | pizzas_without_changes | pizzas_with_changes |
|-------------|------------------------|---------------------|
|         101 |                      2 |                   0 |
|         102 |                      3 |                   0 |
|         103 |                      0 |                   3 |
|         104 |                      1 |                   2 |
|         105 |                      0 |                   1 |

***

#### Q8. How many pizzas were delivered that had both exclusions and extras?

Approach:
- Joined `clean_customer_orders` and `clean_runner_orders` tables using `order_id` to include only delivered pizzas (cancellation IS NULL).
- Used a `CASE` statement within a `SUM` function to count pizzas where both exclusions and extras are not NULL, indicating that customers requested both modifications.
- Returned the total count of such pizzas as `pizza_count_with_exclusions_extras`

```sql
SELECT 
	SUM(CASE WHEN exclusions is NOT NULL AND extras is NOT NULL THEN 1 ELSE 0 END) AS pizza_count_with_exclusions_extras
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
WHERE r.cancellation is NULL
```

Output:
- `pizza_count_with_exclusions_extras` -> Count of pizzas delivered that had both exclusions and extras. 

| pizza_count_with_exclusions_extras |
|------------------------------------|
|                                  1 |

***

#### Q9. What was the total volume of pizzas ordered for each hour of the day?

Approach:
- Extracted the hour component from the `order_time` column using `EXTRACT(HOUR FROM order_time)` to identify the hour each order was placed.
- Grouped the data by the extracted hour to count the total number of pizzas ordered during each hour of the day.
- Ordered the results by hour to visualize the distribution of pizza orders throughout the day in chronological order.

```sql
SELECT
	EXTRACT(HOUR FROM order_time) AS hour_of_the_day,
	COUNT(order_time) AS pizza_count
FROM clean_customer_orders
GROUP BY hour_of_the_day
ORDER BY hour_of_the_day;
```

Output:
- `hour_of_the_day` -> Represents the hour of the day.
- `pizza_count` -> Number of Pizzas ordered by the hour.

| hour_of_the_day | pizza_count |
|-----------------|-------------|
|              11 |           1 |
|              13 |           3 |
|              18 |           3 |
|              19 |           1 |
|              21 |           3 |
|              23 |           3 |

***

#### Q10. What was the volume of orders for each day of the week?

Approach:
- Extracted the weekday name from the `order_time` column using the `TO_CHAR()` function with the 'Day' format.
- Counted the total number of pizzas ordered for each day using the `COUNT()` aggregate function.
- Grouped the results by the extracted day name to get the total orders per weekday.
- Ordered the final output by `Day_of_the_week` for a clean, readable summary.

```sql
SELECT 
	TO_CHAR(order_time, 'Day') AS day_of_the_week,
	COUNT(order_id) AS pizza_count
FROM clean_customer_orders
GROUP BY Day_of_the_week
ORDER BY Day_of_the_week;
```

Output:
- `hour_of_the_day` -> Represents the day of the week.
- `pizza_count` -> Number of Pizzas ordered on a weekday.

| day_of_the_week | pizza_count |
|-----------------|-------------|
|       Wednesday |           5 |
|        Thursday |           3 |
|        Saturday |           5 |
|          Friday |           1 |

