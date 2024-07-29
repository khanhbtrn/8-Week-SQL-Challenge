## 📍Week 1: Danny's Diner

### 🔎 Business Problem

Danny’s Diner is facing challenges in understanding customer behavior and needs assistance in utilizing basic operational data. The goal is to analyze customer visiting patterns, spending habits, and favorite menu items to enhance customer experiences and decide on the expansion of the loyalty program. Additionally, Danny requires help in creating easy-to-inspect datasets for his team without relying on SQL.

### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/6d3920be-9636-46dd-907d-ef5b9486ece8)

### 📊 Example Datasets

#### Table 1: sales
![image](https://github.com/user-attachments/assets/5db7d811-a67a-4500-bc51-0e0de63916b8)

#### Table 2: menu
![image](https://github.com/user-attachments/assets/f9403095-32a0-408c-858d-06ebca7777ce)

#### Table 3: members
![image](https://github.com/user-attachments/assets/d409b8ad-818a-4ed2-a051-39ed52375d73)

### 📒Case Study Questions & Solutions
**1. What is the total amount each customer spent at the restaurant?**
**Logic**
Calculate the total spent by each customer by joining sales and menu tables and summing up the prices
```
SELECT
    customer_id,
    SUM(price) AS total_spent
FROM sales s JOIN menu m ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 1;```
**Output** 
![image](https://github.com/user-attachments/assets/3692f6d6-0fd1-4c09-85a4-1bef02dfa027)

