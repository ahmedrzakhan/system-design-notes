# Elasticsearch 50 Practice Questions

_Complete Elasticsearch Query DSL Practice Guide for Local Environment_

## 🚀 Setup Instructions

1. **Install Elasticsearch** (if not already installed):

```bash
# Ubuntu/Debian (using APT)
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch

# macOS
brew tap elastic/tap
brew install elastic/tap/elasticsearch-full

# Docker (Easiest for local practice)
docker run -d --name elasticsearch \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  elasticsearch:8.11.0

# Start Elasticsearch
sudo systemctl start elasticsearch  # Linux
brew services start elasticsearch   # macOS
```

2. **Verify Installation**:

```bash
# Check Elasticsearch status
curl -X GET "localhost:9200/"

# Check cluster health
curl -X GET "localhost:9200/_cluster/health?pretty"

# Check version
curl -X GET "localhost:9200/" | jq '.version'
```

3. **Install Kibana (Optional but Recommended)**:

```bash
# Docker
docker run -d --name kibana \
  --link elasticsearch:elasticsearch \
  -p 5601:5601 \
  kibana:8.11.0

# Access Kibana Dev Tools at: http://localhost:5601/app/dev_tools#/console
```

4. **Using the Examples**:

```bash
# All examples can be run using curl or Kibana Dev Tools
# For curl: curl -X POST "localhost:9200/index/_search?pretty" -H 'Content-Type: application/json' -d'{ query }'
# For Kibana: Just paste the query body in Dev Tools
```

---

## 📊 Elasticsearch Concepts

### Key Components

- **Index**: Collection of documents (similar to database)
- **Document**: Basic unit of information (JSON)
- **Mapping**: Schema definition for documents
- **Shard**: Partition of an index
- **Replica**: Copy of shard for fault tolerance
- **Inverted Index**: Core data structure for fast search

### Query Types

- **Query Context**: Scoring relevance (how well does it match?)
- **Filter Context**: Yes/no filtering (does it match?)
- **Aggregations**: Analytics and statistics
- **Full-Text Search**: Text analysis and relevance scoring

---

## 🟢 SECTION 1: BASIC OPERATIONS (Questions 1-5)

### Question 1: Creating Index and Indexing Documents

**Difficulty:** Easy
**Topic:** CRUD Operations

```json
# Create index with settings
PUT /products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}

# Index a document (with auto-generated ID)
POST /products/_doc
{
  "name": "Laptop Pro",
  "category": "Electronics",
  "price": 1299.99,
  "in_stock": true,
  "tags": ["laptop", "computer", "electronics"]
}

# Index a document (with specific ID)
PUT /products/_doc/1
{
  "name": "Wireless Mouse",
  "category": "Electronics",
  "price": 29.99,
  "in_stock": true,
  "tags": ["mouse", "wireless", "electronics"]
}

# Get document by ID
GET /products/_doc/1

# Search all documents
GET /products/_search
```

### Question 2: Basic Search Query

**Difficulty:** Easy
**Topic:** Match Query

```json
# Match query (full-text search)
GET /products/_search
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}

# Match all query
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

# Multi-match query (search across multiple fields)
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "wireless",
      "fields": ["name", "tags"]
    }
  }
}
```

### Question 3: Term-level Queries (Exact Match)

**Difficulty:** Easy
**Topic:** Term Query

```json
# Term query (exact match, not analyzed)
GET /products/_search
{
  "query": {
    "term": {
      "category.keyword": "Electronics"
    }
  }
}

# Terms query (match any of multiple values)
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": ["laptop", "mouse", "keyboard"]
    }
  }
}

# Range query
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 20,
        "lte": 100
      }
    }
  }
}

# Exists query
GET /products/_search
{
  "query": {
    "exists": {
      "field": "in_stock"
    }
  }
}
```

### Question 4: Bool Query (Combining Queries)

**Difficulty:** Easy
**Topic:** Boolean Logic

```json
# Bool query with must (AND)
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "category": "Electronics" }},
        { "range": { "price": { "lte": 1000 }}}
      ]
    }
  }
}

# Bool query with should (OR)
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name": "laptop" }},
        { "match": { "name": "mouse" }}
      ],
      "minimum_should_match": 1
    }
  }
}

# Bool query with must_not
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "category": "Electronics" }}
      ],
      "must_not": [
        { "term": { "in_stock": false }}
      ]
    }
  }
}

# Bool query with filter (no scoring)
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" }}
      ],
      "filter": [
        { "range": { "price": { "gte": 1000 }}}
      ]
    }
  }
}
```

### Question 5: Update and Delete

**Difficulty:** Easy
**Topic:** Document Modification

```json
# Update document (partial update)
POST /products/_update/1
{
  "doc": {
    "price": 24.99,
    "in_stock": false
  }
}

# Update with script
POST /products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.increase",
    "params": {
      "increase": 5
    }
  }
}

# Update by query (bulk update)
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock = true"
  },
  "query": {
    "term": {
      "category.keyword": "Electronics"
    }
  }
}

# Delete document
DELETE /products/_doc/1

# Delete by query
POST /products/_delete_by_query
{
  "query": {
    "match": {
      "category": "Obsolete"
    }
  }
}
```

---

## 🟡 SECTION 2: FULL-TEXT SEARCH (Questions 6-12)

