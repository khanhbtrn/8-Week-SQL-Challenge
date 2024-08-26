## 💰Week 4: Data Bank

### 🔎 Business Problem

Data Bank is digital bank for banking activities as well as functions as a distributed data storage platform. Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need. This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments.
### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/efba88d8-be46-43ed-a5fa-43a2846ba74a)

### 📊 Example Datasets
#### Table 1: Regions
 - Just like popular cryptocurrency platforms - Data Bank is also run off a network of nodes where both money and data is stored across the globe. 
 - This `regions` table contains the `region_id` and their respective `region_name` values

<br>![image](https://github.com/user-attachments/assets/6de9c361-cef7-4887-8c06-828e9c84a48d)

#### Table 2: Customer Nodes
 - Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.
 - This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!
 - Below is a sample of the top 10 rows of the `data_bank.customer_nodes`

<br>![image](https://github.com/user-attachments/assets/8d47685e-6a64-4347-bd71-0de8f3a723d0)

#### Table 3: Customer Transactions
 - This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

<br>![image](https://github.com/user-attachments/assets/8f2da0af-5c59-4b49-9c84-a3516d54c985)

### 📒Case Study Questions & Solutions
### A. Customer Nodes Exploration
##### 1. How many unique nodes are there on the Data Bank system?
**Logic:**
 - Use `COUNT(DISTINCT node_id)`
```sql
SELECT 	
	COUNT(DISTINCT node_id) AS nb_of_unique_nodes
FROM data_bank.customer_nodes;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/bc430ef5-b1e1-4cd0-b079-2ee6f8aa984e)



##### 2. What is the number of nodes per region?
**Logic:**
 - Use `GROUP BY region_id` while `COUNT(DISTINCT node_id)`
```sql
SELECT
	region_name,
	COUNT(DISTINCT node_id) AS nodes_per_region
FROM data_bank.customer_nodes cn JOIN data_bank.regions r ON cn.region_id = r.region_id
GROUP BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/422898b3-6f12-4f60-94cd-2095b2f96ab5)



##### 3. How many customers are allocated to each region?
**Logic:**
 - Use `GROUP BY region_id` while `COUNT(customer_id)`
```sql
SELECT
	region_id,
	COUNT(customer_id) AS customers_per_region
FROM data_bank.customer_nodes
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/4a223295-c648-4c91-b49a-248893679474)



##### 4. How many days on average are customers reallocated to a different node?
**Logic:**
 - Use `GROUP BY region_id` while `COUNT(customer_id)`
```sql

```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/4a223295-c648-4c91-b49a-248893679474)
