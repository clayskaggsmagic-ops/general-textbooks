# Chapter 28: System Design Fundamentals — The Architecture Interview

> *"System design isn't about memorizing architectures. It's about navigating trade-offs under ambiguity — the same skill you need on Day 1 of any engineering job."*

---

## Why System Design Interviews Exist

Every major tech company — Google, Meta, Amazon, Microsoft, Palantir, Stripe — includes a system design round. It tests what coding challenges can't: **can you reason about complex, interconnected systems at scale?**

A coding interview tests whether you can implement a function. A system design interview tests whether you can design the system that function lives in:

- **Ambiguity tolerance** — The problem is deliberately vague. Can you scope it?
- **Breadth of knowledge** — Databases, caching, networking, queuing, APIs, storage — can you pull from all of them?
- **Trade-off reasoning** — Every design choice has a cost. Can you articulate why you chose A over B?
- **Scale awareness** — Your design works for 100 users. Does it work for 100 million?
- **Communication** — Can you explain a complex system to another engineer clearly?

---

## 28.1 The 45-Minute Structure — Second-by-Second Execution Guide

### The Five Phases

```
┌──────────────────────────────────────────────────────────┐
│               45-MINUTE SYSTEM DESIGN                     │
│                                                           │
│  MINUTES 0-5:   REQUIREMENTS GATHERING                    │
│  MINUTES 5-7:   BACK-OF-ENVELOPE ESTIMATION               │
│  MINUTES 7-12:  HIGH-LEVEL DESIGN                         │
│  MINUTES 12-35: DEEP DIVE (2-3 components)                │
│  MINUTES 35-45: SCALING AND TRADE-OFFS                    │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### What the Interviewer Is Grading

| Axis | What They Look For | How You Demonstrate It |
|---|---|---|
| **Problem Exploration** | Did you ask the right questions? Did you scope before diving in? | Ask 5-8 targeted questions. Write requirements on the board. |
| **High-Level Design** | Is the architecture sound? Do the components make sense? | Draw a clean diagram with labeled components and data flow arrows. |
| **Detailed Design** | Can you go deep? Schema, API contracts, data flow, edge cases | Write actual SQL schemas, API endpoints with request/response bodies. |
| **Trade-offs** | Can you articulate why you chose one approach over another? | Every decision gets a "I chose X over Y because Z" narration. |
| **Communication** | Is the explanation clear? Diagrams? Check-ins with interviewer? | Signpost every transition. Ask "does this direction make sense?" every 5 min. |

### The #1 Mistake and How to Avoid It

**The mistake:** Jumping to the solution. The interviewer says "Design Twitter" and you immediately start drawing databases.

**Why it kills you:** You haven't asked what features of Twitter. The full platform? Just the tweet feed? What scale? What's more important — read speed or write consistency? Without scoping, you'll build the wrong thing and run out of time redesigning.

**The fix — exact words to say:**

```
INTERVIEWER: "Design Twitter."

YOU (immediately, before touching the whiteboard):
"Great question. Before I start designing, I want to make
sure I'm solving the right problem. Can I take a couple of
minutes to clarify requirements?"

WHY THIS WORKS: This shows discipline. The interviewer has
seen 50 candidates today jump straight to "I'd use Kafka."
You immediately differentiate yourself by showing you think
before you build. It also buys you the most valuable 5 minutes
of the interview — the scoping phase where you control the
problem's complexity.
```

---

## 28.2 Phase 1: Requirements — The Scoping Dialogue (Minutes 0-5)

This is the most important phase. The questions you ask here determine whether you spend 35 minutes building the right system or the wrong one.

### The Exact Dialogue — "Design Twitter"

Here is a complete, word-for-word simulation of what the first 5 minutes should sound like:

```
YOU: "I want to split requirements into functional — what the
system does — and non-functional — how it behaves. Let me
start with functional."

YOU: "For the core features, I'm thinking: users can post
tweets with text. Users can follow other users. Users see a
home timeline of tweets from people they follow. Are there
other features you want me to include — likes, retweets,
search, DMs — or should I focus on the core posting and
feed experience?"

INTERVIEWER: "Let's focus on posting, following, and the
home timeline."

