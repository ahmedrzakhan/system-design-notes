# Elasticsearch Interview Questions & Answers

## 1. What is Elasticsearch and why use it?

**Q: Explain Elasticsearch**

A: Elasticsearch is a distributed, open-source search and analytics engine built on Apache Lucene. Designed for horizontal scalability, reliability, and real-time search.

**Key features**:

- Full-text search
- Distributed and highly available
- Real-time indexing and search
- RESTful API
- Document-oriented (JSON)
- Schema-free (dynamic mapping)
- Aggregations (analytics)
- Near real-time (NRT) search
- Horizontal scalability
- Multi-tenancy
- Geospatial search
- Machine learning capabilities
- Auto-complete and suggestions

**Why use Elasticsearch**:

- Fast full-text search
- Log and event analytics
- Real-time application monitoring
- Security analytics (SIEM)
- Business analytics
- Complex aggregations
- Geographic queries
- Recommendation engines
- Search-as-you-type

**vs PostgreSQL**: Better for full-text search, analytics, unstructured data

**vs Solr**: Similar but easier setup, better for real-time use cases

**Use cases**: Log analytics (ELK stack), e-commerce search, security analytics, metrics

---

## 2. What are basic concepts?

**Q: Core terminology**

A:

**Hierarchy**:

```
Cluster
  └─ Node (server instance)
      └─ Index (like database)
          └─ Document (like row, JSON)
              └─ Fields (like columns)
```

**Document** — Basic unit of data (JSON):

```json
{
  "user_id": "123",
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Index** — Collection of documents:
- Similar to database in RDBMS
- Logical namespace
- Can have multiple types (deprecated in 7.x)

**Shard** — Horizontal partition of an index:
- Primary shards (original data)
- Replica shards (copies for HA)

**Node** — Single Elasticsearch server:
- Master node (cluster management)
- Data node (stores data)
- Ingest node (preprocessing)
- Coordinating node (routes requests)

**Cluster** — Collection of nodes:
- Cluster name identifies the cluster
- Nodes join by cluster name

---

## 3. What are data types?

**Q: Field types**

A:

**String**:

```json
{
  "properties": {
    "title": { "type": "text" },      // Full-text, analyzed
    "status": { "type": "keyword" },   // Exact match, not analyzed
    "email": { "type": "keyword" }
  }
}
```

**Numeric**:

```json
{
  "properties": {
    "age": { "type": "integer" },
    "price": { "type": "float" },
    "amount": { "type": "double" },
    "quantity": { "type": "long" },
    "score": { "type": "short" },
    "count": { "type": "byte" }
  }
}
```

**Date**:

```json
{
  "properties": {
    "created_at": { "type": "date" },
    "timestamp": {
      "type": "date",
      "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
    }
  }
}
```

**Boolean**:

```json
{
  "properties": {
    "is_active": { "type": "boolean" }
  }
}
```

**Complex types**:

```json
{
  "properties": {
    "location": { "type": "geo_point" },    // Geographic point
    "area": { "type": "geo_shape" },        // Geographic shape
    "ip_addr": { "type": "ip" },            // IP address
    "data": { "type": "binary" },           // Binary data
    "completion": { "type": "completion" }, // Auto-complete
    "tags": { "type": "keyword" }           // Array (implicit)
  }
}
```

**Object and Nested**:

```json
{
  "properties": {
    "user": {                     // Object (flattened)
      "properties": {
        "name": { "type": "text" },
        "age": { "type": "integer" }
      }
    },
    "comments": {                 // Nested (maintains relationship)
      "type": "nested",
      "properties": {
        "author": { "type": "keyword" },
        "text": { "type": "text" }
      }
    }
  }
}
```

---

## 4. What are index operations?

**Q: CRUD for indexes**

A:

**Create index**:

```bash
# Simple index
PUT /users

# With mappings
PUT /users
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "email": { "type": "keyword" },
      "age": { "type": "integer" },
      "created_at": { "type": "date" }
    }
  }
}
```

**Get index info**:

```bash
GET /users
GET /users/_mapping
GET /users/_settings
GET /_cat/indices?v
```

**Update index settings**:

```bash
PUT /users/_settings
{
  "number_of_replicas": 3
}
```

**Delete index**:

```bash
DELETE /users
```

**Index templates** (for multiple similar indexes):

```bash
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "message": { "type": "text" },
        "level": { "type": "keyword" }
      }
    }
  }
}
```

**Aliases** (virtual index names):

```bash
POST /_aliases
{
  "actions": [
    { "add": { "index": "users_v1", "alias": "users" } }
  ]
}

