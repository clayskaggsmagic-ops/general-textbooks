# Chapter 13: Data Modeling II — Graph, NoSQL, and Ontology-Based Design

---

## 13.1 Why This Matters for a Deployment Strategist

Chapter 12 gave you the *classical* toolkit — ER diagrams, normalization, star schemas. That toolkit works well when data is tabular, well-structured, and relationships are predictable. But real-world data isn't always like that:

1. **Relationships ARE the data.** In fraud detection, intelligence analysis, and supply chain optimization, the connections between entities matter more than the entities themselves. Relational JOINs break down at 5+ hops; graph databases don't
2. **Not all data fits rows and columns.** Sensor telemetry, JSON API responses, user activity logs, and medical records are better modeled as documents, key-value pairs, or wide columns than relational tables
3. **Palantir Gotham IS a graph platform.** Link analysis — tracing relationships through networks of people, transactions, locations, and events — is Gotham's core capability. Understanding graph modeling *is* understanding Gotham
4. **The Ontology synthesizes everything.** Palantir's Ontology is neither purely relational nor purely graph — it takes the best concepts from both worlds and wraps them in business-friendly language. This chapter shows how all the paradigms connect
5. **Clients use polyglot architectures.** No enterprise runs a single database. You'll encounter PostgreSQL alongside MongoDB alongside Redis alongside Snowflake. Foundry is the integration layer — and understanding *why* each tech was chosen is part of the DS role

> **Key insight:** Chapter 12 asked "How do we structure data?" This chapter asks "What if tables aren't the right structure?" — and shows you when to reach for a different tool entirely.

---

## 13.2 Graph Data Modeling

### 13.2.1 The Property Graph Model

A property graph is a data model that treats **relationships as first-class citizens**, not afterthoughts computed via JOINs:

```
  THE THREE BUILDING BLOCKS:

  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │  NODES (Vertices)                                               │
  │  • Represent entities: Person, Company, Account, Location       │
  │  • Have one or more LABELS: (:Person), (:Company)               │
  │  • Have PROPERTIES: {name: "Alice", age: 32}                    │
  │                                                                 │
  │  EDGES (Relationships)                                          │
  │  • Represent connections between nodes                          │
  │  • Have a TYPE (label): WORKS_AT, TRANSFERRED_TO, KNOWS         │
  │  • Have a DIRECTION: (Alice)-[:WORKS_AT]->(Acme Corp)           │
  │  • Have PROPERTIES: {since: "2020-01-15", role: "Engineer"}     │
  │                                                                 │
  │  PROPERTIES                                                     │
  │  • Key-value pairs attached to BOTH nodes and edges             │
  │  • Flexible: different nodes of the same label can have         │
  │    different properties (schema-flexible)                       │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

**Visual example:**

```
  (Alice:Person)──[:WORKS_AT {since: 2020}]──>(Acme:Company)
       │                                          │
       │                                          │
  [:KNOWS]                               [:LOCATED_IN]
       │                                          │
       ↓                                          ↓
  (Bob:Person)──[:TRANSFERRED $5K]──>(Account:BankAcct)
       │
  [:LIVES_IN]
       ↓
  (NYC:City)

  Key insight: the RELATIONSHIP has properties too.
  "Alice KNOWS Bob" → but WHEN? HOW? Properties on the edge tell you.
  "Bob TRANSFERRED $5K to Account" → relationship IS the transaction.
```

### 13.2.2 When Relational Models Fail and Graphs Excel

```
  THE "JOIN BOMB" PROBLEM:

  Scenario: "Find all people within 4 degrees of connection from Alice"

  ─── SQL (RELATIONAL): ─────────────────────────────────────────

  SELECT DISTINCT p4.name
  FROM connections c1
  JOIN connections c2 ON c1.person_b = c2.person_a
  JOIN connections c3 ON c2.person_b = c3.person_a
  JOIN connections c4 ON c3.person_b = c4.person_a
  JOIN persons p4 ON c4.person_b = p4.person_id
  WHERE c1.person_a = 'Alice';

  • 4 self-JOINs → 5 tables in the query
  • Performance DEGRADES exponentially with depth
  • At 6+ hops, this query may not return in reasonable time
  • Adding "5th degree" means rewriting the query entirely

  ─── CYPHER (GRAPH): ────────────────────────────────────────────

  MATCH (a:Person {name: "Alice"})-[:KNOWS*1..4]-(p:Person)
  RETURN DISTINCT p.name;

  • Variable-depth traversal in ONE line
  • Performance depends on LOCAL neighborhood, not table size
  • Changing to 6 hops: just change *1..4 to *1..6
  • Reads like English: "Find persons within 1 to 4 KNOWS hops"
