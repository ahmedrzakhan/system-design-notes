# 10 Must-Know Database Types for System Design Interviews

Choosing the right database is critical in system design interviews, as it impacts performance, scalability, and complexity. This guide covers the 10 essential database types, their use cases, design considerations, and popular examples, preparing you to confidently discuss them in interviews. Each section includes a diagram to visualize the database structure or use case.

## 1. Relational Database

**What it is**: Stores data in structured tables with rows and columns, using SQL for querying. Relationships are defined via foreign keys, ensuring data integrity.

**When to use it**:
- **Structured, relational data**: Ideal for entities with clear relationships (e.g., e-commerce: Users → Orders → Products).
- **Strong consistency**: ACID compliance ensures reliable transactions (e.g., banking systems for money transfers).
- **Complex queries**: SQL supports joins, filtering, and aggregations for analytics.

**Design Considerations**:
- **Indexing**: Use indexes on frequently queried columns (e.g., `user_id`). Avoid over-indexing in write-heavy systems.
- **Normalization vs. Denormalization**: Normalize for consistency; denormalize for read performance.
- **Joins**: Efficient for analytics but avoid excessive joins in large-scale systems.
- **Scaling**: Vertical scaling is simpler; horizontal scaling uses read replicas, sharding, or caching (e.g., Redis).

**Example Databases**:
- PostgreSQL
- MySQL
- Oracle DB

**Diagram**:
```mermaid
erDiagram
    USER ||--o{ ORDER : places
    ORDER ||--o{ PRODUCT : contains
    PRODUCT ||--o{ REVIEW : has
    PRODUCT ||--o{ CATEGORY : belongs_to
```

## 2. In-Memory Database

**What it is**: Stores data in RAM for ultra-fast reads and writes, often used as a caching layer.

**When to use it**:
- **Low-latency needs**: Real-time applications like gaming leaderboards.
- **Temporary data**: Data that can be regenerated (e.g., cached search results).
- **Offload primary database**: Cache frequent queries (e.g., user profiles in social media).

**Design Considerations**:
- **Volatility**: Data is lost on crashes unless persistence (e.g., Redis RDB/AOF) is enabled.
- **Eviction Policies**: Use LRU, LFU, or TTL for memory management.
- **Replication**: Asynchronous replication risks data loss; pair with a durable database for critical data.

**Example Databases**:
- Redis
- Memcached
- Apache Ignite

**Diagram**:
```mermaid
graph TD
    A[Client] --> B[In-Memory DB]
    B -->|Cache Hit| C[Response]
    B -->|Cache Miss| D[Primary DB]
    D --> B
```

## 3. Key-Value Database

**What it is**: Stores data as key-value pairs, like a distributed HashMap, for simple, fast lookups.

**When to use it**:
- **Fast lookups**: Ideal for `userId → sessionInfo` or `shortURL → fullURL`.
- **No complex queries**: Best when joins or filtering aren’t needed.
- **High throughput**: Handles millions of reads/writes per second.

**Design Considerations**:
- **Lookup-only**: No filtering or sorting; retrieve by key only.
- **Schema-less**: Application handles data serialization.
- **Scaling**: Use high-cardinality keys for even distribution in sharding.

**Example Databases**:
- Redis
- Amazon DynamoDB
- Aerospike

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Key| B[Key-Value DB]
    B -->|Value| A
```

## 4. Document Database

**What it is**: Stores semi-structured JSON/BSON documents, allowing flexible schemas.

**When to use it**:
- **Varying data structures**: Different records have unique fields (e.g., CMS with diverse content types).
- **Nested data**: Store hierarchical data (e.g., user profiles with addresses).
- **Fast iteration**: Schema flexibility supports rapid development.

**Design Considerations**:
- **Indexing**: Index frequently queried fields; avoid deep nested indexing.
- **Document Size**: Split large documents (e.g., MongoDB’s 16MB limit).
- **Denormalization**: Embed related data to avoid joins, but manage duplication.
- **Sharding**: Use high-cardinality shard keys.

**Example Databases**:
- MongoDB
- Couchbase
- Firebase Firestore

**Diagram**:
```mermaid
classDiagram
    class Document_DB {
        user_id: string
        name: string
        addresses: array
        preferences: object
    }
```

## 5. Graph Database

**What it is**: Stores data as nodes (entities) and edges (relationships), optimized for traversals.

**When to use it**:
- **Relationship-focused**: Social networks (e.g., friends of friends).
- **Recommendations**: Find patterns like “customers who bought X also bought Y.”
- **Complex traversals**: Efficiently query multi-level relationships.

**Design Considerations**:
- **Traversal Efficiency**: Outperforms relational joins for deep relationships.
- **Indexing**: Index node properties for fast query starts.
- **Scalability**: Distributed graph DBs (e.g., Neo4j Enterprise) for large datasets.
- **Query Language**: Use Cypher (Neo4j) or Gremlin for traversals.

**Example Databases**:
- Neo4j
- Amazon Neptune
- ArangoDB

**Diagram**:
```mermaid
graph TD
    A[User: Alice] -->|FOLLOWS| B[User: Bob]
    B -->|FOLLOWS| C[User: Charlie]
    A -->|LIKES| D[Post]
