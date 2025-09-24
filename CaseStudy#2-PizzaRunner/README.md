<img src="https://8weeksqlchallenge.com/images/case-study-designs/2.png" alt=Image width=500 height=520>

## Overview
- [Problem Statement](#problem-statement)
- [Database Schema](#database-schema)
- [Data Cleaning and Transformation](#data-cleaning-and-transformation)
- [Questions and Solutions](#questions-and-solutions)

### Problem Statement

Pizza Runner is Danny’s innovative food delivery startup that combines retro pizza styling with an Uber-like delivery model. To scale and optimize operations, Danny needs help cleaning messy order and delivery data, and generating insights from it. The challenge involves working with datasets from the pizza_runner schema to answer real-world business questions, such as tracking orders, analyzing runner performance, and improving operational efficiency.

***

### Database Schema
Pizza Runner contains 6 tables:

- Table 1 - runner
- Table 2 - customer_orders
- Table 3 - runner_orders
- Table 4 - pizza_names
- Table 5 - pizza_recipes
- Table 6 - pizza_toppings

#### Entity Relationship Diagram:

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

***

### Data Cleaning and Transformation

#### Cleaning Table: customer_orders

Approach:
- Standardized the `exclusions` and `extras` columns by replacing inconsistent values ('null', '', and NULL) with proper SQL `NULL`.
- Preserved valid topping values to ensure they can be used for later analysis.
- Created a new cleaned table `clean_customer_orders` to store the standardized data.

```sql
CREATE TABLE clean_customer_orders AS
SELECT 
	order_id,
	customer_id,
	pizza_id,
	CASE
		WHEN exclusions is NULL OR exclusions = '' OR exclusions = 'null' THEN NULL
		ELSE exclusions
	END AS exclusions,
	CASE 
		WHEN extras is NULL OR extras = '' OR extras = 'null' THEN NULL
		ELSE extras
	END AS extras,
	order_time
FROM customer_orders

SELECT *
FROM clean_customer_orders
```

clean_customer_orders table:

| order_id | customer_id | pizza_id | exclusions | extras |      order_time      |
|----------|-------------|----------|------------|--------|----------------------|
|         1|        	101|	       1| NULL	     | NULL  	| 2020-01-01 18:05:02  |
|         2|        	101|	       1| NULL	     | NULL   |	2020-01-01 19:00:52  |
|         3|        	102|       	 1|	NULL	     | NULL	  | 2020-01-02 23:51:23  |
|         3|        	102|         2|	NULL	     | NULL	  | 2020-01-02 23:51:23  |
|         4|        	103|       	 1|	4	         | NULL	  | 2020-01-04 13:23:46  |
|         4|        	103|         1|	4          | NULL	  | 2020-01-04 13:23:46  |
|         4|        	103|         2|	4	         | NULL   |	2020-01-04 13:23:46  |
|         5|         	104|         1|	NULL	     | 1	    | 2020-01-08 21:00:29  |
|         6|        	101|         2|	NULL	     | NULL	  | 2020-01-08 21:03:13  |
|         7|        	105|         2|	NULL	     | 1	    | 2020-01-08 21:20:29  |
|         8|        	102|       	 1|	NULL	     | NULL	  | 2020-01-09 23:54:33  |
|         9|        	103|         1|	4	         | 1, 5	  | 2020-01-10 11:22:59  |
|        10|        	104|         1|	NULL	     | NULL	  | 2020-01-11 18:34:49  |
|        10|        	104|         1|	2, 6	     | 1, 4	  | 2020-01-11 18:34:49  |

***

### Questions and Solutions