# Zero-downtime reindex
POST /_aliases
{
  "actions": [
    { "remove": { "index": "users_v1", "alias": "users" } },
    { "add": { "index": "users_v2", "alias": "users" } }
  ]
}
```

---

## 5. What are document operations?

**Q: CRUD for documents**

A:

**Index (create/update) document**:

```bash
# Auto-generated ID
POST /users/_doc
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

# Specific ID
PUT /users/_doc/1
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

# Create only (fail if exists)
PUT /users/_create/1
{
  "name": "John Doe"
}
```

**Get document**:

```bash
GET /users/_doc/1

# Get specific fields
GET /users/_doc/1?_source=name,email

# Multiple documents
GET /users/_mget
{
  "ids": ["1", "2", "3"]
}
```

**Update document**:

```bash
# Partial update
POST /users/_update/1
{
  "doc": {
    "age": 31
  }
}

# Script update
POST /users/_update/1
{
  "script": {
    "source": "ctx._source.age += params.increment",
    "params": {
      "increment": 1
    }
  }
}

# Upsert (update or insert)
POST /users/_update/1
{
  "doc": {
    "age": 31
  },
  "doc_as_upsert": true
}
```

**Delete document**:

```bash
DELETE /users/_doc/1
```

**Bulk operations** (for efficiency):

```bash
POST /_bulk
{ "index": { "_index": "users", "_id": "1" } }
{ "name": "John", "age": 30 }
{ "index": { "_index": "users", "_id": "2" } }
{ "name": "Jane", "age": 28 }
{ "update": { "_index": "users", "_id": "1" } }
{ "doc": { "age": 31 } }
{ "delete": { "_index": "users", "_id": "3" } }
```

---

## 6. What are basic search queries?

**Q: Search API**

A:

**Match all**:

```bash
GET /users/_search
{
  "query": {
    "match_all": {}
  }
}
```

**Match query** (full-text search):

```bash
GET /users/_search
{
  "query": {
    "match": {
      "name": "john doe"
    }
  }
}
```

**Term query** (exact match):

```bash
GET /users/_search
{
  "query": {
    "term": {
      "email.keyword": "john@example.com"
    }
  }
}
```

**Range query**:

```bash
GET /users/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 18,
        "lte": 65
      }
    }
  }
}
```

**Bool query** (combine multiple queries):

```bash
GET /users/_search
{
  "query": {
    "bool": {
      "must": [                          // AND
        { "match": { "name": "john" } }
      ],
      "filter": [                        // AND (no scoring)
        { "range": { "age": { "gte": 18 } } }
      ],
      "should": [                        // OR (boosts score)
        { "term": { "status": "active" } }
      ],
      "must_not": [                      // NOT
        { "term": { "deleted": true } }
      ]
    }
  }
}
```

**Multi-match** (search multiple fields):

```bash
GET /users/_search
{
  "query": {
    "multi_match": {
      "query": "john",
      "fields": ["name", "email"]
    }
  }
}
```

**Wildcard and fuzzy**:

```bash
# Wildcard
GET /users/_search
{
  "query": {
    "wildcard": {
      "name": "j*n"
    }
  }
}

# Fuzzy (typo tolerance)
GET /users/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "jhn",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

---

## 7. What are advanced queries?

**Q: Complex searches**

A:

**Phrase matching**:

```bash
# Exact phrase
GET /posts/_search
{
  "query": {
    "match_phrase": {
      "content": "elasticsearch tutorial"
    }
  }
}

# With slop (words can be N positions apart)
GET /posts/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "elasticsearch tutorial",
        "slop": 2
      }
    }
  }
}
```

**Prefix search**:

```bash
GET /users/_search
{
  "query": {
    "prefix": {
      "name": "joh"
    }
  }
}
```

**Exists query** (field has value):

```bash
GET /users/_search
{
  "query": {
    "exists": {
      "field": "email"
    }
  }
}
```

**Nested query** (for nested objects):

```bash
GET /posts/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "bool": {
          "must": [
            { "match": { "comments.author": "john" } },
            { "match": { "comments.text": "great" } }
          ]
        }
      }
    }
  }
}
```

**Function score** (custom scoring):

