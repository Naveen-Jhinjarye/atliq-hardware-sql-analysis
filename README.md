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
