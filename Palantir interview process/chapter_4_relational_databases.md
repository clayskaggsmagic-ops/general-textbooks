# Chapter 4: The Data Ecosystem I — Relational Databases and OLTP Systems

> *"Data is the new oil — but like oil, it must be refined. As a Deployment Strategist, the very first thing you will encounter at every client is their database. Understanding what you're connecting to — and why it was built that way — is not optional. It's the foundation of everything Foundry does."*

---

## 4.1 Why This Chapter Matters for Deployment Strategists

Before you can build an Ontology (Chapter 3), you must understand where the data lives. Every Palantir engagement begins with a **data integration phase** — connecting Foundry to the client's existing systems. Those systems are overwhelmingly **relational databases** running **OLTP workloads**.

This chapter gives you the fluency to:

- Walk into a client's data center and immediately understand their stack
- Ask the right questions about schemas, indexes, replication, and data freshness
- Anticipate data quality issues before they contaminate your pipelines
- Speak the same language as the client's DBAs and data engineers

---

## 4.2 Relational Model Fundamentals

The **relational model**, introduced by Edgar F. Codd in 1970, organizes data into **relations** (tables) composed of **tuples** (rows) and **attributes** (columns). Despite being over five decades old, it remains the dominant paradigm for structured data storage.

### Core Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| **Table (Relation)** | A structured collection of rows sharing a common schema | `orders`, `customers`, `inventory` |
| **Row (Tuple)** | A single record in a table | One customer, one order |
| **Column (Attribute)** | A named field with a data type | `customer_name VARCHAR(100)` |
| **Primary Key (PK)** | A column (or combination) that uniquely identifies each row | `order_id`, `customer_id` |
| **Foreign Key (FK)** | A column that references the primary key of another table, enforcing referential integrity | `orders.customer_id → customers.id` |
| **Schema** | The blueprint defining tables, columns, types, and constraints | The DDL that creates the database |

### SQL — The Universal Language

**Structured Query Language (SQL)** is the standard interface for relational databases. Despite vendor-specific dialects (T-SQL, PL/SQL, PL/pgSQL), the core syntax is remarkably consistent:

```sql
-- The four fundamental operations (CRUD)
SELECT * FROM orders WHERE status = 'pending';       -- Read
INSERT INTO orders (customer_id, total) VALUES (42, 99.99);  -- Create
UPDATE orders SET status = 'shipped' WHERE id = 1001;        -- Update
DELETE FROM orders WHERE id = 1001;                          -- Delete
```

> **Interview Tip:** You will not be expected to write production SQL in a DS interview, but you *will* be expected to understand what a `JOIN` does, what a foreign key enforces, and why normalization matters.

---

## 4.3 ACID — The Guarantees That Make Databases Trustworthy

Every relational database worth its name provides **ACID** guarantees for transactions. This is what separates a database from a spreadsheet.

| Property | Guarantee | What Happens Without It |
|----------|-----------|------------------------|
| **Atomicity** | A transaction is all-or-nothing. If any part fails, the entire transaction rolls back. | A bank transfer debits your account but crashes before crediting the recipient — money vanishes. |
| **Consistency** | A transaction moves the database from one valid state to another, respecting all constraints. | An order is created referencing a customer that doesn't exist — orphaned data spreads. |
| **Isolation** | Concurrent transactions don't interfere with each other; each sees a consistent snapshot. | Two users simultaneously purchase the last item in stock — both succeed, but only one item exists. |
| **Durability** | Once committed, a transaction survives power failures, crashes, and hardware faults. | A confirmed payment disappears after a server restart — trust is destroyed. |

### Transactions and Concurrency

A **transaction** is a logical unit of work — one or more SQL statements that execute as a group:

```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
-- If either UPDATE fails, ROLLBACK undoes both.
```

**Concurrency control** prevents transactions from corrupting each other. The two dominant strategies are:

1. **Pessimistic locking** — Acquire locks before reading/writing; other transactions wait
2. **Optimistic / MVCC** — Each transaction sees a snapshot; conflicts detected at commit time

PostgreSQL uses MVCC (§4.4). Oracle uses a combination of undo segments and locking. MySQL/InnoDB uses MVCC with row-level locking.

---

## 4.4 PostgreSQL — The Modern Default

**PostgreSQL** (often called "Postgres") is an open-source **object-relational database management system (ORDBMS)** and is widely considered the most advanced open-source database available. If a Palantir client is a modern tech company, a startup, or a government digital services team, they are very likely running Postgres.

### Why PostgreSQL Dominates

- **Open source** — No licensing fees, no vendor lock-in
- **Strict SQL compliance** — Closest to the SQL standard of any major database
- **Extensibility** — User-defined types, functions, operators, and index methods
- **Advanced data types** — JSON/JSONB, arrays, ranges, geometric, network addresses, UUIDs
- **Mature ecosystem** — PostGIS (geospatial), TimescaleDB (time-series), Citus (distributed)
- **ACID-compliant** with full MVCC implementation

