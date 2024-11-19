### **Exercise Overview**
This practical exercise focuses on:
1. Creating a database and tables.
2. Performing basic SQL queries.
3. Creating and managing indexes.
4. Implementing a trigger for auditing changes.
5. Performance tuning tips and analysis.

### **Prerequisites**
Ensure you have PostgreSQL installed on your Linux machine. You can install it using the following commands:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### **Step-by-Step Exercise**

#### **Step 1: Setting Up the Database and Table**
1. **Start PostgreSQL**:
   
   ```bash
   sudo service postgresql start
   ```
   
2. **Access the PostgreSQL console**:
   
   ```bash
   sudo -u postgres psql
   ```
   
3. **Create a new database** called `performance_db`:

   ```sql
   CREATE DATABASE performance_db;
   ```
   
4. **Connect to the new database**:

   ```sql
   \c performance_db
   ```
   
5. **Create a table** called `employees` to store some basic information:

   ```sql
   CREATE TABLE employees (
       employee_id SERIAL PRIMARY KEY,
       first_name VARCHAR(50) NOT NULL,
       last_name VARCHAR(50) NOT NULL,
       department VARCHAR(50),
       hire_date DATE,
       salary NUMERIC(10, 2)
   );
   ```

6. **Insert sample data**:

   ```sql
   INSERT INTO employees (first_name, last_name, department, hire_date, salary) VALUES
   ('John', 'Doe', 'HR', '2021-01-15', 55000),
   ('Jane', 'Smith', 'Engineering', '2020-03-10', 75000),
   ('Paul', 'Brown', 'Finance', '2019-08-05', 68000),
   ('Laura', 'Johnson', 'Marketing', '2018-06-25', 62000),
   ('Michael', 'Wilson', 'Engineering', '2022-11-12', 80000);
   ```

#### **Step 2: Basic Queries**
1. **Select all records** from the `employees` table:

   ```sql
   SELECT * FROM employees;
   ```
   
2. **Filter records** by department:

   ```sql
   SELECT * FROM employees WHERE department = 'Engineering';
   ```
   
3. **Order records** by salary in descending order:

   ```sql
   SELECT * FROM employees ORDER BY salary DESC;
   ```

4. **Update a record** (increase salary by 10% for employees in Engineering):

   ```sql
   UPDATE employees SET salary = salary * 1.10 WHERE department = 'Engineering';
   ```
   
5. **Delete a record** (remove an employee by `employee_id`):

   ```sql
   DELETE FROM employees WHERE employee_id = 3;
   ```

#### **Step 3: Performance Tuning - Indexing**
1. **Create an index** on the `department` column to improve search performance:

   ```sql
   CREATE INDEX idx_department ON employees (department);
   ```

2. **Analyze query performance** using `EXPLAIN`:

   ```sql
   EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
   ```
   
   - `EXPLAIN` provides the query execution plan and helps identify if the index is being used.

3. **Remove an index** if it's not useful:

   ```sql
   DROP INDEX idx_department;
   ```

#### **Step 4: Using Triggers for Auditing**
1. **Create an audit table** to track changes:

   ```sql
   CREATE TABLE employees_audit (
       audit_id SERIAL PRIMARY KEY,
       employee_id INT,
       change_type VARCHAR(10),
       changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

2. **Create a trigger function** to log updates:

   ```sql
   CREATE OR REPLACE FUNCTION audit_employee_updates() 
   RETURNS TRIGGER AS $$
   BEGIN
       IF TG_OP = 'UPDATE' THEN
           INSERT INTO employees_audit (employee_id, change_type) 
           VALUES (NEW.employee_id, 'UPDATE');
       END IF;
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;
   ```

3. **Attach the trigger to the `employees` table**:

   ```sql
   CREATE TRIGGER employee_update_audit
   AFTER UPDATE ON employees
   FOR EACH ROW
   EXECUTE FUNCTION audit_employee_updates();
   ```

4. **Test the trigger** by updating an employee's information:

   ```sql
   UPDATE employees SET salary = salary + 5000 WHERE employee_id = 1;
   ```

5. **Verify the audit logs**:

   ```sql
   SELECT * FROM employees_audit;
   ```

#### **Step 5: Checking for Performance Issues**
1. **Check for slow queries** using the `pg_stat_activity` view:

   ```sql
   SELECT pid, usename, application_name, query, state, 
          wait_event_type, wait_event, query_start 
   FROM pg_stat_activity 
   WHERE state != 'idle' 
   ORDER BY query_start;
   ```
   
2. **Identify bloated tables**:

   ```sql
   SELECT 
       schemaname, 
       relname, 
       pg_size_pretty(pg_total_relation_size(relid)) AS total_size 
   FROM pg_stat_user_tables 
   ORDER BY pg_total_relation_size(relid) DESC;
   ```

#### **Step 6: Optimization Best Practices**
1. **VACUUM** and **ANALYZE** tables to maintain performance:

   ```sql
   VACUUM FULL;
   ANALYZE;
   ```

   - `VACUUM` removes dead tuples and reclaims storage space.
   - `ANALYZE` updates statistics to improve query planning.

2. **Optimize queries** using indexes effectively:
   - Avoid creating unnecessary indexes, as they increase write time.
   - Use `EXPLAIN ANALYZE` for detailed query execution analysis.

3. **Tune PostgreSQL settings**:
   - Adjust parameters like `work_mem`, `shared_buffers`, and `maintenance_work_mem` in `postgresql.conf` based on server capacity and workload.

#### **Step 7: Cleanup (Optional)**
1. **Remove the trigger** if no longer needed:

   ```sql
   DROP TRIGGER employee_update_audit ON employees;
   ```

2. **Drop the tables**:

   ```sql
   DROP TABLE employees_audit;
   DROP TABLE employees;
   ```

3. **Exit the PostgreSQL console**:

   ```sql
   \q
   ```

### **Conclusion**
This exercise demonstrates how to:
- Create and manipulate tables in PostgreSQL.
- Use basic queries and apply optimizations.
- Implement and test triggers for auditing.
- Analyze query performance and tune accordingly.
