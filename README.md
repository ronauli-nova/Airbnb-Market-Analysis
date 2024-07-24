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
-- Total Revenue
SELECT
    CASE
        WHEN GROUPING(r.name) = 1 THEN 'Total'
        ELSE r.name
    END AS region_name,
    TO_CHAR(SUM(o.standard_amt_usd), 'FM$999,999,999.00') AS total_standard_rev,
    TO_CHAR(SUM(o.gloss_amt_usd), 'FM$999,999,999.00') AS total_gloss_rev,
    TO_CHAR(SUM(o.poster_amt_usd), 'FM$999,999,999.00') AS total_poster_rev,
    TO_CHAR(SUM(o.standard_amt_usd) + SUM(o.gloss_amt_usd) + SUM(o.poster_amt_usd), 'FM$999,999,999.00') AS total_revenue
FROM
    orders o
    JOIN accounts a ON a.id = o.account_id
    JOIN sales_reps s ON a.sales_rep_id = s.id
    JOIN region r ON s.region_id = r.id
GROUP BY
    ROLLUP(r.name);

-- Total Product Sold
 SELECT
	 CASE 
	 	 WHEN GROUPING (r.name) = 1 THEN 'Total'
	 	 ELSE r.name
	 	END AS region_name,
        TO_CHAR(SUM(o.standard_qty), 'FM999,999,999') AS total_standard_qty,
        TO_CHAR(SUM(o.gloss_qty), 'FM999,999,999') AS total_gloss_qty,
        TO_CHAR(SUM(o.poster_qty), 'FM999,999,999') AS total_poster_qty,
	 	TO_CHAR(SUM(o.standard_qty) + SUM(o.gloss_qty) + SUM(o.poster_qty), 
	 'FM999,999,999') AS total_product_sold
    FROM
        orders o
        JOIN accounts a ON a.id = o.account_id
		JOIN sales_reps s ON a.sales_rep_id = s.id
        JOIN region r ON s.region_id = r.id
    GROUP BY
        ROLLUP(r.name);

-- Total Purchase
SELECT
    CASE
        WHEN GROUPING(r.name) = 1 THEN 'Total'
        ELSE r.name
    END AS region_name,
    TO_CHAR(COUNT(o.id), 'FM999,999,999') AS total_order,
    TO_CHAR(COUNT(CASE WHEN o.standard_amt_usd > o.gloss_amt_usd AND o.standard_amt_usd > o.poster_amt_usd THEN o.id END), 'FM999,999,999') AS most_order_standard,
    TO_CHAR(COUNT(CASE WHEN o.gloss_amt_usd > o.standard_amt_usd AND o.gloss_amt_usd > o.poster_amt_usd THEN o.id END), 'FM999,999,999') AS most_order_gloss,
    TO_CHAR(COUNT(CASE WHEN o.poster_amt_usd > o.standard_amt_usd AND o.poster_amt_usd > o.gloss_amt_usd THEN o.id END), 'FM999,999,999') AS most_order_poster
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
	SUM(total) AS total_order,
	DATE_TRUNC('month', occurred_at) AS date
FROM 
	orders
GROUP BY 2
ORDER BY 2 ASC;

-- Monthly Purchase trend
SELECT
	COUNT(id) AS total_purchase,
	DATE_TRUNC('month', occurred_at) AS date
FROM 
	orders
GROUP BY 2
ORDER BY 2 ASC;

-- Top 10 Sales Representatives
SELECT s.name AS sales_rep_name,
	TO_CHAR(SUM(o.total_amt_usd), 'FM$999,999,999.00') AS total_revenue, 
	TO_CHAR(SUM(o.total), 'FM999,999,999') AS total_product_sold,
	COUNT(o.id) AS total_purchase
