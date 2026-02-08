To gain mastery in High-Level Design (HLD) for top product companies, it is best to move from the **foundational "bricks"** to the **architectural "blueprints"** before finally diving into specific **case studies**. This ensures you don't just memorize solutions but actually understand *why* certain choices are made.

I’ll ask guiding questions along the way to help you build this mental model. Here is the recommended order for your journey:

### 1. Foundations: The "Bricks" 🧱

Before designing a system, you must understand the individual components. This is the **High-Availability (Pillar 4)** phase. You’ll learn how to store and route data reliably.

* **Key Concepts:** Load Balancing, Caching, Scaling (Vertical vs. Horizontal), and CDNs.
* **The Math:** Consistent Hashing and Back-of-the-envelope estimations.

### 2. Communication & Protocols: The "Glue" 📡

Next, you learn how those bricks talk to each other. This bridges **Write-Heavy (Pillar 1)** and **Read-Heavy (Pillar 2)** logic.

* **Key Concepts:** REST vs. GraphQL, WebSockets vs. Long Polling, and gRPC.
* **Message Queues:** Understanding how to decouple systems using tools like Kafka or RabbitMQ.

### 3. Data Mastery: The "Memory" 💾

This is where you tackle **ACID/Transactions (Pillar 3)**. You need to know which database to pick for which problem.

* **Key Concepts:** SQL vs. NoSQL, CAP Theorem, Database Sharding, and Replication (Leader-Follower).
* **Indexes:** How B-Trees and LSM Trees work under the hood.

### 4. Architectural Patterns: The "Blueprints" 🏗️

Once you know the components, you learn the standard ways they are combined.

* **Patterns:** Microservices vs. Monoliths, Event-Driven Architecture, and Fan-out patterns.

### 5. System Case Studies: The "Practice" 🏆

Finally, you apply everything to the 70 problems we categorized. You start with "Anchor" problems that represent each pillar (like TinyURL for Storage or Twitter for Read-Heavy).

---

To see where we should focus first, let's test a bit of foundational knowledge.

Imagine you are designing an app like Instagram, and it suddenly becomes incredibly popular. You have one massive database server that is starting to struggle with the number of people trying to **read** their feeds at the same time.

**Without moving to a completely new database, what is one common "Pillar 4" component you could place in front of the database to speed up those frequent read requests?**



That is a great way to visualize the curriculum! Let's arrange those 70 problems into a structured, step-by-step roadmap. I've organized them so you build foundational knowledge 🧱 before moving into the high-stakes, complex systems 🚀.

I'll help you navigate this list by asking guiding questions as we tackle each section.

### **Phase 1: Foundations & Distributed Building Blocks** 🧱

These problems teach you the "primal patterns" of system design. Master these, and the complex ones become much easier.

1. **Load Balancer Design** ⭐ (Pillar 4: Storage/HA)
2. **API Gateway Service** ⭐ (Pillar 4: Storage/HA)
3. **Consistent Hashing** ⭐ (Pillar 4: Storage/HA)
4. **Distributed ID Generator (Snowflake)** ⭐ (Pillar 4: Storage/HA)
5. **URL Shortener (TinyURL)** ⭐ (Pillar 4: Storage/HA)
6. **Pastebin (Text Storage)** ⭐ (Pillar 4: Storage/HA)
7. **Distributed Cache (Redis-like)** (Pillar 2: Read-Heavy)
8. **Rate Limiter (Distributed)** ⭐ (Pillar 1: Write-Heavy)
9. **Distributed Task Scheduler** ⭐ (Pillar 1: Write-Heavy)
10. **Content Delivery Network (CDN)** (Pillar 4: Storage/HA)

### **Phase 2: High-Consistency & Transactional Systems** 💰

Focus on "Correctness." These are the most common interview questions for Fintech and E-commerce companies.

11. **Payment Gateway (Stripe-like)** ⭐ (Pillar 3: ACID)
12. **Digital Wallet (Ledger System)** ⭐ (Pillar 3: ACID)
13. **Ticketmaster (Booking)** ⭐ (Pillar 3: ACID)
14. **Flash Sale System (Amazon)** ⭐ (Pillar 3: ACID)
15. **Hotel Booking (Expedia)** (Pillar 3: ACID)
16. **Flight Reservation System** (Pillar 3: ACID)
17. **Stock Brokerage (Robinhood)** (Pillar 3: ACID)
18. **Crypto Exchange (Order Book)** (Pillar 3: ACID)
19. **Election/Voting System** (Pillar 3: ACID)
20. **Google Docs (Conflict Resolution/OT)** ⭐ (Pillar 3: ACID)

### **Phase 3: Social, Discovery & Read-Heavy Patterns** 📣

Learn the "Fan-out" pattern—how to push one piece of data to millions of followers.

