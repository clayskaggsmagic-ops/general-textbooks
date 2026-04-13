# Appendix A: Vocabulary Glossary — The Language of Palantir, Data, and the Interview

> *"If you can't explain it simply, you don't understand it well enough."*
> — Albert Einstein

---

This chapter compiles every critical term, acronym, and concept from the preceding 19 chapters into a single reference. Use it as a rapid-fire study aid before interviews, a lookup during case prep, and a confidence check — if you can define every term here from memory, you're ready.

Terms are organized by domain. Within each domain, entries are alphabetical.

---

## 20.1 Palantir — Company, Culture, and People

| Term | Definition |
|---|---|
| **8VC** | Venture capital fund co-founded by Joe Lonsdale after Palantir. |
| **AIPCon** | Palantir's flagship customer conference (AI Platform Conference); showcases real deployments and platform demos. AIPCon 8 held Sept 2025. |
| **Alex Karp** | CEO of Palantir since 2005; PhD in social theory from Goethe University Frankfurt (studied under Habermas); self-described "progressive" and "techno-nationalist." |
| **Boot Camp** | Intensive onboarding program (~2 weeks) for new Palantir hires; includes product training, mock deployments, and culture immersion. |
| **CRADA** | Cooperative Research and Development Agreement; mechanism for government-industry collaboration without traditional procurement. |
| **Delta (Δ)** | Internal name for Forward Deployed Engineer (FDE); focuses on technical implementation, data engineering, and pipeline work. |
| **Direct Sales Model** | Palantir's go-to-market strategy emphasizing direct relationships with clients rather than channel partners. |
| **Dual-Class Share Structure** | Palantir's governance model where founders retain outsized voting power through Class B shares (10 votes per share vs. 1 for Class A). |
| **Echo (Ε)** | Internal name for Deployment Strategist (DS); hybrid of product manager, consultant, data strategist, and client relationship manager. |
| **FDE** | Forward Deployed Engineer; Palantir's technical implementers who work on-site with clients to build pipelines and integrations. Same as Delta. |
| **FDS** | Forward Deployed Strategist; early name for what became the Deployment Strategist / Echo role. |
| **Forward Deployed** | Palantir's model of embedding engineers and strategists directly at client sites rather than working remotely from HQ. |
| **Founders Fund** | Peter Thiel's venture capital firm; early Palantir investor. |
| **Helper Model** | Palantir's philosophy that software should augment human decision-making, not replace it; humans remain in the loop. |
| **In-Q-Tel** | CIA's venture capital arm; early Palantir investor and first customer. |
| **Joe Lonsdale** | Palantir co-founder; later founded 8VC, Addepar, and Formation 8. |
| **Nathan Gettings** | Palantir co-founder; early engineering and operations. |
| **Palantír** | The "seeing stone" from Tolkien's *Lord of the Rings*; the namesake symbolizes the power of seeing hidden connections in data. |
| **Peter Thiel** | Palantir co-founder, chairman, initial funder; PayPal co-founder, early Facebook investor, Founders Fund. |
| **SPAC** | Special Purpose Acquisition Company; Palantir chose a Direct Listing (DPO) in Sept 2020 instead. |
| **Stephen Cohen** | Palantir co-founder; built the first working prototype; early engineering lead. |
| **Techno-Nationalist** | Karp's self-identification — belief that technological superiority is a pillar of national security. |
| **The Technological Republic** | Feb 2025 essay by Karp and Nicholas Zamiska; argues for tech-state collaboration, technological sovereignty, and critiques Silicon Valley's abdication of defense responsibility. |
| **Technological Sovereignty** | Core thesis: a nation's geopolitical power rests on its technological capabilities; democracies must out-build authoritarian regimes. |

---

## 20.2 Palantir Platforms and Products

| Term | Definition |
|---|---|
| **AIP (Artificial Intelligence Platform)** | Palantir's newest platform; brings LLMs into the Ontology so users can interact with operational data through natural language and AI-driven actions. |
| **Action** | A defined operation within the Ontology that modifies data or triggers workflows (e.g., approve shipment, reassign technician); ensures changes are auditable and governed. |
| **Apollo** | Palantir's continuous delivery and deployment infrastructure; manages software releases across cloud, on-prem, and air-gapped environments. |
| **Code Repositories** | Development environment within Foundry for writing transforms in Python, SQL, or Java. |
| **Code Workbook** | Interactive notebook-style environment in Foundry for exploratory analysis. |
| **Contour** | Visual analytics tool in Foundry for exploring, pivoting, and filtering Ontology data without code. |
| **Data Connection** | Foundry module for ingesting data from external sources (databases, APIs, file systems, SaaS platforms). |
| **Data Lineage** | Foundry's automatic tracking of every transformation applied to data — full provenance from raw source to final output. |
| **Edge AI** | Palantir's capability to deploy models and Ontology logic to disconnected/tactical edge environments (drones, forward bases, ships). |
| **FDP (Foundry for Defense Platforms)** | Foundry variant purpose-built for UK MoD; achieved OFFICIAL-SENSITIVE accreditation. |
| **Foundry** | Palantir's commercial platform for data integration, analytics, modeling, and operational workflows; built on the Ontology. |
| **Gotham** | Palantir's government/intelligence platform for integrating heterogeneous data, link analysis, geospatial mapping, and threat detection. |
| **Link Analysis** | Gotham's capability to map relationships between entities (people, organizations, transactions, events) and find hidden connections. |
| **Logic** | Rules engine within Foundry's Ontology; defines business logic (constraints, validations, computed properties) that executes on Actions. |
| **MetaConstellation** | Palantir's platform for coordinating autonomous and semi-autonomous systems (satellites, drones, sensors) in defense contexts. |
| **Monocle** | Gotham's search interface for quickly querying across all integrated datasets. |
| **Object Explorer** | Foundry UI for browsing, searching, and interacting with Ontology objects and their properties. |
| **Ontology** | Palantir's core abstraction layer — a semantic model that maps real-world entities (people, assets, transactions) as typed Objects with Properties, Links, and Actions. |
| **Ontology SDK** | Software development kit allowing external applications to interact with the Foundry Ontology programmatically. |
| **Pipeline Builder** | Visual, no-code tool in Foundry for constructing data transformation pipelines. |
| **Quiver** | Analytics and visualization layer in Foundry for building dashboards and reports on Ontology data. |
| **Slate** | Foundry's application-building framework for creating custom, user-facing web applications on top of the Ontology. |
| **Token-Based Access Control** | Foundry's granular security model where every data object, field, and action is governed by permissions tokens — ensuring row-level, column-level, and purpose-limited access. |
| **Workshop** | Foundry's no-code/low-code application builder for creating interactive operational applications using Ontology data. |

