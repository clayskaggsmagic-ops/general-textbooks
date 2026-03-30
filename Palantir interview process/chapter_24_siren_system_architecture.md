# Chapter 24 — Siren (Ithaca): System Architecture & Information Flow

> **Interview Context**: This chapter reverse-engineers every data path in the Siren focus-guard system. You should be able to whiteboard the entire flow from "user types a task description" to "overlay blocks Reddit" with exact function names, data structures, and the mathematical formula that decides the outcome.

---

## 24.1 — System Overview

Siren (publicly branded "Ithaca") is an **AI-powered focus guard** that monitors a user's browsing and decides, in real time, whether each page is relevant to their declared work task. It is a three-tier system:

```
┌───────────────────────────────────────────────────────────────┐
│  TIER 1: Chrome Extension (Manifest V3)                      │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │  Popup   │  │  Background  │  │    Content Script       │  │
│  │ (React-  │  │  Service     │  │  (Readability +         │  │
│  │  like UI)│  │  Worker      │  │   Overlay injection)    │  │
│  └──────────┘  └──────────────┘  └────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
        │ HTTPS + Bearer JWT              │ Message passing
        ▼                                 ▼
┌───────────────────────────────────────────────────────────────┐
│  TIER 2: Express Backend (TypeScript, Railway)               │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │  Session  │  │  Classify    │  │    Auth Middleware      │  │
│  │  Routes   │  │  Routes      │  │  (jose HS256 JWT)      │  │
│  └──────────┘  └──────────────┘  └────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
        │ Supabase SDK             │ Google AI SDK
        ▼                         ▼
┌──────────────────┐    ┌──────────────────────────────────────┐
│  Supabase        │    │  Google Gemini API                   │
│  ┌────────────┐  │    │  ┌──────────────────┐               │
│  │ sessions   │  │    │  │ text-embedding-  │               │
│  │ classifica-│  │    │  │ 004              │               │
│  │ tion_events│  │    │  │ gemini-2.0-flash │               │
│  │ auth.users │  │    │  └──────────────────┘               │
│  └────────────┘  │    └──────────────────────────────────────┘
└──────────────────┘
     TIER 3: Data & AI Layer
```

**Key Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Extension architecture | Manifest V3 | Required by Chrome 2024+; uses service workers, not persistent background pages |
| Backend hosting | Railway | One-command deploys, auto-TLS, persistent process (not serverless — avoids cold starts on classification) |
| Auth | Supabase Auth → JWT → Backend verification | Extension uses `chrome.identity.launchWebAuthFlow` for Google OAuth; backend verifies HS256 JWTs via `jose` |
| Embedding model | `text-embedding-004` (768-dim) | Google's latest text embedding; fast, free-tier friendly, outperforms `gecko` |
| LLM model | `gemini-2.0-flash-lite` / `gemini-2.0-flash` | Sub-second latency for real-time classification; flash-lite for cheap calls, flash for deeper analysis |
| Database | Supabase (PostgreSQL) | Managed Postgres; Service Role key for server-side ops; built-in Auth integration |
| Content extraction | Mozilla Readability | Battle-tested article extraction; strips ads/nav; used by Firefox Reader View |

---

## 24.2 — The Session Lifecycle (Start to End)

A "session" represents one focused work period. Here is every step, with the exact function that executes it.

### Phase 1: Task Analysis (Interactive Refinement Loop)

```
User types: "Working on my machine learning homework"
                    │
                    ▼
    ┌───────────────────────────────┐
    │ Frontend: popup.js            │
    │ Calls: analyzeTask(desc, [])  │
    └───────────────┬───────────────┘
                    │ POST /v1/session/analyze-task
                    ▼
    ┌───────────────────────────────┐
    │ Backend: session.ts           │
    │ router.post('/analyze-task')  │
    │                               │
    │ 1. Validate via Zod:          │
    │    AnalyzeTaskSchema.parse()  │
    │    - taskDescription: string  │
    │    - previousResponses?: []   │
    │                               │
    │ 2. Call analyzeTaskWithAI()   │
    │    from google.ts             │
    └───────────────┬───────────────┘
                    │
                    ▼
    ┌───────────────────────────────┐
    │ google.ts: analyzeTaskWithAI()│
    │                               │
    │ System prompt instructs LLM:  │
    │ "You are a task-analysis      │
    │  assistant. Evaluate if the   │
    │  task description is specific │
    │  enough for semantic matching.│
    │  Output JSON:                 │
    │  {                            │
    │    ready: boolean,            │
    │    durationMinutes?: number,  │
    │    followUpQuestions?: [],     │
    │    refinedTask?: string       │
    │  }"                           │
    │                               │
    │ Model: gemini-2.0-flash-lite  │
    │ Temperature: 0.1              │
    │ Response MIME: application/   │
    │               json            │
    └───────────────┬───────────────┘
                    │
           ┌────────┴────────┐
           │                 │
      ready: false      ready: true
           │                 │
           ▼                 ▼
    Follow-up Qs        Proceed to
    sent to popup       session start
    (loop continues)
```