```

**When to use graphs vs. relational:**

| Scenario | Best Model | Why |
|---|---|---|
| Fixed, well-known relationships (orders, invoices) | **Relational** | Predictable JOINs, ACID transactions |
| Aggregations, reporting, BI dashboards | **Relational / Star schema** | GROUP BY, SUM, COUNT optimized |
| Deep, variable-length relationship traversal | **Graph** | O(local) not O(global) traversal |
| Many-to-many with relationship attributes | **Graph** | Edges with properties are natural |
| Network/topology analysis | **Graph** | Path finding, centrality built-in |
| Fraud detection / money laundering | **Graph** | Follow the money through N hops |
| Recommendation engines | **Graph** | "People who bought X also bought Y" |
| Social network analysis | **Graph** | Influence, communities, connections |
| Simple CRUD with clear schema | **Relational** | Proven, mature, well-tooled |

### 13.2.3 Graph Database Landscape

| Database | Model | Query Language | Key Strength | Used By |
|---|---|---|---|---|
| **Neo4j** | Property graph | Cypher | Most mature, richest ecosystem, ACID compliant | eBay, NASA, Walmart |
| **Amazon Neptune** | Property graph + RDF | Gremlin + SPARQL | AWS-native, managed, multi-model | AWS customers |
| **TigerGraph** | Property graph | GSQL | Extreme scale, deep-link analytics, real-time | Financial institutions |
| **ArangoDB** | Multi-model (graph + doc + KV) | AQL | Polyglot in single engine | Flexible use cases |

**Neo4j Cypher — The "SQL of Graphs":**

```
  ─── CREATE ──────────────────────────────────────
  CREATE (alice:Person {name: "Alice", age: 32})
  CREATE (acme:Company {name: "Acme Corp"})
  CREATE (alice)-[:WORKS_AT {since: 2020}]->(acme)

  ─── READ (MATCH) ────────────────────────────────
  MATCH (p:Person)-[:WORKS_AT]->(c:Company)
  WHERE c.name = "Acme Corp"
  RETURN p.name, p.age

  ─── TRAVERSAL ───────────────────────────────────
  MATCH path = (a:Person {name: "Alice"})-[:KNOWS*1..3]-(friend)
  RETURN friend.name, length(path) AS distance

  ─── SHORTEST PATH ───────────────────────────────
  MATCH path = shortestPath(
    (a:Person {name: "Alice"})-[*]-(b:Person {name: "Dave"})
  )
  RETURN path

  Cypher's ASCII-art syntax makes graph patterns visual:
  (node)-[:RELATIONSHIP]->(node)  ← you can SEE the graph
```

### 13.2.4 Link Analysis — Graphs in Intelligence and Security

Link analysis is the practice of examining relationships between entities to uncover hidden patterns. This is **core Palantir Gotham functionality**:

```
  LINK ANALYSIS TECHNIQUES:

  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │  1. RELATIONSHIP MAPPING                                        │
  │     "Who is connected to whom, and how?"                        │
  │     • Map entities (people, accounts, addresses) and their      │
  │       connections across data sources                           │
  │     • Visualize networks to spot non-obvious links              │
  │     • Example: Person A shares a phone number with Person B,    │
  │       who shares an address with Person C                       │
  │                                                                 │
  │  2. PATH FINDING                                                │
  │     "What's the connection between X and Y?"                    │
  │     • Shortest path: fewest hops between two entities           │
  │     • All paths: every possible route (may reveal alternatives) │
  │     • Weighted paths: cheapest/fastest/most likely connection    │
  │     • Example: How does money flow from shell company to target?│
  │                                                                 │
  │  3. COMMUNITY DETECTION                                         │
  │     "Which entities cluster together?"                          │
  │     • Identify groups of densely connected nodes                │
  │     • Useful in: fraud rings, organized crime cells,            │
  │       social circles, market segments                           │
  │     • Algorithms: Louvain, Label Propagation, Modularity        │
  │     • Example: Cluster of accounts that only transact with      │
  │       each other → possible fraud ring                          │
  │                                                                 │
  │  4. CENTRALITY ANALYSIS                                         │
  │     "Who is the most important/influential node?"               │
  │     • Degree centrality: most connections                       │
  │     • Betweenness centrality: bridges between communities       │
  │     • PageRank: importance based on incoming connections         │
  │     • Example: The person who connects two criminal networks    │
  │       (high betweenness) is a key intelligence target           │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

**Real-world applications:**

| Domain | Link Analysis Use | Palantir Product |
|---|---|---|
| **Intelligence** | Map terrorist networks, trace communication chains | Gotham |
| **Fraud detection** | Trace money flows, identify synthetic identities | Gotham / Foundry |
| **Supply chain** | Map supplier dependencies, identify single points of failure | Foundry |
| **Healthcare** | Track disease transmission, identify drug interaction networks | Foundry |
| **Cybersecurity** | Map attack vectors, trace lateral movement | Gotham |

---

## 13.3 NoSQL Data Modeling

### 13.3.1 The NoSQL Philosophy