FROM orders o
JOIN accounts a ON o.account_id = a.id
JOIN sales_reps s ON s.id = a.sales_rep_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```
</details>


**Total Revenue :** <br>
<p align="center">
<kbd><img width="612" alt="image" src="https://github.com/user-attachments/assets/9982130e-db1c-4092-a49f-ad61fe588d48"></kbd>
</p>

- Revenue by region <br>
The columns total_standard_rev, total_gloss_rev, and total_poster_rev show how much revenue was generated from each paper type in each region.
The total_revenue column sums up the revenue from all paper types for each region. The highest sales by region and paper type were on **standard** type in the **Northeast, USA** in total $3,227,886.29. The lowest sales by region and paper type were on **poster** type in the **Midwest, USA** in total $677,021.24.

- Grand Total <br>
The row labeled 'Total' provides a summary of revenue across all paper type.
This row shows the combined revenue from all region and the overall total revenue.

- Revenue Distribution <br>
The smallest revenue among all paper type is the **poster** type with $5,876,005.52. Among all regions, Midwest is the region with the smallest revenue with $3,013,486.51

**Total Product Sold :** <br>
<p align="center">
<kbd><img width="641" alt="image" src="https://github.com/user-attachments/assets/8bc33a1f-e531-44a4-859c-c974d8e36f95"></kbd>
</p>

In line with revenue, the largest number of products sold were of the type **standard** in the **Northeast, USA** with amount 646,871 and the smallest number of products sold are of type **poster**  in the **Midwest, USA** with amaount 83,377.

**Total Purchase :** <br>
<p align="center">
<kbd><img width="676" alt="image" src="https://github.com/user-attachments/assets/e20f43f6-5d9b-42f8-b65a-3660cc10005d"></kbd>
</p>

The total_order column shows the total number of orders in each region. The total_orders column shows the total number of orders in each region, while the other columns show the number of orders with each type of paper that is ordered the most. Northeast is the region with largest amount of order and Midwest is the region smallest amount of order. The total_order column does not always show the sum of all columns in one row because there may be a same quantity of paper type in an order.

**Monthly Revenue trend :** <br>
<p align="center">
<kbd><img width="679" alt="image" src="https://github.com/user-attachments/assets/97ca76f2-1c2c-4de4-ae32-64de2074375d">
</kbd>
</p>

**Monthly Product Sold trend :** <br>
<p align="center">
<kbd><img width="678" alt="image" src="https://github.com/user-attachments/assets/34b999f7-ca65-4aba-b380-cf08b71ef979">
</kbd>
</p>

**Monthly Purchase trend :** <br>
<p align="center">
<kbd><img width="680" alt="image" src="https://github.com/user-attachments/assets/84e16fe0-2f1d-41f6-a650-be69e13d2378">
</kbd>
</p>

The graph shows that revenue, products sold, and purchases have increased from December 2013 to December 2016.

**Top 10 Performing Sales Representatives :** <br>
<p align="center">
<kbd><img width="595" alt="image" src="https://github.com/user-attachments/assets/167c3678-7223-4748-b377-66353d821097">
</kbd>
</p>

Vernita Plump managed to generate the most revenue among all representatives with $934,212.93 revenue, 150,467 products sold, and 299 purchases.

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
    TO_CHAR(customer_value,'FM$999,999,999.00') AS customer_value,
    avg_years,
    TO_CHAR(cust_value.customer_value * avg_cust_lifespan.avg_years, 'FM$999,999,999.00') AS customer_lifetime_value
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
<kbd><img width="487" alt="image" src="https://github.com/user-attachments/assets/16763f95-5eb1-4708-b308-32ae10efdbb6">
</kbd>
</p>

Average CLV represent average revenue gained from customer times average number of years of someone being a customer. It shows that average CLV of Parch and Posey is $61,735.64.

**Churn Rate :** <br>
<p align="center">
<kbd><img width="477" alt="image" src="https://github.com/user-attachments/assets/9f28ed82-eff1-493b-b138-8f38f4f2985c"></kbd>
</p>

Churn rate is percentage of total customers before the period, but did not make any order during the period. It shows that churn rate of of Parch and Posey is 19.31%

**Customer Segmentation :** <br>
1. Based on spending
<p align="center">
<kbd><img width="432" alt="image" src="https://github.com/user-attachments/assets/b43b7d46-f4d6-4f54-b987-a73783d2ba94"></kbd>
</p>

distribution
<p align="left">
<kbd><img width="342" alt="image" src="https://github.com/user-attachments/assets/2ca5226b-616a-4817-a467-eb78c5e9997d"></kbd>
</p>

This segmentation is based on customers who spend more than $50,000 (High Value), between $10,000 and $50,000 (Medium Value) and under $10,000 (Low Value). Distribution table shows that there are 147 high value customers, 147 medium value customers, and 56 low value customers.

2. Based on region
<p align="center">
<kbd><img width="504" alt="image" src="https://github.com/user-attachments/assets/a77cc353-222a-4715-ba7a-4070071f4438"></kbd>
</p>

distribution
<p align="left">
<kbd><img width="279" alt="image" src="https://github.com/user-attachments/assets/81f1c4f4-389b-4e14-a6fd-af8dcdb6f5cf"></kbd>
</p>

Distribution table shows that there are 106 customers from Northeast, 101 customers from West, 96 customers from Southeast, and 48 customers from Midwest.

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
    TO_CHAR((CAST(cust_action AS DECIMAL) / CAST(listed_cust AS DECIMAL)) * 100, 'FM99,999%') AS conversion_rate
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
<p align="center">
<kbd><img width="268" alt="image" src="https://github.com/user-attachments/assets/bf6bccee-1bce-4eb2-bcf3-2842e8ed0c31"></kbd>
</p>

The result identify which channels bring in the most sales. **Direct** channel is the most effective channel to gain customer because The Marketing team successfully managed to earn the most revenue through this channel.

- Customer Acquisition
<p align="center">
<kbd><img width="247" alt="image" src="https://github.com/user-attachments/assets/fab43cee-d4a5-4765-8209-c1807d1e6315"></kbd>
</p>

This determines how many new customers each channel brings in. The table shows that customers come mostly from **direct** channel.


**Conversion Rates :**
<p align="center">
<kbd><img width="505" alt="image" src="https://github.com/user-attachments/assets/e168c8fc-ff15-41cb-af14-30efa5f3e431">
</kbd>
</p>

Conversion Rate is the percentage of visitors or leads that take a desired action, such as making a purchase, signing up for a newsletter, or filling out a contact form. The table shows percentage of customers that make order in the company based on channel. This indicates that all registered customers have placed an order.

**Customer Engagement :**
  Customer Engagement measures the level of interaction and involvement customers have with a brand through various channels. It includes metrics like time spent on the website, click-through rates, social media interactions, and repeat visits.
<p align="center">
<kbd><img width="297" alt="image" src="https://github.com/user-attachments/assets/a0666f30-571a-46fd-bec5-22e376ff7c0d"></kbd>
</p>




# STAGE 3: Summary and Recommendations
