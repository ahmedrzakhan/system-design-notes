# Strong vs. Eventual Consistency for System Design Interviews

In distributed systems, data consistency models define how and when updates become visible across replicated nodes. This guide explains **Strong Consistency** and **Eventual Consistency**, their mechanics, trade-offs, use cases, and client-centric variations, tailored for system design interviews. It includes Mermaid diagrams and practical examples to help you articulate answers confidently.

## What is Data Consistency?

In distributed systems, data is replicated across multiple nodes for availability, fault tolerance, and low latency. Consistency models govern:
- **When** updates are visible.
- **Where** updates appear (across clients or replicas).
- **Order** of operations seen by the system.

The key question: *If I write data, when and how will others see it?*

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Write| B[Node A]
    B -->|Replicate| C[Node B]
    B -->|Replicate| D[Node C]
    A -->|Read| C
    A -->|Read| D
```

## Strong Consistency

**What it is**: Guarantees that after a write is confirmed, all subsequent reads from any client or node return the latest value, behaving like a single, synchronized data copy.

**How it Works**:
1. Client sends a write to the primary node (e.g., Node A).
2. Node A propagates the write to replicas (e.g., Nodes B, C).
3. Replicas apply the write and send acknowledgments.
4. Write is confirmed only after all (or a quorum of) replicas agree, using consensus protocols like Paxos or Raft.
5. All reads now return the updated value.

**Diagram**:
```mermaid
sequenceDiagram
    participant Client
    participant Node_A
    participant Node_B
    participant Node_C

    Client->>Node_A: Write Request
    Node_A->>Node_B: Replicate Write
    Node_A->>Node_C: Replicate Write
    Node_B-->>Node_A: Ack
    Node_C-->>Node_A: Ack
    Node_A-->>Client: Write Confirmed
    Client->>Node_B: Read Request
    Node_B-->>Client: Latest Value
```

**Pros**:
- **Simpler Logic**: No stale data or conflict resolution needed.
- **Predictable**: Reads always reflect the latest writes.
- **Data Integrity**: Ensures absolute correctness.

**Cons**:
- **Latency**: Coordination across nodes increases write/read times.
- **Lower Availability**: May reject requests during network partitions (CAP theorem).
- **Complex Infrastructure**: Requires sophisticated consensus mechanisms.

**When to Use**:
- **Banking/Financial Systems**: Accurate balances (e.g., money transfers).
- **Inventory Management**: Prevent overselling (e.g., e-commerce stock).
- **Distributed Locking**: Ensure exclusive locks.
- **Unique ID Generation**: Avoid duplicates across nodes.

**Example**: In a banking app, transferring $100 from Account A to B must update both balances atomically to avoid errors.

## Eventual Consistency

**What it is**: Guarantees that, without new updates, all replicas will *eventually* converge to the same value, allowing temporary inconsistencies during replication lag.

**How it Works**:
1. Client sends a write to a node (e.g., Node A).
2. Node A acknowledges the write immediately.
3. The update propagates asynchronously to other replicas (e.g., Nodes B, C).
4. Reads from unsynced replicas may return stale data until propagation completes.

**Diagram**:
```mermaid
sequenceDiagram
    participant Client
    participant Node_A
    participant Node_B
    participant Node_C

    Client->>Node_A: Write Request
    Node_A-->>Client: Write Ack
    Node_A->>Node_B: Async Replicate
    Node_A->>Node_C: Async Replicate
    Client->>Node_B: Read Request
    Node_B-->>Client: Stale Data
    Note over Node_B: Not yet synced
    Node_B->>Node_B: Apply Update
    Client->>Node_B: Read Request
    Node_B-->>Client: Latest Data
```

**Pros**:
- **Low Latency**: Fast writes without waiting for replicas.
- **High Availability**: Operates during network partitions.
- **Scalable**: Ideal for geo-distributed, high-traffic systems.

**Cons**:
- **Stale Data**: Reads may return outdated values temporarily.
- **Complex Logic**: Applications must handle inconsistencies and conflicts.
- **Conflict Resolution**: Requires strategies like Last Write Wins (LWW) or CRDTs.

**When to Use**:
- **Social Media Metrics**: Likes or view counts (e.g., brief delays are acceptable).
- **Analytics**: Page visits or click tracking.
- **Recommendation Systems**: Suggestions (e.g., product or movie recommendations).
- **DNS**: Cached records propagate gradually.
- **CDNs**: Static assets like images or scripts.
- **Shopping Carts**: Merging items added from different devices.

**Example**: Updating a profile picture on a social media platform may show the old image to some users briefly until all replicas sync.

## Client-Centric Consistency Variations

Eventual consistency can be enhanced with client-centric models to improve user experience:

- **Causal Consistency**: Operations with causal relationships (e.g., comment → reply) are seen in order.
  - Example: Replies appear after the original comment.
- **Read-Your-Writes**: A client’s reads reflect their own writes.
  - Example: After updating your bio, you see it immediately on refresh.
- **Monotonic Reads**: Future reads by a client never return older data.
  - Example: If you see 10 likes, later reads show 10 or more, never fewer.
- **Monotonic Writes**: A client’s writes are applied in order.
  - Example: Posting “Hello” then “World” ensures “Hello” appears first.

**Diagram**:
```mermaid
graph TD
    A[Client] -->|Write| B[Node A]
    B -->|Async Replicate| C[Node B]
    A -->|Read-Your-Writes| B
    A -->|Monotonic Read| C
    Note over A: Sees own writes immediately
    Note over C: May lag but never regresses
```

## Choosing the Right Model

Selecting a consistency model depends on your application’s needs:

1. **Data Criticality**:
   - **High**: Use strong consistency (e.g., financial transactions).
   - **Low**: Use eventual consistency (e.g., social media likes).

2. **User Experience**:
   - Use optimistic UI updates or loading indicators to mask eventual consistency delays.
   - Strong consistency reduces user confusion but may slow responses.

3. **Performance (Latency)**:
   - Eventual consistency offers faster writes/reads.
   - Strong consistency adds coordination overhead.

4. **Availability**:
   - Eventual consistency prioritizes availability during partitions (CAP theorem).
   - Strong consistency may reject requests to ensure correctness.

5. **Scalability**:
   - Eventual consistency scales better for read-heavy, geo-distributed systems.
   - Strong consistency limits scaling due to coordination.

6. **Development Complexity**:
   - Strong consistency simplifies app logic (latest state guaranteed).
   - Eventual consistency requires handling stale data and conflicts (e.g., LWW, CRDTs).

**Diagram**:
```mermaid
graph TD
    A[Application Needs] --> B[Data Criticality]
    A --> C[User Experience]
    A --> D[Performance]
    A --> E[Availability]
    A --> F[Scalability]
    A --> G[Complexity]
    B -->|High| H[Strong Consistency]
    B -->|Low| I[Eventual Consistency]
    C --> H
    C --> I
    D --> I
    E --> I
    F --> I
    G --> H
```

## Interview Tips

- **Explain Trade-offs**: Discuss latency vs. correctness, availability vs. consistency (CAP theorem).
- **Use Examples**: Reference real-world systems (e.g., Cassandra for eventual consistency in social media, MySQL for strong consistency in banking).
- **Address Conflicts**: For eventual consistency, mention LWW or CRDTs for conflict resolution.
- **Know Protocols**: Mention Paxos/Raft for strong consistency; async replication for eventual consistency.
- **Tailor to Use Case**: Justify your choice based on the system’s requirements (e.g., low latency for analytics, correctness for transactions).