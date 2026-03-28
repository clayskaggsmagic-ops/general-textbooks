# Chapter 29: The Building Blocks — Deep Dives for System Design

> *"Knowing THAT you need a cache is entry-level. Knowing WHEN to use write-through vs. cache-aside, what eviction policy to choose, and what happens when your cache node fails — that's what gets you hired."*

---

## Why This Chapter Matters

Chapter 28 gave you the methodology — how to run the 45-minute interview. This chapter gives you the **technical depth** to fill that methodology with substance. For each building block, you get: what it is, when to use it, the trade-offs, and **exactly what to say in the interview** to demonstrate mastery.

---

## 29.1 Databases — The Foundation of Every System

### SQL vs. NoSQL — The Decision Script

This is the single most common question in system design interviews. Here's exactly how to answer it:

```
INTERVIEWER: "Why did you choose this database?"

STRONG ANSWER (SQL):
"This system has complex relationships — users follow users,
users write tweets, tweets have replies. I need JOINs to
answer queries like 'show me all tweets from people User A
follows.' I also need ACID transactions for operations like
'transfer money from Account A to Account B' — both the
debit and credit must succeed together or neither does.
PostgreSQL gives me both: relational queries and transactional
integrity. The trade-off is that horizontal scaling is harder
than with NoSQL, but with read replicas and eventual sharding,
it handles our projected scale."

WHY THIS ANSWER WORKS: You named the SPECIFIC features
(JOINs, ACID), tied them to SPECIFIC use cases (follow
relationships, money transfers), and acknowledged the
trade-off (harder horizontal scaling) with your mitigation
strategy (replicas, sharding).

STRONG ANSWER (NoSQL):
"This workload is write-heavy — 50,000 writes per second
of sensor data — with simple access patterns. Every query
is 'give me all readings from Device X in the last hour.'
There are no JOINs, no complex relationships. Cassandra is
designed exactly for this: high write throughput with
partition-key-based reads. It scales horizontally by adding
nodes — I don't need to manage sharding manually. The
trade-off is eventual consistency and no ad-hoc queries,
but for IoT sensor data, that's acceptable."

WEAK ANSWER (what NOT to say):
❌ "I'd use MongoDB because it's popular."
❌ "NoSQL is faster than SQL."
❌ "I'd use Cassandra because it scales." (Too vague — WHY
    does your workload need Cassandra's specific properties?)
```

### The Decision Framework — Memorize This

```
CHOOSE SQL WHEN:                  CHOOSE NoSQL WHEN:
• ACID transactions needed        • 100K+ writes/sec
• Complex JOINs required          • Simple key-value lookups
• Schema is stable                • Schema varies by record
• Flexible ad-hoc queries         • Can tolerate eventual consistency
• Consistency > raw throughput     • Need horizontal scaling built-in

Examples:                         Examples:
PostgreSQL, MySQL,                DynamoDB, Cassandra,
CockroachDB (distributed SQL)    MongoDB, Redis
```

### The Four NoSQL Families — Quick Recognition Guide

```
1. KEY-VALUE: Redis, DynamoDB
   Access: GET/SET by key. No queries on values.
   Say it: "I need sub-millisecond lookups by a known key —
   perfect for caching session data or precomputed feeds."

2. DOCUMENT: MongoDB, Firestore
   Access: Query by any field in a JSON document.
   Say it: "Each record has a different schema — user profiles
   with varying fields. A document store handles this naturally."

3. WIDE-COLUMN: Cassandra, HBase, ScyllaDB
   Access: Fast reads/writes by partition key. Massive write throughput.
   Say it: "Time-series data with millions of writes per second.
   Cassandra's partition key model lets me query by device ID
   and time range efficiently."

4. GRAPH: Neo4j, Amazon Neptune
   Access: Traverse relationships efficiently.
   Say it: "Social graph queries — 'friends of friends who like
   topic X' — require traversing 5+ levels of relationships. A
   SQL JOIN at depth 5 is O(n^5). A graph traversal is O(1) per hop."
```

### Indexing — The Depth That Separates Good from Great

