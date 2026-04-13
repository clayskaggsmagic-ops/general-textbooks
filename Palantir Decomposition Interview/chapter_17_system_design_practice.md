# Chapter 17: System Design Practice Problems — Six Complete Interview Simulations

> *"System design interviews test whether you can reason through a complex system live, not whether you've memorized the 'right' architecture. These worked problems teach you the reasoning, not the answer."*

---

## How to Use This Chapter

**Active practice (strongly recommended):**

1. Read only the prompt for each problem
2. Set a timer for 35 minutes
3. Design the system on paper — draw diagrams, write schemas, estimate scale
4. Then read the model answer
5. Compare: What trade-offs did you miss? What components did you forget?

**Each problem is a full interview simulation** — complete with the dialogue you'd say, interviewer questions, and annotations on why each phrase works.

---

## Problem 1: Design a URL Shortener (like bit.ly)

### The Prompt

*"Design a URL shortening service. Users submit a long URL and receive a short one. When someone visits the short URL, they're redirected to the original."*

### Phase 1: Requirements (Minutes 0-5)

```
YOU: "Let me clarify the scope. For core features: users
submit a long URL and get a short one. Clicking the short
URL redirects to the original. Should I also support custom
short URLs, analytics, or expiration?"

INTERVIEWER: "Keep it simple — just create and redirect."

YOU: "For scale — is this a startup or are we designing for
bit.ly-level traffic? Hundreds of millions of redirects
per month?"

INTERVIEWER: "Assume large scale — 100M new URLs per month,
10 billion redirects per month."

YOU: "Quick estimation. 100M writes per month / 2.6M seconds
per month ≈ 40 writes/sec. Very light — writes are NOT
the bottleneck. 10B reads per month ≈ 3,800 reads/sec.
That's a 100:1 read/write ratio. Read-heavy.

Storage over 5 years: 100M × 60 months = 6 billion URLs.
Each record ≈ 500 bytes. That's 3 TB. Manageable.

For the short key length: base62 encoding with 6 characters
gives me 62^6 = 56.8 billion unique keys. More than enough
for 6 billion URLs.

So: read-heavy system, modest QPS, moderate storage. My
design should optimize for fast redirects."
```

### Phase 2: High-Level Design (Minutes 5-10)

```
YOU: "Two flows — create and redirect.

[Drawing the diagram]

CREATE: Client sends long URL → API Server generates short
key → stores {short_key → long_url} in database → returns
short URL to client.

REDIRECT: Client visits short URL → API Server checks Redis
cache → if hit, return 302 redirect. If miss, query database
→ populate cache → return 302.

The cache is critical for the read path. With a Pareto
distribution — a small fraction of URLs get most of the
traffic — I'd expect 80%+ cache hit rate."
```

### Phase 3: Deep Dive — Key Generation (Minutes 10-20)

```
INTERVIEWER: "How do you generate the short key?"

YOU: "Three approaches. Let me walk through each.

APPROACH 1 — Hash the long URL. MD5(url) and take the first
6 base62 characters. Pros: deterministic — same URL always
gets the same key. Cons: collisions. MD5 is 128 bits but
I'm using only ~36 bits. I'd need collision detection —
check if the key exists, and if so, append a counter and
re-hash. Complexity adds up.

APPROACH 2 — Counter-based. A global counter increments:
1, 2, 3... Base62 encode it: 1→'1', 62→'10'. Guaranteed
unique — no collision checking needed. Cons: sequential,
so predictable. Users could guess the next short URL.
Security concern.

APPROACH 3 — Pre-generated key pool. Generate millions of
random 6-char keys in advance and validate they're unique.
When a user creates a URL, pop a key from the pool. No
collision checking at runtime. But maintaining the pool
across distributed servers adds complexity.

MY RECOMMENDATION: counter-based with a Snowflake-style
distributed counter. Each server gets a range of IDs from
a coordination service (like ZooKeeper) — Server A gets
1-10,000, Server B gets 10,001-20,000. No collisions, no
coordination on each request. To address predictability,
I'd apply a symmetric encryption function on the counter
before base62 encoding — reversible, so lookup is still
O(1), but keys look random to users."

WHY THIS ANSWER WORKS: You showed three options with
trade-offs before choosing. Every interviewer will be
impressed by the hybrid (counter + encryption) because
it shows creative problem-solving, not just textbook recall.
```

