# Chapter 27: The Database Layer — SQL Tables, JSON↔SQL Transforms, and Embedding Storage

---

## 27.1 Why This Chapter Matters for Palantir

When Palantir asks "tell me about how you stored and queried data," they want to hear you talk about:

- **Exact table schemas** — not abstractions, actual DDL
- **Why each column exists** — what decision led to that column vs. a JSON blob
- **Data boundary transforms** — how does a JavaScript object become a SQL row and back?
- **Embedding storage** — how do you persist 768-dimensional vectors in a relational database?
- **Row-Level Security** — how do you enforce data isolation in a multi-tenant system?

This chapter is a **forensic-level walkthrough** of every SQL table in the Siren (Ithaca) system, every JSON↔SQL boundary crossing, and every design trade-off.

---

## 27.2 The Two Tables — Complete DDL

Siren has exactly **two tables** in Supabase (PostgreSQL). Here is the complete, unmodified DDL from `backend/supabase/schema.sql`:

### 27.2.1 `sessions` — The Active Focus Session

```sql
CREATE TABLE IF NOT EXISTS sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  task_description TEXT NOT NULL,
  task_embedding FLOAT8[] NOT NULL,  -- 768-dim vector stored as native PG array
  block_mode TEXT NOT NULL DEFAULT 'warn' 
    CHECK (block_mode IN ('warn', 'block')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL,
  
  CONSTRAINT fk_user FOREIGN KEY (user_id) 
    REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX IF NOT EXISTS idx_sessions_user_id ON sessions(user_id);
CREATE INDEX IF NOT EXISTS idx_sessions_expires_at ON sessions(expires_at);
```

**Why each column exists:**

| Column | Type | Why It Exists |
|--------|------|---------------|
| `id` | `UUID` | Client-generated (via `crypto.randomUUID()` in the Express backend). Not auto-increment — allows the backend to know the ID before the INSERT round-trip completes. |
| `user_id` | `UUID` | FK to Supabase Auth's `auth.users` table. CASCADE delete means when the user account is deleted, all their sessions are wiped. |
| `task_description` | `TEXT` | The raw task the user typed: "Write my CS 401 essay on graph algorithms". Stored verbatim for display and debugging. |
| `task_embedding` | `FLOAT8[]` | **The embedding vector.** 768 float64 values from Google's `text-embedding-004` model. This is the mathematical representation of the task. See §27.4. |
| `block_mode` | `TEXT` with CHECK | Either `'warn'` or `'block'`. A CHECK constraint acts as an enum. `warn` = show overlay with "continue anyway" button. `block` = hard redirect. |
| `created_at` | `TIMESTAMPTZ` | Server-generated timestamp. Used for analytics ("when do users start sessions?"). |
| `expires_at` | `TIMESTAMPTZ` | When the session auto-expires. Set by the backend based on LLM-estimated task duration. Indexed for cleanup. |

**Why two indexes?**

- `idx_sessions_user_id` — Every API call includes the user's JWT. The backend queries `WHERE user_id = ?` constantly. Without this index, every classification request would table-scan.
- `idx_sessions_expires_at` — The `cleanup_expired_sessions()` function runs periodically and queries `WHERE expires_at < NOW()`. Without this index, cleanup would table-scan.

### 27.2.2 `classification_events` — The Decision Audit Log

```sql
CREATE TABLE IF NOT EXISTS classification_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
  event_type TEXT NOT NULL CHECK (event_type IN ('web', 'app', 'screen')),
  decision TEXT NOT NULL CHECK (decision IN ('allow', 'warn', 'block')),
  similarity FLOAT8 NOT NULL,
  confidence FLOAT8 NOT NULL,
  
  -- Web classification fields
  url TEXT,
  title TEXT,
  
  -- App classification fields
  app_name TEXT,
  window_title TEXT,
  
  -- Common fields
  reason TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_events_session_id 
  ON classification_events(session_id);
CREATE INDEX IF NOT EXISTS idx_events_created_at 
  ON classification_events(created_at);
CREATE INDEX IF NOT EXISTS idx_events_decision 
  ON classification_events(decision);
```

