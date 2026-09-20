PL/SQL ASSIGNMENT ONE: SUNRISE SUPERMARKET DATABASE REPORTCourse: Database Systems / PL/SQLStudent Name:Ruhumuriza Yvan Kenny Student ID:29179 Academic Group: GroupB Submission Date: September 2026RDBMS Platform: Oracle Database 21c Express Edition (Release 21.3.0.0.0)Client Environment: Oracle SQL*Plus1. Executive Summary & Business Scenario1.1 Organizational ContextSunrise Supermarket is a regional grocery retailer operating across major urban markets, including Kigali, Musanze, Huye, and Rubavu. The supermarket provides consumers with essential retail products categorized across three primary departments: Produce, Dairy, and Bakery.1.2 Business ChallengeManagement requires clear, data-driven visibility into customer purchasing behaviors and transactional throughput. Key operational priorities include:Identifying customer geography and detecting registered accounts that remain inactive.Analyzing shopping basket composition to evaluate category volume.Segmenting high-value customers who contribute above-average revenue.Tracking repeat purchase cycles to determine customer retention and re-engagement windows.Monitoring storewide cumulative revenue velocity over time.2. System Architecture & Setup Instructions2.1 Technical PrerequisitesDBMS Engine: Oracle Database 21c Express Edition.Pluggable Database: XEPDB1 on default port 1521.Execution Utility: SQL*Plus Command Line Interface.2.2 Execution StepsConnect to Pluggable Database:Bashsqlplus myuser/password@localhost:1521/XEPDB1
Configure Display Settings:Execute these formatting commands in SQL*Plus to prevent column clipping and line-wrapping:SQLSET LINESIZE 200;
SET PAGESIZE 50;
COLUMN customer_name FORMAT A20;
COLUMN product_name FORMAT A22;
COLUMN city FORMAT A15;
COLUMN category FORMAT A12;
Deploy Schema and Data:Run the table generation and insert statements sequentially.3. Database Schema & Data Population3.1 DDL StatementsSQLCREATE TABLE customers (
  customer_id NUMBER PRIMARY KEY,
  customer_name VARCHAR2(100),
  email VARCHAR2(100),
  city VARCHAR2(50)
);

CREATE TABLE products (
  product_id NUMBER PRIMARY KEY,
  product_name VARCHAR2(100),
  category VARCHAR2(50),
  price NUMBER(10,2)
);

CREATE TABLE orders (
  order_id NUMBER PRIMARY KEY,
  customer_id NUMBER REFERENCES customers(customer_id),
  order_date DATE
);

CREATE TABLE order_items (
  order_item_id NUMBER PRIMARY KEY,
  order_id NUMBER REFERENCES orders(order_id),
  product_id NUMBER REFERENCES products(product_id),
  quantity NUMBER
);
3.2 Data Population (DML)The dataset comprises 6 customers (including 1 inactive account), 8 products across 3 categories, 15 orders, and 27 order line items:SQL-- Customers
INSERT INTO customers VALUES (1, 'Alice Smith', 'alice@example.com', 'Kigali');
INSERT INTO customers VALUES (2, 'Bob Jones', 'bob@example.com', 'Musanze');
INSERT INTO customers VALUES (3, 'Charlie Brown', 'charlie@example.com', 'Huye');
INSERT INTO customers VALUES (4, 'Diana Prince', 'diana@example.com', 'Rubavu');
INSERT INTO customers VALUES (5, 'Evan Wright', 'evan@example.com', 'Kigali');
INSERT INTO customers VALUES (6, 'Fiona Gallagher', 'fiona@example.com', 'Kigali');

-- Products
INSERT INTO products VALUES (101, 'Organic Apples', 'Produce', 3.50);
INSERT INTO products VALUES (102, 'Bananas', 'Produce', 1.20);
INSERT INTO products VALUES (103, 'Whole Milk', 'Dairy', 2.80);
INSERT INTO products VALUES (104, 'Cheddar Cheese', 'Dairy', 5.50);
INSERT INTO products VALUES (105, 'Greek Yogurt', 'Dairy', 4.00);
INSERT INTO products VALUES (106, 'Sourdough Bread', 'Bakery', 4.50);
INSERT INTO products VALUES (107, 'Chocolate Croissant', 'Bakery', 2.75);
INSERT INTO products VALUES (108, 'Baguette', 'Bakery', 2.00);

