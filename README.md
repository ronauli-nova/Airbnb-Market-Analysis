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

3. Marketing
- Effective channels

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
### Sales performance
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
<kbd>
</kbd>
</p>

**Monthly Product Sold trend :** <br>
<p align="center">
<kbd>
</kbd>
</p>

**Monthly Purchase trend :** <br>
<p align="center">
<kbd>
</kbd>
</p>

**Total Purchase each Region :** <br>
<p align="center">
<kbd><img width="233" alt="image" src="https://github.com/user-attachments/assets/7f15f230-3653-4bf5-9bc5-be145164cdee"></kbd>
</p>



- Revenue & Sales
- Top-performing Sales Representatives
- Top-performing Paper Type
- Monthly Purchase Trend
- Regional performance




# STAGE 3: Summary and Recommendations
