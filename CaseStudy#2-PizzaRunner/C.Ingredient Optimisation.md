## C. Ingredient Optimisation 🍕

### Questions and Solutions

#### Q1. What are the standard ingredients for each pizza?

Approach:
- Identified that the `toppings` column in `pizza_recipes` contains comma-separated `topping IDs`, which need to be normalized before analysis.
- Split the toppings string into individual `topping IDs` using `regexp_split_to_array` and `UNNEST`.
- Cast each extracted `topping ID` to an integer to enable accurate joins.
- Created a Common Table Expression (CTE) `toppings_names` to make the transformation step clear and reusable.
- Joined the normalized `topping IDs` with `pizza_toppings` to retrieve human-readable topping names.
- Joined with `pizza_names` to associate each pizza with its corresponding toppings.
- Aggregated `topping names` back into a single, comma-separated list per pizza using `STRING_AGG`.
- Grouped results by `pizza name` to produce one row per pizza with its standard ingredients

```sql
WITH toppings_names AS
(
	SELECT
		pizza_id,
		UNNEST(regexp_split_to_array(toppings, ','))::INTEGER AS toppings_id
	FROM pizza_recipes
)
SELECT 
	p.pizza_name, 
	string_agg(s.topping_name::TEXT, ', ') AS toppings
FROM toppings_names t
JOIN pizza_names p
	ON t.pizza_id = p.pizza_id
JOIN pizza_toppings s
	ON s.topping_id = t.toppings_id
GROUP BY p.pizza_name;
```

Output:
- `pizza_name` -> Name of the Pizza
- `toppings` -> Standard ingredients for the pizza

| pizza_name |                                toppings                               |
|------------|-----------------------------------------------------------------------|
| Meatlovers | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes            |

***

#### Q2. What was the most commonly added extra?

Approach:
- Identified that the extras column in `clean_customer_orders` contains comma-separated `topping IDs`.
- Filtered out records where no extras were added to focus only on modified orders.
- Split the extras string into individual `topping IDs` using `regexp_split_to_array` and `UNNEST`.
- Cast each extracted value to an integer to allow accurate joins.
- Joined the normalized `topping IDs` with the `pizza_toppings` lookup table to retrieve topping names.
- Counted the frequency of each extra topping to determine how often it was added.
- Sorted the results in descending order and selected the top result to identify the most commonly added extra.

```sql
WITH CTE AS
(
SELECT 
	UNNEST(regexp_split_to_array(extras, ', '))::INTEGER extras
FROM clean_customer_orders
WHERE extras IS NOT NULL
)
SELECT topping_name, (COUNT(topping_name)) AS extra_count
FROM CTE e
JOIN pizza_toppings t
	ON e.extras = t.topping_id
GROUP BY topping_name
ORDER BY extra_count DESC
LIMIT 1;
```

Output:
- `topping_name` -> Most commonly added extra
- `extra_count` -> Number of times the most commonly added extra was used

| topping_name | extra_count |
|--------------|-------------|
| Bacon        |           4 |

***

#### Q3. What was the most common exclusion?

***

#### Q4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

***

#### Q5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

***

#### Q6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

***