### Phase 3 (continued): Data Model (Minutes 20-25)

```
YOU: "For the schema:"

CREATE TABLE urls (
    short_key    CHAR(7) PRIMARY KEY,
    long_url     VARCHAR(2048) NOT NULL,
    user_id      BIGINT,
    created_at   TIMESTAMP DEFAULT NOW(),
    expires_at   TIMESTAMP,
    click_count  BIGINT DEFAULT 0
);

CREATE UNIQUE INDEX idx_long_url ON urls(long_url);

"The unique index on long_url lets me check whether a URL has
already been shortened — avoiding duplicate entries. The
primary key on short_key makes redirect lookups O(log n) via
the B-tree index."

CACHE STRATEGY:
"Redis cache-aside pattern. On redirect: check Redis first.
Miss → query Postgres → write result to Redis with 24-hour
TTL. For popular links detected via click tracking, I'd
extend the TTL or set no expiration."
```

### Phase 4: Scaling and Trade-offs (Minutes 25-35)

```
YOU: "At 3,800 reads/sec and 80% cache hit rate, only ~760
QPS reach the database. A single Postgres instance handles
this easily. At 10x growth (38K reads/sec), I'd add 3 read
replicas — each handles ~2,500 QPS. At 100x, I'd shard by
short_key using consistent hashing.

One important trade-off: 301 vs 302 redirect. A 301 (permanent)
tells the browser to cache the redirect — subsequent visits
never hit our servers. Great for bandwidth. But I can't count
clicks because the request never reaches us. A 302 (temporary)
always hits our servers, so I can track analytics. I'd default
to 302 if we ever want analytics, 301 for pure URL shortening."
```

---

## Problem 2: Design a Rate Limiter

### The Prompt

*"Design a rate limiting service — each user can make at most 100 API requests per minute."*

### Phase 1: Requirements (Minutes 0-3)

```
YOU: "Quick scoping. The rate limiter sits in front of the API
layer — every request passes through it. Requirements: sub-1ms
latency (it's on the critical path), distributed (works across
all API servers), and configurable rules (per-user, per-IP,
per-endpoint).

Key design question: if the rate limiter itself is down, do
we fail open (allow all requests — dangerous but available)
or fail closed (block all requests — safe but kills the
entire API)?"

INTERVIEWER: "Fail open with degraded protection."
```

### Phase 2: Algorithm Deep Dive (Minutes 3-15)

```
YOU: "There are five rate limiting algorithms. Let me compare
the three most practical ones.

FIXED WINDOW COUNTER: Divide time into 1-minute windows.
Count requests per window. Over limit → reject. Simple,
but has a burst problem: 100 requests at 00:59 and 100 at
01:00 = 200 requests in 2 seconds, both within their windows.

SLIDING WINDOW COUNTER: Combines two fixed windows with
weighted overlap. If we're 30 seconds into the current window:
weighted count = (previous window count × 0.5) + current
window count. Approximates a true sliding window with just
two counters. Low memory, smooth rate enforcement.

TOKEN BUCKET: Bucket holds 100 tokens. Refills at 100/min.
Each request takes 1 token. Empty bucket → reject. Advantage:
allows controlled bursting — a user who's been idle can burst
up to 100 requests instantly, then must wait for refill.

MY CHOICE: Sliding window counter for general rate limiting.
It's precise enough, uses minimal memory (two integers per
user per rule), and avoids the fixed window burst problem.
For APIs where controlled bursting is desirable — like
data ingestion endpoints — I'd use token bucket."
```

