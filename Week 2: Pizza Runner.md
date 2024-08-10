## 🍕Week 2: Pizza Runner

### 🔎 Business Problem

Danny launched Pizza Runner, a pizza delivery service combining 80s retro styling and a modern Uber-like approach, by recruiting delivery runners and developing a mobile app to expand his new Pizza Empire from his home.

### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/3376cd47-9f55-4147-b99e-1bf02024aed3)

### 📊 Example Datasets

#### Table 1: runners
![image](https://github.com/user-attachments/assets/21733b0b-d99b-4a76-814c-4d95c6f3545c)

#### Table 2: customer_orders
![image](https://github.com/user-attachments/assets/a497351b-19f5-4ddd-a4b3-82856ebcf154)

#### Table 3: runner_orders
![image](https://github.com/user-attachments/assets/f7cd7de6-b330-4f20-bd83-2e79779c3bad)

#### Table 4: pizza_names
![image](https://github.com/user-attachments/assets/cf86cb22-7c6a-45d8-83fd-c061e9415df6)

#### Table 5: pizza_recipes
![image](https://github.com/user-attachments/assets/c43d40e5-c842-4fdc-b09c-0689671c1395)

#### Table 6: pizza_toppings
![image](https://github.com/user-attachments/assets/65477dbe-b23d-41aa-b1af-0904369e1f74)

### 📈Data Cleaning
 - A quick glance at the datasets shows that there are null and NaN values in Table 2: `customer_orders` and Table 3: `runner_orders`.
 - There is also data standardization issue in columns `distance` and `duration` in Table 3.
 - Therefore, we will first clean our data through creating new tables before writing queries.

<br>**Table 2: customer_orders**
 - This statement updates the `exclusions` and `extras` columns directly in the `customer_orders` table by replacing NULL and 'null' values with empty strings.
```sql
UPDATE pizza_runner.customer_orders
SET exclusions = CASE
                    WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
                    ELSE exclusions
                 END,
    extras = CASE
                WHEN extras IS NULL OR extras LIKE 'null' THEN ''
                ELSE extras
             END;
```
**Output**
<br>![image](https://github.com/user-attachments/assets/73212555-df46-44f9-9b1a-7fcce200a7d8)

<br>**Table 3: runner_orders**
 - There are four columns that need to be cleaned:
    - `pickup_time`: replace null values with empty strings
    - `distance`: replace null values with empty strings and `TRIM` the unit `km`
    - `duration`: similar to `distance`, replace null values with empty strings and `TRIM` the units `mins`, `minutes`, `minute`
    - `cancellation`: replace null values with empty strings
```sql
UPDATE pizza_runner.runner_orders
SET pickup_time = CASE
                    WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ''
                    ELSE pickup_time
                 END,
    distance = CASE
                WHEN distance IS NULL OR distance LIKE 'null' THEN ''
                WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
                ELSE distance
             END,
    duration = CASE
    			WHEN duration IS NULL OR duration LIKE 'null' THEN ''
    			WHEN duration LIKE '%mins'  THEN TRIM('mins' FROM duration)
    			WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
    			WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    			ELSE duration
    		END,
    cancellation = CASE
    				WHEN cancellation IS NULL OR cancellation LIKE 'null' OR cancellation LIKE 'NaN' THEN ''
    				ELSE cancellation
    		END;
```
 - We also need to standardize the data types:
```sql
ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN pickup_time TYPE TIMESTAMP USING NULLIF(pickup_time, '')::timestamp,
ALTER COLUMN distance TYPE REAL USING NULLIF(distance, '')::real,
ALTER COLUMN duration TYPE INTEGER USING NULLIF(duration, '')::integer;
```
**Output**
<br>![image](https://github.com/user-attachments/assets/5d313ea4-34dd-4851-aad5-0d5d0a25614f)

### 📒Case Study Questions & Solutions
### A. Pizza Metrics
##### 1. How many pizzas were ordered?
**Logic:**
 - This answer is straightforward. We just need to `COUNT` all in `customer_orders` table
```sql
SELECT 
	COUNT(*) AS nb_pizza_ordered
FROM pizza_runner.customer_orders;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/89c4c8d4-97a7-4613-8cbd-2a17dcb7e9f5)



##### 2. How many unique customer orders were made?
**Logic:**
 - Similar to the previous solution
```sql
SELECT
	COUNT(DISTINCT order_id) AS unique_orders
FROM pizza_runner.customer_orders;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/e6c3e4fb-42c8-4524-80c6-93f0979dd7e6)



##### 3. How many successful orders were delivered by each runner?
**Logic:**
 - Since there is no clear value of whether an order was successfully ordered or not, we will intepret this as `cancellation` has null value, which means the order was not cancelled
```sql
SELECT 
	runner_id,
	COUNT(*) AS nb_successful_orders
FROM pizza_runner.runner_orders
WHERE cancellation LIKE ''
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/9ab96728-b995-4e24-96ce-f9987f82e562)



##### 4. How many of each type of pizza was delivered?
**Logic:**
 - We know that there are two types of pizza: Meat Lovers and Vegetarian. And since this information is not included in `runner_orders` table but there is no relation between `runner_orders` table and `pizza_name` table, we will have to join these three tables together to get the names of the pizza type.
 - Based on the solution for the previous question to find the number of pizzas successfully delivered.
```sql
SELECT
	name.pizza_name,
	COUNT(*) AS type_delivered
FROM pizza_runner.pizza_names name 
	JOIN pizza_runner.customer_orders co ON name.pizza_id = co.pizza_id
	JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE cancellation LIKE ''
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/23c529c2-2037-4854-a8c1-9714c8790cfe)



##### 5. How many Vegetarian and Meatlovers were ordered by each customer?
**Logic:**
 - "Vegetarian and Meatlovers" and "ordered" indicate we need to join `customer_orders` table and `pizza_names` table together
```sql
SELECT 
	co.customer_id,
	name.pizza_name,
	COUNT(*) AS nb_ordered
FROM pizza_runner.pizza_names name 
	JOIN pizza_runner.customer_orders co ON name.pizza_id = co.pizza_id
GROUP BY 1,2
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/bc8eb588-7ea2-4df9-8c69-b29dec1e30f2)




##### 6. What was the maximum number of pizzas delivered in a single order?
**Logic:**
 - We need a table counting the number of pizzas delivered for all orders and then select the maximum value from there
```sql
WITH ranking_table AS(
	SELECT 
		co.order_id,
		COUNT(*) AS pizza_delivered
	FROM pizza_runner.customer_orders co
	JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
	WHERE cancellation LIKE ''
	GROUP BY 1
)

SELECT
	MAX(pizza_delivered) AS max_pizza_delivered
FROM ranking_table;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/630c3fe7-caea-47de-bc59-942f1fb51ea4)
<br> Therefore, the maximum number of pizzas delivered in a single order is 3 pizzas



##### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
**Logic:**
 -  Pizzas had at least 1 change means either `exclusions` or `extras` does not have empty values, and in reverse, no change means both columns have empty values. <br> And another condition is "delivered", we need to join `customer_orders` table with 'runner_orders' table
```sql
SELECT 
	co.customer_id,
	SUM(CASE WHEN exclusions NOT LIKE '' OR extras NOT LIKE '' THEN 1 ELSE 0 END) AS at_least_one_change,
	ASUM(CASE WHEN exclusions = '' AND extras = '' THEN 1 ELSE 0 END) AS no_change
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE cancellation LIKE ''
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/b2e9170f-622a-44e8-b6cb-02aba9f43df1)



##### 8. How many pizzas were delivered that had both exclusions and extras?
**Logic:**
 -  Similar to the solution above, we just need to change from `OR` to `AND`
```sql
SELECT 
	SUM(CASE WHEN exclusions NOT LIKE '' AND extras NOT LIKE '' THEN 1 ELSE 0 END) AS both_exclusions_extras
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE cancellation LIKE '';
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/b2e9170f-622a-44e8-b6cb-02aba9f43df1)



##### 9. What was the total volume of pizzas ordered for each hour of the day?
**Logic:**
 -   Use `DATE_PART()` to extract each hour of the day from `order_time` column in `customer_orders` table
```sql
SELECT 
	DATE_PART('hour', order_time) AS hour_of_the_day,
	COUNT(order_id) AS pizza_volume
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/d661f407-4051-4db4-b331-1626124de882)



##### 10. What was the volume of orders for each day of the week?
**Logic:**
 - Similar to the solution in the previous question, but here I used `TO_CHAR(order_time, 'Day')` to display the day of the week as a string instead of a number.
 - A quick explanation for `TO_CHAR(order_time + INTERVAL '2 day', 'FMDay') AS day_of_the_week`:
	 - Original DATE_PART('dow', ...) Output:
	    - Sunday: 0
	    - Monday: 1
	    - Tuesday: 2
	    - Wednesday: 3
	    - Thursday: 4
	    - Friday: 5
	    - Saturday: 6
	 - Shift by 2 Days:
	    - Sunday + 2 days = Tuesday (2)
	    - Monday + 2 days = Wednesday (3)
	    - Tuesday + 2 days = Thursday (4)
	    - Wednesday + 2 days = Friday (5)
	    - Thursday + 2 days = Saturday (6)
	    - Friday + 2 days = Sunday (0)
	    - Saturday + 2 days = Monday (1)
```sql
SELECT 
	TO_CHAR(order_time + INTERVAL '2 day', 'FMDay') AS day_of_the_week, -- This is to shift the starting day of the week to Monday
	COUNT(order_id) AS order_volume
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/f7073c58-c6b2-4881-ab70-5d3b3ae6af75)

### B. Runner and Customer Experience
##### 1. How many runners signed up for each 1 week period? (i.e. week starts `2021-01-01`)
**Logic:**
 - Use `DATE_TRUNC` to truncates the `registration_date` to the start of the week and casts it to a date type.
```sql
SELECT
	DATE_TRUNC('week', registration_date) AS week_start,
	COUNT(runner_id) AS nb_runners
FROM pizza_runner.runners
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/14c6709d-1544-4669-b4a7-421df34373b6)



##### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?
**Logic:**
 - The time it took for each runner to arrive at the HQ to pick up the order equals `pickup_time` - `order_time`.
 - We will first need to create a CTE of pickup time in minutes for each order and that order was taken by what runner, then we calculate the average value.

```sql
WITH time_table AS(
	SELECT
		co.order_id,
		ro.runner_id,
		co.order_time,
		ro.pickup_time,
	    AVG(EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60) AS pickup_minutes
	FROM pizza_runner.customer_orders co 
		JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
	WHERE co.order_time IS NOT NULL 
		AND cancellation LIKE ''
	GROUP BY 1,2,3,4
)

SELECT 
	runner_id,
	AVG(pickup_minutes) AS avg_pickup_minutes
FROM time_table
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/502b78bb-5fe8-4569-9673-f420087ce393)



##### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
**Logic:**
 - To determine whether there is a relationship between the number of pizzas and how long the order takes to prepare, we need to calculate the time it took for each order to prepare, which is the difference between `order_time` and `pickup_time`, assumming that the pizzas were ready to be delivered once the runner came to pick them up.
 - We also need to count the number of pizzas ordered for each order. 
```sql
WITH prep_time AS(
	SELECT 
		co.order_id,
		COUNT(pizza_id) AS nb_pizza,
		co.order_time,
		ro.pickup_time,
		EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60 AS pickup_minutes
	FROM pizza_runner.customer_orders co 
			JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
		WHERE co.order_time IS NOT NULL 
			AND cancellation LIKE ''
		GROUP BY 1,3,4
	)

SELECT 
	nb_pizza,
	AVG(pickup_minutes) AS avg_prep_time
FROM prep_time
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/f8896254-b614-4b26-870a-ce667f25d51f)
 - On average, it takes approximately 12 minutes to prepare one pizza, 18 minutes to prepare two pizzas, and 29 minutes to prepare three pizzas. This implies that preparing two pizzas in one order takes only 9 minutes per pizza, showcasing the highest efficiency rate when making two pizzas in a single order.



##### 4. What was the average distance travelled for each customer?
**Logic:**
 - This is a quite straightforward question, assumming that the distance travelled is calculated from HQ to customer's place.
```sql
SELECT 
	co.customer_id,
	AVG(ro.distance) AS avg_distance
FROM pizza_runner.customer_orders co 
	JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
WHERE cancellation LIKE ''
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/0617a9da-c598-475a-bf65-95cbbd42316f)



##### 5. What was the difference between the longest and shortest delivery times for all orders?
**Logic:**
 - Use `MAX` and `MIN`.
```sql
SELECT 
	MAX(duration) - MIN(duration) AS difference_in_minutes
FROM pizza_runner.runner_orders 
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/83d5a5bd-5480-4915-b026-fc6654408f63)



##### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
**Logic:**
 - To provide a fair evaluation, we shouldn't only consider the average speed of each runner for each delivery. We must also take into account the number of pizzas they had to deliver, as this could affect their delivery speed. I use `distance*60/duration` to calculate their speed.
```sql
	SELECT 
		ro.runner_id,
		ro.order_id,
		distance,
		duration,
		COUNT(pizza_id) AS pizza_count,
		distance*60/duration AS speed_in_km,
		AVG(distance*60/duration) OVER(PARTITION BY runner_id) AS avg_speed_in_km
	FROM pizza_runner.customer_orders co 
		JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id
	WHERE cancellation LIKE ''
	GROUP BY 1,2,3,4
	ORDER BY 1,6 DESC,7 DESC;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/afe30b6a-de1a-4653-845f-6679bd383b0c)
