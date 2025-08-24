# Facebook Post Search System Design - Interview Guide

## 📋 Problem Overview

Design a search system for Facebook posts that allows users to search by keywords and sort results by recency or like count.

**Key Constraint**: Cannot use pre-built search engines (Elasticsearch) or full-text indexes (Postgres FTS).

## 🎯 Requirements

### Functional Requirements

1. Users can create and like posts
2. Users can search posts by keyword
3. Search results can be sorted by:
   - Recency (time)
   - Like count

### Non-Functional Requirements

1. **Fast queries**: < 500ms median response time
2. **High volume**: Handle 10k searches/second
3. **Fresh results**: New posts searchable within 1 minute
4. **High recall**: All posts discoverable (even old/unpopular)
5. **High availability**

### Out of Scope

- Fuzzy matching
- Personalization
- Advanced relevance ranking
- Privacy rules
- Real-time updates on search page

## 📊 Scale Estimations

### Key Numbers

- **Users**: 1B
- **Posts created**: 10k/second (1B posts/day)
- **Likes created**: 100k/second (10 likes per user/day)
- **Searches**: 10k/second
- **Total posts**: 3.6 trillion (10 years)
- **Storage**: 3.6 PB raw data

**Important Insight**: System is **write-heavy** (100k likes/sec vs 10k searches/sec)

## 🏗️ High-Level Architecture

```mermaid
graph TB
    subgraph "Write Path"
        PS[Post Service]
        LS[Like Service]
        LB[Load Balancer]
        EW[Event Writer]
        K[Kafka]
        LBatch[Like Batcher]
        IS[Ingestion Service]
    end

    subgraph "Read Path"
        Client[Client]
        CDN[CDN]
        AG[API Gateway]
        SS[Search Service]
        SC[Search Cache<br/>Redis]
    end

    subgraph "Storage"
        RI[Redis Index<br/>Hot Data]
        CS[Cold Storage<br/>S3/Blob]
    end

    PS --> LB
    LS --> LB
    LB --> EW
    EW --> K
    K --> LBatch
    LBatch --> IS
    IS --> RI
    IS --> CS

    Client --> CDN
    CDN --> AG
    AG --> SS
    SS --> SC
    SS --> RI
    SS --> CS

    style RI fill:#5A0D16
    style SC fill:#1E4C50
    style K fill:#5A0D16
```

## 🔑 Core Design Decisions

### 1. Inverted Index Structure

**Problem**: Cannot scan 3.6T posts for each search query.

**Solution**: Use inverted index mapping keywords to post IDs.

```mermaid
graph LR
    subgraph "Inverted Index"
        K1[Keyword: 'Taylor'] --> P1[PostIDs: 1, 5, 10, 22, 45...]
        K2[Keyword: 'Swift'] --> P2[PostIDs: 2, 5, 13, 45, 67...]
        K3[Keyword: 'Concert'] --> P3[PostIDs: 3, 10, 22, 89...]
    end
```

### 2. Multiple Indexes for Sorting

**Two separate indexes**:

1. **Creation Index**: Redis List (append-only, query from tail)
2. **Likes Index**: Redis Sorted Set (score-based ordering)

```python
# Creation Index (Redis List)
RPUSH "keyword:taylor:creation" post_id

# Likes Index (Redis Sorted Set)
ZADD "keyword:taylor:likes" like_count post_id
```

### 3. Write Optimization Strategies

#### Batch Processing for Likes

- Aggregate likes over 30-second windows
- Reduces write volume for viral posts

#### Logarithmic Updates

- Only update index at milestones (1, 2, 4, 8, 16... likes)
- Reduces writes exponentially
- Trade-off: Index becomes approximately correct

#### Two-Stage Architecture

1. **Stage 1**: Query approximate index (fast)
2. **Stage 2**: Re-rank top N\*2 results with fresh data

```mermaid
sequenceDiagram
    participant C as Client
    participant SS as Search Service
    participant RI as Redis Index
    participant LS as Like Service

    C->>SS: Search "Taylor" sorted by likes
    SS->>RI: Get top 2N posts
    RI-->>SS: [Approximate results]
    SS->>LS: Get current like counts
    LS-->>SS: [Fresh counts]
    SS->>SS: Re-sort with fresh data
    SS-->>C: Top N results
```

## 🚀 Optimization Techniques

### 1. Caching Strategy

**Multi-layer caching**:

1. **CDN (Edge Cache)**: Geographic distribution, 10ms latency
2. **Redis Search Cache**: Application-level, < 1 min TTL
3. **No personalization** = High cache hit rate

### 2. Multi-keyword Query Handling

#### Option A: Intersection + Filter

```python
# For query "Taylor Swift"
taylor_posts = get_posts("taylor")
swift_posts = get_posts("swift")
candidates = intersection(taylor_posts, swift_posts)
# Filter for actual phrase match
results = filter_phrase_match(candidates, "Taylor Swift")
```