WHY YOU ASKED THIS: You just cut the problem from 10 features
to 3. A candidate who tries to design all of Twitter in 45
minutes will design none of it well. By asking, you showed
the interviewer you understand product scope AND you got
explicit permission to simplify. If the interviewer adds
features, great — you handle them. If not, you've bought
yourself focus.

YOU: "Got it. Now for non-functional requirements. What scale
are we designing for? Are we talking about a startup with
10K users, or Twitter-scale with hundreds of millions?"

INTERVIEWER: "Assume Twitter scale — 500 million daily
active users."

YOU: "And for the timeline — is it okay if a new tweet takes
a few seconds to appear in all followers' feeds? Or does it
need to be instant?"

INTERVIEWER: "A few seconds is fine."

WHY YOU ASKED THIS: You just established that eventual
consistency is acceptable. This single answer unlocks an
entirely different (and simpler) architecture than if the
interviewer had said "instant." You can now use async
fan-out, message queues, and caching — instead of
synchronous writes to every follower's feed. One question
just saved you 15 minutes of unnecessary complexity.

YOU: "Last question on requirements. Is the system read-heavy
or write-heavy? My intuition is that most users read far more
tweets than they write — but I want to confirm."

INTERVIEWER: "Yes, heavily read-dominant."

YOU: "Perfect. Let me summarize what I'm designing before
I start."
```

### The Requirements Summary — Say This Every Time

After your scoping questions, **always** restate the requirements. Write them on the board if possible:

```
YOU: "So here's what I'm designing:

FUNCTIONAL:
• Users post tweets (text, 280 chars max)
• Users follow other users
• Users see a home timeline — tweets from followed
  users, sorted by recency

NON-FUNCTIONAL:
• 500M daily active users
• Read-heavy — optimize for fast feed loading
• Eventual consistency is acceptable (few-second delay)
• High availability — the feed should always load
• Tweets must never be lost once posted (durability)

Does this match your expectations, or should I adjust?"

WHY THIS WORKS: Three reasons.
1. You just proved you LISTENED. Most candidates hear "Design
   Twitter" and design what they think Twitter is. You
   designed what the INTERVIEWER asked for.
2. You created a CONTRACT. If the interviewer pushes back on
   your design later, you can point to the requirements:
   "We agreed eventual consistency was fine, which is why
   I chose this async approach."
3. You showed COMMUNICATION SKILL. In a real job, you'd
   never start building without a requirements doc. This
   mirrors professional behavior.
```

### The Requirements Template — Adapt for Any Problem

Memorize this question sequence. It works for every system design problem:

```
FUNCTIONAL SCOPING (3 questions):
1. "What are the core features we need to support?"
   → Narrows the problem from vague to specific
2. "Are there features we should explicitly exclude?"
   → Prevents scope creep mid-interview
3. "What's the primary user action — mostly reading or writing?"
   → Determines your caching and database strategy

NON-FUNCTIONAL SCOPING (4 questions):
4. "What scale are we designing for?"
   → Users, QPS, data volume
5. "What's the latency requirement?"
   → Real-time? Near-real-time? Batch is fine?
6. "Consistency vs availability — which matters more?"
   → Determines your entire data architecture
7. "Any regulatory or compliance requirements?"
   → Data residency, encryption, audit logging

THEN:
8. "Let me do a quick back-of-envelope calculation..."
   → Transitions naturally to Phase 2
```

---

## 28.3 Phase 2: Back-of-Envelope Estimation (Minutes 5-7)

### Why This Matters

Estimation proves you think quantitatively. Good estimation changes your design: if you calculate 600K reads/sec, you know a single database won't work. If you calculate 40 writes/sec, you know writes aren't your bottleneck.

### The Numbers You Should Memorize

```
STORAGE:
─────────
1 char     = 1 byte (ASCII) or 2-4 bytes (UTF-8)
1 tweet    ≈ 300 bytes (280 chars + metadata)
1 photo    ≈ 200 KB (compressed JPEG)
1 min video ≈ 10 MB (compressed 720p)
1 KB = 10³ bytes | 1 MB = 10⁶ | 1 GB = 10⁹ | 1 TB = 10¹²

THROUGHPUT:
──────────
Single server QPS (web): 1,000 - 10,000
Single DB QPS (reads): 10,000 - 100,000
Single DB QPS (writes): 1,000 - 10,000
Redis (in-memory cache): 100,000+ QPS

