# Analyzing Sales Performance in Parch and Posey Company

# STAGE 0: Problem Statement
## Introduction
Parch and Posey is a company specializing in the manufacturing and distribution of three distinct paper varieties: Standard Paper, Gloss Paper, and Poster Paper. The organization consists of 50 Sales Representatives, each strategically positioned across four key regions within the United States, namely the Northwest, Southeast, West, and Midwest.

## Objective
1. Sales performance
- Revenue & Sales
- Top-performing Sales Representatives
- Top-performing Paper Type
- Seosonal Sales Trend
- Regional performance
  
2. Customer Analysis
- Customer Lifetime Value (CLTV)
- Churn Rate
- Customer Segmentation
- Geographic Distribution

3. Marketing Analysis
- Effective channels
- Conversion Rates
- Customer Engagement

## Data Explaination
The data was extracted from Parch and Posey database where it was stored in 5 tables which are:
- **Accounts**: Contains information on customer accounts such as account ID, account name, primary person of contact, account location, and sales representative ID.
- **Order**: Contains information on account ID, quantity of orders, amount spent on the orders for each paper type, as well as the date the transaction occurred.
- **Region**: Contains the region ID and the name of the region.
- **Sales\_reps\_id**: Contains the sales representative's name and ID, as well as region ID.
- **Web\_events**: Contains information on the channel through which the customer found out about the company.

# STAGE 1: Data Preprocessing
## Data Overview
The Parch and Posey dataset consists of 6912 data from December 2013 to January 2017. It consists of several tables such as accounts, orders, sales_reps, web_events, and region.

## Create Database and ERD 
<details>
  <summary>Query to Count Rows in Accounts Table</summary>

  ```sql
-- 1) Create a database by right-clicking Databases > Create > Database.. named "project"
-- 2) Create a table using the CREATE TABLE statement by right-clicking project > CREATE script
  CREATE TABLE orders (
	id integer,
	account_id integer,
	occurred_at timestamp,
	standard_qty integer,
	gloss_qty integer,
	poster_qty integer,
	total integer,
	standard_amt_usd numeric(10,2),
	gloss_amt_usd numeric(10,2),
	poster_amt_usd numeric(10,2),
	total_amt_usd numeric(10,2)
);

CREATE TABLE accounts (
	id integer,
	name bpchar,
	website bpchar,
	lat numeric(11,8),
	long numeric(11,8),
	primary_poc bpchar,
	sales_rep_id integer
);

CREATE TABLE web_events (
	id integer,
	account_id integer,
	occurred_at timestamp,
	channel bpchar
);

CREATE TABLE sales_reps (
	id integer,
	name bpchar,
	region_id integer
);

CREATE TABLE region (
	id integer,
	name bpchar
);

-- 3) Importing csv files into each table that has been created by right-clicking on the table name > Import/Export Data...
-- 4) Determine the PRIMARY KEY and FOREIGN KEY in each table to determine the relationship between tables

-- PRIMARY KEY
ALTER TABLE orders
ADD PRIMARY KEY (id);

ALTER TABLE accounts
ADD PRIMARY KEY (id);

ALTER TABLE web_events
ADD PRIMARY KEY (id);

ALTER TABLE sales_reps
ADD PRIMARY KEY (id);

ALTER TABLE region
ADD PRIMARY KEY (id);

-- FOREIGN KEY
ALTER TABLE accounts ADD FOREIGN KEY (sales_rep_id) REFERENCES sales_reps;
ALTER TABLE orders ADD FOREIGN KEY (account_id) REFERENCES accounts;
ALTER TABLE web_events ADD FOREIGN KEY (account_id) REFERENCES accounts;
ALTER TABLE sales_reps ADD FOREIGN KEY (region_id) REFERENCES region;
```
</details>

**Hasil ERD :** <br>
<p align="center">
 <kbd><img width="555" alt="image" src="https://github.com/user-attachments/assets/8ad4c979-4229-45d3-ab27-c2577a3a2d5d"> </kbd> <br>

