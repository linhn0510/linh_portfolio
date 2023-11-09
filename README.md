# SQL Analysis and Tableau Visualisation for Toy Sales Dataset
The goal of this project is to analyse toy sales data in Mexico to derive actionable insights that can inform business strategies and decision-making. By leveraging various data analysis techniques, we aim to uncover patterns, trends, and key factors influencing the toy market in Mexico.

Examples of insights: 
1. Weekday that generated most sales 
2. Best-selling products
3. Busiest location (store location, store name, city) 

# SQL
### 1. Which year generated more sales and change percentage? 
````sql
SE
````
### 2. Which weekday sold the most by units
````sql
SELECT DATENAME(dw, Date) AS day, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(dw, Date)
ORDER BY total_quantity DESC;
````
### 3. Which month sold the most by units
````sql
SELECT DATENAME(year, Date) AS year, DATENAME(month, Date) AS month, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(year, Date), DATENAME(month, Date)
ORDER BY year, total_quantity DESC
````
### 4. Which quarter sold the most by units 
````sql
SELECT DATENAME(year, Date) AS year, DATENAME(quarter, Date) AS quarter, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(year, Date), DATENAME(quarter, Date)
ORDER BY year, total_quantity DESC;
````
### 5. Which product had the most significant swift in sales 
````sql
WITH yearly_sales AS (
SELECT s.Product_ID as product_id, p.Product_Name as product_name, YEAR(s.Date) AS year, CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
FROM   products$ AS p INNER JOIN
              sales$ AS s ON p.Product_ID = s.Product_ID
WHERE YEAR(s.Date) in ('2017','2018')
GROUP BY YEAR(s.Date), s.Product_ID, p.Product_Name
)

SELECT current_year.product_id, current_year.product_name, current_year.total_revenue AS sales_2018, previous_year.total_revenue AS sales_2017, CONCAT(CAST((current_year.total_revenue - previous_year.total_revenue) * 100 / previous_year.total_revenue AS decimal(10, 2)), '%') AS sales_change
FROM     yearly_sales AS current_year INNER JOIN
                yearly_sales AS previous_year 
		 ON current_year.product_id = previous_year.product_id 
		 AND current_year.year = 2018 
		 AND previous_year.year = 2017
ORDER BY (current_year.total_revenue - previous_year.total_revenue) DESC;
````
### 6. Top 10 best-selling products
````sql
SELECT TOP (10) s.Product_ID, p.Product_Name, CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
FROM     products$ AS p INNER JOIN
                sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY s.Product_ID, p.Product_Name
ORDER BY total_revenue DESC; 
````
### 7. Top 10 worst-selling products 
````sql
SELECT TOP (10) s.Product_ID, p.Product_Name, CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY s.Product_ID, p.Product_Name
ORDER BY total_revenue;
````
### 8. Sales distribution by product category
````sql
WITH category_sales AS (SELECT p.Product_Category as product_category, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS total_sales
FROM   products$ AS p INNER JOIN
       sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY p.Product_Category )

SELECT product_category, total_sales, CONCAT(CAST(total_sales/ SUM(total_sales) OVER () * 100 AS decimal(10, 2)), '%')AS sales_percentage
FROM category_sales
ORDER BY sales_percentage DESC;
````
### 9. Which 5 stores generated the most revenue? 
````sql
SELECT TOP 5 st.Store_Name AS store_name, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS store_sales
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID INNER JOIN
         stores$ AS st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_Name
ORDER BY store_sales DESC
````
### 10. Which 5 cities generated the most revenue? Change percentage from 2022 to 2023? 
````sql
WITH sales_yearly AS (
    SELECT 
        YEAR(s.Date) AS year,
        st.Store_City AS store_city,
        CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS store_sales
    FROM     
        products$ AS p 
    INNER JOIN
        sales$ AS s ON p.Product_ID = s.Product_ID 
    INNER JOIN
        stores$ AS st ON s.Store_ID = st.Store_ID
    GROUP BY 
        YEAR(s.Date), st.Store_City
)

SELECT 
    TOP 5 store_city, 
    SUM(CASE WHEN year = '2017' THEN store_sales ELSE 0 END) AS sales_2017, 
    SUM(CASE WHEN year = '2018' THEN store_sales ELSE 0 END) AS sales_2018, 
    (SUM(CASE WHEN year = '2018' THEN store_sales ELSE 0 END) - SUM(CASE WHEN year = '2017' THEN store_sales ELSE 0 END)) * 100.0 / NULLIF(SUM(CASE WHEN year = '2017' THEN store_sales ELSE 0 END), 0) AS sales_change_percentage
FROM 
    sales_yearly
GROUP BY 
    store_city
ORDER BY 
    sales_change_percentage;
````




