# PostgreSQL Interview Questions & Answers

## 1. What is PostgreSQL and why use it?

**Q: Explain PostgreSQL**

A: PostgreSQL is a powerful, open-source relational database system (RDBMS). Known for reliability, features, and standards compliance.

**Key features**:

- ACID compliance (Atomicity, Consistency, Isolation, Durability)
- Advanced data types (JSON, arrays, ranges, etc.)
- Full-text search
- GIS (geographic data)
- Window functions
- Common Table Expressions (CTEs)
- Materialized views
- Stored procedures and functions
- Triggers
- Replication and high availability
- Partitioning
- Parallel query execution

**Why use PostgreSQL**:

- Reliable and stable
- Feature-rich (better than MySQL for complex queries)
- Great for complex data relationships
- Excellent performance with proper tuning
- Strong community and documentation
- Open source
- Scales well

**vs MySQL**: PostgreSQL has more advanced features, better compliance, better for complex queries

**vs NoSQL**: Relational model better for structured data with relationships

---

## 2. What are data types?

**Q: PostgreSQL data types**

A: PostgreSQL supports many data types.

**Numeric**:

```sql
SMALLINT              -- 2 bytes
INTEGER or INT        -- 4 bytes
BIGINT                -- 8 bytes
DECIMAL or NUMERIC    -- Exact, variable precision
REAL                  -- Single precision floating point
DOUBLE PRECISION      -- Double precision floating point
```

**String**:

```sql
CHAR(n)               -- Fixed-length string
VARCHAR(n)            -- Variable-length string
TEXT                  -- Unlimited-length string
```

**Date/Time**:

```sql
DATE                  -- Date only
TIME                  -- Time only
TIMESTAMP             -- Date and time
TIMESTAMPTZ           -- Date and time with timezone
INTERVAL              -- Time interval
```

**Boolean**:

```sql
BOOLEAN               -- true, false
```

**JSON**:

```sql
JSON                  -- Text-based JSON
JSONB                 -- Binary JSON (better performance)
```

**Arrays**:

```sql
INTEGER[]             -- Array of integers
TEXT[]                -- Array of text
```

**UUID**:

```sql
UUID                  -- Unique identifier
```

**Other**:

```sql
BYTEA                 -- Binary data
ENUM                  -- Enumerated type
```

**Example**:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email TEXT UNIQUE,
    age INTEGER CHECK (age >= 0),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metadata JSONB,
    tags TEXT[]
);
```

---

## 3. What are primary keys and foreign keys?

**Q: Relationships and constraints**

A:

**Primary key** — Uniquely identifies record:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255)
);

-- Or
CREATE TABLE users (
    id SERIAL,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    PRIMARY KEY (id)
);
```

**Foreign key** — References another table:

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    title VARCHAR(255),
    content TEXT
);

-- Or
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(255),
    content TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Referential integrity actions**:

```sql
-- CASCADE — delete child when parent deleted
ON DELETE CASCADE

-- SET NULL — set to NULL when parent deleted
ON DELETE SET NULL

-- RESTRICT — prevent deletion
ON DELETE RESTRICT

-- SET DEFAULT — set to default value
ON DELETE SET DEFAULT
```

**Composite primary key**:

```sql
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

---

## 4. What are indexes?

**Q: Optimize queries with indexes**

A: Index speeds up data retrieval.

**Create index**:

```sql
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_user_email ON posts(user_id, created_at);
```

**Index types**:

```sql
-- B-tree (default, best for most cases)
CREATE INDEX idx_name ON users(name);

-- Hash (equality only)
CREATE INDEX idx_hash ON users USING HASH(email);

-- GiST (for geometric types, full-text search)
CREATE INDEX idx_search ON users USING GiST(search_text);

-- GIN (good for arrays, JSONB, full-text)
CREATE INDEX idx_tags ON posts USING GIN(tags);

