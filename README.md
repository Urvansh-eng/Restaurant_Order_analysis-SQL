# 🍽️ Restaurant Orders SQL Analysis

## Objective

In this project, I designed and built an end-to-end SQL analytics workflow that consists of several stages:
1. Built a relational database in MySQL from a raw restaurant orders dataset.
2. Explored the menu data to understand pricing and category structure.
3. Analyzed order-level patterns such as order size and date range.
4. Joined orders with menu items to uncover item popularity, category performance, and top-spending orders.

As this is a self-directed SQL learning project, my emphasis is on writing clear, business-question-driven queries — from simple aggregations to joins and subqueries — against a realistic relational dataset.

The sections below explain additional details on the data, techniques, and files used.

## Table of Contents

- [Dataset Used](#dataset-used)
- [Technologies](#technologies)
- [Database Schema](#database-schema)
- [Data Analysis Workflow](#data-analysis-workflow)
- [Step 1: Database Setup](#step-1-database-setup)
- [Step 2: Menu Analysis](#step-2-menu-analysis)
- [Step 3: Order Analysis](#step-3-order-analysis)
- [Step 4: Order & Item Analysis](#step-4-order--item-analysis)
- [Bonus Queries](#bonus-queries)
- [Key Insights](#key-insights)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [What I Learned](#what-i-learned)

## Dataset Used

This project uses the **Restaurant Orders** dataset from [Maven Analytics](https://mavenanalytics.io) — a free sample dataset representing a quarter's worth of orders from a fictitious international-cuisine restaurant.

- **Records:** 12,266 across 2 tables
- **Fields:** 8
- **Time period:** January 1 – March 31, 2023 (Q1)
- **Tags:** Food & Beverage, Business, Time Series

## Technologies

The following tools and techniques were used to build this project:
- Database: MySQL
- SQL skills: Aggregations, `GROUP BY` / `HAVING`, `JOIN`s, Subqueries, Sorting & Filtering, `LIMIT`, `ROUND`

## Database Schema

**`order_details`** — one row per item within an order

| Column | Type | Notes |
|---|---|---|
| order_details_id | SMALLINT | Primary key |
| order_id | SMALLINT | Groups items into a single order |
| order_date | DATE | |
| order_time | TIME | |
| item_id | SMALLINT | Foreign key → `menu_items.menu_item_id` (nullable) |

**`menu_items`** — one row per menu item

| Column | Type | Notes |
|---|---|---|
| menu_item_id | SMALLINT | Primary key |
| item_name | VARCHAR(45) | |
| category | VARCHAR(45) | American, Asian, Mexican, Italian |
| price | DECIMAL(5,2) | |

## Data Analysis Workflow

```text
Raw Restaurant Orders Data
   ↓
Database Setup (MySQL schema + data load)
   ↓
Menu Analysis (pricing, categories)
   ↓
Order Analysis (order size, date range)
   ↓
Order & Item Analysis (joins, top items, top spenders)
   ↓
Business Insights
```

Files used at each stage:
- Step 1: [`create_restaurant_db.sql`](./create_restaurant_db.sql)
- Step 2: [`restaurant_menu_analysis.sql`](./restaurant_menu_analysis.sql)
- Step 3: [`order_analysis.sql`](./order_analysis.sql)
- Step 4: [`order_item_analysis.sql`](./order_item_analysis.sql)

## Step 1: Database Setup

[`create_restaurant_db.sql`](./create_restaurant_db.sql) creates the `restaurant_db` schema, defines the `order_details` and `menu_items` tables, and loads the raw dataset (12,234 order-line records and 32 menu items).

## Step 2: Menu Analysis

[`restaurant_menu_analysis.sql`](./restaurant_menu_analysis.sql) explores the menu itself:

```sql
-- 1. View the menu_items table.
SELECT * FROM menu_items;

-- 2. Find the number of items on the menu.
SELECT COUNT(menu_item_id) FROM menu_items;

-- 3. What are the least and most expensive items on the menu?
SELECT * FROM menu_items ORDER BY price;
SELECT * FROM menu_items ORDER BY price DESC;

-- 4. How many Italian dishes are on the menu?
SELECT COUNT(*) FROM menu_items WHERE category = 'Italian';

-- 5. What are the least and most expensive Italian dishes on the menu?
SELECT * FROM menu_items WHERE category = 'Italian' ORDER BY price;
SELECT * FROM menu_items WHERE category = 'Italian' ORDER BY price DESC;

-- 6. How many dishes are in each category?
SELECT COUNT(*), category FROM menu_items GROUP BY category;

-- 7. What is the average dish price within each category?
SELECT ROUND(AVG(price), 2), category FROM menu_items GROUP BY category;
```

## Step 3: Order Analysis

[`order_analysis.sql`](./order_analysis.sql) looks at order-level patterns:

```sql
-- 1. View the order_details table.
SELECT * FROM order_details;

-- 2. What is the date range of the table?
SELECT MIN(order_date), MAX(order_date) FROM order_details;

-- 3. How many orders were made within this date range?
SELECT COUNT(DISTINCT order_id) FROM order_details;

-- 4. How many items were ordered within this date range?
SELECT COUNT(order_id) FROM order_details;

-- 5. Which orders had the most number of items?
SELECT order_id, COUNT(item_id) AS number_of_items
FROM order_details
GROUP BY order_id
ORDER BY number_of_items DESC
LIMIT 10;

-- 6. How many orders had more than 12 items?
SELECT COUNT(*) FROM (
  SELECT order_id, COUNT(item_id) AS number_of_items
  FROM order_details
  GROUP BY order_id
  HAVING number_of_items > 12
) AS num_orders;
```

## Step 4: Order & Item Analysis

[`order_item_analysis.sql`](./order_item_analysis.sql) joins the two tables together to connect order behavior with menu items:

```sql
-- 1. Combine the menu_items and order_details tables into a single table.
SELECT *
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id;

-- 2. What were the least and most ordered items? What categories were they in?
SELECT item_name, COUNT(order_details_id) AS num_purchases
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
GROUP BY item_name
ORDER BY num_purchases DESC;

-- 3. What were the top 5 orders that spent the most money?
SELECT order_id, SUM(price) AS total_spend
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
GROUP BY order_id
ORDER BY total_spend DESC
LIMIT 5;

-- 4. View the details of the highest spend order. What insights can you gather from the results?
SELECT category, COUNT(item_id) AS num_items
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
WHERE order_id = 440
GROUP BY category;

-- 5. View the details of the top 5 highest spend orders. What insights can you gather from the results?
SELECT order_id, category, COUNT(item_id) AS num_items
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
WHERE order_id IN (440, 2075, 1957, 330, 2675)
GROUP BY order_id, category;
```

## Bonus Queries

A few additional business questions I extended the analysis with:

```sql
-- 1. What is the total revenue generated over the quarter?
SELECT ROUND(SUM(price), 2) AS total_revenue
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id;

-- 2. Which category generates the most revenue?
SELECT category, ROUND(SUM(price), 2) AS category_revenue
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
GROUP BY category
ORDER BY category_revenue DESC;

-- 3. What is the average order value?
SELECT ROUND(SUM(price) / COUNT(DISTINCT order_id), 2) AS avg_order_value
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id;

-- 4. How does revenue trend month over month?
SELECT MONTHNAME(order_date) AS month, ROUND(SUM(price), 2) AS monthly_revenue
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
GROUP BY MONTH(order_date), MONTHNAME(order_date)
ORDER BY MONTH(order_date);

-- 5. Which day of the week gets the most orders?
SELECT DAYNAME(order_date) AS day_of_week, COUNT(DISTINCT order_id) AS num_orders
FROM order_details
GROUP BY DAYNAME(order_date), DAYOFWEEK(order_date)
ORDER BY num_orders DESC;

-- 6. What is the busiest hour of the day, by items ordered?
SELECT HOUR(order_time) AS hour_of_day, COUNT(*) AS items_ordered
FROM order_details
WHERE item_id IS NOT NULL
GROUP BY hour_of_day
ORDER BY items_ordered DESC;

-- 7. How many orders contain a missing (NULL) item?
SELECT COUNT(DISTINCT order_id) AS orders_with_missing_item
FROM order_details
WHERE item_id IS NULL;

-- 8. What percentage of total revenue does each category represent?
SELECT
  category,
  ROUND(SUM(price), 2) AS category_revenue,
  ROUND(100 * SUM(price) / (
    SELECT SUM(price)
    FROM order_details od2
    LEFT JOIN menu_items mi2 ON od2.item_id = mi2.menu_item_id
  ), 2) AS pct_of_total_revenue
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
GROUP BY category
ORDER BY pct_of_total_revenue DESC;
```

## Key Insights

- **Total revenue** for the quarter is **$159,217.90** across **5,370 orders**, for an average order value of **$29.65**.
- **Italian is the top-revenue category** (~$49,463), narrowly ahead of Asian (~$46,721), followed by Mexican (~$34,797) and American (~$28,238) — largely reflecting Italian's higher average dish price (~$16.75 vs. ~$10–13 for the other categories).
- **Hamburger** and **Edamame** are the most frequently ordered items (622 and 620 orders respectively), while **Chicken Tacos** is the least ordered (123 orders).
- **Shrimp Scampi** ($19.95) is the most expensive menu item; **Edamame** ($5.00) is the cheapest.
- **Monday is the busiest day** for order volume (885 orders), while **Wednesday is the quietest** (682 orders).
- **Lunch (12–1pm) and dinner (5–7pm) are the two clear peak windows** for items ordered, with noon the single busiest hour.
- **23 orders** contained more than 12 items — the largest orders (5 orders tied at 14 items each) all spent well over $150, roughly 5x the average order value.
- Revenue is fairly stable month to month (~$50.8K–$54.6K across January, February, and March), with no single month dramatically outperforming the others.
- **137 order-line records have a missing (`NULL`) item ID** — worth flagging as a data-quality issue if this dataset is used for further modeling.

## Project Structure

```text
📦 Restaurant_Orders_SQL_Analysis
│
├── 📁 assets
│   └── 🖼️ maven_dataset_card.png
├── 🗄️ create_restaurant_db.sql
├── 🗄️ restaurant_menu_analysis.sql
├── 🗄️ order_analysis.sql
├── 🗄️ order_item_analysis.sql
└── 📄 README.md
```

## How to Run

1. **Clone the repository / download the project files.**
2. **Open a MySQL client** (MySQL Workbench or the CLI).
3. **Run the setup script** to create the schema and load the data:
   ```bash
   mysql -u your_username -p < create_restaurant_db.sql
   ```
4. **Run the analysis scripts** in order, or pick individual queries to explore:
   - `restaurant_menu_analysis.sql`
   - `order_analysis.sql`
   - `order_item_analysis.sql`
5. Try the [Bonus Queries](#bonus-queries) or write your own against the `order_details` and `menu_items` tables.

## What I Learned

This project helped strengthen my practical understanding of:
- Designing and loading a simple relational schema in MySQL
- Writing aggregations with `GROUP BY` and filtering aggregated results with `HAVING`
- Using `LEFT JOIN` to combine transactional and reference tables
- Answering business questions with subqueries (including correlated-style percentage-of-total calculations)
- Spotting data-quality issues (missing item IDs) while exploring a real-shaped dataset
- Framing SQL output as business insights rather than just raw query results

---

*Skills demonstrated: `SQL` `MySQL` `Joins` `Aggregations` `Subqueries` `Data Analysis`*