TIME:
─────
1 day  = 86,400 sec ≈ 10⁵ sec
1 year = 31.5M sec  ≈ 3 × 10⁷ sec

LATENCY:
────────
Cache read: < 1ms
SSD read: ~100 μs
HDD seek: ~10ms
Cross-datacenter: 1-10ms
Cross-continent: 50-150ms
```

### The Exact Estimation Dialogue — "Design Twitter"

```
YOU: "Let me do a quick estimation to ground the design.

500M DAUs. Each user reads about 100 tweets per day —
that's 500M × 100 / 86,400 ≈ 580,000 read requests
per second. That's roughly 600K reads/sec.

For writes — most users consume content, not create it.
Let's say each user posts 0.5 tweets per day on average.
500M × 0.5 / 86,400 ≈ 3,000 writes per second.

So the read/write ratio is about 200:1. This is extremely
read-dominant, which tells me two things for the design:

First, I should invest heavily in caching the home feed.
If I can serve even 80% of reads from cache, I drop my
database load from 600K to 120K QPS.

Second, write performance is not the bottleneck. I can
afford to do more work per write — like fan-out to
followers' feed caches — because writes are rare.

For storage: 500M users × 0.5 tweets/day × 365 days ×
300 bytes per tweet = about 27 TB per year of text.
That's very manageable. Media would be significantly
more, stored in object storage."

WHY THIS WORKS: You didn't just do math. You translated
every number into a DESIGN DECISION. "600K reads/sec"
became "invest in caching." "3K writes/sec" became
"fan-out on write is feasible." The interviewer sees
that your estimation drives your architecture, not the
other way around.

COMMON MISTAKE: Spending 5 minutes on precise math.
Don't calculate 578,703. Say "roughly 600K." Order of
magnitude is all that matters.
```

---

## 28.4 Phase 3: High-Level Design (Minutes 7-12)

### The Universal Web Architecture

Almost every system follows this skeleton. Start here and customize:

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐
│  Client  │────→│ Load Balancer│────→│  Web Servers  │
│  (App/   │     │  (L7/L4)    │     │  (Stateless)  │
│   Web)   │     └──────────────┘     └──────┬───────┘
└──────────┘                                 │
                                    ┌────────┼────────┐
                                    │        │        │
                              ┌─────▼──┐ ┌───▼────┐ ┌─▼───────┐
                              │  API   │ │ Cache  │ │ Message │
                              │ Server │ │(Redis) │ │  Queue  │
                              └───┬────┘ └───┬────┘ └────┬────┘
                                  │          │           │
                              ┌───▼──────────▼───┐  ┌───▼────┐
                              │    Database      │  │Workers │
                              │  (Primary +      │  │(Async) │
                              │   Replicas)      │  └────────┘
                              └──────────────────┘
```

### The Exact Walk-Through Script

This is what you say, word for word, as you draw:

```
YOU: "Here's the high-level architecture. I'll walk through
each component, then we can decide which area to dive deep on.

[DRAW as you talk — don't draw everything then explain]

Clients — mobile apps and web browsers — hit the system
through a CDN for static assets and a Load Balancer for
API requests. The load balancer distributes traffic across
multiple stateless API servers, which means any request
can go to any server. This lets us scale horizontally
just by adding more servers.

For the read path — when a user opens their feed — the
API server checks the Cache first. Redis. If it's a cache
hit, we return immediately — sub-millisecond. If it's a
miss, we query the Database, populate the cache, and return.
With our 200:1 read/write ratio, I want at least an 80%
cache hit rate.

For the write path — when a user posts a tweet — the API
server writes to the Database, then publishes a 'tweet_created'
event to a Message Queue. Downstream workers consume from the
queue to do async work: fan out the tweet to followers' feed
caches, send notifications to mentioned users, and update
the search index.

Media — images and videos — is stored in Object Storage
like S3 and served through the CDN. It never touches our
API servers.

Before I go deeper — which component would you like me
to focus on? The feed generation, the data model, or
the write path and fan-out?"

WHY THIS WORKS:
1. You drew AND talked simultaneously — the interviewer
   can follow your thinking in real time.
2. You referenced your estimation: "200:1 read/write ratio"
   and "80% cache hit rate." This proves your design is
   driven by data, not dogma.
3. You ended with a CHOICE for the interviewer. This is
   collaborative, not a monologue. It also ensures you
   spend the next 20 minutes on what THEY care about.
```