```
WHAT TO SAY WHEN YOU DESIGN A SCHEMA:

"I'm adding a composite index on tweets(author_id, created_at
DESC). Here's why: the most frequent query is 'give me User X's
most recent tweets.' Without this index, the database does a
full table scan — O(n) on potentially billions of rows. With
the index, it jumps directly to User X's tweets and scans in
reverse chronological order — O(log n) to find the starting
point, then O(k) to read k results. The trade-off is that
every INSERT must also update the index, but at 3,000 writes
per second, that's negligible."

INDEX TYPES YOU SHOULD KNOW:
• B-TREE (default): range queries, ORDER BY, equality
• HASH: exact equality only, O(1) lookup
• COMPOSITE: index on (A, B) — useful for WHERE A=x AND B>y
  Important: leftmost prefix rule — the index on (A,B) helps
  queries on A alone, but NOT queries on B alone.
• COVERING: index contains ALL columns the query needs. The
  query never touches the main table — answered purely from the index.
```

### ACID and Isolation Levels — What to Say

```
ACID:
A = Atomicity:   All operations succeed or none do.
C = Consistency:  Database moves between valid states.
I = Isolation:    Concurrent transactions don't interfere.
D = Durability:   Committed data survives crashes.

ISOLATION LEVELS (weakest to strongest):
READ UNCOMMITTED → see others' uncommitted data. Never use.
READ COMMITTED → see only committed data. (Postgres default)
REPEATABLE READ → locked rows can't change mid-transaction.
SERIALIZABLE → transactions execute as if sequential. Slowest.

WHAT TO SAY:
"For the payment service, I'd use SERIALIZABLE isolation to
prevent double-charging — if two requests try to charge the same
order simultaneously, one will fail. For the feed service, READ
COMMITTED is fine — seeing a slightly stale feed for 100ms is
acceptable, and it gives me much higher throughput."

WHY THIS WORKS: You chose DIFFERENT isolation levels for
DIFFERENT services. This shows you understand it's not a
one-size-fits-all decision.
```

---

## 29.2 Caching — The Speed Layer

### When to Cache — The Decision Checklist

```
✅ CACHE WHEN:
• Same data read many times (high read:write ratio)
• Computing the result is expensive (complex JOINs, aggregations)
• Slight staleness is acceptable (seconds to minutes)
• The hot dataset fits in memory

❌ DON'T CACHE WHEN:
• Data changes faster than it's read
• Every request needs unique data (cache miss rate → 100%)
• Strong consistency is required
• The dataset is too large for memory
```

### Redis vs. Memcached — The Interview Answer

```
INTERVIEWER: "Why Redis over Memcached?"

YOU: "Redis, for three reasons. First, data structures — I
need sorted sets for the leaderboard feature. Memcached only
stores strings. Second, persistence — if a Redis node restarts,
it can recover from its AOF log instead of warming the cache
from scratch. Third, pub/sub — I'm using Redis pub/sub for
real-time presence updates. Memcached can't do any of these.

The one advantage Memcached has is multi-threaded performance
— Redis is single-threaded per core. But at our scale, Redis
Cluster across multiple nodes handles the throughput just fine."
```

### Cache Invalidation — The Three Patterns

```
PATTERN 1: TTL-BASED
  SET user:123:profile <data> EX 300 (expires in 5 minutes)
  Say it: "User profile data changes infrequently. A 5-minute
  TTL means the worst case is 5 minutes of stale data, which
  is acceptable. The simplicity is worth the trade-off."

PATTERN 2: EVENT-DRIVEN INVALIDATION
  When data changes → publish event → cache subscriber deletes key
  Say it: "When a user updates their profile, the write service
  publishes 'user.updated' to Kafka. The cache invalidation
  consumer deletes the stale key. The next read triggers a cache
  miss and loads fresh data. Near-real-time freshness."

PATTERN 3: WRITE-THROUGH
  Application writes to BOTH cache and DB simultaneously.
  Say it: "For session data — which is written and immediately
  read on every request — I'd use write-through. The cache is
  always current. The trade-off is higher write latency."
```

### The Thundering Herd — Know This Problem

```
INTERVIEWER: "What happens when a popular cache key expires?"

YOU: "That's the thundering herd problem. If the key for
LeBron James' profile expires and 10,000 concurrent requests
all get a cache miss simultaneously, they all hit the database
at once — potential database crash.

The fix: cache stampede protection. When a key expires, only
ONE request is allowed to refresh the cache. I implement this
with a Redis SETNX lock:

  SETNX cache_lock:lebron_profile <request_id> EX 5

If the lock succeeds, that request fetches from DB and
repopulates the cache. All other requests either wait briefly
or receive a slightly stale cached value while the refresh
happens. This caps the database load at 1 query instead of
10,000."
```

