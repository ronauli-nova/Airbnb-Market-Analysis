# Analyzing Sales Performance in Parch and Posey Company

# STAGE 0: Problem Statement
## Introduction
Parch and Posey is a company specializing in the manufacturing and distribution of three distinct paper varieties: Standard Paper, Gloss Paper, and Poster Paper. The organization consists of 50 Sales Representatives, each strategically positioned across four key regions within the United States, namely the Northwest, Southeast, West, and Midwest.

## Objective
- Sales performance
- Targeted Marketing Strategies
- Top-performing Paper Type
- Seosonal Sales Trend
- Customer Spending Behavior
- Customer Loyalty
- Effective Channels
- Regional performance

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
 Gambar 1. Entity Relationship Diagram



# STAGE 2: Data Analysis
Monthly Hotel Booking Analysis Based on Hotel Type
Impact Analysis of Stay Duration on Hotel Bookings Cancellation Rates
Impact Analysis of Lead Time on Hotel Bookings Cancellation Rates


# STAGE 3: Summary and Recommendations
