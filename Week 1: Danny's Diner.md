## 📍Week 1: Danny's Diner

### 🔎 Business Problem

Danny’s Diner is facing challenges in understanding customer behavior and needs assistance in utilizing basic operational data. The goal is to analyze customer visiting patterns, spending habits, and favorite menu items to enhance customer experiences and decide on the expansion of the loyalty program. Additionally, Danny requires help in creating easy-to-inspect datasets for his team without relying on SQL.

### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/788c9c23-bc08-4e8b-aad1-d125bb759bd6)

### 📊 Example Datasets

#### Table 1: sales
![image](https://github.com/user-attachments/assets/5db7d811-a67a-4500-bc51-0e0de63916b8)

#### Table 2: menu
![image](https://github.com/user-attachments/assets/f9403095-32a0-408c-858d-06ebca7777ce)

#### Table 3: members
![image](https://github.com/user-attachments/assets/d409b8ad-818a-4ed2-a051-39ed52375d73)

### 📒Case Study Questions & Solutions
**1. What is the total amount each customer spent at the restaurant?**
<br> **Logic:**
<br> Calculate the total spent by each customer by joining sales and menu tables and summing up the prices
```sql
SELECT
    customer_id,
    SUM(price) AS total_spent
FROM sales s JOIN menu m ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 1;
```
**Output:** 

<br> ![image](https://github.com/user-attachments/assets/3692f6d6-0fd1-4c09-85a4-1bef02dfa027)


**2. How many days has each customer visited the restaurant?**
<br> **Logic:**
<br> Count the distinct order dates for each customer to get the number of visit days
```sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/b8bbc100-5e95-4949-a199-2d8823777ca5)



**3. What was the first item from the menu purchased by each customer?**
<br> **Logic:**
<br> Use DENSE_RANK to rank order dates for each customer and select the first item
```sql
WITH first_item AS(
    SELECT
        customer_id,
        order_date,
        DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS order_of_item,
        product_name
    FROM sales s JOIN menu m ON s.product_id = m.product_id)

SELECT
    customer_id,
    product_name
FROM first_item
WHERE order_of_item = 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/fa20ddc0-1aae-407c-8c7a-a42de088de4b)


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
<br> **Logic:**
<br> Count the number of times each product is purchased and order by the count in descending order, limit to 1
```sql
SELECT
    product_name,
    COUNT(s.product_id) AS units_sold
FROM sales s JOIN menu m ON s.product_id = m.product_id
GROUP BY 1 
ORDER BY COUNT(s.product_id) DESC
LIMIT 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/46cb5021-1825-4e64-8830-78938269087e)


**5. Which item was the most popular for each customer?**
<br> **Logic:**
<br> Find the most ordered item for each customer by counting orders and ranking them
```sql
WITH popular_item AS(SELECT
    customer_id,
    product_name,
    COUNT(order_date) AS nb_of_orders,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(order_date) DESC) AS rnk
FROM sales s LEFT JOIN menu m ON s.product_id = m.product_id
GROUP BY 1,2)

SELECT
    customer_id,
    product_name AS most_popular_items
FROM popular_item
WHERE rnk = 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/4127dd84-412f-4347-a07c-11987702f392)


**6. Which item was purchased first by the customer after they became a member?**
<br> **Logic:**
<br> To identify the first item purchased by each customer after they became a member, we need to consider only the order dates that are later than the join date (order_date > join_date). Since a customer may have multiple orders after their join date, we can use the ROW_NUMBER() window function to rank these orders and select the first one
```sql
WITH order_time AS(SELECT
    s.customer_id,
    m.product_name,
    s.order_date,
    ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS rnk
FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
    AND S.order_date > mem.join_date)

SELECT
    customer_id,
    product_name
FROM order_time
WHERE rnk = 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/66a7d20f-d4d5-479d-85a7-e46b2d8f6421)


**7. Which item was purchased just before the customer became a member?**
<br> **Logic:**
<br> Retrieve the last item purchased before the join date using ROW_NUMBER to rank order dates in descending order
```sql
WITH order_time AS(SELECT
    s.customer_id,
    m.product_name,
    s.order_date,
    ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS rnk
FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
    AND S.order_date > mem.join_date)

SELECT
    customer_id,
    product_name
FROM order_time
WHERE rnk = 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/7f42a5d5-da63-4e1e-8232-fb911421e4ab)


**8. What is the total items and amount spent for each member before they became a member?**
<br> **Logic:**
<br> Sum the price and count the items ordered before the join date for each member
```sql
SELECT
    mem.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(price) AS amount_spent
FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    JOIN members mem ON s.customer_id = mem.customer_id
    AND order_date < join_date
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/071941c3-f314-40ca-b5d3-18ca60023f52)


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier how many points would each customer have?**
<br> **Logic:**
<br> Calculate points for each order considering sushi has a 2x multiplier
```sql
WITH points_system AS(SELECT
    s.customer_id,
    s.product_id,
    m.product_name,
    CASE WHEN product_name = 'sushi' THEN price*20
    ELSE price*10 END AS points
FROM sales s 
    JOIN menu m ON s.product_id = m.product_id)

SELECT
    customer_id,
    SUM(points) AS total_points
FROM points_system
GROUP BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/8ee7d737-ba6d-4b72-a800-54c800c0ec72)

