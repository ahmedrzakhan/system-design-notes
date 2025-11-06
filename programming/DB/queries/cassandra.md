# Apache Cassandra 50 Practice Questions

_Complete Cassandra CQL Practice Guide for Local Environment_

## 🚀 Setup Instructions

1. **Install Apache Cassandra** (if not already installed):

```bash
# Ubuntu/Debian
echo "deb https://debian.cassandra.apache.org 41x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# macOS
brew install cassandra

# Docker (Alternative)
docker run --name cassandra -p 9042:9042 -d cassandra:latest

# Start Cassandra
sudo service cassandra start  # Linux
brew services start cassandra  # macOS
```

2. **Verify Installation**:

```bash
# Check Cassandra status
nodetool status

# Connect to CQL shell
cqlsh

# Check version
cqlsh --version
```

3. **Create Practice Keyspace**:

```cql
-- Connect to cqlsh
cqlsh

-- Create keyspace
CREATE KEYSPACE IF NOT EXISTS practice
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 1
};

-- Use the keyspace
USE practice;
```

---

## 📊 Cassandra Data Modeling Concepts

### Key Differences from SQL

- **Denormalization**: Data is denormalized for query performance
- **Partition Key**: Primary unit of data distribution
- **Clustering Key**: Determines sort order within partition
- **No JOINs**: Data must be pre-joined during modeling
- **Write-optimized**: Cassandra excels at writes
- **Query-first Design**: Model based on access patterns

---

## 🟢 SECTION 1: BASIC CQL OPERATIONS (Questions 1-5)

### Question 1: Simple Table Creation and Insert

**Difficulty:** Easy
**Topic:** Basic CRUD

```cql
-- Create table
CREATE TABLE IF NOT EXISTS users (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email TEXT,
    created_at TIMESTAMP
);

-- Insert sample data
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'john_doe', 'john@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'jane_smith', 'jane@example.com', toTimestamp(now()));

INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'bob_jones', 'bob@example.com', toTimestamp(now()));

-- Query: Retrieve all users
SELECT * FROM users;

-- Query: Count users
SELECT COUNT(*) FROM users;
```

### Question 2: Using WHERE with Partition Key

**Difficulty:** Easy
**Topic:** Filtering

```cql
-- Create table with partition key
CREATE TABLE IF NOT EXISTS products (
    product_id INT PRIMARY KEY,
    product_name TEXT,
    category TEXT,
    price DECIMAL,
    in_stock BOOLEAN
);

-- Insert sample data
INSERT INTO products (product_id, product_name, category, price, in_stock)
VALUES (1, 'Laptop', 'Electronics', 999.99, true);

INSERT INTO products (product_id, product_name, category, price, in_stock)
VALUES (2, 'Mouse', 'Electronics', 29.99, true);

INSERT INTO products (product_id, product_name, category, price, in_stock)
VALUES (3, 'Desk', 'Furniture', 299.99, false);

INSERT INTO products (product_id, product_name, category, price, in_stock)
VALUES (4, 'Chair', 'Furniture', 199.99, true);

-- Query: Get specific product by partition key
SELECT * FROM products WHERE product_id = 1;

-- Query: Get product name and price
SELECT product_name, price FROM products WHERE product_id = 2;
```

### Question 3: Composite Partition Key

**Difficulty:** Easy
**Topic:** Partition Keys

```cql
-- Create table with composite partition key
CREATE TABLE IF NOT EXISTS orders_by_user (
    user_id UUID,
    order_date DATE,
    order_id UUID,
    total_amount DECIMAL,
    status TEXT,
    PRIMARY KEY ((user_id, order_date), order_id)
);

-- Insert sample data
INSERT INTO orders_by_user (user_id, order_date, order_id, total_amount, status)
VALUES (uuid(), '2024-01-15', uuid(), 150.00, 'completed');

INSERT INTO orders_by_user (user_id, order_date, order_id, total_amount, status)
VALUES (uuid(), '2024-01-16', uuid(), 250.00, 'pending');

-- Query: Must include full partition key
SELECT * FROM orders_by_user
WHERE user_id = <some_uuid> AND order_date = '2024-01-15';
```

### Question 4: Clustering Columns for Sorting

**Difficulty:** Easy
**Topic:** Clustering Order

```cql
-- Create table with clustering column
CREATE TABLE IF NOT EXISTS sensor_readings (
    sensor_id TEXT,
    reading_time TIMESTAMP,
    temperature FLOAT,
    humidity FLOAT,
    PRIMARY KEY (sensor_id, reading_time)
) WITH CLUSTERING ORDER BY (reading_time DESC);

-- Insert sample data
INSERT INTO sensor_readings (sensor_id, reading_time, temperature, humidity)
VALUES ('sensor_001', toTimestamp(now()), 22.5, 45.0);

INSERT INTO sensor_readings (sensor_id, reading_time, temperature, humidity)
VALUES ('sensor_001', '2024-01-15 10:00:00+0000', 21.0, 50.0);

INSERT INTO sensor_readings (sensor_id, reading_time, temperature, humidity)
VALUES ('sensor_001', '2024-01-15 09:00:00+0000', 20.5, 48.0);

-- Query: Get readings for a sensor (automatically sorted DESC)
SELECT * FROM sensor_readings WHERE sensor_id = 'sensor_001';

-- Query: Get readings in time range
SELECT * FROM sensor_readings
WHERE sensor_id = 'sensor_001'
AND reading_time >= '2024-01-15 09:00:00+0000'
AND reading_time <= '2024-01-15 11:00:00+0000';
```

### Question 5: Using Collections (Lists, Sets, Maps)

**Difficulty:** Easy
**Topic:** Collection Types

```cql
-- Create table with collections
CREATE TABLE IF NOT EXISTS user_profile (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email_addresses SET<TEXT>,
    phone_numbers LIST<TEXT>,
    preferences MAP<TEXT, TEXT>
);

-- Insert sample data with collections
INSERT INTO user_profile (user_id, username, email_addresses, phone_numbers, preferences)
VALUES (
    uuid(),
    'alice',
    {'alice@work.com', 'alice@personal.com'},
    ['+1-555-0100', '+1-555-0101'],
    {'theme': 'dark', 'language': 'en'}
);

-- Query: Retrieve user profile
SELECT * FROM user_profile WHERE user_id = <some_uuid>;

-- Update: Add to set
UPDATE user_profile
SET email_addresses = email_addresses + {'alice@new.com'}
WHERE user_id = <some_uuid>;

-- Update: Add to map
UPDATE user_profile
SET preferences['notifications'] = 'enabled'
WHERE user_id = <some_uuid>;
```

---

## 🟡 SECTION 2: TIME-SERIES DATA (Questions 6-12)

### Question 6: Event Logging

**Difficulty:** Medium
**Topic:** Time-series Design

```cql
-- Create table for event logs
CREATE TABLE IF NOT EXISTS event_logs (
    application_id TEXT,
    event_date DATE,
    event_time TIMESTAMP,
    event_type TEXT,
    user_id UUID,
    event_data TEXT,
    PRIMARY KEY ((application_id, event_date), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);

-- Insert sample events
INSERT INTO event_logs (application_id, event_date, event_time, event_type, user_id, event_data)
VALUES ('app_001', '2024-01-15', '2024-01-15 10:30:00+0000', 'login', uuid(), 'User logged in');

INSERT INTO event_logs (application_id, event_date, event_time, event_type, user_id, event_data)
VALUES ('app_001', '2024-01-15', '2024-01-15 10:35:00+0000', 'click', uuid(), 'Button clicked');

INSERT INTO event_logs (application_id, event_date, event_time, event_type, user_id, event_data)
VALUES ('app_001', '2024-01-15', '2024-01-15 10:40:00+0000', 'logout', uuid(), 'User logged out');

-- Query: Get events for a specific day
SELECT * FROM event_logs
WHERE application_id = 'app_001'
AND event_date = '2024-01-15';

-- Query: Get events in time range
SELECT * FROM event_logs
WHERE application_id = 'app_001'
AND event_date = '2024-01-15'
AND event_time >= '2024-01-15 10:30:00+0000'
AND event_time <= '2024-01-15 10:40:00+0000';
```