### Question 6: Match Phrase Query

**Difficulty:** Medium
**Topic:** Phrase Matching

```json
# Create sample data
POST /articles/_bulk
{"index": {"_id": "1"}}
{"title": "Quick brown fox", "content": "The quick brown fox jumps over the lazy dog"}
{"index": {"_id": "2"}}
{"title": "Lazy dog story", "content": "A lazy dog sleeps all day"}
{"index": {"_id": "3"}}
{"title": "Fox hunting", "content": "Brown fox is very quick"}

# Match phrase (exact phrase order)
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": "quick brown fox"
    }
  }
}

# Match phrase with slop (allows word gaps)
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": {
        "query": "quick fox",
        "slop": 2
      }
    }
  }
}
```

### Question 7: Query String Search

**Difficulty:** Medium
**Topic:** Advanced Query Syntax

```json
# Simple query string
GET /articles/_search
{
  "query": {
    "query_string": {
      "query": "quick AND fox",
      "default_field": "content"
    }
  }
}

# Query string with field specification
GET /articles/_search
{
  "query": {
    "query_string": {
      "query": "title:fox OR content:lazy"
    }
  }
}

# Query string with wildcards and operators
GET /articles/_search
{
  "query": {
    "query_string": {
      "query": "qu?ck bro*",
      "default_field": "content"
    }
  }
}

# Simple query string (simpler syntax, fewer operators)
GET /articles/_search
{
  "query": {
    "simple_query_string": {
      "query": "quick + fox | dog",
      "fields": ["title", "content"]
    }
  }
}
```

### Question 8: Fuzzy Search

**Difficulty:** Medium
**Topic:** Typo Tolerance

```json
# Fuzzy query (handles typos)
GET /articles/_search
{
  "query": {
    "fuzzy": {
      "content": {
        "value": "quik",
        "fuzziness": "AUTO"
      }
    }
  }
}

# Match query with fuzziness
GET /articles/_search
{
  "query": {
    "match": {
      "content": {
        "query": "quikc browm",
        "fuzziness": "AUTO",
        "operator": "and"
      }
    }
  }
}

# Fuzzy with prefix length (first N characters must match exactly)
GET /articles/_search
{
  "query": {
    "fuzzy": {
      "content": {
        "value": "quikc",
        "fuzziness": 2,
        "prefix_length": 2
      }
    }
  }
}
```

### Question 9: Boosting and Relevance

**Difficulty:** Medium
**Topic:** Score Manipulation

```json
# Boost specific fields
GET /articles/_search
{
  "query": {
    "multi_match": {
      "query": "fox",
      "fields": ["title^3", "content"]
    }
  }
}

# Bool query with boost
GET /articles/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "fox",
              "boost": 2.0
            }
          }
        },
        {
          "match": {
            "content": "fox"
          }
        }
      ]
    }
  }
}

# Boosting query (positive/negative)
GET /articles/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "content": "fox"
        }
      },
      "negative": {
        "match": {
          "content": "lazy"
        }
      },
      "negative_boost": 0.5
    }
  }
}

# Function score query
GET /articles/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "content": "fox"
        }
      },
      "boost_mode": "multiply",
      "functions": [
        {
          "filter": {
            "match": {
              "title": "quick"
            }
          },
          "weight": 2
        }
      ]
    }
  }
}
```

### Question 10: Highlighting

**Difficulty:** Medium
**Topic:** Search Result Highlighting

```json
# Basic highlighting
GET /articles/_search
{
  "query": {
    "match": {
      "content": "quick brown fox"
    }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}

# Custom highlight tags
GET /articles/_search
{
  "query": {
    "match": {
      "content": "fox"
    }
  },
  "highlight": {
    "pre_tags": ["<strong>"],
    "post_tags": ["</strong>"],
    "fields": {
      "content": {
        "fragment_size": 50,
        "number_of_fragments": 3
      }
    }
  }
}
```

### Question 11: Autocomplete with Prefix

**Difficulty:** Medium
**Topic:** Search Suggestions

```json
# Create index with edge ngram analyzer
PUT /autocomplete
{
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "tokenizer": "autocomplete",
          "filter": ["lowercase"]
        },
        "autocomplete_search": {
          "tokenizer": "lowercase"
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": ["letter", "digit"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "autocomplete",
        "search_analyzer": "autocomplete_search"
      }
    }
  }
}

# Index sample data
POST /autocomplete/_doc/1
{
  "title": "Elasticsearch Guide"
}

POST /autocomplete/_doc/2
{
  "title": "Elastic Search Basics"
}

# Search with prefix
GET /autocomplete/_search
{
  "query": {
    "match": {
      "title": "ela"
    }
  }
}

# Alternative: Prefix query
GET /autocomplete/_search
{
  "query": {
    "prefix": {
      "title.keyword": "Ela"
    }
  }
}
```

### Question 12: Search Suggestions (Completion Suggester)

**Difficulty:** Medium
**Topic:** Type-ahead