### MVCC — Multi-Version Concurrency Control

PostgreSQL's concurrency model is built on **MVCC**. Instead of locking rows when reading or writing, Postgres creates a **new version** of each modified row:

```
Transaction A reads Row 42 (version 1)
Transaction B updates Row 42 → creates version 2
Transaction A still sees version 1 (its snapshot)
Transaction B commits → version 2 becomes visible to new transactions
```

**Key mechanics:**
- Each row carries `xmin` (transaction that created it) and `xmax` (transaction that deleted/updated it)
- Readers never block writers; writers never block readers
- Old row versions are cleaned up by the **VACUUM** process (autovacuum runs by default)
- This enables full ACID compliance without the performance penalty of traditional locking

> **DS Relevance:** When Foundry connectors extract data from Postgres, they benefit from MVCC because their read queries never block the client's production writes.

### Extensibility

PostgreSQL's catalog-driven architecture allows users to extend nearly every aspect of the database:

| Extension Type | Description | Example |
|---------------|-------------|---------|
| **User-defined types** | Create new data types with custom input/output functions | Composite types, enums, ranges, domains |
| **User-defined functions** | Write functions in SQL, PL/pgSQL, Python, C, or JavaScript | Custom business logic inside the database |
| **Custom operators** | Define new operators that work with custom types | Geospatial `&&` (bounding box overlap) |
| **Index access methods** | Implement entirely new index structures | `bloom` filter indexes |
| **Extensions (plugins)** | Packaged bundles of types, functions, and operators | PostGIS, pgvector, pg_trgm |

### Indexing Strategies

Choosing the right index is critical for query performance. PostgreSQL offers **six** index types:

| Index Type | Best For | How It Works | Example Use Case |
|-----------|---------|-------------|-----------------|
| **B-tree** | Equality and range queries (`=`, `<`, `>`, `BETWEEN`) | Balanced tree structure; default index type | `WHERE customer_id = 42`, `WHERE created_at > '2024-01-01'` |
| **Hash** | Simple equality only (`=`) | Hash table lookup; crash-safe since PG 10 | `WHERE session_token = 'abc123'` |
| **GIN** (Generalized Inverted Index) | Multi-value columns: arrays, JSONB, full-text search | Inverted index mapping values → row locations | `WHERE tags @> '{urgent}'`, JSONB key searches |
| **GiST** (Generalized Search Tree) | Geometric data, ranges, full-text search | Flexible tree supporting "nearest neighbor" and containment queries | PostGIS spatial queries, IP range lookups |
| **SP-GiST** (Space-Partitioned GiST) | Non-balanced, partitioned data structures | Supports quadtrees, k-d trees, radix trees | Phone number prefix matching, spatial data |
| **BRIN** (Block Range INdex) | Very large tables with naturally ordered data | Stores min/max values per block range; tiny footprint | Time-series data, append-only logs |

```sql
-- B-tree (default)
CREATE INDEX idx_orders_customer ON orders (customer_id);

-- GIN for JSONB
CREATE INDEX idx_products_attrs ON products USING GIN (attributes);

-- BRIN for time-series
CREATE INDEX idx_logs_timestamp ON server_logs USING BRIN (created_at);
```

### JSON and JSONB

PostgreSQL supports two JSON types:

| Feature | `json` | `jsonb` |
|---------|--------|---------|
| Storage | Exact text copy (preserves whitespace, key order) | Decomposed binary format |
| Speed | Slower queries (reparsed each time) | Fast queries (pre-parsed) |
| Indexing | Not indexable | GIN-indexable |
| Use case | Audit logs where exact representation matters | Everything else |

```sql
-- Store semi-structured product attributes alongside structured columns
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    attributes JSONB  -- {"color": "red", "weight_kg": 2.5, "tags": ["sale"]}
);

-- Query inside JSONB
SELECT name FROM products WHERE attributes @> '{"color": "red"}';
```

> **DS Relevance:** Many modern clients store configuration data, API responses, or sensor metadata in JSONB columns. Foundry can extract and flatten these into structured Ontology properties.

### Full-Text Search

PostgreSQL includes **built-in full-text search** without external dependencies like Elasticsearch:

- Text is tokenized, stop-words removed, and stemmed into **lexemes** (stored as `tsvector`)
- Search queries are parsed into `tsquery` format
- Results can be **ranked** by relevance using `ts_rank`
- Performance is optimized with **GIN indexes** on `tsvector` columns