### Question 7: Metrics Collection

**Difficulty:** Medium
**Topic:** Bucketing Pattern

```cql
-- Create table for metrics (bucketed by hour)
CREATE TABLE IF NOT EXISTS metrics_by_hour (
    metric_name TEXT,
    bucket_hour TIMESTAMP,
    metric_time TIMESTAMP,
    value DOUBLE,
    tags MAP<TEXT, TEXT>,
    PRIMARY KEY ((metric_name, bucket_hour), metric_time)
) WITH CLUSTERING ORDER BY (metric_time DESC);

-- Insert sample metrics
INSERT INTO metrics_by_hour (metric_name, bucket_hour, metric_time, value, tags)
VALUES ('cpu_usage', '2024-01-15 10:00:00+0000', '2024-01-15 10:15:30+0000', 75.5, {'host': 'server1', 'env': 'prod'});

INSERT INTO metrics_by_hour (metric_name, bucket_hour, metric_time, value, tags)
VALUES ('cpu_usage', '2024-01-15 10:00:00+0000', '2024-01-15 10:20:30+0000', 82.3, {'host': 'server1', 'env': 'prod'});

-- Query: Get metrics for specific hour
SELECT * FROM metrics_by_hour
WHERE metric_name = 'cpu_usage'
AND bucket_hour = '2024-01-15 10:00:00+0000';
```

### Question 8: User Activity Tracking

**Difficulty:** Medium
**Topic:** Wide Row Pattern

```cql
-- Create table for user activity (wide rows)
CREATE TABLE IF NOT EXISTS user_activity (
    user_id UUID,
    activity_date DATE,
    activity_time TIMESTAMP,
    action TEXT,
    details TEXT,
    PRIMARY KEY ((user_id, activity_date), activity_time)
) WITH CLUSTERING ORDER BY (activity_time DESC);

-- Insert sample activities
INSERT INTO user_activity (user_id, activity_date, activity_time, action, details)
VALUES (uuid(), '2024-01-15', '2024-01-15 08:00:00+0000', 'login', 'Web browser');

INSERT INTO user_activity (user_id, activity_date, activity_time, action, details)
VALUES (uuid(), '2024-01-15', '2024-01-15 08:05:00+0000', 'page_view', '/dashboard');

INSERT INTO user_activity (user_id, activity_date, activity_time, action, details)
VALUES (uuid(), '2024-01-15', '2024-01-15 08:10:00+0000', 'logout', 'Session ended');

-- Query: Get user's daily activity
SELECT * FROM user_activity
WHERE user_id = <some_uuid>
AND activity_date = '2024-01-15';

-- Query: Get latest N activities
SELECT * FROM user_activity
WHERE user_id = <some_uuid>
AND activity_date = '2024-01-15'
LIMIT 10;
```

### Question 9: Stock Price History

**Difficulty:** Medium
**Topic:** Financial Data

```cql
-- Create table for stock prices
CREATE TABLE IF NOT EXISTS stock_prices (
    symbol TEXT,
    price_date DATE,
    price_time TIMESTAMP,
    open_price DECIMAL,
    close_price DECIMAL,
    high_price DECIMAL,
    low_price DECIMAL,
    volume BIGINT,
    PRIMARY KEY ((symbol, price_date), price_time)
) WITH CLUSTERING ORDER BY (price_time DESC);

-- Insert sample data
INSERT INTO stock_prices (symbol, price_date, price_time, open_price, close_price, high_price, low_price, volume)
VALUES ('AAPL', '2024-01-15', '2024-01-15 09:30:00+0000', 150.00, 152.50, 153.00, 149.50, 1000000);

INSERT INTO stock_prices (symbol, price_date, price_time, open_price, close_price, high_price, low_price, volume)
VALUES ('AAPL', '2024-01-15', '2024-01-15 10:00:00+0000', 152.50, 151.75, 153.25, 151.00, 800000);

-- Query: Get daily prices for a stock
SELECT * FROM stock_prices
WHERE symbol = 'AAPL'
AND price_date = '2024-01-15';
```

### Question 10: IoT Device Data

**Difficulty:** Medium
**Topic:** Device Telemetry

```cql
-- Create table for IoT device data
CREATE TABLE IF NOT EXISTS device_telemetry (
    device_id TEXT,
    telemetry_date DATE,
    timestamp TIMESTAMP,
    battery_level INT,
    signal_strength INT,
    location TEXT,
    sensor_data MAP<TEXT, DOUBLE>,
    PRIMARY KEY ((device_id, telemetry_date), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Insert sample telemetry
INSERT INTO device_telemetry (device_id, telemetry_date, timestamp, battery_level, signal_strength, location, sensor_data)
VALUES ('device_001', '2024-01-15', '2024-01-15 12:00:00+0000', 85, -60, '40.7128,-74.0060', {'temp': 22.5, 'humidity': 45.0});

INSERT INTO device_telemetry (device_id, telemetry_date, timestamp, battery_level, signal_strength, location, sensor_data)
VALUES ('device_001', '2024-01-15', '2024-01-15 12:05:00+0000', 84, -58, '40.7128,-74.0060', {'temp': 22.7, 'humidity': 44.5});

-- Query: Get device data for today
SELECT * FROM device_telemetry
WHERE device_id = 'device_001'
AND telemetry_date = '2024-01-15';
```

### Question 11: Server Log Analytics

**Difficulty:** Medium
**Topic:** Log Aggregation

```cql
-- Create table for server logs
CREATE TABLE IF NOT EXISTS server_logs (
    server_name TEXT,
    log_date DATE,
    log_timestamp TIMESTAMP,
    log_level TEXT,
    message TEXT,
    error_code INT,
    PRIMARY KEY ((server_name, log_date), log_timestamp, log_level)
) WITH CLUSTERING ORDER BY (log_timestamp DESC, log_level ASC);

-- Insert sample logs
INSERT INTO server_logs (server_name, log_date, log_timestamp, log_level, message, error_code)
VALUES ('web-server-01', '2024-01-15', '2024-01-15 14:00:00+0000', 'INFO', 'Server started', null);

INSERT INTO server_logs (server_name, log_date, log_timestamp, log_level, message, error_code)
VALUES ('web-server-01', '2024-01-15', '2024-01-15 14:05:00+0000', 'ERROR', 'Connection timeout', 500);

INSERT INTO server_logs (server_name, log_date, log_timestamp, log_level, message, error_code)
VALUES ('web-server-01', '2024-01-15', '2024-01-15 14:10:00+0000', 'WARN', 'High memory usage', null);

-- Query: Get logs for server on specific date
SELECT * FROM server_logs
WHERE server_name = 'web-server-01'
AND log_date = '2024-01-15';

-- Note: To filter by log_level, you'd need a separate table or use ALLOW FILTERING (not recommended for production)
```

### Question 12: Chat Message History

**Difficulty:** Medium
**Topic:** Messaging System

