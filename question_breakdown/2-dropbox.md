# Dropbox System Design - Interview Guide

## 📋 Problem Statement

Design a cloud-based file storage service like Dropbox that allows users to store, share, and sync files across devices.

## 🎯 Functional Requirements

### Core Requirements

1. **Upload files** from any device
2. **Download files** from any device
3. **Share files** with other users and view shared files
4. **Auto-sync files** across devices

### Out of Scope

- File editing capabilities
- File preview without download
- User storage limits
- File versioning
- Virus/malware scanning

## 🔧 Non-Functional Requirements

### Core Requirements

1. **High Availability** (prioritize availability over consistency)
2. **Support large files** (up to 50GB)
3. **Security & Reliability** (file recovery capabilities)
4. **Low latency** for upload/download/sync

### CAP Theorem Trade-off

- **Choose Availability over Consistency**
- Example: If user in Germany uploads file, user in US can wait a few seconds to see it
- Unlike stock trading where immediate consistency is critical

## 🏗️ Core Entities

```mermaid
erDiagram
    User {
        string userId
        string email
        string name
        string createdAt
        %% Primary Key: userId
    }
    FileMetadata {
        string fileId
        string name
        int size
        string mimeType
        string uploadedBy
        string status
        string s3Url
        string chunks
        string createdAt
        string updatedAt
        %% Primary Key: fileId
        %% Foreign Key: uploadedBy references User(userId)
    }
    SharedFiles {
        string userId
        string fileId
        string permissions
        string sharedAt
        %% Composite Primary Key: (userId, fileId)
    }
    User ||--o{ FileMetadata : uploads
    User ||--o{ SharedFiles : receives
    FileMetadata ||--o{ SharedFiles : shared
```

## 🔌 API Design

### File Upload

```http
POST /files/presigned-url
Request: { fileName, fileSize, mimeType }
Response: { presignedUrl, fileId }
```

### File Download

```http
GET /files/{fileId}/presigned-url
Response: { presignedUrl }
```

### File Sharing

```http
POST /files/{fileId}/share
Request: { userEmails: [] }
```

### Sync Changes

```http
GET /files/changes?since={timestamp}
Response: { files: [FileMetadata] }
```

## 🏛️ High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        UP[Uploader Client]
        DOWN[Downloader Client]
    end

    subgraph "API Layer"
        LB[Load Balancer & API Gateway]
        FS[File Service]
    end

    subgraph "Storage Layer"
        DB[(File Metadata DB)]
        S3[(S3 Blob Storage)]
        CDN[CDN Cache]
    end

    %% Upload Flow
    UP -->|Request presigned URL| LB
    LB --> FS
    FS -->|Generate presigned URL| S3
    FS -->|Store metadata| DB
    UP -->|Upload directly| S3
    S3 -->|Send notification| FS

    %% Download Flow
    DOWN -->|Request download URL| LB
    FS -->|Check permissions| DB
    FS -->|Generate presigned URL| S3
    DOWN -->|Download via CDN| CDN
    CDN -->|Cache miss| S3

```

## 📁 File Upload Evolution

### ❌ Bad: Upload to Server

- Files go through application server
- Server becomes bottleneck
- Expensive and slow

### ✅ Good: Direct to Blob Storage

- Use presigned URLs for direct S3 upload
- Bypass application server for data transfer
- Secure with time-limited URLs

### 🌟 Great: Chunked Upload

- Break large files into 5-10MB chunks
- Enable progress tracking
- Support resumable uploads
- Parallel chunk uploads

```mermaid
sequenceDiagram
    participant C as Client
    participant FS as File Service
    participant S3 as S3 Storage
    participant DB as Metadata DB

    C->>FS: 1. Request presigned URLs for chunks
    FS->>DB: 2. Store file metadata (status: uploading)
    FS->>S3: 3. Generate presigned URLs
    FS->>C: 4. Return presigned URLs

    loop For each chunk
        C->>S3: 5. Upload chunk directly
        S3->>FS: 6. Event notification
        FS->>DB: 7. Update chunk status
    end

    FS->>DB: 8. Mark file as uploaded when all chunks complete