```sql
-- Create a searchable column
ALTER TABLE articles ADD COLUMN search_vector tsvector;
UPDATE articles SET search_vector = to_tsvector('english', title || ' ' || body);
CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);

-- Search with ranking
SELECT title, ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'palantir & ontology') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

---

## 4.5 Oracle Database — The Enterprise Titan

**Oracle Database** is the enterprise-grade RDBMS that dominates **finance, healthcare, telecommunications, and government**. If a Palantir client is a Fortune 500 company, a hospital system, a bank, or a defense contractor, there is a very high probability that Oracle is either their primary database or a critical system of record.

### Architecture

Oracle's architecture is divided into **memory structures** and **physical files**:

```
┌─────────────────────────────────────────────────────────────┐
│                    ORACLE INSTANCE                           │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                   SGA (Shared Memory)                   │  │
│  │  ┌─────────────┐ ┌──────────────┐ ┌────────────────┐  │  │
│  │  │ Buffer Cache │ │  Shared Pool │ │ Redo Log Buffer│  │  │
│  │  │ (data blocks)│ │ (SQL cache,  │ │ (change recs.) │  │  │
│  │  │             │ │  dictionary) │ │                │  │  │
│  │  └─────────────┘ └──────────────┘ └────────────────┘  │  │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐            │  │
│  │  │Large Pool│ │Java Pool │ │Streams Pool│            │  │
│  │  └──────────┘ └──────────┘ └────────────┘            │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │  PGA #1    │ │  PGA #2    │ │  PGA #3    │   (per-     │
│  │ (session   │ │ (session   │ │ (session   │   process)  │
│  │  memory)   │ │  memory)   │ │  memory)   │              │
│  └────────────┘ └────────────┘ └────────────┘              │
└─────────────────────────────────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
     ┌────────────┐ ┌────────────┐ ┌────────────┐
     │ Data Files │ │ Redo Logs  │ │Control File│
     │ (.dbf)     │ │ (.log)     │ │ (.ctl)     │
     └────────────┘ └────────────┘ └────────────┘