**Why each column exists:**

| Column | Type | Why |
|--------|------|-----|
| `session_id` | `UUID FK` | Every classification event belongs to exactly one session. CASCADE means deleting the session deletes all its events. |
| `event_type` | `TEXT CHECK` | Discriminator: `'web'` (browser URL), `'app'` (desktop app), `'screen'` (screenshot analysis). Determines which nullable fields are populated. |
| `decision` | `TEXT CHECK` | The classification verdict: `'allow'`, `'warn'`, `'block'`. Indexed for analytics queries like "how often do we block per session?". |
| `similarity` | `FLOAT8` | The raw cosine similarity score (0.0 to 1.0). Stored for threshold analysis. |
| `confidence` | `FLOAT8` | The derived confidence score. Currently maps 1:1 from similarity but could diverge if we add multi-signal fusion. |
| `url` / `title` | `TEXT NULL` | Only populated when `event_type = 'web'`. The URL and page title that were classified. |
| `app_name` / `window_title` | `TEXT NULL` | Only populated when `event_type = 'app'`. The desktop app and its window title. |
| `reason` | `TEXT NOT NULL` | Human-readable explanation: "Content about graph algorithms is directly relevant to your CS essay task." |

**Design pattern: discriminated nullable columns.** Rather than having separate `web_events` and `app_events` tables, Siren uses a single table with nullable columns. This is a **denormalization trade-off:**

- ✅ **Simpler queries** — no JOINs needed for the analytics dashboard
- ✅ **Simpler INSERT** — one function handles all event types
- ❌ **Nullable columns waste space** — `url` is always NULL for app events
- ❌ **No constraint enforcement** — nothing prevents an `event_type='web'` row with NULL `url` (would need a CHECK constraint to enforce)

**For the interview:** This is a common pattern in analytics tables. Palantir would recognize it as a "wide table" or "denormalized event log." The trade-off is queryability vs. schema strictness.

---

## 27.3 The JSON↔SQL Boundary — Exact Transform Points

This is the most interview-relevant section. Every time data crosses the boundary between TypeScript objects and PostgreSQL rows, a transform happens. Here is **every single transform point:**

### 27.3.1 Creating a Session (JSON → SQL)

**TypeScript object (what the Express route handler has):**

```typescript
// In session.ts route handler
const sessionData = {
  id: crypto.randomUUID(),        // "a1b2c3d4-..."
  userId: req.user.id,            // From JWT: "f5e6d7c8-..."
  taskDescription: "Write essay", // From request body
  taskEmbedding: [0.023, -0.114, 0.892, ...],  // 768 floats from Gemini
  blockMode: "warn",              // From request body
  expiresAt: new Date("2024-03-28T10:00:00Z"),  // Computed by backend
};
```

**The transform in `supabase.ts` → `createSession()`:**

```typescript
const { error } = await client.from('sessions').insert({
  id: session.id,                           // string → UUID (Supabase casts)
  user_id: session.userId,                  // camelCase → snake_case
  task_description: session.taskDescription, // camelCase → snake_case
  task_embedding: session.taskEmbedding,     // number[] → FLOAT8[] (PG array)
  block_mode: session.blockMode,             // camelCase → snake_case
  expires_at: session.expiresAt.toISOString(), // Date → ISO 8601 string → TIMESTAMPTZ
});
```

**What actually hits PostgreSQL:**

```sql
INSERT INTO sessions (id, user_id, task_description, task_embedding, block_mode, expires_at)
VALUES (
  'a1b2c3d4-...',
  'f5e6d7c8-...',
  'Write essay',
  ARRAY[0.023, -0.114, 0.892, ...]::FLOAT8[],  -- Native PG array, NOT a JSON string
  'warn',
  '2024-03-28T10:00:00.000Z'::TIMESTAMPTZ
);
```

