# Chapter 30: Siren: The Multi-Tier Classification Engine

> **Interview Context**: This chapter traces *every single data transformation* from the moment a URL enters the classification pipeline to the moment a decision is rendered. You should be able to draw the exact JSON payload at every boundary crossing, name the mathematical formula that produces the decision, and explain every edge case the system handles.

---

## 25.1 — Classification Entry Point: Exact Data In, Exact Data Out

When the background service worker calls the API, here is the **exact HTTP request** on the wire:

```http
POST /v1/classify/web HTTP/1.1
Host: siren-backend-production.up.railway.app
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJmNDdh
  YzEyMy0xMjM0LTU2NzgtYWJjZC0xMjM0NTY3ODkwMTIiLCJlbWFpbCI6InVzZXJAZXhhbXBs
  ZS5jb20iLCJhdWQiOiJhdXRoZW50aWNhdGVkIiwiaXNzIjoiaHR0cHM6Ly90dG93dHhuYW10
  YnNwbWpxcWNwcS5zdXBhYmFzZS5jby9hdXRoL3YxIiwiaWF0IjoxNzA1MzQxMjAwLCJleHAi
  OjE3MDUzNDQ4MDB9.signature_bytes

{
    "sessionId": "f47ac123-1234-5678-abcd-123456789012",
    "url": "https://www.reddit.com/r/programming/comments/abc123/understanding_transformers",
    "title": "Understanding Transformers - r/programming",
    "pageText": "Transformers are a type of neural network architecture that uses self-attention mechanisms. The original paper 'Attention Is All You Need' by Vaswani et al. introduced the concept in 2017. Key components include multi-head attention, positional encoding, and feed-forward layers. Modern implementations include BERT, GPT, and T5..."
}
```

**Zod validation** — the first thing that happens server-side. The raw body is validated against `ClassifyWebSchema`:

```typescript
// types.ts — exact schema definition
export const ClassifyWebSchema = z.object({
    sessionId: z.string().uuid(),          // Must be valid UUID format
    url: z.string().url(),                 // Must be valid URL
    title: z.string().max(500),            // Capped at 500 chars
    pageText: z.string().max(15000),       // Capped at 15,000 chars
});

// If validation fails, response is:
// HTTP 400
{
    "success": false,
    "error": "Validation failed",
    "details": [
        {
            "code": "too_big",
            "maximum": 15000,
            "type": "string",
            "inclusive": true,
            "exact": false,
            "message": "String must contain at most 15000 character(s)",
            "path": ["pageText"]
        }
    ]
}
```

**After validation, the request hits `classify.ts` → `handleWebClassification()`**. Let's trace every line.

---

## 25.2 — Step-by-Step Data Flow Through `handleWebClassification`

### Step 1: Session Retrieval from Supabase

```typescript
// classify.ts, line ~60
const session = await getSessionById(req.body.sessionId);
```

**The Supabase query** (what actually hits PostgreSQL):

```sql
SELECT * FROM sessions WHERE id = 'f47ac123-1234-5678-abcd-123456789012' LIMIT 1;
```

**The returned `SessionRow` object** (exact shape from `supabase.ts`):

```typescript
const session: SessionRow = {
    id: "f47ac123-1234-5678-abcd-123456789012",
    user_id: "a1b2c3d4-5678-90ab-cdef-123456789abc",
    task_description: "Working on neural network homework using PyTorch deep learning 
        framework. Related topics include backpropagation, gradient descent, loss 
        functions, convolutional neural networks (CNN), recurrent neural networks (RNN), 
        LSTM, transformers, tensor operations, CUDA, GPU computing, Jupyter notebooks, 
        Google Colab, torchvision, academic papers on arxiv.org, Stack Overflow PyTorch 
        questions, PyTorch documentation, matplotlib for visualization, numpy for 
        numerical computing, training loops, model evaluation, cross-entropy loss, 
        Adam optimizer, learning rate scheduling...",
    task_embedding: [0.0124, -0.0567, 0.0891, 0.0234, -0.0123, ...],  // Float64[768]
    block_mode: "warn",
    created_at: "2025-01-15T18:00:00.000Z",
    expires_at: "2025-01-15T20:00:00.000Z"
};
```

**Failure modes at this step**:
- `session === null` → HTTP 404: `{ success: false, error: "Session not found" }`
- `session.expires_at < now` → HTTP 400: `{ success: false, error: "Session has expired" }`

### Step 2: Content Embedding Generation

```typescript
// classify.ts, line ~80
const contentEmbedding = await generateEmbedding(
    `${title} ${pageText}`.substring(0, 10000)
);
```

**Critical data transformation**: The title and page text are **concatenated** into a single string, then truncated to 10,000 characters. This combined string is what gets embedded — the title provides strong signal about the page's topic.

**Exact input to Google's API**:

