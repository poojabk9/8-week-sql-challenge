## B. Runner and Customer Experience 🌟

### Questions and Solutions

#### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

Approach:
- The objective is to find how many runners signed up during each one-week period starting from 2021-01-01.
- Used the expression `FLOOR((registration_date - DATE '2021-01-01') / 7) + 1` to calculate the week number for each runner’s registration date.
- The subtraction (registration_date - DATE '2021-01-01') gives the number of days since the start date.
- Dividing this difference by 7 converts the days into weeks, and `FLOOR()` ensures the value is rounded down to the nearest whole week.
- Added +1 so the first 7 days (Jan 1–Jan 7) fall under Week 1.
- Grouped the results by the calculated `week_number` and counted the number of `runner_id` in each week to get total sign-ups.
- Finally, ordered the output by `week_number` to show the weekly sign-up trend in chronological order.

```sql
SELECT 
	FLOOR((registration_date - DATE '2021-01-01')/7) + 1 AS week_number,
	COUNT(runner_id) AS count_of_runners
FROM runners
GROUP BY week_number
ORDER BY week_number
```

Output:
- `week_number` -> Number of each week starting from DATE '2021-01-01'.
- `count_of_runners` -> Number of runners signed up during each week.

| week_number | count_of_runners |
|-------------|------------------|
|           1 |                2 |
|           2 |                1 |
|           3 |                1 |

***

#### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

Approach:
- Joined `clean_customer_orders` and `clean_runner_orders` to match each order with its corresponding runner.
- Calculated the pickup time delay by subtracting the customer’s `order_time` from the runner’s `pickup_time`.
- Converted the time difference into minutes using `EXTRACT(EPOCH)` (which returns seconds) and dividing by 60.
- Used `AVG()` to compute the average pickup time for each runner.
- Applied `ROUND(..., 2)` to format the result to two decimal places for cleaner readability.
- Grouped the results by runner_id to show each runner’s average pickup time.

```sql
SELECT
	r.runner_id, 
	ROUND(AVG(EXTRACT(EPOCH FROM(r.pickup_time - c.order_time)) / 60), 2) AS avg_pickup_time_minutes
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
GROUP BY r.runner_id
ORDER BY r.runner_id;
```

Output:
- `runner_id` -> Identifies each runner.
- `avg_pickup_time_minutes` -> Average minutes taken by each runner to arrive at the Pizza Runner HQ to pickup the order.

| runner_id | avg_pickup_time_minutes |
|-----------|-------------------------|
|         1 |                   15.68 |
|         2 |                   23.72 |
|         3 |                   10.47 |

***

#### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

Approach:
- Joined the `clean_customer_orders` and `clean_runner_orders` tables to link each order with its pickup timestamp.
- Created a CTE `prep_time` to calculate preparation details at the order level:
  	- Counted how many pizzas were included in each order.
  	- Calculated the prep time in minutes by taking the difference between pickup_time and order_time, converting the result from seconds to minutes using `EXTRACT(EPOCH FROM …) / 60`.
- In the final step, grouped by the number of pizzas in an order to understand how prep time varies with order size.
- Calculated the average preparation time for each pizza count and rounded the result for better readability.
- Ordered the results by pizza count to clearly observe patterns or correlations.


```sql
WITH prep_time AS
(
SELECT 
	c.order_id, 
	c.order_time, 
	r.pickup_time, 
	COUNT(c.order_id) AS no_of_pizzas,
	EXTRACT(EPOCH FROM(r.pickup_time - c.order_time)) / 60 AS prep_time_minutes
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
GROUP BY c.order_id, c.order_time, r.pickup_time
)
SELECT  
	no_of_pizzas,
	ROUND(AVG(prep_time_minutes), 2) AS avg_prep_time_minutes
FROM prep_time
GROUP BY no_of_pizzas
ORDER BY no_of_pizzas:
```

Output:
- `no_of_pizzas` -> Pizzas ordered at a time.
- `avg_prep_time_minutes` -> Average time in minutes to prepare the order.

- The results show a positive relationship between the number of pizzas in an order and the average preparation time.
As the pizza count increases, the time taken for the order to be prepared also increases. This trend is expected, since larger orders require more cooking and assembly effort.

| no_of_pizzas | avg_prep_time_minutes |
|--------------|-----------------------|
|            1 |                 12.36 |
|            2 |                 18.38 |
|            3 |                 29.28 |

***

#### Q4. What was the average distance travelled for each customer?

Approach:
- Joined the tables `clean_customer_orders` and `clean_runner_orders` using the shared `order_id` to connect each customer with the corresponding delivery distance.
- Filtered out rows where `distance_km` is NULL to ensure only valid distance values are used in the calculation.
- Grouped the data by each `customer_id`, so the calculation is performed individually for every customer.
- Calculated the average distance travelled per customer using `AVG(distance_km)` and `round` the result to 2 decimal places for readability.
- Sorted the output by `customer_id` to keep the results tidy and easy to interpret.

```sql
SELECT
	c.customer_id,
	ROUND(AVG(r.distance_km), 2) AS avg_distance
FROM clean_customer_orders c
JOIN clean_runner_orders r
	ON c.order_id = r.order_id
WHERE r.distance_km IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

Output:
- `customer_id` -> Unique Customer ID.
- `avg_distance` -> Average distance travelled for each customer.

- The results reveal ordering patterns—some customers tend to order from farther away, while others consistently place nearby orders.

| customer_id | avg_distance |
|-------------|--------------|
|         101 |        20.00 |
|         102 |        16.73 |
|         103 |        23.40 |
|         104 |        10.00 |
|         105 |        25.00 |

***

#### Q5. What was the difference between the longest and shortest delivery times for all orders?

***

#### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

***

#### Q7. What is the successful delivery percentage for each runner?
