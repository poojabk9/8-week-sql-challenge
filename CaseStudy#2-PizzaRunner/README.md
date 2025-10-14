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


#### Cleaning Table: customer_orders

Approach:
- Created a `CTE` to clean the `runner_orders` table by standardizing missing and inconsistent values.
- Replaced blank strings and 'null' text entries with proper SQL NULL values across all columns.
- Removed unit labels like 'km', 'minutes', 'minute', and 'mins' from the `distance` and `duration` columns using `REPLACE` and `REGEXP_REPLACE` functions for consistency.
- Trimmed any extra spaces to ensure clean and uniform data entries.
- Converted data types in the final `SELECT`:
  	- `pickup_time` → TIMESTAMP
  	- `distance_km` → DECIMAL
    - `duration_min` → INTEGER
- Created a new cleaned table named `clean_runner_orders` to store the processed and standardized data for further analysis.

```sql
CREATE TABLE clean_runner_orders AS
WITH CTE AS 
( 
	SELECT order_id, 
		   runner_id, 
		   CASE -- CASE statement to replace Blank space and null entries to 'NULL' in pickup_time column 
		   		WHEN pickup_time is NULL OR pickup_time = '' OR pickup_time = 'null' THEN NULL 
		   		ELSE pickup_time 
		   END, 
		   CASE -- CASE statement to replace Blank space and null entries to 'NULL' and removing the unit in distance column 
		   		WHEN distance is NULL OR distance = '' OR distance = 'null' THEN NULL 
		   		ELSE TRIM(REPLACE(distance, 'km', '')) 
		   END distance_km, 
		   CASE -- CASE statement to replace Blank space and null entries to 'NULL' and removing the unit in duration column 
		   		WHEN duration is NULL OR duration = '' OR duration = 'null' THEN NULL 
				ELSE TRIM(REGEXP_REPLACE(duration, '(minutes|minute|mins)', '', 'gi')) 
			END duration_min, 
			CASE -- CASE statement to replace Blank space and null entries to 'NULL' in cancellation column 
				WHEN cancellation is NULL OR cancellation = '' OR cancellation = 'null' THEN NULL 
				ELSE cancellation 
			END 
	FROM runner_orders
) 
SELECT 
	order_id, 
	runner_id, 
	-- Standardising the datatype of pickup_time, distance_km, and duration_min columns 
	CAST(pickup_time AS TIMESTAMP) AS pickup_time, 
	CAST(distance_km AS DECIMAL) AS distance_km, 
	CAST(duration_min AS INTEGER) AS duration_min, 
	cancellation 
FROM CTE;
```

clean_runner_orders:
| order_id | runner_id |     pickup_time     | distance_km | duration_min |       cancellation      |
|----------|-----------|---------------------|-------------|--------------|-------------------------|
|        1 |		 1 | 2020-01-01 18:15:34 |	        20 |	       32 | NULL                    |
|        2 |         1 | 2020-01-01 19:10:54 |	        20 |	       27 | NULL                    |
|        3 |	     1 | 2020-01-03 00:12:37 |	      13.4 |	       20 | NULL                    |
|        4 |       	 2 | 2020-01-04 13:53:03 |	      23.4 |           40 | NULL                    |
|        5 |         3 | 2020-01-08 21:10:57 |	        10 |	       15 | NULL                    |
|        6 |	     3 | NULL                |        NULL |         NULL |	Restaurant Cancellation |
|        7 |       	 2 | 2020-01-08 21:30:45 |	        25 |           25 | NULL                    |
|        8 |	     2 | 2020-01-10 00:15:02 |	      23.4 |	       15 | NULL                    |
|        9 |	     2 | NULL                |        NULL |         NULL |	Customer Cancellation   |
|       10 |         1 | 2020-01-11 18:50:20 |          10 |	       10 |	NULL                    | 

***


### Questions and Solutions