---

## 20.3 The Ontology — Core Concepts

| Term | Definition |
|---|---|
| **Action Type** | A defined category of operation (e.g., "Approve Order," "Reassign Asset") that specifies what inputs are required, what Objects are modified, and what validations run. |
| **Computed Property** | A Property whose value is derived in real time from other Properties, Links, or external data rather than stored statically. |
| **Entity Resolution** | The process of determining whether multiple data records refer to the same real-world entity; critical for building clean Ontology objects from messy source systems. |
| **Link** | A typed, directional relationship between two Object Types in the Ontology (e.g., Patient → assigned to → Doctor). |
| **Link Type** | The definition of a relationship class (cardinality, direction, semantics) between Object Types. |
| **Object** | An instance of an Object Type in the Ontology; represents a single real-world entity (e.g., a specific patient, a specific warehouse). |
| **Object Type** | A class definition in the Ontology that describes a category of real-world entity with its Properties and Links (e.g., "Patient," "Warehouse," "Work Order"). |
| **Property** | An attribute of an Object Type (e.g., Patient.date_of_birth, Warehouse.capacity); analogous to a column in a relational table. |
| **Reverse Link** | The inverse traversal of a Link (if Order → Customer exists, Customer → Orders is the reverse link). |
| **Writeback** | The pattern of pushing Ontology changes (from Actions) back into source systems, closing the loop between analytical insight and operational execution. |

---

## 20.4 Databases and SQL