```

| Component | Purpose |
|-----------|---------|
| **SGA (System Global Area)** | Shared memory accessible by all database processes; holds caches and buffers |
| **Buffer Cache** | Caches data blocks read from disk; enables fast subsequent reads |
| **Shared Pool** | Caches parsed SQL, execution plans, and data dictionary; avoids re-parsing |
| **Redo Log Buffer** | Circular buffer holding change records; flushed to disk by LGWR process |
| **PGA (Program Global Area)** | Private memory per server process — sort areas, session state |
| **Data Files** | Physical storage of table and index data |
| **Redo Logs** | Transaction journal — every change is logged for crash recovery |
| **Control Files** | Metadata repository — database name, file locations, checkpoint info |

### High Availability: RAC and Data Guard

| Feature | RAC (Real Application Clusters) | Data Guard |
|---------|-------------------------------|------------|
| **Purpose** | High availability + horizontal scaling | Disaster recovery + zero data loss |
| **Architecture** | Multiple instances on separate servers share one database | Primary database + synchronized standby databases |
| **Failover** | Automatic — if one node dies, others continue; sessions fail over transparently | Manual or automatic (Fast-Start Failover) |
| **Scaling** | Add nodes for more compute capacity | Offload reporting/backups to standby |
| **Replication** | Shared storage (SAN), not replication | Redo transport — synchronous or asynchronous |
| **Use case** | "The database must never go down" | "If the data center burns, we lose nothing" |

### Advanced Partitioning

Oracle can divide large tables into smaller, independently managed **partitions**:

| Strategy | How It Partitions | Best For |
|---------|------------------|---------|
| **Range** | By value ranges (e.g., date ranges) | Time-series data — `orders_2024_Q1`, `orders_2024_Q2` |
| **List** | By explicit value lists (e.g., regions) | Geographic data — `region IN ('US', 'EU', 'APAC')` |
| **Hash** | By hash function on a column | Even distribution when no natural range exists |
| **Composite** | Combines two strategies (e.g., range-hash) | Large tables needing multi-dimensional pruning |
| **Interval** | Auto-creates range partitions as data arrives | Continuous time-series without manual partition creation |

**Partition pruning** allows the optimizer to skip irrelevant partitions entirely, turning a full-table scan of billions of rows into a scan of millions.

### Compression

| Compression Type | Target Workload | Key Behavior |
|-----------------|----------------|--------------|
| **Basic Table Compression** | Read-mostly tables | Compresses on direct-path inserts; available in all editions |
| **Advanced Row Compression (OLTP)** | Transactional workloads | Compresses during normal DML — inserts, updates, deletes |
| **Hybrid Columnar Compression (HCC)** | Data warehousing (Exadata only) | Stores data by columns for 10–50× compression ratios |
| **Index Compression** | Indexes with repetitive leading key values | Reduces B-tree index storage substantially |

### Compliance and Regulated Industries

Oracle dominates in industries where **regulatory compliance** is non-negotiable:

| Regulation | Requirement | Oracle Feature |
|-----------|-------------|---------------|
| **HIPAA** | Protect PHI (Protected Health Information); audit all access | Transparent Data Encryption (TDE), Database Vault, Audit Vault |
| **PCI-DSS** | Secure cardholder data; mask sensitive fields; log access | Data Masking, Fine-Grained Auditing, Label Security |
| **SOX** | Accurate financial reporting; segregation of duties; tamper-proof trails | Database Vault, Audit Vault, Flashback for forensics |

### Licensing Economics

Oracle licensing is notoriously complex and expensive:

| Metric | How It Works | Gotcha |
|--------|-------------|--------|
| **Processor-based** | License per physical core × core factor | Oracle defines "processor" differently from hardware vendors |
| **Named User Plus (NUP)** | License per individual user (minimum 25 per processor for EE) | Indirect access (e.g., web app users) may count |
| **Enterprise Edition options** | RAC, Partitioning, Compression, etc. are **separately licensed add-ons** | Accidentally enabling an EE feature on Standard Edition creates compliance risk |
| **Bring Your Own License (BYOL)** | Use existing on-prem licenses in cloud | VM rules are different from physical — audit risk |

> **DS Relevance:** Oracle's licensing complexity means Palantir clients often want to *reduce* their Oracle footprint. Foundry can become the analytical layer that allows clients to stop running expensive analytics queries against their production Oracle instances.

---

## 4.6 MySQL and MariaDB — When Simpler Is Better

**MySQL** is the world's most popular open-source database, powering the web for three decades. **MariaDB** is a community-driven fork created in 2009 by MySQL's original creator (Monty Widenius) after Oracle acquired Sun Microsystems.

### InnoDB vs. MyISAM

| Feature | InnoDB | MyISAM |
|---------|--------|--------|
| **ACID compliance** | ✅ Full | ❌ None |
| **Transactions** | ✅ `BEGIN / COMMIT / ROLLBACK` | ❌ No transaction support |
| **Locking** | Row-level (fine-grained) | Table-level (coarse-grained) |
| **Foreign keys** | ✅ Enforced | ❌ Not supported |
| **Crash recovery** | ✅ Automatic via redo logs | ❌ Manual repair required |
| **Concurrency** | High (MVCC + row locks) | Low (entire table locked on writes) |
| **Full-text search** | ✅ (since MySQL 5.6) | ✅ (historically faster) |
| **Default since** | MySQL 5.5 (2010) | Pre-5.5 |
| **When to use** | Everything modern | Read-only lookup tables, legacy systems |

> **Bottom line:** InnoDB is the correct choice for virtually all modern applications. MyISAM survives only in legacy systems and very specific read-only scenarios.

### MySQL vs. PostgreSQL at a Glance

| Dimension | MySQL | PostgreSQL |
|-----------|-------|-----------|
| **Philosophy** | Speed and simplicity | Correctness and extensibility |
| **SQL compliance** | Partial (silent type coercions, loose GROUP BY) | Strict (errors on ambiguity) |
| **Data types** | Standard + JSON | Standard + JSON/JSONB + arrays + ranges + geometric + network + UUID |
| **Indexing** | B-tree, Hash, Full-text, Spatial | B-tree, Hash, GIN, GiST, SP-GiST, BRIN |
| **Replication** | Master-slave (async/semi-sync) | Streaming + logical (async/sync), multi-active |
| **Extensions** | Limited plugin architecture | Rich extension ecosystem (PostGIS, pgvector, etc.) |
| **MVCC** | InnoDB only (undo logs) | System-wide (row versioning) |
| **License** | GPL (Oracle-controlled) | PostgreSQL License (truly open) |
| **Typical stack** | LAMP (Linux/Apache/MySQL/PHP), WordPress | PERN (PostgreSQL/Express/React/Node), Django, Rails |

### MariaDB Differentiators

MariaDB diverges from MySQL in several meaningful ways:

- **Galera Cluster** — True multi-master replication with synchronous writes
- **ColumnStore** — Built-in columnar storage engine for OLAP workloads
- **Parallel query execution** and thread pooling
- **Enhanced security** — roles-based access, stronger password validation
- **No Oracle ownership** — fully community-governed

### Replication

All three systems support replication for read scaling and high availability:

| Feature | MySQL | MariaDB | PostgreSQL |
|---------|-------|---------|-----------|
| **Primary model** | Master → Slave (async) | Multi-source, parallel | Streaming (async/sync) |
| **Semi-synchronous** | ✅ (waits for 1 replica ack) | ✅ | ✅ (synchronous_commit) |
| **Logical replication** | Limited (binlog) | Enhanced (GTID) | Full (pub/sub model, PG 10+) |
| **Multi-master** | ❌ (requires Group Replication) | ✅ (Galera Cluster) | ✅ (third-party: BDR, Citus) |
| **Mechanism** | Binary log replay | Binary log + GTID | WAL (Write-Ahead Log) streaming |

---

## 4.7 Microsoft SQL Server — The Windows Ecosystem

**Microsoft SQL Server** is the database of choice for organizations deeply embedded in the Microsoft ecosystem — Windows Server, .NET, Active Directory, Azure. It's particularly prevalent in **mid-market enterprises, healthcare, and financial services**.

### T-SQL (Transact-SQL)

T-SQL is Microsoft's proprietary extension to standard SQL, adding procedural programming:

```sql
-- Stored procedure with error handling
CREATE PROCEDURE dbo.TransferFunds
    @FromAccount INT, @ToAccount INT, @Amount DECIMAL(10,2)
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;
            UPDATE Accounts SET Balance = Balance - @Amount WHERE AccountID = @FromAccount;
            UPDATE Accounts SET Balance = Balance + @Amount WHERE AccountID = @ToAccount;
        COMMIT;
    END TRY
    BEGIN CATCH
        ROLLBACK;
        THROW;  -- Re-raise the error
    END CATCH
