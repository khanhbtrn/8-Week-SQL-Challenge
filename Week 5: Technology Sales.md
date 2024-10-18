## 🛒Week 5: Technology Sales - Data Mart

### 🔎 Business Problem
Data Mart is an online supermarket specializing in fresh produce, founded by Danny, who previously managed international operations. In June 2020, Data Mart made significant supply chain changes by implementing sustainable packaging across all products, from farm to customer.

<br> Danny seeks analysis of how this change affected sales performance across various business areas. The key questions for analysis include:

- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?
### 📉 Business Insights
- 
### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/b310c38b-83a4-487e-9dfd-741c0485bd3f)

#### Table: weekly_sales
 - This case study only has one table.
### 📒Case Study Questions & Solutions
### A. Data Cleansing Steps
##### In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:
- Convert the `week_date` to a `DATE` format
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value
<br> ![image](https://github.com/user-attachments/assets/7796dc9e-f424-40d4-9137-57bc33bc6109)
- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
<br> ![image](https://github.com/user-attachments/assets/68c64317-751b-4e26-ac3e-61d5d8db569d)
- Ensure all `null` string values with an `"unknown"` string value in the original `segment` column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record


```sql
DROP TABLE IF EXISTS clean_weekly_sales;
CREATE TEMP TABLE clean_weekly_sales AS (
SELECT
  TO_DATE(week_date, 'DD/MM/YY') AS week_date,
  DATE_PART('week', TO_DATE(week_date, 'DD/MM/YY')) AS week_number,
  DATE_PART('month', TO_DATE(week_date, 'DD/MM/YY')) AS month_number,
  DATE_PART('year', TO_DATE(week_date, 'DD/MM/YY')) AS calendar_year,
  region, 
  platform, 
  segment,
  CASE 
    WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
    WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
    WHEN RIGHT(segment,1) in ('3','4') THEN 'Retirees'
    ELSE 'unknown' END AS age_band,
  CASE 
    WHEN LEFT(segment,1) = 'C' THEN 'Couples'
    WHEN LEFT(segment,1) = 'F' THEN 'Families'
    ELSE 'unknown' END AS demographic,
  transactions,
  ROUND((sales::NUMERIC/transactions),2) AS avg_transaction,
  sales
FROM data_mart.weekly_sales
);
```
### B. Data Exploration
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