```cql
-- Create table for chat messages
CREATE TABLE IF NOT EXISTS chat_messages (
    conversation_id UUID,
    message_date DATE,
    message_time TIMEUUID,
    sender_id UUID,
    message_text TEXT,
    message_type TEXT,
    attachments LIST<TEXT>,
    PRIMARY KEY ((conversation_id, message_date), message_time)
) WITH CLUSTERING ORDER BY (message_time DESC);

-- Insert sample messages
INSERT INTO chat_messages (conversation_id, message_date, message_time, sender_id, message_text, message_type, attachments)
VALUES (uuid(), '2024-01-15', now(), uuid(), 'Hello!', 'text', []);

INSERT INTO chat_messages (conversation_id, message_date, message_time, sender_id, message_text, message_type, attachments)
VALUES (uuid(), '2024-01-15', now(), uuid(), 'How are you?', 'text', []);

-- Query: Get messages for a conversation on a specific date
SELECT * FROM chat_messages
WHERE conversation_id = <some_uuid>
AND message_date = '2024-01-15';

-- Query: Get latest 50 messages
SELECT * FROM chat_messages
WHERE conversation_id = <some_uuid>
AND message_date = '2024-01-15'
LIMIT 50;
```

---

## 🔵 SECTION 3: MATERIALIZED VIEWS & INDEXES (Questions 13-19)

### Question 13: Secondary Index

**Difficulty:** Medium
**Topic:** Secondary Indexes

```cql
-- Create table
CREATE TABLE IF NOT EXISTS employees (
    employee_id UUID PRIMARY KEY,
    name TEXT,
    department TEXT,
    email TEXT,
    salary DECIMAL,
    hire_date DATE
);

-- Insert sample data
INSERT INTO employees (employee_id, name, department, email, salary, hire_date)
VALUES (uuid(), 'John Doe', 'Engineering', 'john@company.com', 85000, '2020-01-15');

INSERT INTO employees (employee_id, name, department, email, salary, hire_date)
VALUES (uuid(), 'Jane Smith', 'Engineering', 'jane@company.com', 90000, '2019-06-20');

INSERT INTO employees (employee_id, name, department, email, salary, hire_date)
VALUES (uuid(), 'Bob Johnson', 'Sales', 'bob@company.com', 75000, '2021-03-10');

-- Create secondary index on department
CREATE INDEX IF NOT EXISTS ON employees (department);

-- Query: Find employees by department (using secondary index)
SELECT * FROM employees WHERE department = 'Engineering';

-- WARNING: Secondary indexes have performance implications, use with caution
```

### Question 14: Materialized View - Different Query Pattern

**Difficulty:** Medium
**Topic:** Materialized Views

```cql
-- Base table (from Question 13)
-- Now create materialized view for email lookup
CREATE MATERIALIZED VIEW IF NOT EXISTS employees_by_email AS
    SELECT employee_id, name, department, email, salary, hire_date
    FROM employees
    WHERE email IS NOT NULL AND employee_id IS NOT NULL
    PRIMARY KEY (email, employee_id);

-- Query: Find employee by email
SELECT * FROM employees_by_email WHERE email = 'john@company.com';
```

### Question 15: Counter Columns

**Difficulty:** Medium
**Topic:** Counters

```cql
-- Create counter table for page views
CREATE TABLE IF NOT EXISTS page_views (
    page_url TEXT,
    view_date DATE,
    view_count COUNTER,
    PRIMARY KEY (page_url, view_date)
);

-- Increment counter
UPDATE page_views SET view_count = view_count + 1
WHERE page_url = '/home' AND view_date = '2024-01-15';

UPDATE page_views SET view_count = view_count + 1
WHERE page_url = '/home' AND view_date = '2024-01-15';

UPDATE page_views SET view_count = view_count + 5
WHERE page_url = '/home' AND view_date = '2024-01-15';

-- Query: Get view count
SELECT * FROM page_views WHERE page_url = '/home' AND view_date = '2024-01-15';

-- Create counter table for user activity metrics
CREATE TABLE IF NOT EXISTS user_stats (
    user_id UUID PRIMARY KEY,
    login_count COUNTER,
    post_count COUNTER,
    comment_count COUNTER
);

-- Update counters
UPDATE user_stats SET login_count = login_count + 1 WHERE user_id = <some_uuid>;
UPDATE user_stats SET post_count = post_count + 1 WHERE user_id = <some_uuid>;

-- Query: Get user stats
SELECT * FROM user_stats WHERE user_id = <some_uuid>;
```

### Question 16: SASI Index (SSTable Attached Secondary Index)

**Difficulty:** Medium
**Topic:** Advanced Indexing

```cql
-- Create table for products with searchable fields
CREATE TABLE IF NOT EXISTS product_catalog (
    product_id UUID PRIMARY KEY,
    product_name TEXT,
    description TEXT,
    category TEXT,
    price DECIMAL,
    created_at TIMESTAMP
);

-- Insert sample data
INSERT INTO product_catalog (product_id, product_name, description, category, price, created_at)
VALUES (uuid(), 'Laptop Pro', 'High-performance laptop', 'Electronics', 1299.99, toTimestamp(now()));

INSERT INTO product_catalog (product_id, product_name, description, category, price, created_at)
VALUES (uuid(), 'Wireless Mouse', 'Ergonomic wireless mouse', 'Electronics', 29.99, toTimestamp(now()));

-- Create SASI index for text search
CREATE CUSTOM INDEX IF NOT EXISTS ON product_catalog (product_name)
USING 'org.apache.cassandra.index.sasi.SASIIndex'
WITH OPTIONS = {
    'mode': 'CONTAINS',
    'analyzer_class': 'org.apache.cassandra.index.sasi.analyzer.StandardAnalyzer',
    'case_sensitive': 'false'
};

-- Query: Search products by name (partial match)
SELECT * FROM product_catalog WHERE product_name LIKE '%Laptop%';
```

### Question 17: Static Columns

**Difficulty:** Medium
**Topic:** Static Columns

```cql
-- Create table with static columns
CREATE TABLE IF NOT EXISTS team_members (
    team_id UUID,
    team_name TEXT STATIC,
    team_budget DECIMAL STATIC,
    member_id UUID,
    member_name TEXT,
    role TEXT,
    PRIMARY KEY (team_id, member_id)
);

-- Insert team info (static columns shared across partition)
INSERT INTO team_members (team_id, team_name, team_budget, member_id, member_name, role)
VALUES (uuid(), 'Engineering', 500000, uuid(), 'Alice', 'Lead Developer');

INSERT INTO team_members (team_id, team_name, team_budget, member_id, member_name, role)
VALUES (<same_team_id>, 'Engineering', 500000, uuid(), 'Bob', 'Developer');

-- Query: Get all team members (team_name and team_budget appear once)
SELECT * FROM team_members WHERE team_id = <some_uuid>;
```

### Question 18: User-Defined Types (UDT)

**Difficulty:** Medium
**Topic:** UDT

```cql
-- Create user-defined type for address
CREATE TYPE IF NOT EXISTS address (
    street TEXT,
    city TEXT,
    state TEXT,
    zip_code TEXT,
    country TEXT
);

-- Create user-defined type for phone
CREATE TYPE IF NOT EXISTS phone (
    phone_type TEXT,
    number TEXT
);

-- Create table using UDTs
CREATE TABLE IF NOT EXISTS customer_details (
    customer_id UUID PRIMARY KEY,
    name TEXT,
    home_address FROZEN<address>,
    work_address FROZEN<address>,
    phones LIST<FROZEN<phone>>
);

-- Insert sample data
INSERT INTO customer_details (customer_id, name, home_address, work_address, phones)
VALUES (
    uuid(),
    'John Smith',
    {street: '123 Main St', city: 'New York', state: 'NY', zip_code: '10001', country: 'USA'},
    {street: '456 Work Ave', city: 'New York', state: 'NY', zip_code: '10002', country: 'USA'},
    [{phone_type: 'mobile', number: '+1-555-0100'}, {phone_type: 'home', number: '+1-555-0200'}]
);

-- Query: Retrieve customer details
SELECT * FROM customer_details WHERE customer_id = <some_uuid>;
```

