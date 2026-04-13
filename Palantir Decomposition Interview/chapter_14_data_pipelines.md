# Chapter 14: Data Pipelines — ETL, ELT, and Integration Architecture

> *"Data is the new oil. Like oil, data is valuable, but if unrefined it cannot really be used."*
> — Clive Humby

---

## Why This Chapter Matters for the DS Interview

As a Deployment Strategist, you'll collaborate directly with Forward Deployed Engineers (FDEs) to identify customer datasets, design integration architectures, and build stable pipelines that feed the Ontology. You won't write Spark jobs daily — but you *must* understand:

- **How data moves** from source systems into Foundry
- **Why pipelines break** and how to design resilient ones
- **Which integration pattern** fits a given customer scenario
- **How to evaluate data quality** before it poisons downstream analytics

This chapter gives you the vocabulary and mental models to have those conversations fluently.

---

## 14.1 ETL vs. ELT — The Fundamental Architecture Decision

Every data pipeline answers one question: *where does transformation happen?*

### 14.1.1 ETL: Extract, Transform, Load

The traditional approach — dominant from the 1990s through the mid-2010s.

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐
│  SOURCE  │────▶│   STAGING    │────▶│  WAREHOUSE   │
│ SYSTEMS  │     │    AREA      │     │  (cleaned)   │
└──────────┘     │  Transform   │     └──────────────┘
                 │  here first  │
                 └──────────────┘
```

**How it works:**
1. **Extract** — Pull data from operational databases, APIs, flat files
2. **Transform** — Clean, validate, deduplicate, conform, aggregate in a *separate staging area* (often a dedicated ETL server)
3. **Load** — Write the cleaned, structured data into the target warehouse

**When ETL is still the right choice:**
- **Strict compliance requirements** — HIPAA, GDPR, PCI-DSS demand that sensitive data is masked/anonymized *before* it lands in the warehouse
- **Small, structured datasets** — When source data is well-defined and transformations are straightforward
- **Legacy on-prem infrastructure** — When compute at the warehouse is expensive or limited
- **Bandwidth constraints** — When reducing data volume *before* transfer saves significant network costs

**Classic ETL tools:** Informatica PowerCenter, IBM DataStage, Talend, SSIS (SQL Server Integration Services)

### 14.1.2 ELT: Extract, Load, Transform

The modern cloud-native approach — enabled by the separation of compute and storage in platforms like Snowflake, BigQuery, and Databricks.

```
┌──────────┐     ┌──────────────────────────────────┐
│  SOURCE  │────▶│         CLOUD WAREHOUSE          │
│ SYSTEMS  │     │                                    │
└──────────┘     │  Raw Data    ──▶  Transform here  │
                 │  (landed)        (using warehouse │
                 │                   compute power)  │
                 └──────────────────────────────────┘
```

**How it works:**
1. **Extract** — Pull raw data from sources
2. **Load** — Land the raw data directly into the warehouse/lake (as-is)
3. **Transform** — Use the warehouse's own compute engine (SQL, Spark) to clean and model the data

**Why ELT became the default:**
- **Elastic compute** — Cloud warehouses can scale transformation compute independently (Snowflake's virtual warehouses, BigQuery's slots)
- **Schema-on-read flexibility** — Store everything raw; decide how to structure it later
- **Replayability** — Raw data is preserved; transformations can be re-run as business logic changes
- **Cost efficiency** — Storage is cheap ($23/TB/month on S3); compute is pay-per-use
- **Speed** — No separate staging bottleneck; MPP engines transform at warehouse scale

### 14.1.3 Head-to-Head Comparison

| Dimension | ETL | ELT |
|---|---|---|
| **Transform location** | Separate staging server | Inside the warehouse/lake |
| **Raw data preservation** | Usually discarded after transform | Preserved permanently |
| **Compute model** | Fixed ETL server capacity | Elastic cloud compute |
| **Time to first value** | Longer (transform before load) | Faster (load immediately) |
| **Flexibility** | Schema-on-write (rigid) | Schema-on-read (flexible) |
| **Data volume sweet spot** | Small to medium | Medium to massive |
| **Compliance advantage** | Transform/mask before landing | Must secure raw landing zone |
| **Reprocessing** | Requires re-extraction | Re-run transforms on raw data |
| **Dominant era** | 1990s–2015 | 2015–present |
| **Modern tools** | Informatica, SSIS, Talend | dbt, Spark, warehouse-native SQL |

> **DS Interview Angle:** Palantir Foundry uses an ELT-like pattern. Raw data lands via Data Connection into "raw" datasets, and Pipeline Builder / Code Workbook transforms it within Foundry's compute environment. Understanding this pattern helps you explain how customer data flows through the platform.

---

## 14.2 Data Integration Patterns

Different data freshness requirements demand different integration approaches. A DS must match the right pattern to each customer's operational reality.

### 14.2.1 Batch Processing

The workhorse of enterprise data integration — process data in scheduled chunks.

```
┌──────────┐    Scheduled     ┌──────────┐    Bulk      ┌──────────┐
│  Source   │───────────────▶│  Extract  │───────────▶│  Target  │
│  System   │   (hourly,     │  & Load   │   Write     │  System  │
└──────────┘    daily)       └──────────┘             └──────────┘
```

**Characteristics:**
- Data processed at **fixed intervals** (hourly, daily, weekly)
- Higher latency but simpler architecture
- Typically uses **full loads** (re-extract everything) or **incremental loads** (extract only changed records since last run)
- Well-suited for reporting, analytics, compliance snapshots

**Micro-batching** — A middle ground:
- Process data in very small batches (every 1–5 minutes)
- Near-real-time freshness without full streaming complexity
- Spark Structured Streaming defaults to this approach

**When batch is sufficient:**
- Daily financial reconciliation
- Nightly data warehouse refreshes
- Weekly compliance reporting
- Any use case where data can be hours or days old

### 14.2.2 Real-Time / Stream Processing

For use cases where data freshness is measured in *seconds*, not hours.

```
┌──────────┐    Events     ┌──────────────┐   Continuous  ┌──────────┐
│  Source   │─────────────▶│  Message     │──────────────▶│  Stream  │
│  System   │  (continuous) │  Broker      │   Processing  │  Consumer│
└──────────┘               │  (Kafka,     │               └──────────┘
                           │   Kinesis)   │
                           └──────────────┘