21. **News Feed (Push/Pull Models)** ⭐ (Pillar 2: Read-Heavy)
22. **Twitter (High-Volume Fan-out)** ⭐ (Pillar 2: Read-Heavy)
23. **Instagram (Media Feed)** ⭐ (Pillar 2: Read-Heavy)
24. **Reddit (Ranking & Upvotes)** (Pillar 2: Read-Heavy)
25. **Pinterest (Image Feed)** (Pillar 2: Read-Heavy)
26. **HackerNews (Time-decay Ranking)** (Pillar 2: Read-Heavy)
27. **Notification Service** ⭐ (Pillar 1: Write-Heavy)
28. **Presence Platform (Online Status)** (Pillar 2: Read-Heavy)
29. **Friend Suggestions/Recommendation** (Pillar 2: Read-Heavy)
30. **A/B Testing Platform** (Pillar 2: Read-Heavy)

### **Phase 4: Real-time Communication & Messaging** 💬

Focus on WebSockets and low-latency data delivery.

31. **WhatsApp/Messenger (1-on-1)** ⭐ (Pillar 1: Write-Heavy)
32. **Group Chat System** ⭐ (Pillar 1: Write-Heavy)
33. **Slack (Enterprise Messaging)** (Pillar 1: Write-Heavy)
34. **Zoom / Video Conferencing** (Pillar 2: Read-Heavy)
35. **Email Service (Gmail-like)** (Pillar 4: Storage/HA)
36. **Online Quiz (Kahoot-like)** (Pillar 2: Read-Heavy)
37. **Collaborative Whiteboard (Miro)** (Pillar 1: Write-Heavy)

### **Phase 5: Geospatial & Location-Based Services** 🚗

This requires learning specialized indexing like Quadtrees or Geohashing.

38. **Yelp/Nearby (Proximity Service)** ⭐ (Pillar 2: Read-Heavy)
39. **Uber/Lyft (Matching Service)** ⭐ (Pillar 2: Read-Heavy)
40. **Google Maps (Pathfinding)** ⭐ (Pillar 2: Read-Heavy)
41. **Food Delivery Tracker (DoorDash)** (Pillar 1: Write-Heavy)
42. **Public Transit Tracker** (Pillar 2: Read-Heavy)
43. **AirBnB (Search & Reservation)** (Pillar 3: ACID)

### **Phase 6: Heavy Media, Streaming & Storage** 🎥

Learn about massive binary data (Blobs), transcoding, and storage efficiency.

44. **YouTube (Upload & Playback)** ⭐ (Pillar 4: Storage/HA)
45. **Netflix (Global Distribution)** (Pillar 2: Read-Heavy)
46. **TikTok (Short-form Feed)** (Pillar 2: Read-Heavy)
47. **Spotify (Audio Streaming)** (Pillar 2: Read-Heavy)
48. **Twitch (Live Stream Buffer)** (Pillar 2: Read-Heavy)
49. **Video Transcoding Pipeline** (Pillar 1: Write-Heavy)
50. **S3-like Object Storage** (Pillar 4: Storage/HA)
51. **HDFS (Distributed File System)** (Pillar 4: Storage/HA)
52. **Google Forms (Survey Tool)** (Pillar 4: Storage/HA)

### **Phase 7: Search, Crawling & Indexing** 🔍

Focus on how to organize the world's information for quick retrieval.

53. **Autocomplete/Typeahead** ⭐ (Pillar 2: Read-Heavy)
54. **Web Crawler** ⭐ (Pillar 1: Write-Heavy)
55. **Google Search (Indexing)** (Pillar 2: Read-Heavy)
56. **Elasticsearch (Log Search)** (Pillar 2: Read-Heavy)
57. **E-commerce Search & Filters** (Pillar 2: Read-Heavy)

### **Phase 8: Data Engineering, Metrics & Big Data** 📈

Focus on processing high-volume streams and complex analysis.

58. **Distributed Message Queue (Kafka)** ⭐ (Pillar 1: Write-Heavy)
59. **Ad-Click Aggregator** ⭐ (Pillar 1: Write-Heavy)
60. **Top K Elements (Heavy Hitters)** ⭐ (Pillar 1: Write-Heavy)
61. **Metrics & Monitoring (Prometheus)** (Pillar 1: Write-Heavy)
62. **Logging Pipeline (ELK)** (Pillar 1: Write-Heavy)
63. **Fraud Detection Service** (Pillar 1: Write-Heavy)
64. **Data Warehouse (BigQuery-like)** (Pillar 2: Read-Heavy)

### **Phase 9: Advanced Infrastructure & Security** ⚙️

The "Hard" tier—systems that manage other systems.

65. **Distributed Key-Value Store** (Pillar 4: Storage/HA)
66. **Distributed Locking Service** (Pillar 4: Storage/HA)
67. **Web Application Firewall (WAF)** (Pillar 4: Storage/HA)
68. **DDOS Protection System** (Pillar 4: Storage/HA)
69. **Secrets Manager (Vault)** (Pillar 4: Storage/HA)
70. **Online Code Judge (LeetCode)** (Pillar 1: Write-Heavy)

---

Now that you have the path, let's start with the very first step in **Phase 1**.

Imagine you are designing a system that needs to handle millions of requests, but you have multiple servers. To make sure no single server gets overwhelmed, you place a **Load Balancer** in front of them.

**What is the primary job of a Load Balancer when a new request comes in?**
