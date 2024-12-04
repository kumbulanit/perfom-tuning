d
### **Exercise Overview**
This practical exercise will guide you through:
1. Advanced indexing techniques.
2. Query optimization and analysis.
3. Table partitioning for performance gains.
4. Materialized views for optimizing complex queries.
5. Database normalization and denormalization trade-offs.
6. Use of PostgreSQL tools for performance diagnostics.

### **Prerequisites**
Ensure PostgreSQL is installed. If not, use:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### **Step-by-Step Advanced Exercise**

#### **Step 1: Setting Up the Database and Advanced Schema**
1. **Start PostgreSQL** and access the console:

   ```bash
   sudo service postgresql start
   sudo -u postgres psql
   ```
   
2. **Create a new database** called `advanced_performance_db`:

   ```sql
   CREATE DATABASE advanced_performance_db;
   ```
   
3. **Connect to the new database**:

   ```sql
   \c advanced_performance_db
   ```

4. **Create a normalized schema** with multiple tables. Start with an `orders` table that links to `customers` and `products` tables:
```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price NUMERIC(10, 2),
    stock INT DEFAULT 0
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    product_id INT REFERENCES products(product_id),
    order_date TIMESTAMP DEFAULT NOW(),
    quantity INT NOT NULL,
    total_price NUMERIC(10, 2)
);

-- Create the index on `order_date` separately
CREATE INDEX idx_order_date ON orders(order_date);
```
5. **Insert sample data** into the tables:

   ```sql
   INSERT INTO customers (first_name, last_name, email) VALUES
   ('John', 'Doe', 'john.doe@example.com'),
   ('Jane', 'Smith', 'jane.smith@example.com'),
   ('Paul', 'Adams', 'paul.adams@example.com'),
   ('Laura', 'White', 'laura.white@example.com');

   INSERT INTO products (product_name, category, price, stock) VALUES
   ('Laptop', 'Electronics', 1200.50, 10),
   ('Smartphone', 'Electronics', 800.00, 20),
   ('Desk Chair', 'Furniture', 150.00, 15),
   ('Coffee Table', 'Furniture', 120.00, 5);

   INSERT INTO orders (customer_id, product_id, quantity, total_price) VALUES
   (1, 1, 1, 1200.50),
   (2, 2, 2, 1600.00),
   (3, 4, 1, 120.00),
   (4, 3, 3, 450.00);
   ```

#### **Step 2: Query Optimization with Advanced Indexing**
1. **Analyze slow queries** using `EXPLAIN`:

   ```sql
   EXPLAIN SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';
   ```
   
   - Observe if the query is using a sequential scan or an index scan.

2. **Create a composite index** to improve query performance for range-based queries:

   ```sql
   CREATE INDEX idx_orders_customer_date ON orders (customer_id, order_date);
   ```

3. **Re-run the `EXPLAIN`** to check the improvement:

   ```sql
   EXPLAIN SELECT * FROM orders WHERE customer_id = 1 AND order_date BETWEEN '2023-01-01' AND '2023-12-31';
   ```

4. **Check index usage** by running `pg_stat_user_indexes`:

   ```sql
   SELECT indexrelname, idx_scan, idx_tup_read, idx_tup_fetch 
   FROM pg_stat_user_indexes 
   WHERE schemaname = 'public';
   ```

#### **Step 3: Partitioning for Performance**
1. **Partition the `orders` table** by year for improved query performance:

   ```sql
   CREATE TABLE orders_2023 PARTITION OF orders FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
   CREATE TABLE orders_2024 PARTITION OF orders FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
   ```

2. **Verify partitioning** by running a query:

   ```sql
   SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';
   ```

3. **Use `pg_partition_tree`** to verify the partition setup:

   ```sql
   SELECT * FROM pg_partition_tree('orders');
   ```