NoSQL ("Not Only SQL") databases emerged because relational databases struggle with:
- **Massive scale** — billions of rows across distributed clusters
- **Schema flexibility** — data structures that evolve rapidly
- **Specific access patterns** — optimized for particular read/write workloads
- **High availability** — staying online even when nodes fail

```
  ┌──────────────────────────────────────────────────────────────┐
  │  RELATIONAL (SQL)                   NoSQL                    │
  │  ────────────────                   ─────                    │
  │  Schema-on-WRITE                    Schema-on-READ           │
  │  (define schema BEFORE data)        (interpret schema AT     │
  │                                      query time)             │
  │  Normalized (3NF)                   Often denormalized       │
  │  ACID transactions                  BASE (eventual)          │
  │  Scale vertically                   Scale horizontally       │
  │  General-purpose queries            Optimized for specific   │
  │                                      access patterns         │
  │  JOINs across tables                Data often duplicated    │
  │                                      to avoid JOINs          │
  └──────────────────────────────────────────────────────────────┘
```

### 13.3.2 Key-Value Stores

The simplest data model — a **dictionary/hash map** at database scale:

```
  ┌────────────────────────┬──────────────────────────────────────┐
  │         KEY            │              VALUE                   │
  ├────────────────────────┼──────────────────────────────────────┤
  │ user:1001:session      │ {token: "abc123", exp: 1710000000}  │
  │ cache:product:5522     │ {name: "Widget", price: 9.99, ...}  │
  │ feature:dark_mode      │ true                                 │
  │ rate_limit:ip:10.0.0.1 │ 47                                   │
  └────────────────────────┴──────────────────────────────────────┘

  Operations:
  GET(key) → value           O(1)
  SET(key, value)            O(1)
  DELETE(key)                O(1)

  That's it. No queries, no JOINs, no filtering by value.
  You MUST know the key to retrieve the value.
```

**Redis** — The dominant in-memory key-value store:

| Feature | Detail |
|---|---|
| **Speed** | Sub-millisecond reads/writes (all in-memory) |
| **Data structures** | Strings, Lists, Sets, Sorted Sets, Hashes, Streams |
| **Persistence** | Optional (RDB snapshots, AOF logs) |
| **Use cases** | Caching, session storage, rate limiting, leaderboards, pub/sub messaging |
| **Limitation** | Dataset must fit in RAM (or use Redis Cluster) |

**DynamoDB** — AWS fully managed, serverless key-value + document:

| Feature | Detail |
|---|---|
| **Speed** | Single-digit millisecond latency at any scale |
| **Model** | Partition key (hash), optional sort key (range) |
| **Scaling** | Automatic, no capacity planning needed |
| **Use cases** | Gaming backends, mobile apps, IoT, serverless architectures |
| **Pricing** | Pay per read/write unit — cost-efficient at scale |

### 13.3.3 Document Stores

Store data as **self-contained documents** (usually JSON/BSON):

```
  RELATIONAL:                          DOCUMENT (MongoDB):
  ──────────                           ─────────────────
  orders table                         {
  ┌──────┬─────┬────────┐               "_id": "O1001",
  │ o_id │ c_id│ date   │               "customer": {
  │ O1001│ C01 │ Jan 15 │                 "name": "Alice",
  └──────┴─────┴────────┘                 "email": "alice@example.com"
  order_items table                      },
  ┌──────┬──────┬─────┐                  "date": "2024-01-15",
  │ o_id │ p_id │ qty │                  "items": [
  │ O1001│ P005 │  2  │                    {"product": "Widget", "qty": 2,
  │ O1001│ P012 │  1  │                     "price": 9.99},
  └──────┴──────┴─────┘                    {"product": "Gadget", "qty": 1,
  customers table                           "price": 19.99}
  ┌──────┬───────┬──────┐                ],
  │ c_id │ name  │email │                "total": 39.97
  │ C01  │ Alice │ a@.. │              }
  └──────┴───────┴──────┘

  3 tables, 2 JOINs                    1 document, 0 JOINs
  Strong consistency                   Schema-flexible
  Normalized                           Denormalized (embedded)
```

**MongoDB** — The leading document database:

| Feature | Detail |
|---|---|
| **Format** | BSON (Binary JSON) — supports more types than JSON |
| **Schema** | Flexible — documents in the same collection can have different fields |
| **Querying** | Rich query language with aggregation pipeline |
| **Scaling** | Horizontal via sharding |
| **Trade-off** | No JOINs (by design) — denormalize or use `$lookup` (expensive) |

**Embedded documents vs. references:**

```
  EMBEDDED (denormalized):              REFERENCED (normalized):
  ─────────────────────                 ───────────────────────
  {                                     // orders collection
    "_id": "O1001",                     {
    "customer": {                         "_id": "O1001",
      "name": "Alice",  ← EMBEDDED       "customer_id": "C01",  ← REFERENCE
      "email": "a@..."                   "items": [...]
    },                                  }
    "items": [...]
  }                                     // customers collection
                                        {
  ✅ Single read gets everything           "_id": "C01",
  ✅ No JOINs needed                       "name": "Alice",
  ❌ Customer data duplicated              "email": "a@..."
  ❌ Updating customer = update           }
     every order                        
                                        ✅ No duplication
  Best when: data accessed together     ✅ Easy to update customer
  and rarely changes                    ❌ Requires two reads (or $lookup)
                                        Best when: data changes often or
                                        is shared across documents
```