</p>
<br>

# STAGE 2: Data Analysis
## Sales performance
To effectively monitor and improve sales performance, businesses rely on several key metrics. Understanding these metrics in business terms provides valuable insights for strategic decision-making and overall growth.
<details>
  <summary>Query to Count Rows in Accounts Table</summary>
	
```sql
-- Total Revenue & Product Sold by Region
SELECT r.name AS region, 
	TO_CHAR(SUM(o.total_amt_usd), 'FM$999,999,999.00') AS total_revenue,
	TO_CHAR(SUM(o.total), 'FM999,999,999')  AS total_product_sold
FROM orders o
JOIN accounts a ON o.account_id = a.id
JOIN sales_reps s ON s.id = a.sales_rep_id
JOIN region r ON r.id = s.region_id
GROUP BY 1
ORDER BY 2 DESC;

-- Top 10 Sales Representatives
SELECT s.name AS sales_rep_name,
	TO_CHAR(SUM(o.total_amt_usd), 'FM$999,999,999.00') AS total_revenue, 
	TO_CHAR(SUM(o.total), 'FM999,999,999') AS total_product_sold
FROM orders o
JOIN accounts a ON o.account_id = a.id
JOIN sales_reps s ON s.id = a.sales_rep_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

-- Most sold paper type
WITH paper_sales_per_region AS (
    SELECT
        r.name AS region_name,
        SUM(o.standard_qty) AS total_standard_qty,
        SUM(o.gloss_qty) AS total_gloss_qty,
        SUM(o.poster_qty) AS total_poster_qty
    FROM
        orders o
        JOIN accounts a ON a.id = o.account_id
		JOIN sales_reps s ON a.sales_rep_id = s.id
        JOIN region r ON s.region_id = r.id
    GROUP BY
        r.name
)
SELECT
    region_name,
    CASE 
        WHEN total_standard_qty > total_gloss_qty AND total_standard_qty > total_poster_qty THEN 'Standard'
        WHEN total_gloss_qty > total_standard_qty AND total_gloss_qty > total_poster_qty THEN 'Gloss'
        WHEN total_poster_qty > total_standard_qty AND total_poster_qty > total_gloss_qty THEN 'Poster'
    END AS most_sold_paper_type
FROM
    paper_sales_per_region;

-- Total revenue of paper type each region
SELECT
    CASE
        WHEN GROUPING(r.name) = 1 THEN 'Total'
        ELSE r.name
    END AS region_name,
    TO_CHAR(SUM(o.standard_amt_usd), 'FM$999,999,999.00') AS total_standard_rev,
    TO_CHAR(SUM(o.gloss_amt_usd), 'FM$999,999,999.00') AS total_gloss_rev,
    TO_CHAR(SUM(o.poster_amt_usd), 'FM$999,999,999.00') AS total_poster_rev
FROM
    orders o
    JOIN accounts a ON a.id = o.account_id
    JOIN sales_reps s ON a.sales_rep_id = s.id
    JOIN region r ON s.region_id = r.id
GROUP BY
    ROLLUP(r.name);

-- Total product sold based on paper type each region
 SELECT
	CASE 
		WHEN GROUPING (r.name) = 1 THEN 'Total'
		ELSE r.name
	 	END AS region_name,
        TO_CHAR(SUM(o.standard_qty), 'FM999,999,999') AS total_standard_qty,
        TO_CHAR(SUM(o.gloss_qty), 'FM999,999,999') AS total_gloss_qty,
        TO_CHAR(SUM(o.poster_qty), 'FM999,999,999') AS total_poster_qty
    FROM
        orders o
        JOIN accounts a ON a.id = o.account_id
		JOIN sales_reps s ON a.sales_rep_id = s.id
        JOIN region r ON s.region_id = r.id
    GROUP BY
        ROLLUP(r.name);

-- Monthly Revenue trend
SELECT
	SUM(total_amt_usd) AS total_revenue,
	DATE_TRUNC('month', occurred_at) AS date
FROM 
	orders
GROUP BY 2
ORDER BY 2 ASC;

-- Monthly Product Sold trend
SELECT
	SUM(total) AS total_revenue,
	DATE_TRUNC('month', occurred_at) AS date
FROM 
	orders
GROUP BY 2
ORDER BY 2 ASC;

-- Monthly Purchase trend
SELECT
	COUNT(id) AS total_order,
	DATE_TRUNC('month', occurred_at) AS date
FROM 
	orders
GROUP BY 2
ORDER BY 2;

-- Total order per region
SELECT
    CASE
        WHEN GROUPING(r.name) = 1 THEN 'Total'
        ELSE r.name
    END AS region_name,
    TO_CHAR(COUNT(o.id), 'FM999,999,999') AS total_order
FROM
    orders o
    JOIN accounts a ON a.id = o.account_id
    JOIN sales_reps s ON a.sales_rep_id = s.id
    JOIN region r ON s.region_id = r.id
GROUP BY
    ROLLUP(r.name);
```
</details>

