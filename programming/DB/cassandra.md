# Apache Cassandra Interview Questions & Answers

## 1. What is Cassandra and why use it?

**Q: Explain Cassandra**

A: Apache Cassandra is a distributed NoSQL database designed for handling large amounts of data across many commodity servers with no single point of failure.

**Key features**:

- Distributed and decentralized (no master-slave)
- High availability and fault tolerance
- Linear scalability
- Tunable consistency
- Wide column store
- Peer-to-peer architecture
- Multi-datacenter replication
- Handles writes extremely well
- Schema-flexible
- CQL (Cassandra Query Language)
- No single point of failure
- Eventual consistency

**Why use Cassandra**:

- Massive scalability (100s of nodes)
- High write throughput
- No downtime (always available)
- Geographic distribution
- Time-series data
- IoT and sensor data
- Real-time analytics
- Heavy write workloads
- Linear performance scaling

**vs PostgreSQL**: Cassandra scales horizontally, better for writes, eventual consistency

**vs MongoDB**: Better for write-heavy workloads, more predictable scaling

**Use cases**: Time-series data, IoT, messaging, activity feeds, recommendation engines

---

## 2. What is the data model?

**Q: Cassandra data model basics**

A: Cassandra uses a wide column store model with keyspaces, tables, and columns.

**Hierarchy**:

```
Cluster
  └─ Keyspace (like database)
      └─ Table (column family)
          └─ Row (partition)
              └─ Columns
```

**Data types**:

```sql
-- Numeric
INT, BIGINT, SMALLINT, TINYINT
FLOAT, DOUBLE
DECIMAL
VARINT           -- Arbitrary precision

-- String
TEXT, VARCHAR    -- UTF-8 strings
ASCII            -- ASCII strings

-- Binary
BLOB             -- Binary data

-- Boolean
BOOLEAN          -- true/false

-- Date/Time
TIMESTAMP        -- Date and time
DATE             -- Date only
TIME             -- Time only
DURATION         -- Time duration

-- UUID
UUID             -- Random UUID
TIMEUUID         -- Time-based UUID

-- Collections
LIST<type>       -- Ordered list
SET<type>        -- Unique values
MAP<key, value>  -- Key-value pairs

-- Other
INET             -- IP address
COUNTER          -- Distributed counter
TUPLE<type,...>  -- Fixed-length tuple
```

**Example**:

```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email TEXT,
    age INT,
    created_at TIMESTAMP,
    tags SET<TEXT>,
    metadata MAP<TEXT, TEXT>
);
```

---

## 3. What are keyspaces?

**Q: Cassandra keyspaces**

A: Keyspace is the outermost container (like a database in RDBMS).

**Create keyspace**:

```sql
CREATE KEYSPACE my_keyspace
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- Multi-datacenter
CREATE KEYSPACE my_keyspace
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3,
    'dc2': 2
};
```

**Use keyspace**:

```sql
USE my_keyspace;
```

**Alter keyspace**:

```sql
ALTER KEYSPACE my_keyspace
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 5
};
```

**Drop keyspace**:

```sql
DROP KEYSPACE my_keyspace;
```

**Replication strategies**:

- **SimpleStrategy**: Single datacenter
- **NetworkTopologyStrategy**: Multiple datacenters (production)

---

## 4. What are primary keys and partition keys?

**Q: Keys in Cassandra**

A: Understanding keys is crucial for Cassandra performance.

**Primary key components**:

```sql
-- Simple primary key (single partition key)
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);
-- Partition key: user_id

-- Composite primary key (partition key + clustering columns)
CREATE TABLE posts (
    user_id UUID,
    post_id TIMEUUID,
    title TEXT,
    content TEXT,
    PRIMARY KEY (user_id, post_id)
);
-- Partition key: user_id
-- Clustering column: post_id

-- Compound partition key
CREATE TABLE user_events (
    user_id UUID,
    event_type TEXT,
    event_time TIMESTAMP,
    data TEXT,
    PRIMARY KEY ((user_id, event_type), event_time)
);
-- Partition key: (user_id, event_type)
-- Clustering column: event_time
```

**Partition key** — Determines which node stores the data:

```sql
-- All rows with same partition key stored together
CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender TEXT,
    content TEXT,
    PRIMARY KEY (conversation_id, message_id)
);
-- All messages for a conversation on same node
```

**Clustering columns** — Determine sort order within partition:

```sql
CREATE TABLE sensor_data (
    sensor_id UUID,
    timestamp TIMESTAMP,
    temperature FLOAT,
    humidity FLOAT,
    PRIMARY KEY (sensor_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
-- Latest readings first
```

**Key design rules**:

- Partition key must be in WHERE clause
- Query patterns drive data model
- Avoid large partitions (> 100MB)
- Distribute data evenly across nodes

---

## 5. What are collections?

**Q: Collection types**

A: Cassandra supports lists, sets, and maps.

**LIST** — Ordered, can have duplicates:

```sql
CREATE TABLE playlists (
    id UUID PRIMARY KEY,
    name TEXT,
    songs LIST<TEXT>
);

INSERT INTO playlists (id, name, songs)
VALUES (uuid(), 'My Playlist', ['song1', 'song2', 'song3']);

-- Append
UPDATE playlists SET songs = songs + ['song4'] WHERE id = ?;

-- Prepend
UPDATE playlists SET songs = ['song0'] + songs WHERE id = ?;

-- Remove
UPDATE playlists SET songs = songs - ['song2'] WHERE id = ?;

-- Access by index
SELECT songs[0] FROM playlists WHERE id = ?;
```

**SET** — Unordered, unique values:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    tags SET<TEXT>
);

INSERT INTO users (id, name, tags)
VALUES (uuid(), 'John', {'developer', 'golang', 'cassandra'});

-- Add
UPDATE users SET tags = tags + {'python'} WHERE id = ?;

-- Remove
UPDATE users SET tags = tags - {'golang'} WHERE id = ?;
```

**MAP** — Key-value pairs:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    settings MAP<TEXT, TEXT>
);

INSERT INTO users (id, name, settings)
VALUES (uuid(), 'John', {'theme': 'dark', 'lang': 'en'});

-- Add/update
UPDATE users SET settings['timezone'] = 'UTC' WHERE id = ?;

-- Remove
DELETE settings['theme'] FROM users WHERE id = ?;
```

**Best practices**:

- Keep collections small (< 100 items)
- Don't use for unbounded data
- Consider separate tables for large collections

---

## 6. What are basic queries?

**Q: CQL queries**

A:

```sql
-- INSERT
INSERT INTO users (user_id, name, email, age)
VALUES (uuid(), 'John', 'john@example.com', 30);

-- With TTL (time to live)
INSERT INTO sessions (session_id, user_id, data)
VALUES (uuid(), uuid(), 'session_data')
USING TTL 3600;  -- Expires in 1 hour

-- SELECT
SELECT * FROM users;
SELECT name, email FROM users WHERE user_id = ?;

-- WHERE clause (must include partition key)
SELECT * FROM posts WHERE user_id = ? AND post_id = ?;
SELECT * FROM posts WHERE user_id = ? AND post_id > ?;

-- LIMIT
SELECT * FROM users LIMIT 10;

-- ORDER BY (only on clustering columns)
SELECT * FROM posts WHERE user_id = ? ORDER BY post_id DESC;

-- ALLOW FILTERING (avoid in production - slow)
SELECT * FROM users WHERE age > 18 ALLOW FILTERING;

-- UPDATE
UPDATE users SET age = 31 WHERE user_id = ?;

-- Update with conditions
UPDATE users SET email = 'new@example.com'
WHERE user_id = ?
IF email = 'old@example.com';

-- DELETE
DELETE FROM users WHERE user_id = ?;
DELETE email FROM users WHERE user_id = ?;

-- With TTL check
DELETE FROM sessions WHERE session_id = ?;

-- BATCH (atomic operations on same partition)
BEGIN BATCH
    INSERT INTO users (user_id, name) VALUES (?, 'John');
    INSERT INTO posts (user_id, post_id, title) VALUES (?, ?, 'Post');
APPLY BATCH;

-- Lightweight transactions (expensive)
INSERT INTO users (user_id, name, email)
VALUES (?, 'John', 'john@example.com')
IF NOT EXISTS;
```