```
"Understanding Transformers - r/programming Transformers are a type of neural 
network architecture that uses self-attention mechanisms. The original paper 
'Attention Is All You Need' by Vaswani et al. introduced the concept in 2017..."
```

**The `generateEmbedding()` function** (google.ts):

```typescript
export async function generateEmbedding(text: string): Promise<number[]> {
    const model = genAI.getGenerativeModel({ model: 'text-embedding-004' });
    
    const result = await model.embedContent(text);
    
    // result.embedding.values is a Float32Array of length 768
    return result.embedding.values;
}
```

**The API call to Google** (under the hood):

```http
POST https://generativelanguage.googleapis.com/v1/models/text-embedding-004:embedContent
x-goog-api-key: AIzaSy...

{
    "content": {
        "parts": [{ "text": "Understanding Transformers - r/programming..." }]
    }
}
```

**The response** (768 floating-point numbers):

```json
{
    "embedding": {
        "values": [
            0.0187, -0.0423, 0.0756, 0.0112, -0.0334, 0.0891, -0.0267, 0.0445,
            0.0623, -0.0189, 0.0534, 0.0078, -0.0412, 0.0267, -0.0156, 0.0398,
            // ... 752 more values ...
            0.0234, -0.0567, 0.0123
        ]
    }
}
```

**Data shape at this point**:

```
taskEmbedding:    Float64[768]  (from Supabase, stored at session creation)
contentEmbedding: Float32[768]  (just generated from page content)
```

### Step 3: Cosine Similarity Computation

This is **the mathematical core** of the entire system.

```typescript
// cosine.ts — exact implementation
export function cosineSimilarity(a: number[], b: number[]): number {
    // Dimension check
    if (a.length !== b.length) {
        throw new Error(`Vector dimension mismatch: ${a.length} vs ${b.length}`);
    }

    // Dot product: sum(a[i] * b[i])
    const dot = dotProduct(a, b);
    
    // Magnitudes: sqrt(sum(a[i]^2))
    const magA = magnitude(a);
    const magB = magnitude(b);

    // Zero vector guard
    if (magA === 0 || magB === 0) {
        throw new Error('Cannot compute cosine similarity for zero vectors');
    }

    // cos(θ) = (A · B) / (|A| × |B|)
    return dot / (magA * magB);
}
```

**Worked example with real-ish numbers**:

Given two 4-dimensional vectors (simplified from 768-dim):
```
task    = [0.5,  0.3,  0.8,  0.1]    (PyTorch homework)
content = [0.4,  0.2,  0.7,  0.05]   (Transformer article on Reddit)

dot = (0.5×0.4) + (0.3×0.2) + (0.8×0.7) + (0.1×0.05)
    = 0.20 + 0.06 + 0.56 + 0.005
    = 0.825

|task|    = √(0.25 + 0.09 + 0.64 + 0.01) = √0.99 = 0.995
|content| = √(0.16 + 0.04 + 0.49 + 0.0025) = √0.6925 = 0.832

cos(θ) = 0.825 / (0.995 × 0.832) = 0.825 / 0.828 = 0.997
```

In practice with 768 dimensions, typical values are:
- **Same topic**: 0.65 – 0.85
- **Loosely related**: 0.40 – 0.65
- **Unrelated**: 0.15 – 0.40
- **Completely different domains**: 0.05 – 0.15

### Step 4: Threshold-Based Decision

```typescript
// cosine.ts — exact thresholds
export const SIMILARITY_THRESHOLDS = {
    allow: 0.55,    // >= 0.55 → definitely on-task
    warn: 0.4,      // >= 0.40 but < 0.55 → grey zone
                     //  < 0.40 → off-task
} as const;

export function getDecision(similarity: number): Decision {
    if (similarity >= SIMILARITY_THRESHOLDS.allow) return 'allow';
    if (similarity >= SIMILARITY_THRESHOLDS.warn)  return 'warn';
    return 'block';
}
```

**The decision spectrum**:

```
Similarity:  0.0         0.4         0.55         1.0
             ├───────────┼───────────┼────────────┤
             │   BLOCK   │   WARN    │   ALLOW    │
             │           │ grey zone │            │
             │< off-task │ uncertain │ on-task   >│
```

### Step 5: Confidence Calculation

Confidence is **not the same as similarity**. It measures how far the similarity score is from the nearest threshold — i.e., *how sure we are about the decision*.