**Total Revenue & Product Sold by Region :** <br>
<p align="center">
<kbd><img width="391" alt="image" src="https://github.com/user-attachments/assets/2a66631f-d897-4674-bb9c-949e80a20aa2">
</kbd>
</p>
<br>

**Top 10 Performing Sales Representatives :** <br>
<p align="center">
<kbd><img width="457" alt="image" src="https://github.com/user-attachments/assets/3025f5bf-85f6-48cb-9e0a-19b4a11f05b2"></kbd>
</p>

**Total Revenue based on Paper Type each Region :** <br>
<p align="center">
<kbd><img width="493" alt="image" src="https://github.com/user-attachments/assets/86b5d553-8a72-4171-bd54-f6cdccaf89d5"></kbd>
</p>

**Total Product Sold based on Paper Type each Region :** <br>
<p align="center">
<kbd><img width="497" alt="image" src="https://github.com/user-attachments/assets/d98f8312-fe58-480d-bd56-32e8c4c5faa9"></kbd>
</p>

**Monthly Revenue trend :** <br>
<p align="center">
<kbd><img width="678" alt="image" src="https://github.com/user-attachments/assets/e5ae0f3a-0224-4666-b804-4a6126576a21">
</kbd>
</p>

**Monthly Product Sold trend :** <br>
<p align="center">
<kbd><img width="679" alt="image" src="https://github.com/user-attachments/assets/d97855fe-e78e-4f37-ba74-27ede9a9dbd0">
</kbd>
</p>

**Monthly Purchase trend :** <br>
<p align="center">
<kbd><img width="679" alt="image" src="https://github.com/user-attachments/assets/71aac670-d262-47ee-92a8-099c0f3e2f6f">
</kbd>
</p>

**Total Purchase each Region :** <br>
<p align="center">
<kbd><img width="251" alt="image" src="https://github.com/user-attachments/assets/d5f9823a-893c-487e-9301-4083bcd8bc58">
</kbd>
</p>