#### **Step 4: Use of Materialized Views**
1. **Create a materialized view** to optimize complex queries that aggregate data:

   ```sql
   CREATE MATERIALIZED VIEW customer_order_summary AS
   SELECT 
       c.customer_id, 
       c.first_name, 
       c.last_name, 
       COUNT(o.order_id) AS total_orders, 
       SUM(o.total_price) AS total_spent
   FROM 
       customers c
   JOIN 
       orders o ON c.customer_id = o.customer_id
   GROUP BY 
       c.customer_id, c.first_name, c.last_name;
   ```

2. **Query the materialized view** to get aggregated data:

   ```sql
   SELECT * FROM customer_order_summary WHERE total_spent > 1000;
   ```

3. **Refresh the materialized view** when data changes:

   ```sql
   REFRESH MATERIALIZED VIEW customer_order_summary;
   ```

#### **Step 5: Implementing Triggers for Performance Monitoring**
1. **Create an audit table** to track changes to `products`:

   ```sql
   CREATE TABLE products_audit (
       audit_id SERIAL PRIMARY KEY,
       product_id INT,
       action VARCHAR(10),
       changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       old_price NUMERIC(10, 2),
       new_price NUMERIC(10, 2)
   );
   ```

2. **Create a trigger function** to log price updates:

   ```sql
   CREATE OR REPLACE FUNCTION log_price_update() 
   RETURNS TRIGGER AS $$
   BEGIN
       IF OLD.price IS DISTINCT FROM NEW.price THEN
           INSERT INTO products_audit (product_id, action, old_price, new_price) 
           VALUES (NEW.product_id, 'UPDATE', OLD.price, NEW.price);
       END IF;
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;
   ```

3. **Attach the trigger to the `products` table**:

   ```sql
   CREATE TRIGGER product_price_update
   AFTER UPDATE ON products
   FOR EACH ROW
   EXECUTE FUNCTION log_price_update();
   ```

4. **Test the trigger** by updating a product's price:

   ```sql
   UPDATE products SET price = 1300.00 WHERE product_id = 1;
   ```

5. **Check the audit logs**:

   ```sql
   SELECT * FROM products_audit;
   ```

#### **Step 6: Using Performance Analysis Tools**
1. **Check for slow queries** using `pg_stat_statements`:

   ```sql
   CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
   SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 5;
   ```
   
2. **Check database size and index efficiency**:

   ```sql
   SELECT 
       table_name, 
       pg_size_pretty(pg_total_relation_size(table_name::regclass)) AS total_size,
       pg_size_pretty(pg_total_relation_size(table_name::regclass) - pg_relation_size(table_name::regclass)) AS index_size
   FROM 
       information_schema.tables 
   WHERE 
       table_schema = 'public'
   ORDER BY 
       pg_total_relation_size(table_name::regclass) DESC;
   ```

#### **Step 7: Optimization Best Practices**
1. **VACUUM and ANALYZE** your tables regularly to maintain performance:

   ```sql
   VACUUM FULL VERBOSE ANALYZE customers;
   VACUUM FULL VERBOSE ANALYZE products;
   VACUUM FULL VERBOSE ANALYZE orders;
   ```

2. **Adjust PostgreSQL configuration**:
   - Tune parameters like `shared_buffers`, `work_mem`, `effective_cache_size`, `random_page_cost`, and `checkpoint_timeout` based on server specs and workload.

#### **Step 8: Cleanup (Optional)**
1. **Remove the trigger** if not needed:

   ```sql
   DROP TRIGGER product_price_update ON products;
   ```

2. **Drop the materialized view**:

   ```sql
   DROP MATERIALIZED VIEW customer_order_summary;
   ```

3. **Drop the tables**:

   ```sql
   DROP TABLE products_audit;
   DROP TABLE orders;
   DROP TABLE products;
   DROP TABLE customers;
   ```

4. **Exit the PostgreSQL console**:

   ```sql
   \q


   ```