-- BRIN (large tables, good compression)
CREATE INDEX idx_timestamps ON events USING BRIN(created_at);
```

**Unique index**:

```sql
CREATE UNIQUE INDEX idx_unique_email ON users(email);
```

**Partial index** (only on matching rows):

```sql
CREATE INDEX idx_active_users ON users(id) WHERE is_active = true;
```

**Drop index**:

```sql
DROP INDEX idx_email;
```

**Analyze indexes**:

```sql
-- Check if index is used
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
```

---

## 5. What are SELECT queries?

**Q: Basic SELECT statements**

A:

```sql
-- Basic select
SELECT * FROM users;
SELECT id, name, email FROM users;

-- WHERE clause
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE name = 'John' AND age < 30;
SELECT * FROM users WHERE status IN ('active', 'pending');
SELECT * FROM users WHERE name LIKE 'J%';  -- Starts with J
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- Ends with

-- ORDER BY
SELECT * FROM users ORDER BY name ASC;
SELECT * FROM users ORDER BY age DESC, name ASC;

-- LIMIT and OFFSET
SELECT * FROM users LIMIT 10;
SELECT * FROM users LIMIT 10 OFFSET 20;  -- Page 3 (page size 10)

-- Aliases
SELECT id AS user_id, name AS user_name FROM users;
SELECT COUNT(*) AS total_users FROM users;

-- DISTINCT
SELECT DISTINCT country FROM users;

-- CASE statement
SELECT
    name,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age < 65 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group
FROM users;

-- String functions
SELECT UPPER(name), LOWER(email), LENGTH(name) FROM users;
SELECT SUBSTRING(name, 1, 3) FROM users;  -- First 3 characters
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- Math functions
SELECT age, ROUND(salary / 12) AS monthly_salary FROM users;
SELECT ABS(-5), CEIL(3.2), FLOOR(3.8);

-- Date functions
SELECT NOW(), CURRENT_DATE, CURRENT_TIME;
SELECT DATE_TRUNC('month', created_at) FROM users;
SELECT EXTRACT(YEAR FROM created_at) FROM users;
```

---

## 6. What are JOINs?

**Q: Combine data from multiple tables**

A:

```sql
-- INNER JOIN (records in both tables)
SELECT users.name, posts.title
FROM users
INNER JOIN posts ON users.id = posts.user_id;

-- LEFT JOIN (all from left table)
SELECT users.name, posts.title
FROM users
LEFT JOIN posts ON users.id = posts.user_id;

-- RIGHT JOIN (all from right table)
SELECT users.name, posts.title
FROM users
RIGHT JOIN posts ON users.id = posts.user_id;

-- FULL OUTER JOIN (all from both tables)
SELECT users.name, posts.title
FROM users
FULL OUTER JOIN posts ON users.id = posts.user_id;

-- CROSS JOIN (Cartesian product)
SELECT users.name, products.name
FROM users
CROSS JOIN products;

-- Multiple joins
SELECT
    users.name,
    posts.title,
    comments.content
FROM users
INNER JOIN posts ON users.id = posts.user_id
INNER JOIN comments ON posts.id = comments.post_id;

-- Self join
SELECT
    e1.name AS employee,
    e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

**Join visualization**:

```
INNER JOIN     LEFT JOIN      RIGHT JOIN     FULL OUTER
    X              X              X              X
   / \            / \            / \           / \
  A   B          A   B          A   B         A   B

Only    A&B     All A+     All B+     All A&B
both             match B    match A    unmatched
```

---

## 7. What are aggregate functions?

**Q: Summarize data**

A:

```sql
-- Aggregate functions
SELECT COUNT(*) FROM users;                    -- 1000
SELECT COUNT(DISTINCT country) FROM users;    -- 45
SELECT SUM(salary) FROM employees;            -- Total salary
SELECT AVG(age) FROM users;                   -- Average age
SELECT MAX(salary) FROM employees;            -- Highest salary
SELECT MIN(age) FROM users;                   -- Youngest user

-- Multiple aggregates
SELECT
    COUNT(*) AS total,
    COUNT(DISTINCT user_id) AS unique_users,
    AVG(amount) AS avg_amount,
    SUM(amount) AS total_amount,
    MAX(amount) AS max_amount
FROM orders;

-- GROUP BY
SELECT
    country,
    COUNT(*) AS user_count,
    AVG(age) AS avg_age
FROM users
GROUP BY country;

-- GROUP BY multiple columns
SELECT
    country,
    city,
    COUNT(*) AS total
FROM users
GROUP BY country, city;

-- HAVING (filter groups)
SELECT
    country,
    COUNT(*) AS user_count
FROM users
GROUP BY country
HAVING COUNT(*) > 10;  -- Only countries with > 10 users

-- Complex example
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    status,
    COUNT(*) AS total,
    AVG(amount) AS avg_amount
FROM orders
WHERE created_at >= '2023-01-01'
GROUP BY EXTRACT(YEAR FROM created_at), status
HAVING COUNT(*) > 5
ORDER BY year DESC, total DESC;
```

