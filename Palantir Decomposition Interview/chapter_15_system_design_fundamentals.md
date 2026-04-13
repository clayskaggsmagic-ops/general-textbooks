# Chapter 15: System Design Fundamentals — How to Design a Software System From Scratch

> *This chapter teaches you system design from zero. Every concept is explained in plain English first, with the technical term in parentheses so you know what to call it in the interview. If you read this chapter and understand every section, you'll be able to walk through a system design interview.*

---

## What Is System Design?

When someone asks you to "design a system," they're asking: **how would you build the software and its infrastructure to solve this problem at scale?**

Think of it like designing a restaurant. You need to decide:
- What's on the menu? (What does the system do?)
- How many customers will come? (How much traffic will it handle?)
- How many chefs and waiters do you need? (How many servers?)
- Where does the food come from? (Where is the data stored?)
- What happens during the dinner rush? (How do you handle peak load?)
- What happens if a chef calls in sick? (How do you handle failures?)

A "system design interview" is where the interviewer gives you a vague problem — like "design Twitter" or "design a file storage service" — and you spend 45 minutes thinking through all of these questions out loud.

---

## The Building Blocks — Everything You Need to Know

Before we talk about how to *do* the interview, you need to understand the pieces you'll be designing with. Think of these as the LEGO bricks of software systems.

### The Internet: How Users Talk to Your System

When you open Twitter on your phone, here's what actually happens:

1. **Your phone** (the "client") sends a message over the internet to Twitter's computers.
2. That message is a **request** — like "give me the latest tweets."
3. Twitter's computers process the request and send back a **response** — the actual tweet data.
4. Your phone displays the tweets.

This back-and-forth pattern is called **client-server architecture**. The phone/browser is the client. Twitter's computers are the servers. Every web application works this way.

**API (Application Programming Interface):** The "menu" that the server offers. It defines *what* you can ask for and *how* to ask for it. For example, Twitter's API might say: "If you send a request to `/tweets/latest`, I'll give you the newest tweets. If you send a request to `/tweets/create` with some text, I'll post a tweet."

**Endpoint:** A specific item on the menu. `/tweets/latest` is one endpoint. `/tweets/create` is another. Each endpoint does one thing.

**REST:** A style of organizing your API endpoints. Instead of inventing random names, REST uses simple patterns:
- `GET /tweets` → fetch tweets (reading data)
- `POST /tweets` → create a tweet (writing data)
- `DELETE /tweets/123` → delete tweet #123

**HTTP:** The "language" that clients and servers use to talk to each other. Every time you load a webpage, your browser sends an HTTP request and receives an HTTP response. The request includes a *method* (GET, POST, DELETE) and a *path* (like `/tweets`). The response includes a *status code* (200 = success, 404 = not found, 500 = server error).

### Servers: The Computers That Run Your System

A **server** is just a computer that sits in a data center (a building full of computers) and waits for requests from clients. When you hear "server," think: a computer whose job is to respond to requests.

**Why multiple servers?** One server has limits. It can only handle so many requests at once — typically a few thousand per second. If a million people hit your app at once, one server will crash. So real systems use many servers.

**Stateless vs. Stateful Servers:**
- A **stateless** server doesn't remember anything between requests. Each request is independent. If you send a request to Server A, then your next request goes to Server B, that's fine — B doesn't need to know what A did. This is the ideal. It means any server can handle any request, which makes it easy to add more servers.
- A **stateful** server remembers things — like your login session or your shopping cart. If your first request went to Server A, your next request *must* also go to Server A. This is harder to scale because you're tied to one specific server.

**Rule of thumb for interviews:** Always design stateless servers. Store any "memory" in a separate database or cache, not on the server itself.

### Load Balancer: The Traffic Cop

If you have 10 servers, how does a client's request know which server to go to? It doesn't. Instead, all requests go to a **load balancer** — a piece of software that sits in front of your servers and distributes incoming requests across them.

Think of it like a restaurant hostess: customers walk in the door, and the hostess sends them to whichever table has the most room.

**Why it matters:**
- If one server goes down, the load balancer stops sending it traffic. Users don't notice.
- If you need more capacity, you add more servers behind the load balancer. No code changes needed.
- It prevents any single server from getting overwhelmed.