### 13.3.4 Column-Family Stores

Data organized by **columns** rather than rows, grouped into **column families**:

```
  ROW-ORIENTED (Relational):     COLUMN-FAMILY (Cassandra):
  ──────────────────────         ──────────────────────────

  Row 1: [id, name, temp, ts]   Column Family: sensor_data
  Row 2: [id, name, temp, ts]   
  Row 3: [id, name, temp, ts]   Row Key: sensor_001
                                 ┌───────────────────────────────────┐
  Reads ALL columns even if      │ ts:2024-01-15 │ ts:2024-01-16 │...│
  you only need one.             │ temp: 72.1    │ temp: 73.4    │   │
                                 │ humidity: 45  │ humidity: 42  │   │
                                 └───────────────────────────────────┘

  Each row can have DIFFERENT columns (sparse storage).
  Columns are sorted within a row — range scans are fast.
```

**Apache Cassandra:**

| Feature | Detail |
|---|---|
| **Architecture** | Peer-to-peer (no master node), ring-based |
| **Write speed** | Extremely fast — append-only commit log |
| **Availability** | Tunable consistency (ONE, QUORUM, ALL) |
| **Data model** | Partition key + clustering columns = query-first design |
| **Best for** | Time-series, IoT telemetry, event logging, high-write workloads |
| **Trade-off** | Must design tables around specific queries; no ad-hoc JOINs |

### 13.3.5 The CAP Theorem

The CAP theorem (Brewer, 2000) states that a distributed data store can guarantee at most **two of three** properties:

```
                         CONSISTENCY
                            /\
                           /  \
                          /    \
                         / CA   \
                        / (RDBMS)\
                       /  single  \
                      /   node     \
                     /──────────────\
                    /                \
                   /   PICK TWO      \
                  /    (during a      \
                 /      partition)     \
                /                      \
               / CP           AP        \
              / (MongoDB)  (Cassandra)   \
             /  (Redis)     (DynamoDB)    \
            /───────────────────────────── \
           PARTITION TOLERANCE  ────  AVAILABILITY
```

**The three properties defined:**

| Property | Definition | In Plain English |
|---|---|---|
| **Consistency (C)** | All nodes see the same data at the same time | Every read returns the latest write |
| **Availability (A)** | Every non-failing node returns a response | The system always responds, even with stale data |
| **Partition Tolerance (P)** | System works despite network failures between nodes | Nodes can't talk to each other but still operate |

**The critical insight:** Network partitions are **inevitable** in distributed systems. Cables fail, data centers go offline, packets get dropped. Since P is non-negotiable, the real choice is **C vs. A during a partition:**

| Category | Prioritizes | Sacrifices | Behavior During Partition | Examples |
|---|---|---|---|---|
| **CP** | Consistency + Partition Tolerance | Availability | Rejects/delays requests to ensure data correctness | MongoDB, Redis, HBase |
| **AP** | Availability + Partition Tolerance | Strong Consistency | Returns potentially stale data but stays online | Cassandra, DynamoDB, CouchDB |
| **CA** | Consistency + Availability | Partition Tolerance | Cannot exist as a *distributed* system (single-node RDBMS) | PostgreSQL (single node) |

### 13.3.6 Eventual Consistency

When an AP system returns stale data during a partition, it promises that the data will **eventually** converge to a consistent state once the partition heals:

```
  STRONG CONSISTENCY:                  EVENTUAL CONSISTENCY:
  ──────────────────                   ─────────────────────

  Write to Node A                     Write to Node A
  ↓ (synchronous replication)         ↓ (asynchronous replication)
  Read from Node B                    Read from Node B
  → Gets the LATEST value             → Might get STALE value
                                       ...wait...
                                      Read from Node B again
                                      → NOW gets the latest value

  Trade-off:
  Strong = slower writes (must sync)   Eventual = faster writes
  Strong = higher latency              Eventual = lower latency
  Strong = may need to reject writes   Eventual = always accepts writes
```

**When eventual consistency is acceptable:**
- Social media feeds (seeing a post 2 seconds late is fine)
- Shopping cart (brief inconsistency is tolerable)
- Analytics dashboards (near-real-time is good enough)
- DNS (propagation takes hours; the internet still works)

**When eventual consistency is NOT acceptable:**
- Bank account balances (cannot show wrong balance)
- Inventory counts for the last item in stock
- Seat reservations (cannot double-book)
- Medical records (life-critical accuracy)

---

## 13.4 The NoSQL Decision Framework

