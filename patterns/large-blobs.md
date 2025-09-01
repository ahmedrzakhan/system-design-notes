# Handling Large Blobs Pattern - System Design Interview Guide

## Core Concept

**Key Rule**: If you're moving files larger than **100MB** through your API, immediately think of this pattern. For files over **10MB**, consider if direct uploads would benefit your system.

## The Problem

### Traditional Approach (Anti-pattern)

```mermaid
graph LR
    Client -->|2GB Video| API[API Server]
    API -->|2GB Video| Storage[Blob Storage]
    Storage -->|2GB Video| API
    API -->|2GB Video| Client2[Client Download]

    style API fill:#5A0D16
```

**Issues with proxy approach:**

- Servers become bottlenecks (dumb pipes)
- Wastes bandwidth and compute resources
- Increases latency
- Limited by server's network capacity
- No resumability for large uploads
- Memory/CPU constraints on servers

## The Solution: Direct Upload Pattern

### Simple Direct Upload Architecture

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Server
    participant S3 as Blob Storage
    participant DB as Database

    C->>API: Request upload permission
    API->>DB: Create metadata record (status: pending)
    API->>C: Return presigned URL
    C->>S3: Direct upload (PUT)
    S3->>API: Event notification
    API->>DB: Update status to completed
```

### Key Components

#### 1. Presigned URLs

- Temporary, scoped credentials for direct storage access
- Generated in-memory (no network call needed)
- Typical expiry: 15 minutes to 1 hour
- Include restrictions:
  - `content-length-range`: Min/max file sizes
  - `content-type`: Enforce file types
  - Specific storage location

**Example S3 Presigned URL Structure:**

```
https://mybucket.s3.amazonaws.com/uploads/user123/video.mp4
?X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20240115%2Fus-east-1%2Fs3
&X-Amz-Expires=900
&X-Amz-Signature=b2754f5b1c9d7c4b8d4f6e9a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

#### 2. Direct Downloads via CDN

```mermaid
graph TB
    Client -->|Request| CDN[CDN Edge]
    CDN -->|Cache Miss| Storage[Blob Storage]
    CDN -->|Cached Content| Client

    CDN2[CDN Edge 2] -.->|Already Cached| Client2[Client 2]

    style CDN fill:#5A0D16
    style CDN2 fill:#1E4C50
```

## Resumable Uploads for Large Files

### Multipart Upload Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Server
    participant S3 as S3 Storage

    C->>API: Request multipart upload
    API->>C: Return upload ID + presigned URLs

    loop For each chunk
        C->>S3: Upload chunk (5MB-5GB)
        S3->>C: Return ETag
    end

    Note over C: Connection fails at 60%

    C->>S3: List uploaded parts
    S3->>C: Parts 1-60 complete
    C->>S3: Resume from part 61

    C->>S3: Complete multipart upload
    S3->>API: Event notification
