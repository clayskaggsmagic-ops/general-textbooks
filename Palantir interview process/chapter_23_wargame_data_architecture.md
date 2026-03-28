# Chapter 23: The Data Architecture — How I Modeled and Stored Simulation Data

---

## 23.1 Why This Chapter Matters for Your Interview

When Palantir asks "tell me about a data-intensive project," they're not looking for a list of technologies. They want to hear you reason about *trade-offs*:

- Why did you choose this schema design over that one?
- Where did you normalize and where did you denormalize — and why?
- How did you handle data that doesn't fit neatly into rows and columns?
- How do you keep data consistent when multiple processes are writing simultaneously?

This chapter breaks down every data design decision in the Wargame 2 project. By the end, you'll be able to walk through the entire data model from memory, explain each trade-off, and connect the concepts to Palantir's way of thinking about data.

---

## 23.2 The Central Data Model — TypedDicts as the Schema

The entire simulation state is defined as a collection of Python `TypedDict` classes. Think of these as the *schema* of the simulation — they define exactly what fields exist, what types they are, and how they relate to each other. Let's walk through each one.

### 23.2.1 SimulationState — The Root Object

`SimulationState` is the single source of truth for the entire simulation at any point in time. Here's what it contains:

| Field | Type | Purpose |
|-------|------|---------|
| `scenario` | `str` | The original crisis text the user entered |
| `scenario_analysis` | `str` | The LLM's structured analysis of the scenario |
| `agents` | `dict[str, AgentState]` | Map of country code → leader state. Example: `{"US": {...}, "CN": {...}}` |
| `round_number` | `int` | Current round (1-indexed) |
| `max_rounds` | `int` | Maximum rounds before termination |
| `escalation_level` | `int` | Current Kahn Ladder rung (1-44) |
| `escalation_history` | `list[dict]` | Every escalation change: `[{round, level, change, reason}]` |
| `current_kahn_rung` | `dict` | Full rung info: `{rung, name, region, description}` |
| `actions_this_round` | `list[ActionRecord]` | Actions taken in the current round (reset each round) |
| `agent_memories` | `dict[str, list[MemoryEntry]]` | Map of country code → memory stream |
| `turn_order` | `list[str]` | Randomized order of play: `["CN", "US", "JP", "TW"]` |
| `current_turn_index` | `int` | Which position in the turn order is currently acting |
| `round_summaries` | `list[RoundSummary]` | Complete history of every round |
| `simulation_summary` | `str` | Final summary generated when the simulation ends |
| `simulated_date` | `str` | In-world date (advances ~1 week per round) |
| `briefings` | `dict[str, str]` | Map of country code → current briefing text |
| `escalation_reasoning` | `str` | Full LLM reasoning for the latest escalation assessment |

**The key design insight:** This is a **single, flat document** at the top level, not a deeply nested object graph. Country codes serve as natural keys. The top-level fields are all either primitives, flat dictionaries keyed by country code, or append-only lists. This makes state updates simple — you return a partial dictionary with only the fields you changed, and the graph engine merges it.

### 23.2.2 AgentState — One Leader's Complete Profile

Each leader gets an `AgentState` dictionary containing everything the system knows about them:

| Field | Type | Purpose |
|-------|------|---------|
| `country_code` | `str` | ISO-style code: `"US"`, `"CN"`, `"RU"` |
| `country_name` | `str` | Full name: `"United States"` |
| `leader_name` | `str` | `"Joe Biden"` |
| `leader_title` | `str` | `"President"` |
| `persona_description` | `str` | Multi-paragraph personality profile built from real research |
| `cabinet_roster` | `str` | Named officials: Secretary of State, Defense Minister, etc. |
| `current_briefing` | `str` | The last intelligence briefing this leader received |
| `joined_round` | `int` | The round this leader entered the simulation |

**Design choice: persona as unstructured text.** The `persona_description` is a blob of freeform text, not a structured object with fields like `ideology: "hawkish"` or `risk_tolerance: 7`. Why? Because the persona is consumed by an LLM, not by application code. The LLM gets more value from a nuanced, prose description ("Xi Jinping believes in patient strategic accumulation but will act decisively when core interests are threatened, especially regarding Taiwan, which he views as a personal legacy issue") than from structured flags. This is a deliberate choice to optimize for the LLM consumer rather than the database consumer.