---

## 29.3 Message Queues — Async Processing

### Why Queues — The Before/After Script

```
WITHOUT A QUEUE (what to say to set up the contrast):
"If a user posts a tweet and the API server synchronously
fans out to 10K followers, sends push notifications, updates
the search index, and runs content moderation — the user waits
3-5 seconds. Terrible UX."

WITH A QUEUE:
"Instead, the API server writes to the database, publishes a
'tweet.created' event to Kafka, and returns 201 to the user
in 50ms. Background workers consume from Kafka and handle
fan-out, notifications, search indexing, and moderation
asynchronously. The user sees instant response. If the search
worker goes down, tweets still post — when it recovers, it
processes the backlog."
```

### Kafka vs. RabbitMQ vs. SQS — The Decision Script

```
INTERVIEWER: "Why Kafka over RabbitMQ?"

FOR EVENT STREAMING (millions of events, replay needed):
"Kafka, because I need three things RabbitMQ can't give me.
First, replay — if we deploy a buggy search indexer that
corrupts the index, I can reset the consumer offset and
reprocess the last 7 days of events. In RabbitMQ, messages
are deleted after ACK — they're gone. Second, multiple
consumers — the feed service, notification service, and
analytics service all read from the same 'tweet.created'
topic independently. In RabbitMQ, I'd need to set up
exchanges and bindings. Third, throughput — Kafka handles
millions of messages per second; RabbitMQ tops out at tens
of thousands."

FOR TASK QUEUES (process this job exactly once):
"RabbitMQ, because it has built-in features Kafka doesn't:
priority queues, dead letter exchanges, and per-message TTL.
For a system where I need 'process this payment, and if it
fails 3 times, route it to a dead letter queue for manual
review,' RabbitMQ is purpose-built for this."

FOR ZERO OPS (AWS native, simple queue):
"SQS — if we're on AWS and just need a simple produce-consume
queue without managing infrastructure. It auto-scales
infinitely and costs nothing at rest. The trade-off: no
ordering guarantees (unless I use FIFO, which caps at 300
msg/sec), and no replay."
```

### Delivery Guarantees — Critical Knowledge

```
AT-MOST-ONCE: Message may be lost. Never duplicated.
  → Fire-and-forget. Logging, metrics.

AT-LEAST-ONCE: Message never lost. May be duplicated.
  → Consumer ACKs after processing. If it crashes before
    ACK, message is redelivered.
  → THIS IS THE DEFAULT FOR MOST SYSTEMS.
  → YOUR CONSUMERS MUST BE IDEMPOTENT.

EXACTLY-ONCE: Never lost, never duplicated.
  → Nearly impossible in distributed systems.
  → In practice: at-least-once + idempotent consumers.

WHAT TO SAY:
"I design for at-least-once delivery with idempotent consumers.
Each notification has a unique ID. Before sending, the worker
checks: 'have I already sent notification ABC123?' If yes, skip.
If no, send and record. This gives me effectively exactly-once
semantics without the complexity of distributed transactions."
```

---

## 29.4 Load Balancing

### Layer 4 vs. Layer 7 — The Quick Answer

```
L4 (Transport): Routes by IP + port. Fast. Can't inspect content.
L7 (Application): Inspects HTTP — can route by URL path, headers.

WHAT TO SAY:
"L7 for HTTP traffic — I can route /api/users/* to the user
service and /api/tweets/* to the tweet service. This also gives
me SSL termination at the load balancer so backend servers don't
handle encryption. For non-HTTP protocols like database
connections, I'd use L4."
```

### Algorithms — When They Ask

```
ROUND ROBIN: Simple. Equal-capacity servers. "Good enough for
  most cases when servers are identical."
LEAST CONNECTIONS: Route to least-busy server. "Better when
  requests have variable processing time."
IP HASH: Same client → same server. "Only when I need session
  affinity, which I avoid by making servers stateless."

Health Checks: "Both active (ping /health every 10s) and
  passive (track 5xx rates). If a server fails health checks,
  remove it from rotation automatically."
```

---

## 29.5 CDNs — Content Delivery Networks