**Horizontal scaling (scaling out):** Adding more servers behind the load balancer to handle more traffic. This is the standard approach for web systems.

**Vertical scaling (scaling up):** Making a single server bigger (more memory, faster CPU). This has limits — you can only make one machine so powerful.

### Databases: Where Data Lives Permanently

A **database** is a system designed to store and retrieve data reliably. When a user creates an account, posts a tweet, or makes a purchase, that data goes into a database. When they come back later, the database still has it.

**Why not just store data on the server?** Because servers are stateless (remember?), and they can go down. If Server A stores your data in its own memory and then crashes, your data is gone. A database is a *separate* system specifically designed to not lose your data, even when things crash.

#### Relational Databases (SQL Databases)

The most common type. Data is stored in **tables** — like spreadsheets. Each table has **rows** (individual records) and **columns** (the fields/properties of each record).

Example — a `users` table:

| user_id | username | email | created_at |
|---|---|---|---|
| 1 | clay | clay@email.com | 2026-01-01 |
| 2 | sarah | sarah@email.com | 2026-01-15 |

And a `tweets` table:

| tweet_id | author_id | text | created_at |
|---|---|---|---|
| 101 | 1 | "Hello world" | 2026-02-01 |
| 102 | 2 | "Hey everyone" | 2026-02-02 |

The `author_id` column in the `tweets` table points to the `user_id` column in the `users` table. This link between tables is called a **relationship** — and it's why these are called *relational* databases.

**SQL (Structured Query Language):** The language you use to talk to a relational database. It's how you ask it questions:
- "Give me all tweets by user 1" → `SELECT * FROM tweets WHERE author_id = 1`
- "How many users signed up this month?" → `SELECT COUNT(*) FROM users WHERE created_at > '2026-03-01'`

**Popular relational databases:** PostgreSQL (often just called "Postgres"), MySQL. These are the workhorses of the internet — most applications use one of these.

**Primary Key:** The column that uniquely identifies each row. In the `users` table, `user_id` is the primary key — no two users can have the same `user_id`. Think of it as a social security number for each row.

**Index:** A shortcut the database creates to find specific rows faster. Without an index, searching for a specific user means scanning every single row (slow). With an index on the `username` column, the database can jump directly to the right row (fast). It works like the index in the back of a textbook — instead of reading every page to find "photosynthesis," you look it up in the index and it tells you page 47.

**The cost of indexes:** They speed up reading (searching/querying), but they slow down writing (inserting/updating), because the database has to update the index every time data changes. So you only add indexes on columns that you frequently search by.

**Schema:** The blueprint of your database — which tables exist, what columns each table has, what type of data each column holds (text, number, date, etc.), and how the tables relate to each other. Designing the schema is one of the most important parts of system design.

#### Non-Relational Databases (NoSQL Databases)

Sometimes a spreadsheet-style table isn't the best fit. Non-relational databases store data in different formats:

- **Key-Value stores:** Like a dictionary. You give it a key ("user:123"), and it gives you back the value (all of that user's data). Very fast, very simple, but you can only look up by key. Example: Redis, DynamoDB.
- **Document stores:** Store data as documents — think of each record as a JSON file that can have any structure. Good when your data doesn't fit neatly into table columns. Example: MongoDB.
- **Column-family stores:** Good for storing massive amounts of data that you write often but query in simple patterns. Example: Cassandra.

**When to use NoSQL vs. SQL:**
- Use SQL (relational) when your data has clear relationships and you need to ask complex questions across multiple tables.
- Use NoSQL when you need extreme speed for simple lookups, or when your data doesn't fit neatly into tables, or when you're writing huge amounts of data.

### Cache: A Fast Shortcut

A **cache** is a small, extremely fast storage layer that saves recent or frequently-accessed data so you don't have to look it up from the (slower) database every time.

**Analogy:** Your database is a filing cabinet in the basement. Your cache is a sticky note on your desk. If someone asks you for a customer's phone number, you check the sticky note first (fast). If it's not there, you run to the basement, get the number from the filing cabinet, write it on a sticky note for next time, then give it back (slower, but now cached).

**Redis:** The most popular caching system. It stores data in memory (RAM) instead of on a hard drive, which makes it incredibly fast — responses in under 1 millisecond. Almost every large-scale web system uses Redis.

**Cache hit:** When the data you need is already in the cache. Fast.
**Cache miss:** When the data isn't in the cache, so you have to go to the database. Slow (comparatively).
**Cache hit rate:** The percentage of requests served from the cache. If 80% of requests are cache hits, you're only hitting the database for 20% of traffic — massively reducing database load.

**TTL (Time to Live):** How long data stays in the cache before it expires and gets deleted. If you set a TTL of 60 seconds, the cache entry disappears after 1 minute. Next request goes to the database, which re-populates the cache with fresh data. This prevents the cache from serving stale (outdated) data forever.

### Message Queue: A To-Do List for Your Servers

A **message queue** is a system where one part of your application puts a task on a list, and another part picks it up and handles it later.

**Analogy:** In a restaurant, the waiter doesn't cook the food. The waiter writes the order on a ticket and puts it on a rail (the queue). The cook grabs tickets from the rail and prepares them. The waiter and the cook work independently — the waiter doesn't wait for the food to be cooked before taking the next table's order.

**Why this matters:**
- **Speed:** The user doesn't have to wait for slow operations to finish. When you post a tweet, the system saves the tweet and immediately tells you "done!" The slow work — sending notifications to all your followers, updating the search index, etc. — happens in the background via the queue.
- **Reliability:** If the notification system is temporarily down, the messages sit in the queue and wait. When the notification system comes back online, it processes the backlog. Nothing is lost.
- **Decoupling:** The tweet-posting system doesn't need to know about the notification system. It just puts a message on the queue that says "a tweet was created." Any system that cares about new tweets can read from the queue.

**Kafka:** A very popular message queue (technically a "distributed streaming platform") used by most large tech companies. When you see "Kafka" in system design, think: a very reliable to-do list that can handle millions of messages per second.

**Producer:** The system that puts messages onto the queue. (The waiter writing the ticket.)
**Consumer:** The system that reads messages from the queue and processes them. (The cook grabbing tickets.)

### CDN: Copies of Your Stuff Around the World

**CDN (Content Delivery Network):** A network of servers spread across the globe that stores copies of your static files — images, videos, CSS, JavaScript — so users can download them from a server that's physically close to them instead of one that's across the world.

**Why it matters:** If your servers are in New York and a user in Tokyo loads your page, every image has to travel across the Pacific Ocean. That's slow. With a CDN, the images are cached on a server in Tokyo. The user downloads them locally — much faster.

**What goes on a CDN:** Static content that doesn't change per user — images, videos, stylesheets, fonts, the JavaScript code of your app. NOT dynamic content like "what are my latest tweets" (that's different for every user and changes constantly).

### Object Storage: Where Big Files Live

**Object storage** is a system designed to store large files — images, videos, documents, backups. Unlike a database (which stores structured data in tables), object storage just holds blobs of binary data (files) and gives each one a unique URL.

**S3 (Simple Storage Service):** Amazon's object storage product. When engineers say "put it in S3," they mean "upload the file to object storage." It's cheap, virtually unlimited in size, and extremely reliable. Most systems store media files (photos, videos) in S3, not in their database.

---

## Key Concepts You'll Need to Talk About

### Throughput: How Much Work Can Your System Handle?

**Throughput** is how many operations your system can process per unit of time. Think of it as the "bandwidth" of your system.

**QPS (Queries Per Second):** The most common throughput metric. If your database handles 10,000 QPS, it can process 10,000 requests every second. Exceed that, and requests start failing or slowing down.

**Why this matters in interviews:** You'll calculate the expected QPS from the user count, then check whether your design can handle it. If you estimate 600,000 reads per second but a single database can only handle 10,000, you know you need caching or multiple databases.

### Latency: How Long Does Each Request Take?

**Latency** is the time between "user makes a request" and "user gets the response." Lower is better.

- Reading from a cache (memory): less than 1 millisecond
- Reading from a database (disk): 1-10 milliseconds
- Sending data across the country: 30-50 milliseconds
- Sending data across the ocean: 100-200 milliseconds

**Why this matters:** If loading the Twitter feed takes 3 seconds, people will close the app. Most systems aim for under 200 milliseconds for the total response time the user experiences.

### Availability: Is Your System Always Up?

**Availability** is what percentage of time your system is working. It's measured in "nines":

