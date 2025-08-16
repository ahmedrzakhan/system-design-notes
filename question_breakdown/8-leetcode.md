# LeetCode (Online Judge) System Design Interview Guide

## 📌 Quick Overview
**LeetCode** is a platform for coding interview preparation with:
- Coding problems (easy to hard)
- Multi-language support
- Real-time code execution and feedback
- Competitive programming contests
- ~100K users, ~4000 problems (relatively small scale)

## 🎯 Requirements

### Functional Requirements (Core)
1. **View list of coding problems** (with pagination)
2. **View specific problem** and code solutions in multiple languages
3. **Submit solution** and get instant feedback (<5 seconds)
4. **View live leaderboard** for competitions

### Non-Functional Requirements
1. **Availability > Consistency**
2. **Security & Isolation** when running user code
3. **Low latency** - Return results within 5 seconds
4. **Scale** to support 100K concurrent users in competitions

### Out of Scope
- User authentication/profiles
- Payment processing
- Social features
- Analytics

## 🏗️ System Architecture

### High-Level Design

```mermaid
graph TB
    subgraph "Client Layer"
        C[Client<br/>Monaco IDE]
    end
    
    subgraph "API Layer"
        API[API Server]
    end
    
    subgraph "Data Layer"
        DB[(Primary DB<br/>DynamoDB)]
        Redis[(Redis<br/>Leaderboard)]
    end
    
    subgraph "Execution Layer"
        Queue[AWS SQS]
        Worker[Worker Service]
        subgraph "Docker Containers"
            Python[Python Runtime]
            Java[Java Runtime]
            JS[JavaScript Runtime]
        end
    end
    
    C -->|API Calls| API
    API --> DB
    API --> Redis
    API --> Queue
    Queue --> Worker
    Worker --> Python
    Worker --> Java
    Worker --> JS
    Worker -->|Update Results| DB
    Worker -->|Update Leaderboard| Redis
```

## 📊 Data Models

### Core Entities

```mermaid
erDiagram
    Problem {
        string id PK
        string title
        string question
        string level
        string[] tags
        json codeStubs
        json testCases
    }
    
    Submission {
        string id PK
        string userId
        string problemId FK
        string competitionId
        string code
        string language
        timestamp submittedAt
        json testResults
        int runtime
        string error
    }
    
    Leaderboard {
        string competitionId PK
        string userId
        int score
        timestamp completionTime
    }
    
    Problem ||--o{ Submission : "has many"
    Submission }o--|| Leaderboard : "updates"
```

### Problem Schema
```javascript
{
  id: string,
  title: string,
  question: string,
  level: "easy" | "medium" | "hard",
  tags: string[],
  codeStubs: {
    python: string,
    javascript: string,
    java: string,
    // ... other languages
  },
  testCases: [{
    type: string,     // "tree", "array", etc.
    input: string,    // Serialized input
    output: string    // Expected output
  }]
}
```

## 🔌 API Design

### Core Endpoints

| Method | Endpoint | Description | Response |
|--------|----------|-------------|----------|
| GET | `/problems` | List problems | `Partial<Problem>[]` |
| GET | `/problems/:id` | Get problem details | `Problem` |
| POST | `/problems/:id/submit` | Submit solution | `{submissionId: string}` |
| GET | `/check/:id` | Check submission status | `Submission` |
| GET | `/leaderboard/:competitionId` | Get leaderboard | `Leaderboard[]` |

### API Request/Response Examples

```javascript
// Submit solution
POST /problems/:id/submit
{
  code: string,
  language: "python" | "javascript" | "java"
}
// Note: userId from JWT/session, NOT from request body

// Check submission (polling)
GET /check/:submissionId
// Returns: { status: "processing" | "completed", results: {...} }
```

## 🔐 Code Execution Security

### ❌ Bad Approach: Run in API Server
- **Security Risk**: Malicious code can compromise server
- **Performance**: CPU intensive, can crash server
- **Isolation**: No isolation, affects all requests

