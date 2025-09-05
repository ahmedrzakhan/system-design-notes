# System Design Core Concepts Guide

_Essential building blocks for technical interviews_

## 🏗️ Scaling Fundamentals

### Vertical vs Horizontal Scaling

```mermaid
graph TD
    A[Scaling Options] --> B[Vertical Scaling]
    A --> C[Horizontal Scaling]

    B --> B1[Add More CPU/RAM/Storage]
    B --> B2[Single Machine]
    B --> B3[Less Complexity]
    B --> B4[Limited by Hardware]

    C --> C1[Add More Machines]
    C --> C2[Distributed System]
    C --> C3[More Complexity]
    C --> C4[Nearly Unlimited Scale]

    style B fill:#5A0D16
    style C fill:#1E4C50
```

**Key Insights**:

- **Vertical scaling** often requires less complexity - prefer when possible
- Many systems can scale vertically to surprising degrees
- **Don't throw machines at poor design** - fix the design first
- Horizontal scaling introduces distribution challenges

### Horizontal Scaling Challenges

```mermaid
flowchart TD
    A[Horizontal Scaling] --> B[Work Distribution]
    A --> C[Data Distribution]
    A --> D[State Management]

    B --> B1[Load Balancers]
    B --> B2[Consistent Hashing]
    B --> B3[Queue Systems]

    C --> C1[Data Partitioning]
    C --> C2[Shared Databases]
    C --> C3[Geographic Distribution]

    D --> D1[Synchronization]
    D --> D2[Consistency]
    D --> D3[Race Conditions]

    style A fill:#5A0D16
    style B fill:#1E4C50
    style C fill:#5A0D16
    style D fill:#1E4C50
```

---

## ⚖️ Work & Data Distribution

### Work Distribution Strategies

| Strategy               | Use Case                    | Pros                      | Cons                    |
| ---------------------- | --------------------------- | ------------------------- | ----------------------- |
| **Round Robin**        | Simple load balancing       | Even distribution, simple | Ignores server capacity |
| **Least Connections**  | Variable request complexity | Considers server load     | More complex tracking   |
| **Consistent Hashing** | Caching, data distribution  | Minimal redistribution    | Complex implementation  |
| **Geographic**         | Global applications         | Low latency for users     | Data synchronization    |

### Consistent Hashing Deep Dive

```mermaid
graph LR
    A[Hash Ring] --> B[Node A at 0°]
    A --> C[Node B at 120°]
    A --> D[Node C at 240°]

    E[Request Hash] --> F{Find Next Node Clockwise}
    F --> G[Route to That Node]

    H[Add/Remove Node] --> I[Minimal Data Movement]

    style A fill:#5A0D16
    style I fill:#1E4C50
```

### Anti-Patterns to Avoid

```mermaid
flowchart TD
    A[Scatter-Gather Pattern] --> B[High Network Traffic]
    A --> C[Tail Latency Issues]
    A --> D[Failure Sensitivity]

    E[Hot Spot Problem] --> F[Uneven Load Distribution]
    E --> G[One Node 90% Busy]
    E --> H[Others 10% Busy]

    style A fill:#1E4C50
    style E fill:#5A0D16
```

**Better Approaches**:

- **Partition by region** for geographic systems
- **Fan-out to few nodes** instead of many
- **Cache hot data** to reduce database load

---

## 🔄 CAP Theorem

### The Trade-off Triangle

```mermaid
graph TD
    A[CAP Theorem] --> B[Consistency]
    A --> C[Availability]
    A --> D[Partition Tolerance]

    B --> B1[All nodes see same data]
    B --> B2[Strong guarantees]
    B --> B3[May become unavailable]

    C --> C1[System always responds]
    C --> C2[May return stale data]
    C --> C3[Eventually consistent]

    D --> D1[Works despite network failures]
    D --> D2[Required in distributed systems]

    E[In Practice] --> F[Choose: CP or AP]
    F --> G[Usually choose AP]

    style D fill:#1E4C50
    style G fill:#5A0D16
```

### When to Choose Each Model

```mermaid
flowchart LR
    A[Consistency Requirements] --> B{Can Tolerate Stale Data?}

    B -->|Yes| C[Choose Availability]
    B -->|No| D[Choose Consistency]

    C --> C1[Social Media Feeds]
    C --> C2[Product Catalogs]
    C --> C3[User Profiles]

    D --> D1[Banking Systems]
    D --> D2[Inventory Management]
    D --> D3[Booking Systems]

    style C fill:#5A0D16
    style D fill:#1E4C50
```

**Interview Default**: Choose **Availability** unless strong consistency is explicitly required

### Mixed Consistency Models

**Key Insight**: Different features can have different consistency requirements

- **Product descriptions**: Eventually consistent
- **Inventory counts**: Strongly consistent
- **User comments**: Eventually consistent
- **Order processing**: Strongly consistent

