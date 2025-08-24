# Facebook Live Comments System Design Interview Guide

## 📋 Problem Overview

Facebook Live Comments enables viewers to post and see comments on live video feeds in near real-time.

## 🎯 Requirements

### Functional Requirements

1. **Post Comments**: Viewers can post comments on a live video feed
2. **Real-time Updates**: Viewers see new comments in near real-time while watching
3. **Historical Comments**: Viewers can see comments posted before they joined

#### Below the Line (Out of Scope)

- Reply to comments
- React to comments

### Non-Functional Requirements

1. **Scale**: Support millions of concurrent videos, thousands of comments/second per video
2. **Availability > Consistency**: Eventual consistency is acceptable
3. **Low Latency**: < 200ms end-to-end latency for comment broadcast

#### Below the Line

- Security & authentication
- Content moderation/integrity

## 🏗️ High-Level Architecture

### Core Entities

- **User**: Viewer or broadcaster
- **Live Video**: The broadcast (managed by another team)
- **Comment**: Message posted by a user

### API Design

```http
POST /comments/:liveVideoId
Header: JWT | SessionToken
{
  "message": "Cool video!"
}

GET /comments/:liveVideoId?cursor={last_comment_id}&pageSize=10&sort=desc
```

### Basic Architecture Flow

```mermaid
graph LR
    A[Commenter Client] -->|HTTPS| B[Comment Management Service]
    B --> C[(Comments DB<br/>DynamoDB)]

    subgraph "Comment Schema"
        D[commentId - PK<br/>videoId - shard key<br/>content<br/>author<br/>createdAt - SK]
    end
```

## 🔄 Evolution of Real-time Updates

### ❌ Bad Solution: Polling

```mermaid
sequenceDiagram
    participant V as Viewer
    participant S as Server
    participant DB as Database

    loop Every few seconds
        V->>S: GET /comments/:videoId?since=lastId
        S->>DB: Query new comments
        DB-->>S: Return comments
        S-->>V: List of new comments
    end

    Note over V,DB: Heavy load with more viewers<br/>Not truly real-time
```

**Problems**:

- Doesn't scale (database overload)
- High latency (polling interval)
- Wasteful (most polls return nothing)

### ✅ Good Solution: Server-Sent Events (SSE)

```mermaid
graph TB
    A[Commenter Client] -->|HTTPS POST| B[Comment Management Service]
    B --> C[(Comments DB)]
    B -.->|SSE| D[Viewer Clients]

    subgraph "SSE Connection"
        E[One-way persistent connection<br/>Server pushes updates<br/>Client reconnects automatically]
    end
```

**Why SSE over WebSockets?**

