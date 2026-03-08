# Complete High-Level Design (HLD) for a URL Shortening Service (like Bitly)

## Table of Contents
1. [Introduction](#introduction)
2. [Requirements](#requirements)
   - [Functional Requirements](#functional-requirements)
   - [Non-Functional Requirements](#non-functional-requirements)
3. [Traffic Estimation and Capacity Planning](#traffic-estimation-and-capacity-planning)
   - [Assumptions](#assumptions)
   - [Calculations](#calculations)
4. [API Design](#api-design)
   - [Endpoints](#endpoints)
   - [Request/Response Examples](#requestresponse-examples)
   - [Idempotency and Authentication](#idempotency-and-authentication)
5. [Database Design](#database-design)
   - [Data Model](#data-model)
   - [Database Choice](#database-choice)
   - [Indexing and Sharding](#indexing-and-sharding)
   - [Analytics Storage](#analytics-storage)
6. [High-Level Architecture](#high-level-architecture)
   - [Component Diagram](#component-diagram)
   - [Component Roles](#component-roles)
7. [Detailed Component Design](#detailed-component-design)
   - [7.1 Short Code Generation – Key Generation Service (KGS)](#71-short-code-generation--key-generation-service-kgs)
   - [7.2 Caching Layer](#72-caching-layer)
   - [7.3 Redirection Flow](#73-redirection-flow)
   - [7.4 URL Creation Flow](#74-url-creation-flow)
   - [7.5 Analytics Pipeline](#75-analytics-pipeline)
   - [7.6 Handling Hot URLs](#76-handling-hot-urls)
   - [7.7 Bloom Filter for Invalid Requests](#77-bloom-filter-for-invalid-requests)
8. [Multi-Region Deployment](#multi-region-deployment)
   - [Geo-Routing](#geo-routing)
   - [Replication Strategies](#replication-strategies)
9. [Trade-offs and Discussion](#trade-offs-and-discussion)
10. [Failure Scenarios and Mitigations](#failure-scenarios-and-mitigations)
11. [Security Considerations](#security-considerations)
12. [Conclusion and Interview Summary](#conclusion-and-interview-summary)

---

## Introduction
A URL shortening service (like Bitly) converts long URLs into short, easy-to-share aliases. When a user clicks a short link, they are redirected to the original URL. The system must be highly available, low latency, and scalable to billions of redirects. This document presents a complete high‑level design, covering requirements, estimations, APIs, database, architecture, component details, trade‑offs, and failure handling.

---

## Requirements

### Functional Requirements
- **Shorten URL**: Given a long URL, generate a unique, short alias (e.g., `https://short.url/abc123`).
- **Redirect**: When a user accesses the short URL, they are redirected to the original long URL (HTTP 301/302).
- **Custom Alias** (optional): Allow users to specify a custom short path (e.g., `https://short.url/my-link`).
- **Expiration** (optional): URLs may expire after a defined time (TTL) or be persistent.
- **Analytics** (optional but common): Track number of clicks, referrer, geographic location, device, etc.
- **User Accounts** (optional): Users can manage their links and view statistics.

### Non-Functional Requirements
- **High Availability**: The service must be up 99.99% of the time; redirection should never fail.
- **Low Latency**: Redirection should happen in <100ms, ideally <50ms.
- **Scalability**: Support millions of new URLs per day and billions of redirects.
- **Durability**: Once a short URL is created, it should persist (unless expired).
- **Collision‑free**: No two different long URLs can produce the same short code.
- **Security**: Prevent abuse (e.g., excessive creation, malicious URLs) via rate limiting and validation.

---

## Traffic Estimation and Capacity Planning

### Assumptions
- **Total URLs created per month**: 100 million (Bitly‑scale).
- **Read‑to‑write ratio**: 100:1 (each short URL is accessed many times).
- **Peak factor**: 2× average traffic.

### Calculations

**Writes (new short URLs)**
- Per day: 100M / 30 ≈ 3.33M per day.
- Per second (average): 3.33M / 86,400 ≈ 38.5 writes/sec.
- Peak writes: 2× = 77 writes/sec.

**Reads (redirects)**
- Reads per day: 3.33M × 100 = 333M reads/day.
- Reads per second (average): 333M / 86,400 ≈ 3,850 reads/sec.
- Peak reads: 2× = 7,700 reads/sec.

**Storage**
Each record contains:
- Short code (7 chars)
- Long URL (avg 200 chars)
- Metadata (creation timestamp, expiration, user ID, etc.) ≈ 100 bytes
- **Total per record ≈ 500 bytes**
- Daily storage: 3.33M × 500 B = 1.67 GB/day
- **5 years**: 1.67 GB × 365 × 5 ≈ 3.05 TB
- With replication (3×) and overhead: ~10 TB

**Bandwidth (redirects)**
- Each redirect response is an HTTP 301/302 with Location header, approx 500 bytes.
- Daily egress: 333M × 500 B = 166.5 GB/day
- Bandwidth: 166.5 GB / 86,400 ≈ 1.93 MB/s – negligible.

**Caching**
- 80% of traffic often comes from 20% of URLs (Pareto). We can cache the most frequently accessed URLs.
- A cache of 1 million entries (LRU) would require:
  - 1M × 500 B = 500 MB, plus overhead ~1 GB.
- This is feasible with a Redis cluster.

---

## API Design

### Endpoints

#### `POST /api/v1/shorten`
Create a short URL.

**Request Body**:
```json
{
  "long_url": "https://example.com/very/long/path",
  "custom_alias": "my-link",   // optional
  "expiration_days": 30        // optional
}
```

**Response (201 Created)**:
```json
{
  "short_url": "https://short.url/abc123",
  "short_code": "abc123",
  "long_url": "https://example.com/very/long/path",
  "created_at": "2025-03-08T10:37:00Z"
}
```

#### `GET /{short_code}`
Redirect to the original URL.

- **Response**: HTTP 301 (permanent) or 302 (temporary) with `Location` header.
- If short code not found, return 404.

#### `GET /api/v1/analytics/{short_code}`
Retrieve click statistics (requires authentication).

**Response**:
```json
{
  "short_code": "abc123",
  "total_clicks": 10234,
  "clicks_by_country": { "US": 5000, "IN": 3000, ... },
  "clicks_by_device": { "mobile": 7000, "desktop": 3234 },
  "clicks_by_referrer": { "twitter.com": 2000, "facebook.com": 1500 }
}
```

#### `DELETE /api/v1/{short_code}`
Delete a short URL (owner only).

### Idempotency and Authentication
- **Idempotency key** for `POST /shorten`: Clients can pass an `Idempotency-Key` header to prevent duplicate creation on retries.
- **Authentication**: Optional; if user accounts are supported, API keys or OAuth2 tokens are used.

---

## Database Design

### Data Model
We need to store the mapping between short code and long URL, plus metadata.

**Option 1: SQL (if using relational)**
```sql
CREATE TABLE url_mappings (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expiration TIMESTAMP NULL,
    user_id BIGINT NULL,
    INDEX (short_code),
    INDEX (user_id)
);
```

**Option 2: NoSQL (recommended for scale)**
- **Table**: `UrlMappings`
- **Partition key**: `short_code` (string)
- **Attributes**:
  - `long_url` (string)
  - `created_at` (number – epoch time)
  - `expiration` (number – TTL, optional)
  - `user_id` (string, optional)
  - `click_count` (number, denormalized)

For **DynamoDB**:
- Table name: `UrlMappings`
- Partition key: `short_code` (string)
- TTL attribute: `expiration` (enables auto‑deletion)
- Provisioned capacity: on‑demand or auto‑scaling.

### Database Choice
We choose **NoSQL** (DynamoDB or Cassandra) because:
- The access pattern is simple key‑value lookups by `short_code`.
- High throughput (thousands of reads/writes per second) is easy to scale.
- No complex joins are required.
- Horizontal scaling is built‑in.

**Alternative**: Relational database (PostgreSQL) with sharding can work, but adds operational complexity.

### Indexing and Sharding
- The primary index is the partition key `short_code`.
- For user‑specific queries (e.g., list my URLs), a Global Secondary Index (GSI) on `user_id` can be added.
- Sharding is automatic in DynamoDB; in Cassandra, we use `short_code` as the partition key with consistent hashing.

### Analytics Storage
Analytics data (clicks) is high‑volume and requires a different storage solution:
- **Time‑series database** (e.g., ClickHouse, InfluxDB) or **data warehouse** (BigQuery, Snowflake).
- Data is streamed asynchronously via Kafka and consumed by workers that aggregate and store.

---

## High-Level Architecture

### Component Diagram (ASCII)
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Clients   │────▶│   DNS (e.g., │────▶│   CDN (for      │
│ (Web/App)   │     │   Route53)   │     │   static assets)│
└─────────────┘     └──────────────┘     └─────────────────┘
        │                                           │
        │ (HTTP requests)                           │ (optional)
        ▼                                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Load Balancer (e.g., AWS ALB)            │
│                   (distributes traffic to app servers)          │
└─────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Application Servers (Stateless)              │
│   - Handle shortening requests                                   │
│   - Handle redirects (with caching)                              │
│   - Generate short codes via KGS client                          │
└─────────────────────────────────────────────────────────────────┘
                                   │
           ┌───────────────────────┼───────────────────────┐
           │                       │                       │
           ▼                       ▼                       ▼
┌────────────────────┐    ┌────────────────────┐    ┌────────────────────┐
│   Cache (Redis)    │    │  Primary Database  │    │  Message Queue     │
│   - Store frequent │    │  (DynamoDB /       │    │  (Kafka / SQS)     │
│     short→long     │    │   Cassandra)       │    │   - Click events   │
│   - LRU eviction   │    │   - Persistent     │    └────────┬───────────┘
└────────────────────┘    │     storage        │             │
                          └────────────────────┘             │
                                          ┌──────────────────▼──────────────────┐
                                          │        Analytics Workers            │
                                          │   - Aggregate and store in          │
                                          │     ClickHouse / BigQuery           │
                                          └─────────────────────────────────────┘
```

### Component Roles
- **DNS (Route53)**: Geo‑routing to the nearest region.
- **CDN**: Caches static assets (e.g., landing page). Not used for redirection.
- **Load Balancer**: Distributes incoming HTTP requests across application servers; terminates SSL.
- **Application Servers**: Stateless, handle business logic, scaling horizontally.
- **Cache (Redis)**: In‑memory store for frequently accessed mappings. Uses cache‑aside pattern.
- **Primary Database**: Persistent storage for all mappings (DynamoDB / Cassandra).
- **Message Queue (Kafka)**: Buffers click events for analytics.
- **Analytics Workers**: Consume events, aggregate, and store in analytics DB.

---

## Detailed Component Design

### 7.1 Short Code Generation – Key Generation Service (KGS)
To guarantee uniqueness and high throughput, we use a dedicated **Key Generation Service (KGS)**.

**Why KGS?**
- Avoids checking the database for every new URL (which would cause latency).
- Prevents collisions between concurrent requests.
- Pre‑generates keys and serves them in batches.

**Algorithm: Base62 Encoding**
We use characters [a‑z, A‑Z, 0‑9] – 62 possibilities. A 7‑character key gives 62⁷ ≈ 3.5 trillion combinations.

**Snowflake‑like ID Generator** (Alternative)
- 64‑bit ID: timestamp (41 bits) + machine ID (10 bits) + sequence (12 bits).
- Encode the ID in Base62 to get a short string.
- Distributed and collision‑free.

**KGS Workflow**:
1. A background job pre‑generates a large pool of random (or sequential) 7‑char strings and stores them in a separate **Key‑DB** with a `used` flag.
2. Application servers request a batch of keys (e.g., 1,000 at a time) from the KGS.
3. The KGS marks those keys as used and returns them to the server.
4. The server stores the batch in local memory and assigns keys to incoming URLs without further database calls.
5. If a server crashes, the unused keys in its memory are lost. This trade‑off is acceptable because we have a huge key space.

**Handling Concurrency**: Use a distributed lock (e.g., ZooKeeper) to ensure multiple KGS instances don't hand out the same keys.

### 7.2 Caching Layer
- **Cache type**: Redis cluster (multiple nodes).
- **Data**: key = short code, value = long URL.
- **Eviction policy**: LRU (Least Recently Used) to keep hot links.
- **Cache‑aside pattern**:
  - On read, check cache.
  - On miss, query database, then store in cache with a TTL (e.g., 1 hour).
  - On write, optionally update cache (write‑through) or let it be loaded on first read.

### 7.3 Redirection Flow
1. User requests `https://short.url/abc123`.
2. DNS resolves to the nearest load balancer.
3. Load balancer forwards to an application server.
4. Application server:
   - Checks Redis cache for key `abc123`.
   - If **cache hit**: returns HTTP 302 (or 301) with `Location` = long URL.
   - If **cache miss**: queries the database by `short_code`.
     - If found: stores in Redis (with TTL) and returns redirect.
     - If not found: returns 404.
5. **Asynchronous**: A click event (short code, timestamp, referrer, user‑agent, IP) is sent to Kafka for analytics.

### 7.4 URL Creation Flow
1. Client POSTs long URL (and optional custom alias, expiration).
2. Application server validates URL (format, safety check).
3. If custom alias is provided, check uniqueness in database (conditional write).
4. Otherwise, obtain a short code from the local batch (or request a new batch from KGS).
5. Insert mapping into database (with expiration if set).
6. Optionally pre‑warm cache (write‑through) for immediate use.
7. Return short URL to client.

### 7.5 Analytics Pipeline
- **Event**: Each redirect produces a lightweight message (short code, timestamp, referrer, user‑agent, IP).
- **Message queue**: Kafka (or SQS) decouples event collection from the main request path.
- **Stream processing**: Consumers (e.g., Kafka Streams, Flink, Spark) aggregate data:
  - Increment counters per short code.
  - Group by country (IP geolocation), device (user‑agent), referrer.
- **Storage**: Aggregated results are stored in a time‑series database (ClickHouse) or data warehouse (BigQuery) for fast queries.
- **Query API**: The `GET /analytics/{short_code}` endpoint reads from this store.

### 7.6 Handling Hot URLs
Popular links (e.g., a viral tweet) can generate massive traffic.
- **CDN edge caching**: For extremely popular short URLs, configure the CDN to cache the redirect response (HTTP 301/302). The CDN returns the redirect directly, bypassing origin.
- **Replicated cache**: Replicate hot keys across multiple Redis nodes to spread load.
- **Local cache**: Application servers maintain a small in‑memory LRU cache for the hottest links (e.g., Caffeine library).

### 7.7 Bloom Filter for Invalid Requests
Malicious or random short code requests can cause many cache misses and database lookups. Use a **Bloom filter** to quickly reject non‑existent keys.

- **Data structure**: A space‑efficient probabilistic set of all existing short codes.
- **Placement**: In front of the cache (or in application servers).
- **Operation**: When a request arrives, check the Bloom filter:
  - If it says “not present”, return 404 immediately (no cache/db lookup).
  - If it says “possibly present”, proceed to cache/db.
- **Trade‑off**: False positives possible, false negatives impossible. Tune size and hash functions for desired accuracy.

---

## Multi-Region Deployment

### Geo-Routing
Use **GeoDNS** (e.g., AWS Route53) to route users to the nearest data center:
- US‑East, US‑West, EU, Asia, etc.

### Replication Strategies
**Option A: Active‑Active**
- Each region accepts both reads and writes.
- Data is replicated asynchronously between regions (multi‑master).
- Conflicts are rare (short codes are unique per region) but can be resolved via last‑write‑wins or vector clocks.
- **Pros**: Lowest latency for all users.
- **Cons**: Consistency trade‑off; cross‑region replication lag.

**Option B: Active‑Passive**
- One primary region handles writes; others are read‑only replicas.
- Writes from other regions are forwarded to primary.
- **Pros**: Simpler consistency.
- **Cons**: Higher write latency for users far from primary.

For a URL shortener, eventual consistency is acceptable. We choose **Active‑Active** with DynamoDB Global Tables or Cassandra multi‑region.

---

## Trade-offs and Discussion

| Decision | Trade‑off |
|----------|-----------|
| **NoSQL (DynamoDB) vs SQL** | NoSQL gives horizontal scalability and low latency for key‑value lookups, but sacrifices complex queries (e.g., user‑based listing). We add a GSI for user queries. |
| **KGS vs on‑the‑fly hashing** | KGS guarantees no collisions and high throughput, but introduces extra service dependency. Hashing is simpler but may require collision resolution and database checks. |
| **Redis vs local cache** | Redis is a shared cache, consistent across servers, but adds network hop. Local cache is faster but can become inconsistent and duplicates data. We use Redis for shared cache and optional local L1 cache. |
| **301 vs 302 redirect** | 301 (permanent) is cached by browsers, reducing load but making updates difficult. 302 (temporary) is not cached, allowing analytics on every click. We default to 302 for trackability. |
| **Strong vs eventual consistency** | For redirects, eventual consistency is fine; a newly created URL might take a few seconds to propagate. For custom aliases, we need strong consistency (conditional write). DynamoDB supports conditional writes. |

---

## Failure Scenarios and Mitigations

| Failure | Mitigation |
|---------|------------|
| **Database node down** | Use replication (e.g., DynamoDB multi‑AZ). Reads/writes continue if quorum available. |
| **Cache cluster failure** | Fallback to database reads. Cache is rebuilt gradually. Redis has replica nodes for failover. |
| **Load balancer failure** | Multiple load balancers in different AZs with DNS failover. |
| **Application server failure** | Stateless; auto‑scaling group replaces instances. Load balancer health checks stop routing to failed servers. |
| **KGS failure** | KGS is replicated (active‑passive). App servers keep a local batch; they can request a new batch from another KGS instance. |
| **Data corruption / accidental deletion** | Regular backups (DynamoDB point‑in‑time recovery). Soft deletes (mark as deleted) can be used. |
| **DDoS attack** | Rate limiting at load balancer/API gateway (token bucket). Use AWS Shield / Cloudflare. |
| **Malicious URL creation** | Validate URLs against malware lists (e.g., Google Safe Browsing). Rate limit per IP/user. |
| **Region outage** | DNS failover to another region; data replicated asynchronously. Active‑active setup ensures the other regions already have data. |

---

## Security Considerations
- **Rate limiting**: Prevent abuse by limiting number of URL creations per IP/user (e.g., 100/hour). Use Redis counters with sliding window or token bucket.
- **URL validation**: Reject malformed or malicious URLs (phishing, malware) by integrating with threat intelligence services.
- **Authentication**: Optional API keys for users to manage their links.
- **HTTPS**: Enforce TLS for all endpoints.
- **CORS**: Restrict access to allowed domains if providing a web interface.

---

## Conclusion and Interview Summary

### Key Points
- **Short code generation**: Use a Key Generation Service (KGS) with Base62 encoding for uniqueness and performance.
- **Database**: NoSQL (DynamoDB/Cassandra) for key‑value lookups and horizontal scaling.
- **Caching**: Redis cluster with cache‑aside and LRU eviction to handle high read traffic.
- **Analytics**: Asynchronous pipeline via Kafka and ClickHouse to avoid impacting redirect latency.
- **Scalability**: Consistent hashing for database sharding, multi‑region active‑active deployment, CDN for hot URLs.
- **Reliability**: Redundancy at every layer, health checks, auto‑scaling, and disaster recovery strategies.
- **Trade‑offs**: Chosen to prioritize availability, low latency, and scalability over strong consistency.

### Perfect Interview Answer (30‑second version)
> "A URL shortener stores a mapping between a short code and a long URL. When a user creates a short URL, we generate a unique ID using a distributed key generation service (KGS) and encode it in Base62. The mapping is stored in a NoSQL database like DynamoDB for scalability. For redirection, we first check a Redis cache; on a miss, we query the database and populate the cache. To handle billions of redirects, we use CDN edge caching for popular links, a Bloom filter to reject invalid requests, and an asynchronous analytics pipeline with Kafka. The system is deployed across multiple regions with active‑active replication and geo‑routing for low latency and high availability."

This design addresses all aspects from the problem statement and is ready for a top‑level system design interview.