END;
```

### The BI Stack: SSIS / SSAS / SSRS

SQL Server's integrated BI tools form a complete data-to-insight pipeline:

```
   Source Systems                Analysis              Presentation
  ┌──────────┐              ┌──────────┐            ┌──────────┐
  │ Oracle   │──┐           │  SSAS    │            │  SSRS    │
  │ CSV Files│──┤  SSIS     │ (Cubes,  │──────────▶ │ (Reports,│
  │ APIs     │──┼──────────▶│  Tabular │            │  Charts, │
  │ SAP      │──┤  Extract  │  Models) │            │  Dashbds)│
  │ Flat File│──┘  Transform│          │            │          │
  └──────────┘    Load      └──────────┘            └──────────┘
```

| Service | Full Name | Purpose |
|---------|-----------|---------|
| **SSIS** | SQL Server Integration Services | ETL — extract data from diverse sources, transform, and load into SQL Server or a warehouse |
| **SSAS** | SQL Server Analysis Services | OLAP cubes + tabular semantic models for multi-dimensional analysis and data mining |
| **SSRS** | SQL Server Reporting Services | Paginated reports, charts, dashboards delivered via web portal, email, or file shares |

### Enterprise Features

| Feature | Description |
|---------|-------------|
| **Always On Availability Groups** | Multi-replica HA with automatic failover — the SQL Server equivalent of Oracle RAC + Data Guard |
| **In-Memory OLTP** | Memory-optimized tables for high-throughput transactional workloads (natively compiled stored procedures) |
| **Columnstore Indexes** | Columnar storage within a row-store database — dramatically accelerates analytical queries |
| **Transparent Data Encryption (TDE)** | At-rest encryption for compliance (HIPAA, PCI-DSS) |
| **Row-Level Security (RLS)** | Grants/restricts row access based on user context — no application changes needed |
| **Dynamic Data Masking** | Masks sensitive columns (SSN, credit card) for non-privileged users |

### Azure SQL

Azure SQL extends SQL Server to the cloud:

| Azure SQL Flavor | Use Case |
|-----------------|---------|
| **Azure SQL Database** | Fully managed, serverless — ideal for new cloud-native applications |
| **Azure SQL Managed Instance** | Near-100% SQL Server compatibility — lift-and-shift on-prem databases to the cloud |
| **SQL Server on Azure VMs** | Full control — run any SQL Server version/edition on cloud infrastructure |

---

## 4.8 OLTP vs. OLAP — Two Worlds, Two Architectures

This is one of the most fundamental concepts in data engineering. Understanding the distinction is essential for a Deployment Strategist because **Palantir Foundry typically sits on the OLAP side**, consuming data from OLTP systems.

### Side-by-Side Comparison

| Dimension | OLTP (Online Transaction Processing) | OLAP (Online Analytical Processing) |
|-----------|--------------------------------------|--------------------------------------|
| **Purpose** | Run the business | Understand the business |
| **Workload** | Many small, fast read/write transactions | Few large, complex read queries |
| **Users** | Frontline workers, customers, applications | Analysts, data scientists, executives |
| **Data** | Current, detailed, operational | Historical, aggregated, summarized |
| **Schema** | Normalized (3NF) — minimize redundancy | Denormalized (star/snowflake) — optimize reads |
| **Response time** | Milliseconds | Seconds to minutes |
| **Volume** | Gigabytes | Terabytes to petabytes |
| **Updates** | Frequent small writes (INSERT/UPDATE/DELETE) | Periodic bulk loads (ETL) |
| **Concurrency** | Thousands of simultaneous users | Dozens of simultaneous queries |
| **Storage** | Row-oriented | Column-oriented |
| **Examples** | PostgreSQL, MySQL, Oracle, SQL Server | Snowflake, BigQuery, Redshift, ClickHouse |

### Row-Oriented vs. Columnar Storage

This is the *physical* manifestation of the OLTP/OLAP divide:

```
ROW-ORIENTED STORAGE (OLTP)
┌──────────────────────────────────────────────┐
│ Row 1: [id=1, name="Alice", age=30, city="NY"]   │
│ Row 2: [id=2, name="Bob",   age=25, city="LA"]   │
│ Row 3: [id=3, name="Carol", age=35, city="NY"]   │
└──────────────────────────────────────────────┘
→ Fast for: "Get all data for customer id=2"
→ Slow for: "Average age of all customers"