```json
# Create index with completion field
PUT /suggestions
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion"
      },
      "title": {
        "type": "text"
      }
    }
  }
}

# Index documents
POST /suggestions/_doc/1
{
  "title": "Elasticsearch Tutorial",
  "suggest": ["Elasticsearch", "Tutorial", "Elasticsearch Tutorial"]
}

POST /suggestions/_doc/2
{
  "title": "Elastic Stack Guide",
  "suggest": ["Elastic", "Stack", "Guide", "Elastic Stack"]
}

# Get suggestions
POST /suggestions/_search
{
  "suggest": {
    "my-suggestion": {
      "prefix": "ela",
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

## 🔵 SECTION 3: AGGREGATIONS (Questions 13-19)

### Question 13: Metric Aggregations

**Difficulty:** Easy
**Topic:** Statistical Metrics

```json
# Create sample e-commerce data
POST /orders/_bulk
{"index": {}}
{"product": "Laptop", "price": 1200, "quantity": 1, "category": "Electronics", "date": "2024-01-15"}
{"index": {}}
{"product": "Mouse", "price": 25, "quantity": 2, "category": "Electronics", "date": "2024-01-15"}
{"index": {}}
{"product": "Desk", "price": 300, "quantity": 1, "category": "Furniture", "date": "2024-01-16"}
{"index": {}}
{"product": "Chair", "price": 150, "quantity": 4, "category": "Furniture", "date": "2024-01-16"}

# Stats aggregation (count, min, max, avg, sum)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}

# Individual metric aggregations
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    },
    "total_revenue": {
      "sum": {
        "field": "price"
      }
    },
    "max_price": {
      "max": {
        "field": "price"
      }
    },
    "min_price": {
      "min": {
        "field": "price"
      }
    }
  }
}

# Value count
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "product_count": {
      "value_count": {
        "field": "product.keyword"
      }
    }
  }
}
```

### Question 14: Bucket Aggregations

**Difficulty:** Medium
**Topic:** Grouping Data

```json
# Terms aggregation (group by category)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category.keyword",
        "size": 10
      }
    }
  }
}

# Terms with nested metrics
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category.keyword"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "total_revenue": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

# Range aggregation
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 200 },
          { "from": 200 }
        ]
      }
    }
  }
}
```

### Question 15: Date Histogram

**Difficulty:** Medium
**Topic:** Time-based Aggregation

```json
# Date histogram (group by time interval)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "field": "price"
          }
        }
      }
    }
  }
}

# Date histogram with time zone
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "hour",
        "time_zone": "America/New_York",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}

# Fixed interval
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "fixed_interval": "30m"
      }
    }
  }
}
```

### Question 16: Pipeline Aggregations

**Difficulty:** Hard
**Topic:** Aggregation on Aggregations

```json
# Moving average
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_per_day": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "field": "price"
          }
        },
        "moving_avg_revenue": {
          "moving_avg": {
            "buckets_path": "daily_revenue"
          }
        }
      }
    }
  }
}

# Derivative (rate of change)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_per_day": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "field": "price"
          }
        },
        "revenue_derivative": {
          "derivative": {
            "buckets_path": "daily_revenue"
          }
        }
      }
    }
  }
}

# Cumulative sum
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_per_day": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "field": "price"
          }
        },
        "cumulative_revenue": {
          "cumulative_sum": {
            "buckets_path": "daily_revenue"
          }
        }
      }
    }
  }
}
```

### Question 17: Percentiles and Percentile Ranks

**Difficulty:** Medium
**Topic:** Distribution Analysis

```json
# Percentiles
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_percentiles": {
      "percentiles": {
        "field": "price",
        "percents": [25, 50, 75, 95, 99]
      }
    }
  }
}

# Percentile ranks (what percentile is this value?)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "price_percentile_ranks": {
      "percentile_ranks": {
        "field": "price",
        "values": [100, 500, 1000]
      }
    }
  }
}
```

### Question 18: Cardinality (Unique Count)

**Difficulty:** Medium
**Topic:** Distinct Values

```json
# Cardinality aggregation (approximate unique count)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "product.keyword"
      }
    },
    "unique_categories": {
      "cardinality": {
        "field": "category.keyword"
      }
    }
  }
}

# Cardinality with precision threshold
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "product.keyword",
        "precision_threshold": 1000
      }
    }
  }
}
```

### Question 19: Nested and Reverse Nested Aggregations

**Difficulty:** Hard
**Topic:** Complex Object Aggregation

```json
# Create index with nested type
PUT /products_nested
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "reviews": {
        "type": "nested",
        "properties": {
          "author": { "type": "keyword" },
          "rating": { "type": "integer" },
          "comment": { "type": "text" }
        }
      }
    }
  }
}

# Index document with nested objects
POST /products_nested/_doc/1
{
  "name": "Laptop",
  "reviews": [
    { "author": "John", "rating": 5, "comment": "Great!" },
    { "author": "Jane", "rating": 4, "comment": "Good" }
  ]
}

# Nested aggregation
GET /products_nested/_search
{
  "size": 0,
  "aggs": {
    "nested_reviews": {
      "nested": {
        "path": "reviews"
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "reviews.rating"
          }
        },
        "top_reviewers": {
          "terms": {
            "field": "reviews.author"
          }
        }
      }
    }
  }
}