## Customer Analysis
<details>
  <summary>Query to Count Rows in Accounts Table</summary>

 ```sql
-- Customer Lifetime Value
WITH sub1 AS (
    --- Average purchase value
    SELECT
        SUM(total_amt_usd) * 1.0 / COUNT(id) AS avg_purchase_value
    FROM
        orders
    WHERE
        occurred_at BETWEEN '2016-01-02' AND '2017-01-02'
),
sub2 AS (
    --- Average Frequency Rate
    SELECT
        COUNT(id) * 1.0 / COUNT(DISTINCT account_id) AS avg_freq_rate
    FROM 
        orders
    WHERE
        occurred_at BETWEEN '2016-01-02' AND '2017-01-02'
),
cust_value AS (
    SELECT
        ROUND(sub1.avg_purchase_value * sub2.avg_freq_rate, 2) AS customer_value
    FROM
        sub1, sub2
),
lifespan AS (
    --- Number of years each customer has been with you
    SELECT
        account_id,
        EXTRACT(YEAR FROM AGE(MAX(occurred_at), MIN(occurred_at))) + 1 AS years_as_customer
    FROM
        orders
    GROUP BY
        account_id
),
avg_cust_lifespan AS (
    --- Average Customer Lifespan
    SELECT
        ROUND(AVG(years_as_customer), 2) AS avg_years
    FROM
        lifespan
)
SELECT
    customer_value,
    avg_years,
    cust_value.customer_value * avg_cust_lifespan.avg_years AS customer_lifetime_value
FROM
    cust_value,
    avg_cust_lifespan;


-- Churn rate
WITH customers_at_start AS (
    SELECT DISTINCT account_id
    FROM orders
    WHERE occurred_at < '2016-01-02'
),
customers_during_period AS (
    SELECT DISTINCT account_id
    FROM orders
    WHERE occurred_at BETWEEN '2016-01-02' AND '2017-01-02'
),
churned_customers AS (
    SELECT account_id
    FROM customers_at_start
    WHERE account_id NOT IN (SELECT account_id FROM customers_during_period)
)
SELECT
    (SELECT COUNT(*) FROM churned_customers) AS churned_cust,
    (SELECT COUNT(*) FROM customers_at_start) AS total_customers_at_start,
    ROUND((SELECT COUNT(*) FROM churned_customers)::decimal / (SELECT COUNT(*) FROM customers_at_start) * 100, 2) 
	AS churn_rate;

-- Customer segmentation
--- Based on spending
SELECT account_id, SUM(total_amt_usd) AS total_spent,
           CASE
               WHEN SUM(total_amt_usd) > 50000 THEN 'High Value'
               WHEN SUM(total_amt_usd) BETWEEN 10000 AND 50000 THEN 'Medium Value'
               ELSE 'Low Value'
           END AS customer_segment
    FROM orders
    GROUP BY account_id
    ORDER BY total_spent DESC;

---- distribution
WITH segment AS (
SELECT account_id, SUM(total_amt_usd) AS total_spent,
           CASE
               WHEN SUM(total_amt_usd) > 50000 THEN 'High Value'
               WHEN SUM(total_amt_usd) BETWEEN 10000 AND 50000 THEN 'Medium Value'
               ELSE 'Low Value'
           END AS customer_segment
    FROM orders
    GROUP BY account_id
    ORDER BY total_spent DESC
	)
SELECT customer_segment,
		COUNT(*) AS total_account
FROM segment
GROUP BY 1
ORDER BY 2;

--- Based on region
SELECT a.id, a.name, r.name AS region
FROM accounts a
JOIN sales_reps s ON a.sales_rep_id = s.id
JOIN region r ON s.region_id = r.id;

----distribution
SELECT r.name AS region,
		COUNT(a.id) AS total_customers
FROM accounts a
JOIN sales_reps s ON a.sales_rep_id = s.id
JOIN region r ON s.region_id = r.id
GROUP BY 1
ORDER BY total_customers DESC;
```
</details>

**Customer Lifetime Value (CLTV) :** <br>
<p align="center">
<kbd><img width="500" alt="image" src="https://github.com/user-attachments/assets/1692ebf1-9191-4713-a464-0ce24bc99ece"></kbd>
</p>

<p align="center">
<kbd><img width="486" alt="image" src="https://github.com/user-attachments/assets/6530046d-4f2e-46a6-98d1-709022322397"></kbd>
</p>

**Churn Rate :** <br>
Churn rate : persentase dari total customer sebelum periode, namun tidak melakukan order kembali selama periode.
		periode : 1 tahun terakhir (2016-01-02 sampai 2017-01-02)
<p align="center">
<kbd><img width="477" alt="image" src="https://github.com/user-attachments/assets/9f28ed82-eff1-493b-b138-8f38f4f2985c"></kbd>
</p>

**Customer Segmentation :** <br>
1. Based on spending
<p align="center">
<kbd><img width="432" alt="image" src="https://github.com/user-attachments/assets/b43b7d46-f4d6-4f54-b987-a73783d2ba94"></kbd>
</p>

