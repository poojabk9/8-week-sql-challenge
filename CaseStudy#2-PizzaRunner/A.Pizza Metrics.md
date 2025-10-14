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
- `no_of_pizza` -> Count of `order_id` which gives the total number of Pizza's ordered. 

| no_of_pizza |
|-------------|
|          14	|	

#### Q2. How many unique customer orders were made?