### 23.2.3 ActionRecord — What a Leader Did

Every time a leader takes an action, it produces an `ActionRecord`:

| Field | Type | Purpose |
|-------|------|---------|
| `round_number` | `int` | Which round this happened in |
| `actor` | `str` | Country code of the acting leader |
| `actor_name` | `str` | Human-readable name: `"Vladimir Putin"` |
| `action_type` | `str` | One of: `STATEMENT`, `MILITARY`, `ECONOMIC`, `DIPLOMATIC`, `INTELLIGENCE`, `OTHER` |
| `target` | `str \| None` | Target country code, or `None` for non-targeted actions |
| `description` | `str` | Detailed operational account (2-3 paragraphs) |
| `public_statement` | `str \| None` | What the leader said publicly |
| `private_channels` | `str \| None` | Secret back-channel communications |
| `reasoning` | `str` | The leader's private internal reasoning (never shown to other leaders) |
| `escalation_impact` | `int` | How much this action changed the escalation level |

**This is an event record — a log, not a current state.** Each `ActionRecord` is immutable once created. It's appended to `actions_this_round` and then preserved in the round summary. This is fundamentally an **event-sourcing** pattern: the system doesn't just store the current state; it stores every event that led to the current state. You could replay the entire simulation from the action log. In Palantir terms, this is the same pattern as an audit trail — every change is logged, traceable, and non-destructive.

### 23.2.4 MemoryEntry — The Unit of Memory

Every observation, reflection, and consolidated abstraction is stored as a `MemoryEntry`:

| Field | Type | Purpose |
|-------|------|---------|
| `round_number` | `int` | When this memory was formed |
| `timestamp` | `str` | ISO timestamp |
| `content` | `str` | The actual memory text |
| `importance` | `int` | Subjective importance (1-10), rated by the leader's persona |
| `is_reflection` | `bool` | True if this is a reflection or abstraction, False if it's a raw observation |
| `consolidated` | `bool` | True if this memory has been archived (superseded by a higher-level abstraction) |

**The `consolidated` flag is archives, not deletes.** When memories get consolidated, they aren't removed from the list. They're marked `consolidated: True` and filtered out of active queries. This means you can always go back and see the original granular memories that were consolidated — full audit trail, no data loss.

**The `importance` field drives intelligent filtering.** When formatting memories for an LLM prompt, the system filters to only active (non-consolidated) memories, sorts by recency, and caps at 8 entries. But the importance score is also used by consolidation logic: memories with importance ≥ 8 are *protected* from consolidation, surviving longer at full granularity.

---

## 23.3 The Reducer Pattern — How State Updates Work

Here's a problem that doesn't exist in simple CRUD apps: **concurrent partial updates to the same state.**

In the simulation, the `leader_action` node returns a partial state update like:

```python
return {
    "actions_this_round": [action],           # One new action
    "agent_memories": {current_country: [...]},  # New memories for this leader
    "current_turn_index": current_idx + 1       # Move to next leader
}
```

But `actions_this_round` already has previous actions from this round. And `agent_memories` already has memories from previous rounds. How do you merge a partial update into the existing state without overwriting what's already there?

The answer is **reducer functions** — custom merge logic attached to each field via Python's `Annotated` type:

```python
class SimulationState(TypedDict):
    actions_this_round: Annotated[list[ActionRecord], merge_actions]
    agent_memories: Annotated[dict[str, list[MemoryEntry]], merge_memories]
    agents: Annotated[dict[str, AgentState], merge_agents]
    briefings: Annotated[dict[str, str], merge_briefings]
```

### 23.3.1 How `merge_actions` Works

`merge_actions` handles the `actions_this_round` list. Its logic:

1. If the new value contains a special **sentinel value** (`RESET_ACTIONS_SENTINEL`), clear the entire list and return only non-sentinel items. This is used at the end of each round to reset the action list for the next round.
2. Otherwise, **append** the new actions to the existing list.

This is elegant because it handles two different operations through the same reducer: *appending* new actions during a round, and *resetting* the list between rounds.

### 23.3.2 How `merge_memories` Works

`merge_memories` is the most complex reducer. Its logic:

1. For each country code in the new update:
   - If the new memories list starts with a special sentinel value (`REPLACE_MEMORIES_SENTINEL`), **replace** the entire memory stream for that country with the new list (minus the sentinel). This is used during memory consolidation, where the entire memory list is rebuilt with archived originals and new abstractions.
   - Otherwise, **append** the new memories to the existing list for that country.
2. Countries not mentioned in the update are left unchanged.

This is a critical design decision. There are two fundamentally different memory operations — *add new memories* and *replace all memories* (consolidation) — and they need to go through the same reducer interface. The sentinel pattern lets you distinguish between the two without changing the function signature.

### 23.3.3 Why This Matters for Your Interview

Reducers solve the same problem that **MVCC** (Multi-Version Concurrency Control) solves in relational databases and that **CRDTs** (Conflict-free Replicated Data Types) solve in distributed systems. When multiple writers update the same state, you need a merge strategy. The interview-ready version:

> "I implemented custom reducer functions for each complex field in the state. Instead of overwriting, each reducer defines how new data merges with existing data — append for actions, key-level merge for memories, shallow merge for agent profiles. I also designed a sentinel pattern to handle the two different memory operations (append vs. replace) through the same interface. This is conceptually the same problem as conflict resolution in distributed databases."

---

## 23.4 The Database Schema — SQLite and Why

The simulation state exists in memory during execution (as a LangGraph state graph), but it needs to persist to disk for history, replay, and analytics. Here's the SQLite schema:

### 23.4.1 The `simulations` Table

```sql
CREATE TABLE IF NOT EXISTS simulations (
    id TEXT PRIMARY KEY,
    scenario TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    final_state TEXT,           -- Full SimulationState as JSON blob
    status TEXT DEFAULT 'running',
    round_count INTEGER DEFAULT 0,
    escalation_level INTEGER DEFAULT 1,
    participant_names TEXT,     -- Comma-separated leader names for display
    simulation_summary TEXT,    -- Final summary text
    batch_id TEXT               -- Groups simulations into Monte Carlo batches
)
```

### 23.4.2 The `rounds` Table

```sql
CREATE TABLE IF NOT EXISTS rounds (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    simulation_id TEXT NOT NULL,
    round_number INTEGER NOT NULL,
    actions TEXT,               -- JSON array of ActionRecords
    turn_order TEXT,            -- JSON array of country codes
    summary TEXT,
    escalation_before INTEGER,
    escalation_after INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (simulation_id) REFERENCES simulations(id)
)
```

### 23.4.3 The `batches` Table

```sql
CREATE TABLE IF NOT EXISTS batches (
    id TEXT PRIMARY KEY,
    name TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    scenario TEXT,
    simulation_count INTEGER DEFAULT 0,
    store_id TEXT,              -- Gemini File Search vector store ID
    store_status TEXT DEFAULT 'pending'
)
```

### 23.4.4 Schema Design Decisions and Trade-offs

This is where interview value lives — in the *reasoning* behind each decision.

**Decision 1: `final_state` as a JSON blob, not normalized tables.**

The `final_state` column stores the entire `SimulationState` dictionary as a single JSON string. Why not normalize it into separate tables for agents, memories, actions, etc.?

| Option | Pros | Cons |
|--------|------|------|
| **JSON blob** (what I chose) | Schema evolution is free (add fields without migrations); one read loads everything; matches the in-memory structure exactly | Can't easily query individual fields; large row size; no referential integrity on nested data |
| **Normalized tables** | Query individual agents, memories, actions with SQL; proper foreign keys; smaller row sizes | Need migration for every schema change; N+1 query problem when loading full state; complex JOINs for reconstruction |

**Why JSON won:** This is a *write-heavy, read-whole* workload. When the simulation finishes, you store the complete state once. When you load it, you load the *entire* state — you never query "just the memories for Russia." The primary read pattern is "give me everything for simulation X," which makes a single JSON column faster than joining 5 normalized tables. And since the schema was evolving rapidly during development (new fields every few days), avoiding database migrations was a significant productivity advantage.

**Interview framing:** "I made a pragmatic denormalization decision. The access pattern was always full-object reads and writes — we never needed to query individual fields of the nested state from SQL. So storing as a JSON blob eliminated schema migration overhead and matched the in-memory data structure exactly. If the access pattern changed — say, we needed to query 'show me all simulations where China's escalation exceeded rung 12' — I'd normalize those specific fields into first-class columns."

