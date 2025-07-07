# System Design Interview: Networking Essentials

## 🎯 Interview Focus
- **Infrastructure/Distributed Systems**: Deep networking knowledge required
- **Full-stack/Product**: Surface understanding sufficient
- **Universal**: Understanding fundamentals helps make better architectural decisions

---

## 📚 Core Networking Layers

### OSI Model Overview
```mermaid
graph TD
    A[Application Layer - Layer 7] --> B[Transport Layer - Layer 4]
    B --> C[Network Layer - Layer 3]
    C --> D[Lower Layers 1-2]
    
    A1[HTTP, HTTPS, DNS, WebSockets, gRPC] --> A
    B1[TCP, UDP, QUIC] --> B
    C1[IP, Routing, Addressing] --> C
    D1[Physical, Data Link] --> D
```

### Key Layers for Interviews
1. **Network Layer (Layer 3)**: IP protocol, routing, addressing
2. **Transport Layer (Layer 4)**: TCP, UDP, QUIC - reliability and flow control
3. **Application Layer (Layer 7)**: HTTP, WebSockets, gRPC - application protocols

---

## 🌐 HTTP Request Flow Example

```mermaid
sequenceDiagram
    participant Client
    participant DNS
    participant Server
    
    Client->>DNS: 1. Resolve hellointerview.com
    DNS-->>Client: 2. Return IP (32.42.52.62)
    
    Note over Client,Server: TCP Handshake
    Client->>Server: 3. SYN
    Server-->>Client: 4. SYN-ACK
    Client->>Server: 5. ACK
    
    Note over Client,Server: HTTP Communication
    Client->>Server: 6. HTTP GET Request
    Server-->>Client: 7. HTTP Response
    
    Note over Client,Server: TCP Teardown
    Client->>Server: 8. FIN
    Server-->>Client: 9. ACK
    Server->>Client: 10. FIN
    Client-->>Server: 11. ACK
```

**Key Takeaways:**
- Multiple packets for one conceptual request/response
- Stateful connections require maintenance
- Each step adds latency
- Higher stack = more processing overhead

---

## 🚀 Transport Layer Protocols

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented | Connectionless |
| **Reliability** | Guaranteed delivery | Best-effort |
| **Ordering** | Maintains order | No ordering guarantees |
| **Flow Control** | Yes | No |
| **Header Size** | 20-60 bytes | 8 bytes |
| **Speed** | Slower (overhead) | Faster |
| **Use Cases** | Most applications | Streaming, gaming, VoIP |

### When to Choose UDP
```mermaid
flowchart TD
    A[Need Low Latency?] -->|Yes| B[Data Loss Acceptable?]
    A -->|No| F[Use TCP]
    B -->|Yes| C[High Volume Data?]
    B -->|No| F
    C -->|Yes| D[Browser Support Needed?]
    C -->|No| F
    D -->|No| E[Consider UDP]
    D -->|Yes| G[Use TCP/HTTP with optimizations]
    E --> H[Examples: Gaming, VoIP, Live Streaming]
```

**UDP Interview Examples:**
- Live video streaming
- Online gaming
- VoIP calls
- DNS lookups
- Real-time telemetry

---

## 📡 Application Layer Protocols

### Protocol Selection Matrix

```mermaid
graph LR
    A[API Requirements] --> B{Public vs Internal?}
    
    B -->|Public| C{Real-time needed?}
    B -->|Internal| D{Performance critical?}
    
    C -->|No| E[REST/HTTP]
    C -->|Server Push| F[SSE]
    C -->|Bidirectional| G[WebSockets]
    C -->|P2P Audio/Video| H[WebRTC]
    
    D -->|Yes| I[gRPC]
    D -->|No| E
    
    E --> J[JSON over HTTP]
    F --> K[Server-Sent Events]
    G --> L[Real-time Apps]
    H --> M[Video Calling]
    I --> N[Binary Protocol]
```

### REST API Design Patterns

**Core Principles:**
- Resources, not operations
- HTTP verbs for actions
- Stateless design

```http
GET    /users/{id}           → Retrieve user
POST   /users               → Create user
PUT    /users/{id}          → Update user
DELETE /users/{id}          → Delete user
GET    /users/{id}/posts    → Get user's posts
```

**Status Codes to Know:**
- **2xx Success**: 200 OK, 201 Created
- **3xx Redirect**: 301 Permanent, 302 Temporary
- **4xx Client Error**: 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Rate Limited
- **5xx Server Error**: 500 Internal Error, 502 Bad Gateway

### GraphQL vs REST