**Important**:

- Always include partition key in WHERE
- Cannot do arbitrary WHERE clauses like SQL
- Design tables around query patterns

---

## 7. What are indexes?

**Q: Secondary indexes**

A: Allow queries on non-primary-key columns.

**Create secondary index**:

```sql
CREATE INDEX ON users (email);
CREATE INDEX idx_name ON users (name);

-- Now can query by email
SELECT * FROM users WHERE email = 'john@example.com';
```

**Drop index**:

```sql
DROP INDEX idx_name;
```

**Materialized views** (better than secondary indexes):

```sql
-- Base table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email TEXT,
    name TEXT,
    country TEXT
);

-- Materialized view for querying by email
CREATE MATERIALIZED VIEW users_by_email AS
    SELECT user_id, email, name, country
    FROM users
    WHERE email IS NOT NULL AND user_id IS NOT NULL
    PRIMARY KEY (email, user_id);

-- Query the view
SELECT * FROM users_by_email WHERE email = 'john@example.com';
```

**SASI indexes** (advanced):

```sql
CREATE CUSTOM INDEX ON users (name)
USING 'org.apache.cassandra.index.sasi.SASIIndex'
WITH OPTIONS = {
    'mode': 'CONTAINS',
    'analyzer_class': 'org.apache.cassandra.index.sasi.analyzer.StandardAnalyzer',
    'case_sensitive': 'false'
};

-- Supports LIKE queries
SELECT * FROM users WHERE name LIKE '%john%';
```

**Best practices**:

- Avoid secondary indexes on high-cardinality columns
- Prefer materialized views
- Design data model to avoid needing indexes

---

## 8. What are counters?

**Q: Distributed counters**

A: Special column type for counting.

```sql
CREATE TABLE page_views (
    page_id UUID PRIMARY KEY,
    view_count COUNTER
);

-- Increment
UPDATE page_views SET view_count = view_count + 1 WHERE page_id = ?;

-- Decrement
UPDATE page_views SET view_count = view_count - 1 WHERE page_id = ?;

-- Read
SELECT view_count FROM page_views WHERE page_id = ?;
```

**Counter table example**:

```sql
CREATE TABLE user_stats (
    user_id UUID,
    stat_name TEXT,
    stat_value COUNTER,
    PRIMARY KEY (user_id, stat_name)
);

UPDATE user_stats SET stat_value = stat_value + 1
WHERE user_id = ? AND stat_name = 'posts';

UPDATE user_stats SET stat_value = stat_value + 1
WHERE user_id = ? AND stat_name = 'followers';
```

**Limitations**:

- Counter columns cannot mix with regular columns (except primary key)
- Cannot INSERT/overwrite, only UPDATE
- Not idempotent (retries can cause issues)

---

## 9. What is time-to-live (TTL)?

**Q: Automatic expiration**

A: TTL automatically deletes data after specified time.

```sql
-- Insert with TTL
INSERT INTO sessions (session_id, user_id, data)
VALUES (uuid(), ?, 'data')
USING TTL 3600;  -- 1 hour

-- Update with TTL
UPDATE sessions USING TTL 86400  -- 24 hours
SET data = 'new_data'
WHERE session_id = ?;

-- Check remaining TTL
SELECT TTL(data) FROM sessions WHERE session_id = ?;

-- Table-level default TTL
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY,
    user_id UUID,
    data TEXT
) WITH default_time_to_live = 3600;
```

**Use cases**:

- Session data
- Temporary caches
- Time-series data with retention
- Notification systems

---

## 10. What are user-defined types (UDT)?

**Q: Custom data types**

A:

```sql
-- Create UDT
CREATE TYPE address (
    street TEXT,
    city TEXT,
    zip_code TEXT,
    country TEXT
);

-- Use in table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    home_address FROZEN<address>,
    work_address FROZEN<address>
);

-- Insert
INSERT INTO users (user_id, name, home_address)
VALUES (uuid(), 'John', {
    street: '123 Main St',
    city: 'NYC',
    zip_code: '10001',
    country: 'USA'
});

-- Query
SELECT name, home_address.city FROM users WHERE user_id = ?;

-- Nested UDT
CREATE TYPE phone (
    type TEXT,
    number TEXT
);

CREATE TYPE contact_info (
    email TEXT,
    phones LIST<FROZEN<phone>>
);

CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    contact FROZEN<contact_info>
);
```

