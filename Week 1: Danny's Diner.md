## 📍Week 1: Danny's Diner

### 🔎 Business Problem

Danny’s Diner is facing challenges in understanding customer behavior and needs assistance in utilizing basic operational data. The goal is to analyze customer visiting patterns, spending habits, and favorite menu items to enhance customer experiences and decide on the expansion of the loyalty program. Additionally, Danny requires help in creating easy-to-inspect datasets for his team without relying on SQL.

### 🖋 Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/788c9c23-bc08-4e8b-aad1-d125bb759bd6)

### 📊 Example Datasets

#### Table 1: sales
![image](https://github.com/user-attachments/assets/168a377c-5cbd-446f-b426-ad00f1be60b8)

#### Table 2: menu
![image](https://github.com/user-attachments/assets/98bf9376-49b0-4af7-859d-7223825b2d0a)

#### Table 3: members
![image](https://github.com/user-attachments/assets/b9316d61-a493-44e7-8895-f4a3c961f164)

### 📒Case Study Questions & Solutions
##### 1. What is the total amount each customer spent at the restaurant?
**Logic**
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



##### 2. How many days has each customer visited the restaurant?
**Logic:**
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



##### 3. What was the first item from the menu purchased by each customer?
**Logic:**
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



##### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
**Logic:**
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



##### 5. Which item was the most popular for each customer?
**Logic:**
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



##### 6. Which item was purchased first by the customer after they became a member?
**Logic:**
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



##### 7. Which item was purchased just before the customer became a member?
**Logic:**
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



##### 8. What is the total items and amount spent for each member before they became a member?
**Logic:**
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



##### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier how many points would each customer have?
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



##### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
**Logic:**
<br> We need to calculate the total points for customers A and B by the end of January. 
<br> This includes:
<br> • Base Points: Points earned from their purchases normally, without considering the loyalty system. Each dollar spent equals 10 points, and sushi purchases have a 2x multiplier.
<br> • First Week Loyalty Points: Additional points earned during the first week after they join the loyalty program, where all items earn 2x points, not just sushi.**
```sql
WITH valid_dates AS(
	SELECT
		customer_id,
		join_date,
		join_date + 6 AS valid_date,
		DATE_TRUNC('month', '2021-01-31'::DATE) + INTERVAL '1 month' - INTERVAL '1 day' AS last_date -- This is to identify the last day of the month of the given date, which is January 31, 2021
	FROM members 
		)
		
SELECT 
	s.customer_id,
	SUM(CASE
			WHEN s.order_date BETWEEN vd.join_date AND vd.valid_date THEN m.price*20
			WHEN m.product_name = 'sushi' THEN m.price*20
			ELSE m.price*10
		END) AS points
FROM sales s 
	JOIN valid_dates AS vd ON s.customer_id = vd.customer_id 
		AND vd.join_date <= s.order_date 
		AND s.order_date <= vd.last_date
	JOIN menu m ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 1;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/8c78520f-5c34-4845-bc48-d473640e0e94)



##### Bonus Question 1. Recreate a table with the following columns: customer_id, order_date, product_name, price, and member(Y/N) so that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL
**Logic:**
<br> The Member column is marked as "Y" when the order_date is on or after the join_date. We use a LEFT JOIN between the sales and members tables to include all customers, including customer C, who may not be a member
```sql
SELECT
	s.customer_id,
	s.order_date,
	m.product_name,
	m.price,
	CASE
		WHEN s.order_date >= mem.join_date THEN 'Y'
		ELSE 'N'
	END AS member
FROM sales s 
    JOIN menu m ON s.product_id = m.product_id
    LEFT JOIN members mem ON s.customer_id = mem.customer_id
ORDER BY 1,2;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/24aad4a6-5b6a-4f79-aa43-34cce40c7fff)



##### Bonus Question 2. Rank All The Things. Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
**Logic:**
<br> Based on the table above, we first need to fill in the ranking column, assigning NULL values to customers who are not members. The key aspect of using the RANK() function is to PARTITION BY customer_id to ensure that the ranking is calculated separately for each customer. We need to also include the membership status in the partitioning to distinguish between members and non-members, which is to ensure that the ranking only counts the orders of members. The ordering should be done by order_date as required.
```sql
WITH new_cte AS(
	SELECT
		s.customer_id,
		s.order_date,
		m.product_name,
		m.price,
		CASE
			WHEN s.order_date >= mem.join_date THEN 'Y'
			ELSE 'N'
		END AS member
	FROM sales s 
	    JOIN menu m ON s.product_id = m.product_id
	    LEFT JOIN members mem ON s.customer_id = mem.customer_id
	ORDER BY 1,2
	)

SELECT 
	*,
	CASE 
		WHEN member = 'N' THEN NULL
		ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date) 
	END AS ranking
FROM new_cte;
```
**Output:** 
<br>![image](https://github.com/user-attachments/assets/dacaf8eb-318d-4b4a-a8d3-e1bf1a1c09bb)


### Business Insights
##### Customer Spending Patterns:
Customer A is the highest spender, with a total of $85 spent, followed by Customer B with $74, and Customer C with $36. This indicates a need to focus on retaining high-spending customers through personalized offers and loyalty rewards.
##### Most Popular Menu Item:
Ramen is the most purchased item across all customers, indicating its popularity. Marketing efforts could leverage this by promoting ramen-centric deals or special ramen days to attract more customers.
##### Impact of Membership Program:
Members tend to continue purchasing their favorite items even after joining the loyalty program, with sushi being a preferred choice. Enhancing the membership benefits, like exclusive member-only sushi discounts, could further encourage frequent visits.
##### Customer Visit Frequency:
Customer A visited the diner on the most days, highlighting their loyalty. Analyzing the visit frequency can help in identifying peak times and days for each customer, enabling targeted marketing campaigns to increase foot traffic during off-peak times.
### Recommendations
**• Focus on High-Spenders:** Develop personalized marketing strategies for high-spending customers to ensure their loyalty and increase their visit frequency.
<br> **• Leverage Popular Items:** Use popular items like ramen to attract more customers by promoting special deals and events centered around these items.
<br> **• Enhance Membership Benefits:** Offer exclusive benefits and promotions to encourage non-members to join and existing members to remain loyal.
<br> **• Analyze Visit Patterns:** Utilize visit frequency data to identify and target peak and off-peak times with tailored promotions.
<br> **• Optimize Loyalty Points System:** Regularly update and promote the loyalty points system, ensuring customers are aware of how they can maximize their rewards through strategic spending and participation in promotions.