```

**Key technologies:**

| Platform | Provider | Key Feature |
|---|---|---|
| **Apache Kafka** | Open source (Confluent) | Distributed log, topic-based pub/sub, partitioned for parallelism, replay capability |
| **Amazon Kinesis** | AWS | Managed streaming, shard-based scaling, tight AWS integration |
| **Google Pub/Sub** | GCP | Serverless, auto-scaling, at-least-once delivery, global routing |
| **Apache Flink** | Open source | True event-time processing, exactly-once semantics, complex event processing |
| **Spark Structured Streaming** | Open source (Databricks) | Unified batch + streaming API, micro-batch default |

**Event-driven architecture pattern:**

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ Producer│──▶│  Topic  │──▶│Consumer │──▶│ Action  │
│ (POS)   │   │ (Kafka) │   │ Group A │   │ (Fraud  │
└─────────┘   │         │──▶│Consumer │   │ Alert)  │
              │         │   │ Group B │   └─────────┘
              └─────────┘   └─────────┘
```

Producers publish events to topics; multiple independent consumer groups can each read the full stream for different purposes (fraud detection, inventory update, analytics).

**When streaming is necessary:**
- Fraud detection (sub-second response required)
- IoT sensor monitoring (continuous telemetry)
- Operational dashboards (live metrics)
- Real-time recommendation engines
- Financial market data processing

### 14.2.3 Change Data Capture (CDC)

Capture *only what changed* in the source system — the most efficient way to keep a target in sync.

```
┌──────────────┐                              ┌──────────────┐
│   Source DB   │                              │   Target     │
│              │    Transaction Log            │   System     │
│  INSERT ─────│──▶ ┌──────────┐ ──▶ Apply ──▶│              │
│  UPDATE ─────│──▶ │   CDC    │    changes   │              │
│  DELETE ─────│──▶ │  Reader  │              │              │
└──────────────┘    └──────────┘              └──────────────┘
```

**Three CDC approaches:**

| Approach | Mechanism | Pros | Cons |
|---|---|---|---|
| **Log-based** | Read database transaction log (WAL, binlog) | Zero impact on source, captures all changes including deletes | Requires DB-level access, log format varies by DB |
| **Trigger-based** | Database triggers fire on DML events | Simple to implement | Performance overhead on source, misses DDL changes |
| **Timestamp-based** | Query rows WHERE modified_at > last_run | No special access needed | Misses deletes, requires timestamp column on every table |

**Debezium** — the dominant open-source CDC platform:
- Built on Kafka Connect
- Monitors database transaction logs (MySQL binlog, PostgreSQL WAL, MongoDB oplog)
- Streams row-level changes as Kafka events
- Supports initial snapshots plus continuous streaming
- Used extensively in enterprise migration and real-time sync scenarios

**When CDC excels:**
- Migrating from legacy databases to cloud warehouses
- Keeping analytics systems in near-real-time sync
- Reducing load on source systems vs. full batch extracts
- Maintaining audit trails of every data change

### 14.2.4 API-Based Integration

Pull data from systems that expose REST or GraphQL APIs — the standard for SaaS platforms.

```
┌──────────────┐    HTTP/REST    ┌──────────────┐
│  SaaS API    │◀──────────────▶│  Integration │
│  (Salesforce,│    Paginated    │  Layer       │
│   Stripe,    │    Requests     │  (Fivetran,  │
│   HubSpot)   │                │   Airbyte)   │
└──────────────┘                └──────────────┘
```

**Challenges with API-based integration:**
- **Rate limiting** — APIs throttle requests (e.g., 100 requests/minute)
- **Pagination** — Large datasets require cursor-based or offset pagination
- **Authentication** — OAuth2 tokens, API keys, refresh cycles
- **Schema changes** — API versioning (v1 → v2) can break integrations silently
- **Idempotency** — Retried requests must not create duplicate records