---

## 11. What is consistency level?

**Q: Tunable consistency**

A: Control how many replicas must respond.

**Consistency levels**:

```sql
-- ONE: One replica responds (fast, less consistent)
-- TWO: Two replicas respond
-- THREE: Three replicas respond
-- QUORUM: Majority of replicas (most common)
-- LOCAL_QUORUM: Majority in local datacenter
-- EACH_QUORUM: Majority in each datacenter
-- ALL: All replicas (slow, most consistent)
-- ANY: At least one node (write only)
```

**Set consistency in CQL**:

```sql
CONSISTENCY QUORUM;
SELECT * FROM users WHERE user_id = ?;

CONSISTENCY LOCAL_QUORUM;
INSERT INTO users (user_id, name) VALUES (?, 'John');
```

**Application level** (Java example):

```java
// Write with QUORUM
Statement stmt = SimpleStatement.newInstance(
    "INSERT INTO users (user_id, name) VALUES (?, ?)",
    userId, "John"
).setConsistencyLevel(ConsistencyLevel.QUORUM);

// Read with ONE (fast)
Statement stmt = SimpleStatement.newInstance(
    "SELECT * FROM users WHERE user_id = ?",
    userId
).setConsistencyLevel(ConsistencyLevel.ONE);
```

**Formula**: Read CL + Write CL > Replication Factor = Strong consistency

```
Example: RF=3, Write=QUORUM(2), Read=QUORUM(2) → 2+2 > 3 → Strong consistency
Example: RF=3, Write=ONE(1), Read=ONE(1) → 1+1 < 3 → Eventual consistency
```

---

## 12. What is data modeling?

**Q: Design principles**

A: Model around query patterns, not data relationships.

**Rule 1: Know your queries first**

```sql
-- Query: Get all posts by user, newest first
CREATE TABLE posts_by_user (
    user_id UUID,
    post_id TIMEUUID,
    title TEXT,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- Query: Get posts by category, newest first
CREATE TABLE posts_by_category (
    category TEXT,
    post_id TIMEUUID,
    user_id UUID,
    title TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (category, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

**Rule 2: One query per table (denormalization)**

```sql
-- Duplicate data for different access patterns
-- Users table
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email TEXT
);

-- Query users by username
CREATE TABLE users_by_username (
    username TEXT PRIMARY KEY,
    user_id UUID,
    email TEXT
);

-- Query users by email
CREATE TABLE users_by_email (
    email TEXT PRIMARY KEY,
    user_id UUID,
    username TEXT
);
```

**Rule 3: Avoid large partitions**

```sql
-- BAD: Unbounded partition
CREATE TABLE user_events (
    user_id UUID,
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY (user_id, event_id)
);
-- Problem: Years of events in one partition

-- GOOD: Bucketed partitions
CREATE TABLE user_events (
    user_id UUID,
    year_month TEXT,  -- e.g., '2024-01'
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY ((user_id, year_month), event_id)
) WITH CLUSTERING ORDER BY (event_id DESC);
-- Each month is separate partition
```

**Rule 4: Write optimization**

```sql
-- Time-series with TTL
CREATE TABLE sensor_readings (
    sensor_id UUID,
    date TEXT,
    timestamp TIMESTAMP,
    temperature FLOAT,
    PRIMARY KEY ((sensor_id, date), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC)
AND default_time_to_live = 2592000;  -- 30 days
```

---

## 13. What are common patterns?

**Q: Design patterns**

A:

**Time-series data**:

```sql
CREATE TABLE metrics (
    metric_name TEXT,
    bucket TEXT,  -- e.g., '2024-01-15-12' (hour bucket)
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((metric_name, bucket), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC)
AND COMPACTION = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_size': 1,
    'compaction_window_unit': 'DAYS'
};
```

**Activity feed**:

```sql
CREATE TABLE user_feed (
    user_id UUID,
    activity_id TIMEUUID,
    actor_id UUID,
    activity_type TEXT,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, activity_id)
) WITH CLUSTERING ORDER BY (activity_id DESC);

