# Data-Bank-SQL-Case-Study
The Data Bank SQL Case Study focuses on analyzing customer transactions and reallocation patterns for a digital-only bank. I am providing insights on customer activity, transaction metrics, and data storage trends to support business growth and decision-making.

## Project Documentation Structure
- ***Introduction***
- ***Tools Used***
- ***Problem Statement***
- ***Entity Relationship Diagram***
- ***Dataset Explaination***
- ***Case Study Analysis with Insights***

## Introduction
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

![image](https://8weeksqlchallenge.com/images/case-study-designs/4.png)

## Tools Used
- **MySQL** – Data extraction, transformation, and analysis
- **Excel** – Data validation
- **GitHub** – Project documentation

## Problem Statement
The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## Entity Relationship Diagram
![image](https://8weeksqlchallenge.com/images/case-study-4-erd.png)

## Dataset Explaination

### Table 1: regions
This regions table contains the region_id and their respective region_name values.

### Table 2: customer_nodes
In this table, Customers are randomly distributed across the nodes according to their region. This random distribution changes frequently to reduce the risk of hackers getting into Data Bank’s system and stealing customer’s money and data!

### Table 2: customer_transactions
This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

## Case Study Analysis with Insights

### Customer Nodes Exploration:

#### 1. How many unique nodes are there on the Data Bank system?
```sql
 SELECT
      COUNT(DISTINCT node_id) AS unique_nodes
  FROM
      customer_nodes;
```
#### Output
![image](https://github.com/user-attachments/assets/4afc2760-8ece-4921-89c2-53306a652f8a)
###### Insights: There are 5 unique nodes on the Data Bank system.

#### 2. What is the number of nodes per region?
```sql
SELECT 
    region_id, region_name, COUNT(node_id) AS no_of_nodes
FROM
    customer_nodes
	JOIN
    regions USING (region_id)
GROUP BY region_id , region_name
ORDER BY region_id;
```
#### Output
![image](https://github.com/user-attachments/assets/221cf57d-6db1-4e3a-b759-8e07a1b5a4f4)
###### Insights: Australia has the highest number of nodes with 770, while Europe has the lowest number of nodes with 616, indicating a larger customer distribution in Australia compared to Europe.

#### 3. How many customers are allocated to each region?
```sql
SELECT 
    region_name, COUNT(DISTINCT customer_id) AS no_of_customers
FROM
    customer_nodes cn
JOIN
    regions r USING (region_id)
GROUP BY region_name
ORDER BY COUNT(customer_id) DESC;
```
#### Output
![image](https://github.com/user-attachments/assets/c85e8efa-c5dc-455f-9e82-c9c0dcbc8d73)
###### Insights: Australia has the highest customer base (110), while Europe has the lowest (88), indicating higher market penetration in Australia and lower engagement in Europe.

#### 4. How many days on average are customers reallocated to a different node?
```sql
SELECT 
    ROUND(AVG(DATEDIFF(end_date, start_date)), 2) AS avg_days
FROM
    customer_nodes
WHERE
    end_date != '9999-12-31';
```
#### Output
![image](https://github.com/user-attachments/assets/d73f958d-e240-4062-8d51-700348320f66)
###### Insights: On average, customers are reallocated to a different node every 14.63 days, indicating a frequent reallocation pattern across the network

#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
WITH Percentiles AS (
    SELECT 
        r.region_name,
        DATEDIFF(c.end_date, c.start_date) AS reallocation_days,
        ROW_NUMBER() OVER (PARTITION BY r.region_name ORDER BY DATEDIFF(c.end_date, c.start_date)) AS row_num,
        COUNT(*) OVER (PARTITION BY r.region_name) AS total_rows
    FROM customer_nodes c
    INNER JOIN regions r USING (region_id)
    WHERE c.end_date != '9999-12-31'
)
SELECT 
    region_name,
    ROUND(MIN(CASE WHEN row_num >= total_rows * 0.5 THEN reallocation_days END), 2) AS median,
    ROUND(MIN(CASE WHEN row_num >= total_rows * 0.8 THEN reallocation_days END), 2) AS percentile_80,
    ROUND(MIN(CASE WHEN row_num >= total_rows * 0.95 THEN reallocation_days END), 2) AS percentile_95
FROM Percentiles
GROUP BY region_name;
```
#### Output
![image](https://github.com/user-attachments/assets/17b2c391-5e5d-459f-9c01-7252608b0dc6)
###### Insights:
###### - The median reallocation days is 15 days across all regions, indicating that half of the customers are reallocated within this period.
###### - The 80th percentile ranges between 23 to 24 days, meaning 20% of customers take longer than this to be reallocated.
###### - The 95th percentile is consistently 28 days, showing that only 5% of customers take more than 28 days to be reallocated, reflecting a stable reallocation pattern across regions.

### Customer Transactions Analysis:

#### 6. What is the unique count and total amount for each transaction type?
```sql
SELECT DISTINCT
    txn_type,
    COUNT(txn_type) AS unique_count,
    SUM(txn_amount) AS total_amt
FROM
    customer_transactions
GROUP BY txn_type;
```
#### Output
![image](https://github.com/user-attachments/assets/e92b5698-31bf-4c2b-a538-4beaf3d928c9)
###### Insights:
###### - Deposits have the highest transaction count (2671) and total amount (1,359,168), indicating that customers primarily use the bank for fund accumulation.
###### - Purchases (806,537) exceed Withdrawals (793,003) in total amount, showing that customers prefer spending directly from their accounts rather than withdrawing cash.
###### - The lower withdrawal count (1580) suggests that customers are more inclined towards digital transactions than cash-based transactions.

#### 7. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH deposit AS (
SELECT 
    customer_id,
    COUNT(customer_id) AS txn_count,
    AVG(txn_amount) AS avg_amt
FROM
    customer_transactions
WHERE
    txn_type = 'deposit'
GROUP BY customer_id
)
SELECT 
    ROUND(AVG(txn_count), 2) AS avg_deposit_count,
    ROUND(AVG(avg_amt), 2) AS avg_deposit_amt
FROM
    deposit;
```
#### Output
![image](https://github.com/user-attachments/assets/0e814af3-365c-4055-896e-ce1b2ca68eba)
###### Insights:
###### - On average, each customer makes approximately 5.34 deposit transactions.
###### - The average deposit amount is 508.61, indicating that customers prefer making smaller but frequent deposits rather than large lump-sum amounts.

#### 8. For each month, how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH monthly_transactions AS (
  SELECT 
	customer_id,
    MONTH(txn_date) AS month,
    SUM(CASE WHEN txn_type="deposit" THEN 1 ELSE 0 END) AS deposit_count,
	SUM(CASE WHEN txn_type="purchase" THEN 1 ELSE 0 END) AS purchase_count,
	SUM(CASE WHEN txn_type="withdrawal" THEN 1 ELSE 0 END) AS withdrawal_count
    FROM customer_transactions
    GROUP BY customer_id,MONTH(txn_date)
)
SELECT 
	month,
	COUNT(DISTINCT customer_id) AS customer_count
    FROM monthly_transactions
    WHERE deposit_count>1
    AND (purchase_count>=1 OR withdrawal_count>=1)
    GROUP BY month;
```
#### Output
![image](https://github.com/user-attachments/assets/b09aafe4-a5b9-4cbb-ace4-4b17577041ab)
###### Insights:
###### - The number of active customers making more than one deposit along with at least one purchase or withdrawal shows a gradual increase from January (168) to March (192).
###### - However, there is a significant drop in April (70), indicating reduced customer activity during that month.

#### 9. What is the closing balance for each customer at the end of the month?
```sql
WITH MonthlyTxn AS (
    SELECT 
        customer_id,
        MONTH(txn_date) AS txn_month,
        SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE 0 END) AS total_deposits,
        SUM(CASE WHEN txn_type = 'withdrawal' THEN txn_amount ELSE 0 END) AS total_withdrawals,
        SUM(CASE WHEN txn_type = 'purchase' THEN txn_amount ELSE 0 END) AS total_purchases
    FROM customer_transactions
    GROUP BY customer_id, MONTH(txn_date)
),
Balances AS (
    SELECT 
        customer_id, txn_month,
        total_deposits, total_withdrawals, total_purchases,
        SUM(total_deposits - total_withdrawals - total_purchases) OVER (
            PARTITION BY customer_id ORDER BY txn_month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS closing_balance
    FROM MonthlyTxn
)
SELECT 
    customer_id, txn_month,
    closing_balance
FROM Balances
ORDER BY customer_id, txn_month;
```
#### Output
![image](https://github.com/user-attachments/assets/0167ffb2-34c2-4b1e-8cd4-0367000edaba)
###### Insights: Each customer had a varying closing balance each month, fluctuating between positive and negative, reflecting dynamic financial activity

### Further Analysis:

#### 10. How many active customers (those who made at least 1 transaction in the last 3 months)?
```sql
SELECT 
    COUNT(DISTINCT customer_id) AS active_customers
FROM
    customer_transactions
WHERE
    txn_date >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH);
```
#### Output
![image](https://github.com/user-attachments/assets/9c009e1c-570b-4aa2-b0e8-dbca784c9761)
###### Insights: It indicates that their is no active customers from the last 3 months.

#### 11. Which 3 regions have the highest total deposit amounts?
```sql
SELECT 
    r.region_name, SUM(t.txn_amount) AS total_deposits
FROM
    customer_transactions AS t
        INNER JOIN
    customer_nodes AS c ON t.customer_id = c.customer_id
        INNER JOIN
    regions AS r ON c.region_id = r.region_id
WHERE
    txn_type = 'deposit'
GROUP BY r.region_name
ORDER BY total_deposits DESC
LIMIT 3;
```
#### Output
![image](https://github.com/user-attachments/assets/792e8d97-c7ad-462e-8bb1-fd977e701a0e)
###### Insights: America recorded the highest total deposits among regions, followed by Australia and Asia, indicating greater customer deposit activity in America.

#### 12. When did each customer make their first-ever transaction?
```sql
SELECT 
    customer_id, MIN(txn_date) AS first_transaction_date
FROM
    customer_transactions
GROUP BY customer_id;
```
#### Output
![image](https://github.com/user-attachments/assets/5965dab1-9da8-4d6a-b7b5-a63e698a18ee)
###### Insights: Each customer has a unique first transaction date, reflecting individual transaction initiation patterns.