```bash
GET /products/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "name": "laptop" } },
      "functions": [
        {
          "filter": { "term": { "featured": true } },
          "weight": 2
        },
        {
          "field_value_factor": {
            "field": "popularity",
            "modifier": "log1p"
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "multiply"
    }
  }
}
```

**Query string** (Lucene syntax):

```bash
GET /users/_search
{
  "query": {
    "query_string": {
      "query": "(john OR jane) AND age:[18 TO 65]",
      "default_field": "name"
    }
  }
}
```

---

## 8. What are filters vs queries?

**Q: Difference**

A:

**Query context** — Calculates relevance score (_score):

```bash
GET /products/_search
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}
# Returns scored results
```

**Filter context** — Boolean yes/no, cached, faster:

```bash
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  }
}
# No scoring, uses cache
```

**Combined** (best practice):

```bash
GET /products/_search
{
  "query": {
    "bool": {
      "must": [                           // Scored
        { "match": { "name": "laptop" } }
      ],
      "filter": [                         // Not scored, cached
        { "term": { "category": "electronics" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  }
}
```

---

## 9. What are aggregations?

**Q: Analytics in Elasticsearch**

A: Group and analyze data.

**Metric aggregations**:

```bash
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_revenue": {
      "sum": { "field": "amount" }
    },
    "avg_price": {
      "avg": { "field": "amount" }
    },
    "max_price": {
      "max": { "field": "amount" }
    },
    "min_price": {
      "min": { "field": "amount" }
    },
    "stats": {
      "stats": { "field": "amount" }
    }
  }
}
```

**Bucket aggregations**:

```bash
# Terms (group by field value)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "by_status": {
      "terms": {
        "field": "status",
        "size": 10
      }
    }
  }
}

# Range
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500 }
        ]
      }
    }
  }
}

# Date histogram (time-series)
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      }
    }
  }
}
```

**Nested aggregations**:

```bash
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "by_status": {
      "terms": { "field": "status" },
      "aggs": {
        "total_amount": {
          "sum": { "field": "amount" }
        },
        "avg_amount": {
          "avg": { "field": "amount" }
        }
      }
    }
  }
}
```

**Pipeline aggregations** (operate on other aggregations):

```bash
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": { "field": "amount" }
        }
      }
    },
    "avg_monthly_sales": {
      "avg_bucket": {
        "buckets_path": "sales_per_month>total_sales"
      }
    }
  }
}
```

---

## 10. What is sorting and pagination?

**Q: Result ordering**

A:

**Sort**:

```bash
GET /users/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "age": { "order": "desc" } },
    { "name.keyword": { "order": "asc" } }
  ]
}

# Sort by score then field
GET /products/_search
{
  "query": { "match": { "name": "laptop" } },
  "sort": [
    { "_score": { "order": "desc" } },
    { "price": { "order": "asc" } }
  ]
}
```

**Pagination** (from/size):

```bash
GET /users/_search
{
  "from": 0,
  "size": 10,
  "query": { "match_all": {} }
}

# Page 2
GET /users/_search
{
  "from": 10,
  "size": 10,
  "query": { "match_all": {} }
}
```

**Scroll API** (for large result sets):

```bash
# Initial request
POST /users/_search?scroll=1m
{
  "size": 1000,
  "query": { "match_all": {} }
}
# Returns scroll_id

# Subsequent requests
POST /_search/scroll
{
  "scroll": "1m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}
```

**Search after** (better for deep pagination):

```bash
# First page
GET /users/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "age": "desc" },
    { "_id": "asc" }
  ]
}

# Next page (using sort values from last result)
GET /users/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [
    { "age": "desc" },
    { "_id": "asc" }
  ],
  "search_after": [30, "1234"]
}
```

---

## 11. What are analyzers?

**Q: Text analysis**

A: Process text for indexing and searching.

**Standard analyzer** (default):

```bash
POST /_analyze
{
  "analyzer": "standard",
  "text": "The Quick Brown Fox!"
}
# Output: ["the", "quick", "brown", "fox"]
```

**Built-in analyzers**:

```bash
# Whitespace
POST /_analyze
{
  "analyzer": "whitespace",
  "text": "The Quick Brown-Fox!"
}
# Output: ["The", "Quick", "Brown-Fox!"]

# Simple
POST /_analyze
{
  "analyzer": "simple",
  "text": "The Quick Brown-Fox!"
}
# Output: ["the", "quick", "brown", "fox"]

# English (stemming)
POST /_analyze
{
  "analyzer": "english",
  "text": "running runs"
}
# Output: ["run", "run"]
```