When choosing a data model, start with the **access pattern**, not the data:

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                  "WHAT QUESTION ARE YOU ASKING?"                  │
  ├───────────────────────────────────────────────────────────────────┤
  │                                                                   │
  │  "Get this thing by its ID"                                       │
  │  → KEY-VALUE (Redis, DynamoDB)                                    │
  │                                                                   │
  │  "Get this thing and all its nested details"                      │
  │  → DOCUMENT (MongoDB, Firestore)                                  │
  │                                                                   │
  │  "Show me time-ordered events for this entity"                    │
  │  → COLUMN-FAMILY (Cassandra) or TIME-SERIES (InfluxDB)            │
  │                                                                   │
  │  "How are these things connected? Who knows whom?"                │
  │  → GRAPH (Neo4j, Neptune, TigerGraph)                             │
  │                                                                   │
  │  "Aggregate across millions of rows with complex filters"         │
  │  → RELATIONAL / COLUMNAR (PostgreSQL, Snowflake, BigQuery)        │
  │                                                                   │
  │  "Run ad-hoc analytical queries across structured data"           │
  │  → RELATIONAL (PostgreSQL) or WAREHOUSE (Snowflake)               │
  │                                                                   │
  └───────────────────────────────────────────────────────────────────┘
```

**Master comparison table:**

| | Relational | Key-Value | Document | Column-Family | Graph |
|---|---|---|---|---|---|
| **Data model** | Tables, rows, columns | Key → opaque value | JSON/BSON documents | Row key → column families | Nodes + edges |
| **Schema** | Rigid (schema-on-write) | None | Flexible | Flexible per row | Flexible |
| **Query** | SQL (ad-hoc) | GET/SET by key | Rich query + aggregation | Query by partition/cluster key | Traversal (Cypher/Gremlin) |
| **JOINs** | ✅ Native | ❌ None | ⚠️ `$lookup` (expensive) | ❌ None | ✅ Traversal IS the join |
| **Transactions** | ACID | Limited | Varies | Limited | ACID (Neo4j) |
| **Scale** | Vertical (mostly) | Horizontal | Horizontal | Horizontal | Vertical (mostly) |
| **Consistency** | Strong | Varies | Varies | Tunable | Strong (ACID) |
| **Sweet spot** | OLTP, structured data | Caching, sessions | Content, catalogs | Time-series, IoT | Networks, fraud |

---

## 13.5 The Connection to Palantir's Ontology

### 13.5.1 The Rosetta Stone

The Palantir Ontology borrows from every modeling paradigm and stitches them together with business language:

```
  ┌───────────────────────────────────────────────────────────────────┐
  │          THE ONTOLOGY ROSETTA STONE                               │
  ├─────────────────┬─────────────────┬───────────────────────────────┤
  │  ER / Relational │  Graph Model    │  Palantir Ontology            │
  ├─────────────────┼─────────────────┼───────────────────────────────┤
  │  Entity          │  Node           │  Object Type                  │
  │  Attribute       │  Property       │  Property                     │
  │  Relationship    │  Edge           │  Link                         │
  │  Table           │  —              │  Backing dataset              │
  │  Row             │  Node instance  │  Object (instance)            │
  │  Primary Key     │  Node ID        │  Primary Key (on Object Type) │
  │  Foreign Key     │  Edge           │  Link (configured, not FK)    │
  │  Junction table  │  —              │  Link (M:N backed by dataset) │
  │  Stored procedure│  Mutation       │  Action                       │
  │  View            │  Subgraph       │  Object Set + Filters         │
  │  Schema          │  Graph schema   │  Ontology definition          │
  └─────────────────┴─────────────────┴───────────────────────────────┘
```

### 13.5.2 Why Ontology Modeling > ER Diagrams (for Stakeholders)

```
  ER DIAGRAM says:                    ONTOLOGY says:
  ────────────────                    ──────────────

  "Table CUSTOMER with columns        "There are Customers.
   customer_id INT PK,                 Each Customer has a name, email,
   first_name VARCHAR(50),             and a sales region.
   last_name VARCHAR(50),              Customers place Orders.
   email VARCHAR(255) UNIQUE,          Orders contain Products."
   region_id INT FK..."

  Technical, precise,                  Business-friendly, intuitive,
  requires database literacy           anyone can understand
```

The Ontology acts as a **translation layer** between:
- **Data engineers** who think in tables, columns, and pipelines
- **Business users** who think in Customers, Orders, and Revenue
- **Analysts** who think in relationships, patterns, and trends

This is why the DS role exists — you're the person who can speak both languages.

### 13.5.3 Entity Resolution — The Killer Feature

Entity resolution is the process of determining that **multiple records refer to the same real-world entity**, even when the data doesn't match exactly:

```
  RECORD IN SYSTEM A:                 RECORD IN SYSTEM B:
  ─────────────────                   ─────────────────
  Name: "Robert Smith"                Name: "Bob Smith"
  DOB: 1985-03-15                     DOB: 03/15/1985
  Address: "123 Main St"              Address: "123 Main Street"
  Phone: (555) 123-4567               Phone: 5551234567
  Email: rsmith@gmail.com             Email: robert.smith@gmail.com

  Are these the same person? A human would say "probably yes."
  Entity resolution makes this determination systematically.