### **Learning Goals**
- Understand how to improve query performance through indexing and materialized views.
- Use partitioning to handle large datasets efficiently.
- Track data changes with triggers to monitor updates without affecting performance.
- Diagnose performance issues using PostgreSQL's built-in tools.

d


### **Step-by-Step Advanced Exercise - Part II**

#### **Step 1: Additional Data Insertion**
Let’s **populate the database** with a significant amount of data to simulate a realistic environment. This will involve inserting thousands of rows to analyze performance in larger datasets.

1. **Use a loop to bulk-insert customers and products**:

   ```sql
   -- Insert additional customers (10,000 entries)
   DO $$ 
   BEGIN 
       FOR i IN 1..10000 LOOP 
           INSERT INTO customers (first_name, last_name, email) 
           VALUES (
               'Customer_' || i, 
               'LastName_' || i, 
               'customer_' || i || '@example.com'
           ); 
       END LOOP; 
   END $$;

   -- Insert additional products (1,000 entries)
   DO $$ 
   BEGIN 
       FOR i IN 1..1000 LOOP 
           INSERT INTO products (product_name, category, price, stock) 
           VALUES (
               'Product_' || i, 
               CASE WHEN i % 2 = 0 THEN 'Electronics' ELSE 'Furniture' END, 
               (RANDOM() * 1000)::NUMERIC(10, 2), 
               (RANDOM() * 100)::INT
           ); 
       END LOOP; 
   END $$;
   ```

2. **Generate random orders** to create a large dataset:

   ```sql
   -- Insert random orders (50,000 entries)
   DO $$ 
   BEGIN 
       FOR i IN 1..50000 LOOP 
           INSERT INTO orders (customer_id, product_id, quantity, total_price, order_date) 
           VALUES (
               (RANDOM() * 10000)::INT + 1, 
               (RANDOM() * 1000)::INT + 1, 
               (RANDOM() * 10)::INT + 1, 
               ((RANDOM() * 1000)::NUMERIC(10, 2) * ((RANDOM() * 10)::INT + 1)),
               NOW() - (INTERVAL '1 day' * (RANDOM() * 365)::INT)
           ); 
       END LOOP; 
   END $$;
   ```

3. **Verify the number of rows** to ensure that the data is properly inserted:

   ```sql
   SELECT COUNT(*) FROM customers;  -- Should be 10,004 (including initial data)
   SELECT COUNT(*) FROM products;   -- Should be 1,004 (including initial data)
   SELECT COUNT(*) FROM orders;     -- Should be 50,004 (including initial data)
   ```

#### **Step 2: Advanced Indexing Techniques**
1. **Partial Indexes**: Create indexes that are only valid for a subset of data, useful for frequently filtered conditions.

   ```sql
   -- Create a partial index for recent high-value orders
   CREATE INDEX idx_high_value_recent_orders 
   ON orders (customer_id) 
   WHERE total_price > 500 AND order_date > NOW() - INTERVAL '1 year';
   ```

2. **Expression Indexes**: Use expressions within indexes to optimize specific query patterns.

   ```sql
   -- Create an index on the lowercased email to support case-insensitive searches
   CREATE INDEX idx_lower_email 
   ON customers (LOWER(email));
   ```

3. **Covering Indexes**: Create an index that includes additional columns to avoid additional lookups (PostgreSQL supports this starting from version 11).

   ```sql
   -- Create a covering index to optimize common queries on orders
   CREATE INDEX idx_orders_coverage 
   ON orders (order_date, customer_id) INCLUDE (total_price, quantity);
   ```

#### **Step 3: Query Performance Analysis**
1. **Use `EXPLAIN ANALYZE`** to compare query execution plans before and after creating indexes:

   ```sql
   EXPLAIN ANALYZE 
   SELECT * FROM orders 
   WHERE total_price > 500 AND order_date > NOW() - INTERVAL '6 months';
   ```

   - This will give you detailed insights into how long the query takes and what resources are consumed.

