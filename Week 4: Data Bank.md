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
 - We need to create two CTEs: the first one to calculate the number of days in node and the second one to calculate the total days in node for each customer.
 - Finally, we will calculate the `AVG().` 
```sql
WITH node_change_duration AS(
	SELECT
		customer_id,
		node_id,
		end_date - start_date AS node_duration
	FROM data_bank.customer_nodes
	WHERE end_date != '9999-12-31'
	GROUP BY 1, 2, start_date, end_date
	),
total_node_days AS(
	SELECT 
		customer_id,
		node_id,
		SUM(node_duration) AS total_days
	FROM node_change_duration
	GROUP BY 1, 2
	)

SELECT
	ROUND(AVG(total_days)) AS avg_node_reallocation_days
FROM total_node_days;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/06fa9c11-0e80-46f6-9052-c60e1b3c0a35)



### B. Customer Transactions
##### 1. What is the unique count and total amount for each transaction type?
**Logic:**
 - Use `COUNT(customer_id)` to calculate unique count and `SUM(txn_amount)` to calculate the total amount then `GROUP BY` transaction type `txn_type`.
```sql
SELECT
	txn_type,
	COUNT(customer_id) AS unique_count,
	SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/b5be31b4-7642-4b63-9501-d58857884f6c)



##### 2. What is the average total historical deposit counts and amounts for all customers?
**Logic:**
 - Create a CTE to calculate the number of transactions per customer and their average amount.
 - Then calculate the average values across customers. 
```sql
WITH deposit_cte AS(
	SELECT 
		customer_id,
		COUNT(customer_id) AS transaction_count,
		AVG(txn_amount) AS avg_deposit_amount
	FROM data_bank.customer_transactions
	WHERE txn_type = 'deposit'
	GROUP BY 1)

SELECT
	ROUND(AVG(transaction_count)) AS avg_deposit_count,
	ROUND(AVG(avg_deposit_amount)) AS avg_deposit_amount
FROM deposit_cte;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/4f051370-d27a-4f20-892f-a22c727fc5f6)



##### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
**Logic:**
 - Create a CTE to calculate deposit count, purchase count, and withdraw count for each customer in each month.
 - `WHERE deposit_count >= 1 AND (purchase_count >= 1 OR withdrawal_count >= 1)` to satisfy the conditions required.
```sql
WITH monthly_transactions AS(
	SELECT 
		customer_id,
		TO_CHAR(txn_date, 'Month') AS mth,
		SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
	    SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
	    SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
	FROM data_bank.customer_transactions
	GROUP BY 1, 2)

SELECT 
	mth,
	COUNT(DISTINCT customer_id) AS nb_of_customers
FROM monthly_transactions
WHERE deposit_count >= 1 AND (purchase_count >= 1 OR withdrawal_count >= 1)
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/6449c940-14ce-4fc1-9010-daeb8361ac9c)