<br> From the output, we can draw the following insights:
 - The runner with the highest *average* speed per delivery was Runner 2, with an average speed of approximately 63 km/h. The slowest runner was Runner 3, with an *average* speed of 40 km/h.
 - The fastest delivery speed was achieved by Runner 2 for order 8, with a speed of roughly 94 km/h. The slowest delivery speed was by Runner 3 for order 4, with a speed of 35 km/h.



##### 7. What is the successful delivery percentage for each runner?
**Logic:**
 - Use conditional aggregation `SUM(CASE WHEN...)` to calculate the number of successful delivery.
```sql
SELECT 
	runner_id,
	ROUND(100 * SUM(CASE WHEN cancellation LIKE '' THEN 1 ELSE 0 END)/COUNT(*),0) AS percentage
FROM pizza_runner.runner_orders
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/723c9fcd-27a9-4e6b-b14c-9d2bec350f76)

### C. Ingredient Optimisation
##### 1. What are the standard ingredients for each pizza?
**Logic:**
 - Use Regex `REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER` to extract topping_id and `STRING_AGG(pt.topping_name, ', ')` to join the ingredients together.
```sql
WITH toppings_table AS(
	SELECT
		pizza_id, 
		REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
	FROM pizza_runner.pizza_recipes
)
SELECT
	pn.pizza_name,
	STRING_AGG(pt.topping_name, ', ') AS standard_ingredients
