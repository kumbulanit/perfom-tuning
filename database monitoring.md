### **Comprehensive PostgreSQL Performance Tuning Exercise**

This exercise will guide you through creating a PostgreSQL database, populating it with data across multiple related tables, and applying performance tuning techniques. You will also learn how to measure query performance, identify bottlenecks, and apply optimizations for better performance. We'll use tools such as **SQL Trace**, **Query Plan Analysis**, and PostgreSQL's **pg_stat_statements** to monitor performance before and after tuning.

---

### **Step 1: Create and Populate the Database**

#### **1.1 Create the Database**

First, let's create the PostgreSQL database and switch to it:

```sql
-- Create the database
CREATE DATABASE dvdrental;

-- Connect to the newly created database
\c dvdrental;
```

#### **1.2 Create Tables**

Next, we will create the `customer`, `film`, `inventory`, `rental`, and `payment` tables, and populate them with sample data.

```sql
-- Create the 'customer' table
CREATE TABLE customer (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    address VARCHAR(100),
    active BOOLEAN
);

-- Create the 'film' table
CREATE TABLE film (
    film_id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    description TEXT,
    release_year INT,
    rental_duration INT,
    rental_rate DECIMAL(5,2)
);

-- Create the 'inventory' table
CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    film_id INT REFERENCES film(film_id),
    store_id INT,
    available BOOLEAN
);

-- Create the 'rental' table
CREATE TABLE rental (
    rental_id SERIAL PRIMARY KEY,
    rental_date TIMESTAMP,
    inventory_id INT REFERENCES inventory(inventory_id),
    customer_id INT REFERENCES customer(customer_id),
    return_date TIMESTAMP
);

-- Create the 'payment' table
CREATE TABLE payment (
    payment_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customer(customer_id),
    amount DECIMAL(5,2),
    payment_date TIMESTAMP
);
```

#### **1.3 Insert Sample Data**

Now, let's populate the tables with sample data. We'll add 100 customers, 50 films, and some entries for inventory, rental, and payment.

```sql
-- Insert sample data into the 'customer' table
INSERT INTO customer (first_name, last_name, email, address, active)
SELECT
    'First' || generate_series(1, 100), 
    'Last' || generate_series(1, 100),
    'email' || generate_series(1, 100) || '@example.com',
    'Address ' || generate_series(1, 100),
    TRUE
;

-- Insert sample data into the 'film' table
INSERT INTO film (title, description, release_year, rental_duration, rental_rate)
SELECT
    'Film Title ' || generate_series(1, 50),
    'A description for Film ' || generate_series(1, 50),
    2000 + (random() * 20)::int,
    7 + (random() * 3)::int,
    1.99 + (random() * 2)::float
;

-- Insert sample data into the 'inventory' table
INSERT INTO inventory (film_id, store_id, available)
SELECT 
    (random() * 50)::int + 1, 
    (random() * 5)::int + 1,
    TRUE
FROM generate_series(1, 200);

-- Insert sample data into the 'rental' table
INSERT INTO rental (rental_date, inventory_id, customer_id, return_date)
SELECT 
    NOW() - (random() * 30)::int * interval '1 day', 
    (random() * 200)::int + 1,
    (random() * 100)::int + 1,
    NOW() - (random() * 10)::int * interval '1 day'
FROM generate_series(1, 300);

-- Insert sample data into the 'payment' table
INSERT INTO payment (customer_id, amount, payment_date)
SELECT 
    (random() * 100)::int + 1,
    5 + (random() * 50)::float,
    NOW() - (random() * 30)::int * interval '1 day'
FROM generate_series(1, 200);
```

---

### **Step 2: Simulate Inefficient Queries**

Next, let's simulate inefficient queries to introduce performance issues. We will run queries that might trigger **full table scans** or **inefficient joins** without the proper indexes.

```sql
-- Inefficient query 1: Full table scan on rental and film tables
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE '%action%'
AND r.rental_date > NOW() - INTERVAL '30 days';

-- Inefficient query 2: Full table scan with an ORDER BY
SELECT c.first_name, c.last_name, p.amount, p.payment_date
FROM payment p
JOIN customer c ON c.customer_id = p.customer_id
WHERE p.payment_date BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY p.payment_date DESC;
```

---

### **Step 3: Measure Performance Before Tuning**

