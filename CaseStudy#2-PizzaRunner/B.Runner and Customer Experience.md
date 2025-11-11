## B. Runner and Customer Experience 🌟

### Questions and Solutions

#### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT 
	FLOOR((registration_date - DATE '2021-01-01')/7) + 1 AS week_number,
	COUNT(runner_id)
FROM runners
GROUP BY week_number
ORDER BY week_number
```

***

#### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

***

#### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

***

#### Q4. What was the average distance travelled for each customer?

***

#### Q5. What was the difference between the longest and shortest delivery times for all orders?

***

#### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

***

#### Q7. What is the successful delivery percentage for each runner?