```typescript
// cosine.ts — exact confidence formula
export function classifyBySimilarity(taskEmbedding, contentEmbedding) {
    const similarity = cosineSimilarity(taskEmbedding, contentEmbedding);
    const decision = getDecision(similarity);

    let confidence: number;

    if (decision === 'allow') {
        // Farther above 0.55 → higher confidence
        // At similarity=0.55: confidence=0.50
        // At similarity=1.00: confidence=1.00
        confidence = Math.min(1, 
            (similarity - 0.55) / (1 - 0.55) * 0.5 + 0.5
        );
    } else if (decision === 'block') {
        // Farther below 0.40 → higher confidence
        // At similarity=0.40: confidence=0.50
        // At similarity=0.00: confidence=1.00
        confidence = Math.min(1, 
            (0.4 - similarity) / 0.4 * 0.5 + 0.5
        );
    } else {
        // Warn zone: inherently low confidence
        // Peaks at the midpoint between thresholds (0.475)
        const midpoint = (0.55 + 0.4) / 2;  // = 0.475
        confidence = 0.3 + Math.abs(similarity - midpoint) * 0.4;
    }

    return {
        decision,
        similarity,                                    // Raw cosine sim
        confidence: Math.round(confidence * 100) / 100 // Rounded to 2dp
    };
}
```

**Worked examples**:

| Similarity | Decision | Confidence | Why |
|-----------|----------|------------|-----|
| 0.82 | ALLOW | 0.80 | `(0.82-0.55)/(1-0.55)*0.5+0.5 = 0.30/0.45*0.5+0.5 = 0.83` → rounds to 0.80 |
| 0.57 | ALLOW | 0.52 | Just above threshold, low confidence |
| 0.48 | WARN | 0.30 | Near midpoint of warn zone, minimum confidence |
| 0.42 | WARN | 0.32 | Edge of warn zone |
| 0.35 | BLOCK | 0.56 | `(0.4-0.35)/0.4*0.5+0.5 = 0.125*0.5+0.5 = 0.56` |
| 0.10 | BLOCK | 0.88 | `(0.4-0.10)/0.4*0.5+0.5 = 0.75*0.5+0.5 = 0.88` |
| 0.00 | BLOCK | 1.00 | Maximum confidence: completely unrelated |

---

## 25.3 — The Productivity Tool Detection Layer (LLM Bypass)

**Before** the embedding similarity check runs, there is a critical pre-check: **Is this page a productivity tool?**

### The Problem This Solves

Consider: A user's task is "Write my economics essay." They visit `docs.google.com`. Google Docs has very little *textual content* about economics — its DOM is mostly UI controls, menus, and toolbar labels. The embedding of Google Docs' DOM content would have **near-zero similarity** to "economics essay."

Without this layer, Google Docs would be **blocked** while the user is trying to write their essay.

### The Data Flow

```
    ┌───────────────────────────────────────────────────────────────┐
    │                  CLASSIFY WEB REQUEST                        │
    │  sessionId, url, title, pageText                             │
    └──────────────────────┬────────────────────────────────────────┘
                           │
                           ▼
    ┌───────────────────────────────────────────────────────────────┐
    │  STEP 0: CHECK PRODUCTIVITY TOOL (runs FIRST)                │
    │                                                              │
    │  Input to LLM:                                               │
    │    url:   "https://docs.google.com/document/d/1abc..."       │
    │    title: "My Economics Essay - Google Docs"                  │
    │                                                              │
    │  ┌─────────────────────────────────────────────────────────┐ │
    │  │ google.ts: isProductivityTool(url, title)               │ │
    │  │                                                         │ │
    │  │ System prompt (exact text):                             │ │
    │  │ "You are a strict productivity tool detector.           │ │
    │  │  Determine if this URL + page title is a               │ │
    │  │  PRODUCTIVITY TOOL - something people use to           │ │
    │  │  DO work, not consume content.                         │ │
    │  │                                                         │ │
    │  │  Examples of productivity tools:                        │ │
    │  │  - Google Docs, Sheets, Slides                         │ │
    │  │  - Notion, Obsidian                                    │ │
    │  │  - GitHub (repos, PRs, issues)                         │ │
    │  │  - VS Code (web)                                       │ │
    │  │  - Figma, Linear, Jira                                 │ │
    │  │  - ChatGPT, Claude                                     │ │
    │  │  - Compiler/documentation sites                        │ │
    │  │                                                         │ │
    │  │  NOT productivity tools:                                │ │
    │  │  - Social media (Reddit, Twitter, Instagram)           │ │
    │  │  - News sites (CNN, BBC)                               │ │
    │  │  - Entertainment (YouTube, Netflix, Twitch)            │ │
    │  │  - Shopping (Amazon, eBay)                             │ │
    │  │                                                         │ │
    │  │  Reply ONLY with JSON: { \"isTool\": boolean }         │ │
    │  │  Be STRICT. When in doubt, say false."                 │ │
    │  │                                                         │ │
    │  │ Model: gemini-2.0-flash-lite                           │ │
    │  │ Temperature: 0.0 (deterministic)                       │ │
    │  │ Max tokens: 50                                         │ │
    │  │ Response MIME: application/json                         │ │
    │  └─────────────────────────────────────────────────────────┘ │
    │                                                              │
    │  LLM Response: { "isTool": true }                           │
    │                                                              │
    │  If isTool === true:                                         │
    │    → IMMEDIATELY return ALLOW                                │
    │    → Skip embedding entirely                                 │
    │    → Log event with reason: "Productivity tool detected"     │
    │                                                              │
    │  If isTool === false:                                        │
    │    → Proceed to embedding similarity (Step 3 above)          │
    └──────────────────────┬────────────────────────────────────────┘
                           │
                    (if not a tool)
                           │
                           ▼
                    Embedding comparison
```