# Reverse nested (go back to parent)
GET /products_nested/_search
{
  "size": 0,
  "aggs": {
    "nested_reviews": {
      "nested": {
        "path": "reviews"
      },
      "aggs": {
        "high_ratings": {
          "filter": {
            "range": {
              "reviews.rating": { "gte": 4 }
            }
          },
          "aggs": {
            "back_to_product": {
              "reverse_nested": {},
              "aggs": {
                "product_names": {
                  "terms": {
                    "field": "name.keyword"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

---

## 🟣 SECTION 4: MAPPINGS & ANALYZERS (Questions 20-26)

### Question 20: Custom Mappings

**Difficulty:** Medium
**Topic:** Schema Definition

```json
# Create index with explicit mappings
PUT /users
{
  "mappings": {
    "properties": {
      "username": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "full_name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "age": {
        "type": "integer"
      },
      "bio": {
        "type": "text",
        "analyzer": "english"
      },
      "joined_date": {
        "type": "date",
        "format": "yyyy-MM-dd"
      },
      "is_active": {
        "type": "boolean"
      },
      "location": {
        "type": "geo_point"
      },
      "preferences": {
        "type": "object",
        "enabled": false
      }
    }
  }
}

# View mappings
GET /users/_mapping
```

### Question 21: Multi-fields

**Difficulty:** Medium
**Topic:** Multiple Analyzers per Field

```json
# Create index with multi-fields
PUT /books
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      },
      "author": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

# Search on analyzed field
GET /books/_search
{
  "query": {
    "match": {
      "title": "quick guide"
    }
  }
}

# Search on keyword field (exact match)
GET /books/_search
{
  "query": {
    "term": {
      "title.keyword": "Quick Guide to Elasticsearch"
    }
  }
}

# Aggregate on keyword field
GET /books/_search
{
  "size": 0,
  "aggs": {
    "popular_authors": {
      "terms": {
        "field": "author.keyword"
      }
    }
  }
}
```

### Question 22: Custom Analyzers

**Difficulty:** Hard
**Topic:** Text Analysis

```json
# Create index with custom analyzer
PUT /custom_text
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            ":) => happy",
            ":( => sad"
          ]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_"
        },
        "english_stemmer": {
          "type": "stemmer",
          "language": "english"
        },
        "english_possessive_stemmer": {
          "type": "stemmer",
          "language": "possessive_english"
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip", "emoticons"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "english_possessive_stemmer",
            "english_stop",
            "english_stemmer"
          ]
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

# Test analyzer
POST /custom_text/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I'm :) happy with <b>Elasticsearch's</b> capabilities!"
}
```

### Question 23: Normalizers (for Keywords)

**Difficulty:** Medium
**Topic:** Keyword Normalization

```json
# Create index with normalizer
PUT /products_normalized
{
  "settings": {
    "analysis": {
      "normalizer": {
        "lowercase_normalizer": {
          "type": "custom",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "sku": {
        "type": "keyword",
        "normalizer": "lowercase_normalizer"
      },
      "name": {
        "type": "text"
      }
    }
  }
}

# Index documents
POST /products_normalized/_doc/1
{
  "sku": "ABC-123",
  "name": "Product A"
}

# Query (case-insensitive because of normalizer)
GET /products_normalized/_search
{
  "query": {
    "term": {
      "sku": "abc-123"
    }
  }
}
```

### Question 24: Dynamic Templates

**Difficulty:** Hard
**Topic:** Automatic Mapping Rules

```json
# Create index with dynamic templates
PUT /logs
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "match": "*_id",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "longs_as_integers": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "strings_as_text": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    ]
  }
}

# Index document (mapping is auto-generated based on templates)
POST /logs/_doc/1
{
  "user_id": "abc123",
  "message": "User logged in",
  "count": 42
}

# Check generated mapping
GET /logs/_mapping
```

### Question 25: Geo Spatial Data

**Difficulty:** Medium
**Topic:** Location-based Queries

```json
# Create index with geo_point
PUT /locations
{
  "mappings": {
    "properties": {
      "name": { "type": "keyword" },
      "location": { "type": "geo_point" }
    }
  }
}

# Index locations
POST /locations/_doc/1
{
  "name": "New York",
  "location": {
    "lat": 40.7128,
    "lon": -74.0060
  }
}

POST /locations/_doc/2
{
  "name": "San Francisco",
  "location": "37.7749,-122.4194"
}

# Geo distance query
GET /locations/_search
{
  "query": {
    "geo_distance": {
      "distance": "100km",
      "location": {
        "lat": 40.0,
        "lon": -74.0
      }
    }
  }
}

# Geo bounding box query
GET /locations/_search
{
  "query": {
    "geo_bounding_box": {
      "location": {
        "top_left": {
          "lat": 42,
          "lon": -75
        },
        "bottom_right": {
          "lat": 40,
          "lon": -73
        }
      }
    }
  }
}

# Geo distance aggregation
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "rings_around_ny": {
      "geo_distance": {
        "field": "location",
        "origin": "40.7128, -74.0060",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```

### Question 26: Index Templates

**Difficulty:** Medium
**Topic:** Reusable Index Configuration

```json
# Create index template
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "refresh_interval": "5s"
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date"
        },
        "level": {
          "type": "keyword"
        },
        "message": {
          "type": "text"
        },
        "host": {
          "type": "keyword"
        }
      }
    }
  },
  "priority": 200
}

# Create index matching pattern (template is auto-applied)
PUT /logs-2024-01-15

# Verify settings and mappings
GET /logs-2024-01-15
```

---

## 🟠 SECTION 5: ADVANCED QUERIES (Questions 27-33)

### Question 27: Nested Queries

**Difficulty:** Hard
**Topic:** Querying Nested Objects

```json
# Create index with nested type (from Question 19)
# Query nested objects
GET /products_nested/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "bool": {
          "must": [
            { "match": { "reviews.author": "John" }},
            { "range": { "reviews.rating": { "gte": 4 }}}
          ]
        }
      },
      "inner_hits": {}
    }
  }
}

# Nested query with score mode
GET /products_nested/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "match": {
          "reviews.comment": "great"
        }
      },
      "score_mode": "avg"
    }
  }
}
```

### Question 28: Parent-Child Relationships

**Difficulty:** Hard
**Topic:** Join Field

```json
# Create index with join field
PUT /company
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "relation": {
        "type": "join",
        "relations": {
          "department": "employee"
        }
      }
    }
  }
}

# Index parent documents (departments)
POST /company/_doc/1?routing=1
{
  "name": "Engineering",
  "relation": {
    "name": "department"
  }
}

# Index child documents (employees)
POST /company/_doc/101?routing=1
{
  "name": "John Doe",
  "relation": {
    "name": "employee",
    "parent": "1"
  }
}

POST /company/_doc/102?routing=1
{
  "name": "Jane Smith",
  "relation": {
    "name": "employee",
    "parent": "1"
  }
}

# Has child query (find departments with specific employees)
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "match": {
          "name": "John"
        }
      }
    }
  }
}

# Has parent query (find employees in specific department)
GET /company/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "match": {
          "name": "Engineering"
        }
      }
    }
  }
}
```

### Question 29: Script Queries

**Difficulty:** Hard
**Topic:** Custom Scripting

```json
# Script query
GET /orders/_search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": {
            "source": "doc['price'].value * doc['quantity'].value > params.threshold",
            "params": {
              "threshold": 100
            }
          }
        }
      }
    }
  }
}

# Script score query
GET /products/_search
{
  "query": {
    "script_score": {
      "query": {
        "match_all": {}
      },
      "script": {
        "source": "_score * params.boost / doc['price'].value",
        "params": {
          "boost": 1000
        }
      }
    }
  }
}

# Scripted field (computed at query time)
GET /orders/_search
{
  "script_fields": {
    "total_price": {
      "script": {
        "source": "doc['price'].value * doc['quantity'].value"
      }
    }
  }
}
```

### Question 30: Percolate Queries

**Difficulty:** Advanced
**Topic:** Reverse Search

```json
# Create index with percolator field
PUT /alerts
{
  "mappings": {
    "properties": {
      "query": {
        "type": "percolator"
      },
      "alert_name": {
        "type": "keyword"
      }
    }
  }
}

# Index queries (not documents!)
POST /alerts/_doc/1
{
  "alert_name": "high_price_alert",
  "query": {
    "range": {
      "price": {
        "gte": 1000
      }
    }
  }
}

POST /alerts/_doc/2
{
  "alert_name": "electronics_alert",
  "query": {
    "term": {
      "category.keyword": "Electronics"
    }
  }
}

# Percolate: which queries match this document?
GET /alerts/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "product": "Laptop",
        "price": 1200,
        "category": "Electronics"
      }
    }
  }
}
```

### Question 31: More Like This (MLT)

**Difficulty:** Medium
**Topic:** Similarity Search

```json
# More like this query
GET /articles/_search
{
  "query": {
    "more_like_this": {
      "fields": ["content"],
      "like": [
        {
          "_index": "articles",
          "_id": "1"
        }
      ],
      "min_term_freq": 1,
      "min_doc_freq": 1
    }
  }
}

# MLT with input text
GET /articles/_search
{
  "query": {
    "more_like_this": {
      "fields": ["title", "content"],
      "like": "quick brown fox jumps",
      "min_term_freq": 1,
      "max_query_terms": 12
    }
  }
}
```

### Question 32: Wildcard and Regexp Queries

**Difficulty:** Medium
**Topic:** Pattern Matching

```json
# Wildcard query
GET /products/_search
{
  "query": {
    "wildcard": {
      "name.keyword": {
        "value": "*top*",
        "case_insensitive": true
      }
    }
  }
}

# Regexp query
GET /products/_search
{
  "query": {
    "regexp": {
      "name.keyword": {
        "value": "lap[a-z]+",
        "flags": "ALL",
        "case_insensitive": true
      }
    }
  }
}

# Note: These queries can be slow on large datasets
```

### Question 33: Span Queries

**Difficulty:** Advanced
**Topic:** Positional Queries

```json
# Span near query (terms within N positions)
GET /articles/_search
{
  "query": {
    "span_near": {
      "clauses": [
        { "span_term": { "content": "quick" }},
        { "span_term": { "content": "fox" }}
      ],
      "slop": 5,
      "in_order": true
    }
  }
}

# Span first (term within first N positions)
GET /articles/_search
{
  "query": {
    "span_first": {
      "match": {
        "span_term": { "content": "quick" }
      },
      "end": 3
    }
  }
}

# Span multi (combine with wildcards)
GET /articles/_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_multi": {
            "match": {
              "prefix": { "content": "qui" }
            }
          }
        },
        { "span_term": { "content": "fox" }}
      ],
      "slop": 2,
      "in_order": true
    }
  }
}
```

---

## 🔴 SECTION 6: PERFORMANCE & OPTIMIZATION (Questions 34-40)

### Question 34: Pagination

**Difficulty:** Medium
**Topic:** Efficient Result Retrieval

```json
# From/size pagination (simple but inefficient for deep pagination)
GET /products/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}

# Search after (efficient for deep pagination)
GET /products/_search
{
  "size": 10,
  "query": {
    "match_all": {}
  },
  "sort": [
    { "price": "asc" },
    { "_id": "asc" }
  ]
}

# Next page: use sort values from last document
GET /products/_search
{
  "size": 10,
  "query": {
    "match_all": {}
  },
  "search_after": [29.99, "last_doc_id"],
  "sort": [
    { "price": "asc" },
    { "_id": "asc" }
  ]
}

# Scroll API (for processing all documents)
POST /products/_search?scroll=1m
{
  "size": 100,
  "query": {
    "match_all": {}
  }
}

# Continue scrolling
POST /_search/scroll
{
  "scroll": "1m",
  "scroll_id": "<scroll_id_from_previous_response>"
}
```

### Question 35: Source Filtering

**Difficulty:** Easy
**Topic:** Reducing Response Size

```json
# Exclude _source entirely
GET /products/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  }
}

# Include specific fields
GET /products/_search
{
  "_source": ["name", "price"],
  "query": {
    "match_all": {}
  }
}

# Include/exclude patterns
GET /products/_search
{
  "_source": {
    "includes": ["name", "price*"],
    "excludes": ["*.internal"]
  },
  "query": {
    "match_all": {}
  }
}

# Stored fields (requires store:true in mapping)
GET /products/_search
{
  "stored_fields": ["name", "price"],
  "query": {
    "match_all": {}
  }
}
```

### Question 36: Query Optimization

**Difficulty:** Hard
**Topic:** Performance Tuning

```json
# Use filter context instead of query context (when scoring not needed)
# Bad: Query context (calculates scores)
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "category.keyword": "Electronics" }},
        { "range": { "price": { "lte": 1000 }}}
      ]
    }
  }
}

# Good: Filter context (cached, no scoring)
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "category.keyword": "Electronics" }},
        { "range": { "price": { "lte": 1000 }}}
      ]
    }
  }
}

# Combine query and filter
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" }}  # Scored
      ],
      "filter": [
        { "term": { "in_stock": true }},  # Not scored, cached
        { "range": { "price": { "lte": 2000 }}}
      ]
    }
  }
}

# Use constant_score for non-scored queries
GET /products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "category.keyword": "Electronics" }
      },
      "boost": 1.0
    }
  }
}
```

### Question 37: Bulk Operations

**Difficulty:** Medium
**Topic:** Batch Processing

```json
# Bulk index
POST /_bulk
{"index": {"_index": "products", "_id": "1"}}
{"name": "Product 1", "price": 10}
{"index": {"_index": "products", "_id": "2"}}
{"name": "Product 2", "price": 20}

# Bulk with different operations
POST /_bulk
{"index": {"_index": "products", "_id": "1"}}
{"name": "Product 1", "price": 10}
{"update": {"_index": "products", "_id": "2"}}
{"doc": {"price": 25}}
{"delete": {"_index": "products", "_id": "3"}}

# Bulk with error handling
POST /products/_bulk
{"index": {"_id": "1"}}
{"name": "Product 1"}
{"index": {"_id": "2"}}
{"invalid_field": "will_fail"}
{"index": {"_id": "3"}}
{"name": "Product 3"}
# Response shows which operations failed
```

### Question 38: Reindex

**Difficulty:** Medium
**Topic:** Index Migration

```json
# Reindex all documents
POST /_reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}

# Reindex with query filter
POST /_reindex
{
  "source": {
    "index": "products",
    "query": {
      "term": {
        "category.keyword": "Electronics"
      }
    }
  },
  "dest": {
    "index": "electronics_only"
  }
}

# Reindex with script transformation
POST /_reindex
{
  "source": {
    "index": "old_products"
  },
  "dest": {
    "index": "new_products"
  },
  "script": {
    "source": "ctx._source.price = ctx._source.price * 1.1"
  }
}

# Reindex from remote cluster
POST /_reindex
{
  "source": {
    "remote": {
      "host": "http://remote-cluster:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "source_index"
  },
  "dest": {
    "index": "dest_index"
  }
}
```

### Question 39: Index Aliases

**Difficulty:** Medium
**Topic:** Zero-downtime Deployments

```json
# Create alias
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "products_v1",
        "alias": "products"
      }
    }
  ]
}

# Switch alias atomically
POST /_aliases
{
  "actions": [
    { "remove": { "index": "products_v1", "alias": "products" }},
    { "add": { "index": "products_v2", "alias": "products" }}
  ]
}

# Filtered alias
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "products",
        "alias": "electronics",
        "filter": {
          "term": {
            "category.keyword": "Electronics"
          }
        }
      }
    }
  ]
}

# Query via alias
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

### Question 40: Index Lifecycle Management (ILM)

**Difficulty:** Advanced
**Topic:** Automated Index Management

```json
# Create ILM policy
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "7d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

# Create index template with ILM
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs_policy",
      "index.lifecycle.rollover_alias": "logs"
    }
  }
}

# Create initial index with write alias
PUT /logs-000001
{
  "aliases": {
    "logs": {
      "is_write_index": true
    }
  }
}
```

---

## 🟡 SECTION 7: MONITORING & ADMINISTRATION (Questions 41-47)

### Question 41: Cluster Health and Stats

**Difficulty:** Easy
**Topic:** Cluster Monitoring

```json
# Cluster health
GET /_cluster/health

# Detailed cluster health
GET /_cluster/health?level=indices

# Cluster stats
GET /_cluster/stats

# Node stats
GET /_nodes/stats

# Index stats
GET /products/_stats

# Specific metric stats
GET /_nodes/stats/indices,os,process
```

### Question 42: Cat APIs (Human-Readable)

**Difficulty:** Easy
**Topic:** Quick Inspection

```json
# List all cat APIs
GET /_cat

# Indices
GET /_cat/indices?v

# Node information
GET /_cat/nodes?v

# Shards
GET /_cat/shards?v

# Aliases
GET /_cat/aliases?v

# Health
GET /_cat/health?v

# Count
GET /_cat/count/products?v

# Templates
GET /_cat/templates?v

# Segments
GET /_cat/segments/products?v

# Thread pools
GET /_cat/thread_pool?v
```

### Question 43: Explain API

**Difficulty:** Medium
**Topic:** Query Analysis

```json
# Explain why document matches
GET /products/_explain/1
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}

# Validate query
GET /products/_validate/query
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}

# Validate with explanation
GET /products/_validate/query?explain=true
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}
```

### Question 44: Profile API

**Difficulty:** Medium
**Topic:** Performance Analysis

```json
# Profile query performance
GET /products/_search
{
  "profile": true,
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" }}
      ],
      "filter": [
        { "range": { "price": { "gte": 1000 }}}
      ]
    }
  }
}

# Profile aggregations
GET /orders/_search
{
  "profile": true,
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category.keyword"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

### Question 45: Snapshot and Restore

**Difficulty:** Medium
**Topic:** Backup

```json
# Register snapshot repository (filesystem)
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/my_backup"
  }
}

# Create snapshot
PUT /_snapshot/my_backup/snapshot_1?wait_for_completion=true
{
  "indices": "products,orders",
  "ignore_unavailable": true,
  "include_global_state": false
}

# List snapshots
GET /_snapshot/my_backup/_all

# Get snapshot status
GET /_snapshot/my_backup/snapshot_1/_status

# Restore snapshot
POST /_snapshot/my_backup/snapshot_1/_restore
{
  "indices": "products",
  "ignore_unavailable": true,
  "include_global_state": false,
  "rename_pattern": "(.+)",
  "rename_replacement": "restored_$1"
}

# Delete snapshot
DELETE /_snapshot/my_backup/snapshot_1
```

### Question 46: Tasks API

**Difficulty:** Medium
**Topic:** Long-running Operations

```json
# List running tasks
GET /_tasks

# Get specific task
GET /_tasks/<task_id>

# Cancel task
POST /_tasks/<task_id>/_cancel

# List reindex tasks
GET /_tasks?actions=*reindex

# Reindex with task tracking
POST /_reindex?wait_for_completion=false
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}
```

### Question 47: Index Settings

**Difficulty:** Medium
**Topic:** Configuration Management

```json
# Get index settings
GET /products/_settings

# Update index settings (dynamic)
PUT /products/_settings
{
  "index": {
    "number_of_replicas": 2,
    "refresh_interval": "30s"
  }
}

# Close index (required for some settings changes)
POST /products/_close

# Update static settings
PUT /products/_settings
{
  "index": {
    "number_of_shards": 5
  }
}

# Reopen index
POST /products/_open

# Reset to default
PUT /products/_settings
{
  "index": {
    "refresh_interval": null
  }
}
```

---

## 🔵 SECTION 8: PRACTICAL PATTERNS (Questions 48-50)

### Question 48: Time-series Data Pattern

**Difficulty:** Hard
**Topic:** Logs/Metrics Storage

```json
# Index template for time-series data
PUT /_index_template/metrics_template
{
  "index_patterns": ["metrics-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "refresh_interval": "5s",
      "index.lifecycle.name": "metrics_policy"
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date"
        },
        "metric_name": {
          "type": "keyword"
        },
        "value": {
          "type": "double"
        },
        "host": {
          "type": "keyword"
        },
        "tags": {
          "type": "object"
        }
      }
    }
  }
}

# Index metrics with current timestamp pattern
POST /metrics-2024.01.15/_doc
{
  "timestamp": "2024-01-15T10:30:00Z",
  "metric_name": "cpu_usage",
  "value": 75.5,
  "host": "server-01",
  "tags": {
    "env": "production",
    "datacenter": "us-west-1"
  }
}

# Query recent metrics
GET /metrics-*/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "timestamp": {
              "gte": "now-1h"
            }
          }
        },
        {
          "term": {
            "metric_name": "cpu_usage"
          }
        }
      ]
    }
  },
  "sort": [
    { "timestamp": "desc" }
  ]
}

# Time-based aggregation
GET /metrics-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-24h"
      }
    }
  },
  "aggs": {
    "metrics_over_time": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1h"
      },
      "aggs": {
        "avg_value": {
          "avg": {
            "field": "value"
          }
        },
        "max_value": {
          "max": {
            "field": "value"
          }
        }
      }
    }
  }
}
```

### Question 49: E-commerce Search with Facets

**Difficulty:** Hard
**Topic:** Product Catalog

```json
# Create product catalog index
PUT /catalog
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "description": {
        "type": "text"
      },
      "category": {
        "type": "keyword"
      },
      "brand": {
        "type": "keyword"
      },
      "price": {
        "type": "double"
      },
      "rating": {
        "type": "float"
      },
      "in_stock": {
        "type": "boolean"
      },
      "tags": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date"
      }
    }
  }
}

# Search with faceted navigation
GET /catalog/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "wireless headphones",
            "fields": ["name^2", "description"],
            "fuzziness": "AUTO"
          }
        }
      ],
      "filter": [
        { "term": { "in_stock": true }},
        { "range": { "price": { "lte": 200 }}},
        { "range": { "rating": { "gte": 4 }}}
      ]
    }
  },
  "aggs": {
    "categories": {
      "terms": { "field": "category", "size": 10 }
    },
    "brands": {
      "terms": { "field": "brand", "size": 10 }
    },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "key": "Under $50", "to": 50 },
          { "key": "$50-$100", "from": 50, "to": 100 },
          { "key": "$100-$200", "from": 100, "to": 200 },
          { "key": "Over $200", "from": 200 }
        ]
      }
    },
    "avg_rating": {
      "avg": { "field": "rating" }
    }
  },
  "sort": [
    { "_score": "desc" },
    { "rating": "desc" },
    { "created_at": "desc" }
  ],
  "from": 0,
  "size": 20
}
```

### Question 50: Full-Text Search with Autocomplete

**Difficulty:** Hard
**Topic:** Search-as-you-type

```json
# Create index with search-as-you-type field
PUT /search_products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "search_as_you_type"
      },
      "name_suggest": {
        "type": "completion",
        "contexts": [
          {
            "name": "category",
            "type": "category"
          }
        ]
      },
      "category": {
        "type": "keyword"
      },
      "popularity": {
        "type": "integer"
      }
    }
  }
}

# Index products
POST /search_products/_doc/1
{
  "name": "Wireless Bluetooth Headphones",
  "name_suggest": {
    "input": ["Wireless Headphones", "Bluetooth Headphones", "Wireless Bluetooth"],
    "contexts": {
      "category": ["Electronics", "Audio"]
    }
  },
  "category": "Electronics",
  "popularity": 100
}

# Search-as-you-type query
GET /search_products/_search
{
  "query": {
    "multi_match": {
      "query": "wire head",
      "type": "bool_prefix",
      "fields": [
        "name",
        "name._2gram",
        "name._3gram"
      ]
    }
  }
}

# Completion suggester
POST /search_products/_search
{
  "suggest": {
    "product-suggest": {
      "prefix": "wire",
      "completion": {
        "field": "name_suggest",
        "size": 5,
        "contexts": {
          "category": "Electronics"
        },
        "skip_duplicates": true
      }
    }
  }
}

# Phrase suggester (spell correction)
POST /search_products/_search
{
  "suggest": {
    "text": "wireles headphon",
    "simple_phrase": {
      "phrase": {
        "field": "name",
        "size": 1,
        "gram_size": 2,
        "direct_generator": [
          {
            "field": "name",
            "suggest_mode": "always"
          }
        ]
      }
    }
  }
}
```

---

## 📝 Quick Reference - Elasticsearch Patterns

### Query Types Summary

```json
# Full-text queries (analyzed)
- match: Standard full-text search
- match_phrase: Exact phrase
- multi_match: Search across multiple fields
- query_string: Advanced query syntax

# Term-level queries (not analyzed)
- term: Exact match
- terms: Match any of values
- range: Numeric/date ranges
- exists: Field exists
- prefix: Starts with
- wildcard: Pattern matching

# Compound queries
- bool: must, should, must_not, filter
- boosting: Positive and negative boosting
- constant_score: Non-scored filter
```

### Aggregation Types Summary

```json
# Metrics
- avg, sum, min, max: Basic stats
- stats: All stats combined
- cardinality: Unique count
- percentiles: Distribution

# Buckets
- terms: Group by value
- range: Group by ranges
- date_histogram: Time buckets
- histogram: Numeric buckets
- nested: Nested objects
- filter: Filter bucket

# Pipeline
- moving_avg: Moving average
- derivative: Rate of change
- cumulative_sum: Running total
```

---

## 🎯 Practice Tips

1. **Use Kibana Dev Tools**: Much easier than curl for development
2. **Start with Mappings**: Design your schema before indexing
3. **Use Filters**: Leverage query caching with filter context
4. **Monitor Performance**: Use profile API to optimize queries
5. **Understand Analyzers**: Text analysis is key to good search
6. **Design for Scale**: Consider sharding and replication early

---

## 🚦 Learning Path

- **Week 1**: Questions 1-12 (Basics & Full-text Search)
- **Week 2**: Questions 13-26 (Aggregations & Mappings)
- **Week 3**: Questions 27-40 (Advanced Queries & Performance)
- **Week 4**: Questions 41-50 (Administration & Patterns)

---

_Remember: Elasticsearch excels at full-text search and analytics. Design your mappings carefully, use appropriate analyzers, and leverage the power of aggregations!_

**Happy Elasticsearch Querying! 🔍🚀**