### Component Vocabulary — Name-Drop Naturally

| Component | When to Use | Say This |
|---|---|---|
| **Load Balancer** | Always | "L7 load balancer distributing across stateless app servers" |
| **Cache** | Read-heavy systems | "Redis cache-aside pattern with TTL-based invalidation" |
| **CDN** | Static/global | "CDN edge caching for static assets and media" |
| **Message Queue** | Async processing | "Kafka topic for event-driven fan-out to downstream workers" |
| **Object Storage** | Media/blobs | "S3 with pre-signed upload URLs to bypass the API layer" |
| **Search Index** | Full-text search | "Elasticsearch with CDC from the primary database" |

---

## 28.5 Phase 4: The Deep Dive — Where Interviews Are Won (Minutes 12-35)

The high-level design gets you to the midpoint. The deep dive is where you differentiate. The interviewer will push you into 2-3 components. Design them at a level where an engineer could implement your spec.

### Deep Dive Example: The Home Feed (The Hardest Problem in "Design Twitter")

```
YOU: "The home feed is the most interesting design challenge.
When User A opens their feed, they need to see the latest
tweets from everyone they follow, sorted by time. There
are two fundamental approaches, and I want to walk
through both before choosing."

[APPROACH 1: Fan-Out on Read (Pull Model)]

YOU: "Option 1 is fan-out on read. When User A opens their
feed, the system queries: 'who does A follow?' Gets back,
say, 500 user IDs. Then for each of those 500 users, it
fetches their most recent tweets. Merges everything. Sorts
by timestamp. Returns the top 20.

The advantage: it's simple. No precomputation. No extra
storage. Tweets are always fresh.

The problem: it's SLOW. If User A follows 500 people,
that's 500 database queries PER feed request. At 600K
feed requests per second, that's 300 million DB queries
per second just for feeds. That won't work."

[APPROACH 2: Fan-Out on Write (Push Model)]

YOU: "Option 2 is fan-out on write. When User B posts a
tweet, the system immediately pushes that tweet into every
follower's precomputed feed in the cache. So when User A
opens their feed, they just read from their personal cache
entry — one Redis GET. Sub-millisecond.

The advantage: reads are O(1). Lightning fast.
The problem: the celebrity problem. If a user has 50 million
followers, posting one tweet triggers 50 million cache writes.
That's expensive and slow."

[THE HYBRID SOLUTION]

YOU: "The solution — and this is what Twitter actually does —
is a hybrid. For normal users with under 10,000 followers,
we fan out on write. Their tweets get pushed into followers'
caches at write time. For celebrities — users with over
10,000 followers — we DON'T fan out at write time.

Instead, when User A opens their feed, the system:
1. Reads A's precomputed feed cache (has all 'normal' follows)
2. Fetches the latest tweets from the celebrities A follows
   (a small, bounded number of queries)
3. Merges and sorts

This gives us O(1) read speed for 99% of the feed, with
a small, bounded real-time query for the celebrity tweets."

INTERVIEWER: "How do you determine the 10K follower threshold?"

YOU: "It's configurable. You'd monitor system performance
and tune it. The key insight is that the distribution is
extremely skewed — maybe 0.1% of users have over 10K
followers. So the extra real-time queries per feed load
is typically 2-5, not 500. The 99.9% case is a cache hit
plus a handful of targeted queries."

WHY THIS WORKS:
• You showed BOTH options before choosing. This proves you
  considered alternatives — the interviewer's favorite signal.
• You named a REAL problem (celebrity problem) and solved it.
• You referenced what the REAL system does ("this is what
  Twitter actually does") — shows industry awareness.
• When pushed back, you had a quantitative answer (0.1%
  of users), not a hand-wave.
```

### Deep Dive Example: API Design

When the interviewer asks about APIs, give them actual endpoints:

```
YOU: "For the API layer, I'd design RESTful endpoints.
Let me define the two critical ones."

POST /api/v1/tweets
Headers: Authorization: Bearer <token>
Body: {
  "text": "Hello world",
  "media_ids": ["abc123"],
  "reply_to": null
}
Response: 201 Created
{
  "tweet_id": "1234567890",
  "created_at": "2026-03-27T22:00:00Z",
  "author_id": "user_456"
}

GET /api/v1/feed?cursor=<tweet_id>&limit=20
Headers: Authorization: Bearer <token>
Response: 200 OK
{
  "tweets": [...],
  "next_cursor": "tweet_12345"
}

YOU: "I'm using cursor-based pagination instead of offset.
With offset, if new tweets are inserted while the user is
scrolling, they'll see duplicates or skip items. Cursor-based
pagination is stable — 'give me 20 tweets older than this
cursor' always returns the correct set regardless of new
inserts."

WHY THIS DETAIL MATTERS: Most candidates say "I'd have a
POST endpoint for tweets and a GET endpoint for the feed."
By writing actual request/response bodies and explaining
cursor pagination, you show implementation-level thinking.
```

### Deep Dive Example: Schema Design

```
YOU: "For the data model, I'd use PostgreSQL for the core
tables. Let me sketch the schema."

CREATE TABLE users (
    user_id      BIGINT PRIMARY KEY,    -- Snowflake ID
    username     VARCHAR(30) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    bio          VARCHAR(280),
    created_at   TIMESTAMP DEFAULT NOW()
);

CREATE TABLE tweets (
    tweet_id     BIGINT PRIMARY KEY,    -- Snowflake ID
    author_id    BIGINT REFERENCES users(user_id),
    text         VARCHAR(280) NOT NULL,
    reply_to     BIGINT REFERENCES tweets(tweet_id),
    media_url    VARCHAR(500),
    created_at   TIMESTAMP DEFAULT NOW(),
    like_count   INT DEFAULT 0,
    retweet_count INT DEFAULT 0
);

CREATE TABLE follows (
    follower_id  BIGINT REFERENCES users(user_id),
    followee_id  BIGINT REFERENCES users(user_id),
    created_at   TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (follower_id, followee_id)
);

CREATE INDEX idx_tweets_author
    ON tweets(author_id, created_at DESC);
CREATE INDEX idx_follows_followee
    ON follows(followee_id);

YOU: "Two key design choices to explain:

First, I'm using Snowflake IDs instead of auto-increment.
Snowflake gives me 64-bit time-sortable unique IDs that
work across distributed databases without coordination.
The first 41 bits are a timestamp, so sorting by tweet_id
is the same as sorting by time.

Second, the composite index on tweets(author_id, created_at
DESC) is critical. The query 'give me user X's most recent
tweets' — which is the fan-out-on-read query — uses this
index to jump directly to the right author and scan in
reverse chronological order. Without it, every feed load
triggers a full table scan."

WHY THIS MATTERS: The schema is where you prove you're not
just an architect — you're an engineer. Indexes especially.
If you can explain WHY an index exists and what query it
optimizes, you demonstrate a level of depth that 90% of
candidates don't reach.
```

---

## 28.6 Phase 5: Scaling and Trade-Offs (Minutes 35-45)

### The Scaling Conversation Script

```
INTERVIEWER: "What happens when traffic 10x's?"

YOU: "Let me walk through each layer.

THE API LAYER scales horizontally. The servers are stateless —
no session data, no local state. To handle 10x traffic, I add
10x servers behind the load balancer. No code changes.

THE CACHE LAYER — Redis — handles 100K+ QPS per node. At
current load, we're at maybe 500K reads/sec with 80% cache
hit rate hitting Redis around 400K QPS. For 10x, I'd add
Redis nodes with consistent hashing to distribute keys.
Redis Cluster handles this natively.

THE DATABASE is where it gets interesting. Two strategies:

Read replicas — I add 3-5 read replicas. The primary handles
all writes (30K writes/sec at 10x — still manageable for
Postgres). Reads are distributed across replicas. Replication
lag means reads might be 100ms stale, but we already agreed
eventual consistency is fine.

Sharding — if we hit 100x, I'd shard the tweets table by
user_id using consistent hashing. Each shard holds a subset
of users. The query 'get User X's tweets' hits exactly one
shard. Cross-shard queries like 'trending tweets' would
require a scatter-gather pattern or a dedicated analytics
pipeline.

The trade-off with sharding: cross-shard joins are expensive
or impossible. I'd avoid sharding until read replicas are
insufficient."

WHY THIS ANSWER WORKS:
• You walked through EACH layer, not just "add more servers."
• You gave specific numbers: "400K QPS hitting Redis."
• You presented strategies in ORDER of complexity: replicas
  first (simple), sharding second (complex).
• You named a real trade-off: "cross-shard joins are
  expensive." This is the kind of nuance interviewers love.
```