```

**How Palantir does entity resolution:**

```
  ┌───────────────────────────────────────────────────────────────┐
  │                 ENTITY RESOLUTION PIPELINE                    │
  ├───────────────────────────────────────────────────────────────┤
  │                                                               │
  │  1. BLOCKING                                                  │
  │     Group records into candidate pairs to reduce comparisons  │
  │     (comparing every record to every other is O(n²))          │
  │     Example: only compare records with same first letter      │
  │     of last name + same birth year                            │
  │                                                               │
  │  2. FUZZY MATCHING                                            │
  │     Compare candidate pairs using similarity metrics:         │
  │     • Levenshtein distance: "Smith" ↔ "Smyth" = 1 edit       │
  │     • Jaro-Winkler: weights early character matches higher    │
  │     • Phonetic matching: "Smith" ≈ "Smyth" (Soundex/Metaphone)│
  │     • Numeric normalization: "(555) 123-4567" = "5551234567"  │
  │                                                               │
  │  3. FEATURE GENERATION                                        │
  │     Create match/non-match features from fuzzy scores         │
  │     Example: name_similarity=0.85, dob_match=TRUE,            │
  │              address_similarity=0.92, phone_match=TRUE        │
  │                                                               │
  │  4. ML MODEL / RULE ENGINE                                    │
  │     Classify pairs as MATCH / NON-MATCH / UNCERTAIN           │
  │     Can use supervised ML trained on validated matches         │
  │                                                               │
  │  5. HUMAN VALIDATION                                          │
  │     Uncertain cases sent to human reviewers                   │
  │     Feedback loops improve the model over time                │
  │                                                               │
  │  6. IDENTITY GRAPH                                            │
  │     Resolved entities form a graph of "same as" links         │
  │     One canonical entity → many source records                │
  │     This IS the golden record / master data                   │
  │                                                               │
  └───────────────────────────────────────────────────────────────┘
```

**Why this matters for a DS:**

| Challenge | What Entity Resolution Solves |
|---|---|
| Client has 5 systems, each with its own "customer" table | Unified customer view across all systems |
| Same person appears as "Robert Smith" and "Bob R. Smith" | Fuzzy matching links both records to one entity |
| Fraudster creates synthetic identities with slight variations | Identity graph reveals connected fabricated records |
| Hospital needs to match patients across EMR, billing, and lab systems | Single patient record despite different IDs per system |
| Intelligence analysts need to trace a person across data sources | Cross-system identity linkage |

### 13.5.4 The Three Layers of the Ontology (Revisited)

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │  LAYER 1: SEMANTIC (the "what")                                 │
  │  ─────────────────────────────                                  │
  │  The conceptual model. What entities exist? How do they relate? │
  │  • Object Types: Customer, Order, Product, Shipment             │
  │  • Properties: name, status, price, location                    │
  │  • Links: Customer → [places] → Order → [contains] → Product   │
  │  • Actions: "Create Order", "Approve Claim", "Assign Technician"│
  │  This is the ER diagram translated into business language.      │
  │                                                                 │
  │  LAYER 2: KINETIC (the "how")                                   │
  │  ────────────────────────────                                   │
  │  The data plumbing. How do source systems connect?              │
  │  • Pipelines: Extract → Transform → Load from source systems    │
  │  • Backing datasets: which dataset powers each Object Type      │
  │  • Sync schedules: real-time, hourly, daily                     │
  │  • Entity resolution: matching across sources                   │
  │  This is the ETL/ELT layer — where data engineering happens.    │
  │                                                                 │
  │  LAYER 3: DYNAMIC (the "who interacts")                         │
  │  ──────────────────────────────────────                         │
  │  The interaction layer. How do users and AI agents work?        │
  │  • Workshop apps: dashboards, forms, workflows                  │
  │  • AIP agents: AI powered by Ontology context                   │
  │  • Actions triggered by users or automated rules                │
  │  • Security: who can see/edit which Objects                     │
  │  This is where business value is delivered.                      │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

---

## 13.6 Polyglot Persistence

### 13.6.1 What It Means

Polyglot persistence is the practice of using **multiple database technologies** in a single architecture, each chosen for its optimal fit:

```
  MONOLITHIC APPROACH:                 POLYGLOT APPROACH:
  ───────────────────                  ──────────────────

  ┌──────────────┐                    ┌───────────────┐
  │   App Layer   │                    │   App Layer    │
  └──────┬───────┘                    └──┬──┬──┬──┬───┘
         │                               │  │  │  │
  ┌──────┴───────┐                       │  │  │  │
  │  PostgreSQL  │                       ↓  ↓  ↓  ↓
  │  (everything)│              ┌────┐ ┌────┐ ┌────┐ ┌─────┐
  └──────────────┘              │Pg  │ │Redis│ │Mongo│ │Neo4j│
                                │OLTP│ │Cache│ │Docs │ │Graph│
  Simple, proven,               └────┘ └────┘ └────┘ └─────┘
  but one-size-fits-all
                                Each service uses what's best
                                for its specific access pattern