Before optimizing, let's measure the performance of the queries.

#### **3.1 Enable SQL Trace**

To capture the execution details of these queries, we can enable SQL tracing. This will log the time taken by each SQL statement.

```sql
-- Enable log duration and statement logging
SET log_statement = 'all';
SET log_duration = 'on';
```

#### **3.2 Use `EXPLAIN` to Analyze Query Plans**

Use `EXPLAIN` to analyze the execution plans of the inefficient queries. This will help you understand how PostgreSQL executes the query and where the performance bottlenecks are.

```sql
-- Analyze the first inefficient query
EXPLAIN SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE '%action%'
AND r.rental_date > NOW() - INTERVAL '30 days';

-- Analyze the second inefficient query
EXPLAIN SELECT c.first_name, c.last_name, p.amount, p.payment_date
FROM payment p
JOIN customer c ON c.customer_id = p.customer_id
WHERE p.payment_date BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY p.payment_date DESC;
```

#### **3.3 Review the Query Plans**

If the query plan shows **Seq Scan** or **Nested Loop** joins, the query is inefficient. For example:
```
Seq Scan on rental r  (cost=0.00..35.50 rows=100 width=50)
```

---

### **Step 4: Apply Basic Query Tuning**

Now let's optimize the queries by adding indexes and rewriting them for efficiency.

#### **4.1 Create Indexes**

We will create indexes on the columns that are frequently used in `JOIN` and `WHERE` clauses. This should help reduce the query execution time by making lookups faster.

```sql
-- Index on customer_id in rental table
CREATE INDEX idx_rental_customer_id ON rental(customer_id);

-- Index on rental_date in rental table
CREATE INDEX idx_rental_date ON rental(rental_date);

-- Index on film description
CREATE INDEX idx_film_description ON film(description);

-- Index on payment_date in payment table
CREATE INDEX idx_payment_date ON payment(payment_date);
```

#### **4.2 Rewrite Queries**

Rewriting inefficient queries can also help. For instance, avoid using `LIKE` with `%` at both ends of the string, as it causes a full table scan.

Optimized Query 1:
```sql
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE 'action%'  -- more specific matching
AND r.rental_date > NOW() - INTERVAL '30 days';
```

Optimized Query 2:
```sql
SELECT c.first_name, c.last_name, p.amount, p.payment_date
FROM payment p
JOIN customer c ON c.customer_id = p.customer_id
WHERE p.payment_date BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY p.payment_date DESC
LIMIT 100;
```

#### **4.3 Use `EXPLAIN` After Optimization**

Run the `EXPLAIN` command again to analyze the optimized queries.

```sql
EXPLAIN SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE 'action%'
AND r.rental_date > NOW() - INTERVAL '30 days';
```

After optimization, you should see more efficient operations like **Index Scan** instead of **Seq Scan**.

---

### **Step 5: Measure Performance After Tuning**

#### **5.1 Measure Query Execution Time**

After applying the optimizations, we will measure the execution time again using the logs and compare it with the pre

-optimization performance.

```sql
-- Run the optimized query and check the time
EXPLAIN ANALYZE
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE 'action%'
AND r.rental_date > NOW() - INTERVAL '30 days';
```

#### **5.2 Compare Before and After Performance**

Compare the query execution times before and after tuning. The optimized query should be significantly faster.

---

### **Step 6: Conclusion and Monitoring**

You have successfully created a database with sample data, identified and tuned inefficient queries, and measured the performance improvements. Always use tools like **SQL Trace**, **EXPLAIN ANALYZE**, and **pg_stat_statements** for ongoing monitoring of database performance.






### **Comprehensive PostgreSQL Performance Tuning Exercise (Advanced Level)**

This exercise will guide you through creating and populating a PostgreSQL database, identifying and fixing complex performance bottlenecks, and applying advanced performance tuning techniques. The focus will be on **Basic Query Tuning**, **SQL Trace**, **Query Plan Analysis**, **AWR/Statspack analysis**, and the **pg_stat_statements** extension for in-depth performance monitoring. 

---

### **Step 1: Setting Up the Database and Tables**

#### **1.1 Create the Database**

Let's begin by creating the PostgreSQL database and connecting to it:

```sql
-- Create the database
CREATE DATABASE dvdrental;

-- Connect to the newly created database
\c dvdrental;
```