```

## 📥 File Download Evolution

### ❌ Bad: Download through Server

- Double download (S3→Server→Client)
- Slow and expensive

### ✅ Good: Direct from Blob Storage

- Presigned URLs for direct download
- Faster and cheaper

### 🌟 Great: CDN Integration

- Cache frequently accessed files globally
- Serve from nearest edge location
- Signed URLs for security

## 👥 File Sharing Strategies

### ❌ Bad: Sharelist in Metadata

```json
{
  "fileId": "123",
  "name": "file.txt",
  "sharelist": ["user2", "user3"]
}
```

**Problem**: Slow to find files shared with a user (requires scanning all files)

### ✅ Good: Cached Inverse Mapping

- Cache: `user1: ["fileId1", "fileId2"]`
- Fast lookup for user's shared files
- Must keep in sync with metadata

### 🌟 Great: Normalized SharedFiles Table

```sql
SharedFiles:
userId (PK) | fileId (SK) | permissions
user1       | fileId1     | read
user1       | fileId2     | write
```

**Benefits**: No sync issues, clean separation of concerns

## 🔄 File Synchronization

### Local → Remote Sync

```mermaid
graph LR
    FS[File System Watcher] --> Q[Upload Queue]
    Q --> API[Upload API]
    API --> S3[S3 Storage]
    S3 --> Conflict[Conflict Resolution: Last Write Wins]
```

**Client-side sync agent**:

- Monitor local folder with OS file system events
- Queue changes for upload
- Use fingerprinting for deduplication
- Resolve conflicts with "last write wins"

### Remote → Local Sync

**Hybrid Approach**:

- **Fresh files** (recently edited): WebSocket for real-time updates
- **Stale files** (inactive): Periodic polling
- Efficient resource usage

```mermaid
graph TB
    subgraph "Fresh Files (Real-time)"
        WS[WebSocket Connection]
        RT[Real-time Updates]
    end

    subgraph "Stale Files (Polling)"
        POLL[Periodic Polling]
        BATCH[Batch Updates]
    end

    WS --> RT
    POLL --> BATCH
```

## 🚀 Supporting Large Files (Deep Dive)

### Key Challenges

1. **Timeouts**: 50GB at 100Mbps = 1.11 hours
2. **Server Limits**: Most servers limit request size (API Gateway: 10MB)
3. **Network Interruptions**: Need resumability
4. **User Experience**: Need progress indicators

### Solution: File Chunking

```mermaid
graph TB
    FILE[Large File 50GB] --> CHUNK1[Chunk 1: 10MB]
    FILE --> CHUNK2[Chunk 2: 10MB]
    FILE --> CHUNK3[Chunk 3: 10MB]
    FILE --> CHUNKN[Chunk N: 10MB]

    CHUNK1 --> FP1[Fingerprint 1]
    CHUNK2 --> FP2[Fingerprint 2]
    CHUNK3 --> FP3[Fingerprint 3]
    CHUNKN --> FPN[Fingerprint N]
```

### Chunking Process

1. **Client chunks file** into 5-10MB pieces
2. **Calculate fingerprints** (SHA-256) for each chunk and entire file
3. **Check existing uploads** using file fingerprint as ID
4. **Upload missing chunks** with progress tracking
5. **S3 event notifications** update chunk status
6. **Combine chunks** when all uploaded

### Updated File Metadata Schema

```json
{
  "id": "file-fingerprint-sha256",
  "name": "video.mp4",
  "size": 50000000000,
  "status": "uploading",
  "chunks": [
    {
      "id": "chunk1-fingerprint",
      "status": "uploaded"
    },
    {
      "id": "chunk2-fingerprint",
      "status": "uploading"
    },
    {
      "id": "chunk3-fingerprint",
      "status": "not-uploaded"
    }
  ]
}
```

## ⚡ Performance Optimizations

### 1. Compression

- **When to use**: Text files (high compression ratio)
- **When to avoid**: Media files (low compression ratio)
- **Rule**: Compress before encrypt
- **Algorithms**: Gzip (universal), Brotli (better ratio), Zstandard (newest)

### 2. Parallel Chunk Upload

- Maximize bandwidth utilization
- Adaptive chunk sizes based on network conditions
- Only sync changed chunks for file updates

### 3. CDN Optimization

- Cache frequently accessed files
- Strategic caching with TTL headers
- Cache invalidation for updates
- Cost optimization through selective caching

## 🔒 Security Measures

### 1. Encryption

- **In Transit**: HTTPS for all communications
- **At Rest**: S3 server-side encryption with unique keys

### 2. Access Control

- **Authorization**: Check SharedFiles table/cache before generating download URLs
- **Signed URLs**: Time-limited (5 minutes) for downloads
- **CDN Security**: CloudFront signed URLs with expiration

### 3. Signed URL Flow

```mermaid
sequenceDiagram
    participant U as User
    participant FS as File Service
    participant CDN as CDN
    participant S3 as S3

    U->>FS: Request download link
    FS->>FS: Check permissions in SharedFiles
    FS->>FS: Generate signed URL (5 min TTL)
    FS->>U: Return signed URL
    U->>CDN: Download with signed URL
    CDN->>CDN: Validate signature & expiration
    alt Valid signature
        CDN->>S3: Fetch file (if cache miss)
        CDN->>U: Serve file
    else Invalid/expired
        CDN->>U: Access denied
    end