### Database Scaling — The Three Strategies

```
STRATEGY 1: READ REPLICAS (try this first)
──────────────────────────────────────────
• Primary handles writes. Replicas handle reads.
• Easy setup: Postgres streaming replication.
• Trade-off: replication lag (100ms-1sec staleness).
• Scales reads to ~5x with 5 replicas.

STRATEGY 2: SHARDING (when replicas aren't enough)
───────────────────────────────────────────────────
• Split data across multiple databases by shard key.
• Good shard keys: user_id, hash(entity_id)
• Bad shard keys: timestamp (hot shard), country (skew)
• Trade-off: cross-shard queries are expensive. Once
  you shard, you can't easily JOIN across shards.

STRATEGY 3: NOSQL (built for horizontal scaling)
─────────────────────────────────────────────────
• Cassandra, DynamoDB — designed for massive write
  throughput and simple access patterns.
• Trade-off: limited query flexibility, eventual
  consistency, no JOINs.

WHAT TO SAY IN THE INTERVIEW:
"I'd start with read replicas because they're simple and
handle our 200:1 read/write ratio perfectly. If we need
to scale further, I'd shard by user_id — our access
pattern is per-user, so most queries hit a single shard.
I'd avoid sharding as long as possible because it adds
significant operational complexity."
```

---

## 28.7 The Trade-Off Framework — CAP and PACELC

### CAP Theorem — What to Say

```
CP Systems (choose consistency over availability):
  Banking, inventory, booking systems.
  "If a network partition happens, some requests fail
   rather than returning stale data."
  Examples: MongoDB, HBase, ZooKeeper

AP Systems (choose availability over consistency):
  Social feeds, caches, counters.
  "If a partition happens, we still serve requests,
   even if the data might be slightly stale."
  Examples: Cassandra, DynamoDB, CouchDB

THE INTERVIEW SCRIPT:
"For the feed service, I'd choose AP — eventual consistency.
If a new tweet takes 2-3 seconds to appear in every follower's
feed, that's fine. Users won't notice. This lets me use async
fan-out with message queues, which keeps write latency low
and availability high.

For a payment service, I'd choose CP — strong consistency.
A double-charge or missed payment is unacceptable. I'd use
a transactional database with synchronous replication, and
I'd accept that some requests might fail during a partition
rather than risk inconsistent financial data."

WHY THIS WORKS: You didn't just define CAP — you APPLIED it
to two different parts of the SAME system. This shows the
interviewer you understand that consistency is a per-service
choice, not a system-wide decision.
```

### PACELC — The More Nuanced Framework

```
CAP only covers partition scenarios. PACELC extends it:

IF Partition → choose Availability or Consistency
ELSE (normal operation) → choose Latency or Consistency

PA/EL: Available during partitions, fast normally
  → Cassandra, DynamoDB → social feeds, caches

PC/EC: Consistent always, even if slower
  → PostgreSQL, ZooKeeper → banking, booking

PA/EC: Available during partitions, consistent normally
  → Most practical systems — the sweet spot
```

---

## 28.8 Common Patterns — Quick Reference with Interview Scripts

### Pattern 1: Publisher-Subscriber (Pub/Sub)

```
USE CASE: Decoupling producers from consumers.

WHAT TO SAY:
"When a user posts a tweet, the API server doesn't directly
call the feed service, notification service, and search
service. Instead, it publishes a 'tweet.created' event to
Kafka. Each downstream service independently consumes from
the topic. This means if the search service goes down, tweets
still post and feeds still update. When search recovers, it
processes the backlog."
```

### Pattern 2: CQRS (Command Query Responsibility Segregation)