COLUMNAR STORAGE (OLAP)
┌────────────────────────────────────────┐
│ id:   [1, 2, 3, 4, 5, 6, ...]        │
│ name: ["Alice", "Bob", "Carol", ...]  │
│ age:  [30, 25, 35, 28, 42, ...]       │
│ city: ["NY", "LA", "NY", "SF", ...]   │
└────────────────────────────────────────┘
→ Fast for: "Average age of all customers" (reads only the age column)
→ Slow for: "Get all data for customer id=2" (must reconstruct from all columns)
```

**Why columnar is faster for analytics:**

1. **Reduced I/O** — Read only the columns you need, skip the rest
2. **Better compression** — Same-type values compress dramatically (dictionary encoding, run-length encoding)
3. **Vectorized execution** — Process thousands of values per CPU instruction via SIMD
4. **Partition pruning** — Skip entire data blocks based on min/max metadata

**Why row-oriented is faster for transactions:**

1. **Full-row access** — All attributes stored together for fast point lookups
2. **Efficient writes** — Insert one row = write one contiguous block
3. **Row-level locking** — Fine-grained concurrency for updates
4. **Index scans** — B-tree indexes point directly to complete rows

### The ETL Bridge

In practice, organizations do **not** choose one or the other — they run both:

```
OLTP (Source of Truth)        ETL / CDC Pipeline        OLAP (Analytics)
┌──────────────────┐    ──────────────────────▶    ┌──────────────────┐
│ PostgreSQL       │    Extract, Transform,        │ Snowflake        │
│ Oracle           │    Load (batch or streaming)  │ BigQuery         │
│ SQL Server       │                               │ Foundry          │
│ MySQL            │    ◀──────────────────────    │ Redshift         │
└──────────────────┘    (writeback, if any)        └──────────────────┘
```

> **DS Relevance:** Palantir Foundry acts as the analytical platform. Your job is to extract data from OLTP source systems and transform it into the Ontology. Understanding *why* the source system is designed the way it is (normalized, row-oriented, optimized for writes) helps you anticipate data quality issues and design better pipelines.

---

## 4.9 How Palantir Connects — Foundry's Data Integration Layer

This is where everything in this chapter converges. **Foundry's first job is to get the data in.**

### The Data Connection Application

Foundry's **Data Connection** application is the primary mechanism for ingesting data from external systems:

```
┌──────────────────────────────────────────────────────────────┐
│                    FOUNDRY DATA CONNECTION                     │
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Direct     │  │   Agent-    │  │    Cloud-Native     │  │
│  │  Connection  │  │   Based     │  │    Connectors       │  │
│  │             │  │             │  │                     │  │
│  │  Cloud DBs   │  │  On-prem    │  │  S3, GCS, Azure    │  │
│  │  SaaS APIs   │  │  Behind     │  │  Blob, Kafka,      │  │
│  │  Public APIs │  │  Firewalls  │  │  Kinesis, Pub/Sub  │  │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────────┘  │
│         │                │                   │               │
│         ▼                ▼                   ▼               │
│  ┌──────────────────────────────────────────────────────┐    │
│  │              Sync Engine                              │    │
│  │   • Batch sync (scheduled)                            │    │
│  │   • Streaming sync (real-time)                        │    │
│  │   • CDC — Change Data Capture (incremental)           │    │
│  │   • Media sync (files, images, documents)             │    │
│  └───────────────────────┬──────────────────────────────┘    │
│                          ▼                                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │         Foundry Dataset (Parquet on HDFS/S3)          │    │
│  └──────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

### Connector Types

