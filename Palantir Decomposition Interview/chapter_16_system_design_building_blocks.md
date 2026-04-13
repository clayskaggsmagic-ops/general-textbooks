# Chapter 16: The Building Blocks — A Beginner's Guide to Every Piece of a Software System

> *Chapter 15 taught you how to run the interview and the basic vocabulary. This chapter goes deeper on each building block — databases, caches, queues, and more. Everything is explained from scratch in plain English, with the technical name in parentheses so you can sound fluent in the interview.*

---

## 16.1 Databases — How Data Is Stored

In Chapter 15, we introduced databases. Now let's go deeper into the types, when to use each one, and how to talk about them.

### Relational Databases (SQL) — The Default Choice

**What they are:** Tables with rows and columns, where tables can be linked to each other. You query them using SQL (Structured Query Language).

**The key features that make SQL powerful:**

**1. Relationships (JOINs):** The ability to combine data from multiple tables in a single query. Example — you have a `users` table and a `tweets` table. A JOIN lets you ask: "Give me the username AND all the tweets for user 123" in one query, even though the data lives in two separate tables. The database follows the link (`author_id` in the tweets table points to `user_id` in the users table) and combines the results.

Think of it like cross-referencing two spreadsheets: "Look up this person's name in Sheet A, then find all their orders in Sheet B."

**2. Transactions (ACID):** Imagine you're transferring $100 from Account A to Account B. That requires two steps: subtract $100 from A, then add $100 to B. What if the system crashes between step 1 and step 2? Account A lost $100 and Account B never got it — the money vanished. A **transaction** groups both steps together so they either BOTH happen or NEITHER happens. If anything goes wrong, the whole thing is rolled back as if it never started.

ACID is the set of guarantees that make transactions safe:
- **A — Atomicity (all or nothing):** Either every step in the transaction succeeds, or none of them do. No partial updates.
- **C — Consistency (valid states only):** The database always moves from one valid state to another. If a rule says "account balance can't be negative," a transaction that would violate this is rejected.
- **I — Isolation (no interference):** If two transactions are happening at the same time — say, two people try to buy the last concert ticket simultaneously — they don't interfere with each other. Only one will succeed.
- **D — Durability (survives crashes):** Once the database says "done," the data is permanently saved. Even if the power goes out 1 millisecond later, your data is safe on disk.

> **When to say "I need ACID":** Anything involving money, inventory, bookings, or any operation where a partial update would be a disaster.

**3. Flexible Queries:** With SQL, you can ask almost any question about your data — "how many users signed up last month?", "what's the average purchase amount by country?", "which products have never been ordered?" You don't need to know the questions in advance; you can explore the data freely.

**Popular SQL databases:** PostgreSQL (Postgres) and MySQL — these handle the vast majority of web applications.

---

### Non-Relational Databases (NoSQL) — Special-Purpose Tools

Sometimes the "spreadsheet with relationships" model isn't the best fit. NoSQL databases store data differently, each type optimized for a specific use case. Think of them as specialized tools instead of a general-purpose Swiss Army knife.

**There are four families of NoSQL databases. Here's each one explained:**

#### 1. Key-Value Stores (like a dictionary)

**What it is:** The simplest possible database. You store a piece of data under a name (the "key"), and later you retrieve it by that same name. That's it. No tables, no columns, no relationships — just "save this data under this name" and "get the data saved under this name."

**Analogy:** A coat check at a restaurant. You hand over your coat (the value), they give you a numbered ticket (the key). Later, you hand back the ticket and they give you your coat. They don't let you search for "all the blue coats" — you need the ticket number.

**Examples:** Redis, DynamoDB

**When to use:**
- Storing a user's session data (their login state) — you know the session ID (the key), and you just need to grab the data associated with it
- Caching precomputed results — like a user's pre-built news feed, stored under their user ID
- Any time you need extremely fast lookups (under 1 millisecond) and you always know the exact key

**The limitation:** You can only look things up by key. You can't ask "find all entries where the value contains X" — you'd have to check every single entry. If you need to search, query, or filter, a key-value store isn't the right tool.

> **Interview phrasing:** "I'd use Redis here because I need sub-millisecond lookups by a known key — I always know the user ID, so I can store and retrieve their precomputed feed in one operation."

