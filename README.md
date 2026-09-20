# PL/SQL Assignment One: Sunrise Supermarket Database

| | |
|---|---|
| **Course** | Database Systems / PL/SQL |
| **Student** | Ruhumuriza Yvan Kenny |
| **Student ID** | 29179 |
| **Academic Group** | Group B |
| **Submission Date** | September 2026 |
| **RDBMS** | Oracle Database 21c Express Edition (Release 21.3.0.0.0) |
| **Client** | Oracle SQL*Plus |

---

## 1. Overview

Sunrise Supermarket is a regional grocery retailer operating in **Kigali, Musanze, Huye, and Rubavu**, selling products across three departments: **Produce, Dairy, and Bakery**.

This project builds a small relational database for the supermarket and uses SQL analytics to answer the questions management cares about:

- Where are customers located, and which registered accounts are inactive?
- What does a typical shopping basket look like across categories?
- Which customers contribute above-average revenue?
- How often do customers come back to buy again?
- How does storewide revenue accumulate over time?

## 2. Prerequisites

- Oracle Database 21c Express Edition
- Pluggable database `XEPDB1` on the default port `1521`
- SQL*Plus command line client

## 3. Setup

**1. Connect to the pluggable database**

```bash
sqlplus myuser/password@localhost:1521/XEPDB1
```

**2. Configure display settings** (prevents column clipping and line wrapping)

```sql
SET LINESIZE 200;
SET PAGESIZE 50;
COLUMN customer_name FORMAT A20;
COLUMN product_name FORMAT A22;
COLUMN city FORMAT A15;
COLUMN category FORMAT A12;
```

**3. Deploy the schema and data**

Run the `CREATE TABLE` statements (Section 4), then the `INSERT` statements from the report (Section 3.2), and finish with `COMMIT;`.

> **Re-running the script?** Drop tables in child-to-parent order, or use `CASCADE CONSTRAINTS PURGE`:
> `order_items` → `orders` → `products` → `customers`

## 4. Database Schema

```
customers (1) ────< orders (1) ────< order_items >──── (1) products
```

```sql
CREATE TABLE customers (
  customer_id   NUMBER PRIMARY KEY,
  customer_name VARCHAR2(100),
  email         VARCHAR2(100),
  city          VARCHAR2(50)
);

CREATE TABLE products (
  product_id   NUMBER PRIMARY KEY,
  product_name VARCHAR2(100),
  category     VARCHAR2(50),
  price        NUMBER(10,2)
);

CREATE TABLE orders (
  order_id    NUMBER PRIMARY KEY,
  customer_id NUMBER REFERENCES customers(customer_id),
  order_date  DATE
);

CREATE TABLE order_items (
  order_item_id NUMBER PRIMARY KEY,
  order_id      NUMBER REFERENCES orders(order_id),
  product_id    NUMBER REFERENCES products(product_id),
  quantity      NUMBER
);
```

### Dataset summary

| Table | Rows | Notes |
|---|---|---|
| `customers` | 6 | Includes 1 inactive account (Fiona Gallagher, no orders) |
| `products` | 8 | 2 Produce, 3 Dairy, 3 Bakery |
| `orders` | 15 | August to September 2026 |
| `order_items` | 27 | Line items across all orders |

## 5. Analytical Queries

### Section A: Multi-Table JOINs

| # | Query | Technique | Purpose |
|---|---|---|---|
| 1 | Customer Order Activity | `INNER JOIN` | Lists each order with customer name, city, and date. Returns 15 rows. Helps logistics correlate order volume with geographic hubs. |
| 2 | Granular Line-Item Details | `JOIN` | Lists each line item with product, category, price, and quantity. Returns 27 rows. Highlights high-velocity items (Bananas, Organic Apples) versus specialty items (Cheddar Cheese). |
| 3 | Inactive Customer Audit | `LEFT JOIN` | Lists all customers with their orders. Returns 16 rows; Fiona Gallagher (ID 6) appears with `NULL` order fields. Marketing can target her with onboarding offers. |

```sql
-- Query 1
SELECT o.order_id, c.customer_name, c.city, o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.order_id;

-- Query 2
SELECT oi.order_item_id, oi.order_id, p.product_name, p.category, p.price, oi.quantity
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
ORDER BY oi.order_item_id;

-- Query 3
SELECT c.customer_id, c.customer_name, c.city, o.order_id, o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;
```