**What the LLM decides**: Is "Working on my machine learning homework" specific enough to generate an embedding that will correctly distinguish on-task vs. off-task pages? If not, it returns follow-up questions like "Which topic — regression, neural networks, or NLP?" The user answers, the answers are appended to `previousResponses[]`, and the loop repeats.

**The data structure for the loop**:

```typescript
// Request body (Zod-validated)
{
    taskDescription: "Working on my machine learning homework",
    previousResponses: [
        { question: "What specific topic?", answer: "neural networks" },
        { question: "Which framework?", answer: "PyTorch" }
    ]
}

// Response when ready
{
    ready: true,
    refinedTask: "Working on neural network homework using PyTorch",
    durationMinutes: 120
}
```

### Phase 2: Task Expansion (Critical Enrichment Step)

This is **the most architecturally important step** and the one most interviewers would probe.

```
     "neural network homework using PyTorch"
                    │
                    ▼
    ┌───────────────────────────────┐
    │ google.ts: expandTaskDesc()   │
    │                               │
    │ System prompt:                │
    │ "Expand this task description │
    │  into a rich, detailed para-  │
    │  graph. Include related tools,│
    │  concepts, technologies,      │
    │  websites that would be       │
    │  relevant. This description   │
    │  will be used for semantic    │
    │  matching, so include many    │
    │  related terms."              │
    │                               │
    │ Model: gemini-2.0-flash-lite  │
    │ Max tokens: 1024              │
    └───────────────┬───────────────┘
                    │
                    ▼
    Output: "Working on neural network homework using
    PyTorch deep learning framework. Related topics
    include backpropagation, gradient descent, loss
    functions, convolutional neural networks (CNN),
    recurrent neural networks (RNN), LSTM, transformers,
    tensor operations, CUDA, GPU computing, Jupyter
    notebooks, Google Colab, torchvision, academic
    papers on arxiv.org, Stack Overflow PyTorch
    questions, PyTorch documentation..."
```

**Why this matters**: A raw task description of "PyTorch homework" would have a narrow embedding. The expanded description creates a **semantic neighborhood** — when the user visits `pytorch.org/docs`, the cosine similarity between the expanded task embedding and the docs page embedding will be high (~0.7+), because the expansion already mentioned "PyTorch documentation" explicitly.

**Without expansion**: `cos("PyTorch homework", "pytorch.org/docs/stable/nn.html") ≈ 0.45` → WARN
**With expansion**: `cos("... PyTorch documentation, tensor operations ...", "pytorch.org/docs/stable/nn.html") ≈ 0.72` → ALLOW

### Phase 3: Embedding Generation

```
    Expanded task description (paragraph)
                    │
                    ▼
    ┌───────────────────────────────┐
    │ google.ts: generateEmbedding()│
    │                               │
    │ Model: text-embedding-004     │
    │ Output: Float32[768]          │
    │ (768-dimensional vector)      │
    │                               │
    │ The embedding is a point in   │
    │ 768-dim semantic space.       │
    └───────────────┬───────────────┘
                    │
                    ▼
    taskEmbedding: [0.0124, -0.0567, 0.0891, ...]
    (768 floats, normalized to unit length)
```

### Phase 4: Session Persistence

