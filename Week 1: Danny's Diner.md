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


**4.What is the most purchased item on the menu and how many times was it purchased by all customers?**
<br> **Logic:**
Count the number of times each product is purchased and order by the count in descending order, limit to 1
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