```mermaid
graph TD
    A[API Design Choice] --> B{Frontend Flexibility Needed?}
    B -->|High| C{Multiple Teams?}
    B -->|Low| D[Use REST]
    C -->|Yes| E[Consider GraphQL]
    C -->|No| F{Complex Nested Data?}
    F -->|Yes| E
    F -->|No| D
    
    E --> G[Benefits: Flexible queries, single endpoint]
    D --> H[Benefits: Simple, cacheable, well-understood]
```

### Real-time Communication Patterns

```mermaid
graph TD
    A[Real-time Requirement] --> B{Direction of Communication}
    
    B -->|Server → Client| C[Server-Sent Events - SSE]
    B -->|Bidirectional| D{Frequency}
    
    D -->|High Frequency| E[WebSockets]
    D -->|Low Frequency| F[Long Polling]
    
    C --> G[Use Cases: Live updates, notifications, feeds]
    E --> H[Use Cases: Chat, gaming, collaborative editing]
    F --> I[Use Cases: Simple notifications]
```

**SSE Limitations:**
- Connection timeouts require reconnection
- One-way communication only
- Some networks buffer responses

**WebSocket Considerations:**
- Stateful connections (scaling complexity)
- Infrastructure support required
- Overkill for simple push notifications

---

## ⚖️ Load Balancing

### Scaling Strategies

```mermaid
graph LR
    A[Scaling Options] --> B[Vertical Scaling]
    A --> C[Horizontal Scaling]
    
    B --> D[Bigger Servers]
    C --> E[More Servers]
    
    E --> F[Need Load Balancer]
    D --> G[Simpler Architecture]
```

### Load Balancer Types

```mermaid
graph TD
    A[Load Balancing] --> B[Client-Side]
    A --> C[Server-Side]
    
    B --> D[Service Discovery]
    B --> E[DNS Round Robin]
    
    C --> F[Layer 4 - L4]
    C --> G[Layer 7 - L7]
    
    F --> H[TCP/UDP Level]
    G --> I[HTTP Level]
    
    H --> J[Fast, Connection-based]
    I --> K[Content-aware routing]
```

### Layer 4 vs Layer 7 Comparison

| Aspect | Layer 4 (L4) | Layer 7 (L7) |
|--------|--------------|--------------|
| **Operates At** | Transport Layer | Application Layer |
| **Routing Based On** | IP, Port | URL, Headers, Content |
| **Connection Handling** | Persistent TCP | Terminates & recreates |
| **Performance** | Faster | More CPU intensive |
| **Features** | Basic load distribution | Content routing, SSL termination |
| **Best For** | WebSockets, raw performance | HTTP APIs, content-based routing |

### Load Balancing Algorithms

```mermaid
graph LR
    A[LB Algorithms] --> B[Round Robin]
    A --> C[Random]
    A --> D[Least Connections]
    A --> E[Least Response Time]
    A --> F[IP Hash]
    
    B --> G[Sequential distribution]
    C --> H[Random distribution]
    D --> I[Good for persistent connections]
    E --> J[Performance-based]
    F --> K[Session persistence]
```

**Interview Recommendations:**
- **Default**: Round Robin or Random
- **WebSockets/SSE**: Least Connections
- **Session-dependent**: IP Hash

---

## 🌍 Global Distribution & CDNs

### Latency Challenges

```mermaid
graph TD
    A[Global Users] --> B[Physics Problem]
    B --> C[Speed of Light: ~200,000 km/s in fiber]
    B --> D[NY ↔ London: ~56ms minimum]
    
    E[Solutions] --> F[CDNs]
    E --> G[Regional Partitioning]
    E --> H[Edge Computing]
    
    F --> I[Cache static content globally]
    G --> J[Partition data by geography]
    H --> K[Process data closer to users]
```

### CDN Strategy

```mermaid
graph LR
    A[User Request] --> B{Content in CDN?}
    B -->|Hit| C[Return from Edge]
    B -->|Miss| D[Fetch from Origin]
    D --> E[Cache at Edge]
    E --> F[Return to User]
    
    C --> G[Low Latency]
    F --> H[Higher Latency - First Time]
```

**CDN Use Cases:**
- Static assets (images, CSS, JS)
- Video streaming
- API response caching
- Search results caching

---

## 🛡️ Failure Handling Patterns

### Retry Strategy

```mermaid
graph TD
    A[Request Fails] --> B[Retry with Exponential Backoff]
    B --> C[Wait: 100ms, 200ms, 400ms, 800ms...]
    C --> D[Add Jitter]
    D --> E[Randomize timing to prevent thundering herd]
    
    F[Requirements] --> G[Idempotent APIs]
    F --> H[Timeout Configuration]
    F --> I[Maximum Retry Limit]
```