**The exact response object when a tool is detected**:

```json
{
    "success": true,
    "data": {
        "decision": "allow",
        "confidence": 1.0,
        "similarity": 1.0,
        "reason": "Productivity tool detected",
        "isTool": true
    }
}
```

**Edge cases the LLM must handle**:

| URL | Title | LLM Decision | Why |
|-----|-------|--------------|-----|
| `youtube.com/watch?v=abc` | "3Blue1Brown - Neural Networks" | `false` | YouTube IS content consumption, even educational |
| `github.com/pytorch/pytorch` | "pytorch/pytorch: Tensors..." | `true` | GitHub is a work tool |
| `reddit.com/r/learnpython` | "r/learnpython - Reddit" | `false` | Reddit is social media despite educational content |
| `chat.openai.com` | "ChatGPT" | `true` | AI assistant is a productivity tool |
| `docs.google.com/spreadsheets` | "Budget Tracker - Google Sheets" | `true` | Google Sheets is a work tool |
| `stackoverflow.com/questions/123` | "How to fix CUDA error" | Debatable | The system errs toward `false` (content site, not tool) |

**Why this is an LLM call and not a URL whitelist**: A static whitelist can't handle:
1. New tools that emerge (e.g., `cursor.sh`, `v0.dev`)
2. Subdomains and path variations
3. Self-hosted instances (`gitlab.mycompany.com`)
4. Ambiguous sites (is Google Calendar a tool? Yes. Is Google News? No. Both are `google.com/*`)

---

## 25.4 — The Complete Classification Decision Tree

Here is the **full decision tree** that `handleWebClassification()` executes, with every data output at each leaf:

```
                    Incoming request
                    {sessionId, url, title, pageText}
                           │
                           ▼
                 ┌─── isProductivityTool(url, title) ───┐
                 │                                       │
             isTool: true                           isTool: false
                 │                                       │
                 ▼                                       ▼
          RETURN ALLOW                          generateEmbedding(
          confidence: 1.0                         title + " " + pageText
          similarity: 1.0                       )
          reason: "Productivity                      │
            tool detected"                           ▼
                                              contentEmbedding: Float32[768]
                                                     │
                                                     ▼
                                              classifyBySimilarity(
                                                taskEmbedding,    // from session
                                                contentEmbedding  // just generated
                                              )
                                                     │
                                                     ▼
                                              {decision, similarity, confidence}
                                                     │
                                    ┌────────────────┼────────────────┐
                                    │                │                │
                              sim >= 0.55      0.40 <= sim      sim < 0.40
                                    │           < 0.55              │
                                    │                │              │
                                    ▼                ▼              ▼
                              RETURN ALLOW     RETURN WARN    RETURN BLOCK
                                    │                │              │
                                    │                │              │
                              [all paths log to classification_events table]
```

**The exact log entry** written to Supabase for every classification:

```typescript
// supabase.ts: logClassificationEvent()
await client.from('classification_events').insert({
    session_id: "f47ac123-1234-5678-abcd-123456789012",
    event_type: "web",
    decision: "warn",
    similarity: 0.48,
    confidence: 0.32,
    url: "https://www.reddit.com/r/programming/comments/abc123",
    title: "Understanding Transformers - r/programming",
    reason: "Content is loosely related to task but may be a distraction",
    // id: auto-generated UUID
    // created_at: auto-set to NOW()
});
```

**The SQL that Supabase executes**:

```sql
INSERT INTO classification_events 
    (session_id, event_type, decision, similarity, confidence, url, title, reason)
VALUES 
    ('f47ac123-...', 'web', 'warn', 0.48, 0.32, 
     'https://www.reddit.com/r/programming/comments/abc123',
     'Understanding Transformers - r/programming',
     'Content is loosely related to task but may be a distraction');
```

---

## 25.5 — The HTTP Response: Exact Shape

**The API response** sent back to the Chrome extension:

```http
HTTP/1.1 200 OK
Content-Type: application/json
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 84
X-RateLimit-Reset: 1705341320

{
    "success": true,
    "data": {
        "decision": "warn",
        "confidence": 0.32,
        "similarity": 0.48,
        "reason": "Content is loosely related to task but may be a distraction"
    }
}
```

**The TypeScript type** of `data`:

```typescript
// types.ts
interface ClassificationResult {
    decision: 'allow' | 'warn' | 'block';
    confidence: number;     // 0.0 – 1.0, rounded to 2dp
    similarity: number;     // -1.0 – 1.0 (raw cosine)
    reason: string;         // Human-readable explanation
    isTool?: boolean;       // Only present if tool detection ran
}
```

---

## 25.6 — Back in the Extension: Applying the Result

The response flows back through several layers in the extension:

```
    HTTP Response (200 OK)
    { success: true, data: { decision: "warn", ... } }
            │
            ▼
    ┌───────────────────────────────────────────┐
    │  api.js: classifyPage()                   │
    │  return response.data;                    │
    │  // Returns just the data object:         │
    │  // { decision, confidence, similarity,   │
    │  //   reason }                            │
    └───────────────────────┬───────────────────┘
            │
            ▼
    ┌───────────────────────────────────────────┐
    │  background.js: handleClassification()    │
    │                                           │
    │  1. RACE CHECK #5:                        │
    │     if (currentTabUrls.get(tabId)         │
    │         !== tab.url) return;              │
    │                                           │
    │  2. Update URL cache:                     │
    │     await addRecentUrl(tab.url, result);  │
    │     // Stores in chrome.storage.local:    │
    │     // recentUrls["https://reddit.com"]   │
    │     //   = { timestamp: Date.now(),       │
    │     //       result: {decision: "warn",   │
    │     //       reason: "...",               │
    │     //       confidence: 0.32} }          │
    │                                           │
    │  3. Increment stats:                      │
    │     await incrementStat('pagesChecked');  │
    │     await incrementStat('pagesWarned');   │
    │     // Stats in chrome.storage.local:     │
    │     // sessionStats = {                   │
    │     //   pagesChecked: 48,               │
    │     //   pagesAllowed: 38,               │
    │     //   pagesWarned: 7,                 │
    │     //   pagesBlocked: 3                 │
    │     // }                                  │
    │                                           │
    │  4. Show overlay:                         │
    │     if (result.decision !== 'allow') {    │
    │       await showOverlay(tabId, result,    │
    │                         session.blockMode);│
    │     }                                     │
    └───────────────────────┬───────────────────┘
            │
            ▼
    ┌───────────────────────────────────────────┐
    │  background.js: showOverlay()             │
    │                                           │
    │  chrome.tabs.sendMessage(tabId, {         │
    │    type: 'SHOW_OVERLAY',                  │
    │    decision: 'warn',                      │
    │    reason: 'Content is loosely related...│
    │    blockMode: 'warn',                     │
    │    confidence: 0.32                       │
    │  });                                      │
    └───────────────────────┬───────────────────┘
            │
            ▼  (Chrome message passing)
    ┌───────────────────────────────────────────┐
    │  content.js: onMessage listener           │
    │  case 'SHOW_OVERLAY':                     │
    │    showOverlay(                            │
    │      message.decision,   // "warn"        │
    │      message.reason,     // "Content..."  │
    │      message.blockMode,  // "warn"        │
    │      message.confidence  // 0.32          │
    │    );                                     │
    └───────────────────────────────────────────┘
```

### The Overlay DOM Injection

The content script creates a **full-screen overlay** by injecting DOM elements into the page:

```
<div id="siren-overlay" class="siren-overlay siren-overlay-warn">
    <div class="siren-modal">
        
        <!-- Icon -->
        <div class="siren-icon">
            <img src="chrome-extension://<id>/bound-hands.png" alt="Ithaca">
        </div>
        
        <!-- Title -->
        <h2 class="siren-title">
            "This page may be off-topic"        <!-- for warn -->
            "This page doesn't match your focus" <!-- for block -->
        </h2>
        
        <!-- Buttons -->
        <div class="siren-buttons">
            <button class="siren-btn siren-btn-primary">
                ← Go Back
            </button>
            <!-- Only if blockMode === 'warn' OR decision === 'warn' -->
            <button class="siren-btn siren-btn-secondary">
                Continue Anyway →
            </button>
        </div>
        
        <!-- Challenge button (only for warn decisions) -->
        <div>
            <button class="siren-challenge-btn">
                Do you think this is improperly blocked?
            </button>
            <div class="siren-challenge-msg"></div>
        </div>
        
        <!-- Branding -->
        <div class="siren-branding">
            <span>Protected by </span>
            <span class="siren-brand-name">Ithaca</span>
        </div>
    </div>
</div>
```

**Overlay behavior matrix**:

| `blockMode` | `decision` | "Go Back" | "Continue Anyway" | "Challenge" | Escape key | Scroll |
|------------|------------|-----------|-------------------|-------------|------------|--------|
| `warn` | `warn` | ✓ (history.back) | ✓ (remove overlay) | ✓ | ✓ dismisses | Blocked |
| `warn` | `block` | ✓ | ✓ (because blockMode=warn) | ✗ | ✓ | Blocked |
| `block` | `warn` | ✓ | ✓ (decision=warn) | ✓ | ✓ | Blocked |
| `block` | `block` | ✓ | ✗ (hard lock) | ✗ | ✗ | Blocked |