- Read-heavy workload (most viewers don't comment)
- One-way communication is sufficient
- Lower overhead than WebSockets
- Auto-reconnection with Last-Event-ID header

## 📈 Scaling Strategies

### Challenge: Horizontal Scaling

When adding multiple servers, viewers of the same video may connect to different servers.

### Evolution of Solutions

#### 1️⃣ Bad: Simple Pub/Sub with Round-Robin LB

```mermaid
graph TB
    A[Commenter] -->|HTTPS| LB[Load Balancer<br/>Round Robin]
    LB --> CMS[Comment Service]
    CMS --> DB[(DynamoDB)]
    CMS --> PS[Redis Pub/Sub<br/>ALL Comments]

    LB -->|SSE| RMS1[Realtime Server 1]
    LB -->|SSE| RMS2[Realtime Server 2]
    LB -->|SSE| RMS3[Realtime Server 3]

    PS -.->|Subscribe| RMS1
    PS -.->|Subscribe| RMS2
    PS -.->|Subscribe| RMS3

    style PS fill:#5A0D16
```

**Problem**: Every server processes EVERY comment (inefficient at scale)

#### 2️⃣ Better: Partitioned Pub/Sub

```mermaid
graph TB
    A[Commenter] -->|HTTPS| LB[Load Balancer]
    LB --> CMS[Comment Service]
    CMS --> DB[(DynamoDB)]

    CMS --> PS[Redis Pub/Sub]

    subgraph "Partitioned Topics"
        T1[Topic: Video1]
        T2[Topic: Video2]
        T3[Topic: Video3]
    end

    PS --> T1
    PS --> T2
    PS --> T3

    RMS1[Server 1] -.->|Subscribe| T1
    RMS2[Server 2] -.->|Subscribe| T2
    RMS3[Server 3] -.->|Subscribe| T3
```

**Improvement**: Servers only subscribe to relevant topics
**Problem**: Round-robin LB may still scatter viewers across servers

#### 3️⃣ Great: Layer 7 LB with Consistent Hashing

```mermaid
graph TB
    A[Commenter] -->|HTTPS| LB["Layer 7 LB<br/>Consistent Hashing"]
    LB --> CMS[Comment Service]
    CMS --> DB[(DynamoDB)]

    subgraph "Intelligent Routing"
        LB -->|Video1 viewers| RMS1[Server 1]
        LB -->|Video2 viewers| RMS2[Server 2]
        LB -->|Video3 viewers| RMS3[Server 3]
    end

    CMS --> PS[Partitioned Pub/Sub]
    PS -.-> RMS1
    PS -.-> RMS2
    PS -.-> RMS3

    style LB fill:#5A0D16
```

**Benefits**:

- Viewers of same video routed to same server
- Fewer topic subscriptions per server
- Better resource utilization

#### 4️⃣ Alternative: Dispatcher Service

```mermaid
graph TB
    A[Commenter] -->|HTTPS| LB[Layer 7 LB]
    LB --> CMS[Comment Service]
    CMS --> DB[(DynamoDB)]

    CMS -->|New Comment| DS[Dispatcher Service]

    subgraph "Direct Routing"
        DS -->|HTTPS| RMS1[Server 1<br/>Video1 viewers]
        DS -->|HTTPS| RMS2[Server 2<br/>Video2 viewers]
        DS -->|HTTPS| RMS3[Server 3<br/>Video3 viewers]
    end

    DS -.->|Consults| ZK[Zookeeper<br/>Server Mapping]

    style DS fill:#5A0D16
```

**Benefits**:

- No pub/sub overhead
- Direct targeted delivery
- Dynamic server registration

## 🎨 Pagination Strategy

### ❌ Offset Pagination

```sql
SELECT * FROM comments
WHERE videoId = ?
LIMIT 10 OFFSET 20
```

**Problems**:

- Inefficient for large datasets
- Unstable (new comments shift results)

### ✅ Cursor Pagination

```sql
SELECT * FROM comments
WHERE videoId = ? AND commentId < lastCommentId
ORDER BY commentId DESC
LIMIT 10
```

**Benefits**:

- Efficient with indexes
- Stable pagination
- Works well with DynamoDB's LastEvaluatedKey

## 💡 Key Design Decisions

### Database Choice: DynamoDB

- **Partition Key**: videoId (for sharding)
- **Sort Key**: createdAt (for time-based queries)
- Handles high write throughput
- Built-in pagination support

### Real-time Technology: SSE

- Perfect for read-heavy workload
- Lower overhead than WebSockets
- Browser auto-reconnection
- Works over standard HTTP

### Scaling Pattern: Pub/Sub with Co-location

- Partition by videoId
- Route viewers to same server
- Minimize cross-server communication

## 📊 Performance Targets

| Metric             | Target    | Strategy                         |
| ------------------ | --------- | -------------------------------- |
| Latency            | < 200ms   | SSE push model                   |
| Concurrent Viewers | Millions  | Horizontal scaling               |
| Comments/sec       | Thousands | DynamoDB sharding                |
| Availability       | 99.9%     | Multiple servers, auto-reconnect |

## 🎯 Interview Level Expectations

### Mid-Level (L4)

- Identify polling limitations
- Propose push-based solution (SSE/WebSocket)
- Basic pub/sub understanding
- Simple horizontal scaling

### Senior (L5)

- Quickly design initial architecture
- Understand pub/sub trade-offs
- Discuss scaling strategies
- Consider pagination approaches

### Staff+ (L6+)

- Deep dive into scaling challenges
- Multiple solution alternatives
- Technology-specific trade-offs
- Production-ready considerations

## 🔧 Additional Considerations

### Connection Management

```javascript
// Server-side connection tracking
const viewerConnections = {
  liveVideoId1: [sseConnection1, sseConnection2],
  liveVideoId2: [sseConnection3, sseConnection4],
};
```

### Channel Partitioning

```javascript
// Determine pub/sub channel
const channel = `video:${hash(liveVideoId) % NUM_CHANNELS}`;
```

### Reconnection Handling

```javascript
// Client-side SSE reconnection
const eventSource = new EventSource("/comments/stream");
eventSource.addEventListener("error", () => {
  // Browser auto-reconnects with Last-Event-ID
});
```

## 🚀 Production Enhancements

1. **Rate Limiting**: Prevent comment spam
2. **Caching**: Cache recent comments in Redis
3. **CDN**: Serve static assets globally
4. **Monitoring**: Track latency, connection count
5. **Graceful Degradation**: Fallback to polling if SSE fails
6. **Content Moderation**: ML-based filtering pipeline
7. **Analytics**: Track engagement metrics

## 📝 Common Interview Questions

1. **"Why not use WebSockets?"**

   - Read-heavy workload doesn't need bidirectional
   - SSE has lower overhead
   - Simpler implementation

2. **"How do you handle server failures?"**

   - SSE auto-reconnection
   - Last-Event-ID for resuming
   - Multiple server replicas

3. **"What about message ordering?"**

   - Use timestamps for ordering
   - DynamoDB sort key ensures consistency
   - Client-side deduplication

4. **"How to scale beyond single datacenter?"**
   - Regional deployments
   - Cross-region replication
   - Edge servers for SSE connections

## 🎓 Key Takeaways

1. **Start Simple**: Begin with basic architecture, iterate
2. **Consider Trade-offs**: Each solution has pros/cons
3. **Scale Incrementally**: Don't over-engineer initially
4. **Focus on Requirements**: Let NFRs guide decisions
5. **Real-world Experience**: Draw from production systems

## 📚 Technologies to Know

- **Streaming**: SSE, WebSockets, Long Polling
- **Pub/Sub**: Redis, Kafka, RabbitMQ
- **Databases**: DynamoDB, Cassandra, MongoDB
- **Load Balancing**: Layer 4 vs Layer 7, Consistent Hashing
- **Coordination**: Zookeeper, etcd, Consul
- **Caching**: Redis, Memcached, CDN strategies

Here's a concise explanation of the Facebook Live Comments system design:

## The Core Problem

You need to build a system where millions of viewers can post and see comments on live videos in real-time (< 200ms latency).

## Key Design Decisions

**1. Basic Architecture**

- Comments stored in DynamoDB (partitioned by videoId, sorted by timestamp)
- Comment Management Service handles creates/reads
- Real-time delivery to viewers is the main challenge

**2. Real-time Delivery Evolution**

**Polling (Bad):** Clients ask server every few seconds for new comments

- Problem: Doesn't scale, wastes resources, high latency

**Server-Sent Events (Good):** Server pushes comments to clients

- One-way persistent HTTP connection
- Perfect for read-heavy workload (most viewers don't comment)
- Auto-reconnection with Last-Event-ID

**3. Scaling Challenge**
When you have multiple servers, viewers of the same video might connect to different servers. How does Server A know to send a new comment to viewers on Server B?

**Solutions (Progressive Complexity):**

a) **Simple Pub/Sub:** All servers subscribe to ALL comments

- Problem: Every server processes every comment (wasteful)

b) **Partitioned Pub/Sub:** Create topics per video (or hash videos into N topics)