#### 2. Document Stores (like a filing cabinet of folders)

**What it is:** Each record is a "document" — think of it as a JSON file (a blob of structured text) that can contain whatever fields you want. Unlike a SQL table where every row has the same columns, each document can have different fields.

**Analogy:** A filing cabinet where each folder contains a different set of papers. Patient A's folder might have X-rays and lab results. Patient B's folder might have surgical notes and referrals. They're not identical in structure, but they're all "patient records."

**Examples:** MongoDB, Firestore (Google's document database)

**When to use:**
- User profiles where different users have different fields — one user has a LinkedIn URL, another has a TikTok handle, a third has neither
- Product catalogs where different products have different attributes — a T-shirt has "size" and "color," a laptop has "RAM" and "screen size"
- Any data where the structure varies from record to record

**The limitation:** Relationships between documents (like "show me all orders placed by this customer") are harder than in SQL. Document stores work best when each document is self-contained.

> **Interview phrasing:** "Each record in this system has a different structure — some entries have 5 fields, others have 50. A document store handles this naturally without forcing a rigid schema."

#### 3. Wide-Column Stores (like a spreadsheet on steroids)

**What it is:** Designed for storing massive volumes of data with very high write speed. Data is organized by a **partition key** (a field that determines which machine stores the data) and a **clustering key** (a field that determines the sort order within that partition).

**Analogy:** Imagine you have a warehouse full of filing cabinets. Each cabinet is labeled with a device ID (the partition key). Inside each cabinet, the files are sorted by date (the clustering key). Finding "all readings from Device X between March 1 and March 15" means walking to Cabinet X and pulling the files from that date range. Very fast, very predictable.

**Examples:** Cassandra, HBase, ScyllaDB

**When to use:**
- Time-series data — sensor readings, server logs, stock prices — where you're writing millions of data points per second and querying by "device + time range"
- Any workload with massive write volume and simple, predictable query patterns

**The limitation:** You must design your data model around your query patterns. If you decide later that you need to query the data differently, you might have to restructure everything. It's not flexible like SQL.

> **Interview phrasing:** "This is time-series data — millions of sensor readings per second. Cassandra is designed for exactly this: I partition by device ID and cluster by timestamp, so the query 'give me all readings from Device X in the last hour' hits exactly one partition and scans in order."

#### 4. Graph Databases (like a social network map)

**What it is:** Instead of organizing data into tables or documents, a graph database organizes it as **nodes** (things) and **edges** (connections between things). It's specifically designed to answer questions about relationships — like "who are the friends of my friends?" or "what's the shortest path between Person A and Person B?"

**Analogy:** Think of a conspiracy board with photos connected by strings. Each photo is a node (a person, place, or thing). Each string is an edge (a relationship — "knows," "works at," "visited"). A graph database lets you trace through those strings very efficiently.

**Examples:** Neo4j, Amazon Neptune

**When to use:**
- Social networks — "friends of friends who are also in New York"
- Fraud detection — "find clusters of accounts that all share the same phone number or address"
- Recommendation engines — "people who bought X also bought Y, and people who bought Y also bought Z"

**The limitation:** For simple lookups ("get user 123's profile"), a graph database is overkill. It shines when the *relationships* between data points are the most important thing.

> **Interview phrasing:** "Finding 'friends of friends who like topic X' in SQL would require joining the friends table to itself 5 times — each join multiplies the work and gets exponentially slower. In a graph database, traversing relationships is what it's designed for — each 'hop' to the next person is a fast, constant-time operation."

---

### How to Choose: SQL vs. NoSQL Decision Guide

| If you need... | Use... | Because... |
|---|---|---|
| Transactions (all-or-nothing operations) | SQL (PostgreSQL, MySQL) | NoSQL databases generally don't support multi-record transactions |
| Complex queries combining multiple types of data | SQL | JOIN operations let you combine tables; NoSQL doesn't have this |
| The flexibility to ask any question about your data | SQL | You can write any SQL query; NoSQL requires pre-planned access patterns |
| Extremely fast lookups by a known key | Key-value (Redis, DynamoDB) | O(1) lookup — directly to the answer, no search needed |
| Data where each record has a different structure | Document (MongoDB, Firestore) | No rigid schema — each document can have different fields |
| Massive write volume (100K+ writes per second) | Wide-column (Cassandra) | Designed for high-throughput distributed writes |
| Finding complex relationships between entities | Graph (Neo4j, Neptune) | Traverses connections efficiently |

---

### Indexes — How Databases Find Data Fast

We introduced indexes briefly in Chapter 15. Here's the deeper explanation you need.

**The problem:** If your database has 1 billion tweets and you ask "give me all tweets by author 123," the database has two options:

1. **Without an index:** Check every single row. All 1 billion of them. This is called a **full table scan**, and it's extremely slow.
2. **With an index on the `author_id` column:** The database maintains a sorted lookup structure (like a phone book sorted by name). It jumps directly to all rows where `author_id = 123`. This takes a fraction of a second instead of minutes.

**Types of indexes you should know:**

**B-Tree index (the default):** The standard index type. It organizes data in a tree structure that's efficient for equality lookups ("find author 123"), range queries ("find tweets created after March 1"), and sorting ("order by date"). When someone just says "index," they usually mean a B-Tree index.

**Hash index:** Uses a mathematical function to map the key directly to the row. Extremely fast for exact lookups ("find author 123") but can NOT do range queries ("find tweets created after March 1") or sorting. It's like the difference between a phone book (B-Tree — sorted, can browse) and a locker room (Hash — you know the locker number, go directly there, but you can't browse "all lockers between 50 and 60").

