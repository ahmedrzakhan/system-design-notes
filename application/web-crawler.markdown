# Web Crawler System Design

This document outlines the design of a scalable, distributed web crawler, suitable for a system design interview. It covers requirements, scale estimation, architecture, and key components.

## 1. Requirements

### 1.1 Functional Requirements
- **Fetch Web Pages**: Download content for a given URL.
- **Store Content**: Save fetched content (HTML, metadata) for downstream processing.
- **Extract Links**: Parse HTML to identify hyperlinks for new URLs to crawl.
- **Avoid Duplicates**: Prevent redundant crawling/storage at URL and content levels.
- **Respect robots.txt**: Adhere to site-specific rules (disallowed paths, crawl delays).
- **Handle Diverse Content**: Primarily HTML, but recognize PDFs, images, etc.
- **Freshness**: Recrawl pages based on update frequency (e.g., news vs. static pages).

### 1.2 Non-Functional Requirements
- **Scalability**: Handle billions of pages across many domains.
- **Politeness**: Limit request rates to avoid overwhelming servers.
- **Extensibility**: Support new parsers, filters, or storage backends.
- **Robustness**: Handle failures (bad URLs, timeouts, node crashes) gracefully.
- **Performance**: High throughput (pages/sec) with low latency.

## 2. Scale Estimation

### 2.1 Number of Pages
- **Target**: 1 billion pages (blogs, news, e-commerce, forums, etc.).

### 2.2 Data Volume
- **HTML Content**: ~100 KB/page.
- **Metadata** (headers, timestamps): ~10 KB/page.
- **Total per Page**: ~110 KB.
- **Total Storage**: 1 billion × 110 KB = ~110 TB.

### 2.3 Bandwidth
- **Goal**: Crawl 1 billion pages in 10 days.
- **Pages per Day**: 1 billion / 10 = 100 million.
- **Pages per Second**: 100 million / (24 × 3600) ≈ 1157 pages/sec.
- **Bandwidth**: 110 KB × 1157 ≈ 127 MB/sec.

### 2.4 URL Frontier
- **Outbound Links per Page**: ~5.
- **New Links per Second**: 1157 × 5 = 5785.
- **Implication**: URL queue must handle thousands of submissions/sec with deduplication and prioritization.

## 3. High-Level Architecture

```mermaid
graph TD
    A[Seed URLs] --> B[URL Frontier]
    B --> C[Scheduler]
    C --> D[Fetcher]
    D --> E[robots.txt Checker]
    E --> F[HTTP Client]
    F -->|Fetch| G[Web Servers]
    F --> H[Content Parser]
    H --> I[Link Extractor]
    I --> B
    H --> J[Content Storage]
    J --> K[Database]
    J --> L[File Storage]
    C --> M[Politeness Manager]
    M --> N[Domain Rate Limiter]
    D --> O[Duplicate Detector]
    O --> P[URL/Content Hash Store]
```

### Components
- **URL Frontier**: Queue of URLs to crawl, with prioritization and deduplication.
- **Scheduler**: Assigns URLs to fetchers, balancing load and politeness.
- **Fetcher**: Downloads content, checks robots.txt, and handles HTTP requests.
- **Content Parser**: Extracts links and metadata from HTML.
- **Link Extractor**: Identifies new URLs to add to the frontier.
- **Content Storage**: Saves HTML and metadata to database/file storage.
- **Politeness Manager**: Enforces crawl delays and rate limits per domain.
- **Duplicate Detector**: Prevents recrawling using URL/content hashes.

## 4. Core Components

### 4.1 URL Frontier
- **Purpose**: Manages URLs to crawl, prioritizing based on freshness, domain authority, or content volatility.
- **Implementation**:
  - Use a distributed queue (e.g., Kafka) for scalability.
  - Prioritize URLs using a scoring function (e.g., page update frequency, domain rank).
  - Deduplicate URLs using a Bloom filter or key-value store (e.g., Redis).