-- Latest 50 activities
SELECT * FROM user_feed WHERE user_id = ? LIMIT 50;
```

**Messaging/Chat**:

```sql
CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender_id UUID,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- Get conversation history
SELECT * FROM messages WHERE conversation_id = ? LIMIT 100;

-- Pagination
SELECT * FROM messages
WHERE conversation_id = ? AND message_id < ?
LIMIT 50;
```

**Leaderboard**:

```sql
CREATE TABLE leaderboard (
    game_id UUID,
    score INT,
    user_id UUID,
    username TEXT,
    timestamp TIMESTAMP,
    PRIMARY KEY (game_id, score, user_id)
) WITH CLUSTERING ORDER BY (score DESC);

-- Top 10
SELECT * FROM leaderboard WHERE game_id = ? LIMIT 10;
```

---

## 14. What are batch operations?

**Q: Atomic writes**

A:

```sql
-- Logged batch (atomic, same partition recommended)
BEGIN BATCH
    INSERT INTO users (user_id, name, email)
    VALUES (?, 'John', 'john@example.com');

    INSERT INTO users_by_email (email, user_id, name)
    VALUES ('john@example.com', ?, 'John');

    UPDATE user_stats SET stat_value = stat_value + 1
    WHERE user_id = ? AND stat_name = 'registrations';
APPLY BATCH;

-- Unlogged batch (better performance, no atomicity)
BEGIN UNLOGGED BATCH
    INSERT INTO posts_by_user (user_id, post_id, title)
    VALUES (?, ?, 'Title 1');

    INSERT INTO posts_by_user (user_id, post_id, title)
    VALUES (?, ?, 'Title 2');
APPLY BATCH;

-- Counter batch
BEGIN COUNTER BATCH
    UPDATE page_views SET view_count = view_count + 1 WHERE page_id = ?;
    UPDATE user_stats SET stat_value = stat_value + 1 WHERE user_id = ?;
APPLY BATCH;
```

**Best practices**:

- Use for denormalization (updating multiple tables)
- Keep batches in same partition when possible
- Avoid large batches (< 100 statements)
- Use UNLOGGED for better performance when atomicity not needed

---

## 15. What is compaction?

**Q: Cassandra maintenance**

A: Compaction merges SSTables and removes tombstones.

**Compaction strategies**:

```sql
-- SizeTieredCompactionStrategy (default, general purpose)
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT
) WITH COMPACTION = {
    'class': 'SizeTieredCompactionStrategy'
};

-- LeveledCompactionStrategy (read-heavy, better space)
CREATE TABLE posts (
    post_id UUID PRIMARY KEY,
    content TEXT
) WITH COMPACTION = {
    'class': 'LeveledCompactionStrategy'
};

-- TimeWindowCompactionStrategy (time-series)
CREATE TABLE events (
    event_id TIMEUUID PRIMARY KEY,
    data TEXT
) WITH COMPACTION = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_size': 1,
    'compaction_window_unit': 'DAYS'
};
```

**Manual compaction** (nodetool):

```bash
# Full compaction
nodetool compact keyspace_name table_name

# Check compaction status
nodetool compactionstats
```

---

## 16. What are best practices?

**Q: Production tips**

A:

**1. Data modeling**:
- Design for queries, not relationships
- Denormalize data
- Use time buckets for time-series
- Avoid ALLOW FILTERING

**2. Partition size**:
- Keep under 100MB per partition
- Avoid unbounded partitions
- Use bucketing for hot partitions

**3. Consistency**:
- Use LOCAL_QUORUM for multi-DC
- RF=3 in production
- Read CL + Write CL > RF for strong consistency

**4. Performance**:
- Use prepared statements
- Batch denormalization updates
- Use TTL for temporary data
- Monitor partition sizes

**5. Operations**:
- Regular repairs (nodetool repair)
- Monitor compaction
- Use NetworkTopologyStrategy
- Set up monitoring (nodetool, metrics)

```sql
-- Good table design
CREATE TABLE user_sessions (
    user_id UUID,
    session_date TEXT,  -- Bucket: '2024-01-15'
    session_id TIMEUUID,
    data TEXT,
    PRIMARY KEY ((user_id, session_date), session_id)
) WITH CLUSTERING ORDER BY (session_id DESC)
AND default_time_to_live = 2592000  -- 30 days
AND COMPACTION = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_size': 1,
    'compaction_window_unit': 'DAYS'
};
```

---

## 17. What are common operations?

**Q: Maintenance commands**

A:

```bash
# Check cluster status
nodetool status