#### Option B: Bigrams/Shingles

- Pre-index word pairs: "Taylor Swift", "Swift Concert"
- Larger index size but faster queries
- Trade-off: Storage vs. Speed

### 3. Storage Optimization

**Hot/Cold Data Separation**:

- **Hot (Redis)**: Frequently searched keywords, recent posts
- **Cold (S3/Blob)**: Rarely searched, old posts
- Migration based on access patterns

**Index Capping**:

- Limit each keyword index to 1k-10k items
- Reduces storage by orders of magnitude

## 🎓 Interview Tips by Level

### Mid-Level Focus

- Clear API design and data model
- Basic inverted index understanding
- Simple caching strategy
- Acknowledge bottlenecks

### Senior-Level Focus

- Multiple index optimization
- Write volume handling
- Cache layering strategy
- Trade-off analysis

### Staff+ Level Focus

- Two-stage architecture
- Logarithmic update strategies
- Cold/hot data separation
- Real-world experience examples

## 💡 Additional Considerations

### Data Consistency

- **Eventual consistency** acceptable (1 min delay)
- Like counts can be approximate in index
- Final results should be accurate

### Failure Handling

- Kafka for reliable event processing
- Redis replication for index durability
- Fallback to cold storage if Redis fails

### Monitoring & Metrics

- Query latency percentiles (p50, p95, p99)
- Cache hit rates at each layer
- Write throughput to indexes
- Storage growth rate

### Security Considerations

- Rate limiting at API Gateway
- Input sanitization for search queries
- DDoS protection at CDN level

## 🔄 System Evolution Path

1. **Phase 1**: Basic inverted index with single Redis
2. **Phase 2**: Add caching layers (CDN + Redis cache)
3. **Phase 3**: Implement batching for likes
4. **Phase 4**: Two-stage architecture with re-ranking
5. **Phase 5**: Hot/cold separation for storage optimization

## ❓ Common Interview Questions

1. **"How do you handle stop words?"**

   - Filter common words ("the", "and") during tokenization
   - Reduces index size significantly

2. **"What about search relevance?"**

   - Out of scope for basic design
   - Could add: TF-IDF scoring, user engagement signals

3. **"How to handle index updates?"**

   - Async processing via Kafka
   - Batch updates for efficiency
   - Eventually consistent model

4. **"What if Redis goes down?"**

   - Fallback to cold storage (slower)
   - Redis replication for HA
   - Circuit breaker pattern

5. **"How to scale internationally?"**
   - Geographic sharding
   - Regional CDN deployment
   - Cross-region replication

## 📝 Key Takeaways

1. **Inverted index** is fundamental for text search at scale
2. **Write optimization** crucial for high-volume systems
3. **Approximate correctness** acceptable for performance
4. **Caching** dramatically improves read performance
5. **Hot/cold separation** optimizes storage costs

## 🔗 Related Topics to Study

- Redis data structures (Lists, Sorted Sets, HyperLogLog)
- Consistent hashing for sharding
- Bloom filters for existence checks
- Count-min sketch for frequency estimation
- Vector databases for semantic search
- Apache Lucene internals

Here's a brief explanation of the Facebook Post Search system design:

## The Core Problem

You need to search through 3.6 trillion posts (3.6 PB of data) and return results in under 500ms without using Elasticsearch or built-in database full-text search.

## The Solution: Inverted Index

Instead of scanning all posts for each search, you build a reverse mapping:

- **Traditional**: Post → Contains words ["taylor", "swift", "concert"]
- **Inverted**: "taylor" → Points to posts [1, 5, 10, 22...]

This is stored in Redis (in-memory) for speed.

## Key Design Elements

**1. Two Indexes for Sorting**

- Creation time index (Redis List) - for recency
- Likes index (Redis Sorted Set) - for popularity

**2. Handle 100k likes/second**

- Don't update index on every like
- Only update at milestones (1, 2, 4, 8, 16... likes)
- Or batch updates every 30 seconds

**3. Two-Stage Architecture**

- Stage 1: Get approximate top results from index (fast)
- Stage 2: Fetch real-time like counts and re-rank (accurate)

**4. Caching Strategy**

- CDN at edge (10ms response)
- Redis cache for search results (<1 min TTL)
- Works well because no personalization needed

**5. Storage Optimization**

- Hot data (frequently searched) in Redis
- Cold data (rarely searched) in S3/blob storage
- Cap each keyword index at 10k items max

## The Critical Insight

This is a **write-heavy** system (100k likes/sec vs 10k searches/sec). Most of the complexity comes from optimizing writes, not reads. That's why batching and approximate indexing are crucial - you trade perfect accuracy for massive performance gains.

The interviewer wants to see if you recognize this pattern and can design around it rather than just throwing more servers at the problem.