**Managed integration platforms** handle these challenges:
- **Fivetran** — Fully managed, 300+ pre-built connectors, automated schema detection
- **Airbyte** — Open-source, 350+ connectors, self-hosted or cloud
- **Stitch** — Simple, affordable, ETL-focused connector platform

### 14.2.5 File-Based Integration

The oldest and sometimes still the most practical pattern — exchanging data as files.

**Common file formats and their trade-offs:**

| Format | Type | Schema | Compression | Splittable | Best For |
|---|---|---|---|---|---|
| **CSV** | Row-based, text | None (implicit) | External only | Yes | Simple interchange, human-readable |
| **JSON** | Row-based, text | Self-describing | External only | Line-delimited only | API responses, semi-structured data |
| **Parquet** | Columnar, binary | Embedded | Built-in (Snappy, Gzip, Zstd) | Yes | Analytics queries, data lake storage |
| **Avro** | Row-based, binary | Embedded (JSON) | Built-in (Snappy, Deflate) | Yes | Streaming, schema evolution, Kafka |
| **ORC** | Columnar, binary | Embedded | Built-in (Zlib, Snappy, LZO) | Yes | Hive ecosystem, heavy reads |

**Why Parquet dominates the modern data lake:**
- **Columnar storage** — Read only the columns you need (massive I/O savings for analytics)
- **Predicate pushdown** — Skip entire row groups that don't match filters
- **Built-in statistics** — Min/max per column per row group enable intelligent pruning
- **Rich type system** — Nested data, maps, arrays natively supported
- **Compression** — Columnar data compresses 3–10x better than row formats
- **Ecosystem support** — Native in Spark, Hive, Presto, Snowflake, BigQuery, Foundry

**Avro's niche — schema evolution:**
- Schema is embedded in every file (self-describing)
- Supports forward and backward compatible schema changes
- Default serialization format for Kafka (via Confluent Schema Registry)
- Better for write-heavy workloads (row-based = faster writes)

> **Rule of Thumb:**
> - **Parquet** for analytics and storage (read-optimized)
> - **Avro** for streaming and data exchange (write-optimized, schema evolution)
> - **CSV/JSON** for human interchange and simple integrations
> - **ORC** when deep in the Hive/Hadoop ecosystem

---

## 14.3 Pipeline Orchestration

Orchestration is the *control plane* of data engineering — deciding *what* runs, *when*, and *in what order*.

### 14.3.1 Core Concepts

**DAG (Directed Acyclic Graph):**
The fundamental abstraction. Each node is a task; edges define dependencies. "Directed" means flow goes one way. "Acyclic" means no circular dependencies.

```
        ┌──────────┐
        │ Extract  │
        │ Orders   │
        └────┬─────┘
             │
     ┌───────┴───────┐
     ▼               ▼
┌──────────┐   ┌──────────┐
│ Clean    │   │ Extract  │
│ Orders   │   │ Products │
└────┬─────┘   └────┬─────┘
     │               │
     └───────┬───────┘
             ▼
       ┌──────────┐
       │  Join &  │
       │ Aggregate│
       └────┬─────┘
            │
            ▼
       ┌──────────┐
       │  Load    │
       │  Marts   │
       └──────────┘
```

**Idempotency** — The *critical* property:
- Running a task multiple times with the same input produces the same output
- Essential for retry logic: if a task fails mid-way, you can safely re-run it
- Implementation patterns:
  - **UPSERT** (INSERT ... ON CONFLICT UPDATE) instead of INSERT
  - **DELETE-then-INSERT** for full partition refreshes
  - **Deterministic output paths** based on execution date

**Retry and backfill strategies:**
- **Automatic retries** — Configure tasks to retry N times with exponential backoff
- **Backfilling** — Re-run pipelines for historical date ranges when logic changes
- **Dead letter queues** — Route failed records to a separate queue for manual review

### 14.3.2 Apache Airflow

The industry standard — created at Airbnb (2014), now an Apache top-level project.

**Architecture:**
```
┌────────────────────────────────────────────────┐
│                  Airflow                       │
│                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐│
│  │ Scheduler│  │ Web UI   │  │  Metadata DB ││
│  │ (parses  │  │ (DAG     │  │  (PostgreSQL)││
│  │  DAGs)   │  │  viz)    │  │              ││
│  └────┬─────┘  └──────────┘  └──────────────┘│
│       │                                       │
│  ┌────▼─────────────────────────────────┐     │
│  │        Worker Pool (Celery/K8s)     │     │
│  │  ┌──────┐ ┌──────┐ ┌──────┐        │     │
│  │  │Task 1│ │Task 2│ │Task 3│  ...   │     │
│  │  └──────┘ └──────┘ └──────┘        │     │
│  └──────────────────────────────────────┘     │
└────────────────────────────────────────────────┘
```

**Key concepts:**
- **DAGs** defined in Python — full programmability
- **Operators** — pre-built task types (BashOperator, PythonOperator, SQLOperator, S3 sensor)
- **Sensors** — Tasks that wait for external conditions (file arrival, API readiness)
- **XComs** — Cross-communication between tasks (pass small data between steps)
- **Connections/Hooks** — Managed credentials for external systems
- **Managed offerings** — MWAA (AWS), Cloud Composer (GCP), Astronomer (multi-cloud)