---

## 🔒 Locking & Concurrency

### Locking Considerations

```mermaid
mindmap
  root((Locking Strategy))
    Granularity
      Table Level
      Row Level
      Field Level
      Application Level
    Duration
      Transaction Scope
      Request Scope
      Batch Operation
      Long Running
    Bypass Options
      Optimistic Locking
      Compare and Swap
      Read-Only Operations
      Eventual Consistency
```

### Optimistic vs Pessimistic Locking

```mermaid
sequenceDiagram
    participant C1 as Client 1
    participant C2 as Client 2
    participant DB as Database

    Note over C1,DB: Optimistic Locking
    C1->>DB: Read data + version
    C2->>DB: Read data + version
    C1->>DB: Update if version matches
    DB-->>C1: Success
    C2->>DB: Update if version matches
    DB-->>C2: Conflict - retry

    Note over C1,DB: Pessimistic Locking
    C1->>DB: Lock + Read data
    C2->>DB: Try to lock (blocked)
    C1->>DB: Update + Release lock
    C2->>DB: Now can proceed
```

**When to Use Each**:

- **Optimistic**: Low contention, read-heavy workloads
- **Pessimistic**: High contention, critical consistency needs

---

## 📊 Indexing Strategies

### Index Types & Use Cases

```mermaid
graph TD
    A[Indexing Strategies] --> B[Hash Index]
    A --> C[B-Tree Index]
    A --> D[Specialized Indexes]

    B --> B1["O(1) lookup by key"]
    B --> B2[Perfect for exact matches]

    C --> C1["O(log n) range queries"]
    C --> C2[Sorted data access]

    D --> D1[Geospatial]
    D --> D2[Full-text Search]
    D --> D3[Vector/ML]

    style B fill:#1E4C50
    style C fill:#5A0D16
    style D fill:#1E4C50
```

### Database Indexing Best Practices

```mermaid
flowchart TD
    A[Indexing Decision] --> B{Primary Database Supports?}
    B -->|Yes| C[Use Built-in Indexes]
    B -->|No| D[External Index System]

    C --> C1[PostgreSQL + PostGIS]
    C --> C2[MySQL + Full-text]
    C --> C3[DynamoDB + GSI]

    D --> D1[ElasticSearch]
    D --> D2[Redis]
    D --> D3[Custom Solution]

    style C fill:#5A0D16
    style D fill:#1E4C50
```

### ElasticSearch as Secondary Index

```mermaid
graph LR
    A[Primary Database] --> B[Change Data Capture]
    B --> C[ElasticSearch Cluster]
    C --> D[Search Queries]

    E[Considerations] --> F[Data Staleness]
    E --> G[Additional Failure Point]
    E --> H[Extra Latency]

    style A fill:#1E4C50
    style C fill:#5A0D16
    style E fill:#1E4C50
```

**ElasticSearch Capabilities**:

- **Full-text search** via Lucene
- **Geospatial indexing** for location data
- **Vector indexing** for ML similarity
- **CDC integration** with most databases

---

## 🌐 Communication Protocols

### Protocol Selection Matrix

```mermaid
graph TD
    A[Communication Needs] --> B{Real-time Required?}

    B -->|No| C[HTTP/HTTPS]
    B -->|Yes| D{Bidirectional?}

    D -->|No| E[Server-Sent Events]
    D -->|Yes| F[WebSockets]

    G[Alternative] --> H[Long Polling]

    C --> C1[REST APIs]
    C --> C2[Simple Request/Response]
    C --> C3[Stateless & Scalable]

    E --> E1[Server to Client Push]
    E --> E2[HTTP Infrastructure Compatible]

    F --> F1[Real-time Chat]
    F --> F2[Gaming]
    F --> F3[Requires Special Infrastructure]

    H --> H1[Near Real-time]
    H --> H2[Standard Load Balancers]

    style C fill:#1E4C50
    style E fill:#5A0D16
    style F fill:#1E4C50
    style H fill:#5A0D16
```

### Internal vs External Protocols

| Context                | Recommended        | Alternative  | Use Case                    |
| ---------------------- | ------------------ | ------------ | --------------------------- |
| **Internal Services**  | HTTP/HTTPS         | gRPC         | Microservices communication |
| **Simple APIs**        | HTTP/HTTPS         | -            | CRUD operations             |
| **Real-time Updates**  | Server-Sent Events | Long Polling | Live feeds, notifications   |
| **Bidirectional Chat** | WebSockets         | -            | Real-time collaboration     |

### WebSocket Architecture Pattern

