# Atliq-Hardware-SQLAnalysis

## Introduction
This project is part of the Codebasics Resume Project Challenge. AtliQ Hardwares is one of India's leading computer hardware producers with a growing global presence. The management wanted to expand their data analytics team and conducted a SQL challenge to evaluate candidates on both technical and analytical skills.
I solved 10 real-world ad hoc business requests using MySQL, covering product analysis, sales trends, customer insights, and channel performance.


### ❓ Problem Statement
AtliQ Hardwares noticed that they were not getting enough insights to make quick, smart, data-informed decisions. Tony Sharma (Data Analytics Director) designed a SQL challenge with 10 ad hoc business requests to test candidates' ability to extract meaningful insights from raw data.

#### 🗄️ About the Data
Database: gdb023
TableDescriptiondim_customerCustomer details — name, market, region, channeldim_productProduct details — segment, division, categoryfact_gross_priceGross price per product per fiscal yearfact_manufacturing_costManufacturing cost per product per yearfact_pre_invoice_deductionsPre-invoice discount % per customer per yearfact_sales_monthlyMonthly sold quantity per product per customer


### 📋 Requests & Solutions

#### ✅ Request 1
#### Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.
``` SQL
SELECT DISTINCT market
FROM dim_customer
WHERE customer = "Atliq Exclusive"
  AND region   = "APAC";
```