-- Orders
INSERT INTO orders VALUES (1001, 1, DATE '2026-08-01');
INSERT INTO orders VALUES (1002, 2, DATE '2026-08-03');
INSERT INTO orders VALUES (1003, 1, DATE '2026-08-05');
INSERT INTO orders VALUES (1004, 3, DATE '2026-08-07');
INSERT INTO orders VALUES (1005, 4, DATE '2026-08-10');
INSERT INTO orders VALUES (1006, 2, DATE '2026-08-12');
INSERT INTO orders VALUES (1007, 5, DATE '2026-08-14');
INSERT INTO orders VALUES (1008, 1, DATE '2026-08-18');
INSERT INTO orders VALUES (1009, 3, DATE '2026-08-20');
INSERT INTO orders VALUES (1010, 4, DATE '2026-08-22');
INSERT INTO orders VALUES (1011, 2, DATE '2026-08-25');
INSERT INTO orders VALUES (1012, 5, DATE '2026-08-28');
INSERT INTO orders VALUES (1013, 1, DATE '2026-09-02');
INSERT INTO orders VALUES (1014, 3, DATE '2026-09-05');
INSERT INTO orders VALUES (1015, 4, DATE '2026-09-10');

-- Order Items
INSERT INTO order_items VALUES (1, 1001, 101, 2);
INSERT INTO order_items VALUES (2, 1001, 103, 1);
INSERT INTO order_items VALUES (3, 1002, 106, 2);
INSERT INTO order_items VALUES (4, 1002, 104, 1);
INSERT INTO order_items VALUES (5, 1003, 107, 4);
INSERT INTO order_items VALUES (6, 1004, 102, 5);
INSERT INTO order_items VALUES (7, 1004, 105, 2);
INSERT INTO order_items VALUES (8, 1005, 101, 3);
INSERT INTO order_items VALUES (9, 1005, 108, 2);
INSERT INTO order_items VALUES (10, 1006, 104, 2);
INSERT INTO order_items VALUES (11, 1007, 103, 3);
INSERT INTO order_items VALUES (12, 1007, 106, 1);
INSERT INTO order_items VALUES (13, 1008, 105, 3);
INSERT INTO order_items VALUES (14, 1009, 101, 4);
INSERT INTO order_items VALUES (15, 1009, 107, 2);
INSERT INTO order_items VALUES (16, 1010, 102, 6);
INSERT INTO order_items VALUES (17, 1010, 104, 1);
INSERT INTO order_items VALUES (18, 1011, 108, 3);
INSERT INTO order_items VALUES (19, 1011, 103, 2);
INSERT INTO order_items VALUES (20, 1012, 106, 2);
INSERT INTO order_items VALUES (21, 1012, 105, 1);
INSERT INTO order_items VALUES (22, 1013, 101, 2);
INSERT INTO order_items VALUES (23, 1013, 104, 2);
INSERT INTO order_items VALUES (24, 1014, 102, 10);
INSERT INTO order_items VALUES (25, 1014, 107, 3);
INSERT INTO order_items VALUES (26, 1015, 103, 4);
INSERT INTO order_items VALUES (27, 1015, 108, 2);

COMMIT;
4. Analytical SQL Queries & Interpretations4.1 Section A: Multi-Table JOIN QueriesQuery 1: Customer Order ActivityLists every order alongside the customer's name, city, and transaction date.SQLSELECT 
    o.order_id,
    c.customer_name,
    c.city,
    o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
ORDER BY o.order_id;
Query Logic: Joins orders with customers using an INNER JOIN matching on customer_id.Expected Result Set: 15 records linking each order directly to a buyer and city.Business Interpretation: Enables the logistics and supply chain departments to correlate order volumes with geographic hubs, helping allocate fulfillment resources to cities with higher transaction activity.Query 2: Granular Line-Item DetailsLists every order line item with product name, department category, unit price, and purchased quantity.SQLSELECT 
    oi.order_item_id,
    oi.order_id,
    p.product_name,
    p.category,
    p.price,
    oi.quantity
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
ORDER BY oi.order_item_id;
Query Logic: Joins order_items with products on product_id.Expected Result Set: 27 records displaying product descriptions and transaction counts.Business Interpretation: Breaks down individual shopping baskets to highlight high-velocity SKUs (e.g., Bananas and Organic Apples) versus high-margin specialty items (e.g., Cheddar Cheese).Query 3: Inactive Customer AuditLists all customers and their corresponding orders, explicitly surfacing registered customers who have never placed an order.SQLSELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;
Query Logic: Applies a LEFT JOIN from customers to orders, ensuring customers without corresponding foreign keys in orders appear with NULL values.Expected Result Set: 16 rows. Fiona Gallagher (customer_id = 6) appears with NULL for order_id and order_date.Business Interpretation: Identifies inactive or abandoned registrations. Marketing can use this data to target non-converting users with onboarding promotions or discount vouchers.4.2 Section B: Common Table Expression (CTE) QueryQuery 4: High-Value Customer Spend Above AverageCalculates each customer's total expenditure and returns customers whose total spend exceeds the customer average.SQLWITH customer_spending AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id,
    customer_name,
    total_spent