**Custom analyzer**:

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "snowball"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

**Analyzer components**:

```
Input text
  ↓
Character filters (remove HTML, etc.)
  ↓
Tokenizer (split into tokens)
  ↓
Token filters (lowercase, stemming, synonyms, etc.)
  ↓
Output tokens
```

**Search-time analyzer**:

```bash
PUT /my_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "standard",        // Index-time
        "search_analyzer": "english"   // Search-time
      }
    }
  }
}
```

---

## 12. What are mappings?

**Q: Schema definition**

A:

**Dynamic mapping** (auto-detected):

```bash
# Just index a document
PUT /users/_doc/1
{
  "name": "John",
  "age": 30,
  "email": "john@example.com"
}
# ES auto-creates mapping
```

**Explicit mapping** (recommended):

```bash
PUT /users
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "email": {
        "type": "keyword"
      },
      "age": {
        "type": "integer"
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "metadata": {
        "type": "object",
        "enabled": false  // Don't index
      }
    }
  }
}
```

**Multi-fields**:

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",           // For full-text search
        "fields": {
          "keyword": {            // For exact match, sorting
            "type": "keyword"
          },
          "completion": {         // For autocomplete
            "type": "completion"
          }
        }
      }
    }
  }
}
```

**Update mapping** (only add fields):

```bash
PUT /users/_mapping
{
  "properties": {
    "phone": {
      "type": "keyword"
    }
  }
}
```

**Reindex** (to change existing mapping):

```bash
# Create new index with new mapping
PUT /users_v2
{
  "mappings": { ... }
}

# Reindex
POST /_reindex
{
  "source": { "index": "users" },
  "dest": { "index": "users_v2" }
}

# Switch alias
POST /_aliases
{
  "actions": [
    { "remove": { "index": "users_v1", "alias": "users" } },
    { "add": { "index": "users_v2", "alias": "users" } }
  ]
}
```

---

## 13. What are suggesters?

**Q: Auto-complete and suggestions**

A:

**Term suggester** (spelling correction):

```bash
GET /products/_search
{
  "suggest": {
    "text": "laptpo",
    "my_suggestion": {
      "term": {
        "field": "name"
      }
    }
  }
}
```

**Phrase suggester** (phrase correction):

```bash
GET /products/_search
{
  "suggest": {
    "text": "gaming laptpo",
    "my_suggestion": {
      "phrase": {
        "field": "name",
        "size": 1,
        "gram_size": 3,
        "direct_generator": [{
          "field": "name",
          "suggest_mode": "always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```

**Completion suggester** (autocomplete):

```bash
# Mapping
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "suggest": {
        "type": "completion"
      }
    }
  }
}

# Index
PUT /products/_doc/1
{
  "name": "Gaming Laptop",
  "suggest": {
    "input": ["Gaming Laptop", "Laptop", "Gaming"],
    "weight": 10
  }
}

# Search
GET /products/_search
{
  "suggest": {
    "product-suggest": {
      "prefix": "gam",
      "completion": {
        "field": "suggest",
        "size": 5,
        "skip_duplicates": true
      }
    }
  }
}
```

---

## 14. What are highlights?

**Q: Show matched text**

A:

```bash
GET /posts/_search
{
  "query": {
    "match": { "content": "elasticsearch" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
# Returns: "This is an <em>elasticsearch</em> tutorial"

# Custom tags
GET /posts/_search
{
  "query": {
    "match": { "content": "elasticsearch" }
  },
  "highlight": {
    "pre_tags": ["<strong>"],
    "post_tags": ["</strong>"],
    "fields": {
      "content": {
        "fragment_size": 150,
        "number_of_fragments": 3
      }
    }
  }
}
```

---

## 15. What are scripts?

**Q: Custom logic**

A:

**Update with script**:

```bash
POST /products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.increase",
    "params": {
      "increase": 10
    }
  }
}
```

**Script in query**:

```bash
GET /products/_search
{
  "query": {
    "script_score": {
      "query": { "match_all": {} },
      "script": {
        "source": "Math.log(2 + doc['popularity'].value)"
      }
    }
  }
}
```

**Stored scripts**:

```bash
# Store
POST /_scripts/calculate_discount
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.price * (1 - params.discount_rate)"
  }
}

# Use
POST /products/_update/1
{
  "script": {
    "id": "calculate_discount",
    "params": {
      "discount_rate": 0.1
    }
  }
}
```

---

## 16. What are common patterns?

**Q: Design patterns**

A:

**Time-series data** (logs, metrics):

```bash
# Use date-based indices
PUT /logs-2024.01.15
PUT /logs-2024.01.16

# Index template
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "message": { "type": "text" }
      }
    }
  }
}

# Use Index Lifecycle Management (ILM)
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_size": "50gb"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

**E-commerce search**:

```bash
PUT /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "laptop",
            "fields": ["name^3", "description", "brand^2"]
          }
        }
      ],
      "filter": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "lte": 1000 } } },
        { "term": { "in_stock": true } }
      ]
    }
  },
  "sort": [
    { "_score": "desc" },
    { "popularity": "desc" }
  ],
  "aggs": {
    "brands": {
      "terms": { "field": "brand" }
    },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 500 },
          { "from": 500, "to": 1000 },
          { "from": 1000 }
        ]
      }
    }
  }
}
```

**Geo-search**:

```bash
# Find nearby locations
GET /restaurants/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "5km",
          "location": {
            "lat": 40.7128,
            "lon": -74.0060
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": 40.7128,
          "lon": -74.0060
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```

---

## 17. What is cluster management?

**Q: Operations**

A:

**Cluster health**:

```bash
GET /_cluster/health
GET /_cat/health?v
GET /_cat/nodes?v
GET /_cat/indices?v
```

**Node info**:

```bash
GET /_nodes
GET /_nodes/stats
GET /_cat/allocation?v
```

**Shard allocation**:

```bash
# Check shards
GET /_cat/shards?v

# Reroute shard
POST /_cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "my_index",
        "shard": 0,
        "from_node": "node1",
        "to_node": "node2"
      }
    }
  ]
}
```

**Cluster settings**:

```bash
# View settings
GET /_cluster/settings

