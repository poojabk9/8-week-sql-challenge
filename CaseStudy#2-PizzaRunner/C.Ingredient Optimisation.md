## C. Ingredient Optimisation 🍕

### Questions and Solutions

#### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

Approach:
- 

```sql

```

Output:
- 

| week_number | count_of_runners |
|-------------|------------------|
|           1 |                2 |
|           2 |                1 |
|           3 |                1 |

***

#### Q1. What are the standard ingredients for each pizza?

Approach:
- Identified that the `toppings` column in `pizza_recipes` contains comma-separated topping IDs, which need to be normalized before analysis.
- Split the toppings string into individual topping IDs using `regexp_split_to_array` and `UNNEST`.
- Cast each extracted topping ID to an integer to enable accurate joins.
- Created a Common Table Expression (CTE) to make the transformation step clear and reusable.
- Joined the normalized topping IDs with `pizza_toppings` to retrieve human-readable topping names.
- Joined with `pizza_names` to associate each pizza with its corresponding toppings.
- Aggregated topping names back into a single, comma-separated list per pizza using `STRING_AGG`.
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