FROM pizza_runner.pizza_names pn
	JOIN toppings_table tt ON pn.pizza_id = tt.pizza_id 
	JOIN pizza_runner.pizza_toppings pt ON tt.topping_id = pt.topping_id 
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/0385e1c4-f1e6-431a-b405-153229c3ad2c)



##### 2. What was the most commonly added extra?
**Logic:**
 - First, we need to create a table that lists orders with their extras, where each extra is separated into different rows.
 - Next, we count the occurrences of each extra, order descending and limit our result by 1. 
```sql
WITH order_with_extras AS(
	SELECT
		order_id,
		CAST(UNNEST(string_to_array(extras, ', ')) AS INTEGER)  AS separated_extras
	FROM pizza_runner.customer_orders
	GROUP BY 1, extras
),
toppings_table AS(
	SELECT
		pizza_id, 
		REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
	FROM pizza_runner.pizza_recipes
)
SELECT 
	tt.topping_id,
	COUNT(separated_extras) AS extras_count,
	pt.topping_name
FROM order_with_extras owe 
	JOIN toppings_table tt ON owe.separated_extras = tt.topping_id
	JOIN pizza_runner.pizza_toppings pt ON tt.topping_id = pt.topping_id 
GROUP BY 1, 3
ORDER BY 2 DESC
LIMIT 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/600cddb1-d61a-42e6-8920-3c6295162a87)



##### 3. What was the most common exclusion?
**Logic:**
 - Similar to the logic of the previous question, we just need to change extras with exclusions
```sql
WITH order_with_exclusions AS(
	SELECT
		order_id,
		CAST(UNNEST(string_to_array(exclusions, ', ')) AS INTEGER)  AS separated_exclusions
	FROM pizza_runner.customer_orders
	GROUP BY 1, exclusions
),
toppings_table AS(
	SELECT
		pizza_id, 
		REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER AS topping_id
	FROM pizza_runner.pizza_recipes
)
SELECT 
	tt.topping_id,
	COUNT(separated_exclusions) AS exclusions_count,
	pt.topping_name