### Question 19: Lightweight Transactions (LWT)

**Difficulty:** Hard
**Topic:** Conditional Updates

```cql
-- Create table for inventory management
CREATE TABLE IF NOT EXISTS inventory (
    product_id UUID PRIMARY KEY,
    product_name TEXT,
    quantity INT,
    reserved INT,
    last_updated TIMESTAMP
);

-- Insert initial inventory
INSERT INTO inventory (product_id, product_name, quantity, reserved, last_updated)
VALUES (uuid(), 'Widget A', 100, 0, toTimestamp(now()));

-- Conditional update: Reserve items only if enough in stock
UPDATE inventory
SET reserved = reserved + 5, last_updated = toTimestamp(now())
WHERE product_id = <some_uuid>
IF quantity - reserved >= 5;

-- Conditional insert: Insert only if not exists
INSERT INTO inventory (product_id, product_name, quantity, reserved, last_updated)
VALUES (uuid(), 'Widget B', 50, 0, toTimestamp(now()))
IF NOT EXISTS;

-- Compare and swap pattern
UPDATE inventory
SET quantity = 95, last_updated = toTimestamp(now())
WHERE product_id = <some_uuid>
IF quantity = 100;
```

---

## 🟣 SECTION 4: BATCH OPERATIONS & TTL (Questions 20-26)

### Question 20: Batch Statements

**Difficulty:** Medium
**Topic:** Batch Operations

```cql
-- Create tables for e-commerce order
CREATE TABLE IF NOT EXISTS orders (
    order_id UUID PRIMARY KEY,
    user_id UUID,
    order_date TIMESTAMP,
    total_amount DECIMAL,
    status TEXT
);

CREATE TABLE IF NOT EXISTS order_items (
    order_id UUID,
    item_id UUID,
    product_name TEXT,
    quantity INT,
    price DECIMAL,
    PRIMARY KEY (order_id, item_id)
);

-- Execute batch: Insert order and items atomically
BEGIN BATCH
    INSERT INTO orders (order_id, user_id, order_date, total_amount, status)
    VALUES (uuid(), uuid(), toTimestamp(now()), 299.97, 'pending');

    INSERT INTO order_items (order_id, item_id, product_name, quantity, price)
    VALUES (<same_order_id>, uuid(), 'Product A', 2, 99.99);

    INSERT INTO order_items (order_id, item_id, product_name, quantity, price)
    VALUES (<same_order_id>, uuid(), 'Product B', 1, 99.99);
APPLY BATCH;

-- Query orders and items
SELECT * FROM orders WHERE order_id = <some_uuid>;
SELECT * FROM order_items WHERE order_id = <some_uuid>;
```

### Question 21: TTL (Time To Live)

**Difficulty:** Easy
**Topic:** Data Expiration

```cql
-- Create table for session data
CREATE TABLE IF NOT EXISTS user_sessions (
    session_id UUID PRIMARY KEY,
    user_id UUID,
    login_time TIMESTAMP,
    last_activity TIMESTAMP,
    session_data TEXT
);

-- Insert with TTL (expires after 1 hour)
INSERT INTO user_sessions (session_id, user_id, login_time, last_activity, session_data)
VALUES (uuid(), uuid(), toTimestamp(now()), toTimestamp(now()), 'session_data')
USING TTL 3600;

-- Update TTL
UPDATE user_sessions USING TTL 7200
SET last_activity = toTimestamp(now())
WHERE session_id = <some_uuid>;

-- Check remaining TTL
SELECT session_id, TTL(session_data) FROM user_sessions
WHERE session_id = <some_uuid>;
```

### Question 22: Temporary Data Storage

**Difficulty:** Easy
**Topic:** Cache Pattern

```cql
-- Create table for OTP (One-Time Password)
CREATE TABLE IF NOT EXISTS otp_codes (
    phone_number TEXT PRIMARY KEY,
    otp_code TEXT,
    created_at TIMESTAMP,
    attempts INT
);

-- Insert OTP with 5 minute expiration
INSERT INTO otp_codes (phone_number, otp_code, created_at, attempts)
VALUES ('+1-555-0100', '123456', toTimestamp(now()), 0)
USING TTL 300;

-- Verify and update attempts
UPDATE otp_codes
SET attempts = attempts + 1
WHERE phone_number = '+1-555-0100'
IF attempts < 3;

-- Query OTP
SELECT * FROM otp_codes WHERE phone_number = '+1-555-0100';
```

### Question 23: Denormalization Pattern

**Difficulty:** Medium
**Topic:** Data Modeling

```cql
-- Table 1: Posts by user
CREATE TABLE IF NOT EXISTS posts_by_user (
    user_id UUID,
    post_date DATE,
    post_id TIMEUUID,
    title TEXT,
    content TEXT,
    likes_count INT,
    PRIMARY KEY ((user_id, post_date), post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- Table 2: Posts by tag (denormalized)
CREATE TABLE IF NOT EXISTS posts_by_tag (
    tag TEXT,
    post_date DATE,
    post_id TIMEUUID,
    user_id UUID,
    title TEXT,
    content TEXT,
    likes_count INT,
    PRIMARY KEY ((tag, post_date), post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- Batch insert to both tables
BEGIN BATCH
    INSERT INTO posts_by_user (user_id, post_date, post_id, title, content, likes_count)
    VALUES (uuid(), '2024-01-15', now(), 'My First Post', 'Content here...', 0);

    INSERT INTO posts_by_tag (tag, post_date, post_id, user_id, title, content, likes_count)
    VALUES ('technology', '2024-01-15', <same_post_id>, <same_user_id>, 'My First Post', 'Content here...', 0);

    INSERT INTO posts_by_tag (tag, post_date, post_id, user_id, title, content, likes_count)
    VALUES ('cassandra', '2024-01-15', <same_post_id>, <same_user_id>, 'My First Post', 'Content here...', 0);
APPLY BATCH;

-- Query: Get user's posts
SELECT * FROM posts_by_user
WHERE user_id = <some_uuid>
AND post_date = '2024-01-15';

-- Query: Get posts by tag
SELECT * FROM posts_by_tag
WHERE tag = 'technology'
AND post_date = '2024-01-15';
```

### Question 24: Pagination

**Difficulty:** Medium
**Topic:** Query Pagination

```cql
-- Using previous sensor_readings table from Question 4

-- Query: First page (20 results)
SELECT * FROM sensor_readings
WHERE sensor_id = 'sensor_001'
LIMIT 20;

-- Query: Next page using paging state (application handles paging state)
-- In CQL shell, this is automatic with next page
-- In application code, use paging state from driver

-- Alternative: Manual pagination with clustering column
SELECT * FROM sensor_readings
WHERE sensor_id = 'sensor_001'
AND reading_time < '2024-01-15 10:00:00+0000'
LIMIT 20;
```

### Question 25: Bucketing for Large Partitions

**Difficulty:** Hard
**Topic:** Partition Management