**Strengths:** Massive ecosystem, battle-tested at scale, Python-native, rich UI
**Weaknesses:** DAGs are pipeline-oriented not asset-oriented, complex setup, scheduler can bottleneck at very high scale

### 14.3.3 Dagster

The modern "asset-oriented" alternative — focuses on *what* data assets exist rather than *when* tasks run.

**Key differentiator: Software-Defined Assets (SDAs)**
```python
@asset
def raw_orders(context):
    """Extract orders from source system."""
    return extract_from_api("orders")

@asset
def clean_orders(raw_orders):
    """Clean and validate order data."""
    return raw_orders.dropna().drop_duplicates()

@asset
def order_metrics(clean_orders):
    """Calculate key order metrics."""
    return clean_orders.groupby("date").agg({"amount": "sum"})
```

Dependencies are inferred from function signatures — no explicit DAG wiring needed.

**Key concepts:**
- **Assets** — The central abstraction (a dataset, model, or report)
- **Resources** — External services (databases, APIs) injected as dependencies
- **IO Managers** — Handle reading/writing assets to storage
- **Partitions** — First-class support for time-partitioned data
- **Dagit UI** — Rich web interface with asset lineage visualization
- **Type checking** — Built-in data validation through type annotations

**Strengths:** Strong local development experience, integrated testing, asset lineage, better developer ergonomics
**Weaknesses:** Younger ecosystem, smaller community than Airflow

### 14.3.4 Prefect

The "workflow-as-code" approach — minimal boilerplate, maximum Pythonic simplicity.

```python
from prefect import flow, task

@task
def extract_orders():
    return fetch_from_api("orders")

@task
def transform_orders(raw):
    return clean_and_validate(raw)

@flow(name="order_pipeline")
def order_pipeline():
    raw = extract_orders()
    cleaned = transform_orders(raw)
    load_to_warehouse(cleaned)
```

**Key concepts:**
- **Flows** — Python functions decorated with `@flow`
- **Tasks** — Python functions decorated with `@task`
- **Deployments** — Define where, when, and how flows run
- **Automations** — Event-driven triggers and notifications
- **Prefect Cloud** — Managed orchestration with observability

**Strengths:** Simplest API, natural Python, hybrid execution (local/cloud), excellent error handling
**Weaknesses:** Less enterprise adoption than Airflow, cloud-first pricing model

### 14.3.5 Orchestration Tool Comparison

| Feature | Airflow | Dagster | Prefect |
|---|---|---|---|
| **Paradigm** | Task/pipeline-oriented | Asset-oriented | Workflow-as-code |
| **Abstraction** | DAGs, Operators | Assets, Resources | Flows, Tasks |
| **Language** | Python | Python | Python |
| **Testing** | External test frameworks | Built-in testing | Built-in testing |
| **Local dev** | Requires full setup | Excellent (dagit) | Good (local server) |
| **UI** | Functional, DAG-focused | Asset lineage, modern | Clean, observability-focused |
| **Maturity** | High (10+ years) | Medium (5+ years) | Medium (5+ years) |
| **Community** | Largest | Growing | Growing |
| **Best for** | Complex enterprise pipelines | Data asset management | Python-first teams |

---

## 14.4 Data Transformation — dbt and Spark

### 14.4.1 dbt (data build tool)

The cornerstone of the modern data stack's "T" in ELT — transforms data *inside* the warehouse using SQL.

**Core philosophy:** Apply software engineering best practices (version control, testing, documentation, modularity) to SQL-based data transformation.

```
┌───────────────────────────────────────────────────┐
│                  dbt Project                      │
│                                                    │
│  models/                                          │
│  ├── staging/          ← Light cleaning           │
│  │   ├── stg_orders.sql                           │
│  │   └── stg_customers.sql                        │
│  ├── intermediate/     ← Business logic           │
│  │   └── int_order_enriched.sql                   │
│  └── marts/            ← Final business tables    │
│      ├── fct_orders.sql                           │
│      └── dim_customers.sql                        │
│                                                    │
│  tests/                ← Data quality checks      │
│  └── assert_positive_revenue.sql                  │
│                                                    │
│  schema.yml            ← Column docs + tests      │
│  sources.yml           ← Raw table declarations   │
└───────────────────────────────────────────────────┘
```

**Key concepts:**

**Models** — SQL SELECT statements that dbt materializes as tables or views:
```sql
-- models/staging/stg_orders.sql
SELECT
    id AS order_id,
    user_id AS customer_id,
    CAST(amount AS DECIMAL(10,2)) AS order_amount,
    created_at AS ordered_at
FROM {{ source('raw', 'orders') }}
WHERE is_test = false
```

**The `ref()` function** — Creates dependency relationships:
```sql
-- models/marts/fct_orders.sql
SELECT
    o.order_id,
    o.order_amount,
    c.customer_name,
    c.customer_segment
FROM {{ ref('stg_orders') }} o
JOIN {{ ref('stg_customers') }} c
    ON o.customer_id = c.customer_id
```

dbt automatically builds a DAG from `ref()` and `source()` calls, ensuring models execute in the correct order.

**Testing** — Built-in data validation:
```yaml
# schema.yml
models:
  - name: fct_orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: order_amount
        tests:
          - not_null
          - positive_value  # custom test
```