FROM order_with_exclusions owe 
	JOIN toppings_table tt ON owe.separated_exclusions = tt.topping_id
	JOIN pizza_runner.pizza_toppings pt ON tt.topping_id = pt.topping_id 
GROUP BY 1, 3
ORDER BY 2 DESC
LIMIT 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/33a4a137-df3f-4bd9-8fcb-1364b30d4924)

### D. Pricing and Ratings
##### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
**Logic:**
 - We have to be careful that these orders were delivered successfully, or `WHERE cancellation LIKE ''`
```sql
WITH price_cte AS(
	SELECT
		co.order_id,
		co.pizza_id,
		pn.pizza_name,
		CASE
			WHEN pizza_name = 'Meatlovers' THEN 12
			ELSE 10
		END AS price
	FROM 
		pizza_runner.customer_orders co 
			LEFT JOIN pizza_runner.pizza_names pn ON co.pizza_id = pn.pizza_id
			JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id 
	WHERE cancellation LIKE ''
)
SELECT 
	SUM(price) AS total_sales
FROM price_cte
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/b94c7ce5-6927-4606-8d5e-7fdd55095991)



##### 2. What if there was an additional $1 charge for any pizza extras?
 - Add cheese is $1 extra
**Logic:**
 - We need to count the number of extra multiplied by $1 and then calculate the price accordingly
```sql
WITH price_cte AS
	(
	SELECT
		co.order_id,
		co.pizza_id,
	    CASE
	        WHEN extras IS NULL THEN 0
	        ELSE array_length(string_to_array(extras, ','), 1*1)
	    END AS extra_count,
	    pn.pizza_name,
	    CASE
				WHEN pizza_name = 'Meatlovers' THEN 12
				ELSE 10
			END AS price
	FROM 
		pizza_runner.customer_orders co 
			LEFT JOIN pizza_runner.pizza_names pn ON co.pizza_id = pn.pizza_id
			JOIN pizza_runner.runner_orders ro ON co.order_id = ro.order_id 
	WHERE cancellation LIKE ''
	)