- Better: Servers only subscribe to relevant topics
- Problem: Round-robin LB still scatters viewers randomly

c) **Layer 7 LB + Pub/Sub:** Route viewers of same video to same server

- Use consistent hashing on videoId
- Minimizes topic subscriptions per server

d) **Dispatcher Service:** Direct routing without pub/sub

- Dispatcher tracks which server handles which video
- Routes comments directly to correct servers

**4. Other Important Aspects**

- Use cursor pagination (not offset) for loading historical comments
- SSE chosen over WebSockets because it's simpler for one-way communication
- DynamoDB works well for simple comment storage with high throughput

The key insight is recognizing this is a **fan-out problem** - one comment must reach potentially millions of viewers efficiently. The solution evolves from simple but inefficient to complex but scalable.

# Facebook Live Comments - Last Minute Revision Guide

## 🎯 The Core Problem

- Build real-time commenting system for live videos
- Scale: Millions of viewers, thousands of comments/sec per video
- Latency: < 200ms end-to-end
- **Key Challenge**: Fan-out problem - one comment to millions of viewers

## 📊 Requirements Summary

- **Functional**: Post comments, real-time updates, historical comments
- **Non-Functional**: Scale > consistency, low latency, high availability
- **Out of Scope**: Replies, reactions, moderation, auth

## 🏗️ Basic Architecture