2. **Analyze specific scenarios** using `EXPLAIN (BUFFERS, ANALYZE)` to understand buffer usage:

   ```sql
   EXPLAIN (ANALYZE, BUFFERS) 
   SELECT customer_id, COUNT(*) 
   FROM orders 
   WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31'
   GROUP BY customer_id 
   ORDER BY COUNT(*) DESC LIMIT 10;
   ```

#### **Step 4: Normalization and Denormalization Trade-offs**
1. **Create a denormalized table** for faster read-heavy operations:

   ```sql
   CREATE TABLE denormalized_order_summary AS
   SELECT 
       o.order_id,
       c.first_name || ' ' || c.last_name AS customer_name,
       p.product_name,
       o.quantity,
       o.total_price,
       o.order_date
   FROM 
       orders o
   JOIN 
       customers c ON o.customer_id = c.customer_id
   JOIN 
       products p ON o.product_id = p.product_id;
   ```

2. **Run performance comparisons** between normalized and denormalized queries:

   ```sql
   EXPLAIN ANALYZE 
   SELECT * FROM denormalized_order_summary 
   WHERE total_price > 1000 AND order_date BETWEEN '2023-01-01' AND '2023-12-31';
   ```

3. **Evaluate trade-offs**: Denormalization improves read performance at the cost of storage efficiency and increased complexity in data consistency management.

#### **Step 5: Using Table Partitioning for Performance**
1. **Implement range partitioning** by order date to optimize queries filtering by date:

   ```sql
   CREATE TABLE orders_2025 PARTITION OF orders FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
   CREATE TABLE orders_2026 PARTITION OF orders FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');
   ```

2. **Add additional partitions** if needed, and **verify partition boundaries**:

   ```sql
   SELECT * FROM pg_partition_tree('orders');
   ```

3. **Test query performance** on partitioned data:

   ```sql
   EXPLAIN ANALYZE 
   SELECT * FROM orders 
   WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31';
   ```

#### **Step 6: Advanced Query Tuning Techniques**
1. **Optimize complex JOIN queries** by breaking them into Common Table Expressions (CTEs):

   ```sql
   WITH order_totals AS (
       SELECT 
           customer_id, 
           SUM(total_price) AS customer_total 
       FROM 
           orders 
       WHERE 
           order_date BETWEEN '2023-01-01' AND '2023-12-31'
       GROUP BY 
           customer_id
   )
   SELECT 
       c.first_name, 
       c.last_name, 
       ot.customer_total 
   FROM 
       order_totals ot 
   JOIN 
       customers c ON ot.customer_id = c.customer_id 
   WHERE 
       ot.customer_total > 2000;
   ```

2. **Analyze the performance of the query**:

   ```sql
   EXPLAIN ANALYZE 
   WITH order_totals AS (
       SELECT 
           customer_id, 
           SUM(total_price) AS customer_total 
       FROM 
           orders 
       WHERE 
           order_date BETWEEN '2023-01-01' AND '2023-12-31'
       GROUP BY 
           customer_id
   )
   SELECT 
       c.first_name, 
       c.last_name, 
       ot.customer_total 
   FROM 
       order_totals ot 
   JOIN 
       customers c ON ot.customer_id = c.customer_id 
   WHERE 
       ot.customer_total > 2000;
   ```

3. **Optimize aggregate queries** by using indexes with aggregation support:

   ```sql
   CREATE INDEX idx_customer_order_totals 
   ON orders (customer_id) INCLUDE (total_price);
   ```

4. **Check performance improvements** using:

   ```sql
   EXPLAIN ANALYZE 
   SELECT customer_id, SUM(total_price) 
   FROM orders 
   WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31'
   GROUP BY customer_id;
   ```

#### **Step 7: Advanced Use of PostgreSQL Performance Tools**
1. **Track performance metrics** using `pg_stat_activity`:

   ```sql
   SELECT 
       pid, 
       usename, 
       state, 
       query, 
       state_change 
   FROM 
       pg_stat_activity 
   WHERE 
       state != 'idle';
   ```

