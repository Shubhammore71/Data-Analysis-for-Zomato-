
# Zomato Data Analysis: A Deep Dive with SQL

## Overview

This project is a comprehensive analysis of a Zomato-like food delivery service's database. It showcases advanced SQL skills to solve 20 real-world business problems, transforming raw data into actionable insights. The primary goal is to demonstrate proficiency in complex query writing, data modeling, and business intelligence, skills crucial for a Product Analyst role.

The analysis covers key business areas including:

  * **Customer Behavior and Segmentation:** Understanding customer ordering patterns, identifying high-value customers, and analyzing churn.
  * **Restaurant Performance:** Evaluating restaurant revenue, popularity, and operational metrics.
  * **Operational Efficiency:** Analyzing delivery times, rider performance, and sales trends.

This repository contains the complete project, from database setup and data cleaning to the final SQL queries and their business implications.

## Table of Contents

1.  [Project Structure](https://www.google.com/search?q=%23project-structure)
2.  [Tools and Technologies](https://www.google.com/search?q=%23tools-and-technologies)
3.  [Data Model and ERD](https://www.google.com/search?q=%23data-model-and-erd)
4.  [Database Setup and Initialization](https://www.google.com/search?q=%23database-setup-and-initialization)
5.  [Data Cleaning](https://www.google.com/search?q=%23data-cleaning)
6.  [Business Problems and SQL Solutions](https://www.google.com/search?q=%23business-problems-and-sql-solutions)
7.  [Conclusion](https://www.google.com/search?q=%23conclusion)
8.  [Contributors](https://www.google.com/search?q=%23contributors)
9.  [Disclaimer](https://www.google.com/search?q=%23disclaimer)

## Project Structure

The project is organized as follows:

  - **`data/`**: Contains all the raw `.csv` files used for populating the database (`customers.csv`, `orders.csv`, `restaurants.csv`, `riders.csv`, `deliveries.csv`).
  - **`erd.png`**: The Entity-Relationship Diagram illustrating the database schema.
  - **`database_setup.sql`**: SQL script for creating the database schema, including all tables and relationships.
  - **`20_business_problems_solution.sql`**: A single SQL file containing the queries and solutions for all 20 business problems.

## Tools and Technologies

  * **Database:** PostgreSQL
  * **Language:** SQL
  * **Key SQL Concepts Applied:**
      * Joins (INNER, LEFT)
      * Common Table Expressions (CTEs)
      * Window Functions (`RANK()`, `DENSE_RANK()`, `LAG()`)
      * Aggregate Functions (`COUNT()`, `SUM()`, `AVG()`)
      * Date/Time Functions (`EXTRACT`, `TO_CHAR`)
      * Subqueries
      * Conditional Logic (`CASE` statements)

## Data Model and ERD

The database follows a schema designed to capture the core entities of a food delivery platform: customers, restaurants, orders, riders, and deliveries. The relationships between these entities are visualized in the ERD below.
![ERD](erd.png)

## Database Setup and Initialization

The database `zomato_db` is created and populated using the provided SQL scripts.

### 1\. Dropping Existing Tables

To ensure a clean setup, we first drop any existing tables.

```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;
```

### 2\. Creating Tables

The tables are created with appropriate primary keys, foreign keys, and constraints to maintain data integrity.

```sql
-- 2. Creating Tables
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    reg_date DATE
);

CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    sign_up DATE
);

CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(255),
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(20) DEFAULT 'Pending',
    delivery_time TIME,
    rider_id INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);
```

## Data Cleaning

Data integrity is paramount for accurate analysis. A key step in this project was handling missing values. For example, `total_amount` in the `orders` table could have nulls, which would skew financial calculations. This was addressed by updating `NULL` values to `0`.

```sql
-- Example: Handling NULLs in total_amount
UPDATE orders
SET total_amount = COALESCE(total_amount, 0);
```

## Business Problems and SQL Solutions

Here are the 20 business problems solved using SQL, along with the business insight behind each question.

-----

### 1\. Top 5 Dishes for a Specific Customer

  * **Business Question:** What are the top 5 most frequently ordered dishes by the customer "Arjun Mehta" in the last year?
  * **Insight:** This helps in personalizing user experience, offering targeted promotions, and understanding individual customer preferences.

<!-- end list -->

```sql
SELECT
	customer_name,
	dishes,
	total_orders
FROM
	(SELECT
		c.customer_id,
		c.customer_name,
		o.order_item as dishes,
		COUNT(*) as total_orders,
		DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as rank
	FROM orders as o
	JOIN
	customers as c
	ON c.customer_id = o.customer_id
	WHERE
		o.order_date >= CURRENT_DATE - INTERVAL '1 Year'
		AND
		c.customer_name = 'Arjun Mehta'
	GROUP BY 1, 2, 3
	ORDER BY 1, 4 DESC) as t1
WHERE rank <= 5;
```

### 2\. Popular Time Slots for Orders

  * **Business Question:** What are the most popular time slots (in 2-hour intervals) for placing orders?
  * **Insight:** This information is vital for resource management, such as ensuring rider availability during peak hours and planning restaurant kitchen capacity.

<!-- end list -->

```sql
SELECT
    CASE
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM Orders
GROUP BY time_slot
ORDER BY order_count DESC;
```

### 3\. Average Order Value (AOV) for Frequent Customers

  * **Business Question:** What is the average order value (AOV) for customers who have placed more than 750 orders?
  * **Insight:** Identifying the spending habits of power users helps in designing loyalty programs and premium services.

<!-- end list -->

```sql
SELECT
	c.customer_name,
	AVG(o.total_amount) as aov
FROM orders as o
	JOIN customers as c
	ON c.customer_id = o.customer_id
GROUP BY 1
HAVING  COUNT(order_id) > 750;
```

### 4\. Identifying High-Value Customers

  * **Business Question:** Which customers have spent more than 100,000 in total?
  * **Insight:** These are the most valuable customers. The business should focus on retaining them through exclusive offers and excellent service.

<!-- end list -->

```sql
SELECT
	c.customer_name,
	SUM(o.total_amount) as total_spent
FROM orders as o
	JOIN customers as c
	ON c.customer_id = o.customer_id
GROUP BY 1
HAVING SUM(o.total_amount) > 100000;
```

### 5\. Orders Without Delivery

  * **Business Question:** Which orders were placed but never delivered, and at which restaurants?
  * **Insight:** This query identifies operational failures. A high number of undelivered orders for a restaurant could indicate issues with order acceptance, rider assignment, or system errors, which need immediate attention.

<!-- end list -->

```sql
SELECT
	r.restaurant_name,
	r.city,
	COUNT(o.order_id) as cnt_not_delivered_orders
FROM orders as o
LEFT JOIN
restaurants as r
ON r.restaurant_id = o.restaurant_id
LEFT JOIN
deliveries as d
ON d.order_id = o.order_id
WHERE d.delivery_id IS NULL
GROUP BY 1, 2
ORDER BY 3 DESC;
```

### 6\. Restaurant Revenue Ranking by City

  * **Business Question:** How do restaurants rank by total revenue within their respective cities over the last year?
  * **Insight:** This helps identify top-performing restaurants in each market, which can be promoted as "top-rated" or "most popular." It also helps the sales team identify potential partners.

<!-- end list -->

```sql
WITH ranking_table AS (
	SELECT
		r.city,
		r.restaurant_name,
		SUM(o.total_amount) as revenue,
		RANK() OVER(PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC) as rank
	FROM orders as o
	JOIN
	restaurants as r
	ON r.restaurant_id = o.restaurant_id
	WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
	GROUP BY 1, 2
)
SELECT
	*
FROM ranking_table
WHERE rank = 1;
```

### 7\. Most Popular Dish by City

  * **Business Question:** What is the most ordered dish in each city?
  * **Insight:** Understanding regional food preferences allows for city-specific marketing campaigns and helps new restaurants decide which popular cuisines to focus on.

<!-- end list -->

```sql
SELECT *
FROM
(
    SELECT
        r.city,
        o.order_item as dish,
        COUNT(order_id) as total_orders,
        RANK() OVER(PARTITION BY r.city ORDER BY COUNT(order_id) DESC) as rank
    FROM orders as o
    JOIN
    restaurants as r
    ON r.restaurant_id = o.restaurant_id
    GROUP BY 1, 2
) as t1
WHERE rank = 1;
```

### 8\. Customer Churn Analysis

  * **Business Question:** Which customers placed orders in 2023 but have not placed any in 2024?
  * **Insight:** This query identifies churned customers. The marketing team can target this segment with "We miss you" campaigns or special discounts to win them back.

<!-- end list -->

```sql
SELECT DISTINCT customer_id FROM orders
WHERE
	EXTRACT(YEAR FROM order_date) = 2023
	AND
	customer_id NOT IN
					(SELECT DISTINCT customer_id FROM orders
					WHERE EXTRACT(YEAR FROM order_date) = 2024);
```

### 9\. Year-over-Year Cancellation Rate

  * **Business Question:** How does the order cancellation rate for each restaurant in the current year compare to the previous year?
  * **Insight:** A rising cancellation rate is a red flag, indicating potential issues with a restaurant's service (e.g., long wait times, low stock). This metric helps in performance reviews with restaurant partners.

<!-- end list -->

```sql
WITH cancel_ratio_23 AS (
    SELECT
        o.restaurant_id,
        (COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END)::numeric / COUNT(o.order_id)::numeric) * 100 AS cancel_ratio
    FROM orders AS o
    LEFT JOIN deliveries AS d ON o.order_id = d.order_id
    WHERE EXTRACT(YEAR FROM o.order_date) = 2023
    GROUP BY o.restaurant_id
),
cancel_ratio_24 AS (
    SELECT
        o.restaurant_id,
        (COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END)::numeric / COUNT(o.order_id)::numeric) * 100 AS cancel_ratio
    FROM orders AS o
    LEFT JOIN deliveries AS d ON o.order_id = d.order_id
    WHERE EXTRACT(YEAR FROM o.order_date) = 2024
    GROUP BY o.restaurant_id
)
SELECT
    r.restaurant_name,
    COALESCE(curr.cancel_ratio, 0) AS current_year_cancel_ratio,
    COALESCE(prev.cancel_ratio, 0) AS last_year_cancel_ratio
FROM restaurants r
LEFT JOIN cancel_ratio_24 curr ON r.restaurant_id = curr.restaurant_id
LEFT JOIN cancel_ratio_23 prev ON r.restaurant_id = prev.restaurant_id;
```

### 10\. Rider Average Delivery Time

  * **Business Question:** What is the average time it takes for each rider to deliver an order?
  * **Insight:** This is a key performance indicator (KPI) for riders. It helps in identifying both efficient and underperforming riders, allowing for training or incentive programs.

<!-- end list -->

```sql
SELECT
    d.rider_id,
    AVG(EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
	CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE
	INTERVAL '0 day' END)))/60 as avg_delivery_time_minutes
FROM orders AS o
JOIN deliveries AS d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY d.rider_id
ORDER BY 2;
```

### 11\. Monthly Restaurant Growth

  * **Business Question:** What is the month-over-month growth rate in delivered orders for each restaurant?
  * **Insight:** Tracks the performance and trajectory of restaurant partners. A high-growth restaurant could be a candidate for a strategic partnership, while a declining one may need support.

<!-- end list -->

```sql
WITH monthly_orders AS (
    SELECT
        o.restaurant_id,
        TO_CHAR(o.order_date, 'YYYY-MM') as order_month,
        COUNT(o.order_id) as current_month_orders
    FROM orders as o
    JOIN deliveries as d ON o.order_id = d.order_id
    WHERE d.delivery_status = 'Delivered'
    GROUP BY 1, 2
),
growth_data AS (
    SELECT
        restaurant_id,
        order_month,
        current_month_orders,
        LAG(current_month_orders, 1, 0) OVER(PARTITION BY restaurant_id ORDER BY order_month) as prev_month_orders
    FROM monthly_orders
)
SELECT
	restaurant_id,
	order_month,
	prev_month_orders,
	current_month_orders,
	CASE
	    WHEN prev_month_orders > 0 THEN ROUND((current_month_orders::numeric - prev_month_orders::numeric) / prev_month_orders::numeric * 100, 2)
	    ELSE NULL
	END as growth_ratio_percent
FROM growth_data;
```

### 12\. Customer Segmentation (Gold/Silver)

  * **Business Question:** Segment customers into 'Gold' or 'Silver' based on whether their total spending is above or below the platform-wide average order value (AOV). What is the total revenue from each segment?
  * **Insight:** This provides a simple but effective way to segment the user base. The 'Gold' segment contributes more revenue and should be nurtured to maintain their loyalty.

<!-- end list -->

```sql
WITH customer_spending AS (
    SELECT
		customer_id,
		SUM(total_amount) as total_spent,
		COUNT(order_id) as total_orders
	FROM orders
	GROUP BY 1
),
avg_order_value AS (
    SELECT AVG(total_amount) as aov FROM orders
)
SELECT
	CASE
		WHEN cs.total_spent > av.aov THEN 'Gold'
		ELSE 'Silver'
	END as customer_segment,
	COUNT(cs.customer_id) as number_of_customers,
	SUM(cs.total_spent) as total_revenue
FROM customer_spending cs, avg_order_value av
GROUP BY 1;
```

### 13\. Rider Monthly Earnings

  * **Business Question:** Calculate each rider's total monthly earnings, assuming they earn an 8% commission on the order amount.
  * **Insight:** This query is essential for payroll and for riders to track their earnings, ensuring transparency and motivating performance.

<!-- end list -->

```sql
SELECT
	d.rider_id,
	TO_CHAR(o.order_date, 'YYYY-MM') as month,
	SUM(o.total_amount) as total_revenue_generated,
	SUM(o.total_amount) * 0.08 as riders_earning
FROM orders as o
JOIN deliveries as d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1, 2
ORDER BY 1, 2;
```

### 14\. Rider Performance Ratings

  * **Business Question:** How many 5-star, 4-star, and 3-star ratings does each rider have, based on delivery speed?
  * **Insight:** Gamifies performance for riders. This rating system provides a clear, objective measure of rider efficiency that can be tied to bonuses, incentives, and leaderboards.

<!-- end list -->

```sql
WITH delivery_times AS (
    SELECT
        d.rider_id,
        EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
        CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE INTERVAL '0 day' END
        ))/60 as delivery_took_time
    FROM orders as o
    JOIN deliveries as d ON o.order_id = d.order_id
    WHERE d.delivery_status = 'Delivered'
)
SELECT
	rider_id,
	COUNT(CASE WHEN delivery_took_time < 15 THEN 1 END) as five_star_ratings,
	COUNT(CASE WHEN delivery_took_time BETWEEN 15 AND 20 THEN 1 END) as four_star_ratings,
	COUNT(CASE WHEN delivery_took_time > 20 THEN 1 END) as three_star_ratings
FROM delivery_times
GROUP BY 1
ORDER BY 1;
```

### 15\. Peak Order Day for Each Restaurant

  * **Business Question:** What is the busiest day of the week for each restaurant?
  * **Insight:** Helps restaurants optimize their staffing and inventory management for their peak days, ensuring they can handle the demand without compromising service quality.

<!-- end list -->

```sql
WITH daily_orders AS (
	SELECT
		r.restaurant_name,
		TO_CHAR(o.order_date, 'Day') as day_of_week,
		COUNT(o.order_id) as total_orders,
		RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC) as rank
	FROM orders as o
	JOIN
	restaurants as r
	ON o.restaurant_id = r.restaurant_id
	GROUP BY 1, 2
)
SELECT
    restaurant_name,
    day_of_week,
    total_orders
FROM daily_orders
WHERE rank = 1;
```

### 16\. Customer Lifetime Value (CLV)

  * **Business Question:** What is the total revenue generated by each customer over their entire history with the platform?
  * **Insight:** CLV is a critical metric for understanding the long-term value of a customer. It informs marketing budgets for customer acquisition and retention strategies.

<!-- end list -->

```sql
SELECT
	c.customer_name,
	SUM(o.total_amount) as customer_lifetime_value
FROM orders as o
JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 17\. Monthly Sales Trends

  * **Business Question:** How do total sales for each month compare to the previous month?
  * **Insight:** This query provides a high-level view of the business's growth trajectory. It helps identify seasonality and the impact of major marketing campaigns or external events.

<!-- end list -->

```sql
WITH monthly_sales AS (
    SELECT
        TO_CHAR(order_date, 'YYYY-MM') as month,
        SUM(total_amount) as total_sale
    FROM orders
    GROUP BY 1
)
SELECT
	month,
	total_sale,
	LAG(total_sale, 1, 0) OVER(ORDER BY month) as previous_month_sale,
	total_sale - LAG(total_sale, 1, 0) OVER(ORDER BY month) as month_over_month_change
FROM monthly_sales
ORDER BY 1;
```

### 18\. Rider Efficiency Analysis

  * **Business Question:** Who are the fastest and slowest riders on average?
  * **Insight:** Identifies the outliers in rider performance. The fastest riders can be rewarded or studied to share best practices, while the slowest may require additional training or support.

<!-- end list -->

```sql
WITH riders_avg_time AS (
	SELECT
		d.rider_id,
		AVG(EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
		CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE
		INTERVAL '0 day' END))/60) as avg_time_minutes
	FROM orders as o
	JOIN deliveries as d
	ON o.order_id = d.order_id
	WHERE d.delivery_status = 'Delivered'
    GROUP BY 1
)
SELECT
	MIN(avg_time_minutes) as fastest_avg_delivery_time,
	MAX(avg_time_minutes) as slowest_avg_delivery_time
FROM riders_avg_time;
```

### 19\. Seasonal Popularity of Order Items

  * **Business Question:** How does the popularity of different food items change with the seasons?
  * **Insight:** This helps in seasonal menu planning and marketing. For example, promoting "ice cream" during summer or "hot soup" during winter can lead to higher sales.

<!-- end list -->

```sql
WITH seasonal_orders AS (
    SELECT
		order_item,
		CASE
			WHEN EXTRACT(MONTH FROM order_date) IN (3, 4, 5) THEN 'Spring'
			WHEN EXTRACT(MONTH FROM order_date) IN (6, 7, 8) THEN 'Summer'
			WHEN EXTRACT(MONTH FROM order_date) IN (9, 10, 11) THEN 'Autumn'
			ELSE 'Winter'
		END as season
	FROM orders
)
SELECT
	order_item,
	season,
	COUNT(*) as total_orders
FROM seasonal_orders
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

### 20\. City Revenue Ranking

  * **Business Question:** Rank cities based on the total revenue generated in 2023.
  * **Insight:** Identifies the most profitable markets. This helps the business decide where to allocate more resources, invest in marketing, or expand operations.

<!-- end list -->

```sql
SELECT
	r.city,
	SUM(o.total_amount) as total_revenue,
	RANK() OVER(ORDER BY SUM(o.total_amount) DESC) as city_rank
FROM orders as o
JOIN
restaurants as r
ON o.restaurant_id = r.restaurant_id
WHERE EXTRACT(YEAR FROM o.order_date) = 2023
GROUP BY 1;
```

## Conclusion

This project successfully demonstrates the power of SQL in extracting meaningful insights from a transactional database. By tackling 20 distinct business challenges, this analysis showcases a strong capability in data manipulation, problem-solving, and deriving actionable intelligence. The queries, ranging from simple aggregations to complex window functions and CTEs, provide a comprehensive view of customer behavior, restaurant performance, and operational logistics, laying the groundwork for data-driven decision-making in a fast-paced food delivery business.

## Contributors

  * Shubham More
  * Naimkhan Shaikh
  * Pratyaksh Bhayre

## Disclaimer

All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Zomato or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.