**Key transforms:**

| TypeScript | PostgreSQL | How |
|------------|-----------|-----|
| `string` (UUID format) | `UUID` | Supabase client handles the cast |
| `camelCase` keys | `snake_case` columns | Manual mapping in the `insert()` call |
| `number[]` (768 floats) | `FLOAT8[]` (native PG array) | Supabase's PostgREST auto-serializes JS arrays to PG arrays |
| `Date` object | `TIMESTAMPTZ` | `.toISOString()` produces `"2024-03-28T10:00:00.000Z"`, PG parses this |

### 27.3.2 Reading a Session (SQL → JSON)

**What PostgreSQL returns (via PostgREST):**

```json
{
  "id": "a1b2c3d4-...",
  "user_id": "f5e6d7c8-...",
  "task_description": "Write essay",
  "task_embedding": [0.023, -0.114, 0.892, ...],
  "block_mode": "warn",
  "created_at": "2024-03-28T08:00:00+00:00",
  "expires_at": "2024-03-28T10:00:00+00:00"
}
```

**The transform in `session.ts` → `getSession()`:**

```typescript
// Convert DB row (snake_case) to Session domain type (camelCase)
return {
  sessionId: row.id,
  userId: row.user_id,
  taskDescription: row.task_description,
  taskEmbedding: row.task_embedding,    // FLOAT8[] → number[] (automatic)
  blockMode: row.block_mode as BlockMode,  // string → union type (type assertion)
  startedAt: new Date(row.created_at),  // ISO string → Date object
  expiresAt: new Date(row.expires_at),  // ISO string → Date object
};
```

**Key transforms on read:**

| PostgreSQL | TypeScript | How |
|-----------|-----------|-----|
| `snake_case` columns | `camelCase` keys | Manual remapping in the return statement |
| `FLOAT8[]` (PG array) | `number[]` | PostgREST auto-deserializes PG arrays to JSON arrays |
| `TIMESTAMPTZ` string | `Date` object | `new Date("2024-03-28T10:00:00+00:00")` |
| `TEXT` with CHECK | `'warn' \| 'block'` union type | `as BlockMode` type assertion |

### 27.3.3 Logging a Classification Event (JSON → SQL)

```typescript
// In classify.ts — after classification completes
await logClassificationEvent({
  session_id: sessionId,          // Already snake_case (matches DB directly)
  event_type: 'web',
  decision: result.decision,      // 'allow' | 'warn' | 'block'
  similarity: result.similarity,  // 0.0-1.0
  confidence: result.confidence,  // 0.0-1.0
  url: req.body.url,
  title: req.body.title,
  reason: result.reason,
  // app_name, window_title: undefined (omitted for web events)
});
```

**Important detail:** The `ClassificationEventRow` interface already uses `snake_case` — it's designed to be inserted directly with **no field remapping**. This is a deliberate design choice: the event log row type mirrors the DB schema exactly, eliminating the transform layer.

```typescript
// In supabase.ts — the insert is direct, no remapping
const { error } = await client.from('classification_events').insert(event);
```

**The fail-open pattern:**

```typescript
if (error) {
  console.error('[SUPABASE] Error logging event:', error);
  // Don't throw — logging failures shouldn't break classification
  return;
}
```

This is a critical design decision: **classification event logging is fire-and-forget.** If the database is down, the user still gets their classification result. The event is simply lost. This is the fail-open philosophy applied to analytics.

---

## 27.4 Embedding Storage — How 768 Floats Live in PostgreSQL

### 27.4.1 The Storage Type: `FLOAT8[]`

The `task_embedding` column uses PostgreSQL's native array type: `FLOAT8[]`. This means:

- **768 individual `float64` values** stored as a single PostgreSQL array value
- **NOT a JSON string** — it's a native PG type with array operators
- **NOT pgvector** — Siren doesn't use the pgvector extension (yet)

### 27.4.2 Why `FLOAT8[]` Instead of Alternatives

| Option | Pros | Cons | Siren's Choice |
|--------|------|------|-----------------|
| `FLOAT8[]` | Native PG type, no extensions, Supabase auto-serializes JS arrays | No built-in similarity operators, no ANN index | ✅ Used |
| `vector(768)` (pgvector) | Built-in cosine similarity, ANN indexing (IVFFlat/HNSW) | Requires pgvector extension, more complex setup | Not used |
| `TEXT` (JSON string) | Dead simple | No math operators, must parse on every read, huge storage waste | Not used |
| `BYTEA` (binary blob) | Compact storage | Must serialize/deserialize manually, opaque | Not used |

**Why `FLOAT8[]` and not pgvector?** Siren computes cosine similarity **in the application layer** (in `cosine.ts`), not in the database. The embedding is stored in PostgreSQL purely for persistence — when a session is retrieved, the embedding comes back as a `number[]` and the cosine math runs in Node.js. This means:

- The database is used as a **dumb store** for the embedding
- No database-level vector similarity search is performed
- The trade-off: simpler setup, but you can't do `ORDER BY embedding <=> query_vector` in SQL

### 27.4.3 The Embedding Lifecycle

```
                         ┌─────────────────────────────────────────────┐
User types:              │    "Write my CS 401 essay on graph          │
"Write my CS 401         │     algorithms"                             │
essay on graph           └───────────────┬─────────────────────────────┘
algorithms"                              │
                                         ▼
                        ┌────────────────────────────────────┐
Step 1: LLM Expansion   │ Gemini rewrites task into a rich   │
(google.ts)             │ multi-paragraph description:       │
                        │ "computer science academic writing, │
                        │  graph theory, BFS, DFS, Dijkstra, │
                        │  minimum spanning trees..."         │
                        └───────────────┬────────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────────┐
Step 2: Embedding       │ text-embedding-004 converts the    │
(google.ts)             │ expanded text into 768 floats:     │
                        │ [0.023, -0.114, 0.892, ...]        │
                        └───────────────┬────────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────────┐
Step 3: PostgreSQL      │ INSERT INTO sessions (...,         │
INSERT (supabase.ts)    │   task_embedding)                  │
                        │ VALUES (...,                       │
                        │   ARRAY[0.023,-0.114,0.892,...]    │
                        │   ::FLOAT8[])                      │
                        └───────────────┬────────────────────┘
                                        │
                                        ▼ (on every classification request)
                        ┌────────────────────────────────────┐
Step 4: Retrieve        │ SELECT task_embedding FROM sessions│
(supabase.ts)           │ WHERE id = $1                      │
                        │ → Returns: number[768]             │
                        └───────────────┬────────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────────┐
Step 5: Cosine          │ cosineSimilarity(                  │
Similarity (cosine.ts)  │   taskEmbedding,     ← from DB    │
                        │   contentEmbedding   ← fresh      │
                        │ ) → 0.73                           │
                        └───────────────┬────────────────────┘
                                        │
                                        ▼
                        ┌────────────────────────────────────┐
Step 6: Decision        │ 0.73 > 0.55 (allow threshold)     │
(classify.ts)           │ → decision: "allow"                │
                        └────────────────────────────────────┘
```

### 27.4.4 If You Were to Redesign — pgvector Approach

In an interview, you should be able to articulate what the **next level** would look like:

```sql
-- Enable the extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Change the column type
ALTER TABLE sessions 
  ALTER COLUMN task_embedding TYPE vector(768) 
  USING task_embedding::vector;

-- Now you can do similarity search IN the database
SELECT id, task_description, 
       1 - (task_embedding <=> $1::vector) AS similarity
FROM sessions
WHERE user_id = $2
ORDER BY task_embedding <=> $1::vector
LIMIT 5;
```