```
WHAT A CDN DOES:
  Caches static content at edge locations worldwide.
  User in Tokyo serves from Tokyo node (10ms) instead of
  Virginia origin (150ms).

WHAT TO SAY:
"All static assets — images, CSS, JS, video — are served
through a CDN. This does two things: first, it reduces latency
for global users from 150ms to under 10ms. Second, it absorbs
traffic spikes — during a viral tweet, the CDN handles millions
of image requests without touching our origin servers."

PUSH vs. PULL:
  Pull (default): CDN fetches from origin on first request.
  Push: You proactively upload to CDN. For known-popular content.

CACHE INVALIDATION:
  "I use versioned URLs: image_v2.jpg. The URL is the cache key.
  New version = new URL = automatic cache bust. No need to
  purge anything. Old versions expire via TTL."
```

---

## 29.6 API Design

### REST vs. GraphQL vs. gRPC — The Decision Script

```
REST (external APIs):
"RESTful with JSON. Simple, widely understood, cacheable.
Resource-oriented: GET /users/123/tweets. The trade-off is
over-fetching — the client gets the full user object when it
only needs the name."

GraphQL (mobile apps with diverse needs):
"GraphQL if we're serving multiple frontends — mobile, web,
tablet — each needing different fields. The client specifies
exactly what it needs. The trade-off: complex server-side
implementation, and clients can craft expensive queries if we
don't rate-limit query depth."

gRPC (service-to-service):
"gRPC for internal microservice communication. Binary Protocol
Buffers are 10x faster than JSON REST. Strongly typed — compile-
time safety. Supports bidirectional streaming. The trade-off:
not browser-friendly, requires a proxy for web clients."

RECOMMENDATION:
"External API (client → server): REST.
Internal API (service → service): gRPC.
Real-time: WebSockets or Server-Sent Events."
```

### Pagination — Always Use Cursor-Based

```
WHAT TO SAY:
"I'm using cursor-based pagination, not offset-based. With
offset, two problems: first, OFFSET 1000000 is slow because
the database counts and skips a million rows. Second, if new
items are inserted while the user scrolls, items shift — they
see duplicates or miss entries. With cursor-based — 'give me
20 items after cursor=tweet_id_XYZ' — both problems disappear.
The query uses the primary key index, so it's fast regardless
of position. And insertions don't shift the cursor."
```

---

## 29.7 Microservices vs. Monolith — The Mature Answer

```
INTERVIEWER: "Monolith or microservices?"

THE ANSWER THAT SHOWS SENIORITY:
"I'd start with a modular monolith — a single deployable with
clean internal boundaries between the feed module, user module,
and notification module. Each module has its own folder structure,
its own tests, and communicates through well-defined interfaces.

As the product grows and we identify components that need
independent scaling — like feed computation or search indexing —
we extract those into separate services. This gives us the
simplicity of a monolith on day one with a clear migration
path to microservices when we actually need them.

Starting with microservices on a new product is premature
optimization. You're solving distributed systems problems —
service discovery, network failures, distributed tracing —
before you've even found product-market fit."

WHY THIS ANSWER WORKS: It shows you know microservices are
the right answer AT SCALE but demonstrates the maturity to
know they're WRONG for early-stage systems. This "it depends"
answer with specific reasoning is far more impressive than
dogmatically choosing either side.
```

---

## 29.8 Consistency Patterns

```
STRONG CONSISTENCY:
  Every read returns the most recent write.
  How: Synchronous replication — write waits for majority ACK.
  Cost: Higher write latency.
  Use: "Banking. If I transfer $100, the recipient must see the
  balance update immediately. A stale read could cause a
  double-spend."

EVENTUAL CONSISTENCY:
  Reads may return stale data briefly. Converges within seconds.
  How: Async replication — write returns immediately.
  Cost: Possible stale reads for 1-5 seconds.
  Use: "Social feeds. If a tweet takes 3 seconds to appear in
  all followers' feeds, nobody notices."

READ-YOUR-OWN-WRITES:
  YOU always see your own writes. Others may see stale data.
  How: Route the writing user's reads to the primary briefly.
  Use: "After a user posts a tweet, THEY must see it in their
  feed immediately. Other users can see it 2 seconds later."
```

---

## 29.9 Blob Storage

```
WHAT TO SAY:
"Media never touches the API server. Here's the flow:

1. Client asks API: 'I want to upload a 5MB image.'
2. API generates a pre-signed S3 URL and returns it.
3. Client uploads directly to S3 — bypassing our servers entirely.
4. Client tells API: 'My tweet has this image URL.'
5. API stores the URL in the database.

Why pre-signed URLs? If a million users upload photos
simultaneously, those uploads go to S3, not our API servers.
Our servers stay free to handle API requests. This is how
every large system handles media uploads."
```