```
    ┌───────────────────────────────┐
    │ session.ts: startSession()    │
    │                               │
    │ 1. Generate UUID v4           │
    │ 2. Store in Supabase:         │
    │                               │
    │ supabase.createSession({      │
    │   id: "a1b2c3...",            │
    │   userId: req.user.sub,       │
    │   taskDescription: expanded,  │
    │   taskEmbedding: Float32[768],│
    │   blockMode: "warn" | "block",│
    │   expiresAt: now + duration   │
    │ })                            │
    │                               │
    │ 3. Return to extension:       │
    │ {                             │
    │   sessionId: "a1b2c3...",     │
    │   taskDescription: expanded,  │
    │   blockMode: "warn",          │
    │   expiresAt: ISO8601,         │
    │   startedAt: ISO8601          │
    │ }                             │
    └───────────────────────────────┘
```

**Supabase `sessions` table schema** (inferred from `SessionRow`):

| Column | Type | Notes |
|--------|------|-------|
| `id` | `uuid` PRIMARY KEY | Generated server-side |
| `user_id` | `uuid` NOT NULL | FK to `auth.users(id)` |
| `task_description` | `text` | The **expanded** description, not the raw input |
| `task_embedding` | `float8[]` | PostgreSQL array of 768 doubles |
| `block_mode` | `text` CHECK | `'warn'` or `'block'` |
| `created_at` | `timestamptz` | Auto-set |
| `expires_at` | `timestamptz` | Server-calculated expiry |

**Frontend storage** (Chrome extension `chrome.storage.local`):

```javascript
// storage.js STORAGE_KEYS
{
    supabaseSession: { access_token, refresh_token, ... },
    activeSession: {
        sessionId: "a1b2c3...",
        taskDescription: "...",
        blockMode: "warn",
        expiresAt: "2025-01-15T20:00:00Z",
        startedAt: "2025-01-15T18:00:00Z",
        pomodoro: { enabled: true, workMin: 25, breakMin: 5 }
    },
    sessionStats: {
        pagesChecked: 47,
        pagesAllowed: 38,
        pagesWarned: 6,
        pagesBlocked: 3
    },
    recentUrls: {
        "https://reddit.com": {
            timestamp: 1705341234567,
            result: { decision: "block", reason: "...", confidence: 0.85 }
        }
    }
}
```

---

## 24.3 — The Classification Pipeline (Page Visit → Decision)

This is the core of the system. When a user navigates to any page, here is the exact sequence:

### Step 1: Tab Event Detection (Frontend)

```javascript
// background.js
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
    // Guard clauses (in this exact order):
    if (changeInfo.status !== 'complete') return;     // Only on full load
    if (!tab.url) return;                              // Must have URL
    currentTabUrls.set(tabId, tab.url);                // Race condition tracker
    if (!tab.url.startsWith('http')) return;            // Skip chrome://, about:, etc.
    if (shouldSkipUrl(tab.url)) return;                // Skip browser-internal URLs
    const token = await getAccessToken();              // Must be logged in
    if (!token) return;
    const session = await getActiveSession();          // Must have active session
    if (!session) return;
    if (new Date(session.expiresAt) <= new Date()) {   // Session expired check
        await clearActiveSession();
        return;
    }
    // ... proceed to classification
});
```

**The `shouldSkipUrl` function** (config.js) checks against regex patterns:
```javascript
const SKIP_URL_PATTERNS = [
    /^chrome:\/\//,             // Chrome settings pages
    /^chrome-extension:\/\//,   // Other extensions
    /^about:/,                  // about:blank, about:newtab
    /^edge:\/\//,               // Edge browser pages
    /^brave:\/\//,              // Brave browser pages
    /^moz-extension:\/\//,      // Firefox extensions
    /^file:\/\//,               // Local files
];
```

### Step 2: URL Debouncing / Cache Check

```
    tab.url = "https://reddit.com/r/programming"
                    │
                    ▼
    ┌───────────────────────────────┐
    │ storage.js:                   │
    │ isRecentlyClassified(url,     │
    │   CONFIG.CLASSIFICATION_      │
    │   DEBOUNCE_MS = 30000)        │
    │                               │
    │ Checks chrome.storage.local   │
    │ for recentUrls[url]:          │
    │                               │
    │ If cached AND < 30s old:      │
    │   → Return cached result      │
    │   → Skip API call entirely    │
    │   → Re-apply overlay if block │
    │                               │
    │ If expired or missing:        │
    │   → Return null               │
    │   → Proceed to classification │
    └───────────────────────────────┘
```