**Composite index:** An index on two or more columns together. Example: an index on `(author_id, created_at)` lets the database efficiently answer "give me all tweets by author 123, sorted by date." The order of columns matters — an index on `(A, B)` helps queries that filter on A, or on A+B, but NOT queries that filter on B alone. Think of a phone book sorted by last name, then first name: you can look up "Smith" or "Smith, John," but not just "John."

**The trade-off of indexes:** Every index speeds up reads but slows down writes. When you add a new tweet, the database has to update every index that includes that tweet's data. If you have 10 indexes, every write is 10x more work. So you only create indexes on columns that you frequently search or sort by.

> **Interview phrasing:** "I'm adding a composite index on `(author_id, created_at DESC)`. The most common query — 'give me this user's latest tweets' — uses `author_id` to find the right person and `created_at DESC` to sort them newest-first. Without this index, the database scans every tweet to find the matching ones. With it, the database jumps directly to the right author and reads in order. At 3,000 writes per second, the write overhead of maintaining this index is negligible."

---

## 16.2 Caching — Making Reads Lightning Fast

We covered caching basics in Chapter 15 (the "sticky note on your desk" analogy). Here's the deeper knowledge you need for interviews.

### When Should You Cache?

Cache when:
- ✅ The same data is read many times (like a celebrity's profile — millions of people view it)
- ✅ Computing the result is expensive (like assembling a news feed from 500 different users' tweets)
- ✅ Slight staleness is acceptable (if someone's bio is 30 seconds out of date, that's fine)
- ✅ The "hot" data (the most frequently accessed stuff) fits in memory

Don't cache when:
- ❌ Data changes faster than it's read (the cache would be perpetually stale)
- ❌ Every request needs unique, never-repeated data (every request would be a cache miss — the cache adds no value)
- ❌ You absolutely need real-time accuracy (the cache might show data that's a few seconds old)

### How Does the Cache Stay Up to Date?

The data in your database changes. How does the cache know when its copy is outdated? There are three main strategies:

**1. Time-based expiration (TTL — Time to Live):**
Set an expiration timer on each cache entry. After, say, 5 minutes, the entry is automatically deleted. The next request finds the cache empty, goes to the database for fresh data, and re-populates the cache.

- *Pro:* Dead simple. No extra systems needed.
- *Con:* Data can be up to 5 minutes stale.
- *When to use:* When slight staleness is OK — user profiles, product listings, any data that changes infrequently.

> **Interview phrasing:** "I'd set a 5-minute TTL on user profile caches. Worst case, someone sees a bio that's 5 minutes old. The simplicity is worth that trade-off."

**2. Event-driven invalidation (delete the cache entry when the data changes):**
When someone updates their profile, the system publishes an event (a message saying "user 123's profile was updated"). A listener receives this event and immediately deletes the cache entry for user 123. The next time someone requests that profile, the cache is empty, so it fetches fresh data from the database.

- *Pro:* Cache is almost always fresh (within seconds).
- *Con:* Requires a message queue and more infrastructure.
- *When to use:* When freshness matters more — like after a user edits their profile, they should see the update immediately.

**3. Write-through (update the cache at the same time as the database):**
Every time you write to the database, you also write the same data to the cache. The cache is always current.

- *Pro:* Cache is always perfectly up to date.
- *Con:* Writes are slower because you're writing to two places.
- *When to use:* When data is read very frequently and immediately after being written — like session data (the record of a user's login state) that's checked on every single request.

### The "Thundering Herd" Problem

**The scenario:** Imagine a very popular cache entry — say, the profile of a celebrity with 50 million followers. The cache entry expires (its TTL runs out). At that exact moment, 10,000 people happen to request that profile simultaneously. All 10,000 find the cache empty. All 10,000 go to the database. The database gets hit with 10,000 identical queries at once and could crash under the load.

**The fix:** Only allow ONE of those 10,000 requests to actually go to the database. That one request fetches the data, puts it back in the cache, and the other 9,999 requests read from the now-refreshed cache. This is done with a simple lock — the first request to arrive "claims" the right to refresh the cache; everyone else waits briefly.

> **Interview phrasing:** "I'd use a cache lock to prevent thundering herd. When a popular key expires, only one request refreshes from the database; the others either wait a few milliseconds or receive a slightly stale version. This caps the database load at 1 query instead of 10,000."

---

## 16.3 Message Queues — Doing Work in the Background

We covered message queues in Chapter 15 (the "restaurant ticket rail" analogy). Here's the deeper knowledge.

### Why Queues Exist — A Before/After

**Without a queue:** A user posts a tweet. The server saves it to the database, THEN sends notifications to 10,000 followers, THEN updates the search index, THEN runs content moderation. The user waits 3-5 seconds for all of this to finish. Terrible experience.

**With a queue:** The server saves the tweet, puts a message on the queue ("a tweet was created"), and immediately tells the user "done!" (50 milliseconds). Separate worker servers read messages from the queue and handle notifications, search indexing, and moderation in the background. The user never waits for any of it.

**The bonus:** If the notification system temporarily goes down, the messages sit in the queue and wait. When the notification system comes back online, it processes the backlog. No messages are lost. The tweet-posting system doesn't even know the notification system was down.

### Types of Message Queues

**Kafka (the most common choice for large systems):**

What it is: Think of Kafka as a giant, permanent, append-only (add to the end only) log. Messages are written to the end of the log and stay there for a configurable time (often 7 days). Multiple different systems can read from the same log independently, each keeping track of where *they* left off (their "offset," meaning their position in the log).

What makes it special:
- **Replay:** If you deploy a buggy system that processes messages incorrectly, you can rewind to last Tuesday and reprocess everything. The messages are still in the log because Kafka keeps them.
- **Multiple readers:** The notification system, the search system, and the analytics system can all read from the same "new tweet" log independently. Adding a new reader doesn't affect the others.
- **Massive throughput:** Kafka can handle millions of messages per second.

When to use: Event streaming — when many systems need to react to the same events, and you might need to replay events later.

**RabbitMQ (the task queue):**

What it is: A traditional message queue where messages are delivered to a worker, the worker processes them, and the messages are deleted. Think of a literal to-do list where items are checked off.

What makes it special:
- **Priority queues:** Some tasks are more important than others. A payment processing task should jump ahead of an email notification.
- **Dead letter queue:** If a message fails to process 3 times (maybe the data is corrupted), it gets moved to a separate "dead letter queue" for a human to investigate later, instead of blocking the entire system.
- **Per-message TTL:** Each message can have its own expiration — "if this notification hasn't been sent within 1 hour, don't bother."

When to use: Task processing — when you need "process this specific job exactly once" semantics with features like retries, priorities, and failure routing.

**SQS (Simple Queue Service — Amazon's managed queue):**

What it is: A queue that Amazon runs for you. You don't manage any servers or infrastructure — you just put messages in and take messages out.

When to use: When you're already on Amazon Web Services (AWS) and want a simple queue without operating any infrastructure. The trade-off: less control than Kafka or RabbitMQ.

### Message Delivery Guarantees

When a system sends a message, there are three levels of guarantee about whether the message arrives and how many times:

**At-most-once ("fire and forget"):** The message is sent once and the system doesn't check if it was received. If something goes wrong, the message is lost. Like shouting across a noisy room — maybe they heard you, maybe not.
- When to use: Logging, metrics, non-critical analytics — where losing an occasional data point isn't a disaster.

**At-least-once ("keep trying until confirmed"):** The system keeps re-sending the message until the receiver confirms ("ACK" — short for "acknowledgment") that it processed it. If the receiver crashes after processing but before confirming, the message is re-sent — meaning it might be processed twice.
- When to use: Most things. This is the default for most systems.
- **The important concept here — Idempotency:** Since messages might arrive twice, your processing logic needs to handle duplicates gracefully. "Idempotent" means "doing the same operation twice has the same effect as doing it once." Example: "set the user's balance to $100" is idempotent (doing it twice still results in $100). "Add $100 to the user's balance" is NOT idempotent (doing it twice results in $200 instead of $100). You make the second type idempotent by giving each message a unique ID and checking: "have I already processed message ABC123? Yes? Skip."

**Exactly-once ("no losses, no duplicates"):** The holy grail. In practice, this is nearly impossible in distributed systems (systems running on multiple computers). The practical approach: use at-least-once delivery + idempotent processing to *effectively* get exactly-once behavior.

> **Interview phrasing:** "I'd design for at-least-once delivery with idempotent consumers. Each notification has a unique ID. Before sending, the worker checks: 'have I already sent notification ABC123?' If yes, skip. This gives me effectively exactly-once behavior without the complexity of distributed transactions."

---

## 16.4 Load Balancing — How Traffic Gets Distributed

We introduced the load balancer in Chapter 15. Here's the extra detail.

### Layer 4 vs. Layer 7 Load Balancing

These "layers" refer to the OSI model — a framework that describes the different levels of network communication. You don't need to memorize the whole model, just these two layers:

**Layer 4 (the "transport" layer):** The load balancer routes traffic based only on the IP address and port number (the destination address of the request). It's fast because it doesn't inspect the actual content of the message. Think of a mail room sorting packages by address number — it doesn't open the package to see what's inside.

**Layer 7 (the "application" layer):** The load balancer actually opens and inspects the HTTP request — it can see the URL path, headers, and even the request body. This lets it make smarter decisions, like routing `/api/users/*` to one set of servers and `/api/tweets/*` to a different set.

> **Interview phrasing:** "I'd use a Layer 7 load balancer for HTTP traffic — it can route `/api/users/*` to the user service and `/api/tweets/*` to the tweet service. It also handles SSL termination (decrypting the secure connection) so the backend servers don't have to."

### How the Load Balancer Chooses Which Server

**Round Robin:** Give each server a turn in order. Server 1, then Server 2, then Server 3, then back to Server 1. Simple and works well when all servers have the same capacity.

**Least Connections:** Send the request to whichever server is currently handling the fewest requests. Better when some requests take much longer than others.

**Health Checks:** The load balancer regularly pings each server ("are you alive? are you healthy?"). If a server doesn't respond, the load balancer stops sending it traffic until it recovers.

---

## 16.5 API Design — How the Client Talks to the Server

### REST vs. GraphQL vs. gRPC

These are three different styles for designing the communication between clients (apps/browsers) and servers:

**REST (the default and most common):**

Short for **Representational State Transfer.** In practical terms, it means organizing your API around "resources" (things — users, tweets, orders) and using HTTP methods (GET, POST, PUT, DELETE) to act on them.

- `GET /users/123` → Get user 123's data
- `POST /tweets` → Create a new tweet
- `DELETE /tweets/456` → Delete tweet 456

*Pro:* Simple, widely understood, works with any programming language.
*Con:* "Over-fetching" — if you only need a user's name, you still get their entire profile (bio, email, creation date, etc.) because the endpoint returns the full object.

**GraphQL (for complex frontends with different data needs):**

Instead of the server deciding what data to return, the *client* specifies exactly which fields it wants. The client sends a "query" describing the shape of the data it needs:

```
{
  user(id: 123) {
    name
    profilePicture
  }
}
```

This returns ONLY the name and profile picture — nothing else.

*Pro:* No over-fetching. A mobile app (which needs less data) and a desktop app (which needs more data) can use the same API with different queries.
*Con:* More complex to build on the server side. Clients could accidentally request enormous amounts of data if you don't set limits.

**gRPC (for internal communication between servers):**

A faster, more compact format for servers to talk to each other. Instead of sending human-readable text (JSON) over HTTP, gRPC sends compact binary data (Protocol Buffers). Much faster, but not designed for use directly by web browsers.

*Pro:* 5-10x faster than REST for server-to-server communication. Strongly typed — errors are caught at compile time, not at runtime.
*Con:* Not browser-friendly. Requires a translation layer (proxy) for web clients.

> **The rule of thumb:** REST for the public API (clients → server). gRPC for internal APIs (server → server). WebSockets for real-time features (chat, live updates).

### Pagination — How to Return Large Lists

If a user follows 10,000 people, you don't return all 10,000 in one response. You **paginate** — return results in small pages (e.g., 20 at a time).

**Cursor-based pagination (the right way):**
The server returns 20 results and a **cursor** — a bookmark pointing to the last item returned. The client sends the cursor back to get the next 20: "Give me 20 tweets older than this cursor."

*Why this is better than page numbers:* With page numbers, if new tweets are posted while the user is scrolling (shifting all the content down), they'll see duplicate tweets or miss some. A cursor is stable — "give me everything after this specific tweet" always returns the correct results regardless of new inserts.

> **Interview phrasing:** "I'd use cursor-based pagination. With offset-based (page numbers), new inserts cause duplicates and skips. With cursors, each request says 'give me items after this marker' — stable regardless of new writes."

---

## 16.6 Blob Storage and CDNs — Handling Files

### Object Storage (S3) — Where Large Files Live

We mentioned object storage in Chapter 15. The key detail for interviews is **pre-signed URLs** — a pattern for handling file uploads without overwhelming your servers:

**The problem:** If a million users upload photos at the same time and all those uploads go through your API server, your server is overwhelmed — it's spending all its time transferring large files instead of handling API requests.

**The solution — pre-signed URLs:**

1. The client tells the API server: "I want to upload a photo."
2. The API server generates a special temporary URL (the pre-signed URL) that gives the client permission to upload directly to object storage (S3) — bypassing the API server entirely.
3. The client uploads the photo directly to S3 using that URL.
4. The client tells the API server: "My photo is now at this S3 URL."
5. The API server stores the URL in the database alongside the tweet.

This way, your API server never touches the actual file data. All the heavy lifting of file transfer is handled by S3, which is purpose-built for it.

> **Interview phrasing:** "Media never touches the API server. I'd use pre-signed URLs — the client uploads directly to S3. This means a million simultaneous uploads hit S3 (which is designed for it), not our API servers. Our servers stay free to handle API requests."

### CDN Details

Two modes of CDN operation:

**Pull CDN (the default):** The CDN doesn't have your file at first. When a user in Tokyo requests an image, the CDN server in Tokyo fetches it from your origin server in Virginia, caches a local copy, and serves it. The next user in Tokyo gets the cached copy. The CDN "pulls" content as needed.

**Push CDN:** You proactively upload files to the CDN before anyone requests them. Good for content you know will be popular — like a viral video.

**Cache busting:** When you update a file (e.g., a new version of your logo), the CDN might still have the old version cached. Instead of manually clearing the CDN's cache (which is complicated), the common solution is **versioned URLs** — `logo_v2.png` instead of `logo.png`. Since it's a new URL, the CDN treats it as a brand-new file.

---

## 16.7 Search — Finding Things in Large Datasets

**The problem:** If your database has a billion tweets and someone searches for "pizza cat," the database would need to scan every single tweet to find ones containing those words. With a billion records, this could take minutes. Useless.

**The solution — an inverted index:**

A regular index maps "this row → these words." An inverted index does the reverse: "this word → these rows."

Example:
- Tweet 1: "I love pizza"
- Tweet 2: "My cat is cute"  
- Tweet 3: "Pizza cat is the best"

The inverted index would look like:
- "pizza" → [Tweet 1, Tweet 3]
- "cat" → [Tweet 2, Tweet 3]
- "love" → [Tweet 1]
- "cute" → [Tweet 2]
- "best" → [Tweet 3]

Now, searching for "pizza cat" means: find the list for "pizza" ([Tweet 1, 3]), find the list for "cat" ([Tweet 2, 3]), find the overlap → Tweet 3. This is nearly instant, regardless of how many tweets exist.

**Elasticsearch:** The most popular search system. It maintains an inverted index of your data. You keep your main data in your primary database (Postgres) and sync changes to Elasticsearch so the search index stays up to date. There's usually a 1-2 second delay between a tweet being posted and it becoming searchable — this is fine for most applications.

**Change Data Capture (CDC):** The mechanism that syncs data from your main database to Elasticsearch (or any other secondary system). It watches the database for changes — "a new row was inserted," "a row was updated" — and automatically sends those changes to the search system. Think of it as a one-way mirror: the search system always reflects what's in the main database, with a small delay.

> **Interview phrasing:** "I'd use Elasticsearch with an inverted index for search. The primary database (Postgres) is the source of truth. Change Data Capture pipes new tweets to Elasticsearch in near-real-time. The search index is eventually consistent — a new tweet might take 1-2 seconds to become searchable, which is acceptable."

---

## 16.8 Monolith vs. Microservices — How to Organize Your Code

**Monolith:** All of your code — the feed system, the user system, the notification system, the search system — lives in one single application. You deploy the whole thing together. When you make a change to the notification code, you redeploy the entire application.

*Pro:* Simple. Easy to develop, test, and deploy. One codebase, one deployment, one thing to monitor.
*Con:* As the codebase grows, it becomes harder to manage. A bug in the notification code can crash the entire system — including the feed and user systems.

**Microservices:** Each major feature (feed, users, notifications, search) is its own separate application with its own codebase, its own database, its own deployment. They communicate with each other over the network (via APIs or message queues).

*Pro:* Each service can be scaled independently. If the search system needs more capacity, you add more search servers without touching anything else. A bug in notifications doesn't crash the feed system.
*Con:* Much more complex. Now you have network failures between services, data consistency challenges across databases, and you need infrastructure to manage dozens of separate applications.

> **The mature interview answer:** "I'd start with a modular monolith — a single application, but with clean internal boundaries between modules. As the product grows and we identify parts that need independent scaling — like search indexing — we'd extract those into separate services. Starting with microservices on a new product is premature optimization — you're solving distributed systems problems before you've even figured out what the product should do."

---

## 16.9 Consistency Patterns — How Fresh Does the Data Need to Be?

We covered strong vs. eventual consistency in Chapter 15. Here's one more pattern:

**Read-your-own-writes:** A middle ground. When YOU update your data, YOU always see your own update immediately. Other users might see the update with a small delay.

**How it works:** After you post a tweet, the system routes *your* reads to the primary database (which has the latest data) for the next few seconds. Everyone else's reads go to the replicas (which might be a second behind). After a few seconds, the replicas catch up and everyone sees the same thing.

> **Interview phrasing:** "After a user posts a tweet, THEY must see it in their feed immediately — that's a bad user experience otherwise. So I'd route the posting user's reads to the primary database for the next 5 seconds. Everyone else can read from replicas with a small delay."

---

## 16.10 Security Basics — Authentication and Authorization

**Authentication** = "Who are you?" (Proving your identity)  
**Authorization** = "What are you allowed to do?" (Checking your permissions)

**The most common pattern — JWTs (JSON Web Tokens):**

1. The user logs in with their username and password.
2. The server verifies the credentials and creates a **JWT** — a small, encrypted token containing the user's ID and their role (admin, regular user, etc.).
3. The server sends the JWT back to the client.
4. For every future request, the client includes this JWT in the request header. The server reads the token to know who the user is and what they're allowed to do — without needing to check the database every time.

The JWT has an expiration time (usually 15 minutes to a few hours). When it expires, the client uses a separate **refresh token** to get a new JWT without making the user log in again.

> **Interview phrasing:** "For authentication, I'd use JWTs. The user logs in once and gets a token. Every subsequent request includes the token. The server validates the token's signature locally — no database call needed. The token includes the user's role for authorization checks."

---

## 16.11 Data Processing — Batch vs. Stream

**Batch processing:** Process a large pile of data all at once, on a schedule. Like doing all your laundry on Sunday instead of washing each item as it gets dirty.

*Example:* "Every night at midnight, compute yesterday's engagement metrics — total likes, shares, top posts."
*Latency:* Hours. You don't get results until the job finishes.
*Tools:* Apache Spark, Hadoop MapReduce.

**Stream processing:** Process data as it arrives, in real time. Like a factory assembly line — each item is processed the moment it reaches you.

*Example:* "When a tweet is posted, immediately update the trending topics list."
*Latency:* Milliseconds to seconds.
*Tools:* Kafka Streams, Apache Flink.

> **Interview phrasing:** "For trending topics, I'd use stream processing — process tweet events in real time and maintain a running count of hashtags over a sliding window (the last 1 hour). For the daily analytics dashboard, I'd use batch processing — a nightly job that aggregates the full day's data. Real-time where users need immediacy, batch where they can wait."

---

## 16.12 Observability — How You Know If Something Is Broken

**The four golden signals** (from Google's Site Reliability Engineering handbook):

1. **Latency** — How long requests take. If your average response time suddenly jumps from 100ms to 2 seconds, something is wrong.
2. **Traffic** — How many requests your system is handling. A sudden spike could indicate a viral moment or a DDoS attack.
3. **Errors** — What percentage of requests are failing. If your error rate jumps from 0.1% to 5%, investigate immediately.
4. **Saturation** — How close your resources are to capacity. If your database CPU is at 95%, you're about to have problems.

**Distributed tracing:** In a system with many components (load balancer → server → cache → database → message queue → worker), when a request is slow, how do you know which component caused the delay? Distributed tracing gives every request a unique ID that follows it through every component. You can look at the trace and see: "The request spent 2ms in the server, 3ms in the cache, and 4 seconds in the database — the database is the problem."

> **Interview phrasing:** "I'd monitor the four golden signals — latency, traffic, errors, and saturation. For the feed service specifically, I'd alert if p99 latency (the latency that 99% of requests are faster than) exceeds 500ms, or if the cache hit rate drops below 70%. For debugging, I'd use distributed tracing — every request gets a trace ID that follows it across all services."

---

## 16.13 Quick Reference — When to Use What

| Problem | Building Block | Plain English |
|---|---|---|
| Too many reads hitting the database | Cache (Redis) | "Save common answers on a sticky note instead of running to the filing cabinet every time" |
| User shouldn't wait for slow background work | Message Queue (Kafka) | "Take the order and put it on the kitchen rail; don't make the customer wait for the food to cook" |
| Need to search text across millions of records | Search Engine (Elasticsearch) | "Build an index in the back of the book instead of reading every page" |
| Images/videos need to load fast globally | CDN | "Put copies of files on servers in every major city" |
| Users upload large files | Object Storage (S3) + pre-signed URLs | "Let them upload directly to the warehouse; don't route it through the office" |
| Need real-time updates (chat, notifications) | WebSockets | "A persistent open phone line instead of calling back and forth" |
| Money, bookings, inventory | SQL Database (Postgres) + ACID transactions | "All-or-nothing operations — transfer money safely" |
| Millions of sensor readings per second | Wide-Column Store (Cassandra) | "A warehouse of filing cabinets sorted by device and time" |
| "Friends of friends" queries | Graph Database (Neo4j) | "Follow the strings on the conspiracy board" |

---

## 16.14 Chapter Summary

You now know every major building block of a modern software system — what each one does, when to use it, and how to explain it in plain English with the technical term attached. The goal isn't to memorize scripts — it's to understand the *why* behind each choice so you can reason about trade-offs naturally in an interview.

For each building block, you should be able to answer:
1. **What does it do?** (Explain in one sentence, in plain English)
2. **When would I use it?** (What problem does it solve?)
3. **What's the trade-off?** (What do I give up by using it?)
4. **What would I say in the interview?** (How do I demonstrate depth?)

---

*Next Chapter: Chapter 17 — System Design Practice Problems: Six Complete Worked Designs →*
