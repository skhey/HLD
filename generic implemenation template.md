# Generic System Design Interview – Questions to Ask the Interviewer

This template provides a structured set of clarifying questions you can adapt to any system design problem. Use it to uncover requirements, constraints, and trade‑offs. The questions are grouped by the typical phases of a design discussion.

---

## 1. Requirements Gathering

### Functional Requirements
- **Core features:** What are the absolutely essential features the system must support?
- **Optional features:** Are there nice‑to‑have features we might consider (e.g., user accounts, analytics, customisation)?
- **User roles:** Are there different types of users (e.g., regular users, admins)? Do they have different permissions?
- **Input/Output:** What data does the system accept? What does it return? Are there any size or format constraints?
- **Actions/Workflows:** What are the primary user workflows (e.g., create, read, update, delete)?
- **Out of scope:** What features should we explicitly **not** design for now?

### Non‑Functional Requirements
- **Scale:** How many users (DAU/MAU) or operations per day/month? What is the expected read/write ratio?
- **Traffic patterns:** Are there peak times? Should we handle traffic spikes (e.g., viral events)?
- **Latency:** What are the acceptable response times for critical operations (e.g., <100ms, <1s)?
- **Availability:** What uptime SLA is expected (e.g., 99.9%, 99.99%)?
- **Consistency:** Do we need strong consistency, or is eventual consistency acceptable? Any specific consistency requirements (e.g., read‑after‑write)?
- **Durability:** How long must data be retained? Should we support data archival or deletion?
- **Security:** What security concerns must we address (authentication, encryption, abuse prevention, compliance)?
- **Geographical distribution:** Does the system need to serve a global audience? Multi‑region required?

---

## 2. Traffic Estimation and Capacity Planning

- **Daily active users (DAU):** Rough estimate? (e.g., 1M, 100M)
- **Actions per user:** How many reads/writes per user per day?
- **Peak factor:** What is the expected peak‑to‑average ratio (e.g., 2×, 5×)?
- **Data size:** Average size of each data item (e.g., record, file, message)?
- **Retention period:** How long do we keep data (e.g., 1 year, 5 years, forever)?
- **Replication factor:** For durability, do we need multiple copies (e.g., 3×)?
- **Growth rate:** How fast is the data expected to grow (e.g., 20% per year)?

---

## 3. API Design

- **API style:** REST, GraphQL, gRPC, or custom? Any preference?
- **Endpoints:** What are the main endpoints and their HTTP methods?
- **Request/response format:** JSON, Protocol Buffers, XML? Any specific fields required?
- **Authentication:** Do we need API keys, OAuth, JWT, or is the API public?
- **Rate limiting:** Should we implement rate limiting per user/IP? What limits?
- **Idempotency:** Should some operations be idempotent? Do we need idempotency keys?
- **Pagination:** For list endpoints, do we need pagination (offset/limit, cursor)?
- **Versioning:** How should we handle API versioning (URL path, header)?

---

## 4. Database Design

- **Data model:** What entities and relationships exist? (e.g., users, posts, orders)
- **Database type:** SQL or NoSQL? Any preference (e.g., PostgreSQL, Cassandra, DynamoDB)?
- **Access patterns:** What are the main queries (e.g., lookup by ID, range scans, joins)?
- **Indexing:** Which fields need indexes? Any composite indexes?
- **Sharding/Partitioning:** Should we design for sharding? What shard key makes sense?
- **Consistency level:** For NoSQL, what read/write consistency levels should we target?
- **Backup and recovery:** Do we need point‑in‑time recovery, replication, or snapshots?

---

## 5. High‑Level Architecture

- **Deployment environment:** Cloud (AWS/GCP/Azure), on‑premises, or hybrid? Any specific services we can leverage?
- **Components:** What major building blocks are needed (load balancers, web servers, caches, databases, queues)?
- **Communication:** Synchronous (HTTP/gRPC) or asynchronous (message queues)? For which parts?
- **Geographical distribution:** Single region or multi‑region? Active‑active or active‑passive?
- **Existing services:** Are there existing components we can reuse (e.g., CDN, identity provider, monitoring)?

---

## 6. Detailed Component Design

### Data Storage and Processing
- **Caching:** Should we use a cache (Redis/Memcached)? What data should be cached? Eviction policy?
- **File/Object storage:** Do we need to store large files (images, videos)? Use S3, blob storage?
- **Search:** Do we need full‑text search (Elasticsearch)? If so, how to index?
- **Stream processing:** Do we need real‑time analytics or aggregation? Use Kafka, Flink?

### Specific Algorithms/Mechanisms
- **ID generation:** How to generate unique IDs (UUID, Snowflake, database sequence)?
- **Concurrency control:** How to handle concurrent writes/updates (optimistic locking, pessimistic locking)?
- **Rate limiting algorithm:** Token bucket, leaky bucket, sliding window? Where to implement (load balancer, API gateway)?
- **Distributed locking:** If needed, how to implement (Redis, ZooKeeper)?

### Analytics and Monitoring
- **Data to collect:** What metrics/logs are important (request count, latency, errors)?
- **Storage for analytics:** Time‑series DB (InfluxDB, Prometheus), data warehouse (BigQuery)?
- **Alerting:** What thresholds should trigger alerts?

---

## 7. Trade‑offs and Discussion

- **Consistency vs. availability:** Where do we stand on the CAP theorem? What trade‑offs are we making?
- **Cost vs. performance:** Are we optimising for lower cost (e.g., fewer replicas, less caching) or higher performance?
- **Simplicity vs. scalability:** Should we start with a simpler design and plan to scale later, or build for extreme scale from day one?
- **Synchronous vs. asynchronous:** Which operations benefit from async processing? What complexity does it add?
- **SQL vs. NoSQL:** Why choose one over the other given the access patterns?
- **Denormalisation vs. normalisation:** When to denormalise for performance?

---

## 8. Failure Scenarios and Mitigations

- **Single points of failure:** Which components could bring down the system? How to eliminate them (redundancy, failover)?
- **Database failure:** What if the primary database goes down? (Replica promotion, multi‑AZ)
- **Cache failure:** Can the system survive a cache outage? Fallback to database?
- **Service overload:** How to handle traffic spikes (auto‑scaling, rate limiting, queueing)?
- **Network partitions:** How does the system behave during a network split? (CAP trade‑off)
- **Data corruption:** How to recover from accidental deletions or corruption? (Backups, point‑in‑time recovery)
- **Regional outage:** If multi‑region, how does failover work? (DNS, global load balancer)
- **DDoS attacks:** What mechanisms protect against DDoS? (Rate limiting, WAF, CDN)

---

## 9. Scaling to the Next Level (if time permits)

- **What if traffic grows 100×?** Which components become bottlenecks first?
- **Database sharding:** Would we need to re‑shard? How to minimise downtime during rebalancing?
- **Multi‑region active‑active:** How to handle cross‑region replication and conflict resolution?
- **Advanced caching:** Would we use CDN for dynamic content, edge computing (CloudFlare Workers)?
- **Microservices vs. monolith:** Would we split into microservices? How would they communicate?

---

## Tips for Using This Template

- **Adapt to the problem:** Not all questions apply to every system. Pick the most relevant ones.
- **Listen actively:** The interviewer’s answers guide your design. Summarise to confirm understanding.
- **Be concise:** You don’t need to ask everything; focus on clarifying the biggest unknowns.
- **Show your reasoning:** After getting answers, explain how they influence your decisions.

This template will help you navigate any system design interview systematically. Practice by applying it to common problems like designing a chat system, a ride‑hailing service, a video streaming platform, etc.