```

## 6. Wide-Column Database

**What it is**: Stores data in flexible column families, optimized for large-scale, write-heavy workloads.

**When to use it**:
- **High write throughput**: Time-series logging (e.g., service logs).
- **Massive datasets**: Petabytes of sparse, growing data.
- **Flexible schemas**: Rows with varying columns (e.g., user message inboxes).

**Design Considerations**:
- **Schema Design**: Optimize row keys, partition keys, and clustering columns.
- **Denormalization**: Duplicate data for fast reads; avoid joins.
- **Tunable Consistency**: Balance strong vs. eventual consistency.
- **Sharding**: Avoid hotspots with high-cardinality partition keys.

**Example Databases**:
- Apache Cassandra
- ScyllaDB
- Google Bigtable

**Diagram**:
```mermaid
classDiagram
    class Wide_Column_DB {
        row_key: user_id
        column_family: messages
        msg_1: string
        msg_2: string
    }
```

## 7. Time-Series Database

**What it is**: Optimized for timestamped data, supporting high write volumes and time-based queries.

**When to use it**:
- **Chronological data**: Monitoring metrics (e.g., CPU usage).
- **Aggregations**: Rollups or downsampling for trends (e.g., daily averages).
- **Time-bound queries**: Query specific time windows (e.g., last 24 hours).

**Design Considerations**:
- **Write-Optimized**: Append-only writes with time-based compression.
- **Retention Policies**: Auto-expire old data.
- **Sharding**: Shard by time ranges or series IDs.

**Example Databases**:
- InfluxDB
- TimescaleDB
- Prometheus

**Diagram**:
```mermaid
graph TD
    A[Sensor] -->|Timestamped Data| B[Time-Series DB]
    B -->|Query: Last 24h| C[Client]
```

## 8. Text-Search Database

**What it is**: Indexes and searches large text datasets with full-text search, ranking, and fuzzy matching.

**When to use it**:
- **Text search**: Product search in e-commerce (e.g., “running shoes”).
- **Fuzzy search**: Handle typos or synonyms (e.g., “recieve” → “receive”).
- **Filtered search**: Combine keywords with filters (e.g., price, location).

**Design Considerations**:
- **Inverted Indexing**: Maps terms to documents for fast lookups.
- **Tokenization & Stemming**: Normalize words (e.g., “running” → “run”).
- **Relevance Scoring**: Use TF-IDF or BM25 for ranking.
- **Scalability**: Distributed indexing and sharding for large datasets.

**Example Databases**:
- Elasticsearch
- Apache Solr
- Typesense

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Search Query| B[Text-Search DB]
    B -->|Inverted Index| C[Results]
```

## 9. Spatial Database

**What it is**: Handles geospatial data (locations, shapes) with operations like proximity and intersection.

**When to use it**:
- **Location-based features**: Ride-hailing apps (e.g., drivers within 3 km).
- **Geometric queries**: Geofencing or polygon intersections (e.g., delivery zones).

**Design Considerations**:
- **Spatial Indexing**: Use R-Trees, QuadTrees, or Geohashing.
- **Accuracy vs. Performance**: Use approximations (e.g., bounding boxes) for speed.
- **Scalability**: Cache frequent queries; limit search scope.

**Example Databases**:
- PostGIS (PostgreSQL)
- MongoDB
- Elasticsearch

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Geo Query| B[Spatial DB]
    B -->|R-Tree Index| C[Results]
```

## 10. Blob Store

**What it is**: Stores large, unstructured binary files (e.g., images, videos) with metadata.

**When to use it**:
- **Large media files**: Video platforms storing media assets.
- **Unstructured data**: Files unfit for relational/NoSQL databases.
- **Scalable storage**: Cost-effective for cold data.

**Design Considerations**:
- **Metadata Management**: Store metadata in a separate database.
- **Access Control**: Use signed URLs or IAM policies.
- **Chunking**: Split large files for reliable uploads.
- **CDN Integration**: Cache content for low latency.

**Example Databases**:
- Amazon S3
- Google Cloud Storage
- MinIO

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Upload| B[Blob Store]
    B -->|Metadata| C[Relational/NoSQL DB]
    B -->|Serve| D[CDN]
```

## Interview Tips
- **Understand Trade-offs**: Explain why you chose a database (e.g., consistency vs. scalability).
- **Use Examples**: Reference real-world systems (e.g., Cassandra for Instagram’s feeds).
- **Discuss Scaling**: Mention sharding, replication, or caching strategies.
- **Know Popular Tools**: Be ready to name specific databases and their strengths.