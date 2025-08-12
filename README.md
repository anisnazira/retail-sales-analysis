# retail-sales-analysis

This project is for personal learning purposes to practice SQL techniques commonly used by data analysts in exploring, cleaning, and analyzing retail sales data. It involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. Some concepts and approaches in this project are inspired by and credited to Zero Analyst on YouTube. 

Dataset Overview
----------------
The retail_sales dataset contains detailed transactional records from a retail store, capturing both customer and sales-related information. Each entry represents a single transaction, with the transaction_id uniquely identifying it and sale_date indicating when the purchase occurred. Customer demographics are represented through customer_id, gender, and age, enabling segmentation and behavioral analysis. Transactional details include category for the product type, quantities representing the number of units purchased, and price_per_unit showing the cost per item. The total_sale column calculates the total monetary value of each transaction, serving as a key metric for sales performance. This dataset is well-suited for analyzing trends over time, identifying high-value customers, and uncovering patterns in product demand.


Questions
----------------
1. Write a SQL query to retrieve all columns for sales made on '2022-11-05
2. Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022
3. Write a SQL query to calculate the total sales (total_sale) for each category
4. Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category
5. Write a SQL query to find all transactions where the total_sale is greater than 1000
6. Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category
7. Write a SQL query to calculate the average sale for each month. Find out best selling month in each year
8. Write a SQL query to find the top 5 customers based on the highest total sales 
9. Write a SQL query to find the number of unique customers who purchased items from each category
10.Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)
11.Write a SQL query to calculate the total quantity sold for each product_id in the 'Electronics' category.
12.Write a SQL query to find the oldest and youngest customer who made a purchase in December 2022.
13.Write a SQL query to find all customers who purchased from both the 'Clothing' and 'Beauty' categories.
14.Write a SQL query to calculate the percentage contribution of each category to total sales.
15.Write a SQL query to get the total sales for each gender in each month.


## SQL Queries

---

### 1. Retrieve all columns for sales made on `2022-11-05`
```sql
SELECT * 
FROM retail_sales
WHERE sale_date = '2022-11-05';
SELECT * FROM retail_sales
WHERE sale_date = '2022-11-05';
'''
### 2. Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022
```sql
SELECT * FROM retail_sales
WHERE
category = 'Clothing'
AND
quantiy >= 4
AND
TO_CHAR(sale_date, 'YYYY-MM') = '2022-11';


### 3. Write a SQL query to calculate the total sales (total_sale) for each category
```sql
SELECT category,
	   SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY 1

### 4. Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category
```sql
SELECT 
       ROUND(AVG(age),2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'

###5. Write a SQL query to find all transactions where the total_sale is greater than 1000
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000

### 6. Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category
```sql
SELECT category,
	    gender,
	COUNT (*) AS transactions_id
FROM retail_sales
GROUP BY 1,2
ORDER BY 1


 ### 7. Write a SQL query to calculate the average sale for each month. Find out best selling month in each year
```sql

SELECT --select 3 column dri t1
	year,
	month,
	avg_sale
FROM
(
SELECT
	  EXTRACT (YEAR FROM sale_date) as year,
	  EXTRACT (MONTH FROM sale_date) as month,
	  ROUND (AVG (total_sale), 2) as avg_sale,
	  
	  RANK() OVER(
	  PARTITION BY EXTRACT (YEAR FROM sale_date) 
	  ORDER BY ROUND (AVG (total_sale), 2) DESC )
FROM retail_sales
GROUP BY 1,2
) as t1
WHERE rank = 1 --nak yg rank 1 sahaja
--ORDER BY 1,3 DESC
	  
### 8. Write a SQL query to find the top 5 customers based on the highest total sales 
```sql
SELECT 
		customer_id,
	    ROUND (SUM (total_sale), 2) as total_spend
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC 
LIMIT 5


### 9. Write a SQL query to find the number of unique customers who purchased items from each category
```sql
SELECT 
	category,
	COUNT(DISTINCT customer_id) as unique_cust_count
FROM retail_sales
GROUP BY category


###10. Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)
--USING CTE common table expression
```sql
WITH hourly_sale
AS
()
SELECT *,
	CASE
		WHEN EXTRACT (HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT (HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END as shift
FROM retail_sales
)
SELECT * 
	shift,
	COUNT(*) as total_orders
FROM hourly_sale
GROUP BY shift


SELECT EXTRACT (HOUR FROM  CURRENT_TIME)

-- creds Zero Analyst on youtube


### 11.Write a SQL query to find the highest total_sale made in each category.
```sql
SELECT 
	category,
	MAX(total_sale) as highest_sale
FROM retail_sales
GROUP BY category
ORDER BY highest_sale DESC;

### 12.Write an SQL query to calculate the total profit for each product category.
```sql
SELECT 
	category, 
	ROUND(SUM(total_sale - cogs)::numeric, 2) AS total_profit
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
-- NOTES: ROUND(SUM(total_sale - cogs),2) as total_profit doesnt work 
-- bcs cogs is double precision 
-- ::numeric → converts the result of the SUM() from double precision to numeric.


### 12.Write a SQL query to find the oldest and youngest customer who made a purchase in December 2022.
```sql
SELECT 
	   MAX(age) as max_age,
	   MIN(age) as min_age
FROM retail_sales
WHERE
TO_CHAR (sale_date, 'YYYY-MM') = '2022-12'

--OR
SELECT *
FROM retail_sales
WHERE TO_CHAR(sale_date, 'YYYY-MM') = '2022-12'
  AND age IN (
      SELECT MAX(age) FROM retail_sales WHERE TO_CHAR(sale_date, 'YYYY-MM') = '2022-12'
      UNION
      SELECT MIN(age) FROM retail_sales WHERE TO_CHAR(sale_date, 'YYYY-MM') = '2022-12'
  )
ORDER BY age DESC

### 13.Write a SQL query to find all customers who purchased from both the 'Clothing' and 'Beauty' categories.
```sql
SELECT *
FROM retail_sales
WHERE customer_id IN (
    SELECT customer_id
    FROM retail_sales
    WHERE category IN ('Clothing', 'Beauty')
    GROUP BY customer_id
    HAVING COUNT(DISTINCT category) = 2
);

### 14.Write a SQL query to calculate the percentage contribution of each category to total sales.
```sql
SELECT 
    category,
    SUM(total_sale) AS category_sales,
    ROUND((SUM(total_sale) * 100.0 / SUM(SUM(total_sale)) OVER ()),
        2
    ) AS percentage_sale
FROM retail_sales
GROUP BY category
ORDER BY percentage_sale DESC;

### 15.Write a SQL query to get the total sales for each gender in each month.
```sql
SELECT 
    gender,
    EXTRACT(MONTH FROM sale_date) as month,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY 1,2
ORDER BY 2,1

