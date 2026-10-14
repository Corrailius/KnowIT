# SQL Guide — From Zero to Confident

> **Who is this for?** Complete beginners and people who know a bit but want to solidify their understanding. No prior experience needed.

---

## Table of Contents

1. [What is SQL?](#what-is-sql)
2. [How Databases Work](#how-databases-work)
3. [Your First Query — SELECT](#your-first-query--select)
4. [Filtering Results — WHERE](#filtering-results--where)
5. [Sorting & Limiting Results](#sorting--limiting-results)
6. [Combining Conditions](#combining-conditions)
7. [Aggregation — Counting & Summing](#aggregation--counting--summing)
8. [Grouping Data — GROUP BY](#grouping-data--group-by)
9. [Joining Tables](#joining-tables)
10. [Subqueries & CTEs](#subqueries--ctes)
11. [Window Functions](#window-functions)
12. [Modifying Data — INSERT, UPDATE, DELETE](#modifying-data--insert-update-delete)
13. [Creating & Managing Tables](#creating--managing-tables)
14. [Useful Built-in Functions](#useful-built-in-functions)
15. [The CASE Expression](#the-case-expression)
16. [Set Operations](#set-operations)
17. [Indexes & Performance Tips](#indexes--performance-tips)
18. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
19. [Quick Reference Cheatsheet](#quick-reference-cheatsheet)

---

## What is SQL?

**SQL** (Structured Query Language, pronounced "sequel") is the language used to talk to databases. It lets you:

- **Ask questions** — "How many users signed up last month?"
- **Retrieve data** — "Give me all orders over $100."
- **Change data** — "Update the price of product #42."
- **Create structure** — "Make a new table to store blog posts."

SQL is used everywhere: apps, websites, analytics tools, data science, and more. Learning SQL is one of the most practical skills you can pick up.

---

## How Databases Work

Think of a database as a collection of **spreadsheets**, where each spreadsheet is called a **table**.

**Example: A simple shop database**

**Table: `customers`**

| id | name       | email              | city       |
|----|------------|--------------------|------------|
| 1  | Ana Reyes  | ana@example.com    | Madrid     |
| 2  | Tom Brown  | tom@example.com    | London     |
| 3  | Li Wei     | li@example.com     | Shanghai   |

**Table: `orders`**

| id | customer_id | total | created_at  |
|----|-------------|-------|-------------|
| 1  | 1           | 89.50 | 2024-03-01  |
| 2  | 1           | 210.00| 2024-03-15  |
| 3  | 2           | 55.00 | 2024-03-20  |

Key ideas:
- Each **row** is one record (one customer, one order).
- Each **column** is a field (name, email, total).
- Tables are linked by **keys** — `customer_id` in orders refers to `id` in customers.

---

## Your First Query — SELECT

`SELECT` is the most common SQL statement. It retrieves data from a table.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name;
```

**Get all columns:**
```sql
SELECT *
FROM customers;
```
> `*` means "all columns". Useful for exploring, but avoid it in production code.

**Get specific columns:**
```sql
SELECT name, email
FROM customers;
```

**Give a column a friendlier name (alias):**
```sql
SELECT name, email AS contact_email
FROM customers;
```
> `AS` renames the column in your results. It doesn't change the actual table.

**Remove duplicates with DISTINCT:**
```sql
SELECT DISTINCT city
FROM customers;
```
> Returns each city only once, even if multiple customers share it.

---

## Filtering Results — WHERE

Use `WHERE` to filter rows — only return rows that meet a condition.

```sql
SELECT *
FROM customers
WHERE city = 'London';
```

### Comparison operators

| Operator | Meaning              | Example                    |
|----------|----------------------|----------------------------|
| `=`      | Equal to             | `city = 'Madrid'`          |
| `!=` or `<>` | Not equal to    | `city != 'London'`         |
| `>`      | Greater than         | `total > 100`              |
| `<`      | Less than            | `total < 50`               |
| `>=`     | Greater than or equal| `total >= 100`             |
| `<=`     | Less than or equal   | `total <= 100`             |

### Pattern matching with LIKE

```sql
-- Names starting with 'A'
SELECT * FROM customers WHERE name LIKE 'A%';

-- Names ending with 'Brown'
SELECT * FROM customers WHERE name LIKE '%Brown';

-- Names containing 'ei'
SELECT * FROM customers WHERE name LIKE '%ei%';
```
> `%` means "any number of characters". `_` means "exactly one character".

### Checking a list of values with IN

```sql
SELECT * FROM customers
WHERE city IN ('Madrid', 'London', 'Berlin');
```
> Much cleaner than writing `city = 'Madrid' OR city = 'London' OR city = 'Berlin'`.

### Range checking with BETWEEN

```sql
SELECT * FROM orders
WHERE total BETWEEN 50 AND 200;
```
> Inclusive — includes 50 and 200.

### Checking for missing values with IS NULL

```sql
-- Customers with no email on file
SELECT * FROM customers WHERE email IS NULL;

-- Customers who DO have an email
SELECT * FROM customers WHERE email IS NOT NULL;
```
> Always use `IS NULL`, never `= NULL` — that won't work in SQL.

---

## Sorting & Limiting Results

### ORDER BY — sort your results

```sql
-- Sort by name A–Z (ascending, default)
SELECT * FROM customers ORDER BY name;

-- Sort by total, highest first (descending)
SELECT * FROM orders ORDER BY total DESC;

-- Sort by multiple columns
SELECT * FROM orders ORDER BY customer_id ASC, total DESC;
```

### LIMIT — only return a certain number of rows

```sql
-- Get the top 5 biggest orders
SELECT * FROM orders
ORDER BY total DESC
LIMIT 5;
```

### OFFSET — skip rows (useful for pagination)

```sql
-- Skip the first 10, get the next 10 (page 2)
SELECT * FROM orders
ORDER BY id
LIMIT 10 OFFSET 10;
```

---

## Combining Conditions

Use `AND`, `OR`, and `NOT` to combine multiple conditions.

```sql
-- Customers in London who have placed large orders
SELECT * FROM orders
WHERE total > 100
  AND customer_id = 2;

-- Orders that are either very small or very large
SELECT * FROM orders
WHERE total < 20 OR total > 500;

-- Everyone except London customers
SELECT * FROM customers
WHERE NOT city = 'London';
```

> **Tip:** Use parentheses `()` to control order of logic, just like in maths:
```sql
WHERE (city = 'Madrid' OR city = 'London') AND total > 50
```

---

## Aggregation — Counting & Summing

Aggregate functions crunch many rows down into a single number.

| Function      | What it does                        |
|---------------|-------------------------------------|
| `COUNT(*)`    | Counts all rows                     |
| `COUNT(col)`  | Counts non-null values in a column  |
| `SUM(col)`    | Adds up all values                  |
| `AVG(col)`    | Calculates the average              |
| `MIN(col)`    | Finds the smallest value            |
| `MAX(col)`    | Finds the largest value             |

**Examples:**
```sql
-- How many customers do we have?
SELECT COUNT(*) AS total_customers FROM customers;

-- What's our total revenue?
SELECT SUM(total) AS revenue FROM orders;

-- What's the average order value?
SELECT AVG(total) AS avg_order FROM orders;

-- Biggest and smallest orders
SELECT MIN(total), MAX(total) FROM orders;
```

---

## Grouping Data — GROUP BY

`GROUP BY` is used with aggregate functions to get totals **per group** (per category, per customer, per month, etc.).

```sql
-- How many orders does each customer have?
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;

-- Total spent per customer
SELECT customer_id, SUM(total) AS total_spent
FROM orders
GROUP BY customer_id;
```

### HAVING — filter after grouping

`WHERE` filters individual rows *before* grouping. `HAVING` filters groups *after* grouping.

```sql
-- Only customers who've placed more than 1 order
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

> **Rule of thumb:** Use `WHERE` for row-level filters, `HAVING` for aggregate filters.

---

## Joining Tables

Joins let you combine data from multiple tables. This is where SQL gets really powerful.

Imagine the `customers` and `orders` tables from earlier — a join lets you see customer names alongside their orders.

### INNER JOIN — only matching rows

```sql
SELECT customers.name, orders.total
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id;
```

Only returns rows where there's a match in both tables. Customers with no orders won't appear.

### LEFT JOIN — all left rows, plus matches

```sql
SELECT customers.name, orders.total
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id;
```

Returns every customer, with their orders if any exist. Customers without orders will appear with `NULL` for order columns.

### RIGHT JOIN — all right rows, plus matches

Same as LEFT JOIN, but from the other direction. All orders are returned even if their customer record is missing.

### FULL JOIN — everything from both tables

Returns all rows from both tables, with `NULL` where there's no match on either side.

### Using table aliases to keep queries readable

```sql
SELECT c.name, o.total
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
WHERE o.total > 100;
```
> Aliases like `c` and `o` save typing and make queries much cleaner.

### Joining more than two tables

```sql
SELECT c.name, p.name AS product, oi.quantity
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

---

## Subqueries & CTEs

### Subqueries — a query inside a query

```sql
-- Get customers who have placed at least one order
SELECT name FROM customers
WHERE id IN (
  SELECT DISTINCT customer_id FROM orders
);
```

The inner query runs first, and its result is used by the outer query.

### CTEs (Common Table Expressions) — cleaner, reusable subqueries

A CTE is like giving a subquery a name. Use `WITH` to define it.

```sql
WITH big_orders AS (
  SELECT customer_id, SUM(total) AS total_spent
  FROM orders
  GROUP BY customer_id
  HAVING SUM(total) > 200
)
SELECT customers.name, big_orders.total_spent
FROM customers
JOIN big_orders ON customers.id = big_orders.customer_id;
```

CTEs make complex queries much easier to read and debug. You can even define multiple CTEs:

```sql
WITH
  active_customers AS (
    SELECT id FROM customers WHERE active = true
  ),
  recent_orders AS (
    SELECT * FROM orders WHERE created_at >= '2024-01-01'
  )
SELECT ac.id, ro.total
FROM active_customers ac
JOIN recent_orders ro ON ac.id = ro.customer_id;
```

---

## Window Functions

Window functions perform calculations *across related rows* without collapsing them into a single result like `GROUP BY` does.

Think of it like: "for each row, look at a window of related rows and compute something."

### ROW_NUMBER — assign a sequential number

```sql
SELECT name, total,
  ROW_NUMBER() OVER (ORDER BY total DESC) AS rank
FROM orders;
```

### RANK and DENSE_RANK — rank with ties

```sql
-- RANK: ties get the same rank, next rank skips a number (1, 1, 3)
-- DENSE_RANK: ties get the same rank, next rank doesn't skip (1, 1, 2)
SELECT name, score,
  RANK() OVER (ORDER BY score DESC) AS rank,
  DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM results;
```

### PARTITION BY — reset the window per group

```sql
-- Rank orders within each customer independently
SELECT customer_id, total,
  RANK() OVER (PARTITION BY customer_id ORDER BY total DESC) AS rank_within_customer
FROM orders;
```

### LAG and LEAD — look at the previous/next row

```sql
-- Compare each order's total to the previous order
SELECT id, total,
  LAG(total, 1) OVER (ORDER BY created_at) AS previous_total
FROM orders;
```

### Running total

```sql
SELECT created_at, total,
  SUM(total) OVER (ORDER BY created_at) AS running_total
FROM orders;
```

---

## Modifying Data — INSERT, UPDATE, DELETE

### INSERT — add new rows

```sql
-- Add one customer
INSERT INTO customers (name, email, city)
VALUES ('Sofia Diaz', 'sofia@example.com', 'Barcelona');

-- Add multiple rows at once
INSERT INTO customers (name, email, city)
VALUES
  ('Ravi Kumar', 'ravi@example.com', 'Mumbai'),
  ('Emma Clarke', 'emma@example.com', 'Dublin');
```

### UPDATE — change existing rows

```sql
-- Update a specific customer's email
UPDATE customers
SET email = 'newemail@example.com'
WHERE id = 1;

-- Update multiple columns at once
UPDATE customers
SET email = 'new@example.com', city = 'Seville'
WHERE id = 1;
```

> ⚠️ **Always use WHERE with UPDATE.** Without it, you'll update every single row.

### DELETE — remove rows

```sql
-- Delete one customer
DELETE FROM customers WHERE id = 3;

-- Delete all customers in a specific city
DELETE FROM customers WHERE city = 'London';
```

> ⚠️ **Always use WHERE with DELETE.** Without it, you'll wipe the entire table.

---

## Creating & Managing Tables

### CREATE TABLE — build a new table

```sql
CREATE TABLE products (
  id         SERIAL PRIMARY KEY,
  name       VARCHAR(200) NOT NULL,
  price      DECIMAL(10, 2) NOT NULL,
  stock      INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Common data types

| Type              | Use for                              |
|-------------------|--------------------------------------|
| `INT`             | Whole numbers (1, 42, -7)            |
| `SERIAL`          | Auto-incrementing integer (great for IDs) |
| `DECIMAL(10, 2)`  | Precise numbers — currency, measurements |
| `FLOAT`           | Approximate decimal numbers          |
| `VARCHAR(n)`      | Text up to n characters              |
| `TEXT`            | Unlimited length text                |
| `BOOLEAN`         | True or false                        |
| `DATE`            | Date only (2024-03-15)               |
| `TIMESTAMP`       | Date and time                        |

### Constraints — rules for your data

```sql
CREATE TABLE users (
  id       SERIAL PRIMARY KEY,          -- unique, not null, auto-increment
  email    VARCHAR(255) UNIQUE NOT NULL, -- must be unique, can't be blank
  age      INT CHECK (age >= 0),         -- must pass this condition
  role     VARCHAR(50) DEFAULT 'viewer', -- default value if none provided
  team_id  INT REFERENCES teams(id)     -- foreign key link to teams table
);
```

### ALTER TABLE — modify an existing table

```sql
-- Add a new column
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);

-- Remove a column
ALTER TABLE customers DROP COLUMN phone;

-- Change a column's data type
ALTER TABLE customers ALTER COLUMN name TYPE TEXT;

-- Rename a column
ALTER TABLE customers RENAME COLUMN name TO full_name;
```

### DROP TABLE — delete a table entirely

```sql
-- Safe version — won't error if the table doesn't exist
DROP TABLE IF EXISTS old_logs;
```

> ⚠️ This deletes the table AND all its data permanently.

---

## Useful Built-in Functions

### String functions

```sql
UPPER('hello')           -- 'HELLO'
LOWER('WORLD')           -- 'world'
LENGTH('SQL')            -- 3
TRIM('  hello  ')        -- 'hello'
LTRIM('  hello')         -- 'hello' (left trim only)
RTRIM('hello  ')         -- 'hello' (right trim only)
CONCAT('Hello', ' ', 'World')  -- 'Hello World'
'Hello' || ' ' || 'World'      -- 'Hello World' (shorthand)
SUBSTRING('Hello World', 1, 5) -- 'Hello'
REPLACE('Hello World', 'World', 'SQL') -- 'Hello SQL'
POSITION('lo' IN 'Hello')      -- 4
```

### Number functions

```sql
ROUND(3.14159, 2)   -- 3.14
CEIL(4.2)           -- 5
FLOOR(4.9)          -- 4
ABS(-42)            -- 42
MOD(10, 3)          -- 1 (remainder)
POWER(2, 8)         -- 256
```

### Date functions

```sql
NOW()                            -- current date and time
CURRENT_DATE                     -- today's date
CURRENT_TIME                     -- current time

DATE_PART('year', '2024-03-15')  -- 2024
DATE_PART('month', created_at)   -- month number

-- Add/subtract time
created_at + INTERVAL '7 days'
created_at - INTERVAL '1 month'

-- Difference between dates
AGE('2024-12-31', '2024-01-01')  -- 11 months 30 days
```

### Handling NULL values

```sql
-- COALESCE: return the first non-null value
COALESCE(phone, email, 'no contact')

-- NULLIF: return NULL if two values are equal (avoids divide-by-zero)
NULLIF(denominator, 0)

-- IFNULL / ISNULL (MySQL / SQL Server)
IFNULL(column, 'default value')
```

### Type casting

```sql
-- Convert a value to a different type
CAST('42' AS INT)
CAST(price AS VARCHAR)
'42'::INT       -- PostgreSQL shorthand
```

---

## The CASE Expression

`CASE` is SQL's way of doing if/else logic inside a query.

### Simple CASE

```sql
SELECT name,
  CASE grade
    WHEN 'A' THEN 'Excellent'
    WHEN 'B' THEN 'Good'
    WHEN 'C' THEN 'Average'
    ELSE 'Needs improvement'
  END AS feedback
FROM students;
```

### Searched CASE (more flexible)

```sql
SELECT name, total,
  CASE
    WHEN total >= 500 THEN 'VIP'
    WHEN total >= 100 THEN 'Regular'
    ELSE 'Occasional'
  END AS customer_tier
FROM orders;
```

### CASE inside aggregation

```sql
-- Count how many orders fall into each tier
SELECT
  COUNT(CASE WHEN total >= 500 THEN 1 END) AS vip_orders,
  COUNT(CASE WHEN total >= 100 THEN 1 END) AS regular_orders,
  COUNT(CASE WHEN total < 100 THEN 1 END)  AS small_orders
FROM orders;
```

---

## Set Operations

These combine the results of two separate queries into one.

> Both queries must return the same number of columns and compatible data types.

### UNION — combine and remove duplicates

```sql
SELECT email FROM customers
UNION
SELECT email FROM newsletter_subscribers;
```

### UNION ALL — combine and keep duplicates

```sql
SELECT email FROM customers
UNION ALL
SELECT email FROM newsletter_subscribers;
```
> Faster than `UNION` since it doesn't need to check for duplicates.

### INTERSECT — rows that appear in both

```sql
SELECT email FROM customers
INTERSECT
SELECT email FROM newsletter_subscribers;
-- Returns only emails that are in BOTH lists
```

### EXCEPT — rows in the first but not the second

```sql
SELECT email FROM customers
EXCEPT
SELECT email FROM newsletter_subscribers;
-- Returns customers who are NOT subscribed to the newsletter
```

---

## Indexes & Performance Tips

### What is an index?

An index is like the index at the back of a book — instead of scanning every page, you jump straight to what you need. Indexes make queries much faster on large tables.

```sql
-- Create a basic index on the email column
CREATE INDEX idx_customers_email ON customers(email);

-- Unique index — also enforces uniqueness
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Composite index — covers queries filtering on both columns
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);

-- Remove an index
DROP INDEX idx_customers_email;
```

### When to use indexes

**Do index:**
- Columns used frequently in `WHERE` clauses
- Columns used in `JOIN` conditions
- Columns used in `ORDER BY` on large tables
- Foreign key columns

**Don't over-index:**
- Indexes slow down `INSERT`, `UPDATE`, and `DELETE`
- Small tables don't need indexes — a full scan is fine
- Columns with very few distinct values (like `true/false`) rarely benefit

### General performance tips

```sql
-- Avoid SELECT * in production — only fetch what you need
SELECT id, name FROM users;  -- ✅ Better than SELECT *

-- Filter early with WHERE to reduce rows before joining
SELECT * FROM orders WHERE created_at > '2024-01-01';

-- Use LIMIT when you only need a few rows
SELECT * FROM logs ORDER BY created_at DESC LIMIT 100;

-- EXPLAIN shows the query execution plan (great for debugging slow queries)
EXPLAIN SELECT * FROM orders WHERE customer_id = 5;
```

---

## Common Mistakes to Avoid

### 1. Using `= NULL` instead of `IS NULL`

```sql
-- ❌ Wrong — this will never match anything
WHERE phone = NULL

-- ✅ Correct
WHERE phone IS NULL
```

### 2. Forgetting WHERE on UPDATE or DELETE

```sql
-- ❌ Deletes EVERY row in the table!
DELETE FROM users;

-- ✅ Only deletes the intended row
DELETE FROM users WHERE id = 42;
```

### 3. Confusing WHERE and HAVING

```sql
-- ❌ Wrong — can't use aggregate in WHERE
SELECT dept, COUNT(*) FROM employees
WHERE COUNT(*) > 5
GROUP BY dept;

-- ✅ Correct — use HAVING for aggregates
SELECT dept, COUNT(*) FROM employees
GROUP BY dept
HAVING COUNT(*) > 5;
```

### 4. Forgetting that NULL is contagious

```sql
-- Any maths involving NULL returns NULL
SELECT 5 + NULL;    -- returns NULL
SELECT NULL = NULL; -- returns NULL (not TRUE!)
```

### 5. GROUP BY column mismatch

```sql
-- ❌ Wrong — non-aggregated columns must be in GROUP BY
SELECT name, dept, COUNT(*) FROM employees GROUP BY dept;

-- ✅ Correct
SELECT dept, COUNT(*) FROM employees GROUP BY dept;
```

---

## Quick Reference Cheatsheet

```sql
-- ── READING DATA ──────────────────────────────────────
SELECT col1, col2 FROM table;             -- get columns
SELECT * FROM table;                      -- get all columns
SELECT DISTINCT col FROM table;           -- unique values only

-- ── FILTERING ─────────────────────────────────────────
WHERE col = 'value'                       -- equal
WHERE col != 'value'                      -- not equal
WHERE col > 10 AND col < 100              -- range
WHERE col BETWEEN 10 AND 100              -- range (inclusive)
WHERE col IN ('a', 'b', 'c')             -- match list
WHERE col LIKE '%pattern%'               -- text pattern
WHERE col IS NULL                         -- missing value
WHERE col IS NOT NULL                     -- present value

-- ── SORTING & LIMITING ────────────────────────────────
ORDER BY col ASC                          -- sort A→Z, 1→9
ORDER BY col DESC                         -- sort Z→A, 9→1
LIMIT 10                                  -- first 10 rows
LIMIT 10 OFFSET 20                        -- rows 21–30

-- ── AGGREGATION ───────────────────────────────────────
COUNT(*), COUNT(col), SUM(col)
AVG(col), MIN(col), MAX(col)

GROUP BY col                              -- group rows
HAVING COUNT(*) > 5                       -- filter groups

-- ── JOINS ─────────────────────────────────────────────
INNER JOIN t ON a.id = t.a_id            -- matching rows
LEFT JOIN  t ON a.id = t.a_id            -- all left + matches
RIGHT JOIN t ON a.id = t.a_id            -- all right + matches
FULL JOIN  t ON a.id = t.a_id            -- all rows both sides

-- ── CTEs ──────────────────────────────────────────────
WITH cte_name AS (
  SELECT ...
)
SELECT * FROM cte_name;

-- ── MODIFYING DATA ────────────────────────────────────
INSERT INTO t (col1, col2) VALUES ('a', 'b');
UPDATE t SET col = 'val' WHERE id = 1;
DELETE FROM t WHERE id = 1;

-- ── TABLES ────────────────────────────────────────────
CREATE TABLE t (id SERIAL PRIMARY KEY, name VARCHAR(100));
ALTER TABLE t ADD COLUMN col INT;
DROP TABLE IF EXISTS t;

-- ── CASE ──────────────────────────────────────────────
CASE
  WHEN condition THEN result
  ELSE default_result
END

-- ── SET OPERATIONS ────────────────────────────────────
query_A UNION query_B                     -- combine, no dupes
query_A UNION ALL query_B                 -- combine, keep dupes
query_A INTERSECT query_B                 -- rows in both
query_A EXCEPT query_B                    -- rows in A not B
```

---

*Happy querying! The best way to learn SQL is to practise — try running these examples on a real database.*