##### 4. What is the closing balance for each customer at the end of the month?
**Logic:**
- The goal is to calculate the closing balance for each customer at the end of each month based on their transaction history.
- We create `monthly_transactions` CTE to calculate the net transaction amount for each customer for every month and `running_balance` CTE to compute the cumulative balance for each customer up to the end of each month.
```sql
WITH monthly_transactions AS (
    SELECT 
        customer_id,
        DATE_TRUNC('month', txn_date) AS month,
        SUM(CASE 
            WHEN txn_type = 'deposit' THEN txn_amount 
            ELSE -txn_amount 
        END) AS monthly_net_amount
    FROM data_bank.customer_transactions
    GROUP BY 1, 2
),
running_balance AS (
    SELECT
        customer_id,
        month,
        SUM(monthly_net_amount) OVER (
            PARTITION BY customer_id 
            ORDER BY month
        ) AS closing_balance
    FROM monthly_transactions
)
SELECT
    customer_id,
    month,
    closing_balance
FROM running_balance
ORDER BY 1, 2;
```
**Output Preview:** 
<br> ![image](https://github.com/user-attachments/assets/5532c722-6f03-4413-afc6-40ea421b7162)



##### 5. What is the percentage of customers who increase their closing balance by more than 5%?
**Logic:**
- Our goal is to calculate the closing balance for each customer at the end of each month and then determine how many customers have increased their balance by more than 5% from the previous month.
- Similar to the previous question, we will first calculate the monthly closing balance for each customer.
- Next, for each customer, we will calculate the percentage change in the closing balance from one month to the next.
- Next, we will filter the results to find customers whose closing balance increased by more than 5% from the previous month.
- Lastly, we will calculate the percentage of customerrs who had an increase of more than 5%.
```sql
WITH monthly_transactions AS(
	SELECT 
		customer_id,
		DATE_TRUNC('month', txn_date) AS month,
		SUM(CASE 
			WHEN txn_type = 'deposit' THEN txn_amount
			ELSE -txn_amount
		END
		) AS monthly_net_amount
	FROM data_bank.customer_transactions 
	GROUP BY 1, 2
),
running_balance AS(
	SELECT 
		customer_id, 
		month,
		SUM(monthly_net_amount) OVER(PARTITION BY customer_id ORDER BY month) AS closing_balance
	FROM monthly_transactions
),
balance_change AS(
	SELECT 
		customer_id, 
		month,
		closing_balance,
		LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY month) AS previous_closing_balance
    FROM running_balance
),
percentage_change AS (
    SELECT
        customer_id,
        month,
        closing_balance,
        previous_closing_balance,
        CASE 
            WHEN previous_closing_balance IS NOT NULL AND previous_closing_balance > 0 THEN 
                (closing_balance - previous_closing_balance) / previous_closing_balance * 100
            ELSE 0
        END AS balance_change_percentage
    FROM balance_change
)
SELECT
    ROUND((SUM(CASE WHEN balance_change_percentage > 5 THEN 1 ELSE 0 END) * 100.0) / COUNT(DISTINCT customer_id), 2) AS percentage_customers_increased_by_5_percent
FROM percentage_change;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/98810168-cf56-48fe-98c2-26ce1c0df7f6)



### C. Data Allocation Challenge
- To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
	- Option 1: data is allocated based off the amount of money at the end of the previous month
 	- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
	- Option 3: data is updated real-time
- For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:
	- running customer balance column that includes the impact each transaction
	- customer balance at the end of each month
	- minimum, average and maximum values of the running balance for each customer
- Using all of the data available - how much data would have been required for each option on a monthly basis?
**Logic:**
- The goal of the challenge is to find out how much data storage the Data Bank team needs to provide for their customers based on different ways of calculating their customers' balances. The amount of data each customer gets is linked to how much money they have in their accounts.
- To figure this out, we have three options to calculate how much data each customer should be allocated. We need to analyse the data to see which option would work best. Examples for each option:
	- Option 1: If a customer has $2,000 at the end of January, we use that $2,000 to decide how much data they should get for February.
 	- Option 2:  If a customer’s balance varied over 30 days (like they had $1,000 for 15 days and $2,000 for the other 15 days), the average balance would be $1,500. We would use this $1,500 to decide how much data they should get.
	- Option 3: If a customer deposits $500 and then withdraws $200 on the same day, the balance would change twice, and we'd calculate the data allocation based on these real-time changes.
- For each option, we need to:
  	1. Calculate the running balance: total balance after every transaction.
  	2. Calculate the balance at the end of each month.
  	3. Calculate the minimum, average, and maximum balances: This helps us to understand how much data might be needed at different times.
- Finally, we will calculate the required data for each option on a monthly basis
#### Answer:
#### Option 1: Data is allocated based off the amount of money at the end of the previous month
```sql
WITH transactions_with_balance AS (
    SELECT 
        customer_id,
        txn_date,
        txn_type,
        txn_amount,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount
                ELSE -txn_amount
            END
        ) OVER (
            PARTITION BY customer_id 
            ORDER BY txn_date
        ) AS running_balance
    FROM data_bank.customer_transactions
),
monthly_closing_balances AS (
    SELECT 
        customer_id,
        DATE_TRUNC('month', txn_date) AS month,
        LAST_VALUE(running_balance) OVER (
            PARTITION BY customer_id, DATE_TRUNC('month', txn_date)
            ORDER BY txn_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ) AS closing_balance
    FROM transactions_with_balance
)
SELECT * FROM monthly_closing_balances;
```
**Output Preview** 
<br>![image](https://github.com/user-attachments/assets/fdc7e41d-5ff5-43d3-a3db-bb701064fdea)