| Term | Definition |
|---|---|
| **ACID** | Atomicity, Consistency, Isolation, Durability — the four properties guaranteeing reliable database transactions. |
| **Aggregate Function** | SQL functions that operate on sets of rows: COUNT, SUM, AVG, MIN, MAX. |
| **AUTO_INCREMENT** | Column attribute that automatically generates sequential unique integers (MySQL); SERIAL in PostgreSQL. |
| **B-Tree** | Balanced tree data structure used for database indexes; enables O(log n) search, insert, and delete. |
| **Candidate Key** | Any column or combination of columns that could serve as a primary key (unique, non-null). |
| **Cardinality** | (1) In ER modeling: the relationship ratio between entities (1:1, 1:M, M:N). (2) In indexes: the number of distinct values in a column. |
| **CHECK Constraint** | SQL constraint enforcing a Boolean condition on column values (e.g., age > 0). |
| **Clustered Index** | Index that physically reorders table rows to match the index order; one per table. |
| **Composite Key** | A primary key consisting of two or more columns together. |
| **Correlated Subquery** | A subquery that references columns from the outer query; re-executed for each outer row. |
| **Covering Index** | An index that contains all columns needed by a query, allowing the database to answer the query from the index alone without touching the table. |
| **CTE (Common Table Expression)** | A named temporary result set in SQL defined with WITH; improves readability and enables recursion. |
| **DDL** | Data Definition Language — SQL commands that define schema structure: CREATE, ALTER, DROP, TRUNCATE. |
| **Deadlock** | A situation where two or more transactions block each other by each holding a lock the other needs. |
| **DML** | Data Manipulation Language — SQL commands that modify data: INSERT, UPDATE, DELETE, MERGE. |
| **DQL** | Data Query Language — the SELECT statement and its clauses. |
| **EXCEPT / MINUS** | Set operation returning rows in the first result set but not in the second. |
| **Execution Plan** | The database engine's strategy for fulfilling a query; viewable via EXPLAIN/EXPLAIN ANALYZE. |
| **EXPLAIN** | SQL command that reveals the query execution plan the optimizer chose. |
| **Foreign Key (FK)** | A column that references the primary key of another table, enforcing referential integrity. |
| **FULL OUTER JOIN** | Returns all rows from both tables, with NULLs where no match exists on either side. |
| **GROUP BY** | SQL clause that partitions rows into groups for aggregate calculation. |
| **HAVING** | SQL clause that filters groups after aggregation (WHERE filters before). |
| **Index** | A data structure that speeds up data retrieval at the cost of additional storage and write overhead. |
| **Index Scan** | Query execution method that traverses an index to locate matching rows. |
| **INNER JOIN** | Returns only rows with matching values in both joined tables. |
| **INTERSECT** | Set operation returning only rows present in both result sets. |
| **Isolation Levels** | Transaction concurrency controls: Read Uncommitted, Read Committed, Repeatable Read, Serializable. |
| **Junction Table** | A table that resolves a many-to-many (M:N) relationship by holding foreign keys to both related tables. |
| **LEFT JOIN** | Returns all rows from the left table plus matching rows from the right; NULLs for non-matches. |
| **MVCC** | Multi-Version Concurrency Control — readers don't block writers by maintaining multiple row versions; used by PostgreSQL, Oracle, InnoDB. |
| **Non-Clustered Index** | An index stored separately from table data, containing pointers back to rows; multiple allowed per table. |
| **Normalization** | The process of organizing relational data to eliminate redundancy and prevent anomalies (1NF through BCNF). |
| **NULL** | A marker representing missing or unknown data; behaves differently from zero or empty string. |
| **OLAP** | Online Analytical Processing — workloads optimized for complex queries, aggregations, and historical analysis (data warehouses). |
| **OLTP** | Online Transaction Processing — workloads optimized for fast, frequent, small read/write transactions (operational systems). |
| **Optimistic Concurrency** | Conflict detection strategy where transactions proceed without locks and check for conflicts at commit time. |
| **Pessimistic Concurrency** | Conflict prevention strategy where transactions acquire locks before accessing data. |
| **Primary Key (PK)** | A unique, non-null identifier for each row in a table. |
| **Query Optimizer** | The database engine component that evaluates execution strategies and selects the most efficient plan. |
| **Recursive CTE** | A CTE that references itself; used for hierarchical/tree-structured data traversal. |
| **Referential Integrity** | The guarantee that foreign key values always point to existing primary key values. |
| **RIGHT JOIN** | Returns all rows from the right table plus matching rows from the left; NULLs for non-matches. |
| **Sargable** | "Search ARGument ABLE" — a query predicate that can use an index (e.g., WHERE col > 5 is sargable; WHERE YEAR(col) = 2024 is not). |
| **Self-Join** | A join where a table is joined to itself, typically to compare rows within the same table. |
| **Sequential Scan** | Query execution method that reads every row in a table; used when no useful index exists. |
| **SERIAL** | PostgreSQL auto-incrementing integer column type. |
| **SQL Order of Operations** | The logical processing order: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT. |
| **Stored Procedure** | Precompiled SQL code stored in the database for reuse; can contain control flow logic. |
| **Subquery** | A query nested inside another query; can appear in SELECT, FROM, or WHERE clauses. |
| **Transaction** | A sequence of SQL operations treated as a single atomic unit — all succeed or all roll back. |
| **Trigger** | A stored procedure that automatically fires in response to INSERT, UPDATE, or DELETE events on a table. |
| **UNION / UNION ALL** | Set operations combining result sets vertically; UNION eliminates duplicates, UNION ALL preserves them. |
| **Unique Constraint** | Ensures all values in a column or combination of columns are distinct. |
| **View** | A virtual table defined by a stored SELECT query; does not store data itself. |
| **WAL (Write-Ahead Log)** | A durability mechanism where changes are written to a log before being applied to the database; enables crash recovery. |
| **Window Function** | SQL functions that perform calculations across a set of rows related to the current row without collapsing them: ROW_NUMBER(), RANK(), DENSE_RANK(), LAG(), LEAD(), NTILE(), etc. |
| **1NF** | First Normal Form — every column contains atomic (indivisible) values; no repeating groups. |
| **2NF** | Second Normal Form — 1NF plus no partial dependencies (every non-key attribute depends on the entire primary key). |
| **3NF** | Third Normal Form — 2NF plus no transitive dependencies (non-key attributes don't depend on other non-key attributes). |
| **BCNF** | Boyce-Codd Normal Form — every determinant is a candidate key; stricter than 3NF. |

---

## 20.5 Data Modeling — Relational, Dimensional, Graph, and NoSQL

| Term | Definition |
|---|---|
| **Accumulating Snapshot** | A fact table type that captures the lifecycle of a process with milestone dates (e.g., order placed → shipped → delivered). |
| **Bridge Table** | A table used in dimensional models to handle multi-valued dimensions (e.g., a patient with multiple diagnoses). |
| **CAP Theorem** | Consistency, Availability, Partition tolerance — any distributed system can guarantee at most two of three simultaneously. |
| **Cassandra** | Apache column-family NoSQL database; designed for massive write throughput and high availability across data centers. |
| **Columnar Storage** | Storage format that stores data by column rather than by row; optimized for analytical queries (e.g., Parquet, ORC). |
| **Column-Family Store** | NoSQL category organizing data into rows and column families; examples: Cassandra, HBase. |
| **Conformed Dimension** | A dimension table shared across multiple fact tables and business processes, ensuring consistent analysis. |
| **Cypher** | Declarative graph query language used by Neo4j; uses ASCII-art syntax for pattern matching (e.g., (a)-[:KNOWS]->(b)). |
| **Degenerate Dimension** | A dimension with no separate table; its key lives directly in the fact table (e.g., invoice number). |
| **Denormalization** | Intentionally introducing redundancy to improve read performance; the reverse of normalization. |
| **Dimension Table** | A table in a dimensional model containing descriptive attributes (who, what, where, when) used for filtering, grouping, and labeling. |
| **Document Store** | NoSQL category storing data as flexible JSON/BSON documents; examples: MongoDB, Firestore, Couchbase. |
| **DynamoDB** | AWS fully-managed key-value and document store; single-digit millisecond latency at any scale. |
| **ER Diagram** | Entity-Relationship Diagram — visual representation of entities, attributes, and relationships in conceptual data modeling. |
| **Eventual Consistency** | A consistency model where replicas may temporarily diverge but will converge to the same state over time. |
| **Fact Table** | A table in a dimensional model containing quantitative measures (metrics) and foreign keys to dimension tables. |
| **Grain** | The level of detail a fact table row represents (e.g., one row per transaction, per day, per customer-product pair). |
| **Graph Database** | A database that uses nodes, edges, and properties to store and query highly connected data; examples: Neo4j, Amazon Neptune, TigerGraph. |
| **Inmon Approach** | Bill Inmon's data warehousing methodology: build a normalized enterprise data warehouse first, then derive dimensional data marts. |
| **Junk Dimension** | A dimension table that combines miscellaneous low-cardinality flags and indicators to reduce fact table width. |
| **Key-Value Store** | Simplest NoSQL category; O(1) lookups by key; examples: Redis, DynamoDB, Memcached. |
| **Kimball Approach** | Ralph Kimball's methodology: build dimensional data marts (star schemas) first, integrated via conformed dimensions (the "data warehouse bus"). |
| **Knowledge Graph** | A graph-based data structure that represents real-world entities and their relationships with semantic meaning; underpins Google Search, Palantir's Ontology, and enterprise knowledge management. |
| **Labeled Property Graph** | A graph model where both nodes and edges have labels (types) and key-value properties. |
| **Materialized View** | A view whose results are physically stored and periodically refreshed; trades storage for query speed. |
| **MongoDB** | Leading document-oriented NoSQL database; stores data as BSON documents with flexible schemas. |
| **Neo4j** | Leading graph database; uses the Cypher query language; open-source with enterprise edition. |
| **Ontology (General)** | A formal representation of knowledge as a set of concepts, their properties, and the relationships between them within a domain. |
| **OWL** | Web Ontology Language — W3C standard for defining ontologies with classes, properties, and logical constraints. |
| **Parquet** | Apache columnar storage file format optimized for analytics; supports efficient compression and encoding. |
| **Periodic Snapshot** | A fact table type capturing metrics at regular intervals (e.g., daily account balance, weekly inventory level). |
| **Polyglot Persistence** | Using multiple database technologies within a single system, each chosen for the data pattern it handles best. |
| **Property Graph** | A graph data model where nodes and edges can carry arbitrary key-value properties. |
| **RDF** | Resource Description Framework — W3C standard for representing information as subject-predicate-object triples. |
| **Redis** | In-memory key-value store; used for caching, session management, pub/sub messaging, and real-time leaderboards. |
| **Role-Playing Dimension** | A single dimension table used multiple times in a fact table for different purposes (e.g., Date dimension as OrderDate, ShipDate, DeliveryDate). |
| **SCD (Slowly Changing Dimension)** | Techniques for handling changes to dimension attributes over time: Type 1 (overwrite), Type 2 (new row with versioning), Type 3 (add column for previous value). |
| **Schema-on-Read** | Data is stored raw and schema is applied at query time; contrast with Schema-on-Write. |
| **Schema-on-Write** | Data must conform to a predefined schema before it can be stored; traditional RDBMS approach. |
| **Snowflake Schema** | A dimensional model where dimension tables are normalized into sub-dimensions; more storage-efficient but requires more joins than star schema. |
| **SPARQL** | Query language for RDF data; the graph-database equivalent of SQL for triple stores. |
| **Star Schema** | A dimensional model with a central fact table surrounded by denormalized dimension tables; optimized for query performance and simplicity. |
| **Surrogate Key** | A system-generated, meaningless identifier (e.g., auto-increment integer) used as a primary key instead of a natural business key. |
| **Transaction Fact** | A fact table type recording individual discrete events (e.g., each sale, each click, each shipment). |

---

## 20.6 Data Pipelines, ETL, and Integration

| Term | Definition |
|---|---|
| **Airflow** | Apache Airflow — open-source workflow orchestration platform; defines pipelines as Python DAGs. |
| **Avro** | Apache row-based data serialization format; schema embedded in files; popular in streaming pipelines. |
| **Batch Processing** | Processing data in scheduled, discrete chunks (e.g., nightly, hourly); contrast with streaming. |
| **CDC (Change Data Capture)** | Technique for identifying and capturing only the data that has changed since the last extraction; log-based (e.g., Debezium) or trigger-based. |
| **Circuit Breaker Pattern** | A fault-tolerance pattern where a pipeline stops sending requests to a failing system after repeated failures, preventing cascade failures. |
| **Completeness** | Data quality dimension: does the dataset have all expected records and fields? |
| **Consistency** | Data quality dimension: do related data elements agree across datasets? |
| **DAG (Directed Acyclic Graph)** | A graph with directed edges and no cycles; used to model pipeline dependencies in Airflow, Foundry, dbt, and Spark. |
| **Data Lake** | A centralized repository storing raw data in native format at any scale; schema-on-read; examples: S3, ADLS, GCS. |
| **Data Lakehouse** | Architecture combining data lake flexibility with data warehouse structure and performance; examples: Databricks Delta Lake, Apache Iceberg. |
| **Data Lineage** | The complete audit trail of data from source to destination, including every transformation applied. |
| **Data Mesh** | Organizational architecture treating data as a product owned by domain teams, with federated governance and self-serve infrastructure. |
| **Data Quality** | The six dimensions: Accuracy, Completeness, Consistency, Timeliness, Uniqueness, Validity. |
| **Data Warehouse** | A centralized, schema-on-write analytical data store optimized for complex queries and reporting; examples: Snowflake, BigQuery, Redshift. |
| **Databricks** | Unified analytics platform built on Apache Spark; combines data engineering, data science, and machine learning. |
| **dbt (data build tool)** | Open-source tool for transforming data in warehouses using SQL SELECT statements; handles dependency management, testing, and documentation. |
| **Dead Letter Queue** | A holding area for messages that fail processing; allows investigation and reprocessing without blocking the main pipeline. |
| **Debezium** | Open-source CDC platform that captures database changes from transaction logs (MySQL, PostgreSQL, MongoDB, etc.). |
| **Delta Lake** | Open-source storage layer from Databricks providing ACID transactions, schema enforcement, and time travel on data lakes. |
| **ELT** | Extract, Load, Transform — modern pattern: load raw data into the warehouse/lake first, then transform using the target's compute engine. |
| **ETL** | Extract, Transform, Load — traditional pattern: transform data in a staging area before loading into the target system. |
| **Exactly-Once Semantics** | Message processing guarantee ensuring each message is processed exactly one time, despite failures or retries. |
| **Fivetran** | Managed ELT platform offering pre-built connectors for hundreds of data sources. |
| **Great Expectations** | Open-source data quality framework for defining and validating data expectations (tests). |
| **Iceberg** | Apache table format providing ACID transactions, schema evolution, and time travel for data lakes. |
| **Idempotency** | The property ensuring that running the same pipeline or operation multiple times produces the same result. |
| **Kafka** | Apache Kafka — distributed event streaming platform for building real-time data pipelines; uses topics, partitions, producers, and consumers. |
| **Kinesis** | AWS managed streaming service; alternative to Kafka for real-time data ingestion. |
| **Micro-Batching** | Processing data in very small batches (seconds to minutes) to approximate real-time; used by Spark Structured Streaming. |
| **Orchestration** | Coordinating the execution of pipeline tasks in the correct order with dependency management, retries, and monitoring. |
| **ORC** | Optimized Row Columnar — Apache columnar storage format; optimized for Hive workloads. |
| **Prefect** | Modern Python-native workflow orchestration platform; alternative to Airflow with a more Pythonic API. |
| **Pub/Sub** | Google Cloud Pub/Sub — fully-managed messaging service for event-driven architectures. |
| **SLA (Data Pipeline)** | Service Level Agreement defining freshness and availability guarantees (e.g., "dashboard updated within 15 minutes of source change"). |
| **Snowflake (Platform)** | Cloud-native data warehouse platform with separation of storage and compute; supports structured and semi-structured data. |
| **Spark** | Apache Spark — unified analytics engine for large-scale data processing; supports batch, streaming, SQL, ML, and graph processing. |
| **Streaming** | Continuous, real-time data processing as events arrive; contrast with batch. |
| **Timeliness** | Data quality dimension: is the data available when needed and how fresh is it? |

---

## 20.7 Data Structures

| Term | Definition |
|---|---|
| **Adjacency List** | Graph representation where each vertex stores a list of its neighbors; space-efficient for sparse graphs. |
| **Adjacency Matrix** | Graph representation using a 2D matrix; fast O(1) edge lookup but O(V²) space. |
| **Array** | Contiguous block of memory storing elements of the same type; O(1) random access by index. |
| **AVL Tree** | Self-balancing BST where the height difference between left and right subtrees is at most 1; guarantees O(log n) operations. |
| **BFS (Breadth-First Search)** | Graph/tree traversal visiting all nodes at the current depth before moving deeper; uses a queue; O(V+E). |
| **Binary Heap** | Complete binary tree satisfying the heap property (min-heap: parent ≤ children; max-heap: parent ≥ children); used for priority queues. |
| **Binary Search** | O(log n) search on sorted data by repeatedly halving the search space. |
| **Binary Search Tree (BST)** | Binary tree where left children are smaller and right children are larger than the parent; supports O(log n) search, insert, delete when balanced. |
| **Circular Queue** | Queue implemented in a fixed-size array where the end wraps around to the beginning. |
| **Collision (Hash)** | When two distinct keys produce the same hash value; resolved via chaining or open addressing. |
| **Complete Graph** | A graph where every pair of vertices is connected by an edge. |
| **Connected Component** | A maximal subgraph in which every vertex is reachable from every other vertex. |
| **Cycle (Graph)** | A path that starts and ends at the same vertex with no repeated edges. |
| **DAG (Data Structure)** | Directed Acyclic Graph — a directed graph with no cycles; commonly used for dependency ordering. |
| **Degree (Graph)** | The number of edges connected to a vertex (in-degree for incoming; out-degree for outgoing in directed graphs). |
| **Deque** | Double-ended queue — supports insertion and deletion at both front and rear in O(1). |
| **DFS (Depth-First Search)** | Graph/tree traversal that explores as deep as possible before backtracking; uses a stack (explicit or call stack); O(V+E). |
| **Doubly Linked List** | Linked list where each node has pointers to both the next and previous nodes; enables O(1) bidirectional traversal and deletion. |
| **Dynamic Array** | Array that resizes automatically when capacity is exceeded (e.g., Python list, Java ArrayList); amortized O(1) append. |
| **Graph** | Data structure consisting of vertices (nodes) and edges (connections); can be directed/undirected, weighted/unweighted, cyclic/acyclic. |
| **Hash Function** | A function that maps input data to a fixed-size integer (hash code); should distribute keys uniformly. |
| **Hash Map / Hash Table** | Key-value store using a hash function for O(1) average-case lookup, insert, and delete. |
| **Heap** | A specialized tree-based structure satisfying the heap property; supports O(log n) insert and O(1) find-min/max. |
| **In-Order Traversal** | BST traversal visiting left → root → right; produces sorted output. |
| **Level-Order Traversal** | Tree traversal visiting nodes level by level (BFS applied to trees). |
| **Linked List** | Linear data structure where elements (nodes) are linked via pointers; O(1) insert/delete at known positions, O(n) search. |
| **Load Factor** | The ratio of stored elements to bucket count in a hash table; triggers resizing when exceeded (typically > 0.75). |
| **Post-Order Traversal** | Tree traversal visiting left → right → root; used for deletion and expression evaluation. |
| **Pre-Order Traversal** | Tree traversal visiting root → left → right; used for copying trees and serialization. |
| **Priority Queue** | Abstract data type where elements are dequeued by priority, not insertion order; typically implemented with a heap. |
| **Queue** | FIFO (First-In, First-Out) data structure; O(1) enqueue and dequeue. |
| **Red-Black Tree** | Self-balancing BST using node coloring rules; guarantees O(log n) operations; used in Java TreeMap, C++ std::map. |
| **Singly Linked List** | Linked list where each node points only to the next node. |
| **Sparse Graph** | A graph where the number of edges is much less than V²; adjacency lists are preferred. |
| **Stack** | LIFO (Last-In, First-Out) data structure; O(1) push and pop; used for function calls, undo, parsing. |
| **Topological Sort** | Linear ordering of vertices in a DAG such that for every directed edge u→v, u appears before v; used for task scheduling and dependency resolution. |
| **Tree** | A connected, acyclic graph; hierarchical structure with a root node and children; n nodes have exactly n-1 edges. |
| **Trie** | Tree-like structure for storing strings character by character; O(m) lookup where m is string length; used for autocomplete and prefix matching. |
| **Weighted Graph** | A graph where edges carry numerical values (weights) representing cost, distance, or capacity. |

---

## 20.8 Algorithms

| Term | Definition |
|---|---|
| **Activity Selection** | Classic greedy problem: select the maximum number of non-overlapping activities. |
| **Amortized Analysis** | Averaging the cost of operations over a sequence to show that even if individual operations are expensive, the average cost is low. |
| **Bellman-Ford** | Single-source shortest path algorithm handling negative edge weights; O(VE); detects negative cycles. |
| **Big-O Notation** | Upper bound on growth rate of an algorithm's time or space complexity (e.g., O(n), O(log n), O(n²)). |
| **Bubble Sort** | Simple comparison sort; O(n²); repeatedly swaps adjacent out-of-order elements. |
| **Coin Change Problem** | Classic DP problem: find the minimum number of coins summing to a target value. |
| **Counting Sort** | Non-comparison integer sort; O(n+k) where k is the range of values; stable. |
| **Dijkstra's Algorithm** | Single-source shortest path algorithm for non-negative edge weights; O((V+E) log V) with a min-heap. |
| **Divide and Conquer** | Algorithmic paradigm: split the problem into subproblems, solve recursively, combine results. |
| **Dynamic Programming (DP)** | Algorithmic paradigm for optimization problems with overlapping subproblems and optimal substructure; stores solutions to subproblems (memoization or tabulation). |
| **Edit Distance** | DP problem computing the minimum number of insertions, deletions, and substitutions to transform one string into another (Levenshtein distance). |
| **Floyd-Warshall** | All-pairs shortest path algorithm; O(V³); uses dynamic programming. |
| **Greedy Algorithm** | Algorithmic paradigm: make the locally optimal choice at each step; works when greedy choice property and optimal substructure hold. |
| **Heap Sort** | Comparison sort using a binary heap; O(n log n); in-place. |
| **Huffman Coding** | Greedy algorithm for lossless data compression; builds an optimal prefix-free binary code based on character frequencies. |
| **Insertion Sort** | Simple comparison sort; O(n²) worst case, O(n) best case (nearly sorted data); stable; efficient for small datasets. |
| **Knapsack Problem (0/1)** | Classic DP problem: maximize value of items placed in a knapsack with weight capacity W, where each item can be taken or left. |
| **Kruskal's Algorithm** | Minimum spanning tree algorithm using a greedy approach with sorted edges and Union-Find; O(E log E). |
| **LCS (Longest Common Subsequence)** | DP problem finding the longest subsequence common to two sequences. |
| **LIS (Longest Increasing Subsequence)** | DP problem finding the longest strictly increasing subsequence within a sequence. |
| **Memoization** | Top-down DP technique: cache results of recursive calls to avoid recomputation. |
| **Merge Sort** | Divide-and-conquer comparison sort; O(n log n); stable; requires O(n) extra space. |
| **Minimum Spanning Tree (MST)** | A subset of edges connecting all vertices in a weighted graph with the minimum total edge weight and no cycles. |
| **Optimal Substructure** | Property where the optimal solution to a problem contains optimal solutions to its subproblems. |
| **Overlapping Subproblems** | Property where the same subproblems are solved repeatedly in a recursive solution. |
| **Prim's Algorithm** | Minimum spanning tree algorithm growing the tree one vertex at a time; O((V+E) log V) with a min-heap. |
| **Quick Sort** | Divide-and-conquer comparison sort; O(n log n) average, O(n²) worst case; in-place; fastest in practice for most inputs. |
| **Radix Sort** | Non-comparison sort processing digits from least to most significant; O(d·(n+k)). |
| **Selection Sort** | Simple comparison sort; O(n²); selects the minimum element and swaps it into position. |
| **Sliding Window** | Technique for processing subarrays/substrings by maintaining a window that slides across the data; O(n). |
| **Stable Sort** | A sorting algorithm that preserves the relative order of elements with equal keys. |
| **Tabulation** | Bottom-up DP technique: fill a table iteratively starting from the smallest subproblems. |
| **Time Complexity** | A measure of how an algorithm's running time grows as input size increases. |
| **Two-Pointer Technique** | Pattern using two indices traversing a data structure (often from opposite ends) to solve problems in O(n). |
| **Union-Find** | Data structure tracking disjoint sets; supports near-O(1) union and find with path compression and union by rank; used in Kruskal's MST. |

---

## 20.9 Cloud, Big Data, and the Modern Data Ecosystem

| Term | Definition |
|---|---|
| **ADLS** | Azure Data Lake Storage — Microsoft's cloud-based data lake on Azure. |
| **API** | Application Programming Interface — a defined contract for how software components communicate. |
| **AWS** | Amazon Web Services — Amazon's cloud computing platform. |
| **Azure** | Microsoft Azure — Microsoft's cloud computing platform. |
| **BigQuery** | Google Cloud's serverless, petabyte-scale data warehouse. |
| **Cloud-Native** | Software designed from the ground up to run in cloud environments, leveraging containers, microservices, and managed services. |
| **Container** | Lightweight, portable package of code and dependencies (e.g., Docker); runs consistently across environments. |
| **Docker** | Platform for building and running containers; standardized packaging for applications. |
| **EC2** | Elastic Compute Cloud — AWS's virtual server service. |
| **GCS** | Google Cloud Storage — Google's cloud object storage service. |
| **GCP** | Google Cloud Platform — Google's cloud computing platform. |
| **Hadoop** | Apache framework for distributed storage (HDFS) and processing (MapReduce) of large datasets. |
| **HDFS** | Hadoop Distributed File System — distributed storage layer that splits files across cluster nodes. |
| **Hive** | Apache data warehouse software built on Hadoop; provides SQL-like querying (HiveQL) over large datasets. |
| **IAM** | Identity and Access Management — cloud services for controlling who can access what resources. |
| **JSON** | JavaScript Object Notation — lightweight data interchange format. |
| **Kubernetes (K8s)** | Open-source container orchestration platform for automating deployment, scaling, and management. |
| **MapReduce** | Programming model for processing large datasets in parallel across a cluster: Map (transform) → Shuffle → Reduce (aggregate). |
| **Microservices** | Architectural pattern where an application is composed of small, independently deployable services. |
| **ML Model** | Machine Learning model — a trained algorithm that makes predictions or classifications from data. |
| **Object Storage** | Cloud storage service for storing unstructured data as objects (files + metadata); examples: S3, GCS, Azure Blob. |
| **On-Prem (On-Premises)** | Infrastructure hosted in a company's own data centers rather than in the cloud. |
| **PII** | Personally Identifiable Information — data that can identify an individual (name, SSN, email, etc.); subject to privacy regulations. |
| **Presto / Trino** | Distributed SQL query engine for running interactive analytics across heterogeneous data sources. |
| **Redshift** | AWS's cloud data warehouse service; columnar storage with massively parallel processing. |
| **REST API** | Representational State Transfer — architectural style for web APIs using HTTP methods (GET, POST, PUT, DELETE). |
| **S3** | Amazon Simple Storage Service — AWS's object storage service; the de facto cloud data lake storage. |
| **SaaS** | Software as a Service — cloud-delivered application (e.g., Salesforce, Workday). |
| **Serverless** | Cloud execution model where the provider manages infrastructure; users pay per invocation (e.g., AWS Lambda, Cloud Functions). |
| **Terraform** | Infrastructure-as-Code tool for provisioning and managing cloud resources declaratively. |
| **VPC** | Virtual Private Cloud — a logically isolated network within a cloud provider. |

---

## 20.10 Real-World Deployments and Industry Terms

| Term | Definition |
|---|---|
| **AI Kill Chain** | Defense concept: the sensor-to-decision cycle enhanced by AI; Palantir's Gotham/MetaConstellation accelerates targeting and decision-making. |
| **AML** | Anti-Money Laundering — regulatory regime and analytical processes for detecting illicit financial flows. |
| **Army Enterprise Agreement** | Palantir's $10B+ contract to provide the U.S. Army's enterprise data and analytics platform. |
| **C2 (Command and Control)** | Military system for exercising authority over assigned forces; Palantir's platforms serve as C2 decision support. |
| **Control Tower** | Supply chain management concept: a centralized hub providing end-to-end visibility across the supply chain. |
| **Data42** | Novartis's internal data science initiative leveraging Foundry for drug development, clinical trials, and supply chain optimization. |
| **Digital Twin** | A virtual replica of a physical asset, process, or system updated with real-time data; bp and GE Aerospace use Foundry-powered digital twins. |
| **EHR** | Electronic Health Record — digital version of a patient's medical chart; key data source in healthcare deployments. |
| **FedRAMP** | Federal Risk and Authorization Management Program — U.S. government security standard for cloud services; Palantir is FedRAMP authorized. |
| **HIPAA** | Health Insurance Portability and Accountability Act — U.S. law protecting patient health information. |
| **IED** | Improvised Explosive Device — Palantir's early Gotham deployments in Afghanistan focused on predicting IED placement networks. |
| **IL-5/IL-6** | Impact Level 5/6 — DoD cloud security classifications for controlled unclassified and classified information. |
| **ISR** | Intelligence, Surveillance, and Reconnaissance — military data collection functions that feed into analytical platforms. |
| **KYC** | Know Your Customer — due diligence process in financial services to verify client identity. |
| **NHS** | UK National Health Service — used Foundry (FDP) for COVID-19 pandemic response, vaccine rollout, and elective care recovery. |
| **Operational AI** | AI embedded in day-to-day workflows that drives real-time decisions and actions, as opposed to experimental or research-only AI applications. |
| **Project Maven** | DoD initiative (initially Google, now partially Palantir) applying AI/ML to drone imagery analysis and intelligence processing. |
| **SAR (Suspicious Activity Report)** | A financial filing flagging potentially suspicious transactions; generated through AML workflows. |
| **SCIF** | Sensitive Compartmented Information Facility — a secured room for processing classified information. |
| **SOC 2** | Service Organization Control Type 2 — security audit framework; Palantir maintains SOC 2 compliance. |
| **TeleTracking** | Healthcare technology company that partnered with Palantir and HHS during COVID-19 for hospital capacity tracking. |
| **UK MoD** | United Kingdom Ministry of Defence — Palantir's £1.5B contract for defense information systems using FDP. |

---

## 20.11 The Interview — Process, Frameworks, and Techniques

| Term | Definition |
|---|---|
| **Behavioral Deep-Dive** | Interview round probing past experiences using structured storytelling (STAR method) to assess judgment, collaboration, and impact. |
| **Case Study Round** | Interview round presenting an open-ended data scenario where candidates design an end-to-end solution, demonstrating data fluency and structured thinking. |
| **Clarifying Questions** | Questions asked at the start of a case to narrow scope, understand constraints, identify stakeholders, and define success metrics. |
| **Decomposition Round** | Interview round testing the ability to break a complex, ambiguous problem into smaller, tractable components and design a structured approach. |
| **Grain (Interview Context)** | Defining what each row in a dataset represents — a critical step in case study and data modeling exercises. |
| **Hiring Manager Round** | Final interview round focused on fit, career trajectory, and mutual evaluation; often includes product intuition questions. |
| **Learning Round** | Interview round where candidates are taught a new concept on the spot and immediately asked to apply it; tests intellectual curiosity and adaptability. |
| **Phone Screen (Echo)** | 30-45 minute interview with a current Echo testing behavioral/experiential depth plus an open-ended data problem-solving exercise. |
| **Recruiter Screen** | Initial screening call assessing basic fit, communication skills, genuine interest in Palantir, and logistical alignment. |
| **STAR Method** | Behavioral interview framework: Situation, Task, Action, Result — structures answers to demonstrate impact with specificity. |
| **The 7-Step Framework** | Case study approach: (1) Understand scenario, (2) Ask clarifying questions, (3) Define data landscape, (4) Design approach, (5) Address trade-offs, (6) Consider edge cases, (7) Iterate. |
| **Trade-Off Analysis** | Explicitly weighing competing priorities (accuracy vs. speed, completeness vs. timeliness, centralized vs. distributed) and articulating the reasoning. |
| **Virtual Onsite** | The full panel of 3-5 interview rounds conducted remotely, typically 30-60 minutes each. |

---

## 20.12 Philosophy, Governance, and Regulatory

| Term | Definition |
|---|---|
| **Audit Trail** | A chronological record of system activity showing who accessed what data, when, and what they did with it; fundamental to Palantir's privacy model. |
| **CCPA** | California Consumer Privacy Act — state-level privacy regulation giving consumers rights over personal data. |
| **Data Governance** | The organizational framework of policies, processes, and standards for managing data quality, security, privacy, and compliance. |
| **Data Steward** | A role responsible for maintaining data quality, definitions, and governance policies within a domain. |
| **GDPR** | General Data Protection Regulation — EU law governing personal data protection and privacy. |
| **Granular Access Controls** | Security model where permissions can be set at fine-grained levels (row, column, cell, purpose) rather than all-or-nothing. |
| **Habermas, Jürgen** | German philosopher and social theorist; Alex Karp's doctoral advisor at Frankfurt; influenced Karp's thinking on democratic institutions and communicative rationality. |
| **Human-in-the-Loop** | Design philosophy ensuring that humans retain decision-making authority, especially for consequential actions; AI recommends, humans decide. |
| **Master Data Management (MDM)** | The discipline of maintaining a single, authoritative source of truth for core business entities (customers, products, locations). |
| **Metadata** | Data about data — descriptions of structure, meaning, lineage, ownership, and quality characteristics. |
| **Privacy-Security Complementarity** | Palantir's core thesis that privacy and security are not opposed; strong access controls and audit trails enable both. |
| **Purpose Limitation** | The principle that data should only be used for the specific purpose for which it was collected or authorized. |
| **RBAC** | Role-Based Access Control — assigning permissions based on user roles rather than individual identities. |
| **Technological Sovereignty** | The thesis from *The Technological Republic* that a nation's power and freedom depend on its technological capabilities relative to adversaries. |

---

## Quick-Reference: Acronym Index

| Acronym | Expansion |
|---|---|
| ACID | Atomicity, Consistency, Isolation, Durability |
| ADLS | Azure Data Lake Storage |
| AIP | Artificial Intelligence Platform |
| AML | Anti-Money Laundering |
| API | Application Programming Interface |
| AVL | Adelson-Velsky and Landis (tree) |
| AWS | Amazon Web Services |
| BCNF | Boyce-Codd Normal Form |
| BFS | Breadth-First Search |
| BST | Binary Search Tree |
| C2 | Command and Control |
| CAP | Consistency, Availability, Partition tolerance |
| CCPA | California Consumer Privacy Act |
| CDC | Change Data Capture |
| CTE | Common Table Expression |
| DAG | Directed Acyclic Graph |
| DDL | Data Definition Language |
| DFS | Depth-First Search |
| DML | Data Manipulation Language |
| DP | Dynamic Programming |
| DQL | Data Query Language |
| DS | Deployment Strategist |
| EHR | Electronic Health Record |
| ELT | Extract, Load, Transform |
| ER | Entity-Relationship |
| ETL | Extract, Transform, Load |
| FDE | Forward Deployed Engineer |
| FDP | Foundry for Defense Platforms |
| FIFO | First-In, First-Out |
| FK | Foreign Key |
| GCP | Google Cloud Platform |
| GCS | Google Cloud Storage |
| GDPR | General Data Protection Regulation |
| HDFS | Hadoop Distributed File System |
| HIPAA | Health Insurance Portability and Accountability Act |
| IAM | Identity and Access Management |
| IED | Improvised Explosive Device |
| IL | Impact Level (DoD cloud classification) |
| ISR | Intelligence, Surveillance, and Reconnaissance |
| JSON | JavaScript Object Notation |
| K8s | Kubernetes |
| KYC | Know Your Customer |
| LCS | Longest Common Subsequence |
| LIFO | Last-In, First-Out |
| LIS | Longest Increasing Subsequence |
| MDM | Master Data Management |
| ML | Machine Learning |
| MST | Minimum Spanning Tree |
| MVCC | Multi-Version Concurrency Control |
| NHS | National Health Service |
| NF | Normal Form |
| OLAP | Online Analytical Processing |
| OLTP | Online Transaction Processing |
| ORC | Optimized Row Columnar |
| OWL | Web Ontology Language |
| PII | Personally Identifiable Information |
| PK | Primary Key |
| RBAC | Role-Based Access Control |
| RDF | Resource Description Framework |
| REST | Representational State Transfer |
| S3 | Simple Storage Service |
| SaaS | Software as a Service |
| SAR | Suspicious Activity Report |
| SCD | Slowly Changing Dimension |
| SCIF | Sensitive Compartmented Information Facility |
| SLA | Service Level Agreement |
| SOC 2 | Service Organization Control Type 2 |
| SPARQL | SPARQL Protocol and RDF Query Language |
| SQL | Structured Query Language |
| STAR | Situation, Task, Action, Result |
| VPC | Virtual Private Cloud |
| WAL | Write-Ahead Log |

---

*You now have every term you'll need. If you can define each one from memory — without looking — you can speak the language of Palantir, data engineering, and this interview with authority.*

