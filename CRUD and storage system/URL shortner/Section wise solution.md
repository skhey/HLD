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