**Decision 2: The `rounds` table normalizes what matters.**

Notice that while `final_state` is a JSON blob, the `rounds` table has explicit columns for `escalation_before`, `escalation_after`, `round_number`, and `simulation_id`. These are the fields you *do* want to query directly:

- "Show me all rounds where escalation changed by more than 3 rungs"
- "What's the average escalation after Round 4 across all simulations?"
- "Plot escalation over time for simulation X"

The `actions` and `turn_order` columns in `rounds` are still JSON, because those are complex nested structures that are always consumed as a whole.

**Interview framing:** "I applied selective denormalization. Fields that I knew I'd want to filter and aggregate on — escalation levels, round numbers, status — got their own columns with proper types. Complex nested structures that are only consumed whole — action records, turn orders — stayed as JSON blobs. This is the 80/20 rule of schema design: normalize what you query, denormalize what you read in bulk."

**Decision 3: SQLite with WAL mode.**

Why SQLite instead of PostgreSQL?

- **Development speed.** SQLite is zero-config. No Docker container, no connection strings, no migrations framework. For a project where the schema was changing daily, this was a huge productivity advantage.
- **Single-server deployment.** The simulation doesn't need horizontal scaling — it's one server running one simulation at a time (or one batch of simulations sequentially). SQLite handles this perfectly.
- **WAL (Write-Ahead Logging) mode.** SQLite is configured with `journal_mode=WAL`, which allows concurrent reads while a write is in progress. Without WAL, SQLite blocks all reads during any write operation. With WAL, readers see a consistent snapshot of the database while the writer appends to a log.

```python
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA busy_timeout=5000")  # Wait up to 5 seconds if locked
```

**Production migration path:** The storage layer is abstracted behind a `Storage` class with methods like `create_simulation()`, `save_round()`, `get_simulation()`. Swapping SQLite for PostgreSQL means changing the connection logic and SQL dialect, not the application code. The interface is the same.

**Interview framing:** "I used SQLite for development velocity — zero configuration, no container to manage, schema changes without migration headaches. But I designed the storage layer with a clean abstraction, so the upgrade path to PostgreSQL is a connection-string change, not a rewrite. I also enabled WAL mode to handle the concurrent read/write pattern where the frontend reads state while the simulation is writing round results."

**Decision 4: Async retry with exponential backoff.**

Database operations can fail transiently — file locks, disk I/O hiccups, connection timeouts. The storage layer wraps every database operation in an async retry wrapper with exponential backoff:

```python
async def _retry_db_operation(self, operation, *args, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            return await asyncio.to_thread(operation, *args, **kwargs)
        except Exception as e:
            if attempt < max_retries - 1:
                wait = 0.1 * (2 ** attempt)  # 0.1s, 0.2s, 0.4s
                await asyncio.sleep(wait)
            else:
                raise
```

This is a standard reliability pattern, but it's worth mentioning in an interview because it shows you think about failure modes, not just the happy path.

---

## 23.5 Data Flow Through the System

Here's how data moves from user input to persistent storage to frontend display:

```
User types scenario
        │
        ▼
┌─────────────────┐     POST /simulations
│    Frontend      │────────────────────────▶┌──────────────┐
│  (React + TS)    │                         │   FastAPI     │
│                  │◀────── WebSocket ───────│   Backend     │
└─────────────────┘   (real-time events)     └──────┬───────┘
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │  LangGraph    │
                                              │  StateGraph   │
                                              │               │
                                              │  Node 1 ──▶──│──▶ Gemini API (LLM call)
                                              │  Node 2 ──▶──│──▶ Gemini API
                                              │  ...         │
                                              │  Node N ──▶──│──▶ Gemini API
                                              │               │
                                              │  State:       │
                                              │  {SimState}   │
                                              └──────┬───────┘
                                                     │
                                           Each round: save_round()
                                           End: save final_state
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │   SQLite DB   │
                                              │               │
                                              │  simulations  │
                                              │  rounds       │
                                              │  batches      │
                                              └──────┬───────┘
                                                     │
                                           Batch analytics: upload
                                           documents to File Search
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │ Gemini File   │
                                              │ Search Store  │
                                              │               │
                                              │ Vector index  │
                                              │ of simulation │
                                              │ documents     │
                                              └──────────────┘
```

