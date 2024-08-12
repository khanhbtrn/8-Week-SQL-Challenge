## 🥑Week 3: Foodie-Fi

### 🔎 Business Problem

Foodie-Fi is a subscription-run business which offers a new streaming service that only had food related content - something like Netflix but with only cooking shows. This case study focuses on using subscription style digital data to answer important business questions.
### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/69b11935-f13b-48c6-86b5-c02afcb3283b)

### 📊 Example Datasets
 - A dataset of 1,000 customers.
#### Table 1: plans
 - Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90.
 - Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.
 - Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
 - When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

<br>![image](https://github.com/user-attachments/assets/ef0ea703-1159-42f1-9da4-864a122d27ee)

#### Table 2: subscriptions
 - If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.
 - When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.
 - When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

<br>![image](https://github.com/user-attachments/assets/a497351b-19f5-4ddd-a4b3-82856ebcf154)

### 📒Case Study Questions & Solutions
### A. Customer Journey
##### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
 - First, I join relevant columns from two tables together for better observing information of the 8 sample customers
```sql
SELECT 
	s.customer_id,
	s.plan_id,
	p.plan_name,
	p.price,
	s.start_date
FROM foodie_fi.subscriptions s 
	LEFT JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
WHERE customer_id < 9;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/f18a451a-0adc-4738-b1e4-10f3621f1450)
<br> **Descripition:**
 - `customer_id = 1` used 7-day free trial and then downgraded to basic monthly subscription plan.
 - `customer_id = 2` used 7-day free trial and then upgraded it to pro annual subscription plan. 
 - `customer_id = 3` used 7-day free trial and then downgraded to basic monthly subscription plan.
 - `customer_id = 4` used 7-day free trial, downgraded to basic monthly subscription plan, and then decided to cancel subscription after around 03 months subscribing.
 - `customer_id = 5` used 7-day free trial and then downgraded to basic monthly subscription plan.
 - `customer_id = 6` used 7-day free trial, downgraded to basic monthly subscription plan, and then decided to cancel subscription after around 02 months subscribing.
 - `customer_id = 7` used 7-day free trial, downgraded to basic monthly subscription plan, and then upgraded to pro monthly subscription plan.
 - `customer_id = 8` used 7-day free trial, downgraded to basic monthly subscription plan, and then upgraded to pro monthly subscription plan.

### B. Data Analysis Questions
##### 1. How many customers has Foodie-Fi ever had?
**Logic:**
 - Use `COUNT(DISTINCT customer_id)
```sql
SELECT 
	COUNT(DISTINCT customer_id) AS nb_of_customers
FROM foodie_fi.subscriptions;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/4694403c-c679-450a-9344-adfdc58e5774)



##### 2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value
**Logic:**
 - The question is asing for the monthly count of users on the trial plan subscription
```sql
SELECT 
	EXTRACT(month from start_date) AS month_number,
    TO_CHAR(start_date, 'Month') AS month_name, -- To get month name. For example, 'January' instead of number '1'
	COUNT(DISTINCT customer_id) AS user_count
FROM foodie_fi.subscriptions s 
	LEFT JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
WHERE plan_name = 'trial'
GROUP BY 1,2;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/71812008-1a18-481d-95da-55e2abcc88f8)



##### 3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`
**Logic:**
 - The question is asking us to calculate the count of plans with start dates on or after 1 January 2021 for each plan names. We just need to use `COUNT(customer_id)` as the number of events.
```sql
SELECT
	p.plan_id,
	p.plan_name,
	COUNT(customer_id) AS nb_of_events
FROM foodie_fi.subscriptions s JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
WHERE start_date >= '2021-01-01'
GROUP BY 1, 2
ORDER BY 1;
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/5ce8c7c1-50f3-41ad-b812-550939a357ef)



##### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
**Logic:**
 - Percentage of churned customers equals the number of churned customers divided by the total number of customers then multiplied by 100.
```sql
SELECT 
	COUNT(DISTINCT s.customer_id) AS nb_churn_users,
	ROUND(COUNT(DISTINCT s.customer_id)*100.0/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 1) AS percentage_of_churn_users
FROM foodie_fi.subscriptions s LEFT JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
WHERE plan_name = 'churn';
```
**Output:** 
<br> ![image](https://github.com/user-attachments/assets/59385542-f3af-4f6c-8443-7891900147e0)



##### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
**Logic:**
 - To calculate the number of customers who have churned straight after their initial free trial,  we will use `ROW_NUMBER()` and then identify the targeted customers through two conditions: their rank is 2 (indicating immediately) and their plan_id = 4 (churn). 
```sql
WITH churn_num AS(
	SELECT 
		s.customer_id,
		p.plan_id,
		ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY p.plan_id) AS rnk
	FROM foodie_fi.subscriptions s  JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
	)