2. **Use `pg_buffercache`** to see how tables and indexes are cached:

   ```sql
   CREATE EXTENSION IF NOT EXISTS pg_buffercache;
   SELECT 
       relname, 
       count(*) AS buffers
   FROM 
       pg_buffercache 
   JOIN 
       pg_class ON pg_buffercache.relfilenode = pg_class.relfilenode 
   GROUP BY 
       relname 
   ORDER BY 
       buffers DESC LIMIT 10;
   ```

#### **Step 8: Advanced Table Maintenance**
1. **Run maintenance tasks** to keep the database optimized:

   ```sql


   -- Reindex tables
   REINDEX TABLE orders;

   -- Analyze to refresh statistics
   ANALYZE VERBOSE;

   -- Vacuum to reclaim space
   VACUUM FULL orders;
   ```

By following these steps, you will get a deeper understanding of advanced SQL techniques, their application in PostgreSQL, and how to maintain optimal database performance.



```sql
-- Inserting 10 customers
INSERT INTO customers (first_name, last_name, email, created_at)
VALUES
('John', 'Doe', 'john.doe@example.com', NOW()),
('Jane', 'Smith', 'jane.smith@example.com', NOW()),
('Michael', 'Johnson', 'michael.johnson@example.com', NOW()),
('Emily', 'Davis', 'emily.davis@example.com', NOW()),
('Chris', 'Brown', 'chris.brown@example.com', NOW()),
('Jessica', 'Taylor', 'jessica.taylor@example.com', NOW()),
('David', 'Miller', 'david.miller@example.com', NOW()),
('Sarah', 'Wilson', 'sarah.wilson@example.com', NOW()),
('Daniel', 'Moore', 'daniel.moore@example.com', NOW()),
('Laura', 'Anderson', 'laura.anderson@example.com', NOW());

-- Inserting sample products
INSERT INTO products (product_name, category, price, stock)
VALUES
('Laptop', 'Electronics', 999.99, 10),
('Smartphone', 'Electronics', 499.99, 15),
('Headphones', 'Accessories', 199.99, 25),
('Coffee Maker', 'Home Appliances', 79.99, 20),
('Bluetooth Speaker', 'Accessories', 59.99, 30);

-- Inserting orders with random dates between one year ago and today
-- Random dates generated using the `NOW()` function and `INTERVAL` subtraction
-- Sample order quantities and total price

INSERT INTO orders (customer_id, product_id, order_date, quantity, total_price)
VALUES
(1, 1, NOW() - INTERVAL '6 months', 2, 1999.98),
(1, 2, NOW() - INTERVAL '1 year', 1, 499.99),
(2, 3, NOW() - INTERVAL '9 months', 3, 599.97),
(2, 4, NOW() - INTERVAL '1 month', 1, 79.99),
(3, 1, NOW() - INTERVAL '10 months', 1, 999.99),
(3, 5, NOW() - INTERVAL '3 months', 2, 119.98),
(4, 2, NOW() - INTERVAL '4 months', 1, 499.99),
(5, 3, NOW() - INTERVAL '2 weeks', 2, 399.98),
(6, 4, NOW() - INTERVAL '5 months', 1, 79.99),
(7, 5, NOW() - INTERVAL '7 months', 1, 59.99),
(8, 1, NOW() - INTERVAL '1 year', 1, 999.99),
(9, 2, NOW() - INTERVAL '6 months', 2, 999.98),
(10, 3, NOW() - INTERVAL '3 months', 1, 199.99);
```

```sql
-- Create the partitioned orders table
CREATE TABLE orders_new (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    product_id INT REFERENCES products(product_id),
    order_date TIMESTAMP NOT NULL,
    quantity INT NOT NULL,
    total_price NUMERIC(10, 2)
) PARTITION BY RANGE (order_date);

```