### Step-by-step:

1. **User submits scenario** → Frontend sends `POST /simulations` with the scenario text.
2. **Backend creates simulation record** → `Storage.create_simulation()` inserts a row in `simulations` with status `"running"`.
3. **LangGraph begins execution** → The compiled StateGraph starts at `analyze_scenario`, processes through all nodes.
4. **Each LLM call** → The node formats a prompt, calls Gemini with structured output, parses the JSON response, and returns a partial state update.
5. **State merging** → LangGraph applies the partial update using the reducer functions (merge_memories, merge_actions, etc.).
6. **WebSocket events** → After each node completes, the backend emits a WebSocket event with the updated data. The frontend renders it immediately.
7. **Round persistence** → At the end of each round (in `analyze_round`), `Storage.save_round()` inserts a row in `rounds` with the round's actions, escalation changes, and summary.
8. **Final persistence** → When the simulation ends, `Storage.update_simulation()` saves the complete `final_state` JSON and updates status to `"completed"`.
9. **Batch analytics** → When a batch of simulations completes, the system generates structured documents for each run and uploads them to a Gemini File Search vector store for RAG queries.

---

## 23.6 The Agent Memory Data Flow — A Deep Dive

Memory is the most complex data flow in the system because it involves multiple LLM calls and two different update modes (append and replace). Here's exactly what happens:

### During a Round (One Leader's Turn)

```
1. Leader gets briefing (generate_single_briefing)
   └──▶ Briefing text stored in agent["current_briefing"]

2. Leader reflects (generate_pre_action_reflection)
   └──▶ 2-4 thoughts, each with importance 1-10
   └──▶ Each thought → MemoryEntry(is_reflection=True)
   └──▶ Added to agent_memories via merge_memories reducer (APPEND mode)

3. Leader acts (leader_action)
   └──▶ Generates ActionRecord
   └──▶ Actor rates own action importance (rate_memory_importance call)
   └──▶ Actor's action → MemoryEntry(is_reflection=False)
   └──▶ Added to agent_memories via merge_memories reducer (APPEND mode)
```

### After All Leaders Act (End of Round)

```
4. Analyze round (analyze_round)
   ├──▶ Assess escalation using Kahn Ladder
   │
   ├──▶ Generate post-round reflections for each leader
   │    └──▶ Each leader synthesizes 1 key insight from the round
   │    └──▶ MemoryEntry(is_reflection=True, importance: 7-10)
   │    └──▶ Added via merge_memories (APPEND mode)
   │
   └──▶ Check memory consolidation for each leader
        └──▶ IF active memories > 50:
             ├──▶ Protect recent memories (last 3 rounds)
             ├──▶ Protect high-importance memories (≥ 8)
             ├──▶ Send remaining old/low-importance memories to LLM
             ├──▶ LLM produces 2-4 abstractions
             ├──▶ Mark originals as consolidated=True
             ├──▶ Use REPLACE_MEMORIES_SENTINEL in merge_memories
             └──▶ Full memory list replacement via reducer (REPLACE mode)
```

### Why Two Modes Matter

If consolidation always used APPEND mode, the consolidated (archived) memories would lose their `consolidated: True` flag because the reducer would just append new memories to the existing list without modifying existing entries. The REPLACE mode lets the consolidation logic rebuild the entire memory list — with the originals marked as archived and the new abstractions marked as active — and submit it atomically. The sentinel value tells the reducer "don't append — overwrite."

This is a nuanced but critical data integrity decision. In an interview:

> "The memory system has two fundamentally different update operations: 'add a new memory' and 'rebuild the entire memory stream during consolidation.' They go through the same state management interface, but their semantics are different. I solved this with a sentinel pattern — a special marker at the start of the update list tells the merge function whether to append or replace. It's conceptually similar to how databases use log sequence numbers to distinguish between insert and update operations."

---

## 23.7 The RAG Analytics Pipeline — Data for Batch Analysis

After running a batch of 10-20 simulations, the system lets you ask questions about the aggregate results. Here's the data architecture behind it:

### 23.7.1 Document Generation