### Phase 3: Architecture (Minutes 15-25)

```
YOU: "Centralized counter in Redis. Every API server checks
Redis before processing a request.

Flow:
1. Request arrives at API server
2. Server runs atomic Redis operation:
   MULTI
     INCR ratelimit:user123:window_202603272205
     EXPIRE ratelimit:user123:window_202603272205 120
   EXEC
3. If INCR result > 100 → return 429 Too Many Requests
   with headers X-RateLimit-Remaining: 0,
   X-RateLimit-Reset: <seconds until window resets>
4. If under limit → proceed to API handler

Why Redis? Atomic INCR means no race conditions. Two requests
arriving simultaneously can't both read 99, both increment
to 100, and both pass. INCR atomically increments and returns
the new value. Sub-millisecond latency. A single Redis node
handles 100K+ operations per second — enough for our entire
API fleet.

For the fail-open requirement: if Redis is unreachable, each
API server falls back to a local in-memory token bucket.
It's approximate — the per-server limit is total_limit / 
num_servers — but it's better than no protection at all."
```

---

## Problem 3: Design a Chat System (WhatsApp/Slack)

### The Prompt

*"Design a real-time chat application supporting 1-on-1 and group messages."*

### Phase 1: Requirements and Estimation (Minutes 0-5)

```
YOU: "Core features: 1-on-1 messaging, group chats up to 500
members, online/offline status, read receipts. Explicit
exclusions: video calls, file sharing beyond simple images.

Scale: 50M DAU, 1 billion messages per day. That's
1B / 86,400 ≈ 12,000 messages per second writes.

Concurrent connections: 50M × 30% online = 15 million
simultaneous WebSocket connections. Each connection uses ~10KB
of memory, so 15M × 10KB = 150GB of connection state across
my WebSocket servers. A single server handles ~50K connections,
so I need ~300 WebSocket gateway servers.

This is a WRITE-heavy real-time system. The design must
prioritize: (1) reliable message delivery, (2) sub-100ms
latency for online users, (3) persistent message storage."
```

### Phase 2: High-Level Design (Minutes 5-10)

```
YOU: "The core innovation here is the WebSocket layer. Unlike
HTTP — which is request-response — WebSockets maintain a
persistent bidirectional connection. This lets the server PUSH
messages to clients in real-time without polling.

Architecture:
• WebSocket Gateway: 300 servers managing 50K connections each.
  Stateful — each gateway knows which users are connected to it.
• Chat Service: stateless logic layer that routes messages.
• Message Store: Cassandra — write-optimized with partition-key
  access patterns perfect for chat.
• Presence Service: Redis with TTL-based heartbeats.
• Push Notification Service: for offline users (FCM/APNs).

When User A sends a message to User B:
1. A's client sends message via WebSocket to Gateway
2. Gateway routes to Chat Service
3. Chat Service stores message in Cassandra
4. Chat Service checks: is B online?
   YES → route message through B's WebSocket Gateway → B
         receives in real-time (~50ms end-to-end)
   NO  → send push notification via FCM/APNs
5. Return ACK to A: 'message sent'

When B opens the message: B's client sends 'read' receipt →
Chat Service updates status → A receives 'read' indicator."

INTERVIEWER: "Why Cassandra for message storage?"

YOU: "Three reasons. First, write-heavy — 12K messages/sec.
Cassandra is designed for high write throughput. Second, the
access pattern is simple — 'give me all messages in conversation
X, sorted by time.' That's exactly what Cassandra's partition
key + clustering key model does: partition by conversation_id,
cluster by timestamp. Third, horizontal scaling — Cassandra
adds nodes linearly. No manual sharding.

The trade-off: no JOIN queries. But chat has no JOINs — every
query is scoped to a single conversation."
```

### Phase 3: Deep Dive — Online Presence (Minutes 20-28)