### ✅ Good Approaches

#### 1. Virtual Machines (VMs)
- **Pros**: Strong isolation, full OS stack
- **Cons**: Resource intensive, slow startup, costly

#### 2. **Docker Containers** (Recommended)
- **Pros**: Lightweight, fast startup, good isolation
- **Cons**: Shares host kernel (needs proper config)

#### 3. Serverless Functions
- **Pros**: Auto-scaling, managed infrastructure
- **Cons**: Cold start latency, resource limits

### Container Security Measures

```mermaid
graph LR
    subgraph "Security Controls"
        A[Read-Only Filesystem]
        B[CPU/Memory Limits]
        C[5-Second Timeout]
        D[No Network Access]
        E[Seccomp Filters]
        F[VPC Controls]
    end
    
    Container[Docker Container] --> A
    Container --> B
    Container --> C
    Container --> D
    Container --> E
    Container --> F
```

**Implementation Details:**
- **Read-only filesystem** (writes to /tmp only)
- **Resource limits**: CPU and memory bounds
- **Timeout**: Kill process after 5 seconds
- **Network isolation**: VPC security groups
- **System calls**: Restricted via seccomp

## 📈 Scaling Strategies

### Problem: Handling 100K Concurrent Users

#### ❌ Vertical Scaling
- Limited by max instance size (128 cores on AWS X1)
- Expensive, underutilized during off-peak
- **Math**: 10K submissions × 100 test cases × 100ms = 27 hours on single core!

#### ✅ Horizontal Scaling with Queue

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Queue as SQS Queue
    participant Worker
    participant Container
    participant DB
    participant Redis
    
    Client->>API: POST /submit
    API->>Queue: Add submission
    API-->>Client: Return submissionId
    
    Worker->>Queue: Poll for jobs
    Queue-->>Worker: Get submission
    Worker->>Container: Execute code
    Container-->>Worker: Return results
    Worker->>DB: Store results
    Worker->>Redis: Update leaderboard
    
    loop Every 1 second
        Client->>API: GET /check/:id
        API->>DB: Check status
        DB-->>API: Return results
        API-->>Client: Status/Results
    end
```

**Benefits:**
- Handles traffic spikes
- Enables retries on failure
- Prevents container overload
- Auto-scaling with AWS ECS/Fargate

## 🏆 Leaderboard Implementation

### Evolution of Solutions

#### 1. ❌ Direct DB Queries
```sql
SELECT userId, COUNT(*) as score
FROM submissions
WHERE competitionId = :id AND passed = true
GROUP BY userId
ORDER BY score DESC
```
**Problem**: High DB load with frequent polling

#### 2. ✅ Redis with Periodic Updates
- Cache leaderboard for 30 seconds
- Reduce DB queries
**Problem**: Not real-time enough

#### 3. ✅✅ Redis Sorted Sets (Best)
```redis
# Update on submission
ZADD competition:leaderboard:{id} {score} {userId}

# Get top 100
ZREVRANGE competition:leaderboard:{id} 0 99 WITHSCORES
```
**Benefits**: O(log n) updates, O(m) retrieval, real-time

### Polling Strategy
- Default: Every 5 seconds
- Competition final minutes: Every 2 seconds
- Off-peak: Every 10 seconds

## 🧪 Test Case Execution

### Challenge: Language-Agnostic Test Cases

```mermaid
graph LR
    subgraph "Test Case Storage"
        TC[Serialized Test Cases<br/>Language Agnostic]
    end
    
    subgraph "Test Harnesses"
        PY[Python Harness]
        JS[JS Harness]
        JV[Java Harness]
    end
    
    TC --> PY
    TC --> JS
    TC --> JV
    
    PY --> PYR[Python Runtime]
    JS --> JSR[JS Runtime]
    JV --> JVR[Java Runtime]