FROM customer_spending
WHERE total_spent > (SELECT AVG(total_spent) FROM customer_spending)
ORDER BY total_spent DESC;
Query Logic: The CTE (customer_spending) aggregates total expenditure per customer across all orders. The outer query filters this aggregated table using a scalar subquery that computes the overall mean spending.Business Interpretation: Identifies top-tier accounts generating above-average revenue. Management can target this cohort for exclusive loyalty tiers, concierge support, and early access programs.4.3 Section C: Advanced Window-Function QueriesQuery 5: Customer Expenditure RankingRanks customers based on total spend, ordered from highest to lowest.SQLWITH customer_totals AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id,
    customer_name,
    total_spent,
    DENSE_RANK() OVER (ORDER BY total_spent DESC) AS spending_rank
FROM customer_totals;
Query Logic: Evaluates total customer spend within a CTE, then calculates relative rank using DENSE_RANK() over descending spend totals.Business Interpretation: Produces an unbroken rank sequence without skipping numbers in the event of ties, helping commercial teams prioritize accounts for retention programs.Query 6: Chronological Order SequencingAssigns an incrementing sequential number to each customer's orders based on order date.SQLSELECT 
    customer_id,
    order_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id 
        ORDER BY order_date, order_id
    ) AS order_sequence_number
FROM orders
ORDER BY customer_id, order_sequence_number;
Query Logic: Partitions order rows by customer_id and evaluates ROW_NUMBER() ordered chronologically by order_date.Business Interpretation: Separates initial customer conversions (order_sequence_number = 1) from recurring visits (order_sequence_number > 1), providing baseline data for customer lifecycle analysis.Query 7: Cumulative Revenue TrajectoryDisplays a running total of gross store revenue over time, ordered chronologically.SQLWITH daily_order_revenue AS (
    SELECT 
        o.order_id,
        o.order_date,
        SUM(oi.quantity * p.price) AS order_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY o.order_id, o.order_date
)
SELECT 
    order_id,
    order_date,
    order_revenue,
    SUM(order_revenue) OVER (
        ORDER BY order_date, order_id 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_revenue
FROM daily_order_revenue
ORDER BY order_date, order_id;
Query Logic: Aggregates line-item revenue per order, then applies SUM() OVER (...) with an explicit frame clause (ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW).Business Interpretation: Shows cumulative cash inflows across the business cycle, allowing management to assess revenue consistency and evaluate the impact of sales initiatives.Query 8: Inter-Order Frequency & Re-Order IntervalsCalculates the number of days elapsed between successive orders for repeat customers.SQLWITH order_gaps AS (
    SELECT 
        customer_id,
        order_id,
        order_date,
        LAG(order_date) OVER (
            PARTITION BY customer_id 
            ORDER BY order_date
        ) AS previous_order_date,
        COUNT(*) OVER (
            PARTITION BY customer_id
        ) AS total_customer_orders
    FROM orders
)
SELECT 
    customer_id,
    order_id,
    order_date,
    previous_order_date,
    (order_date - previous_order_date) AS days_since_previous_order
FROM order_gaps
WHERE total_customer_orders > 1
ORDER BY customer_id, order_date;
Query Logic: Utilizes LAG(order_date) partitioned by customer_id to retrieve the preceding purchase date, then computes elapsed days via native date subtraction (order_date - previous_order_date). Accounts with only one order are filtered out using COUNT(*) OVER (PARTITION BY customer_id).Business Interpretation: Measures shopping frequency and repurchase cadences. If an active buyer exceeds their normal re-order interval, automated reminders or loyalty offers can be triggered.5. Technical Challenges & ResolutionsChallenge EncounteredTechnical Root CauseResolution AppliedTerminal Output Line-WrappingSQL*Plus defaults to an 80-character terminal width, causing columns like VARCHAR2(100) to wrap across lines and repeat headers.   Configured SET LINESIZE 200;, SET PAGESIZE 50;, and set column display limits using COLUMN <col> FORMAT A<len>.Parent-Table Deletion Errors (ORA-02449)Attempting to drop parent tables (customers, orders) failed because child foreign keys were still referencing them.Re-sequenced drop commands in strict child-to-parent order (order_items → orders → products → customers) or applied CASCADE CONSTRAINTS PURGE.Aggregate Filtering in CTEsAggregate functions such as AVG() cannot be referenced directly in a WHERE clause without a subquery.Created a nested scalar subquery (SELECT AVG(total_spent) FROM customer_spending) within the outer query's WHERE clause.Inter-Order Date DifferenceEvaluating day intervals in SQL often relies on vendor-specific functions (like DATEDIFF).Used Oracle’s native date arithmetic (order_date - previous_order_date), which evaluates differences between DATE types directly into numeric days.