```
INTERVIEWER: "How does the online/offline status work?"

YOU: "Heartbeat-based with Redis TTL.

When a user connects, the gateway sets a Redis key:
  SET presence:user_123 'online' EX 30

The client sends a heartbeat every 10 seconds. Each heartbeat
resets the TTL: SET presence:user_123 'online' EX 30

If the client disconnects — app crash, lost connection — the
heartbeat stops. After 30 seconds, the key expires in Redis.
User is now offline.

The tricky part is fanout. When User A goes offline, all of
A's contacts need to know. If A has 500 contacts, that's 500
status notifications. If 1,000 users change status per second,
that's 500,000 events per second.

My solution: lazy presence updates. I DON'T push presence
changes to every contact. Instead, when User B opens a chat
with User A, B subscribes to A's presence key. When B closes
the chat, B unsubscribes. This limits presence fanout to
actively viewed conversations — perhaps 5-10 subscriptions
per user instead of 500. Massive reduction in event volume."

WHY THIS ANSWER WORKS: The heartbeat pattern is standard.
The fanout optimization is the advanced insight — it shows
you've thought about the scaling implications of a seemingly
simple feature.
```

### Phase 4: Group Chat Fanout (Minutes 28-35)

```
INTERVIEWER: "How do you handle group messages?"

YOU: "For small groups (under 100 members): fan out on write.
When a message is sent to a 20-person group, the Chat Service
sends 19 copies — one to each other member's gateway. At 12K
messages/sec, most groups are 5-20 people, so the fanout
multiplier is manageable.

For large groups (100-500 members): switch to pull model. Don't
push the message to 500 gateways. Instead, store the message
in Cassandra. When a user opens the group chat, they pull
recent messages. For active large groups, I'd pre-aggregate
the last 50 messages in a Redis list for fast access.

This hybrid mirrors what we did with the Twitter celebrity
problem — push for small audiences, pull for large ones."
```

---

## Problem 4: Design a Web Crawler

### The Prompt

*"Design a web crawler that downloads the entire web for a search engine."*

### Phase 1: Scale and Estimation (Minutes 0-4)

```
YOU: "Target: 1 billion pages per day. That's 11,600 pages/sec.
Average page is 500KB, so 5.8 GB/sec bandwidth — about 46 Gbps.
Storage: 1B × 500KB = 500 TB/day raw, ~100 TB compressed.
This is a massively parallel batch processing system."
```

### Phase 2: Architecture (Minutes 4-10)

```
YOU: "Four major components:

URL FRONTIER: a priority queue that decides what to crawl next.
FETCHER: downloads pages. Hundreds of parallel workers.
PARSER: extracts links and content from downloaded pages.
URL FILTER: deduplication via Bloom filter.

The cycle: Frontier serves a URL → Fetcher downloads → Parser
extracts links → Filter deduplicates → new URLs enter Frontier.

The most interesting component is the URL Frontier."
```

### Phase 3: Deep Dive — URL Frontier (Minutes 10-20)

```
YOU: "The Frontier has two sub-queues:

PRIORITY QUEUE — what to crawl. Ranks URLs by importance:
domain authority (google.com > random-blog.com), page depth
(homepage > /page/sub/sub), freshness policy (news sites
re-crawled hourly, static sites monthly).

POLITENESS QUEUE — when to crawl. Never send more than 1
request per second to the same domain. Each domain has its own
queue with a cooldown timer. A scheduler picks the highest-
priority URL whose domain is past its cooldown.

This prevents overwhelming small websites — which would get us
blocked — while ensuring important pages are crawled first."

INTERVIEWER: "How do you handle URL deduplication at 1 billion
URLs per day?"

YOU: "Bloom filter. A probabilistic data structure that tells
me 'definitely not seen' or 'probably seen.' False positive
rate of 1% — meaning 1% of the time it says 'seen' when it
hasn't been. But zero false negatives — it never misses a
new URL.

Memory: about 1 byte per URL. 10 billion URLs = 10GB. Fits
in RAM on a single machine.

For the 1% false positives: those URLs simply don't get
crawled, which is acceptable — a 1% miss rate on the entire
web is negligible. If precision matters for specific URLs,
I'd have a secondary check against a full URL database
on disk."
```