Each simulation is converted into a structured text document:

```
=== SIMULATION: [simulation_id] ===
Scenario: China blockades Taiwan...
Final Escalation: Rung 14 (Declaration of Limited Conventional War)
Participants: Xi Jinping, Joe Biden, Lai Ching-te, Fumio Kishida

--- ROUND 1 ---
Escalation: 1 → 5
Actions:
  - Xi Jinping (CN): Ordered 3rd Fleet to begin live-fire exercises...
  - Joe Biden (US): Directed Pentagon to deploy USS Reagan CSG...
  ...

--- ROUND 2 ---
...
```

### 23.7.2 Indexing

These documents are uploaded to a **Gemini File Search vector store**. The vector store:
1. Chunks the documents into overlapping segments
2. Generates embedding vectors for each chunk
3. Stores the chunks in an index for fast retrieval

The `batches` table tracks the vector store ID (`store_id`) and its indexing status (`store_status`).

### 23.7.3 Query Flow

When you ask a question like "In how many simulations did China escalate past Rung 9?":

1. The question is sent to Gemini with the vector store attached
2. Gemini retrieves relevant chunks from the indexed simulation documents
3. The LLM generates an answer grounded in the retrieved data
4. The response includes citations — references to specific files and chunks
5. The backend parses these citations and maps them to simulation IDs and round numbers

### 23.7.4 Citation Parsing — The Hardest Part

Gemini's File Search API returns citations as file references, not database IDs. The citation parsing logic uses four strategies in fallback order:

1. **Filename parsing** — extract the simulation ID from the document filename
2. **Document header parsing** — look for `=== SIMULATION: [id] ===` in the retrieved chunk
3. **Round number extraction** — look for `--- ROUND N ---` markers
4. **Synthetic citation generation** — if the answer mentions a simulation or round by ID without a grounded citation, generate a citation automatically

The system also produces a structured `AnalystResponse` with typed `Citation` objects:

```python
class Citation:
    simulation_id: str
    round_number: int | None
    text: str       # The text passage that this citation supports
    source: str     # Where the citation came from (File Search, synthetic, etc.)
```

**Interview framing:** "The analytics layer is a RAG pipeline — same pattern Palantir uses in AIP. Simulation data is converted to structured documents, indexed in a vector store, and queried with natural language. The hardest part was citation parsing — mapping the LLM's file references back to database-level identifiers. I wrote a four-strategy fallback parser because no single approach handles all edge cases."

---

## 23.8 Entity Relationship Summary

Here's how all data entities relate to each other:

```
                    ┌────────────────────┐
                    │      Batch          │
                    │                     │
                    │  id (PK)            │
                    │  scenario           │
                    │  simulation_count   │
                    │  store_id ──────────┼──▶ Gemini File Search Store
                    └────────┬───────────┘
                             │ 1:N
                             ▼
                    ┌────────────────────┐
                    │    Simulation       │
                    │                     │
                    │  id (PK)            │
                    │  batch_id (FK) ─────┼──▶ Batch
                    │  scenario           │
                    │  final_state (JSON) │─ ─ ─ ─▶ SimulationState
                    │  status             │           │
                    │  escalation_level   │           ├── agents: {CC → AgentState}
                    └────────┬───────────┘           ├── agent_memories: {CC → [MemoryEntry]}
                             │ 1:N                   ├── actions_this_round: [ActionRecord]
                             ▼                       ├── round_summaries: [RoundSummary]
                    ┌────────────────────┐           ├── escalation_history: [{...}]
                    │      Round          │           └── ... (15+ fields)
                    │                     │
                    │  id (PK)            │
                    │  simulation_id (FK) │
                    │  round_number       │
                    │  actions (JSON) ────┼──▶ [ActionRecord, ActionRecord, ...]
                    │  escalation_before  │
                    │  escalation_after   │
                    └────────────────────┘
```

The left side is the **relational schema** (SQL tables with proper foreign keys). The right side is the **document schema** (nested JSON structures within the `final_state` column). This hybrid approach gives you the queryability of relational data where you need it (batch → simulation → round lookups) and the flexibility of document data where you need it (complex nested simulation state).

---

## 23.9 Connecting to Palantir Concepts

Here's how the data architecture maps to Palantir's world:

| This Project | Palantir Equivalent | Connection |
|-------------|---------------------|------------|
| `SimulationState` TypedDict | **Ontology Object Type** | Both define a structured schema for a domain entity with typed fields |
| `ActionRecord` as immutable event log | **Event table in Foundry** | Both store immutable events with timestamps and actor information |
| `agent_memories` with importance scores | **Enriched data with computed properties** | Both attach computed metadata (importance) to raw data |
| Reducer functions | **Pipeline merge logic** | Both handle the problem of merging partial updates into a master state |
| JSON blob in `final_state` | **Complex property in Foundry** | Foundry supports nested JSON properties on ontology objects |
| Gemini File Search analytics | **AIP (AI Platform)** | Both use LLM + indexed data + citation tracking to answer questions |
| Kahn Ladder rung assessment | **Derived property / computed metric** | Both compute a standardized metric from raw input data |
| OPSEC visibility filtering | **Role-based access control** | Both restrict data visibility based on the observer's identity |
| `REPLACE_MEMORIES_SENTINEL` | **Atomic batch update pattern** | Both handle the "replace the whole thing" vs. "append to it" distinction |
| SQLite → PostgreSQL migration path | **Data source connector strategy** | Both abstract the storage layer behind a clean interface for future flexibility |

This mapping is powerful in an interview because it shows you can think in Palantir's abstractions, not just in raw technology terms.

---

## 23.10 What I'd Do Differently (Self-Critique for Interviews)

Interviewers love self-awareness. Here are honest trade-offs and things you'd change:

**1. I'd add a proper events table.** Right now, actions are stored as JSON inside the `rounds` table. If I needed to query "all military actions by China across all simulations," I'd have to parse JSON in SQL. A dedicated `actions` table with columns for `actor`, `action_type`, `target`, and `round_number` would make those queries trivial.

**2. I'd version the state schema.** The `final_state` JSON blob doesn't include a schema version number. When I add new fields to `SimulationState`, old simulation records don't have those fields. I handle this with Python's `.get()` with defaults, but a formal schema versioning approach (like adding a `schema_version` field to every stored state) would be more robust.

**3. I'd consider a time-series database for escalation tracking.** The `escalation_history` is stored as a JSON array inside `SimulationState`. For batch analytics ("average escalation trajectory across 100 runs"), a time-series store like TimescaleDB would be much more natural. Each row would be `(simulation_id, round_number, escalation_level, timestamp)` — perfectly structured for aggregation queries.

**4. I'd separate the memory store.** Memories are stored inside the `final_state` JSON blob, which means you can't query memories without loading the entire simulation state. A dedicated `memories` table with columns for `simulation_id`, `country_code`, `round_number`, `importance`, `content`, and `consolidated` would enable queries like "show me all high-importance memories across all simulations" without loading megabytes of JSON.

**Interview framing:** "I made pragmatic choices for development velocity — JSON blobs, single-file database, schema-free evolution. If I were building this for production at Palantir scale, I'd normalize the access-pattern-critical data — actions, memories, escalation history — into proper tables while keeping the full state blob for snapshot/replay functionality. It's the classic read-optimization vs. write-optimization trade-off."

---

## 23.11 Summary — Data Architecture Cheat Sheet

| Decision | Choice | Rationale |
|----------|--------|-----------|
| State representation | TypedDicts with annotated reducers | Type safety + custom merge logic for concurrent updates |
| Persona storage | Unstructured text blob | Optimized for LLM consumption, not SQL queries |
| Action storage | Immutable event records | Event-sourcing pattern; full audit trail |
| Memory archival | `consolidated=True` flag, never delete | Audit trail; originals always recoverable |
| Database | SQLite (dev) with WAL mode | Zero-config, concurrent reads, easy migration to PostgreSQL |
| Complex state | JSON blob in `final_state` column | Schema flexibility; full-object read/write access pattern |
| Queryable fields | Explicit columns for escalation, round, status | Selective denormalization for the fields you query |
| Merge strategy | Sentinel-based reducer pattern | Two modes (append/replace) through one interface |
| Batch analytics | Gemini File Search RAG | LLM-powered queries over indexed simulation documents |
| Retry logic | Exponential backoff with corrective re-prompting | Both DB and LLM calls need failure handling |