The `isDismissible` logic:
```javascript
const isDismissible = blockMode === 'warn' || decision === 'warn';
```

**Page scroll is disabled** when overlay is shown:
```javascript
document.body.style.overflow = 'hidden'; // On show
document.body.style.overflow = '';       // On hide
```

---

## 25.7 — The Challenge / Appeals System ("Supreme Court")

When a user believes a page was incorrectly blocked/warned, they can challenge the decision. This triggers an entirely **different classification pipeline** — LLM-based deep analysis instead of embedding similarity.

### Challenge Data Flow

```
    User clicks "Do you think this is improperly blocked?"
                           │
                           ▼
    ┌───────────────────────────────────────────┐
    │  content.js: Challenge button handler     │
    │                                           │
    │  1. Re-extract page content:              │
    │     const content = extractPageContent(); │
    │     // {title: "...", pageText: "..."}    │
    │                                           │
    │  2. Send message to background:           │
    │     chrome.runtime.sendMessage({          │
    │       type: 'CHALLENGE_BLOCK',            │
    │       url: window.location.href,          │
    │       title: document.title,              │
    │       pageText: content.pageText          │
    │     });                                   │
    └───────────────────────┬───────────────────┘
                           │
                           ▼
    ┌───────────────────────────────────────────┐
    │  background.js: onMessage handler         │
    │  case 'CHALLENGE_BLOCK':                  │
    │                                           │
    │  1. Get active session                    │
    │  2. Call api.challengeBlock(              │
    │       session.sessionId,                  │
    │       message.url,                        │
    │       message.title,                      │
    │       message.pageText                    │
    │     );                                    │
    └───────────────────────┬───────────────────┘
                           │
                           ▼
    POST /v1/classify/challenge
    {
        "sessionId": "f47ac123-...",
        "url": "https://reddit.com/r/programming/...",
        "title": "Understanding Transformers",
        "pageText": "Transformers are a type of neural..."
    }
```

### Backend Challenge Processing

```
    ┌───────────────────────────────────────────────────────┐
    │  classify.ts: handleChallenge()                       │
    │                                                       │
    │  1. Validate via ChallengeSchema (same as Classify)   │
    │                                                       │
    │  2. Retrieve session from Supabase                    │
    │     (same as regular classification)                  │
    │                                                       │
    │  3. CHECK: Has this URL been challenged before?       │
    │     await hasChallengedUrl(sessionId, url)            │
    │                                                       │
    │     SQL:                                              │
    │     SELECT COUNT(*) FROM classification_events        │
    │     WHERE session_id = 'f47ac123-...'                 │
    │       AND url = 'https://reddit.com/r/...'           │
    │       AND event_type = 'challenge';                   │
    │                                                       │
    │     If count > 0:                                     │
    │       → HTTP 400: "This URL has already been          │
    │         challenged in this session"                   │
    │       → User gets ONE appeal per URL per session      │
    │                                                       │
    │  4. If not previously challenged:                     │
    │     → Call LLM for deep relevance analysis            │
    └───────────────────────┬───────────────────────────────┘
                           │
                           ▼
    ┌───────────────────────────────────────────────────────┐
    │  google.ts: analyzeContentRelevance()                 │
    │                                                       │
    │  This is a DIFFERENT function than the embedding      │
    │  pipeline. It uses Gemini as a JUDGE, not embeddings. │
    │                                                       │
    │  System prompt (exact):                               │
    │  "You are a strict but fair content relevance judge.  │
    │   A user is working on a specific task and claims     │
    │   that a blocked webpage is actually relevant to      │
    │   their work.                                         │
    │                                                       │
    │   TASK: {session.task_description}                    │
    │   URL: {url}                                          │
    │   PAGE TITLE: {title}                                 │
    │   PAGE CONTENT: {pageText.substring(0, 5000)}         │
    │                                                       │
    │   Evaluate whether this content is DIRECTLY relevant  │
    │   to the user's task.                                 │
    │                                                       │
    │   Be STRICT: 'tangentially related' is NOT enough.    │
    │   The content must provide direct value for           │
    │   completing the task.                                │
    │                                                       │
    │   Reply with JSON:                                    │
    │   {                                                   │
    │     \"relevant\": boolean,                            │
    │     \"confidence\": 0.0 to 1.0,                      │
    │     \"reason\": \"explanation\"                       │
    │   }"                                                  │
    │                                                       │
    │  Model: gemini-2.0-flash (NOT flash-lite — uses       │
    │         the more capable model for appeals)           │
    │  Temperature: 0.1                                     │
    │  Max tokens: 500                                      │
    │  Response MIME: application/json                      │
    └───────────────────────┬───────────────────────────────┘
                           │
                           ▼
    ┌───────────────────────────────────────────────────────┐
    │  LLM Response (parsed JSON):                          │
    │                                                       │
    │  Scenario A — Appeal GRANTED:                         │
    │  {                                                    │
    │    "relevant": true,                                  │
    │    "confidence": 0.78,                                │
    │    "reason": "This r/programming post discusses       │
    │      transformer architectures with code examples     │
    │      directly applicable to the user's PyTorch        │
    │      neural network homework."                        │
    │  }                                                    │
    │  → Mapped to: decision = "allow"                      │
    │                                                       │
    │  Scenario B — Appeal DENIED:                          │
    │  {                                                    │
    │    "relevant": false,                                 │
    │    "confidence": 0.85,                                │
    │    "reason": "While related to programming, this      │
    │      Reddit discussion thread is about industry       │
    │      news and opinions, not directly useful for       │
    │      completing homework assignments."                │
    │  }                                                    │
    │  → Mapped to: decision = "block"                      │
    └───────────────────────┬───────────────────────────────┘
                           │
                           ▼
    ┌───────────────────────────────────────────────────────┐
    │  Log the challenge event to Supabase:                 │
    │                                                       │
    │  INSERT INTO classification_events VALUES (            │
    │    gen_random_uuid(),                                 │
    │    'f47ac123-...',         -- session_id              │
    │    'challenge',            -- event_type (NOT 'web')  │
    │    'allow',                -- decision (appeal result)│
    │    0.48,                   -- original similarity     │
    │    0.78,                   -- LLM's confidence        │
    │    'https://reddit.com/...', -- url                   │
    │    'Understanding Transformers', -- title             │
    │    NULL, NULL,             -- app_name, window_title  │
    │    'Transformer architectures with code examples...', │
    │    NOW()                                              │
    │  );                                                   │
    └───────────────────────────────────────────────────────┘
```

