# Functional Requirements – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about functional requirements for a URL shortening service.

---

## Core Features
**Question:** *What are the absolutely essential features the system must support?*

**Interviewer:** The two core features are:
1.  **Shorten URL:** Given a long URL, the system must generate a unique, short alias (e.g., `https://short.url/abc123`).
2.  **Redirect:** When a user accesses the short URL, they must be redirected (HTTP redirect) to the original long URL.

Everything else is secondary for this design session.

---

## Optional Features
**Question:** *Are there nice‑to‑have features we might consider (e.g., user accounts, analytics, customisation)?*

**Interviewer:** Yes, we can consider the following as optional enhancements, but they are not mandatory for the core design. We can mention them, but focus on the core first:
- **Custom aliases:** Allow users to specify their own short path (e.g., `bit.ly/mylink`).
- **Expiration:** Allow URLs to expire after a certain time (TTL).
- **Basic analytics:** Track the number of clicks per short URL.
- **User accounts:** Users can create an account to manage their links and view analytics.

For now, let's focus on the core. If we have time, we can discuss how we might add one of these.

---

## User Roles
**Question:** *Are there different types of users (e.g., regular users, admins)? Do they have different permissions?*

**Interviewer:** Not applicable. For this design, we are building a public service. There are no user roles or permissions. Anyone can shorten a URL without logging in.

---

## Input/Output
**Question:** *What data does the system accept? What does it return? Are there any size or format constraints?*

**Interviewer:**
- **Input:** The system accepts a long URL (a string). We should validate that it's a properly formatted URL.
- **Output:** The system returns the generated short URL (a string).
- **Constraints:** Long URLs can be up to, say, 2048 characters (common limit). Short URLs should be as short as possible, ideally 6–7 characters.

---

## Actions/Workflows
**Question:** *What are the primary user workflows (e.g., create, read, update, delete)?*

**Interviewer:** There are only two primary workflows:
1.  **Create (Shorten):** User provides a long URL → System returns a short URL.
2.  **Read (Redirect):** User visits a short URL → System redirects to the long URL.

