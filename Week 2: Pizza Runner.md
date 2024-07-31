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
A quick glance at the datasets shows that there are null and NaN values in Table 2: `customer_orders` and Table 3: `runner_orders`.
<br> There is also data standardization issue in columns `distance` and `duration` in Table 3.
<br> Therefore, we will first clean our data through creating new tables before writing queries.

<br>**Table 2: customer_orders**
<br>This statement updates the `exclusions` and `extras` columns directly in the `customer_orders` table by replacing NULL and 'null' values with empty strings.
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
<br>There are four columns that need to be cleaned:
<br>• `pickup_time`: replace null values with empty strings 
<br>• `distance`: replace null values with empty strings and `TRIM` the unit `km`
<br>• `duration`: similar to `distance`, replace null values with empty strings and `TRIM` the units `mins`, `minutes`, `minute`
<br>• `cancellation`: replace null values with empty strings
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
**Output**
<br>![image](https://github.com/user-attachments/assets/5d313ea4-34dd-4851-aad5-0d5d0a25614f)

### 📒Case Study Questions & Solutions
### A. Pizza Metrics
##### 1. How many pizzas were ordered?
**Logic**
<br> This answer is straightforward. We just need to `COUNT` all in `customer_orders` table
```sql
SELECT 
	COUNT(*) AS nb_pizza_ordered
FROM customer_orders;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/770b9dd0-5159-431d-81d0-8a145d5af72f)



##### 2. How many unique customer orders were made?