### Challenge Response Flow Back to UI

```
    HTTP 200:
    {
        "success": true,
        "data": {
            "decision": "allow",   // or "block"
            "confidence": 0.78,
            "reason": "This r/programming post discusses..."
        }
    }
            │
            ▼
    background.js → sendResponse({ success: true, result: data })
            │
            ▼
    content.js → Challenge button handler:
    
    if (data.decision === 'allow') {
        // GREEN success message
        challengeMsg.textContent = 'Analysis: Approved. Unblocking...';
        challengeMsg.classList.add('siren-challenge-success');
        
        // Remove overlay after 1.5 seconds
        setTimeout(() => hideOverlay(), 1500);
        
    } else {
        // RED failure message
        challengeBtn.textContent = 'Challenge Rejected';
        challengeMsg.textContent = data.reason;
        challengeMsg.classList.add('siren-challenge-error');
        
        // PERMANENTLY disable challenge button for this overlay
        challengeBtn.style.pointerEvents = 'none';
        challengeBtn.style.opacity = '0.5';
    }
```

**Key design decisions in the challenge system**:

1. **One challenge per URL per session**: The `hasChallengedUrl()` query prevents abuse. Once a URL is challenged (win or lose), it cannot be challenged again in the same session.

2. **Upgraded model**: Regular classification uses `gemini-2.0-flash-lite` (cheapest, fastest). Challenges use `gemini-2.0-flash` (more capable) — because challenges are rare and the stakes are higher.

3. **Different methodology**: Regular classification = mathematical (cosine similarity). Challenges = semantic (LLM reads the page + task and makes a judgment). This is intentional — the LLM can understand nuance that embeddings miss (e.g., "this Stack Overflow answer is about a specific error I'm debugging").

4. **No recursive challenges**: If a challenge is denied, that's final. There's no "appeal the appeal."

---

## 25.8 — The Reason Generation (Where Explanations Come From)

The human-readable `reason` field in the response has **different origins** depending on the classification path:

| Path | How `reason` is generated |
|------|--------------------------|
| Productivity tool detected | Hardcoded: `"Productivity tool detected"` |
| Embedding → ALLOW | Constructed from similarity: `"Content is relevant to your task"` |
| Embedding → WARN | Constructed: `"Content is loosely related to task but may be a distraction"` |
| Embedding → BLOCK | Constructed: `"Content does not appear related to your current task"` |
| Challenge → ALLOW | From LLM response: `data.reason` (the LLM's natural language explanation) |
| Challenge → BLOCK | From LLM response: `data.reason` |

---

## 25.9 — Error Handling & Failure Taxonomy

### API-Level Errors

| Scenario | HTTP Code | Response Body | What Happens |
|----------|-----------|---------------|--------------|
| Invalid JSON | 400 | `{ error: "Validation failed", details: [...] }` | Extension shows error toast |
| Bad session ID | 404 | `{ error: "Session not found" }` | Extension clears local session |
| Expired session | 400 | `{ error: "Session has expired" }` | Extension clears + prompts restart |
| Bad JWT | 401 | `{ error: "Token has expired" }` | Extension refreshes token |
| Rate limited | 429 | `{ error: "Too many requests" }` | Extension backs off |
| Gemini API failure | 500 | `{ error: "Classification failed" }` | Extension fails open (allows page) |
| Supabase DB failure | 500 | `{ error: "Internal server error" }` | Same fail-open behavior |

### Extension-Level Errors (Content Script)

| Error | Detection | Handling |
|-------|-----------|----------|
| Page in bfcache | `error.includes('cache')` | Silent skip |
| Tab closed during classification | `error.includes('channel')` | Silent skip |
| Content script not loaded | `error.includes('Receiving end')` | Fallback content: `{ title: tab.title, pageText: "" }` |
| Message port closed | `error.includes('message port closed')` | Silent skip |
| Context invalidated (extension update) | `error.includes('context invalidated')` | Silent skip |

**Design philosophy**: The system **fails open**. If any component fails (Gemini API down, Supabase unreachable, content extraction error), the default behavior is to **allow** the page rather than incorrectly blocking it. A focus tool that blocks your work tool due to a bug is worse than one that occasionally misses a distraction.

---

## 25.10 — Performance Characteristics

### Latency Budget

```
Total user-perceived latency: ~500-1200ms

Breakdown:
├── Auth middleware (JWT verify):         ~5ms   (cached HS256 key)
├── Zod validation:                       ~1ms   (compile-time schema)
├── Supabase session fetch:              ~50ms   (PostgreSQL query)
├── Productivity tool LLM check:        ~200ms   (flash-lite, 50 tokens)
│   └── (if tool detected: RETURN here, total ~256ms)
├── Content embedding generation:       ~150ms   (text-embedding-004)
├── Cosine similarity computation:        ~0.1ms (768 multiplications)
├── Confidence calculation:               ~0.01ms
├── Event logging (fire-and-forget):     ~30ms   (non-blocking)
└── Response serialization:               ~1ms
                                        ──────
                                       ~437ms  (if tool detected)
                                       ~437ms  (typical classification)
```

**The extension adds**:
- Content extraction (Readability parse): ~50-200ms depending on page size
- Chrome message passing overhead: ~5ms per message
- Network round-trip to Railway: ~50-150ms depending on user location

### Caching Strategy

```
Layer 1: Extension URL cache (chrome.storage.local)
         TTL: 30 seconds
         Hit rate: ~40% (users revisit recent pages)
         Effect: Eliminates redundant API calls

Layer 2: JWT secret cache (server-side memory)
         TTL: Process lifetime
         Effect: Avoids re-encoding SUPABASE_JWT_SECRET per request

Layer 3: Supabase client singleton (server-side)
         TTL: Process lifetime
         Effect: Connection pool reuse
```

---

## 25.11 — Complete Data Transformation Summary

Here's every data transformation in the entire classification pipeline, in order:

```
1. RAW DOM → Readability Parser → Clean article text
   Input:  Full HTML document (100KB+)
   Output: { title: string, pageText: string (≤15KB) }

2. Title + PageText → String concatenation + truncation
   Input:  "My Title" + " " + "Article body..."
   Output: "My Title Article body..." (≤10KB)

3. Concatenated text → Google Embedding API → Float32[768]
   Input:  "My Title Article body..." (string, ≤10KB)
   Output: [0.018, -0.042, 0.075, ...] (768 floats)

4. Task embedding × Content embedding → Cosine Similarity → Float64
   Input:  Float64[768] (task) + Float32[768] (content)
   Output: 0.48 (single scalar, range -1 to 1)

5. Similarity → Threshold comparison → Decision enum
   Input:  0.48 (float)
   Output: "warn" (one of: "allow" | "warn" | "block")

6. Similarity + Decision → Confidence formula → Float64
   Input:  similarity=0.48, decision="warn"
   Output: 0.32 (float, range 0 to 1, rounded to 2dp)

7. Decision + Confidence + Similarity → ClassificationResult
   Input:  decision="warn", confidence=0.32, similarity=0.48
   Output: { decision, confidence, similarity, reason }

8. ClassificationResult → Supabase INSERT (fire-and-forget)
   Input:  ClassificationResult + session metadata
   Output: classification_events row (UUID PK)

9. ClassificationResult → HTTP JSON response
   Input:  ClassificationResult
   Output: { success: true, data: ClassificationResult }

10. HTTP Response → Extension cache update
    Input:  ClassificationResult
    Output: chrome.storage.local.recentUrls[url] = {timestamp, result}

11. ClassificationResult → Chrome message → Content script overlay
    Input:  { decision, reason, blockMode, confidence }
    Output: DOM elements injected into page
```

---

*Next chapter: Frontend deep-dive — the popup UI state machine, content script lifecycle, and Readability parsing internals.*