---

## Problem 5: Design a Notification System

### The Prompt

*"Design a notification system supporting push, SMS, and email for 100M users."*

### Phase 1: Requirements (Minutes 0-4)

```
YOU: "Core requirements: multi-channel (push, SMS, email),
user preferences (opt-in/out per channel and notification type),
priority levels (critical = instant, standard = within 5 min,
marketing = scheduled), rate limiting (don't spam users),
delivery tracking.

Scale: 1 billion notifications per day ≈ 12,000/sec.
Critical constraint: no duplicate deliveries. If a notification
worker crashes after sending but before ACKing the queue,
the message must not be re-sent."
```

### Phase 2: The Pipeline (Minutes 4-20)

```
YOU: "The notification pipeline has six steps:

STEP 1: INGESTION — any service sends a notification request
to the Notification Service via API or event.

STEP 2: PREFERENCE CHECK — look up user's preferences. User
has push enabled, SMS disabled, email enabled? Send via push
and email only.

STEP 3: TEMPLATE RENDERING — 'order_shipped' template +
data → 'Your order #456 shipped! ETA: March 30.'

STEP 4: RATE LIMITING — has this user received more than 5
notifications in the last hour? If yes, queue for later
(except critical). This prevents user fatigue.

STEP 5: CHANNEL ROUTING — push notification goes to the
Push Queue, email goes to the Email Queue. Separate queues
per channel with dedicated worker pools.

STEP 6: DELIVERY — push workers call FCM/APNs, email workers
call SES/SendGrid. Record delivery status."

INTERVIEWER: "How do you prevent duplicate notifications?"

YOU: "Idempotent delivery. Each notification has a unique
notification_id generated at Step 1. Before sending, the
worker checks a Redis dedup cache:

  EXISTS dedup:notif_ABC123

If the key exists → skip (already sent). If not → send,
then SET dedup:notif_ABC123 1 EX 86400 (24-hour TTL).

This gives me effective exactly-once delivery using at-least-
once queue semantics. The dedup window of 24 hours handles
any reasonable retry scenario."
```

---

## Problem 6: Design a Distributed Key-Value Store

### The Prompt

*"Design a distributed key-value store like DynamoDB — GET, PUT, DELETE across multiple nodes."*

### Phase 1: Requirements (Minutes 0-3)

```
YOU: "Billions of keys, petabytes of data. High availability —
reads and writes succeed even during node failures. Tunable
consistency — the client chooses strong or eventual per request.
Sub-10ms reads, sub-50ms writes."
```

### Phase 2: Consistent Hashing (Minutes 3-12)

```
YOU: "Data distribution via consistent hashing. Nodes sit on a
hash ring. Each key hashes to a position on the ring and is
assigned to the next node clockwise.

Why not hash(key) % num_nodes? Because adding a node remaps
~75% of all keys. With consistent hashing, adding a node
moves only ~1/N of keys.

Virtual nodes: each physical node gets 150 virtual positions
on the ring. This ensures uniform distribution — without
virtual nodes, 3 physical nodes might split the ring 60/30/10.

Replication: each key is stored on N nodes (N=3 typically).
The primary node and the next N-1 nodes clockwise on the ring."
```

### Phase 3: Quorum Reads/Writes (Minutes 12-22)

```
INTERVIEWER: "How do you support tunable consistency?"

YOU: "Quorum consensus. N = replica count, W = write quorum,
R = read quorum.

STRONG CONSISTENCY: W + R > N.
Example: N=3, W=2, R=2. A write succeeds when 2 nodes ACK.
A read queries 2 nodes. Since 2+2=4 > 3, at least one node
in the read participated in the latest write. Guaranteed to
see the most recent data.

EVENTUAL CONSISTENCY: W=1, R=1.
Fastest possible. Write to 1 node, read from 1 node. But you
might get stale data. Replicas converge via background gossip.

The client chooses per request:
  PUT(key, value, consistency=STRONG)
  GET(key, consistency=EVENTUAL)

This lets the same store serve both banking queries (strong)
and analytics queries (eventual)."
```

