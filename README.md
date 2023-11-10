# SQL Analysis and Tableau Visualisation for Toy Sales Dataset
This project aims to analyse toy sales data in Mexico (Jan 2017- Sep 2018)to derive actionable insights that can inform business strategies and decision-making. By leveraging various data analysis techniques, we aim to uncover patterns, trends, and key factors influencing the toy market in Mexico.

# SQL
### 1. Which weekday sold the most by units
````sql
SELECT DATENAME(dw, Date) AS day, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(dw, Date)
ORDER BY total_quantity DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/51dd82b7-1b64-4e96-940d-5bd159110440)

Toys were sold the most in units on Sunday and Saturday, people are more likely to do toy shopping on the weekends  
### 2. Which month sold the most by units
````sql
SELECT DATENAME(year, Date) AS year, DATENAME(month, Date) AS month, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(year, Date), DATENAME(month, Date)
ORDER BY year, total_quantity DESC
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/24d90923-1f17-49cd-9486-0e8f13b8344d)

2017: Toys were sold the most in units in December 

2018: Toys were sold the most in units in March 
### 3. Which quarter sold the most by units 
````sql
SELECT DATENAME(year, Date) AS year, DATENAME(quarter, Date) AS quarter, COUNT(*) AS total_quantity
FROM     sales$
GROUP BY DATENAME(year, Date), DATENAME(quarter, Date)
ORDER BY year, total_quantity DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/e3c17e28-35e3-4fe4-9e81-7022b1c64367)

2017: Toys were sold the most in units in quarter 4 

2018: Toys were sold the most in units in quarter 2
### 4. Which product had the most significant swift in sales during summer? 
````sql
WITH yearly_sales AS (
    SELECT s.Product_ID AS product_id,
        p.Product_Name AS product_name,
        YEAR(s.Date) AS year,
        MONTH(s.Date) AS month,
        CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
    FROM products$ AS p
        INNER JOIN sales$ AS s ON p.Product_ID = s.Product_ID
    WHERE YEAR(s.Date) IN (2017, 2018)
    GROUP BY YEAR(s.Date), MONTH(s.Date), s.Product_ID, p.Product_Name)

SELECT y2.product_name,
    SUM(CASE WHEN y1.year = 2017 AND y1.month IN (6, 7, 8) THEN y1.total_revenue ELSE 0 END) AS summer_2017,
    SUM(CASE WHEN y2.year = 2018 AND y2.month IN (6, 7, 8) THEN y2.total_revenue ELSE 0 END) AS summer_2018,
    COALESCE(
        (SUM(CASE WHEN y2.year = 2018 AND y2.month IN (6, 7, 8) THEN y2.total_revenue ELSE 0 END) - 
         SUM(CASE WHEN y1.year = 2017 AND y1.month IN (6, 7, 8) THEN y1.total_revenue ELSE 0 END)) *100/
         NULLIF(SUM(CASE WHEN y1.year = 2017 AND y1.month IN (6, 7, 8) THEN y1.total_revenue ELSE 0 END), 0),
         0
    ) AS sales_change
FROM yearly_sales y1
    JOIN yearly_sales y2 ON y1.product_id = y2.product_id AND y1.year = 2017 AND y2.year = 2018
GROUP BY y2.product_name
ORDER BY sales_change;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/8647cafd-d7b3-497a-be8a-a2b6c6437667)

![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/ec4db92c-01fe-4da7-bfc7-2449f5ecabc0)


Jenga generated significantly less by 64% in sales from 2017 till 2018 while Magic Sand generated the highest increase in sales, by %
### 5. Top 10 best-selling products
````sql
SELECT TOP (10) s.Product_ID, p.Product_Name, CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
FROM     products$ AS p INNER JOIN
                sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY s.Product_ID, p.Product_Name
ORDER BY total_revenue DESC; 
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/662a8f4f-2f98-4cdc-8f25-31de655f451d)

Among top 10, Lego Bricks was the best-selling product
### 6. Top 10 worst-selling products 
````sql
SELECT TOP (10) s.Product_ID, p.Product_Name, CAST(SUM(p.Product_Price * s.Units) AS decimal(10, 2)) AS total_revenue
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY s.Product_ID, p.Product_Name
ORDER BY total_revenue;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/dd4f6fc4-55c4-4853-a0b5-33ff36867b5f)

Among top 10, Uno Card Game was the worst-selling product
### 7. Sales distribution by product category
````sql
WITH category_sales AS (SELECT p.Product_Category as product_category, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS total_sales
FROM   products$ AS p INNER JOIN
       sales$ AS s ON p.Product_ID = s.Product_ID
GROUP BY p.Product_Category )

SELECT product_category, total_sales, CONCAT(CAST(total_sales/ SUM(total_sales) OVER () * 100 AS decimal(10, 2)), '%')AS sales_percentage
FROM category_sales
ORDER BY sales_percentage DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/7dc26a89-0f7d-41cf-aa9d-b3ecf144dd6a)

Toys accounted for the most amount of sales
### 8. Sales distribution by store location
````sql
WITH category_sales AS (SELECT st.Store_Location as store_location, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS total_sales
FROM   products$ AS p
       JOIN sales$ AS s ON p.Product_ID = s.Product_ID
       JOIN stores$ as st on s.Store_ID = st.Store_ID
GROUP BY st.Store_Location )

SELECT store_location, total_sales, CONCAT(CAST(total_sales/ SUM(total_sales) OVER () * 100 AS decimal(10, 2)), '%')AS sales_percentage
FROM category_sales
ORDER BY sales_percentage DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/b6d30f67-3cc7-4cd5-94c1-b255108a9b2d)

Stores in downtown brought in more than half of the sales 
### 9. Which 5 stores generated the most/least revenue? 
````sql
SELECT TOP 5 st.Store_Name AS store_name, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS store_sales
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID INNER JOIN
         stores$ AS st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_Name
ORDER BY store_sales DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/dbdeaf89-f633-4af6-a321-83e86c5e244f)
````sql
SELECT TOP 5 st.Store_Name AS store_name, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS store_sales
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID INNER JOIN
         stores$ AS st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_Name
ORDER BY store_sales;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/5d10d957-b290-4ea1-8554-03ec5edcae15)

### 10. Which 5 cities generated the most/least revenue? 
````sql
SELECT TOP 5 st.Store_City AS city, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS city_sales
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID INNER JOIN
         stores$ AS st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_City
ORDER BY city_sales DESC;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/ecf8b7c2-8c8c-4908-b230-adcff3198794)
````sql
SELECT TOP 5 st.Store_City AS city, CAST(SUM(s.Units * p.Product_Price) AS decimal(10, 2)) AS city_sales
FROM     products$ AS p INNER JOIN
         sales$ AS s ON p.Product_ID = s.Product_ID INNER JOIN
         stores$ AS st ON s.Store_ID = st.Store_ID
GROUP BY st.Store_City
ORDER BY city_sales;
````
![image](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/f4af0943-67db-447f-a4af-5231816f7b43)
# Tableau
[Link to Dashboard](https://public.tableau.com/app/profile/linh.nguyen6533/viz/ModelToySales/Dashboard1)

![Mexico_Toy_Sales](https://github.com/linhn0510/linhnguyen_portfolio/assets/125606128/a53b0860-0840-44a0-bc3f-8e61e8a27c35)