This would let you do things like: "find all past sessions whose tasks were similar to this new task" — useful for suggesting related work or detecting session patterns.

---

## 27.5 Row-Level Security (RLS) — Multi-Tenant Data Isolation

Siren uses **two separate auth pathways** to the database, and understanding why is interview-critical:

### 27.5.1 The Service Role Key (Backend)

```typescript
// In supabase.ts — the backend uses the SERVICE ROLE key
supabaseClient = createClient(supabaseUrl, supabaseServiceKey, {
  auth: { autoRefreshToken: false, persistSession: false },
});
```

The Service Role key **bypasses RLS entirely**. The backend is trusted — it already verified the user's JWT in the `authenticateUser` middleware before any database call.

### 27.5.2 The RLS Policies (Defense in Depth)

Even though the backend bypasses RLS, the policies exist as defense-in-depth:

```sql
-- Sessions: users can only see their own
CREATE POLICY "Users can view own sessions" ON sessions
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own sessions" ON sessions
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own sessions" ON sessions
  FOR DELETE USING (auth.uid() = user_id);

-- Classification events: scoped through session ownership
CREATE POLICY "Users can view own events" ON classification_events
  FOR SELECT USING (
    session_id IN (SELECT id FROM sessions WHERE user_id = auth.uid())
  );
```

**The indirect RLS pattern for `classification_events`:** There's no `user_id` column on `classification_events`. Instead, the policy does a **subquery join** — "can this user see events for this session? Only if they own the session." This avoids denormalizing `user_id` onto the events table while still enforcing isolation.

**Interview talking point:** This is a common pattern in multi-tenant SaaS. You secure the leaf table through its parent's ownership rather than duplicating the tenant ID everywhere.

---

## 27.6 The `cleanup_expired_sessions()` Function

```sql
CREATE OR REPLACE FUNCTION cleanup_expired_sessions()
RETURNS INTEGER AS $$
DECLARE
  deleted_count INTEGER;
BEGIN
  DELETE FROM sessions WHERE expires_at < NOW();
  GET DIAGNOSTICS deleted_count = ROW_COUNT;
  RETURN deleted_count;
END;
$$ LANGUAGE plpgsql;
```

This is a **server-side PL/pgSQL function** that:

1. Deletes all sessions where `expires_at` has passed
2. Thanks to `ON DELETE CASCADE`, also deletes all associated `classification_events`
3. Returns the count of deleted sessions

**How it's triggered:** Designed to be called by a Supabase cron job (`pg_cron`), a database webhook, or a periodic backend task. This keeps the `sessions` table clean without the application needing to poll.

**Why not just check expiration at query time?** You could add `AND expires_at > NOW()` to every query. But:

- The `classification_events` rows for expired sessions would pile up forever
- Storage would grow unbounded
- Index performance would degrade as dead sessions accumulate

---

## 27.7 Complete Data Lifecycle — End-to-End

Here is the **complete sequence** of every SQL operation from session creation to session cleanup:

### Phase 1: Session Creation

```
User clicks "Start Session"
    → POST /api/sessions
    → Backend validates JWT (Supabase Auth)
    → Backend calls Gemini to expand task + embed it
    → INSERT INTO sessions (id, user_id, task_description, 
                            task_embedding, block_mode, expires_at)
    → 201 Created { sessionId, expiresAt }
```

### Phase 2: Classification (repeated N times per session)

```
User navigates to URL
    → POST /api/classify
    → Backend: SELECT * FROM sessions WHERE id = $1 (get embedding)
    → Backend: embed page content (Gemini API, NOT a DB call)
    → Backend: cosineSimilarity(task_embedding, content_embedding)
    → Backend: derive decision from similarity score
    → INSERT INTO classification_events (session_id, event_type, 
                                         decision, similarity, 
                                         confidence, url, title, reason)
    → 200 OK { decision, confidence, reason }
```