# Check node health
nodetool info

# Flush memtables to disk
nodetool flush

# Run compaction
nodetool compact keyspace_name table_name

# Repair data
nodetool repair keyspace_name

# Check ring
nodetool ring

# Decommission node
nodetool decommission

# Stream data to new node
nodetool bootstrap

# Clear snapshot
nodetool clearsnapshot

# Table statistics
nodetool tablestats keyspace_name.table_name

# Thread pool stats
nodetool tpstats

# Check gossip
nodetool gossipinfo
```

**CQL commands**:

```sql
-- Describe cluster
DESCRIBE CLUSTER;

-- Describe keyspace
DESCRIBE KEYSPACE my_keyspace;

-- Describe table
DESCRIBE TABLE users;

-- Tracing
TRACING ON;
SELECT * FROM users WHERE user_id = ?;
TRACING OFF;

-- Show version
SHOW VERSION;
```

---

## 18. What are client best practices?

**Q: Application integration**

A:

**Python example**:

```python
from cassandra.cluster import Cluster
from cassandra.query import SimpleStatement, BatchStatement
from cassandra import ConsistencyLevel
import uuid

# Connect
cluster = Cluster(['127.0.0.1'])
session = cluster.connect('my_keyspace')

# Prepared statements (faster, safer)
insert_user = session.prepare(
    "INSERT INTO users (user_id, name, email) VALUES (?, ?, ?)"
)

user_id = uuid.uuid4()
session.execute(insert_user, (user_id, 'John', 'john@example.com'))

# With consistency level
stmt = insert_user.bind((user_id, 'John', 'john@example.com'))
stmt.consistency_level = ConsistencyLevel.QUORUM
session.execute(stmt)

# Batch
batch = BatchStatement()
batch.add(insert_user, (uuid.uuid4(), 'User1', 'user1@example.com'))
batch.add(insert_user, (uuid.uuid4(), 'User2', 'user2@example.com'))
session.execute(batch)

# Async execution
future = session.execute_async(insert_user, (user_id, 'John', 'john@example.com'))
row = future.result()

# Close
cluster.shutdown()
```

**Best practices**:
- Use prepared statements
- Reuse session objects
- Handle retries
- Set timeouts
- Use async for concurrency

---

## 19. What are monitoring tips?

**Q: Monitor Cassandra**

A:

**Key metrics**:

```bash
# Read/write latency
nodetool proxyhistograms

# Table statistics
nodetool tablestats

# Thread pool statistics
nodetool tpstats

# Compaction statistics
nodetool compactionstats

# GC statistics
nodetool gcstats

# Check for dropped messages
nodetool netstats
```

**JMX metrics**:
- Read latency (p95, p99)
- Write latency (p95, p99)
- Disk usage
- Compaction pending
- Memtable flush queue
- Dropped messages
- GC pause time

**Log files**:
- system.log
- debug.log
- gc.log

---

## 20. What are common mistakes?

**Q: Avoid these**

A:

**1. Wrong data model**:

```sql
-- BAD: Modeling like RDBMS
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);

CREATE TABLE posts (
    post_id UUID PRIMARY KEY,
    user_id UUID,  -- Foreign key mindset
    title TEXT
);
-- Cannot JOIN in Cassandra!

-- GOOD: Denormalized
CREATE TABLE posts_by_user (
    user_id UUID,
    post_id TIMEUUID,
    username TEXT,  -- Denormalized
    title TEXT,
    PRIMARY KEY (user_id, post_id)
);
```

**2. Using ALLOW FILTERING**:

```sql
-- BAD: Full table scan
SELECT * FROM users WHERE age > 18 ALLOW FILTERING;