**Documentation** — Auto-generated lineage and data dictionary:
- Column descriptions live alongside model definitions
- `dbt docs generate` creates an interactive website
- Full lineage graph shows how data flows from source to marts

**Materializations:**
- **View** — Lightweight, always fresh, no storage cost (default)
- **Table** — Full table rebuild on each run (fast queries, slower builds)
- **Incremental** — Append only new/changed rows (efficient for large tables)
- **Ephemeral** — CTE injected into downstream models (no database object created)

### 14.4.2 Apache Spark

The distributed computing engine for transformations that exceed SQL's capabilities or warehouse scale.

**When Spark over dbt:**
- Transformations on **raw files** (Parquet, JSON, CSV) before warehouse loading
- **Complex logic** requiring Python/Scala (ML feature engineering, graph processing)
- **Multi-terabyte** datasets that strain even cloud warehouse compute
- **Streaming transformations** (Spark Structured Streaming)

**Spark's role in the modern stack:**
```
Raw Files (S3/GCS/ADLS)
         │
    ┌────▼────┐
    │  Spark  │ ← Heavy lifting: parsing, deduplication,
    │         │   complex joins, ML feature engineering
    └────┬────┘
         │
    Clean Parquet files
         │
    ┌────▼────────────┐
    │  Cloud Warehouse│ ← dbt transforms: business logic,
    │  + dbt          │   aggregations, dimensional modeling
    └─────────────────┘
```

Many modern stacks use **Spark AND dbt** together: Spark for raw-to-clean transformations, dbt for clean-to-business transformations.

---

## 14.5 Data Quality — Frameworks and Best Practices

Bad data is more dangerous than no data — it creates *false confidence* in wrong decisions.

### 14.5.1 The Six Dimensions of Data Quality

| Dimension | Definition | Example Check |
|---|---|---|
| **Completeness** | All required values present | NULL rate < 1% for critical fields |
| **Accuracy** | Values match real-world truth | ZIP codes validate against postal database |
| **Consistency** | Same value across systems | Customer name identical in CRM and billing |
| **Validity** | Values conform to defined rules | Email contains @ sign, dates are valid |
| **Uniqueness** | No unintended duplicates | Primary keys are truly unique |
| **Freshness** | Data is current enough for use | Table updated within last 4 hours |

### 14.5.2 Data Quality Tools

**Great Expectations (GX):**
- Python-based validation framework
- "Expectations" as code — programmatic assertions about data:
  ```python
  expect_column_values_to_not_be_null("customer_id")
  expect_column_values_to_be_between("age", min_value=0, max_value=150)
  expect_column_values_to_match_regex("email", r"^[^@]+@[^@]+\.[^@]+$")
  ```
- **Data Docs** — auto-generated validation reports (human-readable HTML)
- Integrates with Pandas, Spark, SQL databases
- Steeper learning curve but highly flexible and powerful

**Soda Core:**
- YAML-based check definitions (SodaCL language):
  ```yaml
  checks for orders:
    - row_count > 0
    - missing(customer_id) = 0
    - duplicate_count(order_id) = 0
    - freshness(created_at) < 4h
    - avg(amount) between 10 and 1000
  ```
- More accessible than GX for non-Python teams
- Strong anomaly detection and observability dashboard
- Checks translate directly to SQL queries

### 14.5.3 Data Contracts

A **data contract** is a formal agreement between data producers and consumers that defines:

```
┌──────────────────────────────────────────────────┐
│              DATA CONTRACT                        │
│                                                    │
│  Schema:     Order must have (order_id INT,       │
│              customer_id INT, amount DECIMAL,      │
│              created_at TIMESTAMP)                 │
│                                                    │
│  Quality:    NULL rate for customer_id < 0.1%     │
│              Amount > 0                            │
│                                                    │
│  Freshness:  Updated within 2 hours of source     │
│                                                    │
│  SLA:        99.5% availability                   │
│                                                    │
│  Owner:      orders-team@company.com              │
│  Consumers:  analytics, fraud-detection           │
└──────────────────────────────────────────────────┘
```

**Why contracts matter:**
- Shift data quality responsibility **left** to producers (not consumers)
- Create a "circuit breaker" — pipelines halt when contracts are violated
- Enable autonomous teams to evolve schemas without breaking downstream
- Formalize what was previously tribal knowledge

### 14.5.4 Observability vs. Validation

| Approach | When | Focus | Tools |
|---|---|---|---|
| **Validation** | During pipeline execution | "Does this batch pass checks?" | Great Expectations, Soda, dbt tests |
| **Observability** | Continuous monitoring | "Is the overall system healthy?" | Monte Carlo, Bigeye, Soda Cloud |

**Observability monitors for:**
- **Schema drift** — unexpected column additions/removals/type changes
- **Volume anomalies** — sudden spikes or drops in row counts
- **Freshness delays** — tables not updating on schedule
- **Distribution shifts** — statistical properties of data changing unexpectedly

---

## 14.6 Common Pipeline Challenges

These are the real-world problems that break pipelines and erode trust in data systems. Understanding them is essential for a DS who will help customers debug integration issues.