- **Database**: DynamoDB with videoId (partition key) + createdAt (sort key)
- **API**: POST/GET endpoints with cursor pagination
- **Schema**: commentId, videoId, content, author, createdAt

## 🔄 Real-time Solutions Evolution

### ❌ Polling (Bad)

- Clients ask server every few seconds for new comments
- **Problems**: Database overload, high latency, wasteful

### ✅ Server-Sent Events (Good)

- Server pushes comments to clients via persistent HTTP connection
- **Why SSE over WebSocket**: Read-heavy workload, one-way communication, lower overhead
- **Benefits**: Auto-reconnection with Last-Event-ID header

## 📈 Horizontal Scaling Challenge

When multiple servers exist, viewers of same video might connect to different servers - how does Server A notify viewers on Server B?

### Solution Evolution:

#### 1️⃣ Simple Pub/Sub (Inefficient)

- All servers subscribe to ALL comments
- **Problem**: Every server processes every comment

#### 2️⃣ Partitioned Pub/Sub (Better)

- Create topics per video or hash videos into N topics
- **Problem**: Round-robin LB still scatters viewers randomly

#### 3️⃣ Layer 7 LB + Consistent Hashing (Great)

- Route viewers of same video to same server
- Servers only subscribe to relevant topics
- **Best balance** of efficiency and simplicity

#### 4️⃣ Dispatcher Service (Alternative)

- Direct routing without pub/sub overhead
- Dispatcher tracks server-to-video mapping
- Uses service discovery (Zookeeper/etcd)

## 💡 Key Technology Decisions

### Database: DynamoDB

- **Why**: High write throughput, auto-sharding, built-in pagination
- **Schema**: Partition by videoId for distribution, sort by timestamp

### Real-time: Server-Sent Events

- **Why**: Perfect for read-heavy, one-way communication
- **Benefits**: Browser support, auto-reconnection, HTTP-based

### Pagination: Cursor-based

- **Why**: Efficient with indexes, stable results
- **Implementation**: Use commentId < lastCommentId with LIMIT

## 🎨 Architecture Patterns

- **Fan-out on Write**: Comment service publishes to pub/sub
- **Co-location**: Route same video viewers to same server
- **Eventual Consistency**: Acceptable for comment ordering

## 📊 Performance Targets

- **Latency**: < 200ms (push model)
- **Scale**: Millions concurrent viewers
- **Throughput**: Thousands comments/sec
- **Availability**: 99.9% (multiple servers, auto-reconnect)

## 🎯 Interview Level Expectations

### Mid-Level (L4)

- Identify polling problems
- Propose SSE/WebSocket
- Basic pub/sub understanding

### Senior (L5)

- Design complete architecture
- Discuss scaling trade-offs
- Consider pagination approaches

### Staff+ (L6+)

- Multiple scaling solutions
- Deep technology trade-offs
- Production considerations

## 🔧 Common Interview Questions & Answers

### "Why SSE over WebSockets?"

- Read-heavy workload doesn't need bidirectional
- Lower overhead and simpler implementation
- Built-in auto-reconnection

### "How handle server failures?"

- SSE auto-reconnects with Last-Event-ID
- Multiple server replicas
- Graceful degradation to polling

### "What about message ordering?"

- Use timestamps for client-side ordering
- DynamoDB sort key ensures consistency
- Client deduplication for reliability

### "How scale beyond single datacenter?"

- Regional deployments with replication
- Edge servers for SSE connections
- Cross-region pub/sub topics

## 🚀 Production Enhancements

- **Rate Limiting**: Prevent spam
- **Caching**: Redis for recent comments
- **CDN**: Global static asset delivery
- **Monitoring**: Latency/connection tracking
- **Content Moderation**: ML filtering pipeline

## 🎓 Key Takeaways

1. **Fan-out Problem**: One comment → millions of viewers efficiently
2. **Technology Choice**: SSE perfect for read-heavy, one-way communication
3. **Scaling Strategy**: Co-locate viewers + partitioned pub/sub
4. **Database Design**: Partition by videoId, sort by timestamp
5. **Iteration**: Start simple, scale incrementally based on bottlenecks

## 🔥 Must-Remember Points

- **Core Challenge**: Real-time fan-out at massive scale
- **Solution**: SSE + Layer 7 LB + Partitioned Pub/Sub
- **Database**: DynamoDB with smart partitioning
- **Pagination**: Cursor-based for stability
- **Trade-off**: Eventual consistency for availability/performance
