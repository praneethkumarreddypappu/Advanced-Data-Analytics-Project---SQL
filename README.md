# Advanced-Data-Analytics-Project---SQL
A comprehensive collection of SQL scripts for data exploration, analytics, and reporting. These scripts cover various analyses such as database exploration, measures and metrics, time-based trends, cumulative analytics, segmentation, and more.
This repository contains SQL queries designed to help data analysts and BI professionals quickly explore, segment, and analyze data within a relational database. Each script focuses on a specific analytical theme and demonstrates best practices for SQL queries.


-- ===============================
-- Database Setup
-- ===============================
IF EXISTS (SELECT 1 FROM sys.databases WHERE name = 'DataWarehouseAnalytics')
BEGIN
    ALTER DATABASE DataWarehouseAnalytics SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    DROP DATABASE DataWarehouseAnalytics;
END;
GO

CREATE DATABASE DatawarehouseAnalytics;
GO
USE DatawarehouseAnalytics;
GO
CREATE SCHEMA gold;
GO

-- ===============================
-- Table Creation
-- ===============================
CREATE TABLE gold.dim_customers(
    customer_key int,
    customer_id int,
    customer_number nvarchar(50),
    first_name nvarchar(50),
    last_name nvarchar(50),
    country nvarchar(50),
    marital_status nvarchar(50),
    gender nvarchar(50),
    birthdate date,
    create_date date
);
GO

CREATE TABLE gold.dim_products(
    product_key int,
    product_id int,
    product_number nvarchar(50),
    product_name nvarchar(50),
    category_id nvarchar(50),
    category nvarchar(50),
    subcategory nvarchar(50),
    maintenance nvarchar(50),
    cost int,
    product_line nvarchar(50),
    start_date date
);
GO

CREATE TABLE gold.fact_sales(
    order_number nvarchar(50),
    product_key int,
    customer_key int,
    order_date date,
    shipping_date date,
    due_date date,
    sales_amount int,
    quantity tinyint,
    price int
);
GO

-- ===============================
-- Data Loading
-- ===============================
TRUNCATE TABLE gold.dim_customers;
TRUNCATE TABLE gold.dim_products;
TRUNCATE TABLE gold.fact_sales;

BULK INSERT gold.dim_customers
FROM 'C:\Users\prane\Downloads\gold.dim_customers.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', TABLOCK);
GO

BULK INSERT gold.dim_products
FROM 'C:\Users\prane\Downloads\gold.dim_products.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', TABLOCK);
GO

BULK INSERT gold.fact_sales
FROM 'C:\Users\prane\Downloads\gold.fact_sales.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', TABLOCK);
GO

-- ===============================
-- Analytical Queries
-- ===============================

-- Changes Over Time Analysis
SELECT
    FORMAT(order_date, 'yyy-MMM') AS month,
    DATETRUNC(MONTH, order_date) AS order_date,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(quantity) AS total_quantity,
    SUM(sales_amount) AS revenue
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY FORMAT(order_date, 'yyy-MMM'), DATETRUNC(MONTH, order_date)
ORDER BY 1;

-- Cumulative Analysis
SELECT
    DATETRUNC(MONTH, order_date) AS month_name,
    SUM(sales_amount) AS total_sales,
    SUM(SUM(sales_amount)) OVER(PARTITION BY DATETRUNC(YEAR, order_date) ORDER BY DATETRUNC(MONTH, order_date)) AS running_total,
    AVG(sales_amount) AS avg_sales,
    AVG(AVG(sales_amount)) OVER(PARTITION BY DATETRUNC(YEAR, order_date) ORDER BY DATETRUNC(MONTH, order_date)) AS running_avg
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY DATETRUNC(MONTH, order_date), DATETRUNC(YEAR, order_date)
ORDER BY 1;

-- Running Total per Year
SELECT
    *,
    SUM(total_sales) OVER(ORDER BY month_name) AS running_total
FROM (
    SELECT
        DATETRUNC(YEAR, order_date) AS month_name,
        SUM(sales_amount) AS total_sales
    FROM gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(YEAR, order_date)
) t
ORDER BY 1;

-- Performance Analysis
WITH yearly_sales AS (
    SELECT
        YEAR(s.order_date) AS order_year,
        p.product_name,
        SUM(s.sales_amount) AS total_amount
    FROM gold.fact_sales s
    LEFT JOIN gold.dim_products p
        ON s.product_key = p.product_key
    WHERE s.order_date IS NOT NULL
    GROUP BY YEAR(s.order_date), p.product_name
)
SELECT
    *,
    LAG(total_amount, 1) OVER(PARTITION BY product_name ORDER BY order_year) AS previous_year_sales,
    AVG(total_amount) OVER(PARTITION BY product_name) AS avg_amount
FROM yearly_sales
GROUP BY order_year, product_name, total_amount
ORDER BY 2, 1;

-- Part-to-Whole Analysis
SELECT
    p.category,
    SUM(s.sales_amount) AS total_amount,
    CONCAT(
        ROUND(
            CAST(SUM(s.sales_amount) AS FLOAT) * 100 / (SELECT SUM(sales_amount) FROM gold.fact_sales),
            2
        ),
        '%'
    ) AS pct