```

### 13.6.2 Real-World Example: E-Commerce Platform

```
  ┌─────────────────────────────────────────────────────────────────┐
  │  SERVICE         │  DATABASE     │  WHY THIS ONE               │
  ├─────────────────────────────────────────────────────────────────┤
  │  User accounts   │  PostgreSQL   │  ACID for financial data,   │
  │  & payments      │               │  strong consistency needed  │
  │─────────────────────────────────────────────────────────────────│
  │  Product catalog  │  MongoDB      │  Flexible schema: each      │
  │                   │               │  product has different attrs│
  │─────────────────────────────────────────────────────────────────│
  │  Session &        │  Redis        │  Sub-ms reads, auto-expire │
  │  caching          │               │  for ephemeral data        │
  │─────────────────────────────────────────────────────────────────│
  │  Recommendations  │  Neo4j        │  "Customers who bought X   │
  │                   │               │   also bought Y" = graph   │
  │─────────────────────────────────────────────────────────────────│
  │  Order activity   │  Cassandra    │  High-write event log with │
  │  & event log      │               │  time-bucketed partitions  │
  │─────────────────────────────────────────────────────────────────│
  │  Analytics &      │  Snowflake    │  Star schema for BI,       │
  │  reporting        │               │  complex aggregations      │
  └─────────────────────────────────────────────────────────────────┘
```

### 13.6.3 The Integration Challenge

Polyglot persistence creates a **data integration problem** — data is scattered across multiple systems that don't natively talk to each other:

```
  CHALLENGES:

  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │  1. CROSS-DATABASE JOINS                                        │
  │     "Show me customer details (Postgres) with their browsing    │
  │      history (Cassandra) and recommendations (Neo4j)"           │
  │     → No database can JOIN across these natively                │
  │     → Requires application-level stitching                      │
  │                                                                 │
  │  2. DATA CONSISTENCY                                            │
  │     "Update a customer's email in Postgres, but it's also       │
  │      cached in Redis and embedded in MongoDB documents"          │
  │     → Must propagate changes across all systems                 │
  │     → Eventual consistency between stores                       │
  │                                                                 │
  │  3. OPERATIONAL COMPLEXITY                                      │
  │     "Our team needs to manage 5 different database technologies" │
  │     → Different monitoring, backup, scaling, security setups    │
  │     → Specialized expertise for each                            │
  │                                                                 │
  │  4. DISTRIBUTED TRANSACTIONS                                    │
  │     "Deduct payment (Postgres) AND update inventory (Cassandra) │
  │      AND send confirmation (Queue) — atomically"                │
  │     → No native cross-database transactions                    │
  │     → Must use Saga pattern or eventual consistency             │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

### 13.6.4 Foundry as the Integration Layer

This is **exactly** the problem Palantir Foundry solves:

```
  WITHOUT FOUNDRY:                     WITH FOUNDRY:
  ──────────────                       ──────────────

  PostgreSQL ──┐                       PostgreSQL ──┐
  MongoDB ─────┤ ??? ──→ Dashboard     MongoDB ─────┤
  Cassandra ───┤ (manual integration)  Cassandra ───┼──→ FOUNDRY ──→ Ontology
  SAP ─────────┤                       SAP ─────────┤     • Unified model
  Snowflake ───┘                       Snowflake ───┘     • Cross-source JOINs
                                                          • Entity resolution
  Who stitches this together?                             • Single source of truth
  Custom ETL? Data engineers?                             • Apps & AI on top
  A year of integration work?
                                       Foundry IS the polyglot
                                       integration platform.
```

The DS role is to understand:
1. **What** each source system stores and **why** that technology was chosen
2. **How** to map those systems into a unified Ontology
3. **Where** entity resolution and data quality rules apply
4. **What** the client can DO with a unified view they've never had before

---

## 13.7 Glossary