distribution
<p align="left">
<kbd><img width="342" alt="image" src="https://github.com/user-attachments/assets/2ca5226b-616a-4817-a467-eb78c5e9997d"></kbd>
</p>

2. Based on region
<p align="center">
<kbd><img width="504" alt="image" src="https://github.com/user-attachments/assets/a77cc353-222a-4715-ba7a-4070071f4438"></kbd>
</p>

distribution
<p align="left">
<kbd><img width="279" alt="image" src="https://github.com/user-attachments/assets/81f1c4f4-389b-4e14-a6fd-af8dcdb6f5cf"></kbd>
</p>

**Geographic Distribution :** <br>
<p align="center">
<kbd><img width="671" alt="image" src="https://github.com/user-attachments/assets/73148a57-f8d1-43a3-9d8a-68fa96202dc9"></kbd>
</p>

## Marketing Analysis
<details>
  <summary>Query to Count Rows in Accounts Table</summary>

 ```sql
-- Channel Effectiveness
--- Revenue Generation:
SELECT channel, 
	TO_CHAR(SUM(total_amt_usd), 'FM$999,999,999,00') AS total_revenue
FROM web_events we
JOIN orders o ON we.account_id = o.account_id
GROUP BY 1
ORDER BY 2 DESC;

--- Customer Acquisition
SELECT channel,
		COUNT(account_id)  AS total_cust
FROM web_events
GROUP BY 1
ORDER BY 2 DESC;
	
-- Conversion Rates
WITH sub AS (
    SELECT channel, 
        COUNT(DISTINCT o.account_id) AS cust_action,
        COUNT(DISTINCT we.account_id) AS listed_cust 
    FROM web_events we
    LEFT JOIN orders o ON we.account_id = o.account_id
    GROUP BY 1
)
SELECT *,
    (CAST(cust_action AS DECIMAL) / CAST(listed_cust AS DECIMAL)) * 100 AS conversion_rate
FROM sub;

--- Customer Engagement
SELECT channel, ROUND(AVG(events),2) AS average_events
FROM (
    SELECT 
		DATE_TRUNC('day', occurred_at) AS day, 
		channel, 
		COUNT(*) AS events
    FROM web_events
    GROUP BY day, channel
) sub
GROUP BY channel
ORDER BY average_events DESC;
```
</details>

**Effective channels :**
- Revenue Generation
  Identify which channels bring in the most sales. For example, if email marketing generates $100,000 in sales while social media generates $50,000, email marketing is more effective.
<p align="center">
<kbd>
<img width="268" alt="image" src="https://github.com/user-attachments/assets/bf6bccee-1bce-4eb2-bcf3-2842e8ed0c31"></kbd>
</p>

- Customer Acquisition
  Determine how many new customers each channel brings in. Channels that attract more customers are generally considered more effective.

<p align="center">
<kbd>
<img width="247" alt="image" src="https://github.com/user-attachments/assets/fab43cee-d4a5-4765-8209-c1807d1e6315"></kbd>
</p>

**Conversion Rates :**
  Conversion Rate is the percentage of visitors or leads that take a desired action, such as making a purchase, signing up for a newsletter, or filling out a contact form. It’s a critical metric for assessing the effectiveness of marketing efforts.
<p align="center">
<kbd>
<img width="563" alt="image" src="https://github.com/user-attachments/assets/8fd0a389-653e-4f3d-859a-54016ecba238"></kbd>
</p>

**Customer Engagement :**
  Customer Engagement measures the level of interaction and involvement customers have with a brand through various channels. It includes metrics like time spent on the website, click-through rates, social media interactions, and repeat visits.
<p align="center">
<kbd><img width="297" alt="image" src="https://github.com/user-attachments/assets/a0666f30-571a-46fd-bec5-22e376ff7c0d"></kbd>
</p>


# STAGE 3: Summary and Recommendations
