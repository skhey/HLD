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