![Query Result Screenshot](https://github.com/Naveen-Jhinjarye/atliq-hardware-sql-analysis/blob/main/asssts/Screenshot%20(739).png)

#### ✅ Request 2
#### What is the percentage of unique product increase in 2021 vs 2020?
``` SQL
WITH unique_product_table AS (
    SELECT
        COUNT(CASE WHEN fiscal_year = '2020' THEN product END) AS unique_product_2020,
        COUNT(CASE WHEN fiscal_year = '2021' THEN product END) AS unique_product_2021
    FROM dim_product d
    JOIN fact_gross_price f ON f.product_code = d.product_code
)
SELECT *,
    ROUND(
        (unique_product_2021 - unique_product_2020) / unique_product_2020 * 100,
    2) AS perc_chg
FROM unique_product_table;
```
#### 📊 Result:
Show Image

#### ✅ Request 3
#### Provide a report with all the unique product counts for each segment, sorted in descending order.
``` SQL
SELECT
    segment,
    COUNT(product) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;
```
#### 📊 Result:
Show Image

#### ✅ Request 4
#### Which segment had the most increase in unique products in 2021 vs 2020?
``` SQL
WITH uni_pro_tab AS (
    SELECT
        segment,
        COUNT(CASE WHEN fiscal_year = '2020' THEN product END) AS unique_product_2020,
        COUNT(CASE WHEN fiscal_year = '2021' THEN product END) AS unique_product_2021
    FROM dim_product d
    JOIN fact_gross_price f ON f.product_code = d.product_code
    GROUP BY segment
)
SELECT *,
    unique_product_2021 - unique_product_2020 AS difference
FROM uni_pro_tab
ORDER BY difference DESC;
```
#### 📊 Result:
Show Image

#### ✅ Request 5
#### Get the products with the highest and lowest manufacturing costs.
``` SQL
SELECT
    p.product_code,
    p.product,
    m.manufacturing_cost
FROM fact_manufacturing_cost m
JOIN dim_product p ON p.product_code = m.product_code
WHERE manufacturing_cost IN (
    (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost),
    (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost)
);
```
#### 📊 Result:
Show Image

#### ✅ Request 6
#### Top 5 customers with the highest average pre-invoice discount in the Indian market for FY2021.
``` SQL
SELECT
    c.customer_code,
    c.customer,
    f.pre_invoice_discount_pct AS average_discount_percentage
FROM fact_pre_invoice_deductions f
JOIN dim_customer c ON c.customer_code = f.customer_code
WHERE f.fiscal_year = '2021'
  AND c.market      = 'India'
  AND f.pre_invoice_discount_pct > (
        SELECT AVG(pre_invoice_discount_pct)
        FROM fact_pre_invoice_deductions
  )
ORDER BY average_discount_percentage DESC
LIMIT 5;
```
#### 📊 Result:
Show Image

#### ✅ Request 7
#### Complete monthly Gross Sales report for customer "Atliq Exclusive".
``` SQL
SELECT
    MONTHNAME(DATE_ADD(m.date, INTERVAL 4 MONTH)) AS month_name,
    m.fiscal_year                                  AS year,
    ROUND(SUM(g.gross_price * m.sold_quantity) / 1000000, 2) AS gross_sales_mln
FROM fact_sales_monthly m
JOIN fact_gross_price g
    ON m.product_code = g.product_code
    AND m.fiscal_year = g.fiscal_year
JOIN dim_customer c
    ON c.customer_code = m.customer_code
WHERE c.customer = 'Atliq Exclusive'
GROUP BY year, MONTH(m.date)
ORDER BY year, MONTH(m.date);
```
#### 📊 Result:
Show Image

#### ✅ Request 8
#### In which quarter of 2020 was the total sold quantity the highest?
``` SQL
SELECT
    CONCAT("Q", QUARTER(DATE_ADD(date, INTERVAL 4 MONTH))) AS quarter,
    ROUND(SUM(sold_quantity) / 1000000, 2)                 AS total_sold_quantity
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY quarter
ORDER BY total_sold_quantity DESC;
```
#### 📊 Result:
Show Image

#### ✅ Request 9
#### Which channel contributed the most to gross sales in FY2021 and what was its percentage?
``` SQL
WITH gross_table AS (
    SELECT
        c.channel,
        ROUND(SUM(g.gross_price * f.sold_quantity) / 1000000, 2) AS gross_sales_mln
    FROM fact_sales_monthly f
    JOIN fact_gross_price g
        ON g.product_code = f.product_code
        AND f.fiscal_year = g.fiscal_year
    JOIN dim_customer c
        ON f.customer_code = c.customer_code
    WHERE f.fiscal_year = 2021
    GROUP BY c.channel
)
SELECT *,
    ROUND(gross_sales_mln * 100 / SUM(gross_sales_mln) OVER(), 2) AS percentage
FROM gross_table
ORDER BY gross_sales_mln DESC;
```
#### 📊 Result:
Show Image

#### ✅ Request 10
#### Get the Top 3 products in each division by total sold quantity in FY2021.
``` SQL
WITH product_totals AS (
    SELECT
        p.division,
        p.product_code,
        p.product,
        SUM(f.sold_quantity) AS total_sold_quantity
    FROM fact_sales_monthly f
    JOIN fact_gross_price g
        ON f.product_code = g.product_code
        AND g.fiscal_year = f.fiscal_year
    JOIN dim_product p
        ON f.product_code = p.product_code
    WHERE f.fiscal_year = 2021
    GROUP BY p.division, p.product_code, p.product
),
ranked_products AS (
    SELECT *,
        DENSE_RANK() OVER (
            PARTITION BY division
            ORDER BY total_sold_quantity DESC
        ) AS rank_order
    FROM product_totals
)
SELECT *
FROM ranked_products
WHERE rank_order <= 3;
```
#### 📊 Result:
Show Image

## 🧠 SQL Concepts Used

CTEs — Common Table Expressions for clean, modular queries
Window Functions — DENSE_RANK(), OVER(), PARTITION BY
Aggregate Functions — SUM(), COUNT(), AVG(), MAX(), MIN()
Conditional Aggregation — CASE WHEN inside aggregate functions
Subqueries — For filtering with MIN / MAX / AVG
Multi-table JOINs — Across fact and dimension tables
Date Functions — MONTHNAME(), QUARTER(), DATE_ADD()