-- When a customer immediately churned after their free trial, their rnk = 2 should has plan_id = 4.
SELECT
	SUM(CASE WHEN rnk = 2 and plan_id = 4 THEN 1 ELSE 0 END) AS immediate_churn_count,
	ROUND(SUM(CASE WHEN rnk = 2 and plan_id = 4 THEN 1 ELSE 0 END)*100/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 1) AS immediate_churn_percentage
FROM churn_num;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/0f5653f0-2b78-42e5-8b5d-05732df03c73)



##### 6. What is the number and percentage of customer plans after their initial free trial?
**Logic:**
 - This question is asking us to count the number of subscribers for each plan and their percentage after their initial free trial.  
```sql
WITH plans_cte AS (
  SELECT 
    customer_id, 
    plan_id, 
    LEAD(plan_id) OVER(PARTITION BY customer_id ORDER BY plan_id) as next_plan
  FROM foodie_fi.subscriptions
)

SELECT 
  next_plan AS plan_id, 
  COUNT(customer_id) AS customers_after_trial,
  ROUND(COUNT(customer_id)*100/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1) 
  	AS percentage_after_trial
FROM plans_cte
WHERE next_plan IS NOT NULL 
	AND plan_id = 0
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/eb1f7272-c10f-4051-bf16-c731a653c8f2)



##### 7. What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?
**Logic:**
 - Create a CTE to retrieve the next start date using `LEAD()`. This is useful in the outer query later on to get the latest update of start_date.
 - Then, similar to solutions above, we can count and then calculate percentage for each plan.
```sql
WITH next_plans AS (
  SELECT
    customer_id,
    plan_id,
  	start_date,
    LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_start_date
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
	plan_id, 
	COUNT(DISTINCT customer_id) AS customers,
	ROUND(COUNT(DISTINCT customer_id)*100.0/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1) AS percentage
FROM next_plans
WHERE next_start_date IS NULL -- This is to get the latest update of start_date
GROUP BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/94a79ea8-c108-4dca-a6ac-9a8f447d82ef)



##### 8. How many customers have upgraded to an annual plan in 2020?
**Logic:**
 - We need to satisfy two conditions: `plan_id = 3` and `start_date BETWEEN '2020-01-01' AND '2020-12-31'`.
```sql
SELECT 
	COUNT(customer_id) AS nb_annual_customers
FROM foodie_fi.subscriptions
WHERE
	plan_id = 3
	AND start_date BETWEEN '2020-01-01' AND '2020-12-31';
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/b05d6dac-72ac-48c7-95b5-049a178b3a27)



##### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
**Logic:**
 - Create two CTEs to extract customers with pro annual subscriptions `annual_start_date` and another CTE to extract their `trial_start_date`.
 - Then we just need to calculate using `AVG()`.
```sql
WITH annual_date_cte AS(
	SELECT
		customer_id,
		start_date AS annual_start_date
	FROM foodie_fi.subscriptions
	WHERE plan_id = 3
	),
	
trial_date_cte AS(
	SELECT
		an.customer_id,
		start_date AS trial_start_date
	FROM foodie_fi.subscriptions s JOIN annual_date_cte an ON s.customer_id = an.customer_id
	WHERE plan_id = 0
	)
SELECT
	ROUND(AVG(annual_start_date - trial_start_date)) AS avg_switch_plan
FROM annual_date_cte an JOIN trial_date_cte trial ON an.customer_id = trial.customer_id;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/2ceefed0-66cd-4d2f-ab6d-b37f3d583626)