- **Challenges**:
  - Handling rapid URL growth (thousands/sec).
  - Ensuring low-latency access to prioritized URLs.

### 4.2 Scheduler
- **Purpose**: Distributes URLs to fetchers, ensuring load balancing and politeness.
- **Implementation**:
  - Partition URLs by domain to enforce rate limits.
  - Use a worker pool for parallel fetching.
- **Challenges**:
  - Coordinating across distributed nodes.
  - Dynamic adjustment of crawl priorities.

### 4.3 Fetcher
- **Purpose**: Downloads content for a URL.
- **Implementation**:
  - Check robots.txt using a cached parser.
  - Use asynchronous HTTP clients (e.g., Python’s aiohttp).
  - Handle retries, timeouts, and redirects.
- **Challenges**:
  - Managing high request volumes (1157/sec).
  - Handling diverse server responses (errors, rate limits).

### 4.4 Content Parser & Link Extractor
- **Purpose**: Extracts links and metadata from HTML.
- **Implementation**:
  - Use a robust HTML parser (e.g., BeautifulSoup, lxml).
  - Extract links (`<a href>`), canonical URLs, and metadata (title, headers).
  - Normalize URLs (remove fragments, resolve relative paths).
- **Challenges**:
  - Parsing malformed HTML.
  - Efficient link extraction at scale.

### 4.5 Content Storage
- **Purpose**: Stores HTML and metadata for downstream use.
- **Implementation**:
  - **Database**: Store metadata (URL, timestamp, hash) in a NoSQL DB (e.g., Cassandra).
  - **File Storage**: Store raw HTML in a distributed file system (e.g., HDFS, S3).
- **Challenges**:
  - Managing 110 TB of data.
  - Ensuring fast write/read performance.

### 4.6 Politeness Manager
- **Purpose**: Prevents server overload by enforcing crawl delays and rate limits.
- **Implementation**:
  - Track requests per domain using a rate limiter (e.g., token bucket).
  - Cache robots.txt rules with TTL.
- **Challenges**:
  - Balancing politeness with throughput.
  - Handling dynamic crawl delays.

### 4.7 Duplicate Detector
- **Purpose**: Avoids redundant crawling/storage.
- **Implementation**:
  - **URL Deduplication**: Use a Bloom filter or key-value store to check seen URLs.
  - **Content Deduplication**: Compute content hash (e.g., MD5) and check against a store.
- **Challenges**:
  - Minimizing false positives in Bloom filters.
  - Scaling hash storage for billions of pages.

## 5. Database & Storage Options
- **Metadata Store**: Cassandra or DynamoDB for high write throughput and scalability.
- **Content Store**: S3 or HDFS for cost-effective, durable storage of raw HTML.
- **URL Frontier**: Kafka for distributed queueing, Redis for fast deduplication.
- **robots.txt Cache**: Redis with TTL for quick lookups.

## 6. Key Challenges & Solutions
- **Scalability**: Use distributed systems (Kafka, Cassandra) and horizontal scaling.
- **Politeness**: Implement domain-specific rate limiters and robots.txt compliance.
- **Deduplication**: Combine Bloom filters for quick checks with key-value stores for accuracy.
- **Fault Tolerance**: Use retries, circuit breakers, and checkpointing to handle failures.
- **Freshness**: Prioritize recrawling based on content change frequency (e.g., news sites).

## 7. Interview Tips
- **Clarify Requirements**: Ask about scale, content types, and constraints.
- **Start High-Level**: Draw the architecture first, then dive into components.
- **Address Trade-offs**: Discuss storage, deduplication, and politeness strategies.
- **Handle Scale**: Emphasize distributed systems and horizontal scaling.
- **Be Ready for Follow-ups**: Expect questions on robots.txt, deduplication, or error handling.

This design provides a robust foundation for a web crawler, adaptable to various interview scenarios.