---

## 8. What are subqueries?

**Q: Queries within queries**

A:

```sql
-- Scalar subquery (returns single value)
SELECT name, age FROM users
WHERE age > (SELECT AVG(age) FROM users);

-- IN subquery
SELECT * FROM users
WHERE id IN (SELECT user_id FROM posts WHERE category = 'tech');

-- EXISTS subquery
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM posts WHERE user_id = u.id);

-- NOT EXISTS
SELECT * FROM users u
WHERE NOT EXISTS (SELECT 1 FROM posts WHERE user_id = u.id);

-- Subquery in FROM clause
SELECT * FROM (
    SELECT id, name, ROUND(salary / 12) AS monthly FROM employees
) subq
WHERE monthly > 5000;

-- Correlated subquery
SELECT
    u.name,
    (SELECT COUNT(*) FROM posts WHERE user_id = u.id) AS post_count
FROM users u;

-- Multiple subqueries
SELECT * FROM users
WHERE age > (SELECT AVG(age) FROM users)
AND country IN (SELECT country FROM top_countries);
```

---

## 9. What are CTEs (WITH clause)?

**Q: Common Table Expressions**

A: Named temporary result sets.

```sql
-- Simple CTE
WITH user_posts AS (
    SELECT user_id, COUNT(*) AS post_count
    FROM posts
    GROUP BY user_id
)
SELECT u.name, up.post_count
FROM users u
JOIN user_posts up ON u.id = up.user_id;

-- Multiple CTEs
WITH
user_posts AS (
    SELECT user_id, COUNT(*) AS post_count
    FROM posts
    GROUP BY user_id
),
user_comments AS (
    SELECT user_id, COUNT(*) AS comment_count
    FROM comments
    GROUP BY user_id
)
SELECT
    u.name,
    COALESCE(up.post_count, 0) AS posts,
    COALESCE(uc.comment_count, 0) AS comments
FROM users u
LEFT JOIN user_posts up ON u.id = up.user_id
LEFT JOIN user_comments uc ON u.id = uc.user_id;

-- Recursive CTE
WITH RECURSIVE numbers(n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 10
)
SELECT * FROM numbers;

-- Finding hierarchy
WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    INNER JOIN subordinates ON e.manager_id = subordinates.id
)
SELECT * FROM subordinates;
```

---

## 10. What are window functions?

**Q: Advanced analytics**

A: Perform calculations across rows.

```sql
-- ROW_NUMBER
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- RANK (with ties)
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK (no gaps)
SELECT
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- PARTITION BY (groups)
SELECT
    department,
    name,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- LAG and LEAD (previous/next row)
SELECT
    name,
    salary,
    LAG(salary) OVER (ORDER BY hire_date) AS prev_salary,
    LEAD(salary) OVER (ORDER BY hire_date) AS next_salary
FROM employees;

-- Running total
SELECT
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;

-- Average over window
SELECT
    name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees;

-- FIRST_VALUE and LAST_VALUE
SELECT
    department,
    name,
    salary,
    FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary) AS min_salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY salary
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS max_salary
FROM employees;
```

---

## 11. What are INSERT, UPDATE, DELETE?

**Q: Modify data**

A:

```sql
-- INSERT
INSERT INTO users (name, email, age)
VALUES ('John', 'john@example.com', 30);

-- Multiple inserts
INSERT INTO users (name, email, age)
VALUES
    ('John', 'john@example.com', 30),
    ('Jane', 'jane@example.com', 28),
    ('Bob', 'bob@example.com', 35);

-- INSERT from SELECT
INSERT INTO archive_users (name, email)
SELECT name, email FROM users WHERE deleted = true;

-- UPDATE
UPDATE users SET age = 31 WHERE id = 1;

-- UPDATE multiple columns
UPDATE users
SET age = 31, status = 'active'
WHERE id = 1;

-- UPDATE with expression
UPDATE employees
SET salary = salary * 1.1
WHERE department = 'Sales';

-- UPDATE from another table
UPDATE users
SET premium = true
FROM premium_list
WHERE users.id = premium_list.user_id;

-- DELETE
DELETE FROM users WHERE id = 1;

-- DELETE all
DELETE FROM users;

-- DELETE with condition
DELETE FROM posts
WHERE created_at < '2020-01-01';

-- RETURNING (get deleted/updated rows)
DELETE FROM users WHERE id = 1 RETURNING *;
UPDATE users SET age = 31 WHERE id = 1 RETURNING id, name, age;
```

---

## 12. What are transactions?

**Q: ACID properties**

A: Ensure data consistency.

```sql
-- Transaction
BEGIN;
    INSERT INTO accounts (name, balance) VALUES ('Alice', 1000);
    INSERT INTO accounts (name, balance) VALUES ('Bob', 1000);
COMMIT;

-- Rollback
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';
    -- If error occurs
    ROLLBACK;
COMMIT;

-- Savepoint
BEGIN;
    INSERT INTO log VALUES ('Step 1');
    SAVEPOINT sp1;

    INSERT INTO log VALUES ('Step 2');
    ROLLBACK TO sp1;  -- Undo Step 2

    INSERT INTO log VALUES ('Step 2 again');
COMMIT;

-- Isolation levels
-- READ UNCOMMITTED: Dirty reads possible
-- READ COMMITTED: Default, no dirty reads
-- REPEATABLE READ: Phantom reads possible
-- SERIALIZABLE: Strictest

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN;
    -- transaction
COMMIT;
```

---

## 13. What are stored procedures?

**Q: Database functions and procedures**

A:

```sql
-- Simple function
CREATE OR REPLACE FUNCTION get_user_age(user_id INTEGER)
RETURNS INTEGER AS $$
    SELECT age FROM users WHERE id = user_id;
$$ LANGUAGE SQL;

SELECT get_user_age(1);

-- Function with logic
CREATE OR REPLACE FUNCTION calculate_bonus(salary NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
    IF salary < 50000 THEN
        RETURN salary * 0.05;
    ELSIF salary < 100000 THEN
        RETURN salary * 0.1;
    ELSE
        RETURN salary * 0.15;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Function returning multiple columns
CREATE OR REPLACE FUNCTION get_user_stats(user_id INTEGER)
RETURNS TABLE(name VARCHAR, post_count INT, comment_count INT) AS $$
    SELECT
        u.name,
        COUNT(DISTINCT p.id) AS post_count,
        COUNT(DISTINCT c.id) AS comment_count
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    LEFT JOIN comments c ON u.id = c.user_id
    WHERE u.id = user_id
    GROUP BY u.id, u.name;
$$ LANGUAGE SQL;

SELECT * FROM get_user_stats(1);

-- Procedure
CREATE OR REPLACE PROCEDURE transfer_money(
    from_account INTEGER,
    to_account INTEGER,
    amount NUMERIC
) AS $$
BEGIN
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;
    COMMIT;
END;
$$ LANGUAGE plpgsql;

CALL transfer_money(1, 2, 100);
```

---

## 14. What are triggers?

**Q: Automatic actions**

A:

```sql
-- Trigger to update timestamp
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_modified
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_modified_time();

-- Trigger for audit log
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR,
    operation VARCHAR,
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_data)
        VALUES (TG_TABLE_NAME, 'INSERT', row_to_json(NEW));
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_data, new_data)
        VALUES (TG_TABLE_NAME, 'UPDATE', row_to_json(OLD), row_to_json(NEW));
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_data)
        VALUES (TG_TABLE_NAME, 'DELETE', row_to_json(OLD));
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION audit_trigger();
```