- 99% availability ("two nines") = down up to 3.65 days per year
- 99.9% ("three nines") = down up to 8.76 hours per year
- 99.99% ("four nines") = down up to 52.6 minutes per year

Most systems aim for 99.9% or higher. To achieve this, you build **redundancy** — backup systems that take over when the primary goes down. If you have 3 servers and one crashes, the other 2 keep running. Users don't notice.

### Consistency: Does Everyone See the Same Data?

When you post a tweet, does every user see it immediately? Or does it take a few seconds for the tweet to appear in everyone's feed?

**Strong consistency:** Every user sees the update immediately. When you post a tweet, the next person who loads their feed will definitely see it. This is essential for things like banking — if you transfer $100, both accounts must update at exactly the same time.

**Eventual consistency:** The update will *eventually* reach everyone, but it might take a few seconds. When you post a tweet, some followers might see it immediately, while others don't see it for 2-3 seconds. This is fine for social media, and it's much easier to build.

**Why you can't always have both availability and strong consistency:** Imagine your system stores data on two servers — East Coast and West Coast. When a user writes data to the East Coast server, you have two options:
1. **Wait** until the West Coast server also has the update before confirming to the user. (Strong consistency — but if the connection between servers fails, users can't write anything. Lower availability.)
2. **Confirm immediately** and update the West Coast server in the background. (Eventual consistency — but both servers can keep accepting requests even if the connection between them fails. Higher availability.)

This tension is called the **CAP theorem:** in a distributed system (one that spans multiple computers), when a network failure happens, you can either have **C**onsistency or **A**vailability, but not both. (**P** stands for **P**artition tolerance, which means "the system keeps working even when the network between servers breaks." You always need this, so the real choice is between C and A.)

**What to say in the interview:** "For the feed service, I'd choose eventual consistency — a new tweet can take a couple seconds to propagate, and that's fine. For the payment service, I'd choose strong consistency — a double-charge is unacceptable."

### Durability: Will My Data Survive a Crash?

**Durability** means that once the system confirms your data was saved, it won't be lost — even if servers crash, power goes out, or disks fail. Databases achieve this by writing data to disk and keeping copies on multiple machines (**replication**).

---

## Scaling: What Happens When Your System Gets Popular?

The most important mental model: **what works at 100 users often doesn't work at 1 million users.** Scaling is about identifying which part will break first and fixing it before it does.

### The Three Scaling Strategies for Databases

#### Strategy 1: Add Read-Only Copies (Read Replicas)

Most systems are read-heavy — users read way more than they write. The most common first step is to create copies of your database that can only be read from, not written to.

How it works:
- One **primary** database handles all writes (creating tweets, updating profiles).
- Several **replica** databases are exact copies of the primary. They handle all reads (loading the feed, viewing profiles).
- The primary automatically sends its updates to the replicas (this is called **replication**).

**The trade-off:** There's a tiny delay (usually under 1 second) between when data is written to the primary and when the replicas have it. So a user might post a tweet and not see it in their own feed for a fraction of a second. For most applications, this is totally fine.

**When to use:** Almost always the first thing you try when the database is the bottleneck.

#### Strategy 2: Split the Data Across Multiple Databases (Sharding)

If even read replicas aren't enough, you can split your data across multiple separate databases. Each database holds a portion of the data.

**Example:** If you have 10 million users, you could split them into 10 databases of 1 million users each. Users whose IDs start with 1 go to Database A, IDs starting with 2 go to Database B, etc.

**Shard key:** The field you use to decide which database a record goes to. In the example above, `user_id` is the shard key. A good shard key distributes data evenly — you don't want 90% of users on one shard and 10% across the other nine.

**The trade-off:** Once your data is split across databases, it's very hard to run queries that span multiple shards. "Give me user 123's tweets" is easy — you know exactly which shard to ask. "What are the top 10 most-liked tweets across ALL users" requires asking every shard and combining the results, which is slow and complex.

**When to use:** Only when read replicas aren't enough. Sharding adds significant complexity.

#### Strategy 3: Use a Database Designed for Scale (NoSQL)

Some databases, like Cassandra or DynamoDB, are specifically designed to spread data across many machines automatically. They handle the sharding for you, but the trade-off is that you can't ask complex questions the way you can with SQL (no JOINs across tables).

**When to use:** When your access pattern is simple (look up by key, write a lot) and you need massive throughput.

---

## How a System Design Interview Works — Step by Step

Now that you know the building blocks, here's how to use them in a 45-minute interview.

### Phase 1: Ask Questions and Narrow the Problem (Minutes 0-5)

The interviewer will give you a vague prompt: "Design Twitter," "Design a URL shortener," "Design a chat application." Your job is NOT to start building immediately. Your first job is to ask questions.

**Why this is the most important step:** If you design the wrong thing, you've wasted 40 minutes. The interviewer actually wants you to ask questions — it shows you think before you build.

**The exact dialogue — "Design Twitter" example:**

> **You:** "Before I start designing, let me clarify what we're building. Should I focus on the full Twitter experience, or a specific part of it?"
> 
> **Interviewer:** "Let's focus on posting tweets, following users, and the home feed."
> 
> **You:** "Got it. A few more questions:
> - **How many users are we designing for?** Thousands? Millions? Hundreds of millions?"
> - **Is it okay for a new tweet to take a few seconds to appear in everyone's feed, or does it need to be instant?**  *(This determines how complex your design needs to be.)*
> - **Do most users read tweets or write tweets?** My guess is they read a lot more than they write."
>
> **Interviewer:** "500 million daily users. A few seconds delay is fine. Yes, heavily read-dominant."

Now you know what you're building: a system for 500 million users that needs to be fast at reading, can tolerate a small delay in propagation, and focuses on three features.

**Always restate the requirements before designing:**

> **You:** "Let me summarize before I start:
> - Users can post tweets (text, 280 characters max)
> - Users can follow other users
> - Users see a timeline of tweets from people they follow, sorted by newest first
> - 500 million daily users
> - Read-heavy — optimize for fast feed loading
> - A few seconds of delay is acceptable
> - The feed should always load (high availability), and tweets should never be lost once posted (durability)
>
> Does that match what you're looking for?"

**Why this works:** You've turned a two-word prompt ("Design Twitter") into a precise specification. The interviewer knows you understand the problem, and you've given yourself guardrails — if you get lost later, you can point back to these requirements.

### Phase 2: Estimate the Scale (Minutes 5-7)

Before you design the architecture, figure out how much traffic you're dealing with. This determines whether a simple design works or whether you need something more complex.

**The math is simple — don't overthink it:**

> **You:** "Let me estimate the traffic.
> 
> 500 million daily users. If each user reads about 100 tweets per day, that's 500 million × 100 = 50 billion reads per day. Divided by about 100,000 seconds in a day, that's roughly **500,000 reads per second.**
> 
> For writes — most users consume content, few create it. Say each user posts half a tweet per day on average. That's 500 million × 0.5 = 250 million writes per day, or about **3,000 writes per second.**
> 
> So the read-to-write ratio is about 200:1. This tells me:
> 1. I should invest heavily in caching — if I can serve most reads from a fast shortcut (cache) instead of the database, I massively reduce the database load.
> 2. Write speed is not the bottleneck, so I can afford to do extra work per write."

**Key numbers to know** (rough orders of magnitude):
- A single web server can handle about 1,000-10,000 requests per second
- A single database can handle about 10,000 reads per second (or about 1,000-10,000 writes per second)
- A cache (like Redis) can handle 100,000+ requests per second
- 1 day ≈ 100,000 seconds (actually 86,400, but round up for easy math)
- A small text entry (like a tweet) ≈ 300 bytes
- A compressed photo ≈ 200 KB (kilobytes)
- A minute of compressed video ≈ 10 MB (megabytes)

**The point of estimation:** You're not trying to be exact. You're trying to figure out the order of magnitude, which drives design decisions. 3,000 writes per second means a single database is fine for writes. 500,000 reads per second means you definitely need caching.

### Phase 3: Draw the High-Level Architecture (Minutes 7-12)

Now sketch the overall system. Almost every web system follows a similar pattern:

```
USERS (phones/browsers)
    ↓ send requests to
LOAD BALANCER (the traffic cop - distributes requests across servers)
    ↓ routes to
SERVERS (stateless - any server can handle any request)
    ↓ read from / write to
    ├── CACHE (fast shortcut for frequently-read data, like Redis)
    ├── DATABASE (permanent storage for all data, like Postgres)
    ├── MESSAGE QUEUE (to-do list for background work, like Kafka)
    │        ↓ feeds tasks to
    │     WORKERS (servers that handle background tasks)
    └── OBJECT STORAGE (for large files like images/videos, like S3)
             ↓ served to users via
         CDN (copies of files on servers around the world)
```

**The walk-through script:**

> **You:** "Here's the high-level architecture. I'll walk through each component.
>
> Users — on their phones or browsers — send requests through a **load balancer**, which distributes traffic across multiple **stateless servers**. Stateless means any server can handle any request, so we can add more servers to handle more traffic without changing any code.
>
> There are two main flows:
>
> **Reading the feed:** When a user opens the app, their server checks the **cache** first. If the feed data is already cached, we return it immediately — very fast. If it's not cached, we query the **database**, save the result in the cache for next time, and return it.
>
> **Posting a tweet:** When a user writes a tweet, the server saves it to the **database**. Then instead of doing all the follow-up work immediately — like notifying followers, updating the search index — the server puts a message on the **message queue** that says 'a new tweet was created.' Separate **worker** servers pick up these messages and handle the follow-up work in the background. The user doesn't wait for any of that.
>
> For images and videos, users upload directly to **object storage** (like S3), and those files are served through a **CDN** so they load fast from servers near the user's location. Media never touches our main servers.
>
> Before I go deeper — which component would you like me to focus on? The feed generation, the data model, or the data flow when a tweet is posted?"

**Why end with a question for the interviewer:** This is collaborative. You let them steer you toward what they care about testing. It also ensures you spend your remaining 25 minutes on the right topic.

### Phase 4: Deep Dive Into 2-3 Components (Minutes 12-35)

This is where the interview is won or lost. The interviewer will pick 1-2 areas and push you to design them in much more detail.

#### Deep Dive Example: The Home Feed

The hardest problem in "Design Twitter" is how to build the home feed. When a user opens their app, they need to see the latest tweets from everyone they follow. There are two approaches:

**Approach 1: Build the feed at read time (called "fan-out on read" or "pull model")**

When User A opens their feed:
1. Look up who A follows → get back 500 user IDs
2. For each of those 500 users, go to the database and get their latest tweets
3. Combine all the tweets, sort by time, return the top 20

*Pro:* Simple. No extra storage needed. Always fresh.
*Con:* Slow. If A follows 500 people, that's 500 database queries every time A opens the app. Multiply by 500,000 feed requests per second, and the database melts.

**Approach 2: Pre-build the feed at write time (called "fan-out on write" or "push model")**

When User B posts a tweet:
1. Look up who follows B → get back 10,000 follower IDs
2. For each follower, add the tweet to their pre-built feed in the cache

Now when User A opens their feed, the server just reads A's pre-built feed from cache — one read, sub-millisecond.

*Pro:* Lightning fast reads.
*Con:* The "celebrity problem." If a user has 50 million followers, posting one tweet means writing to 50 million cache entries. That's expensive.

**The solution (and what Twitter actually does): Hybrid**

- Normal users (under ~10,000 followers): Fan out on write. Their tweets are pushed into followers' cached feeds at write time.
- Celebrity users (over ~10,000 followers): Do NOT fan out on write. Instead, when User A opens their feed, the system reads A's pre-built cache (which has all the "normal" follow tweets) and then does a few targeted database queries for just the celebrities A follows. Merge and sort.

> **You:** "This gives us near-instant reads for 99% of the feed from cache, plus a small number of real-time queries for celebrity tweets. Since only about 0.1% of users have over 10,000 followers, those extra queries per feed load are typically 2-5, not 500."

**Why this impresses interviewers:** You showed both options, named the real problem with each, and then proposed a solution that takes the best of both approaches. This is trade-off reasoning — exactly what they're testing.

#### Deep Dive Example: The Data Model (Schema)

When the interviewer asks about the data model, sketch out the tables:

```
USERS TABLE:
- user_id (unique identifier, the primary key)
- username (unique text, like "@clay")
- display_name (text, like "Clay Skaggs")
- bio (text, up to 280 characters)
- created_at (timestamp of when the account was created)

TWEETS TABLE:
- tweet_id (unique identifier, the primary key)
- author_id (points to a user in the users table)
- text (the tweet content, up to 280 characters)
- reply_to (points to another tweet if this is a reply, otherwise empty)
- created_at (timestamp)

FOLLOWS TABLE:
- follower_id (the user who is doing the following)
- followee_id (the user who is being followed)
- created_at (timestamp)
- Primary key: the combination of follower_id + followee_id
  (this prevents duplicate follow entries)
```

> **You:** "Two design decisions to highlight:
>
> First, I'd add an **index** on the tweets table on the `author_id` and `created_at` columns together. The most common query — 'get this user's latest tweets' — needs to find all tweets by a specific author and sort them by time. Without this index, the database would scan every tweet in the entire table. With it, the database jumps directly to the right author's tweets and reads them in order. For a table with billions of rows, this is the difference between a 10-millisecond response and a 10-second response.
>
> Second, I'd use **time-sortable IDs** (sometimes called 'Snowflake IDs') instead of simple counting IDs (1, 2, 3...). Time-sortable IDs encode the creation time in the ID itself, so sorting tweets by ID automatically sorts them by time. This means I don't need a separate 'sort by timestamp' step in my queries."

#### Deep Dive Example: The API

When they ask about the API, define the specific endpoints:

> **You:** "Two main endpoints:
>
> **To post a tweet:**
> Send a `POST` request to `/api/v1/tweets` with the tweet text in the request body.
> The server responds with the created tweet, including its ID and timestamp.
>
> **To load the feed:**
> Send a `GET` request to `/api/v1/feed?cursor=<last_tweet_id>&limit=20`
> The server responds with the next 20 tweets in the feed.
>
> I'm using **cursor-based pagination** instead of page numbers. Here's why: imagine you're scrolling through your feed and you've loaded 20 tweets. With page numbers, you'd request 'page 2.' But if someone posted a new tweet while you were reading, all the tweets shift down — you'd see duplicates or miss tweets. With cursor-based pagination, you say 'give me 20 tweets older than the last one I saw' — that's always stable regardless of new inserts."

### Phase 5: Scaling and Trade-Offs (Minutes 35-45)

Wrap up by discussing what happens if traffic increases dramatically:

> **You:** "Let me walk through how each layer scales.
>
> **The servers** scale easily. They're stateless, so I just add more servers behind the load balancer. 10x traffic = add 10x servers. No code changes needed.
>
> **The cache** (Redis) can handle 100,000+ operations per second per machine. At 10x traffic, I'd add more cache machines and use a strategy called **consistent hashing** to distribute data across them.

*(Consistent hashing — simply: a method for distributing data across multiple cache machines so that when you add or remove a machine, you only have to move a small fraction of the data, not rearrange everything.)*

> **The database** is the hardest layer to scale.
> - **First step:** Add read replicas. The primary database handles all writes. 3-5 replica copies handle reads. At 10x, our write volume is about 30,000 per second, which one Postgres primary can handle. Reads get distributed across replicas.
> - **If that's not enough:** Shard by user_id — split users across multiple separate databases. Since most queries are 'get this user's tweets,' each query hits exactly one shard.
> - **Trade-off of sharding:** Queries that span all users — like 'what's trending globally' — become much harder. Those would need a separate analytics pipeline.
>
> **The message queue** (Kafka) scales by adding more partitions. It's designed for this — it can handle millions of messages per second."

---

## Design Patterns — Reusable Solutions to Common Problems

### Pattern 1: Separate Your Read and Write Systems (CQRS)

**CQRS (Command Query Responsibility Segregation)** — a fancy name for a simple idea: use one system optimized for writing data and a different system optimized for reading data.

**Why:** Reading and writing have very different requirements. Writes need to be reliable and structured (so use a relational database). Reads need to be fast (so use a cache or a specially-formatted read-optimized copy of the data). Instead of forcing one system to do both jobs, let each system do what it's best at.

**How to explain it:** "Writes go to a normalized Postgres database — optimized for data integrity. Reads come from a pre-built view in Redis — optimized for speed. Events from the write database keep the read model in sync."

### Pattern 2: Decouple with Events (Pub/Sub)

**Publisher-Subscriber (Pub/Sub):** Instead of System A directly calling System B, System A publishes an event ("a tweet was created") to the message queue. System B subscribes to that event and processes it when it's ready.

**Why this is better than direct calls:**
- If System B goes down, the events wait in the queue until it's back — nothing is lost
- If you add a System C later that also cares about new tweets, it just subscribes to the same queue — no changes to System A
- System A stays fast because it doesn't wait for B to finish

### Pattern 3: Distribute Data Evenly (Consistent Hashing)

When you spread data across multiple servers, you need a way to decide which server holds which data. The naive approach — `server = hash(key) % number_of_servers` — breaks badly when you add or remove a server, because *everything* gets remapped.

**Consistent hashing** is a smarter approach: servers are arranged in a virtual circle. Each piece of data hashes to a position on the circle, and the nearest server clockwise handles that data. When you add a new server, only a small fraction of data moves. This is how systems like Cassandra and DynamoDB distribute data.

---

## How to Communicate During the Interview

### Always Tell Them Where You Are

Don't just silently design. Narrate your process:

- **At the start:** "I'm going to begin with requirements, then estimate the scale, draw a high-level architecture, and deep-dive into 2-3 components."
- **At transitions:** "I've covered the data model. Now I want to talk about how the feed is generated."
- **Every 5 minutes:** "Before I go deeper on caching, is there a different area you'd prefer I focus on?"

### Always Show Alternatives Before Choosing

Don't say "I'd use Cassandra." Say:

"I'm deciding between a relational database like Postgres and a NoSQL database like Cassandra. This workload is write-heavy with simple lookup patterns — 'get data by key.' Cassandra handles high write volume and scales horizontally by adding machines. The trade-off is that complex queries using JOINs aren't supported. Since I don't need complex queries for this use case, I'd go with Cassandra."

This narrate-the-trade-off pattern — "I chose X over Y because Z" — is the single most important habit in system design interviews.

### Always Explain "Why," Not Just "What"

| ❌ What most candidates do | ✅ What strong candidates do |
|---|---|
| "I'd add a cache" | "I'd add a cache because we're handling 500,000 reads per second and the database can't handle that alone. With an 80% cache hit rate, the database load drops to 100,000 — well within its capacity." |
| "I'd use a message queue" | "I'd use a message queue because when a tweet is posted, the user shouldn't have to wait for all 10,000 followers to be notified. The queue lets us do that work in the background." |
| "I'd shard the database" | "I'd shard by user_id because our main access pattern is per-user queries. This means each query hits exactly one shard. I'd only shard after read replicas aren't enough, because sharding adds significant operational complexity." |

---

## Self-Assessment — Grade Yourself After Practice

| Dimension | Question | Target |
|---|---|---|
| **Requirements** | Did I ask questions before designing? | Yes |
| **Estimation** | Did I translate the math into design decisions? | Yes |
| **Architecture** | Did I draw a clear diagram and walk through each component? | Yes |
| **Data Model** | Did I sketch tables, define primary keys, and explain indexes? | Yes |
| **API Design** | Did I define specific endpoints and explain pagination? | Yes |
| **Deep Dive** | Did I go detailed on 2+ components? | Yes |
| **Trade-offs** | Did I say "I chose X over Y because Z" at least 3 times? | Yes |
| **Scaling** | Did I walk through each layer and discuss growth? | Yes |
| **Communication** | Did I narrate transitions, check in, and use diagrams? | Yes |

---

## Chapter Summary

### The Method (45 Minutes)
1. **Requirements** (5 min) — Ask 5-8 questions. Restate the scope. Get agreement.
2. **Estimation** (2 min) — Calculate reads per second, writes per second, storage. Translate numbers into design decisions.
3. **High-Level Design** (5 min) — Draw the architecture. Walk through each component. Ask which area to focus on.
4. **Deep Dive** (23 min) — Data model, APIs, feed generation for 2-3 critical components. Show alternatives before choosing.
5. **Scaling & Trade-offs** (10 min) — Walk through each layer. Read replicas first, then sharding, then NoSQL if needed.

### The Communication Protocol
- Narrate every transition ("Now I'll move to the data model...")
- Show alternatives before choosing ("I'm deciding between X and Y...")
- Every decision gets a trade-off explanation ("I chose X over Y because Z")
- Check in every 5 minutes ("Does this direction make sense?")
- Always draw as you talk — diagrams make everything clearer

---

*Next Chapter: Chapter 16 — The Building Blocks: Deep Dives into Databases, Caching, Queuing, and More →*