| Term | Definition |
|---|---|
| **AP system** | Database prioritizing Availability + Partition Tolerance; may return stale data (Cassandra, DynamoDB) |
| **Blocking** | Entity resolution step that groups records into candidate pairs to reduce comparison volume |
| **BSON** | Binary JSON — MongoDB's storage format, supports additional types beyond JSON |
| **CAP theorem** | In a distributed system, you can guarantee at most two of: Consistency, Availability, Partition Tolerance |
| **Centrality** | Graph metric measuring a node's importance (degree, betweenness, PageRank) |
| **Column-family** | NoSQL model storing data in row-key → column-family groups (Cassandra, HBase) |
| **Community detection** | Graph algorithm identifying clusters of densely connected nodes |
| **CP system** | Database prioritizing Consistency + Partition Tolerance; may reject requests (MongoDB, Redis) |
| **Cypher** | Neo4j's declarative graph query language using ASCII-art syntax for pattern matching |
| **Document store** | NoSQL database storing self-contained JSON/BSON documents (MongoDB, Firestore) |
| **Edge** | A relationship between two nodes in a graph; has type, direction, and properties |
| **Entity resolution** | Determining that multiple records from different sources refer to the same real-world entity |
| **Eventual consistency** | Guarantee that all replicas will converge to the same state, given enough time without new writes |
| **Fuzzy matching** | Comparing records using similarity metrics (Levenshtein, Jaro-Winkler) rather than exact equality |
| **Identity graph** | Network of "same as" links connecting resolved entity records to a canonical master record |
| **Key-value store** | Simplest NoSQL model — GET/SET by unique key with O(1) lookups (Redis, DynamoDB) |
| **Link analysis** | Examining relationships between entities to uncover hidden patterns, used in intelligence and fraud |
| **Node** | An entity in a graph database; has labels and properties |
| **NoSQL** | "Not Only SQL" — database category optimized for specific access patterns beyond relational |
| **Ontology** | Palantir's unified data model mapping real-world entities, properties, and relationships |
| **Path finding** | Graph algorithm finding routes between nodes (shortest path, weighted path, all paths) |
| **Polyglot persistence** | Using multiple database technologies in one architecture, each for its optimal use case |
| **Property graph** | Graph model where both nodes and edges can carry key-value properties |
| **Schema-on-read** | Data is stored without enforcing schema; structure is interpreted at query time |
| **Schema-on-write** | Schema must be defined before data is inserted; structure is enforced at write time |

---

## 13.8 Chapter Summary — The "Cheat Sheet"

```
┌─────────────────────────────────────────────────────────────────┐
│  CHAPTER 13 SUMMARY CARD                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GRAPH DATA MODELING:                                            │
│  • Property graph: nodes + edges + properties on both            │
│  • Excels at: deep traversals, variable-depth relationships      │
│  • Key databases: Neo4j (Cypher), Neptune (Gremlin), TigerGraph  │
│  • Link analysis: path finding, community detection, centrality  │
│  • Core to Palantir Gotham's intelligence capabilities           │
│                                                                  │
│  NoSQL PARADIGMS:                                                │
│  • Key-Value (Redis, DynamoDB): O(1) lookups, caching, sessions  │
│  • Document (MongoDB): flexible JSON docs, schema-on-read        │
│  • Column-Family (Cassandra): high-write, time-series, IoT       │
│  • Choose based on ACCESS PATTERN, not data structure             │
│                                                                  │
│  CAP THEOREM:                                                    │
│  • Consistency + Availability + Partition Tolerance: pick 2       │
│  • P is mandatory → real choice is C vs. A during partition      │
│  • CP (MongoDB/Redis): correct data OR reject request            │
│  • AP (Cassandra/DynamoDB): always respond, maybe stale          │
│  • Eventual consistency: replicas converge over time              │
│                                                                  │
│  THE ONTOLOGY CONNECTION:                                        │
│  • Object Types ≈ Entities ≈ Nodes                               │
│  • Properties ≈ Attributes ≈ Node/Edge props                     │
│  • Links ≈ Relationships ≈ Edges                                 │
│  • Actions ≈ Business logic / mutations                          │
│  • Entity resolution: fuzzy matching + ML → identity graph       │
│  • Why Ontology > ER: business language, non-technical audience   │
│                                                                  │
│  POLYGLOT PERSISTENCE:                                           │
│  • Real architectures use MULTIPLE databases                     │
│  • Challenge: integration, consistency, cross-DB joins           │
│  • Foundry IS the integration layer for polyglot architectures   │
│  • DS role: understand WHY each tech was chosen + map to Ontology│
│                                                                  │
│  THE DECISION QUESTION:                                          │
│  • "Get by ID" → Key-Value                                       │
│  • "Get with nested details" → Document                          │
│  • "Time-ordered events" → Column-family                         │
│  • "How are things connected?" → Graph                           │
│  • "Aggregate with complex filters" → Relational / Warehouse     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 13.9 What's Next

With both traditional and non-relational modeling complete, **Chapter 14: Data Integration and Transformation** dives into the plumbing — ETL vs. ELT pipelines, data cleaning, transformation patterns, and how tools like Apache Spark, dbt, and Palantir's own pipeline transforms move data from raw source to Ontology-ready form.

---

*"A Deployment Strategist walks into a client and finds PostgreSQL for transactions, MongoDB for product data, Redis for caching, Cassandra for event logs, Snowflake for analytics, and a Neo4j proof-of-concept for fraud detection. Six technologies, six mental models, zero integration. The DS's job: map this polyglot chaos into a unified Ontology where 'Customer' means the same thing everywhere and every analyst can ask any question. That's data modeling in practice — and that's what this chapter prepares you for."*