We do not need to support update or delete operations for this design. (If we add user accounts later, we might, but for now, it's not required.)

---

## Out of Scope
**Question:** *What features should we explicitly not design for now?*

**Interviewer:** We will explicitly **not** design the following:
- User accounts and authentication.
- Detailed analytics dashboards (though we might mention how we'd collect click data).
- Link management (editing, deleting).
- Admin interfaces or moderation tools.
- Billing or payment integration.
- Support for custom domains.

We are building a simple, public URL shortener with just the core shortening and redirection features.

---

## Summary for the Candidate

Based on these answers, you now know:
- **Core:** Shorten + Redirect.
- **Optional:** Custom aliases, expiration, basic analytics (nice to mention).
- **Scale:** Not given yet – you'll need to ask about traffic estimates next.
- **Constraints:** URL length up to ~2000 chars; short code length 6–7 chars.
- **Out of scope:** User accounts, management, admin features.

This gives you a clear scope to start your design. You can now move on to **non‑functional requirements** (scale, latency, availability) and **traffic estimation**.


# Non‑Functional Requirements – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about non‑functional requirements for a URL shortening service.

---

## Scale
**Question:** *How many users (DAU/MAU) or operations per day/month? What is the expected read/write ratio?*

**Interviewer:** Let's assume we're building a service similar to Bitly, targeting:
- **New URLs per month:** 100 million
- **Read/write ratio:** 100:1 (each short URL is accessed about 100 times on average)
- **Redirects per month:** 100 million × 100 = 10 billion redirects

You can use these numbers to calculate QPS and storage. Feel free to round them for simplicity.

---

## Traffic Patterns
**Question:** *Are there peak times? Should we handle traffic spikes (e.g., viral events)?*

**Interviewer:** Yes, traffic will not be perfectly uniform. There will be:
- **Daily peaks:** For example, evenings in each time zone may see 2–3× average traffic.
- **Viral events:** Occasionally, a short URL may go viral and receive millions of clicks in a short period. The system should handle such spikes gracefully without degrading performance.

Assume a **peak factor of 5×** average for worst‑case planning, but you can use 2–3× for typical peak estimation.

---

## Latency
**Question:** *What are the acceptable response times for critical operations (e.g., <100ms, <1s)?*

**Interviewer:**
- **Redirection:** This is the most critical operation. It should be **<50ms** ideally, and definitely under **100ms**. Users expect instant redirects.
- **URL creation:** This can be slightly slower, but still should be under **500ms** for a good user experience.

---

## Availability
**Question:** *What uptime SLA is expected (e.g., 99.9%, 99.99%)?*

**Interviewer:** Since redirection is a core function that many other services may rely on (e.g., links in emails, social media), we need high availability. Target **99.99%** availability. This translates to about **52 minutes of downtime per year**. The system should be designed to survive component failures without noticeable impact.

---

## Consistency
**Question:** *Do we need strong consistency, or is eventual consistency acceptable? Any specific consistency requirements (e.g., read‑after‑write)?*

**Interviewer:**
- For **redirects**, eventual consistency is acceptable. If a newly created URL takes a few seconds to propagate to all nodes, it's not a disaster – the user can just retry.
- For **custom aliases** (if we implement them), we need stronger consistency to ensure uniqueness. Two users should not be able to claim the same custom alias simultaneously. This requires a conditional write with uniqueness check.
- **Read‑after‑write consistency** for the user who just created the URL would be nice but not strictly required. If they try to access it immediately and get a 404, that's a poor experience, so we should aim for it.

So, we can live with **eventual consistency for reads**, but need **strong consistency for writes that require uniqueness**.

---

## Durability
**Question:** *How long must data be retained? Should we support data archival or deletion?*

**Interviewer:** Assume we need to keep mappings **indefinitely** (or for at least 5–10 years). Short URLs should not disappear. However, we may eventually implement expiration for inactive links to save storage, but that's an optional feature. For core design, assume **permanent storage**.

No explicit archival requirement, but we should consider that the dataset will grow over time.

---

## Security
**Question:** *What security concerns must we address (authentication, encryption, abuse prevention, compliance)?*

**Interviewer:**
- **Encryption:** All traffic should be over HTTPS (TLS). No exceptions.
- **Abuse prevention:** We need to prevent malicious users from:
  - Creating too many URLs (spam).
  - Pointing short URLs to malicious sites (phishing, malware).
- **Rate limiting:** Implement rate limiting on URL creation per IP to prevent abuse.
- **URL validation:** Check that long URLs are valid and optionally check against blacklists (e.g., Google Safe Browsing API).
- **No authentication** required for this design (public service).

---

## Geographical Distribution
**Question:** *Does the system need to serve a global audience? Multi‑region required?*

**Interviewer:** Yes, the service will be used worldwide. We should aim for **low latency for all users**, which suggests a multi‑region deployment. However, for the core design, you can assume a single region initially and discuss how you would extend to multiple regions. If you have time, mention multi‑region replication and geo‑routing as enhancements.

For this interview, you can focus on a single region but be prepared to discuss how you'd scale globally.

---

## Summary Table for Reference

| Category | Requirement |
|----------|-------------|
| **Scale** | 100M new URLs/month; 100:1 read/write ratio; 10B redirects/month |
| **Traffic peaks** | 2–5× average; handle viral spikes |
| **Latency** | Redirect <100ms; Create <500ms |
| **Availability** | 99.99% uptime |
| **Consistency** | Eventual for reads; strong for unique writes (custom aliases) |
| **Durability** | Permanent storage (≥5 years) |
| **Security** | HTTPS, rate limiting, URL validation |
| **Geo distribution** | Global audience – multi‑region desired |

You now have a complete set of non‑functional requirements to inform your design decisions. Next, you can move on to **traffic estimation and capacity planning** using these numbers.



# Traffic Estimation and Capacity Planning – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about traffic estimation and capacity planning for a URL shortening service.

---

## Daily Active Users (DAU)
**Question:** *Rough estimate? (e.g., 1M, 100M)*

**Interviewer:** For a URL shortener, DAU isn't the best metric because many users may create URLs infrequently. Instead, let's focus on **operations per month** as we discussed earlier:
- **New URLs per month:** 100 million
- **Redirects per month:** 10 billion

You can derive daily numbers from these. If you need DAU for creation, assume about **10–20 million users** might create at least one URL per month, but it's not critical. Focus on the operation counts.

---

## Actions per User
**Question:** *How many reads/writes per user per day?*

**Interviewer:** This is already captured in the read/write ratio. For creation, a "user" might create 1–2 URLs per session, but we don't have user accounts. Better to stick with the **100:1 read/write ratio** we already established:
- **Writes:** 100M new URLs/month → ~3.3M/day
- **Reads:** 10B redirects/month → ~333M/day

If you want per "creator," you could say each creator generates about 5–10 URLs on average, but it's not necessary for capacity planning.

---

## Peak Factor
**Question:** *What is the expected peak‑to‑average ratio (e.g., 2×, 5×)?*

**Interviewer:** Assume:
- **Daily peaks:** 2–3× average traffic (e.g., evenings in each time zone)
- **Viral spikes:** Occasionally, a link may go viral and cause a short burst of 10–20× normal traffic for that specific URL

For infrastructure sizing, design for **3× average** for general traffic, but also discuss how you'd handle extreme spikes (caching, CDN, auto‑scaling).

---

## Data Size
**Question:** *Average size of each data item (e.g., record, file, message)?*

**Interviewer:** Estimate per stored URL:
- **Short code:** 6–7 characters (7 bytes)
- **Long URL:** Average 200 characters (200 bytes) – some longer, some shorter
- **Metadata:** Creation timestamp, expiration (if any), user ID (optional) – approx 100 bytes
- **Total per record:** ~300–500 bytes. Let's use **500 bytes** to be safe and include overhead.

For analytics events (if we add them later), each click event might be ~100 bytes (timestamp, short code, referrer, user agent, IP).

---

## Retention Period
**Question:** *How long do we keep data (e.g., 1 year, 5 years, forever)?*

**Interviewer:** Assume we keep mappings **permanently** – at least **5–10 years**. Short URLs should not expire by default. If we later add expiration as a feature, it will be user‑controlled, but for core design, plan for indefinite storage.

For analytics, we might keep detailed data for 30–90 days and aggregated data forever.

---

## Replication Factor
**Question:** *For durability, do we need multiple copies (e.g., 3×)?*

**Interviewer:** Yes, to achieve 99.99% durability and availability, assume we need **3× replication** across availability zones or nodes. This is typical for production systems (e.g., DynamoDB, Cassandra, S3). Use **3×** in your storage calculations.

---

## Growth Rate
**Question:** *How fast is the data expected to grow (e.g., 20% per year)?*

**Interviewer:** Assume **steady growth** of about **20–30% per year** in new URL creations. This is reasonable for a popular service. Your capacity plan should account for this growth over a 5‑year horizon.

---

## Summary of Numbers for Your Calculations

| Metric | Value |
|--------|-------|
| New URLs per month | 100 million |
| New URLs per day | ~3.33 million |
| Redirects per month | 10 billion |
| Redirects per day | ~333 million |
| Read/write ratio | 100:1 |
| Peak factor (general) | 3× average |
| Peak factor (viral) | 10–20× (handle with caching/CDN) |
| Record size | ~500 bytes |
| Retention | 5+ years (permanent) |
| Replication factor | 3× |
| Annual growth | 20–30% |

---

## What the Candidate Should Do Next

Now that you have these numbers, you should:

1. **Calculate average and peak QPS** for reads and writes.
2. **Estimate daily storage growth** and 5‑year total with replication.
3. **Estimate bandwidth** for redirects (egress).
4. **Estimate cache size** (e.g., cache the top 20% of URLs).
5. **Use these numbers** to justify your choice of database, caching tier, and scaling strategy.

Example:
- Write QPS average: 3.33M / 86,400 ≈ 38 writes/sec
- Write QPS peak (3×): 115 writes/sec
- Read QPS average: 333M / 86,400 ≈ 3,850 reads/sec
- Read QPS peak (3×): 11,550 reads/sec
- Daily storage: 3.33M × 500 bytes = 1.67 GB/day
- 5‑year storage: 1.67 GB × 365 × 5 ≈ 3.05 TB × 3 (replication) ≈ 9.15 TB

These numbers will drive your design decisions.


# API Design – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about API design for a URL shortening service.

---

## API Style
**Question:** *REST, GraphQL, gRPC, or custom? Any preference?*

**Interviewer:** Let's use **REST** – it's simple, widely understood, and perfect for this use case. We have straightforward resource operations (create a short URL, retrieve long URL). No need for GraphQL's complexity or gRPC's overhead for a public-facing API.

---

## Endpoints
**Question:** *What are the main endpoints and their HTTP methods?*

**Interviewer:** We need just two core endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/shorten` | `POST` | Create a new short URL |
| `/{shortCode}` | `GET` | Redirect to the original long URL |

If we add analytics later, we might add:
| `/analytics/{shortCode}` | `GET` | Retrieve click statistics |

For optional features (if time permits):
| `/shorten` | `DELETE` | Delete a short URL (requires auth) |

But focus on the first two for now.

---

## Request/Response Format
**Question:** *JSON, Protocol Buffers, XML? Any specific fields required?*

**Interviewer:** Use **JSON** – it's the standard for REST APIs. Keep it simple.

**POST /shorten Request:**
```json
{
  "long_url": "https://example.com/very/long/path",
  "custom_alias": "my-link",        // optional
  "expiration_days": 30              // optional
}
```

**Successful Response (201 Created):**
```json
{
  "short_url": "https://short.url/abc123",
  "short_code": "abc123",
  "long_url": "https://example.com/very/long/path",
  "created_at": "2025-03-08T10:37:00Z"
}
```

**Error Response (e.g., 400 Bad Request):**
```json
{
  "error": "Invalid URL format",
  "code": "INVALID_URL"
}
```

For the redirect endpoint, it's just a standard HTTP 301/302 with `Location` header – no JSON body needed.

---

## Authentication
**Question:** *Do we need API keys, OAuth, JWT, or is the API public?*

**Interviewer:** The API is **public**. No authentication required for creating short URLs or for redirects. This is a free, open service. (If we later add user accounts, we'd introduce API keys or OAuth, but that's out of scope for now.)

---

## Rate Limiting
**Question:** *Should we implement rate limiting per user/IP? What limits?*

**Interviewer:** Yes, absolutely. We need to prevent abuse (e.g., someone creating millions of URLs to spam). Implement rate limiting per **IP address**:

- **URL creation:** 100 requests per hour per IP
- **Redirects:** No rate limiting needed – they're cheap and should be unlimited

Use a **token bucket** or **sliding window** algorithm. Return `429 Too Many Requests` with a `Retry-After` header when limit is exceeded.

---

## Idempotency
**Question:** *Should some operations be idempotent? Do we need idempotency keys?*

**Interviewer:** Good question. `POST /shorten` is **not** idempotent by default – if the client retries, you might create duplicate short URLs for the same long URL. This may or may not be acceptable.

For this design, **idempotency is optional** but nice to have. If you want to implement it, clients can send an `Idempotency-Key` header (e.g., a UUID). The server would store the key and return the same result for subsequent requests with the same key within a time window (e.g., 24 hours). This prevents accidental duplicates caused by network retries.

If you skip it, we can live with duplicates – it's not a critical failure. But mentioning it shows foresight.

---

## Pagination
**Question:** *For list endpoints, do we need pagination (offset/limit, cursor)?*

**Interviewer:** **Not applicable** for the core design. We don't have a "list all URLs" endpoint. If we later add user accounts and a dashboard to list a user's URLs, we would need pagination. But for now, it's out of scope.

---

## Versioning
**Question:** *How should we handle API versioning (URL path, header)?*

**Interviewer:** Use **URL path versioning** – it's simple and explicit:
- `https://api.short.url/v1/shorten`
- `https://api.short.url/v2/shorten`

This allows us to introduce breaking changes in the future without affecting existing clients. For now, we can start with `/v1/`. If we don't version, we risk breaking existing integrations when we evolve the API.

---

## Summary Table

| Aspect | Decision |
|--------|----------|
| **API style** | REST |
| **Core endpoints** | `POST /v1/shorten`, `GET /{shortCode}` |
| **Request format** | JSON |
| **Authentication** | None (public) |
| **Rate limiting** | 100 requests/hour per IP for creation |
| **Idempotency** | Optional – can use `Idempotency-Key` header |
| **Pagination** | Not needed |
| **Versioning** | URL path (`/v1/`) |

---

## What the Candidate Should Do Next

Now that you have these API specifications, you should:

1. **Incorporate these endpoints** into your high‑level architecture diagram.
2. **Explain how rate limiting** would be implemented (e.g., Redis + token bucket).
3. **Discuss idempotency** if you choose to include it – where would you store the keys? (Redis with TTL)
4. **Show how versioning** allows future evolution.

Example:
> "I'll design the API with `/v1/shorten` for creation and `GET /{shortCode}` for redirects. To prevent abuse, I'll implement rate limiting per IP using a token bucket in Redis. For idempotency, clients can optionally send an `Idempotency-Key` header, and I'll store the response in Redis for 24 hours to return on duplicate requests. This ensures we don't accidentally create duplicate URLs even if the client retries."

This shows you've thought through the API design comprehensively.


# Database Design – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about database design for a URL shortening service.

---

## Data Model
**Question:** *What entities and relationships exist? (e.g., users, posts, orders)*

**Interviewer:** For the core design, we have a very simple data model with just one main entity:

**URL Mapping**
- `short_code` (string, unique identifier)
- `long_url` (string)
- `created_at` (timestamp)
- `expiration` (timestamp, optional)
- `user_id` (optional – if we add user accounts later)

There are no relationships between entities. It's essentially a key-value store: given a short code, retrieve the long URL.

If we add analytics later, we'd have a separate entity for **Click Events**:
- `id` (unique identifier)
- `short_code` (string)
- `timestamp`
- `referrer`
- `user_agent`
- `ip_address` (or geolocation data)

But for now, focus on the URL mappings.

---

## Database Type
**Question:** *SQL or NoSQL? Any preference (e.g., PostgreSQL, Cassandra, DynamoDB)?*

**Interviewer:** This is a key design decision. You need to justify your choice. I'll give you the requirements, and you decide:

We need:
- **High read throughput** (thousands of QPS)
- **Low latency** (<10ms for reads)
- **Horizontal scalability** (data will grow to TBs)
- **Simple key-value lookups** (no complex joins)
- **High availability** (99.99%)

Based on these, a **NoSQL key-value store** is a natural fit. Options:
- **DynamoDB** (fully managed, auto-scaling, consistent performance)
- **Cassandra** (open source, excellent write scalability, tunable consistency)
- **Redis** (in-memory only – not suitable as primary store for TBs of data)

I have no strong preference – choose one and justify it. DynamoDB is simpler to operate; Cassandra gives you more control. Both are acceptable.

If you choose SQL (e.g., PostgreSQL with sharding), explain how you'd handle scaling and why. That's also valid but more complex.

---

## Access Patterns
**Question:** *What are the main queries (e.g., lookup by ID, range scans, joins)?*

**Interviewer:** Very simple:
- **Primary query:** `SELECT long_url FROM url_mappings WHERE short_code = :code` – this is 99.9% of queries.
- **Insert:** `INSERT INTO url_mappings (short_code, long_url, created_at) VALUES (...)`
- **Optional (if we add user accounts):** `SELECT * FROM url_mappings WHERE user_id = :user_id ORDER BY created_at DESC` – this would be a secondary access pattern.

No range scans, no joins, no complex aggregations on the main table.

---

## Indexing
**Question:** *Which fields need indexes? Any composite indexes?*

**Interviewer:**
- **Primary index:** `short_code` – this is the main lookup key. In NoSQL, this is the partition key.
- **Secondary index:** If we add user accounts, we'd need an index on `user_id` to list a user's URLs. In DynamoDB, that would be a Global Secondary Index (GSI). In Cassandra, you'd create a secondary index or denormalize.

No composite indexes needed for the core design.

---

## Sharding/Partitioning
**Question:** *Should we design for sharding? What shard key makes sense?*

**Interviewer:** Yes, we need to design for horizontal scaling from the start because we'll have terabytes of data.

**Shard key:** `short_code` is the obvious choice. It's how we access data 99% of the time, so it ensures queries are routed to a single shard. In DynamoDB, this is the partition key. In Cassandra, it's the primary key's partition key.

If we use a hash-based sharding (consistent hashing), we get even distribution. With 3.5 trillion possible short codes, the distribution will be uniform.

For the user‑listing query (if added), it would become a scatter‑gather operation across all shards, which is inefficient. That's a trade‑off we accept for the primary access pattern's efficiency. We could mitigate by also storing data by user_id in a separate table (denormalization).

---

## Consistency Level
**Question:** *For NoSQL, what read/write consistency levels should we target?*

**Interviewer:** Based on our earlier non‑functional requirements:

- **For redirects (reads):** Eventual consistency is acceptable. In DynamoDB, use `EventuallyConsistent` reads (cheaper, lower latency). In Cassandra, use `ONE` or `LOCAL_ONE`.
- **For writes (new URLs):** We need durability. In DynamoDB, use `STANDARD` write (acknowledged after replication to multiple AZs). In Cassandra, use `QUORUM` or `LOCAL_QUORUM` to ensure the write is persisted on multiple nodes.
- **For custom aliases (if implemented):** We need strong consistency to ensure uniqueness. Use a conditional write with `ConsistentRead` in DynamoDB, or `SERIAL` consistency in Cassandra.

So we'll use **tunable consistency** – weaker for reads, stronger for writes where uniqueness matters.

---

## Backup and Recovery
**Question:** *Do we need point‑in‑time recovery, replication, or snapshots?*

**Interviewer:** Yes. For a production system with 99.99% availability and permanent data retention, we need:

- **Automated backups:** Daily snapshots (or continuous backups) to recover from accidental deletion or corruption.
- **Point‑in‑time recovery (PITR):** Ability to restore to any point within the last 35 days (DynamoDB feature). This protects against human errors.
- **Cross‑region replication:** Optional but recommended for disaster recovery. If the primary region goes down, we can fail over to a replica.

In DynamoDB, you'd enable PITR and consider Global Tables for multi‑region replication. In Cassandra, you'd use `nodetool snapshot` and replicate to another cluster.

For our design, at minimum, we need **daily backups** and **replication within the region** (across AZs). Multi‑region is a nice‑to‑have.

---

## Summary Table

| Aspect | Decision |
|--------|----------|
| **Data model** | Single table: `url_mappings` (short_code, long_url, created_at, expiration, user_id) |
| **Database type** | NoSQL (DynamoDB or Cassandra) – justify your choice |
| **Access patterns** | Primary: lookup by short_code; Secondary: list by user_id (optional) |
| **Indexing** | Primary key on short_code; GSI on user_id (optional) |
| **Shard key** | `short_code` (hash-based for even distribution) |
| **Consistency** | Eventual for reads, strong for writes with uniqueness (custom aliases) |
| **Backup** | Daily snapshots + point‑in‑time recovery |

---

## What the Candidate Should Do Next

Now that you have these database requirements, you should:

1. **Choose a specific database** (e.g., DynamoDB) and justify why it fits the access patterns and scale.
2. **Design the table schema** with partition key, sort key (if any), and attributes.
3. **Explain how sharding works** – e.g., consistent hashing in Cassandra, or DynamoDB's automatic partitioning.
4. **Discuss how you'd handle the secondary access pattern** (user's URLs) – e.g., denormalize into a separate table or use a GSI.
5. **Describe the backup strategy** – e.g., enable PITR, daily exports to S3.

Example:
> "I'll use DynamoDB because it's fully managed, scales automatically, and provides single‑digit millisecond latency for key‑value lookups. The table will have `short_code` as the partition key. For reads, I'll use `EventuallyConsistent` reads to reduce cost and latency. For writes, I'll use standard writes. If we add custom aliases, I'll use conditional writes with `ConsistentRead` to ensure uniqueness. For backups, I'll enable point‑in‑time recovery and export daily snapshots to S3 for long‑term archival. If we later need user‑specific queries, I'll add a Global Secondary Index on `user_id`."

This shows you've thought through all aspects of the database design.



# High-Level Architecture – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about the high‑level architecture for a URL shortening service.

---

## Deployment Environment
**Question:** *Cloud (AWS/GCP/Azure), on‑premises, or hybrid? Any specific services we can leverage?*

**Interviewer:** We're building this for a cloud environment – let's assume **AWS** since it's widely used and has a rich set of managed services. You're free to mention equivalent services on other clouds if you prefer. Leveraging managed services will speed up development and reduce operational overhead.

You can assume we have access to:
- **Compute:** EC2, ECS (Fargate), or Lambda
- **Load balancing:** Application Load Balancer (ALB) or Network Load Balancer (NLB)
- **DNS:** Route 53
- **CDN:** CloudFront
- **Caching:** ElastiCache for Redis (or Memcached)
- **Database:** DynamoDB (or RDS if you choose SQL)
- **Message queue:** SQS or Kafka (MSK)
- **Monitoring:** CloudWatch, X-Ray

Feel free to pick the ones that make sense for your design.

---

## Components
**Question:** *What major building blocks are needed (load balancers, web servers, caches, databases, queues)?*

**Interviewer:** Based on the requirements, the core components are:
- **DNS (Route 53):** For domain resolution and geo‑routing.
- **CDN (CloudFront):** Optional but recommended for caching static assets and potentially redirects for hot URLs.
- **Load Balancer (ALB):** Distributes incoming traffic across application servers; terminates SSL.
- **Application Servers (EC2/ECS/Lambda):** Stateless, handle shortening and redirection logic.
- **Cache (ElastiCache for Redis):** Stores frequently accessed mappings to reduce database load.
- **Database (DynamoDB):** Persistent storage for all URL mappings.
- **Message Queue (SQS) + Workers (optional):** For decoupling analytics processing from the main request path.

You may also include a **Key Generation Service (KGS)** as a separate component if you choose that approach for short code generation.

---

## Communication
**Question:** *Synchronous (HTTP/gRPC) or asynchronous (message queues)? For which parts?*

**Interviewer:**
- **Synchronous (HTTP):** All user‑facing operations – shortening and redirection – are synchronous. Clients expect immediate responses.
- **Asynchronous (message queue):** Analytics events (click tracking) should be sent asynchronously to avoid slowing down redirects. The redirect path publishes a message to a queue, and workers process it later.

So, the main request path is synchronous; the analytics pipeline is asynchronous.

---

## Geographical Distribution
**Question:** *Single region or multi‑region? Active‑active or active‑passive?*

**Interviewer:** The service is global, so we should aim for **multi‑region deployment** to reduce latency for users worldwide. However, for the core design, you can start with a **single region** and explain how you would extend to multiple regions.

If you go multi‑region:
- Use **Route 53 latency‑based or geo‑DNS** to route users to the nearest region.
- For the database, you have two options:
  - **Active‑passive:** One region handles writes, others are read‑replicas. Writes from other regions are forwarded to the primary. Simpler, but write latency is higher for users far from the primary.
  - **Active‑active:** All regions accept writes, with data replicated asynchronously between regions. More complex (conflict resolution needed), but lower write latency.

For a URL shortener, eventual consistency is acceptable, so **active‑active with DynamoDB Global Tables** (or Cassandra multi‑region) is a viable choice. You can mention both and discuss trade‑offs.

---

## Existing Services
**Question:** *Are there existing components we can reuse (e.g., CDN, identity provider, monitoring)?*

**Interviewer:** Yes, leverage AWS managed services as mentioned:
- **CDN:** CloudFront – can cache redirect responses for popular URLs.
- **Monitoring:** CloudWatch for metrics, logs, and alarms; X‑Ray for tracing.
- **Identity provider:** Not needed (public API).
- **Secrets management:** AWS Secrets Manager (if we later add API keys).
- **Auto‑scaling:** EC2 Auto Scaling groups or ECS service auto‑scaling.

You can also mention using **Terraform** or **CloudFormation** for infrastructure as code, but it's not required for the design discussion.

---

## Summary Table

| Aspect | Decision / Guidance |
|--------|---------------------|
| **Deployment environment** | AWS (managed services preferred) |
| **Core components** | DNS, CDN (optional), Load Balancer, App Servers, Cache, Database, Queue |
| **Communication** | Sync for user requests; async for analytics |
| **Geographical distribution** | Multi‑region (active‑active or active‑passive) – discuss trade‑offs |
| **Existing services** | Leverage CloudFront, Route53, ElastiCache, DynamoDB, SQS, CloudWatch |

---

## What the Candidate Should Do Next

Now that you have these architecture guidelines, you should:

1. **Draw a high‑level diagram** showing all components and how they interact.
2. **Explain the data flow** for both URL creation and redirection.
3. **Justify your component choices** based on the non‑functional requirements (scale, latency, availability).
4. **Discuss how you'd handle multi‑region** if you have time.
5. **Mention any trade‑offs** (e.g., using DynamoDB Global Tables adds complexity but improves global write latency).

Example:
> "I'll design a multi‑region active‑active architecture using Route53 for geo‑routing, CloudFront at the edge, and DynamoDB Global Tables for data replication. Each region has its own load balancer, stateless application servers, and a local Redis cache. Redirects are served from cache when possible; cache misses hit the local DynamoDB replica. Analytics events are published to a regional SQS queue and processed asynchronously. This gives us low latency worldwide and high availability."

This demonstrates you've considered all aspects of the architecture.



# Detailed Component Design – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about the detailed component design for a URL shortening service.

---

## Data Storage and Processing

### Caching
**Question:** *Should we use a cache (Redis/Memcached)? What data should be cached? Eviction policy?*

**Interviewer:** Yes, caching is essential to achieve low-latency redirects and handle high read QPS. Use **Redis** because it supports data structures, persistence, and clustering.

- **What to cache:** The mapping from short code to long URL. Cache only the most frequently accessed URLs – the "hot" set.
- **Cache size:** Based on Pareto principle, 20% of URLs generate 80% of traffic. With 3.3M new URLs/day, the total URLs after a year could be ~1.2B. But we only need to cache, say, the top 1 million URLs. Memory estimate: 1M × 500 bytes = 500 MB, plus overhead ~1 GB – easily fits in a Redis cluster.
- **Eviction policy:** **LRU (Least Recently Used)** – keeps recently accessed items, which aligns with access patterns (viral links may come and go). Redis supports `allkeys-lru` or `volatile-lru` with TTL.
- **Cache‑aside pattern:** Application checks cache first; on miss, reads from DB and updates cache.

### File/Object Storage
**Question:** *Do we need to store large files (images, videos)? Use S3, blob storage?*

**Interviewer:** **Not applicable** – a URL shortener only stores short strings (the long URLs). No large files. If we later add features like QR code generation, those images could be stored in S3, but for core design, it's out of scope.

### Search
**Question:** *Do we need full‑text search (Elasticsearch)? If so, how to index?*

**Interviewer:** **Not applicable** – there's no search functionality. Users don't search for URLs by content. If we later add a feature to search a user's own links, we could consider it, but it's not required now.

### Stream Processing
**Question:** *Do we need real‑time analytics or aggregation? Use Kafka, Flink?*

**Interviewer:** Yes, for **analytics** (click tracking), we need to collect and process events without impacting redirect latency. Use a **message queue** (Kafka or SQS) to decouple.

- **Events:** Each redirect produces a small event (short code, timestamp, referrer, user agent, IP). 
- **Processing:** Stream processors (e.g., Kafka Streams, Flink, or simple workers) consume events and aggregate counts (e.g., increment per‑URL counters, store raw data for later analysis).
- **Storage:** Aggregated results go into a fast query store (e.g., Redis for real‑time counts, or a time‑series DB like ClickHouse). Raw data may go to a data lake (S3) for batch analytics.

Real‑time is not critical; near‑real‑time (seconds) is fine.

---

## Specific Algorithms/Mechanisms

### ID Generation
**Question:** *How to generate unique IDs (UUID, Snowflake, database sequence)?*

**Interviewer:** We need to generate unique short codes (6–7 characters). Options:

- **Hashing (MD5, SHA-256) then take first N chars:** Decentralized but collisions possible; need collision resolution (re‑hash with salt). Works but adds complexity.
- **Pre‑generated random strings (Key Generation Service):** Pre‑generate a pool of random strings, store them, and hand them out. Ensures uniqueness without collisions. Good for high throughput.
- **Distributed ID generator (Snowflake) + Base62 encode:** Generate 64‑bit unique IDs (timestamp + machine ID + sequence), then encode in Base62 to get a short string. Guarantees uniqueness, no collisions, and the ID is roughly time‑sortable.

I'd lean toward the **Key Generation Service (KGS)** approach or **Snowflake + Base62**. Both are scalable. The KGS can be simpler to implement, while Snowflake gives you ordering and doesn't need a separate key DB. Choose one and justify.

If you use KGS, you'll need a small database to store unused keys and a service to hand them out in batches. If you use Snowflake, you need a coordination mechanism for worker IDs (e.g., ZooKeeper).

### Concurrency Control
**Question:** *How to handle concurrent writes/updates (optimistic locking, pessimistic locking)?*

**Interviewer:** For the core `INSERT` of a new URL, concurrency is not a big issue because each short code is unique. The only contention point is **custom aliases** – two users might try to claim the same custom alias simultaneously. To handle that, use **optimistic locking** with a conditional write:

- In DynamoDB, use a `ConditionExpression` to ensure the `short_code` does not already exist.
- In Cassandra, use lightweight transactions (`INSERT ... IF NOT EXISTS`).
- This gives you atomic uniqueness checks without explicit locks.

No need for pessimistic locking.

### Rate Limiting Algorithm
**Question:** *Token bucket, leaky bucket, sliding window? Where to implement (load balancer, API gateway)?*

**Interviewer:** Implement rate limiting at the **API gateway** or **load balancer** level to reject abusive requests early. Use **token bucket** algorithm – it's simple, allows bursts, and is easy to implement with Redis.

- **Key:** IP address (or API key if we had authentication).
- **Bucket size:** 100 tokens (allowing bursts up to 100).
- **Refill rate:** 100 tokens per hour (i.e., 100 requests per hour).

Store counters in Redis with a TTL. When a request arrives, check the token count; if >0, consume one; otherwise, return 429.

Sliding window log is more accurate but more complex. Token bucket is sufficient.

### Distributed Locking
**Question:** *If needed, how to implement (Redis, ZooKeeper)?*

**Interviewer:** For the core design, we may not need distributed locks. If we use a KGS, we need to ensure multiple KGS instances don't hand out the same key batch. That can be handled by having a single leader or using a database with conditional updates.

If we need distributed locks (e.g., for leader election), we could use **Redis with Redlock** or **ZooKeeper**. For simplicity, we can avoid locks by designing the KGS to use a database with atomic updates (e.g., each KGS instance marks a range of keys as taken using a transaction). Locks are not a primary concern for this design.

---

## Analytics and Monitoring

### Data to Collect
**Question:** *What metrics/logs are important (request count, latency, errors)?*

**Interviewer:** For operational monitoring:
- **Request count:** Total redirects per second, per short code (top N).
- **Latency:** Redirect latency (p50, p95, p99).
- **Error rates:** 404s (short code not found), 5xx errors.
- **Cache hit ratio:** Redis cache hit/miss.
- **System metrics:** CPU, memory, disk, network for app servers and databases.

For business analytics (optional but good to mention):
- **Click counts per URL** (total, over time).
- **Geographic distribution** (from IP).
- **Referrer domains**.
- **Device types** (from user agent).

### Storage for Analytics
**Question:** *Time‑series DB (InfluxDB, Prometheus), data warehouse (BigQuery)?*

**Interviewer:**
- **Operational metrics** can go to **Prometheus** + Grafana, or CloudWatch.
- **Click analytics** (raw events) can be stored in **S3** as Parquet/JSON for batch processing, and aggregated results in a **time‑series DB** like **ClickHouse** or **Elasticsearch** for fast querying.
- For simplicity, you could use **DynamoDB** to store aggregated counts per URL (e.g., increment a counter), but that adds write load. Better to decouple.

A common pattern: redirect → Kafka → Flink/Spark → ClickHouse for real‑time dashboards, and also S3 for archival.

### Alerting
**Question:** *What thresholds should trigger alerts?*

**Interviewer:**
- **High error rate:** e.g., >1% 5xx errors over 5 minutes.
- **High latency:** p99 redirect latency >200ms for 5 minutes.
- **Cache hit ratio drop:** below 80% (indicates cache may be misconfigured or thrashing).
- **Low disk space** on database nodes.
- **Rate limiting triggered heavily** (possible DDoS attack).
- **Service down** (health check failures).

Use CloudWatch alarms or Prometheus Alertmanager.

---

## Summary of Key Design Decisions

| Area | Decision |
|------|----------|
| **Caching** | Redis, cache‑aside, LRU eviction, cache hot URLs |
| **Analytics pipeline** | Kafka + stream processors + ClickHouse/S3 |
| **ID generation** | KGS or Snowflake + Base62 – justify |
| **Concurrency control** | Conditional writes (optimistic) for custom aliases |
| **Rate limiting** | Token bucket (Redis), 100 req/hour per IP |
| **Distributed locking** | Not needed for core; if needed, use Redis Redlock |
| **Metrics & alerts** | Prometheus/CloudWatch with standard thresholds |

---

## What the Candidate Should Do Next

Now that you have these detailed requirements, you should:

1. **Choose and justify your ID generation method** – explain how it works, how it scales, and how you avoid collisions.
2. **Describe the cache architecture** – Redis cluster size, eviction policy, and how you handle cache misses.
3. **Design the analytics pipeline** – what events you emit, how they're queued, processed, and stored.
4. **Explain rate limiting implementation** – where it lives, data structure, and how it scales.
5. **Discuss any trade‑offs** – e.g., using KGS introduces an extra service but guarantees no collisions; using hashing is simpler but may need collision resolution.

Example:
> "I'll use a Key Generation Service that pre‑generates batches of random 7‑character strings and stores them in a small database. Application servers request batches of 1000 keys from the KGS and serve them locally. This ensures uniqueness without database checks on every write. For caching, I'll use a Redis cluster with LRU eviction, following the cache‑aside pattern. Analytics events will be published to Kafka, consumed by Flink jobs that update ClickHouse tables for real‑time dashboards and also archive raw data to S3. Rate limiting will be implemented at the API gateway using token buckets stored in Redis."

This shows you've considered each component in depth.


# Trade-offs and Discussion – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about trade-offs for a URL shortening service. These are not definitive "correct" answers – they represent the kinds of constraints and preferences an interviewer might express, and they expect you to navigate these trade-offs in your design.

---

## Consistency vs. Availability
**Question:** *Where do we stand on the CAP theorem? What trade‑offs are we making?*

**Interviewer:** This is a fundamental trade-off. Given our requirements:
- We need **high availability** (99.99%) – redirection must never fail.
- We can tolerate **eventual consistency** for reads – it's okay if a newly created URL takes a few seconds to propagate.
- However, for **custom aliases**, we need strong consistency to ensure uniqueness.

**Where we stand:** We are **AP (Available + Partition Tolerant)** for the core redirect path. During a network partition, we will continue serving redirects (possibly stale) rather than returning errors. For custom aliases, we temporarily become **CP** by using conditional writes that require consensus.

**Trade‑off:** Users may occasionally see a 404 for a very fresh URL, but the system stays up. That's acceptable.

---

## Cost vs. Performance
**Question:** *Are we optimising for lower cost (e.g., fewer replicas, less caching) or higher performance?*

**Interviewer:** This is a public-facing service where user experience is critical. We should **optimise for performance** first, but within reasonable cost constraints. For example:
- We'll use caching (Redis) even though it adds cost, because it drastically reduces latency and database load.
- We'll use 3× replication for durability, even though it triples storage cost.
- We'll use auto-scaling to handle peaks without over-provisioning 24/7.

If you have ideas to reduce cost without sacrificing performance (e.g., using spot instances for non-critical workers), mention them. But the primary goal is performance.

**Trade‑off:** Higher performance means higher cost (more servers, more caching, more replicas). We accept that.

---

## Simplicity vs. Scalability
**Question:** *Should we start with a simpler design and plan to scale later, or build for extreme scale from day one?*

**Interviewer:** Given that we're designing for a hypothetical large-scale service (100M URLs/month), we should **build for scale from the start**. However, you can mention that in a real-world scenario, you might start simpler and evolve. For this interview, design for scale.

But also show you understand the trade‑off: building for scale adds complexity (sharding, distributed caching, multi-region). If you were building an MVP, you might start with a monolith and single database, then scale later. For our requirements, we need the scalable design.

**Trade‑off:** More complexity now, but avoids painful re-architecture later.

---

## Synchronous vs. Asynchronous
**Question:** *Which operations benefit from async processing? What complexity does it add?*

**Interviewer:** 
- **Synchronous (must be sync):** URL creation and redirection – users expect immediate responses.
- **Asynchronous (can be async):** Analytics tracking. Emit an event and process it later.

**Benefits of async:** Decouples analytics from the critical path, ensuring redirect latency isn't affected by analytics processing. Also allows buffering during traffic spikes.

**Complexity added:** Need a message queue (Kafka/SQS), consumers, and a separate analytics storage. Must handle duplicate events (idempotency) and ensure no data loss.

**Trade‑off:** More moving parts, but better performance and reliability for the core feature.

---

## SQL vs. NoSQL
**Question:** *Why choose one over the other given the access patterns?*

**Interviewer:** Based on our access patterns (simple key-value lookups, high throughput, need for horizontal scaling), **NoSQL is the better fit**. Reasons:
- **Scalability:** NoSQL databases like DynamoDB or Cassandra scale horizontally out of the box. SQL can scale but requires manual sharding, which adds complexity.
- **Performance:** NoSQL offers consistent low-latency reads/writes at scale.
- **Data model:** We don't need joins, complex transactions, or relationships. A key-value store is perfect.

**If you chose SQL:** You'd need to justify it – perhaps for strong consistency or because the team is more familiar with SQL. But you'd then have to explain how you'd shard (e.g., application-level sharding based on short_code) and handle the complexity. Both are valid, but NoSQL is the more natural fit.

**Trade‑off:** NoSQL may lack some features (like secondary indexes) – but we can add GSIs or denormalise.

---

## Denormalisation vs. Normalisation
**Question:** *When to denormalise for performance?*

**Interviewer:** For a URL shortener, denormalisation is relevant in a few places:
- **Caching:** We're denormalising by storing the short_code → long_url mapping in Redis, separate from the primary store.
- **Analytics:** We might store aggregated click counts directly in the URL mappings table (denormalised) to avoid joins. This adds a write operation on every click but makes reads faster.
- **User listings:** If we need to list a user's URLs, we could store the same data in two tables: one keyed by short_code, another keyed by user_id (duplication). This is denormalisation for query efficiency.

**General principle:** Denormalise when read performance is critical and you can tolerate some duplication and eventual consistency. For our core, we denormalise by caching. For analytics, we might keep counts separate or denormalise – it's a trade‑off.

**Trade‑off:** Denormalisation speeds up reads but makes writes more complex and can lead to inconsistency if not managed carefully.

---

## Summary of Trade‑off Preferences

| Trade‑off | Our Preference |
|-----------|----------------|
| **Consistency vs. Availability** | Availability first; strong consistency only for uniqueness |
| **Cost vs. Performance** | Performance first, but be cost-aware |
| **Simplicity vs. Scalability** | Build for scale from the start |
| **Synchronous vs. Asynchronous** | Sync for user actions; async for analytics |
| **SQL vs. NoSQL** | NoSQL (DynamoDB/Cassandra) |
| **Denormalisation vs. Normalisation** | Denormalise for performance where needed |

---

## What the Candidate Should Do Next

Now that you understand the trade‑offs, you should:

1. **Incorporate these trade‑offs into your final design summary** – show that you've considered alternatives and made conscious choices.
2. **Be prepared to defend your choices** – if the interviewer pushes back (e.g., "Why not use SQL?"), explain your reasoning.
3. **Mention how you'd handle the downsides** – e.g., if you chose eventual consistency, explain how you'd handle the rare case of a stale read (maybe a "read-after-write" consistency option for users who just created a link).

Example:
> "I've chosen to prioritise availability and performance over strong consistency, which is why I'm using DynamoDB with eventual consistency for reads. For custom aliases, I'll use conditional writes to ensure uniqueness. I'm accepting that a newly created URL might not be immediately available in all regions, but that's a rare edge case. To mitigate it, I could implement read-after-write consistency for the user who just created the link by reading from the primary region. This is a trade‑off I'm comfortable with."

This shows you understand the nuances of system design.

# Failure Scenarios and Mitigations – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about failure scenarios and mitigations for a URL shortening service.

---

## Single Points of Failure
**Question:** *Which components could bring down the system? How to eliminate them (redundancy, failover)?*

**Interviewer:** We need to identify potential single points of failure at each layer:
- **DNS:** Could be a SPOF if we rely on a single DNS provider. Use multiple DNS providers or Route 53's highly available design.
- **Load Balancer:** A single load balancer instance can fail. Use a multi-AZ load balancer (e.g., AWS ALB is regional and inherently redundant).
- **Application Servers:** Individual servers can fail. Run multiple instances in an auto-scaling group across availability zones.
- **Cache (Redis):** A single Redis node can fail. Use Redis Cluster with replicas or ElastiCache for Redis with multi-AZ automatic failover.
- **Database:** A single database instance is a SPOF. Use managed databases with multi-AZ replication (e.g., DynamoDB is already multi-AZ; Cassandra with replication factor 3 across AZs).
- **Message Queue:** Kafka or SQS are inherently distributed – single broker failure should not stop the system if configured with replication.

The key is **redundancy at every layer** and **automatic failover** so that no single component failure causes downtime.

---

## Database Failure
**Question:** *What if the primary database goes down? (Replica promotion, multi‑AZ)*

**Interviewer:** Our database choice should handle this automatically:
- If using **DynamoDB**, it's a managed service with multi-AZ replication by default. If one replica fails, requests are automatically routed to other replicas with no downtime.
- If using **Cassandra**, we configure a replication factor of 3 across multiple availability zones. If a node goes down, reads and writes can still achieve quorum (e.g., consistency level = LOCAL_QUORUM) using the remaining nodes. The failed node will be replaced automatically.
- If using **SQL** with a primary and replicas, we'd need automated failover (e.g., RDS Multi-AZ with automatic failover to standby). Writes would be redirected to the new primary.

In any case, we design so that a single database node failure does not cause downtime – reads and writes continue (maybe with slightly reduced capacity) while the system recovers.

---

## Cache Failure
**Question:** *Can the system survive a cache outage? Fallback to database?*

**Interviewer:** Yes, the cache is for performance, not availability. If Redis goes down:
- All redirect requests will miss the cache and hit the database directly.
- This will increase latency and database load, but the system remains operational.
- Once Redis recovers, the cache will gradually warm up again.

To minimise impact:
- Use Redis Cluster with replicas so a single node failure doesn't take down the entire cache.
- Consider a multi-AZ Redis deployment for automatic failover.
- Monitor cache health and alert immediately.

The database must be provisioned to handle the full read load during a cache outage (possibly with reduced performance, but still functional).

---

## Service Overload
**Question:** *How to handle traffic spikes (auto‑scaling, rate limiting, queueing)?*

**Interviewer:** Multiple layers of protection:
- **Auto‑scaling:** Application servers should scale out based on CPU or request count. Use target tracking scaling policies.
- **Rate limiting:** Already implemented at the API gateway to reject excessive requests per IP. This protects against both abuse and unintentional spikes.
- **Caching:** Absorbs a large portion of read traffic; hot URLs are served from cache, reducing load on app servers and database.
- **Database:** DynamoDB auto-scales read/write capacity based on load. Cassandra can handle spikes if provisioned appropriately.
- **Queueing:** For non‑critical operations (analytics), we use a queue to buffer events during spikes; workers process at their own pace.
- **CDN:** For viral URLs, we can cache redirect responses at the edge (CloudFront) so that even the app servers are bypassed.

The goal is to have **graceful degradation** – the system may become slower but should not fail completely.

---

## Network Partitions
**Question:** *How does the system behave during a network split? (CAP trade‑off)*

**Interviewer:** During a network partition, we must choose between consistency and availability. Based on our earlier trade‑off, we **prioritise availability**:
- Reads may return stale data from the local replica.
- Writes (new URLs) may be accepted in the partition, but if the partition affects the database quorum, some writes may fail or be queued.
- For custom aliases that require uniqueness, writes may fail if they cannot achieve consensus.

In a multi‑region active‑active setup with DynamoDB Global Tables, writes are eventually replicated across regions; during a partition, each region continues to accept writes, and conflicts are resolved via last‑write‑wins.

Our design accepts that during a partition, the system remains available but may serve stale data or have temporary inconsistencies – which is acceptable for a URL shortener.

---

## Data Corruption
**Question:** *How to recover from accidental deletions or corruption? (Backups, point‑in‑time recovery)*

**Interviewer:** We need both **backups** and **point‑in‑time recovery (PITR)**:
- **DynamoDB:** Enable PITR to restore to any point in the last 35 days. Also take daily on‑demand backups to S3 for long‑term retention.
- **Cassandra:** Use `nodetool snapshot` regularly and store backups in S3. Can also use incremental backups.
- **Application‑level protection:** Implement soft deletes? Possibly not for core, but for user‑facing deletes, we could mark as deleted rather than physically removing data, allowing recovery.

In case of accidental deletion or corruption, we can restore the affected data from backups or PITR, possibly with some data loss (depending on recovery point).

---

## Regional Outage
**Question:** *If multi‑region, how does failover work? (DNS, global load balancer)*

**Interviewer:** For multi‑region deployment, we need:
- **DNS failover:** Use Route 53 with health checks on each region's load balancer. If a region becomes unhealthy, Route 53 automatically routes traffic to the remaining healthy regions.
- **Database replication:** For active‑active, each region already has its own database (DynamoDB Global Tables). For active‑passive, we promote a replica in another region to primary.
- **Application servers:** Auto‑scaling in each region ensures capacity.

During a regional outage:
1. Health checks fail for that region's endpoints.
2. Route 53 stops sending traffic to that region.
3. All traffic is directed to other regions.
4. If the database in the failed region was the primary (in active‑passive), we promote a replica in another region to primary (automated via database failover).
5. Users may experience slightly higher latency but continue to use the service.

This requires designing for cross‑region data replication and ensuring that applications can connect to any region.

---

## DDoS Attacks
**Question:** *What mechanisms protect against DDoS? (Rate limiting, WAF, CDN)*

**Interviewer:** Defence in depth:
- **CDN (CloudFront):** Absorbs many types of DDoS attacks by caching content at the edge and filtering malicious traffic before it reaches origin.
- **Web Application Firewall (WAF):** Can block malicious requests (SQL injection, XSS) and implement rate‑based rules.
- **Rate limiting:** Already implemented per IP at the API gateway to limit request rates.
- **AWS Shield (or similar):** Provides automatic DDoS protection at the network layer.
- **Auto‑scaling:** Helps absorb traffic spikes, but DDoS can still overwhelm; rate limiting and WAF are first lines of defence.
- **Monitoring and alerting:** Detect unusual traffic patterns and trigger mitigation.

For a URL shortener, most DDoS would target the redirection endpoint. Since that's cheap to serve (cached responses), we can handle moderate attacks, but large‑scale attacks may still overwhelm. The above measures reduce risk.

---

## Summary of Mitigation Strategies

| Failure Scenario | Mitigation |
|------------------|------------|
| **Single point of failure** | Redundancy at all layers (multi‑AZ, multiple instances) |
| **Database failure** | Multi‑AZ replication, automatic failover |
| **Cache failure** | Redis replication, fallback to database |
| **Traffic spikes** | Auto‑scaling, rate limiting, CDN, queueing |
| **Network partition** | AP design (eventual consistency), local reads/writes |
| **Data corruption** | Backups, point‑in‑time recovery |
| **Regional outage** | Multi‑region deployment, DNS failover |
| **DDoS attack** | CDN, WAF, rate limiting, AWS Shield |

---

## What the Candidate Should Do Next

Now that you understand the failure scenarios and expected mitigations, you should:

1. **Incorporate these mitigations into your architecture diagram** – show redundancy, multi‑AZ, etc.
2. **Explain how the system behaves during each failure** – e.g., "If the cache fails, redirects fall back to database, increasing latency but the system stays up."
3. **Discuss trade‑offs** – e.g., multi‑region adds complexity and cost but improves availability.
4. **Be prepared to dive deeper** – e.g., how exactly does DynamoDB handle a node failure? How does Route 53 health checking work?

Example:
> "To handle database failure, I'll use DynamoDB with multi‑AZ replication – it's fully managed and automatically routes requests away from unhealthy replicas. For cache failure, I'll deploy Redis Cluster with replicas; if the entire cluster fails, traffic falls back to DynamoDB. I'll also enable point‑in‑time recovery for DynamoDB to guard against accidental deletions. For regional outages, I'll use Route 53 health checks to route traffic away from an unhealthy region, and DynamoDB Global Tables to keep data in sync across regions."

This shows you've thought about resilience at every level.


# Scaling to the Next Level – URL Shortener (Interviewer's Answers)

Here are the answers an interviewer might give to your clarifying questions about taking the URL shortener to extreme scale. These are advanced topics that demonstrate deeper system design knowledge.

---

## What if Traffic Grows 100×?
**Question:** *What if traffic grows 100×? Which components become bottlenecks first?*

**Interviewer:** Let's assume our current scale is 100M new URLs/month and 10B redirects/month. A 100× increase would mean:
- **10B new URLs/month** (333M/day, 3,850 writes/sec average)
- **1 trillion redirects/month** (33B/day, 385,000 reads/sec average)
- **Storage:** 1.67 GB/day × 100 = 167 GB/day → 61 TB/year × 3 replication = 183 TB/year

The first bottlenecks would be:

1. **Database reads:** Even with caching, cache misses would generate 385K reads/sec to the database. DynamoDB can scale to millions of reads/sec with enough capacity, but cost becomes significant.
2. **Cache cluster:** Redis would need to scale horizontally (cluster mode) to handle 385K reads/sec and store a larger working set. Memory would need to be increased significantly.
3. **Application servers:** Would need to scale out to handle the request volume – but they're stateless, so auto-scaling can handle it.
4. **Network bandwidth:** Redirects would generate ~77 GB/sec egress (385K/sec × 200 bytes). This would require multiple 100 GbE links and significant cost.
5. **Key Generation Service:** Would need to generate 3,850 new keys per second – still feasible with pre-generated batches.
6. **Analytics pipeline:** 385K events/sec would require a robust Kafka cluster and stream processing infrastructure.

The **database reads** and **cache** would likely be the first to show strain if not properly scaled.

---

## Database Sharding
**Question:** *Would we need to re‑shard? How to minimise downtime during rebalancing?*

**Interviewer:** If we started with a fixed number of shards (e.g., 8 shards in Cassandra), at 100× scale we would need more shards to distribute the load and data. Re‑sharding (or rebalancing) is necessary.

**Approaches to minimise downtime:**

1. **Consistent hashing with virtual nodes (Cassandra):** Adding new nodes automatically redistributes data using consistent hashing. Only a fraction of data (1/N) moves per new node. This can happen online with minimal impact.

2. **DynamoDB:** Automatic partitioning – as throughput or storage increases, DynamoDB splits partitions automatically with zero downtime. No manual re‑sharding needed.

3. **Pre‑sharding (if using SQL):** Start with many more shards than needed (e.g., 1,000 logical shards) mapped to fewer physical nodes. When you need to scale, you move logical shards to new physical nodes. This avoids rehashing keys – just move data.

4. **Range‑based sharding with dynamic splitting:** Use a shard key like `short_code` (which is roughly random) and split ranges when they get too large. Requires a metadata service to track which range belongs to which shard.

**Minimising downtime:**
- Perform rebalancing in the background during low traffic.
- Use **read‑only mode** on shards being moved, or use **dual‑writing** (write to both old and new location during migration).
- Use database tools that support online rebalancing (Cassandra, DynamoDB, Citus).

For our design with DynamoDB, this is handled automatically – one less thing to worry about.

---

## Multi‑Region Active‑Active
**Question:** *How to handle cross‑region replication and conflict resolution?*

**Interviewer:** At extreme scale, multi‑region active‑active becomes essential for low latency and availability. Key challenges:

**Cross‑region replication:**
- **DynamoDB Global Tables:** Automatically replicates data across selected regions with last‑writer‑wins conflict resolution. You just enable it.
- **Cassandra:** Can be configured for multi‑region with replication factor per region. Use `NetworkTopologyStrategy` and set `LOCAL_QUORUM` for reads/writes within a region, with asynchronous replication across regions.
- **Custom solution:** Use Kafka to replicate change data capture (CDC) events between regions.

**Conflict resolution strategies:**
- **Last‑write‑wins (LWW):** Simplest; uses timestamps. But relies on clock synchronisation (use version numbers or logical clocks).
- **Version vectors / vector clocks:** Track causal history; conflicts are returned to the application to resolve. More complex.
- **CRDTs (Conflict‑free Replicated Data Types):** Data types that automatically merge (e.g., counters, sets). For a URL shortener, a simple LWW is sufficient because each short code is unique and never updated (except maybe expiration).

**Consistency considerations:**
- Reads in each region are eventually consistent with writes from other regions.
- For custom aliases, you might need to check all regions or use a single region for uniqueness (defeating active‑active). A compromise: use a hash of the custom alias to determine a "home region" for that alias, and always route requests for that alias to its home region.

At our scale, we'd likely use DynamoDB Global Tables for simplicity.

---

## Advanced Caching
**Question:** *Would we use CDN for dynamic content, edge computing (CloudFlare Workers)?*

**Interviewer:** Yes, at extreme scale we can push caching even closer to users:

**CDN for redirects:**
- Configure CloudFront (or other CDN) to cache HTTP 301/302 responses for popular short URLs. The CDN edge returns the redirect without hitting our origin.
- Cache key = short code. Set TTL based on popularity – longer for extremely hot URLs.
- This can absorb a huge percentage of read traffic (80%+).

**Edge computing (CloudFlare Workers / Lambda@Edge):**
- Run lightweight code at the edge to handle redirects without even hitting our regional load balancers.
- Worker can check a local edge cache, then if miss, fetch from regional cache or database via a faster path.
- Can also handle URL creation? Possibly, but creation is lower volume and might still need origin.

**Benefits:**
- Reduces latency to single‑digit milliseconds worldwide.
- Offloads traffic from origin infrastructure – huge cost savings.
- Scales automatically with CDN's global network.

**Challenges:**
- Cache invalidation – if a URL needs to be updated or expires, we need to purge CDN caches (can be done via API).
- Consistency – edge cache may serve stale data for a short time (acceptable for this use case).

At 100× scale, this becomes almost mandatory to keep costs manageable and latency low.

---

## Microservices vs. Monolith
**Question:** *Would we split into microservices? How would they communicate?*

**Interviewer:** For a URL shortener, the core functionality is simple – it could remain a **monolith** even at large scale. However, as we add features, we might split:

**Potential microservice boundaries:**
- **URL Service:** Core shortening and redirection.
- **Analytics Service:** Collects and serves click statistics.
- **User Service:** Manages user accounts (if added).
- **Key Generation Service:** Dedicated service for handing out short codes.
- **Expiration Service:** Background service to delete/archive expired URLs.

**Communication:**
- **Synchronous (HTTP/gRPC):** For request‑response between services (e.g., URL Service calls User Service to validate API key). Use gRPC for low latency and efficiency.
- **Asynchronous (events/messages):** For decoupled workflows (e.g., URL Service emits "URL created" event; Analytics Service consumes it). Use Kafka.

**API Gateway:** Single entry point that routes requests to appropriate services.

**Trade‑offs:**
- **Pros:** Independent scaling, team autonomy, technology diversity.
- **Cons:** Increased complexity, latency, operational overhead.

At our scale, a well‑factored monolith could still work, but if we have multiple teams, microservices make sense. For this design, we can mention that we'd likely keep it as a modular monolith initially, then split if needed.

---

## Summary of Extreme Scaling Strategies

| Challenge | Solution |
|-----------|----------|
| **100× traffic growth** | Auto‑scaling, CDN for redirects, edge computing, database auto‑partitioning |
| **Database re‑sharding** | Use DynamoDB (auto‑partitioning) or Cassandra (consistent hashing) to avoid manual re‑sharding |
| **Multi‑region active‑active** | DynamoDB Global Tables or Cassandra multi‑region; LWW conflict resolution |
| **Advanced caching** | CDN for redirects, edge workers, multi‑layer caching |
| **Microservices** | Split by domain (URL, Analytics, User, KGS); communicate via gRPC and Kafka |

---

## What the Candidate Should Do Next

Now that you've discussed extreme scaling, you should:

1. **Summarise how your design would evolve** – what you'd add or change to handle 100× traffic.
2. **Highlight the most critical bottlenecks** and how you'd address them.
3. **Show you understand the trade‑offs** – e.g., edge computing adds cache invalidation complexity but dramatically reduces latency and cost.

Example:
> "At 100× scale, I'd first push redirects to the edge using CloudFront with Lambda@Edge, caching popular redirects and handling requests close to users. This would absorb the majority of read traffic. I'd rely on DynamoDB's automatic partitioning to handle database scaling, and use Global Tables for multi‑region active‑active replication with last‑write‑wins conflict resolution. For analytics, I'd scale Kafka to handle millions of events per second and use stream processors to aggregate data in real time. The core URL service might remain a monolith, but I'd consider splitting out the Key Generation Service and Analytics Service as separate microservices if team size warranted it."

This demonstrates you can think beyond the initial design and anticipate future challenges.