SELECT 
	SUM(price) + SUM(extra_count) AS total_sales
FROM price_cte 
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/75fe365a-a2c0-4ecb-8e7c-8c2ba2c09485)


### Business Insights 
 - Meat Lovers pizzas are more popular than Vegetarian pizzas based on the data.
 - Preparing two pizzas in a single order is the most efficient, taking only 9 minutes per pizza on average. This demonstrates a higher efficiency rate when handling orders with two pizzas.
- Runner 1 has the highest successful delivery rate and is also one of the fastest runners. This suggests that Runner 1 is both reliable and efficient in delivering orders.
- The most popular times for pizza orders are between 11:00-13:00 (lunch) and 21:00-23:00 (late-night snacks). Additionally, Friday and Monday are the busiest days for orders.
### Recommendations
 - **Ingredient Management:** Prioritize stocking ingredients for Meat Lovers pizzas to ensure consistent supply and reduce waste. Consider adjusting the inventory levels of vegetarian ingredients accordingly. Explore opportunities to introduce new meat-based pizza varieties to cater to customer preferences. 
 - **Kitchen Performance:** Consider offering promotions that encourage customers to order two pizzas at a time, leveraging the efficiency in preparation to handle increased demand without overburdening the kitchen.
 - **Runner Performance:** Offer a performance-based bonus to Runner 1 to reward their reliability and speed. This could serve as motivation for other runners to improve their performance as well.
 - **Peak Hours:** Develop targeted marketing campaigns for these peak times, such as lunch deals for the midday crowd and late-night specials. Consider offering discounts or bundle deals on Fridays and Mondays to capitalize on the high order volume during these days.