### 14.6.1 Schema Evolution

Source systems change — columns are added, renamed, retyped. These changes propagate downstream and can silently break pipelines.

**The problem spectrum:**
```
Backward Compatible (Safe)          Breaking Changes (Dangerous)
──────────────────────────────────────────────────────────────
Adding a new column                 Removing a column
Adding a new table                  Renaming a column
Widening a column (INT → BIGINT)   Narrowing a type (VARCHAR → INT)
                                    Changing column semantics
```

**Mitigation strategies:**
- **Schema registries** (Confluent Schema Registry) — enforce compatibility rules for Avro/Protobuf schemas
- **Schema validation gates** — reject ingestion if schema doesn't match the contract
- **Additive-only policies** — allow adding columns but never removing or renaming
- **Versioned APIs** — maintain v1, v2, v3 simultaneously during migration periods

**Key statistic:** 48% of data engineers cite schema issues as the primary cause of pipeline failures.

### 14.6.2 Late-Arriving Data

Data that arrives *after* its expected processing window — common in distributed systems with network delays or offline devices.

**Example:** A mobile app collects GPS coordinates offline. When the phone reconnects 3 hours later, those events arrive with timestamps from 3 hours ago — but the hourly aggregation window has already closed.

**Handling strategies:**
- **Event time vs. processing time** — Always use event timestamps, not arrival timestamps, for windowed computations
- **Watermarks** — Define how long to wait for late data before closing a window
- **Grace periods** — Keep windows open slightly beyond the nominal close time
- **Side outputs** — Route late events to a separate queue for reprocessing
- **Recomputation** — Periodically re-run past windows to incorporate late arrivals

### 14.6.3 Deduplication

Duplicate records inflate metrics, waste storage, and corrupt analytics.

**Common sources of duplicates:**
- Network retries (HTTP request sent twice due to timeout)
- CDC captures during snapshot + streaming overlap
- Multiple upstream systems reporting the same event
- Failed-and-retried pipeline runs

**Deduplication strategies:**
- **Natural keys** — Use business-meaningful unique identifiers (order_id, transaction_id)
- **Deterministic dedup** — `SELECT DISTINCT` or `ROW_NUMBER() OVER (PARTITION BY key ORDER BY timestamp DESC)`
- **Idempotent writes** — UPSERT/MERGE operations that handle duplicates at write time
- **Probabilistic dedup** — Bloom filters for approximate duplicate detection at streaming scale

### 14.6.4 Delivery Semantics

How does the system guarantee message delivery during failures?

| Semantic | Guarantee | Trade-off | Use Case |
|---|---|---|---|
| **At-most-once** | Message may be lost, never duplicated | Fastest, simplest | Metrics that tolerate gaps (ad impressions) |
| **At-least-once** | Message always delivered, may be duplicated | Must handle duplicates downstream | Most data pipelines (with dedup) |
| **Exactly-once** | Message delivered exactly once, no loss or duplication | Complex, higher latency | Financial transactions, inventory |

**Achieving exactly-once in practice:**
Most systems implement "effectively exactly-once" by combining at-least-once delivery with idempotent processing at the consumer:
```
At-Least-Once Delivery + Idempotent Consumer = Effectively Exactly-Once
```

Apache Kafka achieved this through idempotent producers (unique sequence numbers) and transactional consumers.

### 14.6.5 The "Dirty Dozen" — Everyday Data Landmines

| Issue | Description | Best Practice |
|---|---|---|
| **Timezones** | Timestamps without timezone info are ambiguous | Normalize everything to UTC; convert to local only at display time |
| **Character encoding** | UTF-8 vs. Latin-1 vs. Windows-1252 | Enforce UTF-8 everywhere; validate at ingestion |
| **NULL semantics** | NULL ≠ 0 ≠ "" ≠ "null" (the string) | Define explicit NULL handling rules per field |
| **Date formats** | Is 03/04/2025 March 4th or April 3rd? | Use ISO 8601 (YYYY-MM-DD) everywhere |
| **Floating point** | 0.1 + 0.2 ≠ 0.3 in binary | Use DECIMAL for financial data, never FLOAT |
| **Case sensitivity** | "John Smith" vs. "john smith" vs. "JOHN SMITH" | Normalize to lowercase for matching, preserve original for display |
| **Trailing whitespace** | " value " vs. "value" | TRIM() all string fields at ingestion |
| **Boolean ambiguity** | True/False, 1/0, Y/N, Yes/No, T/F | Standardize to a single format at the integration layer |
| **Empty arrays vs. NULL** | `[]` vs. `null` vs. missing key | Define explicit semantics in the data contract |
| **Surrogate vs. natural keys** | Auto-increment IDs vs. business keys | Use business keys for dedup, surrogates for joins |
| **PII leakage** | SSN, email in unexpected columns | Scan with automated PII detection at ingestion |
| **Units mismatch** | Meters vs. feet, USD vs. EUR | Document units in metadata; convert at ingestion |

---

## 14.7 Palantir Foundry — Data Integration in Practice

Understanding how Foundry handles data integration gives you a concrete, real-world platform to reference during the interview.