### Circuit Breaker Pattern

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: Failure threshold exceeded
    Open --> HalfOpen: Timeout period elapsed
    HalfOpen --> Closed: Test request succeeds
    HalfOpen --> Open: Test request fails
    
    Closed: Normal operation\nAll requests allowed
    Open: Failing fast\nNo requests allowed
    HalfOpen: Testing recovery\nLimited requests allowed
```

**Circuit Breaker Benefits:**
- Fail fast instead of hanging
- Reduce load on struggling services
- Automatic recovery testing
- Prevent cascading failures

### Idempotency Keys

```http
POST /payments
{
  "amount": 1000,
  "currency": "USD",
  "idempotency_key": "user123_2025-01-01"
}
```

**Implementation:**
- Store processed idempotency keys
- Return same result for duplicate requests
- Prevent double charges/operations

---

## 🎯 Interview Decision Framework

### Protocol Selection Guide

```mermaid
flowchart TD
    A[Start] --> B{Real-time needed?}
    B -->|No| C[REST/HTTP]
    B -->|Server Push Only| D[Server-Sent Events]
    B -->|Bidirectional| E{High Frequency?}
    E -->|Yes| F[WebSockets]
    E -->|No| G[Long Polling]
    
    H{Internal vs External?} --> I[Internal]
    H --> J[External]
    I --> K{Performance Critical?}
    K -->|Yes| L[gRPC]
    K -->|No| C
    J --> C
```

### Load Balancer Selection

```mermaid
flowchart TD
    A[Need Load Balancing?] --> B{Connection Type}
    B -->|Persistent - WebSockets| C[Layer 4 LB]
    B -->|HTTP/REST| D[Layer 7 LB]
    
    E{Client Control?} --> F[Many Uncontrolled Clients]
    E --> G[Few Controlled Clients]
    F --> H[Dedicated Load Balancer]
    G --> I[Client-Side Load Balancing]
```

---

## 📝 Common Interview Scenarios

### Scenario 1: Chat Application
**Requirements**: Real-time bidirectional messaging
```
Choice: WebSockets
Load Balancer: Layer 4
Fallback: Long polling for browsers with issues
```

### Scenario 2: Live Sports Updates
**Requirements**: Server pushes score updates
```
Choice: Server-Sent Events
Load Balancer: Layer 7
CDN: Cache team info, player stats
```

### Scenario 3: Microservices Communication
**Requirements**: Internal service-to-service calls
```
Choice: gRPC for performance, REST for simplicity
Load Balancing: Client-side with service discovery
Circuit Breakers: Essential for fault tolerance
```

### Scenario 4: Global Video Streaming
**Requirements**: Low latency worldwide
```
Protocols: HTTP for metadata, CDN for content
Distribution: Regional CDNs
Optimization: Edge caching, adaptive bitrate
```

---

## 🚨 Red Flags to Avoid

### ❌ Don't Do This:
- Use WebSockets for simple notifications
- Choose UDP without justifying browser support
- Ignore failure modes and cascading failures
- Over-engineer with gRPC for public APIs
- Forget about connection state with load balancers

### ✅ Do This:
- Start with simple defaults (TCP, REST, Layer 7 LB)
- Justify protocol choices based on requirements
- Consider failure scenarios and mitigation
- Think about global distribution early
- Mention monitoring and observability

---

## 🎤 Key Interview Talking Points

### Impressive Details to Mention:
1. **"Retry with exponential backoff and jitter"** - Shows production experience
2. **Circuit breaker pattern** - Demonstrates fault tolerance thinking
3. **Client-side load balancing for microservices** - Advanced architecture knowledge
4. **Physics constraints** - Understanding of fundamental limitations
5. **Idempotency keys** - API design best practices

### Questions to Ask:
- "What's the expected global user distribution?"
- "How real-time do the updates need to be?"
- "Are there any specific latency requirements?"
- "Should we optimize for read or write performance?"

---

## 📚 Study Recommendations

### Hands-on Learning:
1. **Wireshark**: Capture and analyze network traffic
2. **Network Link Conditioner**: Simulate poor network conditions
3. **Build a simple chat app**: Experience WebSocket challenges
4. **Set up a load balancer**: Use NGINX or HAProxy

### Advanced Topics for Senior Interviews:
- HTTP/2 and HTTP/3 benefits
- QUIC protocol advantages
- BGP routing basics
- DNS optimization strategies
- TLS/SSL performance considerations

---

*Remember: In interviews, start with simple, well-understood solutions and optimize based on specific requirements. Most problems don't need the most advanced protocols—reliability and simplicity often win over premature optimization.*