-- GOOD: Materialized view or separate table
CREATE MATERIALIZED VIEW users_by_age AS
    SELECT * FROM users
    WHERE age IS NOT NULL AND user_id IS NOT NULL
    PRIMARY KEY (age, user_id);
```

**3. Large partitions**:

```sql
-- BAD: Unbounded
CREATE TABLE events (
    user_id UUID,
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY (user_id, event_id)
);

-- GOOD: Bucketed
CREATE TABLE events (
    user_id UUID,
    bucket TEXT,  -- '2024-01'
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY ((user_id, bucket), event_id)
);
```

**4. Not using prepared statements**

**5. Over-using secondary indexes**

**6. Not setting TTL for temporary data**

**7. Using ALL consistency level**

---

## 21. What is a real-world example?

**Q: Complete schema**

A:

```sql
-- Keyspace
CREATE KEYSPACE social_network
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3,
    'dc2': 2
};

USE social_network;

-- Users
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    username TEXT,
    email TEXT,
    full_name TEXT,
    created_at TIMESTAMP
);

CREATE MATERIALIZED VIEW users_by_username AS
    SELECT user_id, username, email, full_name
    FROM users
    WHERE username IS NOT NULL AND user_id IS NOT NULL
    PRIMARY KEY (username, user_id);

-- Posts
CREATE TABLE posts (
    post_id TIMEUUID PRIMARY KEY,
    user_id UUID,
    username TEXT,
    content TEXT,
    created_at TIMESTAMP,
    likes_count COUNTER
);

-- User timeline (home feed)
CREATE TABLE user_timeline (
    user_id UUID,
    post_id TIMEUUID,
    author_id UUID,
    author_username TEXT,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- User posts
CREATE TABLE user_posts (
    user_id UUID,
    post_id TIMEUUID,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);

-- Followers
CREATE TABLE followers (
    user_id UUID,
    follower_id UUID,
    follower_username TEXT,
    followed_at TIMESTAMP,
    PRIMARY KEY (user_id, follower_id)
);

CREATE TABLE following (
    user_id UUID,
    following_id UUID,
    following_username TEXT,
    followed_at TIMESTAMP,
    PRIMARY KEY (user_id, following_id)
);

-- Direct messages
CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender_id UUID,
    sender_username TEXT,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- User stats
CREATE TABLE user_stats (
    user_id UUID,
    stat_name TEXT,
    stat_value COUNTER,
    PRIMARY KEY (user_id, stat_name)
);

-- Sample queries:
-- Get user by username
SELECT * FROM users_by_username WHERE username = 'john';

-- Get user timeline (home feed)
SELECT * FROM user_timeline WHERE user_id = ? LIMIT 50;

-- Get user's posts
SELECT * FROM user_posts WHERE user_id = ? LIMIT 20;

-- Post to timeline (batch denormalization)
BEGIN BATCH
    INSERT INTO user_posts (user_id, post_id, content, created_at)
    VALUES (?, ?, ?, ?);

    INSERT INTO user_timeline (user_id, post_id, author_id, author_username, content, created_at)
    VALUES (?, ?, ?, ?, ?, ?);
    -- Repeat for each follower's timeline
APPLY BATCH;
```

---

## Cassandra Interview Tips

1. **Understand CAP theorem** — Cassandra favors AP (Availability + Partition tolerance)
2. **Know data modeling** — Query-first design, denormalization
3. **Partition keys** — Critical for performance
4. **Consistency levels** — Tunable consistency
5. **No JOINs** — Design accordingly
6. **Write path** — Understand how writes work (fast!)
7. **Read path** — Bloom filters, indexes, caching
8. **Compaction** — SSTable management
9. **Replication** — RF, consistency formula
10. **Time-series data** — Common use case
11. **Anti-patterns** — Large partitions, ALLOW FILTERING
12. **Operations** — nodetool commands
13. **Monitoring** — Key metrics to watch
14. **Client drivers** — Prepared statements
15. **Real projects** — Production experience