**Cache data structure** (recentUrls in `chrome.storage.local`):
```javascript
{
    "https://reddit.com/r/programming": {
        timestamp: 1705341234567,     // Date.now() at classification
        result: {
            decision: "warn",         // allow | warn | block
            reason: "Social media...",
            confidence: 0.65
        }
    },
    "https://pytorch.org/docs/": {
        timestamp: 1705341200000,
        result: {
            decision: "allow",
            reason: "Relevant to task",
            confidence: 0.92
        }
    }
}
```

Cleanup runs every 30 seconds, evicting entries older than 60 seconds:
```javascript
setInterval(async () => {
    await cleanupOldUrls(60000);
}, 30000);
```

### Step 3: Pomodoro Break Check

Before any classification, the system checks if the user is currently in a Pomodoro break:

```javascript
function checkPomodoroBreak(session) {
    if (!session.pomodoro?.enabled) return false;
    
    const elapsedMinutes = (Date.now() - new Date(session.startedAt).getTime()) / 60000;
    const cycleDuration = session.pomodoro.workMin + session.pomodoro.breakMin;
    const cyclePosition = elapsedMinutes % cycleDuration;
    
    // Break starts after workMin elapsed in each cycle
    return cyclePosition >= session.pomodoro.workMin;
}
```

**Example**: Work=25min, Break=5min. At minute 27, `cyclePosition = 27 % 30 = 27`, which is `>= 25` → **in break**. At minute 31, `cyclePosition = 31 % 30 = 1`, which is `< 25` → **working**.

During a break, all pages are auto-allowed without an API call.

### Step 4: Content Extraction (Content Script)

```
    background.js sends:
    chrome.tabs.sendMessage(tabId, { type: 'EXTRACT_CONTENT' })
                    │
                    ▼
    ┌───────────────────────────────┐
    │ content.js: extractPageContent│
    │                               │
    │ 1. Clone the DOM:             │
    │    document.cloneNode(true)   │
    │    (required: Readability     │
    │     mutates the DOM)          │
    │                               │
    │ 2. Parse with Readability:    │
    │    new Readability(clone)     │
    │    .parse()                   │
    │    → { textContent, title }   │
    │                               │
    │ 3. Fallback:                  │
    │    document.body.innerText    │
    │    (if Readability fails)     │
    │                               │
    │ 4. Clean + truncate:          │
    │    .replace(/\s+/g, ' ')     │
    │    .substring(0, 15000)       │
    │                               │
    │ Returns:                      │
    │ {                             │
    │   title: "Intro to CNNs",     │
    │   pageText: "Convolutional..."│
    │ }                             │
    └───────────────────────────────┘
```

**Why Readability matters**: Without it, you'd extract navigation menus, footer links, cookie banners, and sidebar ads along with the article text. Readability isolates the **main content**, which dramatically improves embedding quality. The same library powers Firefox's built-in Reader View.

### Step 5: The 5-Point Race Condition Guard

This is a **critical engineering detail** that demonstrates production thinking. Between content extraction and API response, the user might navigate away. The system checks at **five different checkpoints**:

```
Race Check #1: BEFORE content extraction
    → Is currentTabUrls.get(tabId) still === tab.url?

Race Check #2: BEFORE sendMessage (content script)
    → Same check

Race Check #3: ON sendMessage ERROR
    → If the error is bfcache/channel closed, abort silently

Race Check #4: BEFORE API call (most expensive step)
    → Same URL check

Race Check #5: AFTER API response returns
    → Before applying overlay, verify URL hasn't changed
```

**Why this matters for interviews**: Without these guards, the system would:
1. Start classifying `reddit.com`
2. User navigates to `pytorch.org` during the API call (~500ms)
3. API returns "block reddit"
4. System injects a blocking overlay onto `pytorch.org` ← **wrong page blocked**

The `currentTabUrls` Map provides O(1) lookups for the latest URL per tab, preventing stale classifications from affecting the wrong page.

### Step 6: The API Call (Backend Classification)

```
    POST /v1/classify/web
    Authorization: Bearer <supabase_jwt>
    
    {
        "sessionId": "a1b2c3...",
        "url": "https://reddit.com/r/programming",
        "title": "r/programming - Reddit",
        "pageText": "Programming community discussion about..."
    }
```

