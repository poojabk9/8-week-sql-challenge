## A. Pizza Metrics 🍕

### Questions and Solutions

#### Q1. How many pizzas were ordered?

Approach:
- Each record in the `clean_customer_orders` table represents a single pizza order.
- Therefore, we can simply count the number of rows (or order_id values) in the table.
- The `COUNT()` aggregate function is used to return the total count of `order_id` entries, and the result is renamed as `no_of_pizza` for readability.

```sql
SELECT COUNT(order_id) AS no_of_pizza
FROM clean_customer_orders
```

Output:
- `no_of_pizza` -> 14 is the total number of Pizza's ordered. 

| no_of_pizza |
|-------------|
|          14	|	

#### Q2. How many unique customer orders were made?

Approach:
- Used the `COUNT(DISTINCT order_id)` function to count unique entries in the `order_id` column.
- This ensures that each order is counted only once, even if duplicate records exist.
- Applied the query on the `clean_customer_orders` table to return the total number of distinct orders placed by customers.

```sql
SELECT COUNT(DISTINCT(order_id)) AS unique_customer_orders
FROM clean_customer_orders
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
GROUP BY runner_id
```

Output:
- `runner_id ` -> Lists unique runners id
- `successful_orders` -> Number of successful orders by each runner.

| runner_id | successful_orders |
|-----------|-------------------|
|         1 |                 4 |
|         2 |                 3 |
|         3 |                 1 |

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
GROUP BY p.pizza_name
```

Output:
- `pizza_name` -> Name of the Pizza.
- `pizza_delivered` -> Number of time the pizza was delivered.

| pizza_name | pizza_delivered |
|------------|-----------------|
| Vegetarian |               3 |
| Meatlovers |               9 |
