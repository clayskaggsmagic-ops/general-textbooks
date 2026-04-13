# Appendix G: Wargame Data Architecture — Exhaustive Follow-Up Q&A

> **How to use this**: This is your airplane study guide. Each question has a **Quick Answer** (what you'd actually say in 30-60 seconds) and a **Deep Dive** (the technical backstory so you can handle "tell me more" or "why?"). Read the Quick Answers until they feel automatic. Read the Deep Dives so you never get caught off guard.

---

## Q1: "Why SQLite and not Postgres from the start?"

### Quick Answer
"SQLite was the right choice for the development phase. It's zero-config — no server process, no connection strings, no Docker compose file. The entire database is one file I can copy, email, or `git add`. For a research prototype where I'm iterating on the schema daily, that speed of iteration matters more than concurrent write throughput. I knew I'd outgrow it — and I designed the storage layer behind a clean interface so the swap would be mechanical, not architectural."

### Deep Dive
SQLite and PostgreSQL are fundamentally different products despite both speaking SQL:

| Property | SQLite | PostgreSQL |
|----------|--------|-----------|
| **Deployment** | Library linked into your process | Separate server process (client-server) |
| **Setup time** | Zero — `import sqlite3` | Install, configure `pg_hba.conf`, create roles, create database |
| **Concurrent writers** | One at a time (file-level lock) | Many (row-level MVCC locking) |
| **Database file** | Single `.db` file you can copy | Data directory managed by the server |
| **Good for** | Embedded apps, prototypes, edge/mobile | Production servers, multi-user apps |

The key insight: **SQLite is an application file format, not a database server.** The SQLite documentation literally says this. It competes with `fopen()`, not with PostgreSQL. For a solo developer building a demo on a laptop, it's the optimal choice because the bottleneck is schema iteration speed, not concurrent write throughput.

The migration path is also clean: SQLite and PostgreSQL share most SQL syntax. The `CREATE TABLE`, `INSERT`, `SELECT` statements are nearly identical. The main things that change:
- `TEXT` → `TEXT` (same)
- `INTEGER` → `SMALLINT` or `INTEGER` (more options)
- No `PRAGMA` commands in Postgres
- Add `UUID` type, `TIMESTAMPTZ`, `TEXT[]`, and `vector(768)` (none of which SQLite supports)

---

## Q2: "What's the difference between WAL mode and default journal mode?"

### Quick Answer
"Default journal mode uses a rollback journal — before changing a page, SQLite copies the original page to a separate journal file. If the write fails, it restores from the journal. The problem: while writing, the entire database file is locked. Readers block writers, writers block readers.

WAL mode flips this. Instead of copying old pages, new changes are *appended* to a Write-Ahead Log file. Readers just read the main database file — they see a consistent snapshot from before the write started. This means readers never block writers and writers never block readers. For my Monte Carlo setup with a hundred concurrent simulations all reading and writing, WAL was mandatory."

### Deep Dive
Here's what happens physically on disk:

**Default (Rollback Journal)**:
```
Step 1: Copy original page → .db-journal file (backup)
Step 2: Modify page in .db file
Step 3: On commit, delete .db-journal
Step 4: On crash, restore from .db-journal

LOCK: Entire .db file is locked during Step 1-3
→ No other process can read OR write
```

**WAL Mode**:
```
Step 1: Append new page to .db-wal file
Step 2: On commit, mark as committed in WAL
Step 3: Periodically, "checkpoint" the WAL back into .db

LOCK: Only the WAL file is locked during writes
→ Readers read from .db (old data, consistent snapshot)
→ One writer + many readers can coexist
```

The limitation: SQLite WAL mode still allows only **one writer at a time**. If two simulations try to write simultaneously, one waits. That's where my `busy_timeout=60000` (60 seconds) and exponential backoff retry come in — they handle the contention when a hundred simulations all try to write their state updates.

This is *good enough* for a demo, but it's why the production redesign uses document-per-simulation (zero shared state, zero contention).

---

## Q3: "Why did you use json.dumps() instead of a JSONB column?"

### Quick Answer
"SQLite doesn't have a JSONB type — it only has five types: NULL, INTEGER, REAL, TEXT, and BLOB. So `json.dumps()` into a TEXT column was the only option. If I'd been on PostgreSQL from the start, I'd have used JSONB for the semi-structured data. But even JSONB wouldn't have solved the core problem — deeply nested JSONB queries like `WHERE data->'rounds'->0->'actions'->0->>'actor' = 'CHN'` are slow without expression indexes, and you'd need a separate expression index for every path you might query. At that point, you're fighting the database. Normalization into separate tables is the cleaner solution."

### Deep Dive

**What's the difference between JSON and JSONB in PostgreSQL?**

| Feature | JSON (or TEXT with JSON) | JSONB |
|---------|-------------------------|-------|
| **Storage** | Stores the raw JSON string as-is | Parses and stores in a binary format |
| **Duplicate keys** | Preserved | Last value wins |
| **Key order** | Preserved | Not guaranteed |
| **Indexing** | None | GIN indexes on keys/values |
| **Query speed** | Must re-parse on every query | Pre-parsed, faster |
| **Write speed** | Faster (no parsing) | Slightly slower (must parse) |

Even with JSONB, the problem is **query ergonomics**. Compare:

```sql
-- JSONB (ugly, fragile, no type safety):
SELECT data->'rounds'->0->'actions'->0->>'actor' FROM simulations;

-- Normalized table (clean, indexed, type-safe):
SELECT actor_code FROM actions WHERE round_number = 1;
```

The JSONB approach also breaks down for aggregation:

```sql
-- JSONB: requires jsonb_array_elements() to "unwind" arrays
SELECT elem->>'actor' AS actor, COUNT(*)
FROM simulations,
     jsonb_array_elements(data->'rounds') AS round,
     jsonb_array_elements(round->'actions') AS elem
GROUP BY elem->>'actor';

-- Normalized: trivial
SELECT actor_code, COUNT(*) FROM actions GROUP BY actor_code;
```

**The takeaway**: JSONB is great for semi-structured data you rarely query (user preferences, config blobs). For data you need to aggregate, filter, and join — like actions and briefings — normalize it.

---

## Q4: "How would you handle schema migrations in production?"

### Quick Answer
"For the demo, I used an idempotent `try/except` pattern around `ALTER TABLE ADD COLUMN` — SQLite doesn't support `IF NOT EXISTS` for column additions, so you just catch the 'duplicate column' error and continue. It runs on every server startup and is safe to run N times. For production PostgreSQL, I'd use Alembic — Python's standard migration framework. It generates versioned migration scripts, tracks what's been applied in an `alembic_version` table, and supports both forward and rollback migrations."

### Deep Dive
My current migration pattern in `storage.py`:

```python
# Idempotent: safe to run on every startup
try:
    await db.execute("ALTER TABLE simulations ADD COLUMN peak_escalation INTEGER")
except Exception:
    pass  # Column already exists
```

**Why this works for a solo developer**: It's simple, stateless, and idempotent. There's no migration tracking table, no version numbers, no rollback scripts. You just add columns and ignore errors.

**Why this breaks in production**:
1. No rollback capability — if a migration breaks something, you can't undo it
2. No history — you can't tell what migrations have run
3. No data migrations — `ALTER TABLE ADD COLUMN` can't transform existing data (my backfill is a separate step)
4. No coordination — two servers starting simultaneously might race on the same migration

**Alembic** (the production solution) works like this:

```
alembic/
├── versions/
│   ├── 001_initial_schema.py        # CREATE TABLE statements
│   ├── 002_add_peak_escalation.py   # ALTER TABLE ADD COLUMN
│   └── 003_add_batch_support.py     # New table + FK
└── env.py                           # Database connection config
```

Each migration file has an `upgrade()` and `downgrade()` function:

```python
# 002_add_peak_escalation.py
def upgrade():
    op.add_column('simulations', sa.Column('peak_escalation', sa.SmallInteger))
    
    # Backfill from existing data
    op.execute("""
        UPDATE simulations SET peak_escalation = (
            SELECT MAX(escalation_after) FROM rounds 
            WHERE rounds.simulation_id = simulations.id
        )
    """)

def downgrade():
    op.drop_column('simulations', 'peak_escalation')
```

Alembic tracks applied migrations in a `alembic_version` table — it knows exactly which migrations have run and which haven't.

---

## Q5: "What's 3NF and why does it matter here?"

### Quick Answer
"Third Normal Form means every non-key column depends on the primary key, the whole primary key, and nothing but the primary key. In practice it means: no repeated groups of data inside a single row, and no columns that depend on other non-key columns. Why it matters here: my original design stored actions as a JSON array inside the rounds table — that's a repeating group, a 1NF violation. Normalizing actions into their own table means each action is independently queryable, indexable, and joinable. That's the difference between 'parse all JSON blobs in Python' and 'write one SQL query.'"

### Deep Dive
The three normal forms, applied to the wargame data:

**1NF (First Normal Form)**: No repeating groups. Every column contains atomic values.

```
VIOLATION: rounds.actions = '[{"actor":"USA",...},{"actor":"CHN",...}]'
           That's a repeating group — multiple actions stuffed into one cell.

FIX: Separate 'actions' table with one row per action.
```

**2NF (Second Normal Form)**: Every non-key column depends on the *entire* primary key, not just part of it.

```
VIOLATION (hypothetical): If actions had composite PK (round_id, actor_code)
           and included 'actor_name' — that depends only on actor_code,
           not the full key.

FIX: actor_name lives in agent_snapshots, looked up via JOIN.
     (I kept actor_name denormalized for query convenience — a deliberate 
     violation I'd acknowledge in an interview.)
```

**3NF (Third Normal Form)**: No transitive dependencies — non-key columns don't depend on other non-key columns.

```
VIOLATION (hypothetical): If 'country_name' was stored alongside 'country_code'
           in the actions table — country_name depends on country_code,
           not on the action's primary key.

FIX: country_name lives only in agent_snapshots.
```

**Why this matters for the wargame**: The current `final_state` blob is essentially **0NF** — zero normal form. It's a giant nested document with no structural constraints. Normalizing to 3NF gives you:
- **Queryability**: `SELECT * FROM actions WHERE action_type = 'MILITARY'`
- **Integrity**: Foreign keys prevent orphaned rows
- **Indexing**: B-tree indexes on any column
- **Aggregation**: `GROUP BY`, `COUNT()`, `AVG()` work natively
- **Storage efficiency**: No data duplication (in theory — see the `actor_name` trade-off)

---

## Q6: "How does your ETL pipeline handle failures mid-transform?"

### Quick Answer
"The entire ETL function runs inside a single database transaction. If any INSERT fails — malformed data, constraint violation, disk full — the whole transaction rolls back. The SQL store either has the complete simulation or nothing, never a partial state. The document store remains untouched either way, so I can fix the bug and re-run the ETL. The document is the source of truth; the SQL store is a derived view."

### Deep Dive
Here's the failure handling visually:

```
ETL Start
    │
    ▼
BEGIN TRANSACTION
    │
    ├── INSERT INTO simulations     ✅
    ├── INSERT INTO agent_snapshots ✅
    ├── INSERT INTO rounds          ✅
    ├── INSERT INTO actions         ✅
    ├── INSERT INTO briefings       ❌ FAILS (e.g., NULL in NOT NULL column)
    │
    ▼
ROLLBACK  ← All previous INSERTs are undone
    │
    ▼
SQL store: unchanged (as if ETL never ran)
Document store: unchanged (source of truth intact)
Error logged with simulation_id for retry
```

**What could go wrong?**

| Failure | Cause | Recovery |
|---------|-------|----------|
| Schema mismatch | Document has a new field the SQL schema doesn't know about | Add Alembic migration, re-run ETL |
| NULL constraint | Document is missing a required field | Fix the document (or make the column nullable) |
| Duplicate key | ETL ran twice for the same simulation | Use `ON CONFLICT DO NOTHING` or check before insert |
| Disk full | PostgreSQL partition is full | Ops problem — clear space, re-run |
| Network timeout | Connection to managed Postgres dropped | Retry with exponential backoff |

**The idempotency pattern**: To make ETL safe to retry, you can either:
1. Delete all rows for that `simulation_id` before re-inserting (clean slate)
2. Use `INSERT ... ON CONFLICT (simulation_id) DO UPDATE` (upsert)
3. Check if rows exist before starting (skip if already ETL'd)

Option 1 is simplest and avoids partial-update bugs:
```python
async with db.transaction():
    # Clean slate
    await db.execute("DELETE FROM simulations WHERE id = $1", sim_id)
    # CASCADE deletes all child rows (rounds, actions, etc.)
    # Then re-insert everything
    await etl_simulation(doc, db)
```

---

## Q7: "Why not just use a document database for everything?"

### Quick Answer
"Because analytical queries on document databases are painful. MongoDB's aggregation pipeline can do GROUP BY and JOIN-like operations, but the syntax is verbose and the performance on cross-document aggregation is significantly worse than a column-oriented SQL query on indexed tables. When my primary analytical question is 'across a hundred simulations, group actions by actor and type and count them,' that's a SQL-shaped question. Forcing it through a document database's aggregation framework is fighting the tool."

### Deep Dive
Let's compare the same query in both worlds:

**The question**: "How many military actions did each country take across all simulations in batch X?"

**PostgreSQL** (normalized):
```sql
SELECT a.actor_code, COUNT(*)
FROM actions a
JOIN simulations s ON a.simulation_id = s.id
WHERE s.batch_id = 'batch-uuid'
  AND a.action_type = 'MILITARY'
GROUP BY a.actor_code
ORDER BY COUNT(*) DESC;
```
*Execution*: Index scan on `batch_id`, index scan on `action_type`, hash aggregate. Sub-100ms on 100K rows.

**MongoDB** (aggregation pipeline):
```javascript
db.simulations.aggregate([
  { $match: { batch_id: "batch-uuid" } },
  { $unwind: "$rounds" },
  { $unwind: "$rounds.actions" },
  { $match: { "rounds.actions.action_type": "MILITARY" } },
  { $group: {
      _id: "$rounds.actions.actor",
      count: { $sum: 1 }
  }},
  { $sort: { count: -1 } }
]);
```
*Execution*: Full document scan of all matching simulations, double unwind (explodes nested arrays into temporary documents), then group. Much slower, much more memory.

**When documents ARE the right answer**: For the hot path — reading/writing a single simulation's complete state. You never need to JOIN simulation 47 with simulation 48 during execution. The access pattern is always "give me this one document" or "save this one document."

**When SQL IS the right answer**: For analytics — grouping, counting, joining, filtering across many simulations. The access pattern is always "scan many rows, aggregate, return a summary."

Use each tool where it's strongest. That's the dual-store thesis.

---

## Q8: "How would you scale this beyond a single SQLite file?"

### Quick Answer
"Three steps. First, replace the hot-path SQLite with per-simulation JSON files or MongoDB — this eliminates write contention entirely since each simulation is independent. Second, replace the cold-path SQLite with PostgreSQL — this gives you real concurrency, proper types, and pgvector for embeddings. Third, if the analytical workload outgrows PostgreSQL, I'd add a columnar store like ClickHouse or BigQuery for the heavy aggregation queries, keeping PostgreSQL for transactional reads."

### Deep Dive
The scaling bottlenecks and their solutions:

| Bottleneck | Current Limit | Solution |
|------------|--------------|----------|
| Write concurrency | SQLite: 1 writer | Document-per-sim: unlimited parallel writers |
| Read concurrency | SQLite WAL: many readers OK | PostgreSQL connection pooling (PgBouncer) |
| Storage size | SQLite: ~140TB theoretical, practical ~1TB | PostgreSQL: partitioned tables by batch_id |
| Query complexity | SQLite: no window functions, no CTEs (older versions) | PostgreSQL: full analytical SQL |
| Vector search | SQLite: not supported | PostgreSQL + pgvector: HNSW indexes |
| Geographic distribution | SQLite: single file, single machine | MongoDB Atlas: multi-region replication |

**Table partitioning** for very large analytical tables:

```sql
-- Partition actions by batch_id for fast pruning
CREATE TABLE actions (
    id UUID DEFAULT gen_random_uuid(),
    batch_id UUID NOT NULL,
    simulation_id UUID NOT NULL,
    -- ... other columns
) PARTITION BY HASH (batch_id);

-- Create 8 partitions
CREATE TABLE actions_p0 PARTITION OF actions FOR VALUES WITH (MODULUS 8, REMAINDER 0);
CREATE TABLE actions_p1 PARTITION OF actions FOR VALUES WITH (MODULUS 8, REMAINDER 1);
-- ... etc.
```

When you query `WHERE batch_id = X`, PostgreSQL only scans the one partition that contains that batch's data — the other 7 partitions are skipped entirely (partition pruning).

---

## Q9: "What indexes would you add and why?"

### Quick Answer
"I'd index every column that appears in a WHERE clause, JOIN condition, or ORDER BY. Specifically: `actions.actor_code` and `actions.action_type` for the action distribution queries, `simulations.batch_id` for batch filtering, `escalation_events.level_after` for the escalation distribution charts, and a GIN index on `simulations.participating_actors` for the array-contains operator. For the embedding table, an HNSW index on the vector column for approximate nearest-neighbor search."

### Deep Dive
Here's every index in the redesigned schema and why:

**B-tree indexes** (default, for equality and range queries):

| Index | Table.Column | Query It Accelerates |
|-------|-------------|---------------------|
| `idx_sim_batch` | `simulations.batch_id` | "Get all sims in this batch" — used in every batch view |
| `idx_sim_status` | `simulations.status` | "Get running sims" — polling for completion |
| `idx_action_actor` | `actions.actor_code` | "All actions by China" — action distribution |
| `idx_action_type` | `actions.action_type` | "All military actions" — type breakdown |
| `idx_action_target` | `actions.target_code` | "Who targeted Taiwan?" — target analysis |
| `idx_action_round` | `actions.round_id` | JOIN back to rounds table |
| `idx_action_sim` | `actions.simulation_id` | "All actions in this sim" — detail view |
| `idx_outcome_country` | `country_outcomes.country_code` | "How did China fare across all sims?" |
| `idx_outcome_label` | `country_outcomes.cumulative_label` | Count outcome distribution |
| `idx_esc_level` | `escalation_events.level_after` | "Sims that reached rung 20+" |
| `idx_mem_importance` | `memories.importance DESC` | "Top importance memories" |

**GIN index** (for array containment):

| Index | Table.Column | Query It Accelerates |
|-------|-------------|---------------------|
| `idx_sim_actors` | `simulations.participating_actors` | `WHERE participating_actors @> ARRAY['CHN']` |

**HNSW index** (for vector similarity):

| Index | Table.Column | Query It Accelerates |
|-------|-------------|---------------------|
| `idx_emb_vector` | `simulation_embeddings.embedding` | `ORDER BY embedding <=> query_vector LIMIT 10` |

**How to decide what to index**: If a column appears in a `WHERE`, `JOIN ON`, or `ORDER BY` clause of a query you actually run, it needs an index. If it only appears in `SELECT`, it doesn't. Every index costs write performance (the DB must update the index on every INSERT), so don't index columns you never filter on.

**Composite indexes** I'd consider adding if query patterns warranted:

```sql
-- If you frequently filter by both actor AND type:
CREATE INDEX idx_action_actor_type ON actions(actor_code, action_type);

-- If you frequently query outcomes by simulation + country:
CREATE INDEX idx_outcome_sim_country ON country_outcomes(simulation_id, country_code);
```

---

## Q10: "How would you add vector search / embeddings to this?"

### Quick Answer
"I'd use PostgreSQL's pgvector extension. Each embeddable text chunk — round summaries, action descriptions, high-importance memories — gets a 768-dimensional embedding from Gemini's text-embedding-004 model, stored in a `simulation_embeddings` table with a `vector(768)` column and an HNSW index. The killer feature is combining vector similarity with relational filters in one query: 'find actions most similar to naval deployment, but only from China in this batch.' You can't do that if your vectors are in a separate service."

### Deep Dive
**What's an embedding?** A vector (list of numbers) that captures the *meaning* of a piece of text. Similar texts have similar vectors. "China deployed warships" and "PLA Navy sent destroyers" would have vectors that are close together, even though they share zero words.

**Why pgvector over a dedicated vector database?**

| Feature | pgvector (in PostgreSQL) | Pinecone (dedicated) |
|---------|------------------------|---------------------|
| **Operational complexity** | Zero — it's an extension in your existing Postgres | Separate service to manage, monitor, pay for |
| **Combined queries** | Vector + WHERE clause in one query | Two queries: vector search in Pinecone, then filter in SQL |
| **Consistency** | Same transaction as relational data | Eventual consistency between two systems |
| **Scale** | Millions of vectors is fine | Billions of vectors (enterprise scale) |
| **Cost** | Free (open source extension) | Pay per vector per month |

**The HNSW index explained simply**: Without an index, finding the nearest vector requires comparing your query against *every* vector in the table — O(n). HNSW (Hierarchical Navigable Small World) builds a graph structure that lets you find approximate nearest neighbors in O(log n). It's "approximate" because it might miss the actual closest vector, but in practice it's >99% accurate and 1000x faster.

```sql
-- Create the index (one-time, takes a few seconds for <1M vectors)
CREATE INDEX idx_emb_vector ON simulation_embeddings 
    USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

-- Query: "Find round summaries most similar to 'nuclear escalation threat'"
SELECT chunk_text, 1 - (embedding <=> $1) AS similarity
FROM simulation_embeddings
WHERE chunk_type = 'summary'
  AND simulation_id IN (SELECT id FROM simulations WHERE batch_id = $2)
ORDER BY embedding <=> $1  -- <=> is the cosine distance operator
LIMIT 5;
```

The `<=>` operator computes cosine distance. The HNSW index makes this fast even with hundreds of thousands of embeddings.

---

## Q11: "What's the Kahn Ladder data and why doesn't it fit neatly into SQL?"

### Quick Answer
"The Kahn Escalation Ladder is a 44-rung model created by Herman Kahn at RAND in 1965 that classifies conflict intensity from 'ostensible crisis' at rung 1 to 'insensate war' at rung 44. It doesn't fit neatly into SQL because each rung has a name, a description, and a region ('subcrisis,' 'conventional war,' 'nuclear war'), and the simulation needs to reference this metadata at runtime. It's reference data — read-only, never changes. I'd store it as a lookup table in SQL, but during simulation I'd keep it as an in-memory dict or enum for fast access without a database round-trip."

### Deep Dive
The Kahn Ladder data looks like this:

```python
KAHN_LADDER = {
    1: {"name": "Ostensible Crisis", "region": "Subcrisis Maneuvering"},
    2: {"name": "Political/Economic Subcrisis", "region": "Subcrisis Maneuvering"},
    # ...
    19: {"name": "Justifiable Counterforce Attack", "region": "Nuclear War"},
    # ...
    44: {"name": "Spasm / Insensate War", "region": "Nuclear War"}
}
```

**The tension**: This data is used in two ways:
1. **Runtime** (hot path): The simulation engine reads `KAHN_LADDER[current_level]` to get the current rung's name and region for prompt injection. This happens hundreds of times per simulation. A database round-trip each time would be absurd.
2. **Analytical** (cold path): Escalation analysis might want to GROUP BY region: "In how many simulations did escalation reach the 'nuclear war' region?"

**Solution**: Keep it as both:
- **In code**: A Python dict/enum for O(1) runtime access
- **In SQL**: A `kahn_rungs` reference table for analytical JOINs

```sql
CREATE TABLE kahn_rungs (
    rung    SMALLINT PRIMARY KEY,
    name    TEXT NOT NULL,
    region  TEXT NOT NULL
);
-- Pre-populated with all 44 rungs

-- Now you can JOIN:
SELECT kr.region, COUNT(DISTINCT ee.simulation_id)
FROM escalation_events ee
JOIN kahn_rungs kr ON ee.level_after = kr.rung
GROUP BY kr.region;
-- → "47 sims reached 'Nuclear War' region"
```

This is a common pattern: **reference data** lives in both the application layer (for speed) and the database (for queryability).

---

## Q12: "How did you handle participating_actors being a list vs. a join table?"

### Quick Answer
"I used a PostgreSQL `TEXT[]` array column instead of a separate join table. `participating_actors` is a fixed-at-creation attribute — the list of countries in a simulation doesn't change after setup. A join table (`simulation_actors`) would add a table, a foreign key, and a JOIN on every query — complexity that's not justified for a static, small list (typically 3-6 countries). The `TEXT[]` column supports GIN indexing, so `WHERE participating_actors @> ARRAY['CHN']` is indexed and fast."

### Deep Dive
**A join table would look like**:

```sql
CREATE TABLE simulation_actors (
    simulation_id UUID REFERENCES simulations(id),
    country_code  VARCHAR(3),
    PRIMARY KEY (simulation_id, country_code)
);
```

**Trade-off comparison**:

| Factor | TEXT[] Array | Join Table |
|--------|------------|-----------|
| **Query: "sims with China"** | `WHERE actors @> ARRAY['CHN']` (GIN indexed) | `JOIN simulation_actors sa ON sa.simulation_id = s.id WHERE sa.country_code = 'CHN'` |
| **Query: "list actors"** | `SELECT participating_actors FROM simulations` (already there) | `SELECT country_code FROM simulation_actors WHERE simulation_id = ?` |
| **Insert** | One INSERT with array literal | One INSERT per actor (3-6 rows) |
| **Normalization** | Technically 1NF violation (multi-valued) | Fully normalized |
| **Cardinality** | 3-6 values max | 3-6 rows per simulation |
| **Schema changes** | None needed | Extra table, extra FK, extra index |

**When a join table IS the right choice**: If the relationship has its own attributes (e.g., "when did this country join the simulation?" or "what role does this country play?"), then the join table becomes a real entity with its own columns. For a simple "which countries are in this simulation?" question, the array is simpler.

**Palantir framing**: "This is actually a common trade-off in ontology design. In Foundry, you'd model `participating_actors` as a link type between a Simulation object and a Country object. The join table is essentially what Foundry uses under the hood. For my scale — up to 6 actors per simulation — the array column is a pragmatic shortcut that avoids the overhead of a separate link type."

---

## Q13: "Walk me through exactly what happens when a simulation round completes — what SQL runs?"

### Quick Answer
"In the current architecture, seven things happen: First, `save_round()` inserts a row into the `rounds` table with `json.dumps(actions)` for the actions list and `json.dumps(turn_order)` for turn ordering. Second, `update_simulation()` updates the `simulations` row with the new `rounds_completed` count. Third, the full `accumulated_state` is serialized via `json.dumps()` and written to the `final_state` column. Fourth, `peak_escalation` is updated if this round's escalation exceeded the previous peak. In the redesigned architecture, those JSON blobs are eliminated — instead you'd get individual INSERTs into the `actions`, `briefings`, `country_outcomes`, and `escalation_events` tables."

### Deep Dive
Here's the exact sequence in the **current architecture** (SQLite):

```
Round N analysis completes
         │
         ▼
① save_round():
   INSERT INTO rounds (id, simulation_id, round_number, 
                       escalation_before, escalation_after,
                       summary, actions, created_at)
   VALUES (uuid, sim_id, N, 
           before_level, after_level,
           "narrative text",
           json.dumps([{actor:USA,...},{actor:CHN,...}]),
           timestamp)

② update_round_analysis():
   UPDATE rounds 
   SET kahn_analysis = json.dumps({analysis_dict}),
       turn_order = json.dumps(["USA","CHN","TWN"])
   WHERE id = round_uuid

③ update_simulation():
   UPDATE simulations 
   SET final_state = json.dumps(accumulated_state),  -- 50-100KB blob
       rounds_completed = N,
       peak_escalation = MAX(peak, current_level)
   WHERE id = sim_id
```

And here's what the **redesigned architecture** would do instead:

```
Round N analysis completes
         │
         ▼
Document store: save_state(sim_id, updated_state_dict)
         │
         │ (At simulation completion, ETL runs:)
         ▼
① INSERT INTO rounds (id, simulation_id, round_number, ...)

② For each action in round.actions:
   INSERT INTO actions (round_id, actor_code, action_type, ...)

③ For each country in round.briefings:
   INSERT INTO briefings (round_id, country_code, intelligence, ...)

④ For each country in round.country_outcomes:
   INSERT INTO country_outcomes (round_id, country_code, round_impact, ...)

⑤ INSERT INTO escalation_events (simulation_id, round_number, 
                                  level_before, level_after, reasoning)
```

**The key difference**: Current architecture writes JSON blobs during the simulation (hot path). Redesigned architecture writes nothing to SQL during the simulation — all SQL writes happen later in the ETL (cold path). The simulation just updates its document.

---

## Q14: "If you had to add real-time dashboards, what would change?"

### Quick Answer
"The ETL-after-completion pattern has inherent latency — analytics only update when a simulation finishes. For real-time dashboards showing live escalation across all running simulations, I'd add a third component: a streaming layer. Each simulation emits events to a message queue — escalation changes, new actions, round completions — and the dashboard subscribes to that stream. I wouldn't change the cold-path SQL store; I'd add a lightweight real-time layer on top."

### Deep Dive
The architecture with real-time dashboards:

```
┌────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Document Store │──── │  Redis Pub/Sub│────▶│ Dashboard (WS)  │
│  (hot path)     │     │  or SSE       │     │ Live escalation  │
└─────────┬──────┘     └──────────────┘     │ Live actions     │
          │                                  │ Batch progress   │
          │ On completion                    └─────────────────┘
          ▼
┌──────────────┐     ┌─────────────────┐
│  ETL Pipeline │────▶│ PostgreSQL      │
│               │     │ (cold path)     │
└──────────────┘     └─────────────────┘
```

**Technology options for the streaming layer**:

| Option | Complexity | Best For |
|--------|-----------|----------|
| **Server-Sent Events (SSE)** | Low | Simple one-way streaming (server → browser). Already used in the current codebase for progress updates. |
| **WebSockets** | Medium | Bidirectional communication. Already implemented in the current codebase for simulation control. |
| **Redis Pub/Sub** | Medium | Multi-client fan-out. If many dashboards watch the same batch, Redis distributes the events efficiently. |
| **Kafka** | High | Durable event streaming with replay. Overkill for this scale, but appropriate if events need to be replayed or reprocessed. |

**What you'd emit per event**:

```json
{
  "event_type": "escalation_change",
  "simulation_id": "uuid",
  "batch_id": "uuid",
  "round_number": 3,
  "escalation_before": 8,
  "escalation_after": 12,
  "timestamp": "2026-03-15T10:35:00Z"
}
```

**What DOESN'T change**: The cold-path SQL store. Real-time dashboards show *current* state (streaming); analytical dashboards show *historical* state (SQL). They serve different purposes and don't interfere.

---

## Q15: "How would Palantir's Foundry/Ontology handle this differently?"

### Quick Answer
"Foundry would model this as an ontology with five object types — Simulation, Round, Action, Agent, and Outcome — connected by link types: a Simulation *has many* Rounds, a Round *has many* Actions, an Action *was taken by* an Agent. The raw simulation documents would land in Foundry as a raw dataset, then a series of transforms — essentially my ETL — would shred them into separate ontology-ready datasets. The ontology would give you Quiver or Workshop dashboards for free, with drag-and-drop filtering by actor, action type, or escalation level. It's conceptually identical to my dual-store design, just implemented in Foundry's platform rather than PostgreSQL."

### Deep Dive
Here's the mapping between my architecture and Foundry concepts:

| My Architecture | Foundry Equivalent |
|----------------|-------------------|
| Document Store (JSON files / MongoDB) | **Raw Dataset** — data in its source format |
| ETL Pipeline (Python shredding function) | **Transforms** (Code Repositories) — PySpark or Pandas transforms that clean and reshape data |
| Normalized SQL Tables | **Clean Datasets** — structured, validated, ready for the ontology |
| Foreign Keys | **Link Types** — explicit relationships between object types |
| SQL Tables (simulations, rounds, actions...) | **Object Types** in the Ontology |
| pgvector embeddings | **Semantic Search** in AIP (Foundry's AI layer) |
| SQL GROUP BY queries | **Quiver dashboards** or **Pipeline Builder** aggregations |
| The `SimulationStore` protocol | **Data Connection** — pluggable source system integration |

**The Ontology for Wargame data would look like**:

```
Object Types:
├── Simulation    (properties: crisis, status, escalation_final, batch_id)
├── Round         (properties: round_number, escalation_before, escalation_after, summary)
├── Action        (properties: actor, action_type, target, description)
├── Agent         (properties: country_code, leader_name, persona)
├── Outcome       (properties: round_impact, cumulative_impact, label)
└── Batch         (properties: crisis_summary, simulation_count)

Link Types:
├── Simulation → has_rounds → Round
├── Round → has_actions → Action
├── Round → has_outcomes → Outcome
├── Action → taken_by → Agent
├── Outcome → affects → Agent
└── Batch → contains → Simulation
```

**Why this matters for the interview**: By describing your architecture in Foundry terms, you show the interviewer that your thinking aligns with how Palantir builds systems. The dual-store pattern (raw source → transform → clean ontology) IS the Foundry workflow. You just implemented it with open-source tools instead of Foundry's platform.

**The interview line**: "My dual-store design follows the same philosophy as Foundry: capture data in the source system's native format, transform it asynchronously into a clean ontology, and serve analytics from the normalized layer. The only difference is I used PostgreSQL instead of Foundry datasets and foreign keys instead of link types. The architecture would port to Foundry almost directly."

---

## Bonus: Rapid-Fire Questions

These might come up as "quick, just curious" follow-ups:

**Q: "Why UUIDs instead of auto-incrementing integers?"**
> "Simulations are created in parallel by different async tasks. Auto-increment requires a centralized counter — that's a bottleneck. UUIDs can be generated independently with no coordination. The 16-byte storage overhead is acceptable."

**Q: "What's your biggest regret about the original design?"**
> "The `final_state` column. I used it as an escape hatch — dump everything to JSON and deal with it later. The problem is 'later' never came until I needed analytical queries. If I'd started with even a basic `actions` table from day one, every subsequent feature would have been easier."

**Q: "How do you test the ETL pipeline?"**
> "I'd create fixture documents — hand-crafted JSON documents with known values — run them through the ETL, then assert the SQL rows match. For example, a document with 2 rounds, 3 actions each, should produce exactly 6 rows in the `actions` table. I'd also test edge cases: empty rounds, missing optional fields, maximum escalation values."

**Q: "How much data are we talking about?"**
> "A single simulation: ~1 MB of structured data (5 agents, 10 rounds, ~50 actions, ~500 memories, ~50 briefings). A Monte Carlo batch of 100 simulations: ~100 MB. After a year of research with daily batches: ~36 GB. Well within PostgreSQL's comfort zone — no need for distributed databases."

**Q: "What's the most complex query your redesigned schema enables?"**
> "Cross-simulation escalation trajectory analysis: for each round in a batch, compute the median, p25, and p75 escalation levels across all 100 simulations. That's a window function over the `escalation_events` table grouped by round number — a 5-line SQL query that would require hundreds of lines of Python with the JSON blob approach."

```sql
SELECT 
    ee.round_number,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY ee.level_after) AS p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY ee.level_after) AS median,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY ee.level_after) AS p75
FROM escalation_events ee
JOIN simulations s ON ee.simulation_id = s.id
WHERE s.batch_id = $1
GROUP BY ee.round_number
ORDER BY ee.round_number;
```