---

## 15. What are views?

**Q: Virtual tables**

A: Stored query results.

```sql
-- Simple view
CREATE VIEW user_summary AS
SELECT
    id,
    name,
    email,
    COUNT(posts.id) AS post_count
FROM users
LEFT JOIN posts ON users.id = posts.user_id
GROUP BY users.id, users.name, users.email;

SELECT * FROM user_summary;

-- Materialized view (cached results)
CREATE MATERIALIZED VIEW user_stats AS
SELECT
    u.id,
    u.name,
    COUNT(p.id) AS post_count,
    COUNT(c.id) AS comment_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
LEFT JOIN comments c ON u.id = c.user_id
GROUP BY u.id, u.name;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW user_stats;

-- Drop view
DROP VIEW user_summary;
DROP MATERIALIZED VIEW user_stats;
```

---

## 16. What is normalization?

**Q: Database design**

A: Organize data to reduce redundancy.

**1NF — Atomic values**:

```sql
-- BAD: Repeating group
CREATE TABLE employees_bad (
    id INT,
    name VARCHAR,
    skills VARCHAR  -- "SQL, Python, Go"
);

-- GOOD: Atomic values
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR
);

CREATE TABLE skills (
    id INT PRIMARY KEY,
    employee_id INT REFERENCES employees(id),
    skill VARCHAR
);
```

**2NF — Eliminate partial dependencies**:

```sql
-- BAD: Non-key attribute depends on partial key
CREATE TABLE orders_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR,  -- Depends only on product_id
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- GOOD: Separate table
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    product_id INT REFERENCES products(id),
    quantity INT
);
```

**3NF — Eliminate transitive dependencies**:

```sql
-- BAD: Non-key attribute depends on another non-key
CREATE TABLE employees_bad (
    id INT PRIMARY KEY,
    name VARCHAR,
    department_id INT,
    department_name VARCHAR,  -- Transitive dependency
    department_location VARCHAR
);

-- GOOD: Separate table
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR,
    location VARCHAR
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR,
    department_id INT REFERENCES departments(id)
);
```

---

## 17. What is EXPLAIN and query optimization?

**Q: Analyze and optimize queries**

A:

```sql
-- EXPLAIN (show query plan)
EXPLAIN SELECT * FROM users WHERE id = 1;

-- EXPLAIN ANALYZE (execute and show actual stats)
EXPLAIN ANALYZE SELECT * FROM users WHERE id = 1;

-- Query plan example output:
-- Seq Scan on users  (cost=0.00..35.50 rows=1 width=32)
--   Filter: (id = 1)

-- EXPLAIN ANALYZE better output:
-- Seq Scan on users (cost=0.00..35.50 rows=1 width=32)
--   (actual time=0.015..0.015 rows=1 loops=1)

-- Key metrics
-- cost: Arbitrary units
-- rows: Estimated/actual row count
-- width: Average bytes per row
-- time: Actual execution time (milliseconds)

-- Common scan types:
-- Seq Scan: Table scan (slow for large tables)
-- Index Scan: Using index (fast)
-- Index Only Scan: Index contains all needed data (fastest)
-- Bitmap Index Scan: Multiple indexes
-- Hash Join: Hash table join
-- Nested Loop: For small results

-- Optimization tips:
-- 1. Add indexes on WHERE columns
CREATE INDEX idx_email ON users(email);

-- 2. Use LIMIT for large results
SELECT * FROM users LIMIT 10;  -- Better than SELECT *

-- 3. Index on join columns
CREATE INDEX idx_post_user ON posts(user_id);

-- 4. Avoid functions in WHERE
SELECT * FROM users WHERE UPPER(email) = 'JOHN@EXAMPLE.COM';  -- Bad
SELECT * FROM users WHERE email = 'john@example.com';  -- Good

-- 5. Use appropriate data types
-- VARCHAR vs CHAR for strings
-- SMALLINT vs INTEGER

-- 6. Partition large tables
CREATE TABLE events_2024 PARTITION OF events
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

---

## 18. What is replication and backup?

**Q: High availability and disaster recovery**

A:

**Backup**:

```bash
# Full backup
pg_dump database_name > backup.sql