FROM gold.dim_products p
JOIN gold.fact_sales s
    ON p.product_key = s.product_key
WHERE p.category IS NOT NULL
GROUP BY p.category;

-- Data Segmentation by Product Cost
WITH pro_cat AS (
    SELECT
        product_name,
        cost,
        CASE
            WHEN cost <= 700 THEN 'Low_Price'
            WHEN cost BETWEEN 700 AND 1400 THEN 'Medium_Price'
            ELSE 'High_Price'
        END AS Price_Category
    FROM gold.dim_products
)
SELECT
    Price_Category,
    COUNT(*) AS each_count
FROM pro_cat
GROUP BY Price_Category
ORDER BY 2 DESC;

-- Customer Segmentation Example
WITH customers_spending AS (
    SELECT
        c.customer_key,
        SUM(s.quantity) AS total_qty,
        MIN(s.order_date) AS first_order,
        MAX(s.order_date) AS last_order,
        SUM(s.sales_amount) AS total_sales
    FROM gold.dim_customers c
    JOIN gold.fact_sales s
        ON c.customer_key = s.customer_key
    GROUP BY c.customer_key
),
dates AS (
    SELECT *,
           DATEDIFF(MONTH, first_order, last_order) AS months_history
    FROM customers_spending
),
segmant AS (
    SELECT *,
           CASE
               WHEN months_history >= 12 AND total_sales > 5000 THEN 'VIP'
               WHEN months_history >= 12 AND total_sales <= 5000 THEN 'Regular'
               ELSE 'New'
           END AS segments
    FROM dates
)
SELECT
    segments,
    COUNT(*) AS total_customers
FROM segmant
GROUP BY segments;

-- ===============================
-- Customer Report View
-- ===============================
CREATE VIEW gold.report_customers AS
WITH base_query AS (
    SELECT
        f.order_number,
        f.product_key,
        f.order_date,
        f.sales_amount,
        f.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        DATEDIFF(YEAR, birthdate, GETDATE()) AS age
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_customers c
        ON f.customer_key = c.customer_key
    WHERE order_date IS NOT NULL
),
customer_aggregation AS (
    SELECT
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(DISTINCT product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS life_span
    FROM base_query
    GROUP BY customer_key, customer_number, customer_name, age
)
SELECT
    customer_key,
    customer_number,
    customer_name,
    age,
    CASE
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        ELSE '50 and above'
    END AS age_group,
    CASE
        WHEN life_span >= 12 AND total_sales > 5000 THEN 'VIP'
        WHEN life_span >= 12 AND total_sales <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,
    DATEDIFF(MONTH, last_order_date, GETDATE()) AS recency,
    total_orders,
    total_sales,
    total_quantity,
    total_products,
    last_order_date,
    CASE WHEN total_orders = 0 THEN 0 ELSE total_sales / total_orders END AS avg_order_value,
    CASE WHEN life_span = 0 THEN 0 ELSE total_sales / life_span END AS avg_monthly_spending;

-- ===============================
-- Products Report View
-- ===============================
CREATE VIEW gold.report_products AS
WITH basic_query AS (
    SELECT
        f.order_number,
        f.order_date,
        f.customer_key,
        f.sales_amount,
        f.quantity,
        p.product_key,
        p.product_name,
        p.category,
        p.subcategory,
        p.cost
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON f.product_key = p.product_key
    WHERE order_date IS NOT NULL
),
product_aggregations AS (
    SELECT
        product_key,
        product_name,
        category,
        subcategory,
        cost,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
        MAX(order_date) AS last_sale_date,
        COUNT(DISTINCT order_number) AS total_orders,
        COUNT(DISTINCT customer_key) AS total_customers,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        SUM(CAST(sales_amount AS FLOAT)) / NULLIF(SUM(quantity), 0) AS avg_selling_price
    FROM basic_query
    GROUP BY product_key, product_name, category, subcategory, cost, quantity
)
SELECT
    product_key,
    product_name,
    category,
    subcategory,
    cost,
    lifespan,
    last_sale_date,
    DATEDIFF(MONTH,last_sale_date, GETDATE()) AS recency,
    CASE 
        WHEN total_sales > 50000 THEN 'High-Performer'
        WHEN total_sales BETWEEN 10000 AND 50000 THEN 'Mid-Range'
        ELSE 'Low-Performer'
    END AS product_segment,
    total_orders,
    total_customers,
    total_sales,
    total_quantity,
    avg_selling_price,
    CASE WHEN total_orders = 0 THEN 0 ELSE total_sales / total_orders END AS average_order_revenue,
    CASE WHEN lifespan = 0 THEN 0 ELSE total_sales / lifespan END AS average_monthly_revenue;

-- View the products report
SELECT * FROM gold.report_products;


## 🌟 About Me

Hi there! I'm **Praneeth Kumar Reddy Pappu**. I’m a Data Analyst, and I love working with data

Let's stay in touch! Feel free to connect with me on the following platforms:

[![LinkedIn](https://www.linkedin.com/in/praneethrdy/)
