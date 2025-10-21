# Advanced-Data-Analytics-Project---SQL
A comprehensive collection of SQL scripts for data exploration, analytics, and reporting. These scripts cover various analyses such as database exploration, measures and metrics, time-based trends, cumulative analytics, segmentation, and more.
This repository contains SQL queries designed to help data analysts and BI professionals quickly explore, segment, and analyze data within a relational database. Each script focuses on a specific analytical theme and demonstrates best practices for SQL queries.

This project is a hands-on SQL analytics pipeline built to analyze and report retail sales data. It demonstrates data modeling, ETL, analysis, and reporting using structured data and SQL queries.

### 1. Database Setup

* Created a new database `DataWarehouseAnalytics`.
* Defined a schema `gold` for clean data organization.
* Built dimension tables (`dim_customers`, `dim_products`) and a fact table (`fact_sales`) to structure transactional and master data.

### 2. Data Import & Cleaning

* Used **BULK INSERT** to load CSV files into the tables.
* Cleaned, validated, and formatted data to ensure consistency.
* Removed duplicates and handled missing values for accurate analysis.

### 3. Sales & Trend Analysis

* Performed **time-based trend analysis** (monthly and yearly sales).
* Calculated **cumulative sales, average order value, and running totals** to track business performance.
* Identified seasonal trends and customer engagement patterns.

### 4. Performance Analysis

* Evaluated **product performance** using historical sales data.
* Compared year-over-year sales and calculated average revenue per product.
* Highlighted top-performing products and revenue drivers.

### 5. Customer Segmentation

* Segmented customers into **VIP, Regular, and New** based on purchase history and spending.
* Calculated KPIs such as total orders, total sales, recency, and average monthly spending.
* Provided actionable insights for targeted marketing and retention strategies.

### 6. Reporting Views

* Created reusable **SQL views** for customer and product reporting:

  * `report_customers` – aggregates customer data, segments, and KPIs.
  * `report_products` – summarizes product sales, segments, and revenue metrics.
* These views are **ready for integration with BI tools** like Power BI or Tableau.

---

## Key Skills Demonstrated

* SQL: Joins, Window Functions, Aggregations, Subqueries, Views
* Data Engineering: ETL Pipelines, Data Cleaning, Data Integration
* Business Analysis: KPI Tracking, Trend Analysis, Customer Segmentation
* Reporting: Data-ready SQL views for dashboards and visualizations

---

## Outcome & Impact

* Built an **end-to-end analytics pipeline** for retail sales data.
* Produced **actionable insights** for customer retention, sales growth, and product management.
* Delivered a **reproducible framework** that can be scaled for larger datasets or integrated with BI platforms.


## About Me

Hi there! I'm **Praneeth Kumar Reddy Pappu**. I’m a Data Analyst, and I love working with data

Let's stay in touch! Feel free to connect with me on the following platforms:

[![LinkedIn] https://www.linkedin.com/in/praneethrdy/