#### **1.2 Create Tables**

We will create the following tables to simulate a real-world database for a DVD rental business:

- `customer`
- `film`
- `inventory`
- `rental`
- `payment`

```sql
-- Create the 'customer' table
CREATE TABLE customer (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    address VARCHAR(100),
    active BOOLEAN
);

-- Create the 'film' table
CREATE TABLE film (
    film_id SERIAL PRIMARY KEY,
    title VARCHAR(100),
    description TEXT,
    release_year INT,
    rental_duration INT,
    rental_rate DECIMAL(5,2)
);

-- Create the 'inventory' table
CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    film_id INT REFERENCES film(film_id),
    store_id INT,
    available BOOLEAN
);

-- Create the 'rental' table
CREATE TABLE rental (
    rental_id SERIAL PRIMARY KEY,
    rental_date TIMESTAMP,
    inventory_id INT REFERENCES inventory(inventory_id),
    customer_id INT REFERENCES customer(customer_id),
    return_date TIMESTAMP
);

-- Create the 'payment' table
CREATE TABLE payment (
    payment_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customer(customer_id),
    amount DECIMAL(5,2),
    payment_date TIMESTAMP
);
```

#### **1.3 Insert Sample Data**

Let's populate the tables with data. This time, we will insert a larger set of data and some records with unusual or edge-case values to simulate complex performance issues.

```sql
-- Insert sample data into the 'customer' table
INSERT INTO customer (first_name, last_name, email, address, active)
SELECT
    'First' || generate_series(1, 200),
    'Last' || generate_series(1, 200),
    'email' || generate_series(1, 200) || '@example.com',
    'Address ' || generate_series(1, 200),
    TRUE
;

-- Insert sample data into the 'film' table
INSERT INTO film (title, description, release_year, rental_duration, rental_rate)
SELECT
    'Film Title ' || generate_series(1, 50),
    'A description for Film ' || generate_series(1, 50),
    2000 + (random() * 20)::int,
    7 + (random() * 3)::int,
    1.99 + (random() * 2)::float
;

-- Insert sample data into the 'inventory' table
INSERT INTO inventory (film_id, store_id, available)
SELECT 
    (random() * 50)::int + 1, 
    (random() * 5)::int + 1,
    TRUE
FROM generate_series(1, 500);

-- Insert sample data into the 'rental' table
INSERT INTO rental (rental_date, inventory_id, customer_id, return_date)
SELECT 
    NOW() - (random() * 60)::int * interval '1 day', 
    (random() * 500)::int + 1,
    (random() * 200)::int + 1,
    NOW() - (random() * 20)::int * interval '1 day'
FROM generate_series(1, 1000);

-- Insert sample data into the 'payment' table
INSERT INTO payment (customer_id, amount, payment_date)
SELECT 
    (random() * 200)::int + 1,
    5 + (random() * 50)::float,
    NOW() - (random() * 60)::int * interval '1 day'
FROM generate_series(1, 500);
```

---

### **Step 2: Simulate Complex Inefficient Queries**

In this section, we will simulate complex, inefficient queries that participants will need to optimize.

```sql
-- Inefficient Query 1: Multiple JOINs with missing indexes and large datasets
SELECT c.first_name, c.last_name, f.title, p.amount, r.rental_date
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
JOIN payment p ON p.customer_id = c.customer_id
WHERE f.release_year BETWEEN 2005 AND 2010
AND p.payment_date BETWEEN '2022-01-01' AND '2022-12-31'
ORDER BY r.rental_date DESC;

-- Inefficient Query 2: A `WHERE` clause using the LIKE operator with wildcard at the start
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE '%action%' 
AND r.rental_date > NOW() - INTERVAL '60 days';

-- Inefficient Query 3: Aggregation and sorting without indexing
SELECT f.title, COUNT(r.rental_id) AS rental_count
FROM film f
LEFT JOIN rental r ON r.inventory_id = f.film_id
GROUP BY f.title
HAVING COUNT(r.rental_id) > 100
ORDER BY rental_count DESC;
```

---

### **Step 3: Identify Performance Bottlenecks**

Now that we have simulated inefficient queries, let's analyze their performance using various PostgreSQL tools.

#### **3.1 Enable SQL Trace and Log Duration**

To begin performance analysis, enable query tracing and logging:

```sql
-- Enable query duration logging and statement logging
SET log_statement = 'all';
SET log_duration = 'on';
```

You can also check the logs in the PostgreSQL log directory to see detailed query times.

#### **3.2 Use `EXPLAIN ANALYZE` to Investigate Query Plans**

Use the `EXPLAIN ANALYZE` command to understand how PostgreSQL is executing the queries. This will show whether PostgreSQL is using efficient **Index Scans** or **Seq Scans**, which are usually indicative of suboptimal performance.

```sql
-- Analyze Query 1 execution plan
EXPLAIN ANALYZE 
SELECT c.first_name, c.last_name, f.title, p.amount, r.rental_date
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
JOIN payment p ON p.customer_id = c.customer_id
WHERE f.release_year BETWEEN 2005 AND 2010
AND p.payment_date BETWEEN '2022-01-01' AND '2022-12-31'
ORDER BY r.rental_date DESC;

-- Analyze Query 2 execution plan
EXPLAIN ANALYZE 
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE '%action%' 
AND r.rental_date > NOW() - INTERVAL '60 days';

-- Analyze Query 3 execution plan
EXPLAIN ANALYZE
SELECT f.title, COUNT(r.rental_id) AS rental_count
FROM film f
LEFT JOIN rental r ON r.inventory_id = f.film_id
GROUP BY f.title
HAVING COUNT(r.rental_id) > 100
ORDER BY rental_count DESC;
```

#### **3.3 Review the Query Plans**

If the output shows something like **Seq Scan** or **Nested Loop**, it indicates that the query is inefficient. For example:

```
Seq Scan on rental r  (cost=0.00..35.50 rows=100 width=50)
```

---

### **Step 4: Apply Indexes and Rewrite Queries for Optimization**

In this step, participants will add indexes to the necessary columns and optimize the queries.

#### **4.1 Create Indexes**

Create indexes on frequently used columns in `JOIN` and `WHERE` clauses to reduce the number of table scans:

```sql
-- Create indexes to speed up the queries
CREATE INDEX idx_rental_customer_id ON rental(customer_id);
CREATE INDEX idx_payment_date ON payment(payment_date);
CREATE INDEX idx_film_description ON film(description);
CREATE INDEX idx_film_release_year ON film(release_year);
CREATE INDEX idx_inventory_film_id ON inventory(film_id);
```

#### **4.2 Rewrite Queries for Efficiency**

For the queries that use **`LIKE '%action%'`**, we will rewrite them to be more specific:

```sql
-- Optimized Query 1: Reduce scope of LIKE statement
SELECT c.first_name, c.last_name, f.title, p.amount, r.rental_date
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i

 ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
JOIN payment p ON p.customer_id = c.customer_id
WHERE f.description LIKE 'action%' -- Remove leading wildcard
AND r.rental_date > NOW() - INTERVAL '30 days'
ORDER BY r.rental_date DESC;
```

---

### **Step 5: Measure Before and After Performance**

#### **5.1 Measure the Before Performance**

Run `EXPLAIN ANALYZE` again to see the performance impact of the optimizations. Note the execution time, rows scanned, and index usage.

#### **5.2 Measure the After Performance**

Compare the query execution times after applying indexes and optimizations. You should observe significant improvements in query time and reduced disk I/O.

```sql
-- Measure execution time after optimization
EXPLAIN ANALYZE
SELECT c.first_name, c.last_name, f.title
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.description LIKE 'action%' 
AND r.rental_date > NOW() - INTERVAL '30 days';
```

#### **5.3 Measure Query Performance Using `pg_stat_statements`**

Use the `pg_stat_statements` extension to gather statistics on the queries:

```sql
-- Load pg_stat_statements extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- View query execution statistics
SELECT * FROM pg_stat_statements WHERE query LIKE '%rental%';
```

---

### **Step 6: Conclusion and Monitoring**

You have successfully tuned complex queries and improved the overall performance of your database. The key steps in performance tuning include:

1. **Identifying inefficiencies** using tools like `EXPLAIN ANALYZE`.
2. **Creating indexes** to reduce the cost of queries.
3. **Rewriting queries** for better performance.
4. **Using monitoring tools** such as SQL Trace and `pg_stat_statements` to continuously measure performance.

By regularly applying these steps, you can keep your PostgreSQL database running efficiently, even as your data grows.

---