### 14.7.1 Foundry's Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     PALANTIR FOUNDRY                         │
│                                                               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │    Data      │    │   Pipeline   │    │   Ontology   │   │
│  │  Connection  │───▶│   Builder /  │───▶│   (Object    │   │
│  │  (Ingest)    │    │  Code Workbk │    │    Types)    │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│        │                    │                    │            │
│   Raw Datasets         Transforms          Object Links     │
│   (versioned)          (Spark/SQL)         & Properties      │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Infrastructure: Health Checks, Branching, Scheduling   │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### 14.7.2 Data Connection

Foundry's ingestion layer — connects to customer data wherever it lives.

**Capabilities:**
- Pre-built connectors for databases, APIs, SaaS platforms, file systems
- **Sources** — configured connections to external systems
- **Workers** — compute infrastructure that runs sync processes (Foundry-hosted or customer-hosted agent workers)
- Supports **batch**, **streaming**, **CDC**, and **virtual table** sync modes
- Each sync creates a versioned dataset transaction with full lineage

**Why this matters for a DS:** When you're scoping a deployment, you need to identify all customer data sources and determine which connector type and sync mode fits each one. This is often the first technical conversation you'll have with a customer's data team.

### 14.7.3 Pipeline Builder and Code Workbook

Foundry's transformation layer — where raw data becomes clean, modeled datasets.

**Pipeline Builder:**
- Primary application for building data pipelines
- Supports both **code-based** (Python, SQL) and **no-code** (visual) authoring
- Runs on **Spark and Flink** under the hood
- Enables collaboration between technical engineers and business analysts

**Code Workbook:**
- Interactive environment for importing, exploring, and transforming datasets
- **Graph interface** — visual DAG showing data flow between transform nodes
- Three transform types:
  - **Code transforms** — Write logic in Python, SQL, or R
  - **Template transforms** — Reusable, parameterized code blocks with form-based interfaces
  - **Manual entry transforms** — Direct data input for small reference tables
- Results saved as **derived datasets** — available across Foundry

### 14.7.4 Branching and Versioning

Foundry applies Git-like version control to *data pipelines*, not just code.

```
Main Branch (Production)
    │
    ├── Branch: "add-new-customer-feed"
    │     │
    │     ├── Modify pipeline code
    │     ├── Test with sample data
    │     ├── Create Proposal (Pull Request)
    │     └── Review → Approve → Merge
    │
    └── Production continues uninterrupted
```

**Key features:**
- **Isolated environments** — Test pipeline changes without affecting production
- **Proposals** — Analogous to pull requests; review and approve before merging
- **Multi-asset branching** — Branch across Pipeline Builder, Ontology, and Workshop simultaneously
- **Controlled deployment** — Production data flows are never interrupted by development work

### 14.7.5 Incremental Pipelines

Foundry's approach to processing efficiency — only compute what's new.

**How it works:**
- Datasets evolve through **transactions**: `APPEND` (add new data) or `SNAPSHOT` (replace all data)
- Incremental pipelines read only new `APPEND` transactions since the last run
- Must be resilient to occasional `SNAPSHOT` transactions (full recomputation)

**Benefits:**
- Dramatically reduced compute costs for frequently updated datasets
- Lower end-to-end latency (minutes vs. hours)
- More efficient resource utilization

### 14.7.6 Health Checks

Foundry's built-in data quality and pipeline monitoring.

**Types of health checks:**
| Check Type | What It Monitors |
|---|---|
| **Job-level** | Did the individual transformation job succeed? |
| **Build-level** | Did the entire build process complete? |
| **Schedule-level** | Is the pipeline running on schedule? |
| **Freshness** | How current is the data? (time since last update, data timestamp freshness, sync freshness) |

**Alerting:**
- Health checks trigger **automated alerts** when thresholds are breached
- Can create **issues** automatically for triage
- **Lineage view** shows health status across the entire pipeline graph — enabling you to trace quality problems back to their root cause

---

## 14.8 Putting It All Together — Choosing the Right Architecture

### 14.8.1 Decision Framework

When a customer asks "how should we get our data into Foundry?", use this decision tree:

```
What's the data source?
│
├── Database (OLTP)?
│   ├── Need real-time? ──▶ CDC (Debezium → Kafka → Foundry)
│   └── Daily is fine? ──▶ Batch extract (JDBC connector)
│
├── SaaS Platform (Salesforce, SAP)?
│   └── API-based integration (managed connector)
│
├── Files (SFTP, S3, Azure Blob)?
│   ├── Structured (CSV)? ──▶ File-based, schema mapping
│   └── Semi-structured (JSON, Parquet)? ──▶ File-based, schema inference
│
├── Streaming source (IoT, clickstream)?
│   └── Kafka/Kinesis → Foundry streaming sync
│
└── Legacy system (mainframe, proprietary)?
    └── Custom connector or file export + batch load
```

### 14.8.2 Reference Architecture for a Typical Foundry Deployment