```cql
-- Problem: Unbounded partition growth
-- Solution: Add bucket to partition key

-- Create table with bucketing
CREATE TABLE IF NOT EXISTS notifications (
    user_id UUID,
    bucket_id INT,  -- e.g., year_month as integer 202401
    notification_time TIMEUUID,
    notification_type TEXT,
    message TEXT,
    is_read BOOLEAN,
    PRIMARY KEY ((user_id, bucket_id), notification_time)
) WITH CLUSTERING ORDER BY (notification_time DESC);

-- Insert notifications with bucket
INSERT INTO notifications (user_id, bucket_id, notification_time, notification_type, message, is_read)
VALUES (uuid(), 202401, now(), 'message', 'You have a new message', false);

INSERT INTO notifications (user_id, bucket_id, notification_time, notification_type, message, is_read)
VALUES (<same_user_id>, 202401, now(), 'like', 'Someone liked your post', false);

-- Query: Get current month notifications
SELECT * FROM notifications
WHERE user_id = <some_uuid>
AND bucket_id = 202401
LIMIT 50;
```

### Question 26: Tombstones and Deletion

**Difficulty:** Medium
**Topic:** Data Deletion

```cql
-- Create table
CREATE TABLE IF NOT EXISTS temporary_cache (
    cache_key TEXT PRIMARY KEY,
    cache_value TEXT,
    created_at TIMESTAMP
);

-- Insert data
INSERT INTO temporary_cache (cache_key, cache_value, created_at)
VALUES ('key1', 'value1', toTimestamp(now()));

-- Delete single row
DELETE FROM temporary_cache WHERE cache_key = 'key1';

-- Delete specific column
DELETE cache_value FROM temporary_cache WHERE cache_key = 'key1';

-- TTL is preferred over deletion to avoid tombstones
INSERT INTO temporary_cache (cache_key, cache_value, created_at)
VALUES ('key2', 'value2', toTimestamp(now()))
USING TTL 3600;
```

---

## 🟠 SECTION 5: CONSISTENCY LEVELS (Questions 27-33)

### Question 27: Understanding Consistency Levels

**Difficulty:** Easy
**Topic:** Consistency Concepts

```cql
-- Set consistency level for session
CONSISTENCY QUORUM;

-- Insert with QUORUM consistency
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'test_user', 'test@example.com', toTimestamp(now()));

-- Read with QUORUM consistency
SELECT * FROM users WHERE user_id = <some_uuid>;

-- Available consistency levels:
CONSISTENCY ONE;        -- Read/Write to one replica
CONSISTENCY TWO;        -- Read/Write to two replicas
CONSISTENCY THREE;      -- Read/Write to three replicas
CONSISTENCY QUORUM;     -- Read/Write to majority (RF/2 + 1)
CONSISTENCY ALL;        -- Read/Write to all replicas
CONSISTENCY LOCAL_QUORUM; -- Quorum in local datacenter
CONSISTENCY EACH_QUORUM;  -- Quorum in each datacenter
CONSISTENCY LOCAL_ONE;    -- One replica in local datacenter
```

### Question 28: Write Performance vs Consistency

**Difficulty:** Medium
**Topic:** Write Optimization

```cql
-- Fast writes with eventual consistency
CONSISTENCY ONE;

CREATE TABLE IF NOT EXISTS metrics (
    metric_id UUID,
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY (metric_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- High-throughput writes
INSERT INTO metrics (metric_id, timestamp, value)
VALUES (uuid(), toTimestamp(now()), 42.5);

-- For critical data, use higher consistency
CONSISTENCY QUORUM;

CREATE TABLE IF NOT EXISTS financial_transactions (
    account_id UUID,
    transaction_id TIMEUUID,
    amount DECIMAL,
    transaction_type TEXT,
    balance_after DECIMAL,
    PRIMARY KEY (account_id, transaction_id)
) WITH CLUSTERING ORDER BY (transaction_id DESC);

INSERT INTO financial_transactions (account_id, transaction_id, amount, transaction_type, balance_after)
VALUES (uuid(), now(), 100.00, 'deposit', 1100.00);
```

### Question 29: Multi-Datacenter Setup

**Difficulty:** Hard
**Topic:** Multi-DC Configuration

```cql
-- Create keyspace with NetworkTopologyStrategy
CREATE KEYSPACE IF NOT EXISTS multi_dc_app
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'datacenter1': 3,
    'datacenter2': 3
};

USE multi_dc_app;

-- Create table
CREATE TABLE IF NOT EXISTS global_users (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email TEXT,
    preferred_dc TEXT
);

-- Use LOCAL_QUORUM for better latency in multi-DC
CONSISTENCY LOCAL_QUORUM;

INSERT INTO global_users (user_id, username, email, preferred_dc)
VALUES (uuid(), 'global_user', 'user@example.com', 'datacenter1');

-- Reads from local DC only
SELECT * FROM global_users WHERE user_id = <some_uuid>;
```

### Question 30: Tunable Consistency

**Difficulty:** Medium
**Topic:** Consistency Tuning

```cql
-- Shopping cart - eventual consistency acceptable
CONSISTENCY ONE;

CREATE TABLE IF NOT EXISTS shopping_carts (
    user_id UUID,
    session_id UUID,
    item_id UUID,
    quantity INT,
    added_at TIMESTAMP,
    PRIMARY KEY ((user_id, session_id), item_id)
);

INSERT INTO shopping_carts (user_id, session_id, item_id, quantity, added_at)
VALUES (uuid(), uuid(), uuid(), 2, toTimestamp(now()));

-- Order placement - strong consistency required
CONSISTENCY QUORUM;

CREATE TABLE IF NOT EXISTS completed_orders (
    order_id UUID PRIMARY KEY,
    user_id UUID,
    order_date TIMESTAMP,
    total_amount DECIMAL,
    payment_status TEXT
);

INSERT INTO completed_orders (order_id, user_id, order_date, total_amount, payment_status)
VALUES (uuid(), uuid(), toTimestamp(now()), 299.99, 'completed');
```

### Question 31: Read Repair

**Difficulty:** Medium
**Topic:** Consistency Maintenance

```cql
-- Configure read repair chance (at table level)
CREATE TABLE IF NOT EXISTS user_preferences (
    user_id UUID PRIMARY KEY,
    theme TEXT,
    language TEXT,
    notifications_enabled BOOLEAN
) WITH read_repair_chance = 0.1
AND dclocal_read_repair_chance = 0.1;

-- Normal queries automatically trigger read repair based on settings
INSERT INTO user_preferences (user_id, theme, language, notifications_enabled)
VALUES (uuid(), 'dark', 'en', true);

SELECT * FROM user_preferences WHERE user_id = <some_uuid>;
```

### Question 32: Hinted Handoff

**Difficulty:** Advanced
**Topic:** Write Availability

```cql
-- Hinted handoff is automatic in Cassandra
-- When a replica is down, coordinator stores hints

-- Create table with appropriate settings
CREATE TABLE IF NOT EXISTS critical_events (
    event_id TIMEUUID PRIMARY KEY,
    event_type TEXT,
    event_data TEXT,
    created_at TIMESTAMP
) WITH gc_grace_seconds = 864000;  -- 10 days

-- Writes succeed even if replica is down
INSERT INTO critical_events (event_id, event_type, event_data, created_at)
VALUES (now(), 'user_signup', '{"user": "john"}', toTimestamp(now()));

-- Hints are replayed when node comes back online
```

### Question 33: Consistency Level Tradeoffs

**Difficulty:** Conceptual
**Topic:** CAP Theorem