```

### Example: Binary Tree Problem
```javascript
// Serialized test case (language-agnostic)
{
  "type": "tree",
  "input": [3, 9, 20, null, null, 15, 7],  // Inorder traversal
  "output": 3  // Max depth
}

// Each language has TreeNode deserializer
// Python: class TreeNode with deserialize method
// Java: TreeNode class with deserialize method
// JS: TreeNode constructor with deserialize function
```

## 🎯 Interview Level Expectations

### Mid-Level (IC4)
- **Focus**: 80% breadth, 20% depth
- **Expected**:
  - Clear API design and data models
  - Basic understanding of containers/VMs
  - Functional high-level design
  - Security awareness (isolation needed)

### Senior (IC5)
- **Focus**: 60% breadth, 40% depth
- **Expected**:
  - Detailed security implementation
  - Trade-off analysis (VM vs Container vs Serverless)
  - Performance calculations
  - Proactive problem identification
  - Test harness implementation details

### Staff+ (IC6+)
- **Expected**:
  - Drive entire conversation
  - Simple, scalable design (avoid over-engineering)
  - Clear scaling path
  - Production considerations
  - Operational excellence

## 💡 Key Insights & Tips

### 1. **Start Simple**
- Monolithic architecture is fine for small scale
- Don't over-engineer initially

### 2. **Security First**
- Never run user code directly on servers
- Always mention isolation requirements early

### 3. **Real-World Patterns**
- LeetCode actually uses polling for submissions
- WebSockets often overkill for this scale

### 4. **Performance Math**
- Always do back-of-envelope calculations
- 10K submissions × 100 tests × 100ms = Important!

### 5. **Progressive Enhancement**
```mermaid
graph LR
    A[Basic Design] --> B[Add Security]
    B --> C[Add Caching]
    C --> D[Add Queue]
    D --> E[Add Auto-scaling]
```

## 🚀 Advanced Considerations

### 1. **Multi-Region Deployment**
- CDN for problem statements
- Regional execution clusters
- Cross-region replication for competitions

### 2. **Optimization Techniques**
- Container pooling (warm containers)
- Test case result caching
- Incremental compilation

### 3. **Monitoring & Observability**
```mermaid
graph TB
    subgraph "Metrics to Track"
        A[Submission Latency]
        B[Container Utilization]
        C[Queue Depth]
        D[Error Rates]
        E[Leaderboard Accuracy]
    end
```

### 4. **Cost Optimization**
- Spot instances for containers
- Reserved capacity for predictable load
- Serverless for off-peak times

## 📝 Common Pitfalls to Avoid

1. **Don't pass userId in API requests** - Use JWT/session
2. **Don't mention localStorage** in artifacts - Not supported
3. **Don't over-complicate** for 100K users - It's relatively small
4. **Don't forget polling** for async operations
5. **Don't skip security discussion** - Critical for code execution

## 🔄 System Evolution Path

1. **MVP**: Simple monolith with Docker
2. **Scale**: Add queue and horizontal scaling
3. **Optimize**: Add caching and CDN
4. **Enterprise**: Multi-region, advanced monitoring

## 📚 Additional Resources

### Similar System Design Problems
- HackerRank
- CodeChef
- TopCoder Arena
- Google Code Jam infrastructure

### Related Patterns
- **Long-running tasks pattern**
- **Job queue pattern**
- **Real-time leaderboard pattern**
- **Sandboxed execution pattern**

---

## 🎓 Final Checklist for Interview

- [ ] Define clear requirements (functional & non-functional)
- [ ] Start with simple design, iterate
- [ ] Address security explicitly
- [ ] Do performance calculations
- [ ] Discuss trade-offs for each decision
- [ ] Consider scalability progressively
- [ ] Mention monitoring/operations
- [ ] Keep it practical, not theoretical

Remember: **The goal is to show systematic thinking, not to create the perfect system!**