# Update settings
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  },
  "transient": {
    "indices.recovery.max_bytes_per_sec": "50mb"
  }
}
```

---

## 18. What are best practices?

**Q: Production tips**

A:

**1. Index design**:
- Use date-based indices for time-series
- Set appropriate number of shards (5-10 GB per shard)
- Plan for replica shards (at least 1)
- Use index templates

**2. Mapping**:
- Define explicit mappings
- Use keyword for exact match, sorting, aggregations
- Use text for full-text search
- Disable `_source` if not needed (save space)

**3. Query optimization**:
- Use filters instead of queries when possible
- Avoid wildcards at start of string
- Use bool queries effectively
- Limit result size

**4. Shard management**:
- Don't over-shard (too many small shards is bad)
- Target 20-40 GB per shard
- Use shrink API to reduce shards
- Monitor shard distribution

**5. Performance**:
- Use bulk API for indexing
- Increase refresh interval for write-heavy
- Use routing for co-location
- Warm up queries and filters

```bash
# Good index settings
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "refresh_interval": "30s",
    "index": {
      "codec": "best_compression"
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "timestamp": { "type": "date" },
      "status": { "type": "keyword" }
    }
  }
}
```

---

## 19. What are monitoring tips?

**Q: Monitor Elasticsearch**

A:

**Key metrics**:

```bash
# Cluster health
GET /_cluster/health

# Node stats
GET /_nodes/stats

# Index stats
GET /_stats

# Pending tasks
GET /_cluster/pending_tasks

# Hot threads
GET /_nodes/hot_threads
```

**Important metrics to watch**:
- Cluster status (green/yellow/red)
- JVM heap usage (< 75%)
- Search latency
- Indexing latency
- Rejected threads
- Disk space
- CPU usage
- Unassigned shards

**Cat APIs** (human-readable):

```bash
GET /_cat/nodes?v&h=name,heap.percent,ram.percent,cpu,load_1m
GET /_cat/indices?v&s=store.size:desc
GET /_cat/shards?v
GET /_cat/thread_pool?v
```

---

## 20. What are common mistakes?

**Q: Avoid these**

A:

**1. Wrong shard count**:

```bash
# BAD: Too many small shards
PUT /small_index
{
  "settings": {
    "number_of_shards": 50  # Only 1 GB of data
  }
}

# GOOD: Appropriate shard count
PUT /small_index
{
  "settings": {
    "number_of_shards": 1
  }
}
```

**2. Not using filters**:

```bash
# BAD: Query context for exact matches
GET /products/_search
{
  "query": {
    "term": { "category": "electronics" }
  }
}