### Phase 4: Failure Handling (Minutes 22-35)

```
INTERVIEWER: "What happens when a node goes down?"

YOU: "Three mechanisms:

FAILURE DETECTION — Gossip protocol. Each node pings random
peers every second. If a node doesn't respond, it's suspected.
If multiple nodes suspect it, it's declared dead.

SLOPPY QUORUM + HINTED HANDOFF — if a replica node is down,
the write goes to the next available node with a 'hint'
attached. When the failed node recovers, the hint is handed
off — data transferred to the correct node. This ensures
writes succeed during failures at the cost of temporary
inconsistency.

ANTI-ENTROPY WITH MERKLE TREES — to detect and repair
inconsistencies between replicas. Each replica builds a hash
tree of its data. Two replicas compare tree roots — if they
match, they're in sync. If not, they traverse the tree to
find exactly which keys differ. Only those keys are synced.
O(log n) comparison instead of O(n).

These three mechanisms together give me a store that handles
node failures gracefully: writes don't fail (hinted handoff),
inconsistencies self-heal (Merkle trees), and dead nodes are
detected within seconds (gossip)."
```

---

## 17.1 Pattern Library — Patterns That Repeat Across Problems

| Pattern | Where It Appears | The Principle |
|---|---|---|
| **Cache for reads** | URL Shortener, Chat | 80% of traffic hits 20% of data |
| **Queue for writes** | Chat, Notifications | Decouple request from processing |
| **Fan-out on write vs. read** | Chat groups, Notifications | Push for small audiences, pull for large |
| **Consistent hashing** | KV Store, URL Shortener | Add nodes without remapping everything |
| **Bloom filter** | Web Crawler | Probabilistic membership at massive scale |
| **Idempotent consumers** | Notifications, Chat | At-least-once delivery requires dedup |
| **Priority queues** | Crawler, Notifications | Not all work is equal |
| **WebSockets** | Chat | Real-time bidirectional |
| **Quorum consensus** | KV Store | Tunable consistency per-request |

---

## 17.2 Additional Practice Prompts

| # | Prompt | Key Challenge |
|---|---|---|
| 1 | Design a news feed (Facebook) | Fan-out, ranking, caching personalized feeds |
| 2 | Design Uber | Real-time matching, geospatial queries, surge pricing |
| 3 | Design YouTube | Transcoding, CDN, adaptive bitrate |
| 4 | Design Google Autocomplete | Trie data structure, ranking by popularity |
| 5 | Design Stripe | ACID, idempotency, double-entry accounting |
| 6 | Design Dropbox | Chunked uploads, dedup, sync conflicts |
| 7 | Design Datadog | Time-series storage, aggregation, alerting |
| 8 | Design a ticket booking system | Seat locking, race conditions |

---

## 17.3 The Active Practice Method

```
FOR EACH PRACTICE PROBLEM:
1. Read ONLY the prompt — not the model answer
2. Set a 35-minute timer
3. Grab paper or a whiteboard
4. Minutes 0-5: Write requirements + estimation
5. Minutes 5-10: Draw high-level architecture
6. Minutes 10-30: Deep dive into 2-3 components
7. Minutes 30-35: Scaling and trade-offs
8. Read the model answer
9. Compare: What did you miss? What was your strongest section?
10. Write down the top 3 concepts you need to review

DO THIS FOR ONE PROBLEM PER DAY FOR TWO WEEKS.
The structure becomes instinctive. The depth comes from repetition.
```

---

*With these three chapters — fundamentals (Ch. 28), building blocks (Ch. 29), and practice problems (Ch. 30) — you have a complete system design interview curriculum. Every section tells you not just what to know, but exactly what to say, why to say it, and what signal it sends to the interviewer.*