```mermaid
graph TD
    A[Client] --> B[Load Balancer]
    B --> C[WebSocket Gateway]
    C --> D[Message Broker]
    D --> E[Backend Services]

    F[Benefits] --> G[Stateless Backend]
    F --> H[Horizontal Scaling]
    F --> I[Connection Management]

    style C fill:#5A0D16
    style D fill:#1E4C50
    style F fill:#5A0D16
```

---

## 🔐 Security Essentials

### Multi-Layer Security Approach

```mermaid
graph TD
    A[Security Layers] --> B[Authentication/Authorization]
    A --> C[Encryption]
    A --> D[Data Protection]

    B --> B1[API Gateway]
    B --> B2[Auth0/OAuth]
    B --> B3[User-Specific Access]

    C --> C1[HTTPS/TLS in Transit]
    C --> C2[Database Encryption at Rest]
    C --> C3[User-Controlled Keys]

    D --> D1[Rate Limiting]
    D --> D2[Request Throttling]
    D --> D3[Input Validation]

    style B fill:#1E4C50
    style C fill:#5A0D16
    style D fill:#1E4C50
```

### Security Best Practices

```mermaid
flowchart LR
    A[Security Implementation] --> B[Delegate to Specialists]
    B --> C[API Gateway for Auth]
    B --> D[Managed Services]

    E[Encryption Strategy] --> F[Data in Transit]
    E --> G[Data at Rest]
    E --> H[User-Controlled Keys]

    I[Data Protection] --> J[Rate Limiting]
    I --> K[Access Logging]
    I --> L[Input Sanitization]

    style B fill:#1E4C50
    style E fill:#5A0D16
    style I fill:#1E4C50
```

**Key Principles**:

- **Don't reinvent security** - use proven solutions
- **Defense in depth** - multiple security layers
- **User control** for sensitive data encryption
- **Monitor and log** all access patterns

---

## 📈 Monitoring & Observability

### Three Pillars of Monitoring

```mermaid
graph TD
    A[Monitoring Strategy] --> B[Infrastructure Level]
    A --> C[Service Level]
    A --> D[Application Level]

    B --> B1[CPU/Memory/Disk Usage]
    B --> B2[Network Performance]
    B --> B3[Hardware Health]
    B --> B4[Tools: Datadog, New Relic]

    C --> C1[Request Latency]
    C --> C2[Error Rates]
    C --> C3[Throughput/QPS]
    C --> C4[Service Dependencies]

    D --> D1[User Sessions]
    D --> D2[Business Metrics]
    D --> D3[Feature Usage]
    D --> D4[Tools: Analytics, Mixpanel]

    style B fill:#1E4C50
    style C fill:#1E4C50
    style D fill:#5A0D16
```

### Monitoring Metrics Matrix

| Level              | Key Metrics              | Alert Thresholds   | Tools               |
| ------------------ | ------------------------ | ------------------ | ------------------- |
| **Infrastructure** | CPU, Memory, Disk        | >80% usage         | Datadog, Prometheus |
| **Service**        | Latency, Error Rate, QPS | >500ms, >1% errors | APM tools           |
| **Application**    | DAU, Conversion, Revenue | Business-specific  | Analytics platforms |

### Leading vs Lagging Indicators

```mermaid
flowchart LR
    A[Monitoring Types] --> B[Leading Indicators]
    A --> C[Lagging Indicators]

    B --> B1[Disk Space Growth]
    B --> B2[Memory Leaks]
    B --> B3[Connection Pool Usage]

    C --> C1[Service Downtime]
    C --> C2[User Complaints]
    C --> C3[Revenue Impact]

    D[Goal] --> E[Catch Problems Early]
    E --> F[Leading Indicators]

    style B fill:#1E4C50
    style C fill:#5A0D16
    style F fill:#1E4C50
```

---

## 🔧 Design Patterns & Anti-Patterns

### Common Architecture Patterns

```mermaid
graph TD
    A[Design Patterns] --> B[Microservices]
    A --> C[Event-Driven]
    A --> D[CQRS]
    A --> E[Circuit Breaker]

    B --> B1[Service Independence]
    B --> B2[Technology Diversity]
    B --> B3[Team Autonomy]

    C --> C1[Pub/Sub Messaging]
    C --> C2[Loose Coupling]
    C --> C3[Event Sourcing]

    D --> D1[Separate Read/Write Models]
    D --> D2[Optimized Queries]

    E --> E1[Failure Isolation]
    E --> E2[Graceful Degradation]

    style B fill:#1E4C50
    style C fill:#5A0D16
    style D fill:#1E4C50
    style E fill:#5A0D16
```

### Anti-Patterns to Avoid