```

### Chunk Size Guidelines

- **AWS S3**: 5MB - 5GB per part
- **Google Cloud**: Flexible chunk sizes
- **Azure**: 4MB - 100MB blocks

## State Synchronization

### Database Schema

```sql
CREATE TABLE files (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    filename VARCHAR(255),
    size_bytes BIGINT,
    content_type VARCHAR(100),
    storage_key VARCHAR(500), -- s3://bucket/user123/files/abc-123.pdf
    status VARCHAR(50), -- 'pending', 'uploading', 'completed', 'failed'
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

### Event-Driven Architecture

```mermaid
graph TB
    subgraph Upload Flow
        Client -->|Direct Upload| Storage[Blob Storage]
        Storage -->|S3 Event| SNS[SNS/EventBridge]
        SNS -->|Trigger| Lambda[Lambda/Worker]
        Lambda -->|Update Status| DB[(Database)]
    end

    subgraph Reconciliation
        Cron[Periodic Job] -->|Check Pending Files| DB
        Cron -->|Verify in Storage| Storage
        Cron -->|Update Missing| DB
    end
```

## Common Interview Scenarios

### When to Use This Pattern

| Scenario                | Example               | Implementation                                   |
| ----------------------- | --------------------- | ------------------------------------------------ |
| **Video Platforms**     | YouTube, TikTok       | Multipart uploads → S3 → Transcoding pipeline    |
| **Photo Sharing**       | Instagram, Flickr     | Direct upload → S3 events → Thumbnail generation |
| **File Storage**        | Dropbox, Google Drive | Chunked uploads with sync notifications          |
| **Messaging**           | WhatsApp media        | Presigned URLs with expiry for privacy           |
| **Document Management** | DocuSign, SharePoint  | Upload → Virus scan → Processing                 |

### When NOT to Use

1. **Small files (<10MB)**: Overhead not worth it
2. **Synchronous validation needed**: Must inspect data during upload
3. **Compliance requirements**: Data must pass through certified systems
4. **Immediate processing**: Real-time face detection, instant previews

## Deep Dive Topics

### 1. Handling Upload Failures

```mermaid
stateDiagram-v2
    [*] --> Uploading: Start Upload
    Uploading --> Failed: Connection Lost
    Failed --> Resuming: Query Completed Parts
    Resuming --> Uploading: Continue from Last Part
    Uploading --> Completed: All Parts Done
    Completed --> [*]
```

**Solution**: Use multipart uploads with:

- Client stores upload session ID (localStorage)
- Query completed parts on resume
- Lifecycle policies clean up incomplete uploads after 24-48 hours

### 2. Abuse Prevention

```mermaid
graph LR
    Upload[Client Upload] -->|Quarantine| TempBucket[Temp Storage]
    TempBucket --> Scan{Validation Pipeline}
    Scan -->|Virus Scan| VS[Scanner]
    Scan -->|Content Check| ML[ML Model]
    Scan -->|Size Verify| SV[Size Validator]
    VS --> Decision{Pass?}
    ML --> Decision
    SV --> Decision
    Decision -->|Yes| Final[Production Storage]
    Decision -->|No| Reject[Delete & Log]
```

**Key Strategies**:

- Quarantine uploads before public access
- Automated content analysis
- File type validation
- Size restrictions in presigned URLs
- Rate limiting at API level

### 3. Metadata Management

```mermaid
graph TB
    subgraph "Storage Pattern"
        Key[Storage Key: uploads/user_id/timestamp/uuid]
        Meta[Database Metadata]
        File[Blob Storage File]

        Key -->|Links| Meta
        Key -->|References| File
    end
```

**Best Practices**:

- Create DB record when generating presigned URL
- Use consistent storage key patterns
- Never let clients specify storage keys
- Keep rich metadata in database, not object tags
- Use event notifications to sync state

### 4. Performance Optimization

```mermaid
graph TB
    subgraph "Global Distribution"
        User[User in Sydney] -->|Upload| S3AU[S3 Sydney]
        User2[User in NYC] -->|Download| CF[CloudFront NYC Edge]
        CF -->|Cache Miss| S3AU
        CF -->|Cached| User2
    end
```

**Techniques**:

- CDN for downloads (reduce latency from 200ms to 5ms)
- Range requests for resumable downloads
- Parallel chunk downloads for massive files (rare)
- Regional storage endpoints

## Cloud Provider Reference

| Feature             | AWS                  | Google Cloud         | Azure              |
| ------------------- | -------------------- | -------------------- | ------------------ |
| **Temp Upload URL** | Presigned URLs       | Signed URLs          | SAS Tokens         |
| **Multipart**       | Multipart Upload API | Resumable Uploads    | Block Blobs        |
| **Events**          | S3 Events → SNS/SQS  | Pub/Sub              | Event Grid         |
| **CDN**             | CloudFront           | Cloud CDN            | Azure CDN          |
| **Cleanup**         | Lifecycle Rules      | Lifecycle Management | Lifecycle Policies |

## Critical Interview Points

### Must Mention

1. **Threshold**: Use pattern for files >100MB
2. **State sync**: Database shows "pending" while upload happens
3. **Event notifications**: How to know upload completed
4. **Resumability**: Multipart uploads for large files
5. **Security**: Presigned URLs expire, include size limits

### Common Mistakes to Avoid

1. Forgetting about state synchronization
2. Not mentioning size restrictions in URLs
3. Ignoring the reconciliation/cleanup needs
4. Over-engineering for small files
5. Not considering CDN for downloads

### Advanced Topics (If Time)

1. Parallel multipart uploads
2. Server-side encryption options
3. Cross-region replication
4. Bandwidth throttling strategies
5. Cost optimization (storage classes, lifecycle policies)

## Sample Interview Response Structure

1. **Identify the problem**: "For large files over 100MB, proxying through servers creates bottlenecks..."

2. **Propose solution**: "We'll use presigned URLs for direct uploads to S3, keeping servers out of the data path..."

3. **Draw architecture**: Show client → API (presigned URL) → direct to S3

4. **Address reliability**: "For large files, we'll use multipart uploads for resumability..."

5. **Handle state sync**: "We'll use S3 events to update our database when uploads complete..."

6. **Optimize downloads**: "We'll serve files through CloudFront CDN with signed URLs..."

7. **Discuss trade-offs**: "This adds complexity but is worth it for files over 100MB. For smaller files, we'd use traditional uploads..."

## Quick Decision Framework

```mermaid
graph TD
    Start[File Upload Needed] --> Size{File Size?}
    Size -->|Less than 10MB| Direct[Normal API Upload]
    Size -->|10-100MB| Consider[Consider Context]
    Size -->|Greater than 100MB| Pattern[Use This Pattern]
    Consider --> Freq{Upload Frequency?}
    Freq -->|High| Pattern
    Freq -->|Low| Direct
    Pattern --> Impl[Implement: Presigned URLs, Multipart, Events, CDN]
```

## Additional Considerations

### Security

- Never expose permanent credentials
- Implement IP-based restrictions if needed
- Consider encryption at rest and in transit
- Audit trail for all uploads

### Cost Optimization

- Set lifecycle policies for incomplete uploads
- Use appropriate storage classes (S3 IA, Glacier)
- Consider transfer costs between regions
- CDN costs vs. direct storage serving

### Monitoring

- Track upload success rates
- Monitor incomplete upload costs
- Alert on stuck "pending" files
- Dashboard for upload/download metrics

# Large Blobs Pattern - Last Minute Revision

## 🎯 Core Rule

- **Use this pattern for files > 100MB** (consider for > 10MB)
- Never proxy large files through API servers

## ❌ Anti-Pattern (Don't Do)

- Client → API Server → Storage → API Server → Client
- Problems: Bottleneck, wasted bandwidth, no resumability, memory constraints

## ✅ Solution: Direct Upload Pattern

1. Client requests upload permission from API
2. API returns **presigned URL** (temp credentials)
3. Client uploads **directly to storage** (S3/GCS/Azure)
4. Storage sends event notification to API
5. API updates database status

## 🔑 Key Components

### Presigned URLs

- Temporary credentials (15min - 1hr expiry)
- Generated in-memory (no network call)
- Include restrictions: file size range, content type, storage location
- Example: `https://bucket.s3.amazonaws.com/file?X-Amz-Credential=...&X-Amz-Expires=900`

### Multipart Uploads (Resumability)

- Break large files into chunks (5MB - 5GB per part)
- Each chunk gets an ETag
- Can resume from last successful chunk
- Cleanup incomplete uploads after 24-48 hours

### State Synchronization

- Database states: `pending` → `uploading` → `completed`/`failed`
- Use storage events (S3 Events → SNS/Lambda)
- Reconciliation job for orphaned files

## 📊 Database Schema Essentials

```sql
files table:
- id, user_id, filename
- size_bytes, content_type
- storage_key (s3://bucket/path)
- status (pending/uploading/completed/failed)
- created_at, updated_at
```

## 🌍 Downloads Optimization

- Serve through **CDN** (CloudFront/Cloud CDN)
- Reduces latency from 200ms → 5ms
- Use signed URLs for private content
- Enable range requests for partial downloads

## 🚀 When to Use

### Use For:

- Video platforms (YouTube, TikTok)
- Photo sharing (Instagram)
- File storage (Dropbox, Google Drive)
- Document management (DocuSign)
- Messaging media (WhatsApp)

### Don't Use For:

- Small files (< 10MB) - overhead not worth it
- Synchronous validation needed
- Strict compliance requirements
- Immediate processing (real-time face detection)

## 🛡️ Security & Abuse Prevention

- Quarantine uploads before public access
- Virus scanning pipeline
- File type/size validation in presigned URLs
- Rate limiting at API level
- Never let clients specify storage keys

## ☁️ Cloud Provider Quick Reference

| Feature   | AWS              | GCP               | Azure       |
| --------- | ---------------- | ----------------- | ----------- |
| Temp URLs | Presigned URLs   | Signed URLs       | SAS Tokens  |
| Multipart | Multipart Upload | Resumable Uploads | Block Blobs |
| Events    | S3 Events → SNS  | Pub/Sub           | Event Grid  |
| CDN       | CloudFront       | Cloud CDN         | Azure CDN   |

## 🎤 Interview Must-Mentions

1. **Threshold**: Pattern for files > 100MB
2. **Direct upload**: Client → Storage (bypass server)
3. **Presigned URLs**: Temporary, scoped credentials
4. **State sync**: Database pending → completed via events
5. **Resumability**: Multipart uploads for large files
6. **CDN**: For optimized downloads
7. **Security**: URL expiry, size limits, validation

## 🚫 Common Interview Mistakes

- Forgetting state synchronization
- Not mentioning URL restrictions
- Ignoring cleanup/reconciliation
- Over-engineering for small files
- Missing CDN for downloads

## 🔄 Event Flow Summary

```
Upload: Client → S3 (direct) → S3 Event → Lambda → Update DB
Download: Client → CDN → S3 (on cache miss)
```

## 💡 Advanced Topics (If Time)

- Parallel multipart uploads
- Cross-region replication
- Storage classes (S3 IA, Glacier)
- Bandwidth throttling
- Cost optimization strategies

## 🎯 Quick Decision Framework

- **< 10MB**: Normal API upload
- **10-100MB**: Consider frequency/context
- **> 100MB**: Always use this pattern

## 📝 Sample Answer Structure

1. "For files over 100MB, proxying creates bottlenecks..."
2. "Use presigned URLs for direct uploads..."
3. Draw: Client → API (URL) → Direct to S3
4. "Multipart uploads for resumability..."
5. "S3 events update database status..."
6. "CloudFront CDN for downloads..."
7. "Trade-off: complexity vs. performance..."

## ⚡ Remember

- **Never** proxy large files through servers
- **Always** sync state with events
- **Consider** multipart for resumability
- **Use** CDN for global distribution
- **Implement** proper security/validation