# GOOD: Filter context (cached)
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "category": "electronics" } }
      ]
    }
  }
}
```

**3. Wildcard at start**:

```bash
# BAD: Very slow
GET /users/_search
{
  "query": {
    "wildcard": { "email": "*@gmail.com" }
  }
}

# GOOD: Use suffix tree or different approach
```

**4. Not using bulk API**:

```bash
# BAD: Individual requests
PUT /users/_doc/1 {...}
PUT /users/_doc/2 {...}

# GOOD: Bulk
POST /_bulk
{"index": {"_index": "users", "_id": "1"}}
{...}
{"index": {"_index": "users", "_id": "2"}}
{...}
```

**5. Deep pagination**:

```bash
# BAD: from=10000 is very expensive
GET /users/_search
{
  "from": 10000,
  "size": 10
}

# GOOD: Use search_after
```

**6. Not setting replica shards**

**7. Using default mapping for everything**

---

## 21. What is a real-world example?

**Q: Complete setup**

A:

```bash
# Index template for application logs
PUT /_index_template/app_logs_template
{
  "index_patterns": ["app-logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1,
      "refresh_interval": "30s",
      "codec": "best_compression"
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "logger": { "type": "keyword" },
        "message": { "type": "text" },
        "thread": { "type": "keyword" },
        "user_id": { "type": "keyword" },
        "request_id": { "type": "keyword" },
        "duration_ms": { "type": "long" },
        "status_code": { "type": "short" },
        "ip_address": { "type": "ip" },
        "url": { "type": "keyword" },
        "user_agent": {
          "type": "text",
          "fields": {
            "keyword": { "type": "keyword" }
          }
        },
        "metadata": { "type": "object" }
      }
    }
  }
}

# Sample queries

# 1. Error logs in last hour
GET /app-logs-*/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "level": "ERROR" } },
        {
          "range": {
            "timestamp": {
              "gte": "now-1h"
            }
          }
        }
      ]
    }
  },
  "sort": [{ "timestamp": "desc" }]
}

# 2. Slow requests (>1s)
GET /app-logs-*/_search
{
  "query": {
    "bool": {
      "filter": [
        { "range": { "duration_ms": { "gte": 1000 } } }
      ]
    }
  },
  "sort": [{ "duration_ms": "desc" }]
}

# 3. Errors by logger (aggregation)
GET /app-logs-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "term": { "level": "ERROR" } },
        { "range": { "timestamp": { "gte": "now-24h" } } }
      ]
    }
  },
  "aggs": {
    "errors_by_logger": {
      "terms": {
        "field": "logger",
        "size": 10
      },
      "aggs": {
        "recent_errors": {
          "top_hits": {
            "size": 3,
            "sort": [{ "timestamp": "desc" }],
            "_source": ["timestamp", "message"]
          }
        }
      }
    }
  }
}

# 4. Request rate over time
GET /app-logs-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "timestamp": { "gte": "now-24h" }
    }
  },
  "aggs": {
    "requests_over_time": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1h"
      },
      "aggs": {
        "avg_duration": {
          "avg": { "field": "duration_ms" }
        },
        "status_codes": {
          "terms": { "field": "status_code" }
        }
      }
    }
  }
}

# 5. User activity tracking
GET /app-logs-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "term": { "user_id": "user123" } },
        { "range": { "timestamp": { "gte": "now-7d" } } }
      ]
    }
  },
  "aggs": {
    "activity_timeline": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      }
    },
    "popular_urls": {
      "terms": {
        "field": "url",
        "size": 10
      }
    }
  }
}
```

---

## Elasticsearch Interview Tips

1. **Understand Lucene** — ES is built on top
2. **Full-text search** — Analyzers, tokenizers
3. **Inverted index** — How search works
4. **Distributed nature** — Shards, replicas, nodes
5. **Query vs filter** — Scoring vs boolean
6. **Aggregations** — Analytics capabilities
7. **Mapping** — Schema design
8. **Index lifecycle** — Time-series patterns
9. **Performance** — Bulk API, filters, caching
10. **ELK stack** — Elasticsearch + Logstash + Kibana
11. **Common use cases** — Logs, search, metrics
12. **Cluster operations** — Health, shards, allocation
13. **Best practices** — Shard sizing, replicas
14. **Monitoring** — Key metrics to watch
15. **Real projects** — Production experience
