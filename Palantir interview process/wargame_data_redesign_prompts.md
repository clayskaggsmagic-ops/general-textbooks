# War Game Data Architecture Redesign — Prompt Sequence

> **Purpose:** These are sequential prompts to feed one at a time. Each builds on the previous. The goal is to prepare a Palantir-interview-quality answer to "Tell me about a time you dealt with data" — specifically about re-architecting a geopolitical simulation's data layer.

---

## Prompt 1: Forensic Audit of Current Data Architecture

```
Go through the wargamep2 project's data layer in extreme depth. I need to understand exactly:

1. **The In-Memory Layer (state.py)**: Walk through every TypedDict (SimulationState, AgentState, ActionRecord, MemoryEntry, RoundSummary, AdvisorBriefing, CountryOutcome). What field does each one have, what's the type, and WHY? How do the LangGraph Annotated reducers (merge_actions, merge_agents, merge_memories, merge_briefings) work — including the sentinel reset pattern?

2. **The Persistence Layer (storage.py)**: Walk through the 3 SQLite tables (simulations, rounds, batches). For EACH table: exact DDL (every column, type, constraint). Why does each column exist? What's the normalization level?

3. **The JSON↔SQL Boundary**: Identify every place where `json.dumps()` is called on write and `json.loads()` is called on read. What data crosses this boundary? Map out the exact data that lives as structured SQL columns vs. opaque JSON blobs. Which fields get "promoted" to first-class SQL columns (e.g. final_escalation, peak_escalation) and which stay buried in the final_state JSON blob?

4. **The WAL Pattern**: Explain the _WALConnection class, PRAGMA settings, busy_timeout, retry-with-backoff logic, and WHY SQLite needs this for async concurrency.

5. **Migration Pattern**: Document every ALTER TABLE migration in init_db() — why these columns were added later rather than in the original schema, and the backfill pattern for simulation_summary from the final_state JSON blob.

Output as a detailed chapter. Use code snippets, tables, and diagrams.
```

---

## Prompt 2: Optimal Redesign — If You Did It From Scratch

```
Now that we've documented the current architecture, design the OPTIMAL data architecture from scratch. The key insight I had: during the simulation (hot path), everything should be NoSQL/document-oriented for flexibility and speed. After simulations complete (cold path), data should be transformed/ETL'd into normalized SQL for analysis, querying, and embedding. Specifically:

1. **Hot Path (Simulation Runtime)**: Design the document store approach. Should we use MongoDB, Firestore, DynamoDB, or just JSON files? What's the document schema? How does LangGraph's state pass through? How do we handle the 20+ concurrent simulations in a Monte Carlo batch?

2. **Cold Path (Post-Simulation Analytics)**: Design the normalized SQL schema. Separate tables for: simulations, rounds, actions, escalation_events, agent_states, briefings, memories. Full 3NF normalization. Show the exact DDL. Show the foreign key relationships. Explain WHY each table exists separately instead of being a JSON blob.

3. **The ETL Pipeline**: Design the transformation that takes a completed simulation's document and shreds it into normalized SQL rows. Show the exact mapping: which document fields go to which SQL columns. Show the INSERT statements that would be generated.

4. **Embedding Strategy**: How would you store vector embeddings (for semantic search over simulation narratives) in the SQL layer? pgvector? FLOAT8[]? Separate embedding table linked by FK?

5. **Trade-off Analysis**: Why is this dual-store approach better than what we have? What are the downsides? When would you NOT use this pattern?

Frame everything as if explaining to a Palantir FDE interviewer. Use their vocabulary: ontology, data modeling, schema design, trade-offs.
```

---

## Prompt 3: The Big-Picture Interview Answer (1-2 Pages)

```
Now write the polished 1-2 page narrative I would actually SAY in a Palantir interview when asked "Tell me about a time you dealt with data at scale" or "Walk me through a data architecture decision you made."

Structure it as:
1. **Context** (30 seconds): What was the project? What was the data challenge?
2. **The Problem** (30 seconds): What was suboptimal about the original design? (JSON blobs in TEXT columns, no queryability, dual-purpose schema)
3. **My Analysis** (1 minute): How I identified the hot-path vs. cold-path split. Why NoSQL during simulation and SQL after.
4. **The Redesign** (1 minute): Concrete changes — document store for runtime, normalized SQL for analytics, ETL pipeline between them.
5. **The Results** (30 seconds): What improved — query speed, analytical capability, data accessibility.
6. **The Trade-offs** (30 seconds): What I gave up (operational complexity, eventual consistency during ETL).

This should read like a polished spoken answer — conversational but precise. Include specific numbers/metrics where possible. Mention column names, table names, and specific SQL patterns by name.
```

---

## Prompt 4: Deep-Dive Follow-Up Preparation (Beginner-Friendly)

```
A Palantir interviewer will ask follow-up questions. Write an extremely detailed FAQ/follow-up guide that covers every possible follow-up question, explained simply enough for someone who isn't a database expert. For each question, give:
- The question they'll ask
- The ideal 30-60 second answer
- A deeper explanation with examples

Cover at minimum:
1. "Why SQLite and not Postgres from the start?"
2. "What's the difference between WAL mode and default journal mode?"
3. "Why did you use json.dumps() instead of a JSONB column?"
4. "How would you handle schema migrations in production?"
5. "What's 3NF and why does it matter here?"
6. "How does your ETL pipeline handle failures mid-transform?"
7. "Why not just use a document database for everything?"
8. "How would you scale this beyond a single SQLite file?"
9. "What indexes would you add and why?"
10. "How would you add vector search / embeddings to this?"
11. "What's the Kahn Ladder data and why doesn't it fit neatly into SQL?"
12. "How did you handle the participating_actors being a list vs. a join table?"
13. "Walk me through exactly what happens when a simulation round completes — what SQL runs?"
14. "If you had to add real-time dashboards, what would change?"
15. "How would Palantir's Foundry/Ontology handle this differently?"

Write this as a study guide I can review on a plane. Make it exhaustive.
```