```cql
-- High Availability Pattern (AP in CAP)
CONSISTENCY ONE;
CREATE TABLE IF NOT EXISTS session_cache (
    session_id UUID PRIMARY KEY,
    session_data TEXT,
    last_access TIMESTAMP
) WITH default_time_to_live = 3600;

-- Strong Consistency Pattern (CP in CAP)
CONSISTENCY QUORUM;
CREATE TABLE IF NOT EXISTS account_balance (
    account_id UUID PRIMARY KEY,
    balance DECIMAL,
    last_transaction_id UUID,
    updated_at TIMESTAMP
);

-- Eventual Consistency with LWT for critical operations
CREATE TABLE IF NOT EXISTS distributed_lock (
    lock_name TEXT PRIMARY KEY,
    owner_id UUID,
    acquired_at TIMESTAMP,
    expires_at TIMESTAMP
);

-- Acquire lock with LWT
UPDATE distributed_lock
SET owner_id = <some_uuid>, acquired_at = toTimestamp(now()), expires_at = toTimestamp(now()) + 30000
WHERE lock_name = 'resource_lock'
IF owner_id = null OR expires_at < toTimestamp(now());
```

---

## 🔴 SECTION 6: PERFORMANCE OPTIMIZATION (Questions 34-40)

### Question 34: Partition Size Management

**Difficulty:** Hard
**Topic:** Partition Design

```cql
-- Bad: Unbounded partition
CREATE TABLE IF NOT EXISTS user_events_bad (
    user_id UUID,
    event_time TIMESTAMP,
    event_data TEXT,
    PRIMARY KEY (user_id, event_time)
);

-- Good: Bounded partition with bucketing
CREATE TABLE IF NOT EXISTS user_events_good (
    user_id UUID,
    year_month INT,  -- e.g., 202401
    event_time TIMESTAMP,
    event_data TEXT,
    PRIMARY KEY ((user_id, year_month), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);

-- Insert with bucket
INSERT INTO user_events_good (user_id, year_month, event_time, event_data)
VALUES (uuid(), 202401, toTimestamp(now()), 'event data');

-- Query specific time period
SELECT * FROM user_events_good
WHERE user_id = <some_uuid>
AND year_month = 202401;
```

### Question 35: Write-Through Cache Pattern

**Difficulty:** Medium
**Topic:** Caching Strategy

```cql
-- Cache layer with TTL
CREATE TABLE IF NOT EXISTS product_cache (
    product_id UUID PRIMARY KEY,
    product_data TEXT,  -- JSON or serialized data
    cached_at TIMESTAMP
) WITH default_time_to_live = 300;  -- 5 minutes

-- Master data (no TTL)
CREATE TABLE IF NOT EXISTS products_master (
    product_id UUID PRIMARY KEY,
    name TEXT,
    description TEXT,
    price DECIMAL,
    inventory INT,
    updated_at TIMESTAMP
);

-- Write to both tables
BEGIN BATCH
    INSERT INTO products_master (product_id, name, description, price, inventory, updated_at)
    VALUES (uuid(), 'Product A', 'Description', 99.99, 100, toTimestamp(now()));

    INSERT INTO product_cache (product_id, product_data, cached_at)
    VALUES (<same_id>, '{"name":"Product A","price":99.99}', toTimestamp(now()));
APPLY BATCH;
```

### Question 36: Optimal Data Types

**Difficulty:** Easy
**Topic:** Data Type Selection

```cql
-- Use appropriate data types for performance
CREATE TABLE IF NOT EXISTS optimized_table (
    -- Use INT instead of TEXT for numeric IDs when possible
    numeric_id INT,

    -- Use UUID for distributed unique IDs
    distributed_id UUID,

    -- Use TIMEUUID for time-based ordering with uniqueness
    event_id TIMEUUID,

    -- Use proper numeric types
    small_number TINYINT,      -- -128 to 127
    medium_number SMALLINT,    -- -32768 to 32767
    regular_number INT,        -- -2^31 to 2^31-1
    large_number BIGINT,       -- -2^63 to 2^63-1

    -- Use DECIMAL for exact precision (money)
    price DECIMAL,

    -- Use DOUBLE for approximate (scientific)
    measurement DOUBLE,

    -- Use DATE for dates without time
    birth_date DATE,

    -- Use TIMESTAMP for date+time
    created_at TIMESTAMP,

    -- Use BOOLEAN instead of INT for flags
    is_active BOOLEAN,

    PRIMARY KEY (numeric_id)
);
```

### Question 37: Compression Settings

**Difficulty:** Medium
**Topic:** Storage Optimization

```cql
-- Create table with optimal compression
CREATE TABLE IF NOT EXISTS logs_compressed (
    log_id TIMEUUID,
    message TEXT,
    log_level TEXT,
    timestamp TIMESTAMP,
    PRIMARY KEY (log_id)
) WITH compression = {
    'class': 'LZ4Compressor',
    'chunk_length_in_kb': 64
}
AND compaction = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_unit': 'DAYS',
    'compaction_window_size': 1
};

-- Alternative: Deflate for better compression ratio (slower)
CREATE TABLE IF NOT EXISTS archived_data (
    archive_id UUID PRIMARY KEY,
    data BLOB,
    archived_at TIMESTAMP
) WITH compression = {
    'class': 'DeflateCompressor'
};
```

### Question 38: Compaction Strategies

**Difficulty:** Hard
**Topic:** Compaction Tuning

```cql
-- Size-Tiered Compaction (STCS) - Default, good for write-heavy
CREATE TABLE IF NOT EXISTS write_heavy_data (
    id UUID PRIMARY KEY,
    data TEXT,
    created_at TIMESTAMP
) WITH compaction = {
    'class': 'SizeTieredCompactionStrategy',
    'min_threshold': 4,
    'max_threshold': 32
};

-- Leveled Compaction (LCS) - Good for read-heavy
CREATE TABLE IF NOT EXISTS read_heavy_data (
    id UUID PRIMARY KEY,
    data TEXT,
    updated_at TIMESTAMP
) WITH compaction = {
    'class': 'LeveledCompactionStrategy',
    'sstable_size_in_mb': 160
};

-- Time-Window Compaction (TWCS) - Best for time-series
CREATE TABLE IF NOT EXISTS time_series_data (
    sensor_id TEXT,
    bucket_time TIMESTAMP,
    reading_time TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((sensor_id, bucket_time), reading_time)
) WITH CLUSTERING ORDER BY (reading_time DESC)
AND compaction = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_unit': 'DAYS',
    'compaction_window_size': 1
}
AND default_time_to_live = 2592000;  -- 30 days
```

### Question 39: Bloom Filter Optimization

**Difficulty:** Advanced
**Topic:** Read Optimization

```cql
-- Adjust bloom filter false positive chance
CREATE TABLE IF NOT EXISTS frequently_read (
    key UUID PRIMARY KEY,
    value TEXT
) WITH bloom_filter_fp_chance = 0.01;  -- Default is 0.01

-- For tables with negative lookups, lower value helps
CREATE TABLE IF NOT EXISTS rare_lookups (
    key UUID PRIMARY KEY,
    value TEXT
) WITH bloom_filter_fp_chance = 0.001;  -- More memory, fewer false positives
```

### Question 40: Prepared Statements (Best Practice)

**Difficulty:** Easy
**Topic:** Query Performance

```cql
-- Prepared statements (shown conceptually - actual prep happens in driver)

-- Prepare once
PREPARE insert_user AS
    INSERT INTO users (user_id, username, email, created_at)
    VALUES (?, ?, ?, ?);

PREPARE get_user AS
    SELECT * FROM users WHERE user_id = ?;

PREPARE update_user AS
    UPDATE users SET email = ? WHERE user_id = ?;

-- Execute many times (in application code)
-- execute(insert_user, uuid(), 'john', 'john@example.com', now())
-- execute(get_user, some_uuid)
-- execute(update_user, 'newemail@example.com', some_uuid)

-- Benefits:
-- 1. Query parsing happens once
-- 2. Better performance
-- 3. Protection against injection
```