### Section B: Common Table Expression (CTE)

**Query 4: High-Value Customers (Spend Above Average)**

A CTE totals each customer's spend, and a scalar subquery compares each total against the overall average. Management can use the result to design loyalty tiers and early-access programs.

```sql
WITH customer_spending AS (
    SELECT c.customer_id, c.customer_name,
           SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o       ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p     ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT customer_id, customer_name, total_spent
FROM customer_spending
WHERE total_spent > (SELECT AVG(total_spent) FROM customer_spending)
ORDER BY total_spent DESC;
```

### Section C: Window Functions

| # | Query | Function | Purpose |
|---|---|---|---|
| 5 | Customer Expenditure Ranking | `DENSE_RANK()` | Ranks customers by total spend with no gaps in rank numbers when there are ties. |
| 6 | Chronological Order Sequencing | `ROW_NUMBER()` | Numbers each customer's orders by date. Sequence 1 marks the first purchase, and higher numbers mark repeat visits. |
| 7 | Cumulative Revenue Trajectory | `SUM() OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` | Running total of gross revenue over time. |
| 8 | Inter-Order Frequency | `LAG()` + `COUNT() OVER` | Days between successive orders for repeat customers. Single-order customers are filtered out. |

```sql
-- Query 5
WITH customer_totals AS (
    SELECT c.customer_id, c.customer_name,
           SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o       ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p     ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT customer_id, customer_name, total_spent,
       DENSE_RANK() OVER (ORDER BY total_spent DESC) AS spending_rank
FROM customer_totals;

-- Query 6
SELECT customer_id, order_id, order_date,
       ROW_NUMBER() OVER (
           PARTITION BY customer_id
           ORDER BY order_date, order_id
       ) AS order_sequence_number
FROM orders
ORDER BY customer_id, order_sequence_number;

-- Query 7
WITH daily_order_revenue AS (
    SELECT o.order_id, o.order_date,
           SUM(oi.quantity * p.price) AS order_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p     ON oi.product_id = p.product_id
    GROUP BY o.order_id, o.order_date
)
SELECT order_id, order_date, order_revenue,
       SUM(order_revenue) OVER (
           ORDER BY order_date, order_id
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total_revenue
FROM daily_order_revenue
ORDER BY order_date, order_id;

-- Query 8
WITH order_gaps AS (
    SELECT customer_id, order_id, order_date,
           LAG(order_date) OVER (
               PARTITION BY customer_id ORDER BY order_date
           ) AS previous_order_date,
           COUNT(*) OVER (PARTITION BY customer_id) AS total_customer_orders
    FROM orders
)
SELECT customer_id, order_id, order_date, previous_order_date,
       (order_date - previous_order_date) AS days_since_previous_order
FROM order_gaps
WHERE total_customer_orders > 1
ORDER BY customer_id, order_date;
```

## 6. Technical Challenges and Resolutions

| Challenge | Root Cause | Resolution |
|---|---|---|
| Terminal output line-wrapping | SQL*Plus defaults to an 80-character width, so wide `VARCHAR2(100)` columns wrap and repeat headers. | `SET LINESIZE 200;`, `SET PAGESIZE 50;`, and `COLUMN <col> FORMAT A<len>`. |
| Parent-table deletion errors (`ORA-02449`) | Parent tables (`customers`, `orders`) could not be dropped while child foreign keys still referenced them. | Dropped tables in child-to-parent order, or used `CASCADE CONSTRAINTS PURGE`. |
| Aggregate filtering in CTEs | `AVG()` cannot be used directly in a `WHERE` clause. | Used a nested scalar subquery: `(SELECT AVG(total_spent) FROM customer_spending)`. |
| Inter-order date difference | Day intervals often rely on vendor-specific functions such as `DATEDIFF`. | Used Oracle's native date arithmetic (`order_date - previous_order_date`), which returns the difference in days. |

## 7. Key Concepts Demonstrated

- Primary and foreign key design across four related tables
- `INNER JOIN` and `LEFT JOIN` (including detecting rows with no match)
- Common Table Expressions with scalar subqueries
- Ranking, sequencing, running totals, and lag-based analysis with window functions
- Native Oracle date arithmetic