```
USE CASE: When read and write patterns are very different.

WHAT TO SAY:
"The write model uses normalized PostgreSQL — optimized for
transactional integrity. The read model uses a denormalized
view in Redis — optimized for fast queries. Events from the
write DB keep the read model in sync. This lets me optimize
each path independently. Writes get ACID guarantees. Reads
get sub-millisecond latency."
```

### Pattern 3: Consistent Hashing

```
USE CASE: Distributing data across nodes.

WHAT TO SAY:
"Instead of hash(key) % num_servers — which remaps everything
when you add a server — I'd use consistent hashing. Servers
sit on a hash ring. Each key maps to the next server clockwise.
Adding a server only moves ~1/N of keys. This is how Cassandra
and DynamoDB distribute data."
```

---

## 28.9 Unique ID Generation

```
OPTION 1: AUTO-INCREMENT
  Simple. Breaks with sharding (collisions). Sequential = predictable.

OPTION 2: UUID (v4)
  128-bit random. Globally unique. NOT sortable by time. 36 chars.

OPTION 3: SNOWFLAKE ID (recommended)
  64-bit: timestamp (41 bits) + machine ID (10) + sequence (12)
  Time-sortable. Unique across machines. Compact (8 bytes).
  4,096 IDs per millisecond per machine.

WHAT TO SAY:
"I'd use Snowflake-style IDs. They're time-sortable, so
sorting by ID is the same as sorting by creation time —
which makes feed queries efficient. They're globally unique
without coordinating between servers. And they're 64 bits
instead of UUID's 128, which saves storage and index space."
```

---

## 28.10 Communication Protocol — How to Talk During the Interview

### The Signpost Technique

Always tell the interviewer where you are and where you're going:

```
OPENING:
"I'm going to start with requirements, then do a quick
estimation, draw a high-level architecture, and then deep-dive
into the most complex components."

TRANSITIONS:
"I've covered the API layer. Now I want to talk about the
data model — specifically the schema and indexing strategy."

CHECK-INS (every 5 minutes):
"Before I go deeper on caching, is there a different area
you'd prefer I focus on?"

"I realize I haven't talked about failure modes yet. Should
I cover that, or continue with the scaling discussion?"

TRADE-OFF NARRATION:
❌ "I'd use Cassandra for storage."
✅ "For storage, I'm choosing between SQL and NoSQL. This
   workload is write-heavy with simple key-value access
   patterns. I'd lean toward Cassandra — it handles high
   write throughput and scales horizontally. The trade-off
   is that ad-hoc queries are harder, but for this use case,
   I don't need them."
```

---

## 28.11 Self-Assessment Rubric

After every practice session, grade yourself:

| Dimension | Question | Target |
|---|---|---|
| **Requirements** | Did I clarify scope before designing? | 4+ |
| **Estimation** | Did I translate numbers into design decisions? | 4+ |
| **High-Level Design** | Did I draw a clear diagram with data flow? | 4+ |
| **Data Model** | Did I write a schema with keys and indexes? | 4+ |
| **API Design** | Did I define endpoints with request/response? | 4+ |
| **Deep Dive** | Did I go deep on 2+ components with real detail? | 4+ |
| **Trade-offs** | Did I articulate 3+ trade-offs with justification? | 4+ |
| **Scaling** | Did I discuss 10x and 100x growth strategies? | 4+ |
| **Communication** | Did I signpost, check in, and use diagrams? | 4+ |

---

## 28.12 Chapter Summary

### The Method (45 Minutes)
1. **Requirements** (5 min) — Ask 5-8 questions. Restate the scope. Get agreement.
2. **Estimation** (2 min) — Calculate QPS, storage, bandwidth. Translate to design decisions.
3. **High-Level Design** (5 min) — Draw the skeleton. Walk through data flow. Ask which area to focus on.
4. **Deep Dive** (23 min) — Schema, APIs, data flow for 2-3 critical components. Show alternatives before choosing.
5. **Scaling & Trade-offs** (10 min) — Walk through each layer. Replicas → sharding → NoSQL.

### The Communication Protocol
- Signpost every transition
- Show alternatives before choosing
- Narrate trade-offs: "I chose X over Y because Z"
- Check in every 5 minutes: "Does this direction make sense?"
- Use diagrams — always draw as you talk

---

*Next Chapter: Chapter 29 — The Building Blocks: Deep Dives into Databases, Caching, Queuing, and More →*