---

## 🟡 SECTION 7: ADVANCED PATTERNS (Questions 41-47)

### Question 41: Inverted Index Pattern

**Difficulty:** Hard
**Topic:** Search Implementation

```cql
-- Main table: Documents
CREATE TABLE IF NOT EXISTS documents (
    doc_id UUID PRIMARY KEY,
    title TEXT,
    content TEXT,
    author TEXT,
    created_at TIMESTAMP
);

-- Inverted index: Words to documents
CREATE TABLE IF NOT EXISTS word_to_documents (
    word TEXT,
    doc_id UUID,
    frequency INT,
    PRIMARY KEY (word, doc_id)
);

-- Insert document and update index (application logic)
BEGIN BATCH
    INSERT INTO documents (doc_id, title, content, author, created_at)
    VALUES (uuid(), 'Cassandra Guide', 'This is a guide about Cassandra...', 'John', toTimestamp(now()));

    INSERT INTO word_to_documents (word, doc_id, frequency)
    VALUES ('cassandra', <doc_id>, 3);

    INSERT INTO word_to_documents (word, doc_id, frequency)
    VALUES ('guide', <doc_id>, 2);
APPLY BATCH;

-- Search: Find documents containing word
SELECT doc_id, frequency FROM word_to_documents
WHERE word = 'cassandra';
```

### Question 42: Event Sourcing Pattern

**Difficulty:** Hard
**Topic:** Event-Driven Design

```cql
-- Event store
CREATE TABLE IF NOT EXISTS account_events (
    account_id UUID,
    event_id TIMEUUID,
    event_type TEXT,
    event_data TEXT,  -- JSON
    created_at TIMESTAMP,
    PRIMARY KEY (account_id, event_id)
) WITH CLUSTERING ORDER BY (event_id ASC);

-- Snapshot table for performance
CREATE TABLE IF NOT EXISTS account_snapshots (
    account_id UUID,
    snapshot_id TIMEUUID,
    balance DECIMAL,
    last_event_id TIMEUUID,
    snapshot_data TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (account_id, snapshot_id)
) WITH CLUSTERING ORDER BY (snapshot_id DESC);

-- Insert events
INSERT INTO account_events (account_id, event_id, event_type, event_data, created_at)
VALUES (uuid(), now(), 'account_created', '{"initial_balance": 1000}', toTimestamp(now()));

INSERT INTO account_events (account_id, event_id, event_type, event_data, created_at)
VALUES (<same_account>, now(), 'deposit', '{"amount": 500}', toTimestamp(now()));

-- Create periodic snapshots
INSERT INTO account_snapshots (account_id, snapshot_id, balance, last_event_id, snapshot_data, created_at)
VALUES (<account_id>, now(), 1500, <last_event_id>, '{"balance": 1500}', toTimestamp(now()));

-- Rebuild state: Get latest snapshot + subsequent events
SELECT * FROM account_snapshots
WHERE account_id = <some_uuid>
LIMIT 1;

SELECT * FROM account_events
WHERE account_id = <some_uuid>
AND event_id > <snapshot_last_event_id>;
```

### Question 43: Leaderboard Pattern

**Difficulty:** Hard
**Topic:** Ranking System

```cql
-- Global leaderboard (limited size)
CREATE TABLE IF NOT EXISTS global_leaderboard (
    game_id TEXT,
    score BIGINT,
    player_id UUID,
    player_name TEXT,
    achieved_at TIMESTAMP,
    PRIMARY KEY (game_id, score, player_id)
) WITH CLUSTERING ORDER BY (score DESC, player_id ASC);

-- Player scores (for personal history)
CREATE TABLE IF NOT EXISTS player_scores (
    player_id UUID,
    game_id TEXT,
    score_id TIMEUUID,
    score BIGINT,
    achieved_at TIMESTAMP,
    PRIMARY KEY ((player_id, game_id), score_id)
) WITH CLUSTERING ORDER BY (score_id DESC);

-- Insert score to both tables
BEGIN BATCH
    INSERT INTO global_leaderboard (game_id, score, player_id, player_name, achieved_at)
    VALUES ('game_001', 10000, uuid(), 'Alice', toTimestamp(now()));

    INSERT INTO player_scores (player_id, game_id, score_id, score, achieved_at)
    VALUES (<player_id>, 'game_001', now(), 10000, toTimestamp(now()));
APPLY BATCH;

-- Get top 100 players
SELECT * FROM global_leaderboard
WHERE game_id = 'game_001'
LIMIT 100;

-- Get player's personal best
SELECT * FROM player_scores
WHERE player_id = <some_uuid>
AND game_id = 'game_001'
LIMIT 10;
```

### Question 44: Queue Pattern

**Difficulty:** Hard
**Topic:** Message Queue

```cql
-- Queue table (with timestamp for processing)
CREATE TABLE IF NOT EXISTS message_queue (
    queue_name TEXT,
    priority INT,
    message_id TIMEUUID,
    message_data TEXT,
    status TEXT,  -- 'pending', 'processing', 'completed', 'failed'
    created_at TIMESTAMP,
    processed_at TIMESTAMP,
    PRIMARY KEY ((queue_name, priority), message_id)
) WITH CLUSTERING ORDER BY (message_id ASC);

-- Insert message
INSERT INTO message_queue (queue_name, priority, message_id, message_data, status, created_at)
VALUES ('email_queue', 1, now(), '{"to": "user@example.com", "subject": "Hello"}', 'pending', toTimestamp(now()));

-- Consume messages (with LWT for single processing)
SELECT * FROM message_queue
WHERE queue_name = 'email_queue'
AND priority = 1
LIMIT 10;

-- Mark as processing
UPDATE message_queue
SET status = 'processing', processed_at = toTimestamp(now())
WHERE queue_name = 'email_queue'
AND priority = 1
AND message_id = <some_timeuuid>
IF status = 'pending';

-- Mark as completed
UPDATE message_queue
SET status = 'completed'
WHERE queue_name = 'email_queue'
AND priority = 1
AND message_id = <some_timeuuid>;
```

### Question 45: Fan-out Pattern

**Difficulty:** Medium
**Topic:** Social Media Feed

```cql
-- User's timeline (fan-out on write)
CREATE TABLE IF NOT EXISTS user_timeline (
    user_id UUID,
    post_time TIMEUUID,
    post_id UUID,
    author_id UUID,
    author_name TEXT,
    content TEXT,
    PRIMARY KEY (user_id, post_time)
) WITH CLUSTERING ORDER BY (post_time DESC);

-- When user creates post, write to all followers' timelines
-- (This would be done by application logic)
BEGIN BATCH
    -- Write to author's timeline
    INSERT INTO user_timeline (user_id, post_time, post_id, author_id, author_name, content)
    VALUES (<author_id>, now(), uuid(), <author_id>, 'John', 'Hello World!');

    -- Fan-out to follower 1
    INSERT INTO user_timeline (user_id, post_time, post_id, author_id, author_name, content)
    VALUES (<follower1_id>, <same_post_time>, <same_post_id>, <author_id>, 'John', 'Hello World!');

    -- Fan-out to follower 2
    INSERT INTO user_timeline (user_id, post_time, post_id, author_id, author_name, content)
    VALUES (<follower2_id>, <same_post_time>, <same_post_id>, <author_id>, 'John', 'Hello World!');
APPLY BATCH;

-- Read timeline (fast!)
SELECT * FROM user_timeline
WHERE user_id = <some_uuid>
LIMIT 50;
```

### Question 46: Saga Pattern

**Difficulty:** Advanced
**Topic:** Distributed Transactions