```
┌─────────────────────────────────────────────────────────┐
│                    CUSTOMER SYSTEMS                      │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────────┐ │
│  │ ERP  │ │ CRM  │ │ IoT  │ │ DWH  │ │ Flat Files   │ │
│  │(SAP) │ │(SFCM)│ │Kafka │ │(Snow)│ │ (SFTP/S3)    │ │
│  └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └──────┬───────┘ │
│     │        │        │        │             │          │
└─────┼────────┼────────┼────────┼─────────────┼──────────┘
      │        │        │        │             │
      ▼        ▼        ▼        ▼             ▼
┌─────────────────────────────────────────────────────────┐
│              FOUNDRY DATA CONNECTION                     │
│   JDBC    REST API  Streaming   JDBC      File Sync    │
│  Connector Connector  Connector  Connector  Connector   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              RAW DATASETS (Versioned)                    │
│   raw_sap_orders  raw_sfdc_accounts  raw_iot_telemetry  │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│         PIPELINE BUILDER / CODE WORKBOOK                 │
│   Clean → Standardize → Join → Enrich → Aggregate      │
│   (Health checks at each stage)                          │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                     ONTOLOGY                             │
│   Object Types: Order, Customer, Equipment, Alert       │
│   Links: Customer → Orders, Equipment → Alerts          │
│   Actions: Approve Order, Dispatch Technician            │
└─────────────────────────────────────────────────────────┘
```

---

## 14.9 Chapter Summary — Quick Reference

### ETL vs. ELT
- **ETL**: Transform before loading; compliance, small data, legacy systems
- **ELT**: Load then transform; cloud-native, flexible, scalable, preserves raw data
- **Foundry uses ELT**: Raw → Pipeline Builder transforms → Ontology

### Integration Patterns
- **Batch**: Scheduled bulk loads; sufficient when hourly/daily freshness is acceptable
- **Streaming**: Kafka/Kinesis/Pub/Sub; sub-second freshness for operational use cases
- **CDC**: Capture only changes; Debezium (log-based); efficient for database sync
- **API**: REST/GraphQL; standard for SaaS; managed by Fivetran/Airbyte
- **File**: CSV/JSON/Parquet/Avro; Parquet for analytics, Avro for streaming

### Orchestration
- **Airflow**: Industry standard, DAG-based, Python, massive ecosystem
- **Dagster**: Asset-oriented, better dev experience, built-in testing
- **Prefect**: Simplest API, Pythonic, hybrid execution model

### Transformation
- **dbt**: SQL transformations inside the warehouse; models, tests, docs, lineage
- **Spark**: Distributed compute for heavy processing; complements dbt for raw data

### Data Quality
- **Six dimensions**: Completeness, Accuracy, Consistency, Validity, Uniqueness, Freshness
- **Tools**: Great Expectations (Python), Soda Core (YAML), dbt tests (SQL)
- **Data contracts**: Formal agreements on schema, quality rules, SLAs between teams
- **Observability**: Continuous monitoring for schema drift, volume anomalies, freshness

### Common Challenges
- **Schema evolution**: Additive-only policies, schema registries, versioned APIs
- **Late data**: Event time processing, watermarks, grace periods
- **Deduplication**: Natural keys, UPSERT, ROW_NUMBER partitioning
- **Delivery semantics**: At-least-once + idempotent consumer = effectively exactly-once

### Palantir Foundry Pipeline Stack
- **Data Connection**: Managed connectors (batch, streaming, CDC, API, file)
- **Pipeline Builder / Code Workbook**: Spark/Flink transforms with visual DAG
- **Branching**: Git-like version control for pipelines and Ontology
- **Incremental pipelines**: Process only new transactions (APPEND vs. SNAPSHOT)
- **Health checks**: Job, build, schedule, and freshness monitoring with alerting

---

## Interview Cheat Sheet

| If They Ask... | Key Points to Hit |
|---|---|
| "How does data get into Foundry?" | Data Connection → Raw datasets (versioned) → Pipeline Builder transforms → Ontology. Supports batch, streaming, CDC, API, file sync modes. |
| "ETL or ELT?" | ELT is the modern default (transform using warehouse compute). ETL still right for compliance masking or bandwidth-constrained environments. Foundry follows ELT pattern. |
| "How do you ensure data quality?" | Six dimensions (completeness, accuracy, etc.), automated validation (dbt tests, Great Expectations, Soda), data contracts between producer/consumer teams, continuous observability for drift and anomalies. Foundry has built-in health checks. |
| "What orchestrator would you choose?" | Airflow for complex enterprise pipelines with existing ecosystem. Dagster for asset-oriented development with strong testing. Prefect for Python-first simplicity. Foundry has its own built-in scheduling and dependency management. |
| "How do you handle schema changes?" | Schema registries for streaming (Avro/Protobuf), additive-only policies, schema validation gates at ingestion, versioned APIs for breaking changes, data contracts that define expectations explicitly. |
| "Batch vs. streaming?" | Start with batch unless the business case *requires* real-time (fraud, IoT, live operations). Batch is simpler, cheaper, and easier to debug. Streaming adds complexity — use it when latency requirements demand it. |
| "What file format for a data lake?" | Parquet for analytics (columnar, compressed, predicate pushdown). Avro for streaming/CDC (schema evolution, row-based writes). CSV/JSON only for human interchange or simple integrations. |

---

*Next Chapter: Chapter 15 — System Design Fundamentals — How to Design a Software System From Scratch →*