| Connector Category | Protocol | Common Sources |
|-------------------|----------|---------------|
| **JDBC** | Java Database Connectivity | PostgreSQL, Oracle, MySQL, SQL Server, DB2, Teradata |
| **ODBC** | Open Database Connectivity | Legacy systems, Access, older enterprise tools |
| **REST API** | HTTP/HTTPS | Salesforce, ServiceNow, Workday, custom internal APIs |
| **File/Blob** | S3, GCS, Azure Blob, SFTP | CSV drops, JSON exports, Parquet files |
| **Streaming** | Kafka, Kinesis, Pub/Sub | IoT sensors, event streams, real-time feeds |
| **ERP-specific** | SAP RFC, RFC/BAPI | SAP ECC, S/4HANA |
| **NoSQL** | Native drivers | MongoDB, Elasticsearch, DynamoDB |

### Direct vs. Agent-Based Sources

| Type | When to Use | How It Works |
|------|------------|-------------|
| **Direct connection** | Source is accessible over the internet (cloud DBs, SaaS APIs) | Foundry connects outbound to the source |
| **Agent-based** | Source is behind a firewall (on-prem Oracle, air-gapped systems) | Lightweight agent installed on-prem; initiates outbound connection to Foundry |

> **Interview Tip:** A common DS interview question is: "How would you connect Foundry to a client's on-premise Oracle database that's behind a corporate firewall?" The answer is the **Foundry Agent** — a lightweight, containerized process that runs inside the client's network, establishes an outbound connection to Foundry (no inbound firewall rules needed), and syncs data on a schedule.

### Sync Strategies

| Strategy | Mechanism | Tradeoff |
|---------|-----------|---------|
| **Full snapshot** | Extract entire table on each sync | Simple but slow for large tables; works for small/medium tables |
| **Incremental (timestamp)** | Extract only rows where `updated_at > last_sync_time` | Fast, but requires a reliable timestamp column |
| **CDC (Change Data Capture)** | Capture individual INSERT/UPDATE/DELETE events from database logs | Real-time, minimal source load; requires CDC support on source DB |
| **Streaming** | Continuous real-time ingest from Kafka/Kinesis/Pub/Sub | Lowest latency; requires streaming infrastructure |

### Why Understanding the Source Database Matters

As a Deployment Strategist, your job is *not* to be a DBA — but you must understand enough to ask the right questions:

| Question | Why It Matters |
|---------|----------------|
| "What database engine is this?" | Determines which connector and driver version to use |
| "Is this table normalized or denormalized?" | Affects how you model Objects and Links in the Ontology |
| "How often is this data updated?" | Determines sync frequency — hourly, daily, or real-time |
| "Is there a reliable `updated_at` column?" | Determines whether incremental sync is possible |
| "Are there soft deletes or hard deletes?" | Soft deletes require filtering `WHERE deleted_at IS NULL`; hard deletes require snapshot comparison |
| "What's the primary key?" | Maps directly to the Object Type primary key in the Ontology |
| "Are there indexes on the columns we'll filter?" | Affects extraction query performance — slow queries can impact the client's production system |
| "Is CDC enabled?" | If yes, near-real-time sync is possible; if no, you'll need batch extraction |

### Data Quality Pipeline

Once data is ingested, Foundry's **Data Health** framework ensures quality:

```
Source DB → Connector → Raw Dataset → Health Checks → Curated Dataset → Ontology Object
                                          │
                                          ▼
                                    ┌──────────┐
                                    │ Checks:  │
                                    │ • Timeliness (is it fresh?)     │
                                    │ • Completeness (any nulls?)     │
                                    │ • Consistency (valid values?)   │
                                    │ • Uniqueness (duplicate PKs?)   │
                                    │ • Schema drift (columns added?) │
                                    └──────────┘
                                          │
                                    If checks fail:
                                    → Block downstream propagation
                                    → Alert data steward
                                    → Log for audit trail
```

> **Key Principle:** Foundry treats data quality as a **first-class concern**, not an afterthought. Health checks can be configured to **halt pipeline execution** if data quality drops below a threshold, preventing bad data from contaminating the Ontology.

---

## 4.10 Database Selection Decision Framework

When evaluating a client's database landscape, use this framework:

| Factor | PostgreSQL | Oracle | MySQL | SQL Server |
|--------|-----------|--------|-------|------------|
| **Open source** | ✅ Free | ❌ Expensive | ✅ Free (GPL) | ❌ Licensed |
| **Enterprise compliance** | Via extensions | ✅ Built-in (TDE, Vault, Audit) | Limited | ✅ Built-in (TDE, RLS, Always Encrypted) |
| **Extensibility** | ✅ Best-in-class | Moderate | Limited | Moderate |
| **JSON/Document support** | JSONB (excellent) | JSON (good) | JSON (basic) | JSON (good) |
| **Geospatial** | PostGIS (excellent) | Oracle Spatial (good) | GIS (basic) | Spatial (good) |
| **Cloud-native** | AlloyDB, Aurora, Neon, Supabase | Oracle Cloud, Autonomous DB | Aurora, PlanetScale | Azure SQL |
| **Common industries** | Tech, startups, gov digital | Finance, healthcare, telecom, legacy enterprise | Web, e-commerce, media | Healthcare, finance, mid-market enterprise |
| **Palantir connector** | JDBC ✅ | JDBC ✅ | JDBC ✅ | JDBC ✅ |
| **CDC support** | Logical replication (PG 10+) | Oracle GoldenGate, LogMiner | Binlog | SQL Server CDC |