# Compressed backup
pg_dump -Fc database_name > backup.dump

# Restore
psql database_name < backup.sql
pg_restore -d database_name backup.dump

# Continuous archiving (WAL)
archive_command = 'cp %p /backup/wal_archive/%f'
```

**Replication setup** (basic):

```bash
# On primary (master)
# In postgresql.conf:
wal_level = replica
max_wal_senders = 3
max_replication_slots = 3

# In pg_hba.conf:
host replication repuser 192.168.1.2/32 md5

# Create replication user
CREATE ROLE repuser WITH REPLICATION ENCRYPTED PASSWORD 'password' LOGIN;

# On standby (replica)
pg_basebackup -h primary_host -U repuser -D /var/lib/postgresql/data -Pv -W

# Create recovery.conf
standby_mode = 'on'
primary_conninfo = 'host=primary_host user=repuser password=password'
```

---

## 19. What are full-text search?

**Q: Search documents**

A:

```sql
-- Create table with text search
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title VARCHAR,
    content TEXT,
    search_vector tsvector
);

-- Generate search vector
UPDATE documents SET search_vector =
    to_tsvector('english', title || ' ' || content);

-- Create index for speed
CREATE INDEX idx_search ON documents USING GIN(search_vector);

-- Query
SELECT * FROM documents
WHERE search_vector @@ to_tsquery('english', 'database & search');

-- OR query
SELECT * FROM documents
WHERE search_vector @@ to_tsquery('english', 'database | search');

-- NOT query
SELECT * FROM documents
WHERE search_vector @@ to_tsquery('english', 'database & !cache');

-- Rank results
SELECT
    id,
    title,
    ts_rank(search_vector, query) AS rank
FROM documents,
to_tsquery('english', 'database & search') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

---

## 20. What are JSON operations?

**Q: Work with JSON data**

A:

```sql
-- Store JSON
CREATE TABLE users (
    id INT,
    name VARCHAR,
    metadata JSONB
);

INSERT INTO users VALUES (1, 'John', '{"age": 30, "city": "NYC", "hobbies": ["reading", "coding"]}');

-- Extract value
SELECT metadata->>'age' FROM users;      -- String: "30"
SELECT metadata->'age' FROM users;       -- JSON: 30

-- Extract nested
SELECT metadata->'hobbies'->>0 FROM users;  -- "reading"

-- Check key exists
SELECT * FROM users WHERE metadata ? 'age';

-- Array operations
SELECT * FROM users WHERE metadata->'hobbies' ? 'coding';

-- Update JSON
UPDATE users SET metadata = jsonb_set(metadata, '{age}', '31') WHERE id = 1;

-- Merge JSON
UPDATE users SET metadata = metadata || '{"verified": true}' WHERE id = 1;

-- Convert to records
SELECT jsonb_to_record(metadata) AS data(age INT, city VARCHAR) FROM users;

-- Array functions
SELECT jsonb_array_elements(metadata->'hobbies') FROM users;
```

---

## 21. What are security best practices?

**Q: Secure PostgreSQL**

A:

```sql
-- Create non-superuser role
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password';

-- Grant specific permissions
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON table_name TO app_user;

-- Revoke permissions
REVOKE INSERT, UPDATE ON table_name FROM app_user;

-- Row-level security
CREATE POLICY user_policy ON users
USING (id = current_user_id());

ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Encrypt password in pg_hba.conf
# md5: hash passwords
# scram-sha-256: better hashing
host all all 127.0.0.1/32 scram-sha-256

-- Enable SSL
ssl = on
ssl_cert_file = '/path/to/cert.pem'
ssl_key_file = '/path/to/key.pem'

-- Restrict connections
listen_addresses = 'localhost,192.168.1.10'

-- Audit logging
log_statement = 'all'
log_connections = on
log_disconnections = on
```

---

## 22. What is connection pooling?

**Q: Improve performance**

A: Reuse connections instead of creating new ones.

```bash
# Using PgBouncer
# pgbouncer.ini:
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25

# Start pgbouncer
pgbouncer pgbouncer.ini
```

**In application**:

```python
# Python with psycopg2
from psycopg2 import pool

connection_pool = pool.SimpleConnectionPool(1, 20,
    host='localhost',
    database='mydb',
    user='user',
    password='password'
)

conn = connection_pool.getconn()
try:
    # Use connection
    pass
finally:
    connection_pool.putconn(conn)
```

---

## 23. What are common mistakes?

**Q: Avoid these**

A:

**1. No indexes**:

```sql
-- BAD: Slow queries
SELECT * FROM users WHERE email = 'john@example.com';

-- GOOD: Add index
CREATE INDEX idx_email ON users(email);
```

**2. SELECT \* in application**:

```sql
-- BAD: Fetch unnecessary columns
SELECT * FROM users;

-- GOOD: Fetch only needed columns
SELECT id, name, email FROM users;
```

**3. N+1 queries**:

```sql
-- BAD: Loop in application
users = SELECT * FROM users;
FOR each user:
    posts = SELECT * FROM posts WHERE user_id = user.id;

-- GOOD: Single query with JOIN
SELECT u.*, p.* FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

**4. Missing foreign key constraints**:

```sql
-- BAD: No referential integrity
CREATE TABLE posts (
    id INT,
    user_id INT,
    title VARCHAR
);

-- GOOD: Foreign key
CREATE TABLE posts (
    id INT,
    user_id INT REFERENCES users(id),
    title VARCHAR
);
```

**5. Not using transactions**:

```sql
-- BAD: Partial updates if error
UPDATE account1 SET balance = balance - 100;
UPDATE account2 SET balance = balance + 100;  -- May fail

-- GOOD: Transaction ensures all or nothing
BEGIN;
    UPDATE account1 SET balance = balance - 100;
    UPDATE account2 SET balance = balance + 100;
COMMIT;
```

---

## 24. What is performance tuning?

**Q: Optimize PostgreSQL**

A:

```sql
-- Check server parameters
SHOW max_connections;
SHOW shared_buffers;
SHOW work_mem;
SHOW effective_cache_size;

-- Recommendations in postgresql.conf:
shared_buffers = 1GB              -- 25% of RAM
effective_cache_size = 4GB        -- 50-75% of RAM
work_mem = 50MB                   -- Per sort operation
maintenance_work_mem = 250MB
random_page_cost = 1.1            -- For SSD

-- Vacuum and analyze
VACUUM;              -- Reclaim space
ANALYZE;             -- Update statistics
VACUUM ANALYZE;      -- Both

-- Rebuild indexes
REINDEX TABLE table_name;

-- Find slow queries
SELECT query, calls, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Check table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

## 25. What is real-world example?

**Q: Complete database schema**

A:

```sql
-- Users table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);

-- Posts table
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    category VARCHAR(50),
    published BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_posts_user ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);

-- Comments table
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_comments_user ON comments(user_id);

-- View for post summary
CREATE VIEW post_summary AS
SELECT
    p.id,
    p.title,
    u.name AS author,
    COUNT(c.id) AS comment_count,
    p.created_at
FROM posts p
JOIN users u ON p.user_id = u.id
LEFT JOIN comments c ON p.id = c.post_id
GROUP BY p.id, p.title, u.name;

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_timestamp
BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

CREATE TRIGGER update_posts_timestamp
BEFORE UPDATE ON posts
FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- Sample queries
SELECT * FROM post_summary ORDER BY created_at DESC;

SELECT u.name, COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name
ORDER BY post_count DESC;
```

---

## PostgreSQL Interview Tips

1. **ACID properties** — Understand transactions
2. **Indexes** — Know when to use them
3. **Query optimization** — EXPLAIN ANALYZE
4. **JOINs** — Different types and when to use
5. **Window functions** — Advanced analytics
6. **CTEs** — Readable complex queries
7. **Foreign keys** — Referential integrity
8. **Normalization** — Database design
9. **Transactions** — ACID transactions
10. **Full-text search** — Text searching
11. **JSON support** — Modern data types
12. **Security** — Roles, permissions, encryption
13. **Backup and recovery** — High availability
14. **Performance tuning** — Optimization techniques
15. **Real projects** — Production experience