This triggers the **multi-tier classification engine** (detailed in Chapter 25).

---

## 24.4 — The Authentication Architecture

### Extension-Side OAuth Flow

```
    User clicks "Sign in with Google"
                    │
                    ▼
    ┌───────────────────────────────┐
    │ supabase.js: signInWithGoogle()│
    │                               │
    │ 1. Get redirect URL:          │
    │    chrome.identity             │
    │      .getRedirectURL()        │
    │    → "https://<ext-id>        │
    │       .chromiumapp.org/"      │
    │                               │
    │ 2. Generate OAuth URL:        │
    │    supabase.auth.signIn       │
    │      WithOAuth({              │
    │        provider: 'google',    │
    │        skipBrowserRedirect:   │
    │          true,                │
    │        redirectTo: redirectUrl│
    │      })                       │
    │                               │
    │ 3. Launch Chrome auth flow:   │
    │    chrome.identity             │
    │      .launchWebAuthFlow({     │
    │        url: oauthUrl,         │
    │        interactive: true      │
    │      })                       │
    │                               │
    │ 4. Parse callback URL hash:   │
    │    #access_token=xxx&         │
    │     refresh_token=yyy         │
    │                               │
    │ 5. Set Supabase session:      │
    │    supabase.auth.setSession({ │
    │      access_token,            │
    │      refresh_token            │
    │    })                         │
    └───────────────────────────────┘
```

**Custom storage adapter**: The Supabase client can't use `localStorage` in a Chrome extension service worker (no DOM access). Instead, it uses a custom adapter backed by `chrome.storage.local`:

```javascript
const chromeStorageAdapter = {
    getItem: async (key) => {
        const session = await getSupabaseSession();
        return session ? JSON.stringify(session) : null;
    },
    setItem: async (key, value) => {
        const session = JSON.parse(value);
        await setSupabaseSession(session);
    },
    removeItem: async () => {
        await clearSupabaseSession();
    },
};
```

### Backend JWT Verification

```
    Incoming request with header:
    Authorization: Bearer eyJhbGciOi...
                    │
                    ▼
    ┌───────────────────────────────┐
    │ auth.ts: authMiddleware()     │
    │                               │
    │ 1. Extract Bearer token       │
    │ 2. Call verifySupabaseJWT():  │
    │                               │
    │    jose.jwtVerify(token,      │
    │      secretKey, {             │
    │        issuer: "https://      │
    │          <project>.supabase   │
    │          .co/auth/v1",        │
    │        audience: "authenti-   │
    │          cated",              │
    │        algorithms: ["HS256"]  │
    │      })                       │
    │                               │
    │ 3. Decoded payload:           │
    │    {                          │
    │      sub: "user-uuid",        │
    │      email: "user@...",       │
    │      aud: "authenticated",    │
    │      exp: 1705345000,         │
    │      role: "authenticated"    │
    │    }                          │
    │                               │
    │ 4. Attach to request:         │
    │    req.user = payload         │
    └───────────────────────────────┘
```

**Key detail**: Supabase uses **symmetric HS256 signing** (shared secret), not asymmetric RS256/JWKS. This means the backend needs the `SUPABASE_JWT_SECRET` environment variable (the secret from the Supabase dashboard) to verify tokens. This is simpler but requires the secret to be kept strictly server-side.

**Error handling taxonomy**:

| Error Class | HTTP Code | User Message |
|-------------|-----------|--------------|
| `JWTExpired` | 401 | "Token has expired" |
| `JWTClaimValidationFailed` | 401 | "Token validation failed: ..." |
| `JWSSignatureVerificationFailed` | 401 | "Invalid token signature" |
| Missing header | 401 | "Missing Authorization header" |
| Malformed header | 401 | "Invalid Authorization header format" |

---

## 24.5 — Server Configuration & Security

### CORS Policy

```typescript
// server.ts
app.use(cors({
    origin: [
        'chrome-extension://*',        // Any Chrome extension
        'http://localhost:*',           // Local development
        'https://ithacafocus.com',      // Production website
        'https://www.ithacafocus.com',  // www variant
    ],
    credentials: true,
}));
```

**Interview insight**: The `chrome-extension://*` wildcard is necessary because Chrome extension IDs change between development and production. In a hardened production system, you'd pin to the specific extension ID.