```

## 📊 System Components Deep Dive

### File Service Responsibilities

- Generate presigned URLs for S3
- Manage file metadata in database
- Handle permissions and sharing
- **Does NOT** handle actual file data

### Database Schema

```sql
-- File Metadata
FileMetadata:
- fileId (PK): SHA-256 fingerprint
- name: Original filename
- size: File size in bytes
- mimeType: File type
- uploadedBy: User ID
- status: uploading/uploaded/failed
- s3Url: S3 object location
- chunks: Array of chunk metadata
- createdAt, updatedAt: Timestamps

-- Shared Files (Normalized)
SharedFiles:
- userId (PK): User receiving access
- fileId (SK): File being shared
- permissions: read/write
- sharedAt: Timestamp
```

## 🎯 Interview Expectations by Level

### Mid-Level (E4) - 80% Breadth, 20% Depth

**Expected to deliver**:

- Clear API endpoints and data model
- Functional high-level design for upload/download/sharing
- Respond well to probing questions about optimization

**Not expected to know initially**:

- Presigned URLs
- Chunking strategies
- CDN implementation details

### Senior (E5) - 60% Breadth, 40% Depth

**Expected to deliver**:

- Advanced system design principles
- Trade-off discussions with justification
- Proactive problem identification
- Deep dive into large file handling

**Should demonstrate**:

- Experience with blob storage patterns
- Understanding of CDN benefits
- Knowledge of chunking/multipart uploads

### Staff+ (E6+) - 40% Breadth, 60% Depth

**Expected to deliver**:

- Deep technical expertise from real-world experience
- Proactive problem solving without guidance
- Complex trade-off analysis
- Advanced scalability considerations

**Should demonstrate**:

- Practical technology application
- Anticipate edge cases and solutions
- Drive conversation based on experience

## 🔄 AWS Services Mapping

### Real-World Implementation

- **S3 Multipart Upload**: Handles chunking automatically
- **CloudFront**: CDN with signed URL support
- **API Gateway**: Rate limiting, SSL termination
- **DynamoDB**: File metadata storage
- **Lambda**: Event processing for S3 notifications

## 💡 Key Interview Tips

### 1. Start Simple, Iterate

- Begin with basic upload/download
- Add complexity incrementally
- Show evolution of thinking

### 2. Justify Design Choices

- Explain trade-offs explicitly
- Reference real-world constraints
- Consider cost implications

### 3. Think About Scale

- Mention global distribution needs
- Discuss caching strategies
- Consider bandwidth optimization

### 4. Security Mindset

- Always mention encryption
- Discuss access control
- Explain signed URL benefits

### 5. User Experience Focus

- Progress indicators for uploads
- Resumable operations
- Fast sync across devices

## 🎪 Common Follow-up Questions

1. **"How would you handle file versioning?"**

   - Append-only storage with version pointers
   - Metadata tracks version history
   - Cleanup policies for old versions

2. **"What about conflict resolution?"**

   - Last-write-wins for simplicity
   - Could implement operational transforms
   - User notification for conflicts

3. **"How to optimize for mobile clients?"**

   - Smaller chunk sizes for poor networks
   - Progressive download for media
   - Offline capability with local cache

4. **"Database consistency concerns?"**

   - Eventual consistency acceptable for file sync
   - Strong consistency for permissions
   - Use distributed transactions when needed

5. **"Cost optimization strategies?"**
   - Intelligent CDN caching
   - Compression for appropriate file types
   - Tiered storage (frequent vs. archival)

Remember: **Show your thought process, discuss trade-offs, and demonstrate practical experience with distributed systems!**
