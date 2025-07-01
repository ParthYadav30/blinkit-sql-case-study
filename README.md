# 🛒 SQL Project: Data Analysis for Blinkit - A Grocery Delivery Company

## 📌 Overview

This project showcases my SQL data analysis skills using a simulated dataset modeled after Blinkit, a leading grocery delivery platform. The goal is to clean, transform, and analyze structured retail data using MySQL to uncover business insights across sales, pricing, inventory, and outlet performance.

---

## 🧱 Project Structure

- **📂 Database Setup:** Normalized Blinkit data into 3 core tables (`items_economics`, `items_qualities`, `outlets`)
- **🧼 Data Cleaning:** Replaced blanks, standardized types
- **📊 Business Queries:** Solved 15 questions using joins, aggregations, rankings, and window functions
- **📈 Advanced Insights:** Built custom metrics like Outlet Performance Index, Dominance Score, and Fat-Preference Index

---

## 📂 Database Schema

![Screenshot 2025-07-01 124451](https://github.com/user-attachments/assets/192163e9-1608-47a2-b72e-c3ae169780ea)


---

## 📦 Sample Business Problems Solved

### ✅ Easy-Level Queries
1. Top 5 selling products by total sales
2. Most profitable item types
3. Total and average sales by outlet size & type
4. Highest visibility item types
5. Outlet age vs revenue analysis

### 🟡 Moderate-Level Queries
6. Sales trends by MRP price bins (0–300 range)
7. Products with low visibility yet higher sales
8. tem types sales stability across outlet types
9. Hidden champions: low visibility, high sales items
10. Outlet fat preference index

### 🔴 Advanced-Level Insights
11. Sales share of top 10% outlets in each tier
12. High Margin, Low Turnover items
13. Item Type Concentration by outlet type
14. Dominance score of item types
15. Price Band Dependency by outlet size

---

## 🧠 SQL Concepts Used

- `JOIN`, `GROUP BY`, `ORDER BY`, `ROUND()`
- `CASE`, `IFNULL()`, `TRIM()` for cleaning & binning
- `CTE`, `NTILE()`, `PERCENT_RANK()`, `RANK()` for segmentation
- `STDDEV_POP()` for sales stability analysis
- Subqueries and window functions for advanced KPI analysis

### 1.CREATING THE DATABASE

```sql
CREATE database blinkit_db;

CREATE TABLE blinkit_sales (
    id INT AUTO_INCREMENT PRIMARY KEY,
    item_identifier VARCHAR(20),
    item_weight FLOAT,
    item_fat_content VARCHAR(20),
    item_visibility FLOAT,
    item_type VARCHAR(100),
    item_mrp FLOAT,
    outlet_identifier VARCHAR(20),
    outlet_establishment_year INT,
    outlet_size VARCHAR(20),
    outlet_location_type VARCHAR(20),
    outlet_type VARCHAR(50),
    item_outlet_sales FLOAT
);

SELECT *
FROM blinkit_sales;
```
### 2.CREATING TABLES 
```sql
CREATE TABLE items_economics (
    item_identifier VARCHAR(20) PRIMARY KEY,
    item_visibility FLOAT,
    item_type VARCHAR(100),
    item_mrp FLOAT
);

CREATE TABLE items_qualities (
    item_identifier VARCHAR(20) PRIMARY KEY,
    item_weight FLOAT,
    item_fat_content VARCHAR(20),
    item_type VARCHAR(100)
);

CREATE TABLE outlets (
    outlet_identifier VARCHAR(20),
    outlet_establishment_year INT,
    outlet_size VARCHAR(20),
    outlet_location_type VARCHAR(20),
    outlet_type VARCHAR(50),
    item_identifier VARCHAR(20),
    item_outlet_sales FLOAT,
    PRIMARY KEY (outlet_identifier, item_identifier),
    FOREIGN KEY (item_identifier) REFERENCES items_economics(item_identifier)
);

select *
from items_economics;

select *
from items_qualities;

select *
from outlets;
```
### 3.TOP 5 SELLING PRODUCTS ACROSS ALL OUTLETS
```sql
SELECT E.item_identifier, ROUND(SUM(O.item_outlet_sales), 2) AS ItemWiseSales
FROM items_economics E
JOIN Outlets O
  ON E.item_identifier = O.item_identifier
GROUP BY E.item_identifier
ORDER BY ItemWiseSales DESC 
LIMIT 5;

```
### 4.MOST PROFITABLE ITEM TYPE
```sql
SELECT Q.item_type,
       ROUND(SUM(O.item_outlet_sales), 2) AS total_sales
FROM items_qualities Q
JOIN outlets O 
  ON Q.item_identifier = O.item_identifier
GROUP BY Q.item_type
ORDER BY total_sales DESC
LIMIT 1;

```
### 5.TOTAL SALES DONE BY OUTLET TYPE
```sql
SELECT outlet_type, ROUND(SUM(O.item_outlet_sales), 2) AS total_sales
FROM Outlets O
GROUP BY outlet_type
ORDER BY total_sales DESC;

```
### 6.AVERAGE SALES PER OUTLET SIZE
```sql
SELECT 
    CASE 
        WHEN outlet_size = '' THEN 'Unknown'
        ELSE outlet_size
    END AS outlet_size_group,
    ROUND(AVG(item_outlet_sales), 2) AS avg_sales
FROM outlets
GROUP BY 
    CASE 
        WHEN outlet_size = '' THEN 'Unknown'
        ELSE outlet_size
    END;
```
### 7.ITEM TYPE WITH HIGHEST VISIBILITY
```sql
SELECT Item_Type, ROUND(AVG(Item_visibility) * 100, 2) AS AvgVisibilityPercent
FROM items_economics
GROUP BY Item_type
ORDER BY AvgVisibilityPercent DESC
LIMIT 1;
```
### 8.SALES BY OUTLET AGE
```sql
SELECT (YEAR(CURDATE()) - outlet_establishment_year) AS Outlet_Age, 
       ROUND(SUM(item_outlet_sales), 2) AS Revenue
FROM outlets
GROUP BY Outlet_age
ORDER BY Outlet_age DESC;

```
### 9.MRP BINNING AND SALES ANALYSIS
```sql
SELECT 
  CASE 
    WHEN item_mrp BETWEEN 0 AND 50 THEN '0–50'
    WHEN item_mrp BETWEEN 51 AND 100 THEN '51–100'
    WHEN item_mrp BETWEEN 101 AND 150 THEN '101–150'
    WHEN item_mrp BETWEEN 151 AND 200 THEN '151–200'
    WHEN item_mrp BETWEEN 201 AND 250 THEN '201–250'
    WHEN item_mrp BETWEEN 251 AND 300 THEN '251–300'
    ELSE 'Other'
  END AS mrp_range,
  ROUND(SUM(item_outlet_sales), 2) AS total_sales
FROM items_economics e
JOIN outlets o ON e.item_identifier = o.item_identifier
GROUP BY mrp_range
ORDER BY 
  FIELD(mrp_range, '0–50', '51–100', '101–150', '151–200', '201–250', '251–300', 'Other');
```    
### 10.PRODUCTS WITH LOW VISIBILITY YET HIGHER SALES
```sql
WITH Low_Visibility_Items AS (
  SELECT E.item_identifier, 
         E.item_visibility, 
         ROUND(SUM(O.item_outlet_sales), 2) AS total_sales
  FROM items_economics E
  JOIN outlets O 
    ON E.item_identifier = O.item_identifier
  WHERE E.item_visibility > 0.05
  GROUP BY item_identifier, item_visibility
),
Ranked_items AS (
  SELECT *, 
         NTILE(10) OVER (ORDER BY total_sales DESC) AS Sales_rank
  FROM Low_Visibility_Items
) 
SELECT *
FROM ranked_items
WHERE Sales_rank = 10
ORDER BY total_sales DESC;

 ```
### 11.ITEM TYPES SALES STABILITY ACROSS ALL OUTLET TYPES
```sql
 WITH sales_by_type_outlet AS (
    SELECT 
        iq.item_type,
        o.outlet_type,
        SUM(o.item_outlet_sales) AS total_sales
    FROM items_qualities iq
    JOIN outlets o ON iq.item_identifier = o.item_identifier
    GROUP BY iq.item_type, o.outlet_type
),
sales_variance AS (
    SELECT 
        item_type,
        ROUND(AVG(total_sales), 2) AS avg_sales,
        ROUND(STDDEV_POP(total_sales), 2) AS sales_stddev
    FROM sales_by_type_outlet
    GROUP BY item_type
)
SELECT *
FROM sales_variance
ORDER BY sales_stddev ASC
LIMIT 10;
```
### 12.HIDDEN CHAMPIONS BY CATEGORY, FIND LOW VISIBILITY ITEMS (VISIBILITY < 0.05) THAT RANK IN THE TOP 5% OF SALES WITHIN THEIR ITEM TYPE
```sql
WITH Sales AS (
  SELECT Q.item_identifier, 
         Q.item_type, 
         E.item_visibility, 
         ROUND(SUM(item_outlet_sales), 2) AS total_sales
  FROM items_qualities Q
  JOIN items_economics E 
    ON Q.item_identifier = E.item_identifier
  JOIN outlets O 
    ON E.item_identifier = O.item_identifier
  WHERE E.item_visibility < 0.05
  GROUP BY Q.item_identifier, Q.item_type, E.item_visibility
),
Rank_data AS (
  SELECT *, 
         PERCENT_RANK() OVER (PARTITION BY item_type ORDER BY total_sales DESC) AS Sales_percent
  FROM Sales
)
SELECT *
FROM Rank_data
WHERE Sales_percent <= 0.05;

```
### 13.Outlet Fat-Preference Index, Which stores sell more ‘Low Fat’ vs ‘Regular’ items?
```sql
SELECT 
  o.outlet_identifier,
  iq.item_fat_content,
  ROUND(SUM(o.item_outlet_sales), 2) AS total_sales
FROM items_qualities iq
JOIN outlets o 
  ON iq.item_identifier = o.item_identifier
GROUP BY o.outlet_identifier, iq.item_fat_content
ORDER BY o.outlet_identifier, total_sales DESC;

```
### 14.Sales Share of Top 10% Outlets in Each Tier
```sql
WITH outlet_sales AS (
  SELECT 
    outlet_identifier,
    outlet_location_type,
    SUM(item_outlet_sales) AS total_sales
  FROM outlets
  GROUP BY outlet_identifier, outlet_location_type
),

ranked_outlets AS (
  SELECT *,
         NTILE(10) OVER (PARTITION BY outlet_location_type ORDER BY total_sales DESC) AS sales_rank
  FROM outlet_sales
),

tier_agg AS (
  SELECT 
    outlet_location_type,
    SUM(CASE WHEN sales_rank = 1 THEN total_sales ELSE 0 END) AS top_10_sales,
    SUM(total_sales) AS total_tier_sales
  FROM ranked_outlets
  GROUP BY outlet_location_type
)

SELECT 
  outlet_location_type,
  ROUND((top_10_sales / total_tier_sales) * 100, 2) AS top_10_percent_sales_share
FROM tier_agg
ORDER BY top_10_percent_sales_share DESC;
```
### 15.High Margin, Low Turnover Items
```sql
WITH global_stats AS (
  SELECT 
    AVG(e.item_mrp) AS avg_mrp_global,
    AVG(o.item_outlet_sales) AS avg_sales_global
  FROM items_economics e
  JOIN outlets o ON e.item_identifier = o.item_identifier
),

item_summary AS (
  SELECT 
    e.item_identifier,
    ROUND(AVG(e.item_mrp), 2) AS avg_mrp,
    COUNT(DISTINCT o.outlet_identifier) AS outlet_count,
    ROUND(SUM(o.item_outlet_sales), 2) AS total_sales
  FROM items_economics e
  JOIN outlets o ON e.item_identifier = o.item_identifier
  GROUP BY e.item_identifier
)

SELECT 
  i.item_identifier,
  i.avg_mrp,
  i.outlet_count,
  i.total_sales
FROM item_summary i
JOIN global_stats g
  ON 1=1
WHERE 
  i.avg_mrp > g.avg_mrp_global
  AND i.outlet_count < 3
  AND i.total_sales < g.avg_sales_global
ORDER BY i.avg_mrp DESC;
```
### 16.Item Type Concentration Index per Outlet Type
```sql
WITH item_sales AS (
  SELECT 
    o.outlet_type,
    iq.item_type,
    ROUND(SUM(o.item_outlet_sales), 2) AS item_type_sales
  FROM items_qualities iq
  JOIN outlets o ON iq.item_identifier = o.item_identifier
  GROUP BY o.outlet_type, iq.item_type
),

outlet_totals AS (
  SELECT 
    outlet_type,
    ROUND(SUM(item_type_sales), 2) AS total_sales
  FROM item_sales
  GROUP BY outlet_type
),

contribution AS (
  SELECT 
    i.outlet_type,
    i.item_type,
    i.item_type_sales,
    o.total_sales,
    ROUND((i.item_type_sales / o.total_sales) * 100, 2) AS sales_percent,
    RANK() OVER (PARTITION BY i.outlet_type ORDER BY i.item_type_sales DESC) AS rank_within_outlet
  FROM item_sales i
  JOIN outlet_totals o ON i.outlet_type = o.outlet_type
)
SELECT 
  outlet_type,
  item_type,
  item_type_sales,
  total_sales,
  sales_percent
FROM contribution
WHERE rank_within_outlet = 1
ORDER BY sales_percent DESC;
```
### 17.Dominance Score of Item Types
```sql
WITH type_sales_per_outlet AS (
  SELECT 
    o.outlet_identifier,
    iq.item_type,
    SUM(o.item_outlet_sales) AS total_sales
  FROM items_qualities iq
  JOIN outlets o ON iq.item_identifier = o.item_identifier
  GROUP BY o.outlet_identifier, iq.item_type
),

ranked_items AS (
  SELECT *,
         RANK() OVER (PARTITION BY outlet_identifier ORDER BY total_sales DESC) AS sales_rank
  FROM type_sales_per_outlet
),

top_items AS (
  SELECT item_type, COUNT(*) AS top_count
  FROM ranked_items
  WHERE sales_rank = 1
  GROUP BY item_type
),

total_outlets AS (
  SELECT COUNT(DISTINCT outlet_identifier) AS outlet_count FROM outlets
)

SELECT 
  t.item_type,
  t.top_count,
  o.outlet_count,
  ROUND(t.top_count / o.outlet_count, 2) AS dominance_score
FROM top_items t
CROSS JOIN total_outlets o
ORDER BY dominance_score DESC;
```
### 18.Price Band Dependency by Outlet Size
```sql
WITH banded_sales AS (
  SELECT 
    CASE 
      WHEN TRIM(o.outlet_size) = '' OR o.outlet_size IS NULL THEN 'Unknown'
      ELSE o.outlet_size
    END AS outlet_size,
    CASE 
      WHEN e.item_mrp IS NULL THEN 'Unknown'
      WHEN e.item_mrp < 50 THEN '0-50'
      WHEN e.item_mrp < 100 THEN '50-100'
      WHEN e.item_mrp < 150 THEN '100-150'
      WHEN e.item_mrp < 200 THEN '150-200'
      WHEN e.item_mrp < 250 THEN '200-250'
      WHEN e.item_mrp <= 300 THEN '250-300'
      ELSE 'Above 300'
    END AS mrp_bin,

    SUM(o.item_outlet_sales) AS total_sales

  FROM items_economics e
  JOIN outlets o ON e.item_identifier = o.item_identifier
  GROUP BY outlet_size, mrp_bin
),

sales_with_pct AS (
  SELECT 
    outlet_size,
    mrp_bin,
    total_sales,
    SUM(total_sales) OVER (PARTITION BY outlet_size) AS total_sales_by_size
  FROM banded_sales
)

SELECT 
  outlet_size,
  mrp_bin,
  total_sales,
  ROUND((total_sales / total_sales_by_size) * 100, 2) AS percent_share
FROM sales_with_pct
ORDER BY outlet_size, mrp_bin;
```

##### 📝 Notice

All data used in this project is synthetic and has been generated or modified for educational purposes. This project is **not affiliated with Blinkit** or any real-world grocery delivery company. Any similarity to actual products, outlets, customers, or business metrics is purely coincidental.

This case study is intended solely to showcase SQL problem-solving and analytical capabilities in a simulated retail context.