---

## 29.10 Search Systems

```
WHAT TO SAY:
"SQL LIKE '%query%' scans every row — O(n). At our scale,
that's billions of rows. Instead, I'd use Elasticsearch with
an inverted index.

An inverted index maps every word to the documents containing
it: 'brown' → [Doc 1, Doc 2], 'fox' → [Doc 1]. Searching
'brown fox' intersects the two lists → Doc 1. O(1) per term.

Architecture: writes go to PostgreSQL (source of truth).
Change Data Capture pipes changes to Elasticsearch. Reads
for search go to Elasticsearch. The index is eventually
consistent — a new tweet might take 1-2 seconds to become
searchable, which is fine."
```

---

## 29.11 Observability — The Maturity Signal

```
WHAT TO SAY WHEN ASKED ABOUT MONITORING:
"I'd track the four golden signals from Google's SRE handbook:
latency, traffic, errors, and saturation.

For the feed service specifically, I'd set alerts on:
• p99 latency exceeding 500ms (degraded experience)
• Error rate exceeding 1% (something is broken)
• Cache hit rate dropping below 70% (cache capacity issue)

For debugging, distributed tracing — every request gets a
trace ID that follows it across all services. When a feed
request is slow, I can see exactly which service in the
chain caused the delay."

WHY THIS IMPRESSES: Most candidates ignore operational
concerns entirely. Mentioning golden signals and distributed
tracing signals that you've worked on production systems.
```

---

## 29.12 Security Building Blocks

```
WHAT TO SAY:
"For authentication, OAuth 2.0 with JWTs. The client
authenticates once, gets a JWT containing user ID and role.
Every subsequent request includes the JWT in the Authorization
header. The API server validates the JWT signature without
hitting the database — it's self-contained. JWTs expire after
15 minutes, with a refresh token for renewal.

For authorization, role-based access control. Users have roles
— admin, editor, viewer — and each role maps to specific
permissions. The JWT includes the role, so permission checks
are local and fast."
```

---

## 29.13 Data Processing Paradigms

```
BATCH: Process large volumes on a schedule.
  "Every night, compute yesterday's engagement metrics."
  Tools: Spark, MapReduce. Latency: hours.

STREAM: Process data as it arrives.
  "When a tweet is posted, immediately update trending topics."
  Tools: Kafka Streams, Flink. Latency: milliseconds.

WHAT TO SAY:
"For trending topics, stream processing — Kafka Streams reads
tweet events in real time and maintains a sliding-window count
of hashtags. For the daily analytics dashboard, batch processing
— a nightly Spark job aggregates the full day's data. I'd use
the Kappa architecture: stream-only processing with Kafka's
durable log for replay when I need to recompute."
```

---

## 29.14 Quick Reference — When to Use What

| Problem | Building Block | Say This |
|---|---|---|
| High read traffic | Cache (Redis) + CDN | "80/20 rule — cache the hot 20% of data" |
| High write traffic | Kafka + write-optimized DB | "Buffer writes, process async" |
| Full-text search | Elasticsearch | "Inverted index, O(1) per term" |
| Global static content | CDN | "Edge caching, sub-10ms globally" |
| File/media storage | S3 + pre-signed URLs | "Bypass API servers entirely" |
| Real-time features | WebSockets | "Persistent bidirectional connection" |
| Transaction integrity | PostgreSQL + ACID | "SERIALIZABLE isolation for payments" |
| Time-series data | Cassandra | "Partition by device, cluster by time" |
| Analytics | BigQuery/Redshift | "Columnar storage, fast aggregations" |
| Rate limiting | Redis INCR + EXPIRE | "Atomic counter, sub-1ms" |

---

## 29.15 Chapter Summary

This chapter armed you with the technical depth to answer **any** follow-up question in a system design interview. For each building block, you now have:

- **When to use it** (and when NOT to)
- **The trade-offs** (every choice has a cost)
- **The exact words** to say in the interview
- **Why those words work** (what signal they send to the interviewer)

The goal isn't memorization. It's having enough depth that when you need a building block in your design, you can speak about it with the confidence of someone who's used it in production — not just read about it.

---

*Next Chapter: Chapter 30 — System Design Practice Problems: Six Complete Worked Designs →*