### Phase 3: Challenge (optional, per user action)

```
User clicks "Challenge" on a blocked page
    → POST /api/classify/challenge
    → Backend: SELECT COUNT(*) FROM classification_events 
               WHERE session_id = $1 AND url = $2 AND event_type = 'challenge'
               (check if already challenged — one challenge per URL per session)
    → Backend: LLM deep analysis of page content vs. task
    → INSERT INTO classification_events (session_id, event_type='challenge', ...)
    → 200 OK { decision, reason }
```

### Phase 4: Session End

```
Session expires (expires_at < NOW())
    → cleanup_expired_sessions() runs
    → DELETE FROM sessions WHERE expires_at < NOW()
    → CASCADE: all classification_events for those sessions are deleted
    → Returns count of deleted sessions
```

**Or the user manually ends:**

```
User clicks "End Session"
    → DELETE /api/sessions/:id
    → Backend: DELETE FROM sessions WHERE id = $1
    → CASCADE: classification_events deleted
    → 200 OK
```

---

## 27.8 What Supabase Adds Beyond Raw PostgreSQL

For the interview, understand that Supabase is not just Postgres:

| Supabase Layer | What It Does | How Siren Uses It |
|----------------|-------------|-------------------|
| **PostgREST** | Auto-generates REST API from DB schema | The `@supabase/supabase-js` client calls PostgREST, not raw SQL. `.from('sessions').insert({...})` becomes a REST call that PostgREST converts to SQL. |
| **GoTrue (Auth)** | JWT issuance, Google OAuth, user management | Issues the JWTs that the Chrome extension sends to the Express backend. |
| **Realtime** | WebSocket-based change notifications | Not used by Siren (frontend polls via REST). |
| **Storage** | S3-compatible file/blob storage | Not used by Siren (no file uploads). |
| **Edge Functions** | Deno-based serverless functions | Not used (Siren has its own Express backend on Railway). |

**The PostgREST transform chain:**

```
supabase.from('sessions').insert({ user_id: '...',  task_embedding: [...] })
         ↓
PostgREST translates to:
POST /rest/v1/sessions
Content-Type: application/json
Body: { "user_id": "...", "task_embedding": [...] }
         ↓
PostgREST generates:
INSERT INTO sessions (user_id, task_embedding, ...) 
VALUES ($1, $2::FLOAT8[], ...)
         ↓
PostgreSQL executes the INSERT
```

This is why the Supabase JS client handles type coercion (JS array → PG FLOAT8[]) automatically — PostgREST does the heavy lifting.

---

## 27.9 Interview Cheat Sheet — Key Talking Points

1. **"How do you store embeddings?"** — FLOAT8[] native PG array. 768 floats from text-embedding-004. Not pgvector because similarity is computed app-side in Node.js, not in SQL.

2. **"Why two tables not five?"** — Sessions + events. Discriminated nullable columns on events table (url/title for web, app_name/window_title for desktop). Denormalized for query simplicity.

3. **"How does JSON become SQL?"** — camelCase TS → snake_case PG mapping in the Supabase client layer. `number[]` → `FLOAT8[]` via PostgREST auto-coercion. `Date.toISOString()` → `TIMESTAMPTZ`.

4. **"How do you enforce multi-tenancy?"** — RLS policies on both tables. Events are scoped through session ownership via subquery. Backend uses Service Role key (bypasses RLS) but validates JWT before any DB call.

5. **"What happens when the DB is down?"** — Classification event logging is fire-and-forget (don't throw). Session retrieval failure = fail-open (allow). The system prioritizes user experience over perfect audit logging.

6. **"How would you scale this?"** — Add pgvector for embedding similarity search. Partition classification_events by created_at for time-range queries. Add connection pooling (PgBouncer) for high concurrency.