---

## 4.11 Glossary

| Term | Definition |
|------|-----------|
| **ACID** | Atomicity, Consistency, Isolation, Durability — the four guarantees of a database transaction |
| **B-tree** | Balanced tree index structure; default index type in most databases |
| **BRIN** | Block Range Index — stores min/max per block; ideal for time-series |
| **CDC** | Change Data Capture — captures individual row changes from database logs |
| **Columnar storage** | Stores data by columns rather than rows; optimized for analytics |
| **Control file** | Oracle metadata file containing database structure information |
| **Data Guard** | Oracle's disaster recovery solution using synchronized standby databases |
| **ETL** | Extract, Transform, Load — the process of moving data from source to analytical systems |
| **FK** | Foreign Key — a column referencing another table's primary key |
| **Full-text search** | Linguistic search capability built into PostgreSQL (tsvector/tsquery) |
| **GIN** | Generalized Inverted Index — optimal for arrays, JSONB, and full-text search |
| **GiST** | Generalized Search Tree — flexible index for geometric and range data |
| **InnoDB** | MySQL/MariaDB's default ACID-compliant storage engine |
| **JDBC** | Java Database Connectivity — standard API for connecting to databases from Java |
| **JSONB** | PostgreSQL's binary JSON type; indexable and queryable |
| **MVCC** | Multi-Version Concurrency Control — readers don't block writers |
| **MyISAM** | Legacy MySQL storage engine — fast reads, no transactions |
| **NUP** | Named User Plus — Oracle's per-user licensing metric |
| **ODBC** | Open Database Connectivity — older, language-agnostic database API |
| **OLAP** | Online Analytical Processing — complex queries on large historical datasets |
| **OLTP** | Online Transaction Processing — fast, small, concurrent transactions |
| **PGA** | Program Global Area — per-process private memory in Oracle |
| **PK** | Primary Key — uniquely identifies each row in a table |
| **RAC** | Real Application Clusters — Oracle's multi-instance HA solution |
| **Redo log** | Transaction journal file; used for crash recovery |
| **Row-oriented storage** | Stores entire rows together; optimized for transactions |
| **SGA** | System Global Area — shared memory in Oracle |
| **T-SQL** | Transact-SQL — Microsoft's proprietary SQL extension |
| **VACUUM** | PostgreSQL process that reclaims space from dead row versions |

---

## 4.12 Chapter Summary

```
┌───────────────────────────────────────────────────────────────────┐
│                    CHAPTER 4 — KEY TAKEAWAYS                       │
│                                                                    │
│  1. The RELATIONAL MODEL (tables, PKs, FKs, SQL) is the          │
│     foundation of enterprise data. Every Palantir client has one. │
│                                                                    │
│  2. ACID guarantees make databases trustworthy. Understand        │
│     transactions, concurrency, and why they matter.               │
│                                                                    │
│  3. Know the BIG FOUR:                                            │
│     • PostgreSQL — modern default, extensible, MVCC               │
│     • Oracle — enterprise titan, regulated industries, expensive  │
│     • MySQL/MariaDB — web-scale workhorse, InnoDB is default      │
│     • SQL Server — Microsoft ecosystem, SSIS/SSAS/SSRS stack      │
│                                                                    │
│  4. OLTP ≠ OLAP: Foundry sits on the OLAP side, ingesting from   │
│     OLTP sources. Row storage for transactions, columnar for       │
│     analytics. Know both and why they're different.               │
│                                                                    │
│  5. Palantir connects via JDBC/ODBC/REST/CDC connectors.         │
│     Agent-based for on-prem. Data quality checks are first-class. │
│     Understanding the source DB drives Ontology design quality.   │
└───────────────────────────────────────────────────────────────────┘
```

---

## 4.13 Connecting to Chapter 5

Now that you understand where data *lives* (relational databases, OLTP systems), the next chapter explores where data *travels* — **data warehouses, data lakes, and the modern data stack**. Chapter 5 will cover Snowflake, BigQuery, Redshift, Databricks, the lakehouse paradigm, and how Foundry fits into — and sometimes replaces — these analytical architectures.

---

*Chapter 4 complete. Proceed to Chapter 5: The Data Ecosystem II — Warehouses, Lakes, and the Modern Data Stack.*