### Rate Limiting

```typescript
app.use(rateLimit({
    windowMs: 60 * 1000,     // 1-minute window
    max: 120,                 // 120 requests per minute per IP
    message: { error: 'Too many requests' },
    standardHeaders: true,    // RateLimit-* headers
}));
```

**Why 120/min**: A power user with 30+ tabs might trigger 60+ classifications per minute during heavy browsing. The rate limit is generous enough for legitimate use but blocks brute-force abuse.

### Request Validation (Zod)

Every endpoint validates its input with Zod schemas before processing:

```typescript
// types.ts - Example schemas

const ClassifyWebSchema = z.object({
    sessionId: z.string().uuid(),
    url: z.string().url(),
    title: z.string().max(500),
    pageText: z.string().max(15000),
});

const StartSessionSchema = z.object({
    taskDescription: z.string().min(3).max(2000),
    blockMode: z.enum(['warn', 'block']),
    durationMinutes: z.number().positive().optional(),
    pomodoroSettings: z.object({
        enabled: z.boolean(),
        workMin: z.number().min(1).max(120),
        breakMin: z.number().min(1).max(60),
    }).optional().nullable(),
    strictness: z.number().min(0).max(1).optional(),
});
```

---

## 24.6 — API Surface Reference

| Method | Endpoint | Auth | Purpose | Input (key fields) | Output |
|--------|----------|------|---------|---------------------|--------|
| POST | `/v1/session/analyze-task` | JWT | Evaluate task clarity | `taskDescription`, `previousResponses[]` | `{ ready, refinedTask?, followUpQuestions? }` |
| POST | `/v1/session/start` | JWT | Create new session | `taskDescription`, `blockMode`, `durationMinutes?`, `pomodoroSettings?`, `strictness?` | `{ sessionId, taskDescription, expiresAt }` |
| POST | `/v1/session/end` | JWT | Terminate session | `sessionId` | `{ success: true }` |
| POST | `/v1/classify/web` | JWT | Classify a web page | `sessionId`, `url`, `title`, `pageText` | `{ decision, confidence, similarity, reason }` |
| POST | `/v1/classify/challenge` | JWT | Appeal a block | `sessionId`, `url`, `title`, `pageText` | `{ decision, confidence, reason }` |
| GET | `/health` | None | Health check | — | `{ status: "ok", timestamp, uptime }` |

---

## 24.7 — Supabase Data Model

### Table: `sessions`

```sql
CREATE TABLE sessions (
    id          UUID PRIMARY KEY,
    user_id     UUID NOT NULL REFERENCES auth.users(id),
    task_description TEXT NOT NULL,
    task_embedding   FLOAT8[] NOT NULL,   -- 768-element vector
    block_mode       TEXT NOT NULL CHECK (block_mode IN ('warn','block')),
    created_at       TIMESTAMPTZ DEFAULT NOW(),
    expires_at       TIMESTAMPTZ NOT NULL
);
```

### Table: `classification_events`

```sql
CREATE TABLE classification_events (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id  UUID NOT NULL REFERENCES sessions(id),
    event_type  TEXT NOT NULL CHECK (event_type IN ('web','app','screen','challenge')),
    decision    TEXT NOT NULL CHECK (decision IN ('allow','warn','block')),
    similarity  FLOAT NOT NULL,
    confidence  FLOAT NOT NULL,
    url         TEXT,
    title       TEXT,
    app_name    TEXT,
    window_title TEXT,
    reason      TEXT NOT NULL,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

**Event types**:
- `web`: Standard page classification
- `app`: Desktop application classification (future)
- `screen`: Screenshot-based classification (future)
- `challenge`: User appealed a block/warn decision

**Logging is fire-and-forget**: In `supabase.ts`, `logClassificationEvent()` catches errors silently and never throws. A failed log write must never prevent a classification from completing.

```typescript
export async function logClassificationEvent(event) {
    const { error } = await client.from('classification_events').insert(event);
    if (error) {
        console.error('[SUPABASE] Error logging event:', error);
        return; // Don't throw - logging failures shouldn't break classification
    }
}
```

---

*Next chapter: The multi-tier AI classification engine — embedding similarity, productivity tool detection, and the challenge/appeals system.*