```cql
-- Saga state table
CREATE TABLE IF NOT EXISTS saga_state (
    saga_id UUID PRIMARY KEY,
    saga_type TEXT,
    status TEXT,  -- 'pending', 'compensating', 'completed', 'failed'
    current_step INT,
    step_data MAP<INT, TEXT>,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Start saga
INSERT INTO saga_state (saga_id, saga_type, status, current_step, step_data, created_at, updated_at)
VALUES (uuid(), 'order_placement', 'pending', 1, {1: 'reserve_inventory'}, toTimestamp(now()), toTimestamp(now()));

-- Update saga progress
UPDATE saga_state
SET current_step = 2,
    step_data = step_data + {2: 'charge_payment'},
    updated_at = toTimestamp(now())
WHERE saga_id = <some_uuid>;

-- Compensating transaction
UPDATE saga_state
SET status = 'compensating',
    step_data = step_data + {99: 'refund_payment'},
    updated_at = toTimestamp(now())
WHERE saga_id = <some_uuid>
IF status = 'pending';
```

### Question 47: Geospatial Data (Simple)

**Difficulty:** Medium
**Topic:** Location Data

```cql
-- Simple geohash-based location storage
CREATE TABLE IF NOT EXISTS locations (
    geohash TEXT,  -- Partition by geohash prefix (e.g., first 4 chars)
    place_id UUID,
    place_name TEXT,
    latitude DOUBLE,
    longitude DOUBLE,
    category TEXT,
    PRIMARY KEY (geohash, place_id)
);

-- Insert locations with geohash
INSERT INTO locations (geohash, place_id, place_name, latitude, longitude, category)
VALUES ('9q8y', uuid(), 'Restaurant A', 37.7749, -122.4194, 'restaurant');

INSERT INTO locations (geohash, place_id, place_name, latitude, longitude, category)
VALUES ('9q8y', uuid(), 'Cafe B', 37.7750, -122.4190, 'cafe');

-- Query by geohash prefix (returns all places in area)
SELECT * FROM locations WHERE geohash = '9q8y';

-- For production: Use DataStax Enterprise with geo search
-- or integrate with external geospatial service
```

---

## 🔵 SECTION 8: ADMINISTRATION & MONITORING (Questions 48-50)

### Question 48: User Management

**Difficulty:** Easy
**Topic:** Security

```cql
-- Create users
CREATE ROLE IF NOT EXISTS developer WITH PASSWORD = 'dev_password' AND LOGIN = true;
CREATE ROLE IF NOT EXISTS read_only WITH PASSWORD = 'readonly_password' AND LOGIN = true;

-- Create role for grouping permissions
CREATE ROLE IF NOT EXISTS app_users;

-- Grant permissions
GRANT SELECT ON KEYSPACE practice TO read_only;
GRANT ALL PERMISSIONS ON KEYSPACE practice TO developer;
GRANT SELECT, MODIFY ON TABLE practice.users TO app_users;

-- Assign role to user
GRANT app_users TO developer;

-- Check permissions
LIST ALL PERMISSIONS OF developer;

-- Revoke permissions
REVOKE SELECT ON KEYSPACE practice FROM read_only;

-- Drop user
DROP ROLE IF EXISTS read_only;
```

### Question 49: Tracing Queries

**Difficulty:** Easy
**Topic:** Performance Analysis

```cql
-- Enable tracing
TRACING ON;

-- Run query to trace
SELECT * FROM users WHERE user_id = <some_uuid>;

-- Tracing output shows:
-- - Which nodes were contacted
-- - Latency for each step
-- - Read repair activity
-- - Coordination overhead

-- Disable tracing
TRACING OFF;

-- Probabilistic tracing (application level)
-- Set in cassandra.yaml:
-- request_scheduler: org.apache.cassandra.scheduler.NoScheduler
-- request_scheduler_options:
--   throttle_limit: 80
```

### Question 50: Monitoring and Metrics

**Difficulty:** Medium
**Topic:** Operations

```cql
-- Check table statistics
SELECT * FROM system.size_estimates WHERE keyspace_name = 'practice';

-- Check table details
SELECT * FROM system_schema.tables WHERE keyspace_name = 'practice';

-- Check compaction stats
-- (Run via nodetool)
-- nodetool compactionstats

-- Check table metrics
-- nodetool tablestats practice

-- Check partition size (via nodetool)
-- nodetool cfhistograms practice users

-- Common monitoring queries
SELECT * FROM system.local;
SELECT * FROM system.peers;

-- Token ranges
SELECT * FROM system.size_estimates;
```

---

## 📝 Quick Reference - Cassandra Best Practices

### Data Modeling

```cql
-- 1. Model based on queries, not entities
-- 2. One table per query pattern
-- 3. Denormalize data
-- 4. Avoid ALLOW FILTERING
-- 5. Keep partition size < 100MB
-- 6. Use bucketing for time-series
-- 7. Use appropriate data types
```

### Query Patterns

```cql
-- Good: Query by partition key
SELECT * FROM users WHERE user_id = <uuid>;

-- Good: Query with clustering column
SELECT * FROM events WHERE user_id = <uuid> AND event_time > '2024-01-01';

-- Bad: Full table scan
SELECT * FROM users;  -- Requires ALLOW FILTERING

-- Bad: Query without partition key
SELECT * FROM users WHERE email = 'john@example.com';  -- Requires index or ALLOW FILTERING
```

### Collections Best Practices

```cql
-- Good: Small, bounded collections
preferences MAP<TEXT, TEXT>  -- Few key-value pairs

-- Bad: Unbounded collections
tags SET<TEXT>  -- Could grow indefinitely

-- Alternative: Separate table for unbounded data
CREATE TABLE user_tags (
    user_id UUID,
    tag TEXT,
    PRIMARY KEY (user_id, tag)
);
```

### Write Patterns

```cql
-- Batch for single partition (good)
BEGIN BATCH
    INSERT INTO orders (...) VALUES (...);
    INSERT INTO order_items (...) VALUES (...);  -- Same partition
APPLY BATCH;

-- Batch across partitions (use carefully)
BEGIN BATCH
    INSERT INTO table1 (...) VALUES (...);
    INSERT INTO table2 (...) VALUES (...);  -- Different partition
APPLY BATCH;
-- Note: Cross-partition batches are atomic but slower
```

---

## 🎯 Practice Tips

1. **Understand the Data Model**: Cassandra is NOT relational - think in terms of partitions
2. **Query-First Design**: Design tables based on query patterns, not entities
3. **Avoid Anti-Patterns**: No JOINs, no unbounded partitions, no full scans
4. **Use cqlsh DESCRIBE**: `DESCRIBE TABLE` and `DESCRIBE KEYSPACE` are your friends
5. **Monitor Partition Sizes**: Use `nodetool tablestats` regularly
6. **Test at Scale**: Cassandra behavior changes with data volume

---

## 🚦 Learning Path

- **Week 1**: Questions 1-12 (Basics & Time-Series)
- **Week 2**: Questions 13-26 (Advanced Features & Batching)
- **Week 3**: Questions 27-40 (Consistency & Performance)
- **Week 4**: Questions 41-50 (Advanced Patterns & Operations)

---

## 📚 Additional Resources

```bash
# Check Cassandra logs
tail -f /var/log/cassandra/system.log

# Monitoring with nodetool
nodetool status
nodetool tpstats
nodetool cfstats
nodetool describecluster

# Backup
nodetool snapshot practice

# Repair
nodetool repair practice
```

---

_Remember: Cassandra excels at high-throughput writes and time-series data. Design your data model around your query patterns, and you'll achieve excellent performance!_

**Happy Cassandra Querying! 🚀**