```mermaid
flowchart TD
    A[Common Anti-Patterns] --> B[Premature Optimization]
    A --> C[Distributed Monolith]
    A --> D[Chatty Interfaces]
    A --> E[Shared Database]

    B --> B1[Over-engineering Early]
    B --> B2[Complex Before Simple]

    C --> C1[Services Too Coupled]
    C --> C2[Synchronous Dependencies]

    D --> D1[Too Many API Calls]
    D --> D2[N+1 Query Problem]

    E --> E1[Tight Data Coupling]
    E --> E2[Schema Change Conflicts]

    style A fill:#1E4C50
    style B fill:#5A0D16
    style C fill:#1E4C50
    style D fill:#5A0D16
    style E fill:#1E4C50
```

---

## 🎯 Interview Application Strategy

### Concept Selection Framework

```mermaid
flowchart TD
    A[System Design Problem] --> B{Scale Requirements?}
    B -->|Large Scale| C[Horizontal Scaling + CAP]
    B -->|Medium Scale| D[Vertical Scaling OK]

    C --> E{Real-time Features?}
    E -->|Yes| F[WebSockets/SSE + Messaging]
    E -->|No| G[HTTP APIs + Databases]

    H{Search Required?} --> I[Indexing Strategy]
    I --> J[ElasticSearch or DB indexes]

    K{Sensitive Data?} --> L[Security Layer]
    L --> M[Auth + Encryption + Rate Limiting]

    style C fill:#1E4C50
    style F fill:#5A0D16
    style I fill:#1E4C50
    style L fill:#5A0D16
```

### When to Apply Each Concept

| System Type         | Key Concepts                              | Focus Areas                        |
| ------------------- | ----------------------------------------- | ---------------------------------- |
| **Social Media**    | Horizontal scaling, CAP (AP), Indexing    | Feed generation, Content delivery  |
| **E-commerce**      | CAP (CP for inventory), Locking, Security | Inventory management, Payments     |
| **Chat/Gaming**     | WebSockets, Message brokers, Real-time    | Low latency, Connection management |
| **Search Engine**   | Indexing, ElasticSearch, Distributed data | Query performance, Relevance       |
| **Video Streaming** | CDN, Geographic distribution, Bandwidth   | Content delivery, Global scale     |

### Interview Progression Strategy

```mermaid
gantt
    title Concept Introduction Timeline
    dateFormat X
    axisFormat %s

    section Early Design
    Basic Architecture :done, arch, 0, 10

    section Scale Discussion
    Horizontal Scaling :done, scale, 10, 15
    CAP Theorem Choice :done, cap, 15, 18

    section Deep Dives
    Indexing Strategy :done, index, 18, 25
    Communication Protocols :done, comm, 25, 30
    Security Considerations :done, sec, 30, 35

    section Optional
    Monitoring Setup :done, mon, 35, 40
```

---

## 📋 Quick Reference Cheat Sheet

### Decision Trees for Common Choices

#### Scaling Decision

```
Small/Medium Scale → Vertical Scaling
Large Scale + Simple → Horizontal + Load Balancer
Large Scale + Complex → Horizontal + Consistent Hashing
```

#### Consistency Decision

```
Banking/Inventory/Booking → Strong Consistency (CP)
Social Media/Content/Profiles → Eventual Consistency (AP)
Mixed System → Per-feature decision
```

#### Communication Decision

```
Simple CRUD → HTTP/REST
Real-time Updates → Server-Sent Events
Bidirectional Real-time → WebSockets + Message Broker
```

#### Indexing Decision

```
Primary DB supports needed indexes → Use built-in
Complex search requirements → ElasticSearch
Simple key-value lookups → Redis/Hash indexes
```

### Common Technology Combinations

| Use Case         | Database   | Cache | Search        | Communication |
| ---------------- | ---------- | ----- | ------------- | ------------- |
| **Social Media** | PostgreSQL | Redis | ElasticSearch | HTTP + SSE    |
| **E-commerce**   | MySQL      | Redis | ElasticSearch | HTTP          |
| **Chat App**     | PostgreSQL | Redis | -             | WebSockets    |
| **Analytics**    | Cassandra  | Redis | ElasticSearch | HTTP          |

---

## 💡 Key Takeaways

### Essential Principles

- **Start simple, scale incrementally** - don't over-engineer early
- **Choose availability over consistency** unless explicitly required
- **Prefer vertical scaling** when possible for simplicity
- **Use battle-tested solutions** over custom implementations
- **Consider the entire system** - not just individual components

### Interview Success Tips

- **Justify your choices** - explain why you picked each technology
- **Consider trade-offs** - acknowledge limitations and alternatives
- **Think about operations** - monitoring, debugging, maintenance
- **Scale appropriately** - match complexity to actual requirements
- **Communicate clearly** - walk through your reasoning process

### Red Flags to Avoid

- Jumping to complex solutions without justification
- Ignoring consistency requirements for critical data
- Over-engineering for unrealistic scale
- Forgetting about monitoring and security
- Not considering failure scenarios

Remember: **System design is about making informed trade-offs, not memorizing perfect architectures.**
