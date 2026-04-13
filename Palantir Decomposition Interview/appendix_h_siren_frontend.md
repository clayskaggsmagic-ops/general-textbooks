# Appendix H: Siren: Frontend Architecture, State Management & Data Lifecycle

> **Interview Context**: This chapter maps every piece of state the Chrome extension manages, every message that flows between its components, and the complete end-to-end data lifecycle from user input to Supabase persistence. If an interviewer asks "walk me through what happens to the user's data," you can trace a single datum from the moment a keystroke enters the popup to the moment it becomes a row in PostgreSQL.

---

## 26.1 — Chrome Extension Component Model

A Chrome Manifest V3 extension has **three execution contexts** that cannot share memory. All communication is via Chrome's message-passing API.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     CHROME BROWSER PROCESS                         │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  SERVICE WORKER (background.js)                              │   │
│  │  Lifecycle: Starts on install, wakes on events, sleeps idle  │   │
│  │  Capabilities: chrome.tabs, chrome.storage, chrome.runtime,  │   │
│  │                 chrome.identity, fetch()                     │   │
│  │  CANNOT: Access DOM, use window/document                     │   │
│  │                                                               │   │
│  │  State held in MEMORY (lost on sleep):                       │   │
│  │    const currentTabUrls = new Map<number, string>();         │   │
│  │    // tabId → latest URL (race condition tracker)            │   │
│  │                                                               │   │
│  │  State held in STORAGE (persisted):                          │   │
│  │    chrome.storage.local → see §26.2                          │   │
│  └──────────────────────────┬──────────────────────────────────┘   │
│                              │                                      │
│            chrome.tabs.sendMessage() │  chrome.runtime.sendMessage()│
│            (background → content)    │  (content → background)     │
│                              │       │                              │
│  ┌───────────────────────────▼───────▼─────────────────────────┐   │
│  │  CONTENT SCRIPT (content.js) — injected into EVERY http tab  │   │
│  │  Lifecycle: Loaded when tab loads an http(s) page            │   │
│  │  Capabilities: Full DOM access to the host page              │   │
│  │  CANNOT: chrome.tabs, chrome.identity                        │   │
│  │  CAN: chrome.runtime.sendMessage(), chrome.storage           │   │
│  │                                                               │   │
│  │  Imports: @mozilla/readability (bundled by Vite)             │   │
│  │  Injects: Overlay DOM + content.css                          │   │
│  │  Also:    Reader Mode view (optional)                        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  POPUP (popup.html + popup.js + popup.css)                   │   │
│  │  Lifecycle: Exists ONLY while popup is open (destroyed on    │   │
│  │             close). Completely ephemeral.                     │   │
│  │  Capabilities: Full DOM (own popup DOM), chrome.storage,     │   │
│  │                 chrome.runtime, fetch()                      │   │
│  │  CANNOT: Access host page DOM                                │   │
│  │                                                               │   │
│  │  UI states: Login, Session Setup (task input, Pomodoro       │   │
│  │             config, block mode), Active Session (timer,      │   │
│  │             stats), Settings                                 │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Message Protocol

All inter-component messages use a discriminated union on the `type` field:

```typescript
// Messages FROM background TO content script
type BackgroundToContent = 
    | { type: 'EXTRACT_CONTENT' }                    // → content responds with {title, pageText}
    | { type: 'SHOW_OVERLAY',
        decision: 'warn' | 'block',
        reason: string,
        blockMode: 'warn' | 'block',
        confidence: number }
    | { type: 'HIDE_OVERLAY' };

// Messages FROM content script / popup TO background
type ToBackground =
    | { type: 'SESSION_STARTED', session: ActiveSession }
    | { type: 'SESSION_ENDED' }
    | { type: 'LOGGED_OUT' }
    | { type: 'GET_SESSION' }                        // → responds with { session }
    | { type: 'GET_STATS' }                          // → responds with { stats }
    | { type: 'CHALLENGE_BLOCK',
        url: string,
        title: string,
        pageText: string };                          // → responds with { success, result? }
```

**Async response pattern**: For messages that require async responses (`GET_SESSION`, `GET_STATS`, `CHALLENGE_BLOCK`), the handler returns `true` from `onMessage.addListener` to keep the message channel open:

```javascript
// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
    switch (message.type) {
        case 'GET_SESSION':
            getActiveSession().then(session => {
                sendResponse({ session });
            });
            return true; // ← CRITICAL: keeps channel open for async response
        // ...
    }
});
```

Without `return true`, the channel closes synchronously before the `await` resolves, and `sendResponse` silently fails.

---

## 26.2 — Chrome Storage Schema (The Extension's "Database")

`chrome.storage.local` is the extension's persistent key-value store. It survives browser restarts, extension updates, and service worker sleep cycles. Here is the **complete schema** with exact shapes:

```javascript
// The four top-level keys:
chrome.storage.local = {
    
    // ═══════════════════════════════════════════════════════════
    // 1. SUPABASE AUTH SESSION
    // ═══════════════════════════════════════════════════════════
    // Written by: supabase.js chromeStorageAdapter
    // Read by:    supabase.js getAccessToken(), getSession()
    // Cleared by: signOut(), LOGGED_OUT message
    
    "supabaseSession": {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "refresh_token": "v1.MbQvXy3z...",
        "token_type": "bearer",
        "expires_in": 3600,
        "expires_at": 1705344800,
        "user": {
            "id": "a1b2c3d4-5678-90ab-cdef-123456789abc",
            "email": "user@example.com",
            "app_metadata": { "provider": "google" },
            "user_metadata": {
                "full_name": "Clay Skaggs",
                "avatar_url": "https://lh3.googleusercontent.com/..."
            },
            "created_at": "2025-01-10T12:00:00.000Z"
        }
    },
    
    // ═══════════════════════════════════════════════════════════
    // 2. ACTIVE FOCUS SESSION
    // ═══════════════════════════════════════════════════════════
    // Written by: popup.js after startSession() API returns
    // Read by:    background.js (every tab event), popup.js (on open)
    // Cleared by: endSession(), session expiry, LOGGED_OUT
    
    "activeSession": {
        "sessionId": "f47ac123-1234-5678-abcd-123456789012",
        "taskDescription": "Working on neural network homework using PyTorch...",
        "blockMode": "warn",      // "warn" = dismissible, "block" = hard lock
        "expiresAt": "2025-01-15T20:00:00.000Z",
        "startedAt": "2025-01-15T18:00:00.000Z",
        "pomodoro": {              // null if not using Pomodoro
            "enabled": true,
            "workMin": 25,
            "breakMin": 5
        }
    },
    //  OR null (no active session)
    
    // ═══════════════════════════════════════════════════════════
    // 3. SESSION STATISTICS
    // ═══════════════════════════════════════════════════════════
    // Written by: background.js incrementStat() after each classification
    // Read by:    popup.js (displayed in active session view)
    // Reset by:   session end, session expiry, LOGGED_OUT
    
    "sessionStats": {
        "pagesChecked": 47,   // Total classifications attempted
        "pagesAllowed": 38,   // Decision: "allow"
        "pagesWarned": 6,     // Decision: "warn"
        "pagesBlocked": 3     // Decision: "block"
    },
    
    // ═══════════════════════════════════════════════════════════
    // 4. RECENT URL CACHE (Debounce Layer)
    // ═══════════════════════════════════════════════════════════
    // Written by: background.js addRecentUrl() after API response
    // Read by:    background.js isRecentlyClassified() before API call
    // Cleaned by: setInterval every 30s (entries > 60s evicted)
    // Cleared by: session start, session end, LOGGED_OUT
    
    "recentUrls": {
        "https://reddit.com/r/programming": {
            "timestamp": 1705341234567,    // Date.now() at classification
            "result": {
                "decision": "warn",
                "reason": "Content is loosely related to task...",
                "confidence": 0.32
            }
        },
        "https://pytorch.org/docs/stable/": {
            "timestamp": 1705341200000,
            "result": {
                "decision": "allow",
                "reason": "Relevant to task",
                "confidence": 0.92
            }
        },
        "https://instagram.com/explore": {
            "timestamp": 1705341180000,
            "result": {
                "decision": "block",
                "reason": "Content does not appear related...",
                "confidence": 0.95
            }
        }
    }
};
```

### Storage Write Patterns

| Key | Write Frequency | Write Source | Size Estimate |
|-----|----------------|--------------|---------------|
| `supabaseSession` | On login, token refresh (~hourly) | `supabase.js` adapter | ~2 KB |
| `activeSession` | On session start/end (~2-3x/day) | `popup.js` | ~500 bytes |
| `sessionStats` | On every classification (~1/sec peak) | `background.js` | ~100 bytes |
| `recentUrls` | On every classification + cleanup | `background.js` | ~5-50 KB (0-100 entries) |

**Potential bottleneck**: `sessionStats` and `recentUrls` are written on every single page classification. `chrome.storage.local` operations are async and serialized — this could create write contention at high browsing speeds. In practice, Chrome's storage API handles this gracefully with internal queuing.

---

## 26.3 — The Popup State Machine

The popup is the user's control surface. It is **entirely ephemeral** — destroyed when closed, reconstructed from `chrome.storage.local` when opened.

```
                              ┌─────────────────┐
                              │     LAUNCH       │
                              │   popup.html     │
                              └────────┬─────────┘
                                       │
                              Read supabaseSession
                              from chrome.storage
                                       │
                         ┌─────────────┴─────────────┐
                         │                           │
                    session exists              session null
                         │                           │
                         ▼                           ▼
                ┌─────────────────┐         ┌──────────────────┐
                │  CHECK SESSION  │         │   LOGIN SCREEN   │
                │  VALIDITY       │         │                  │
                │                 │         │  [Google Sign In]│
                │  Is token       │         │  [Email/Password]│
                │  expired?       │         │                  │
                └────┬───────────┘         └──────────────────┘
                     │                              │
              ┌──────┴──────┐               signInWithGoogle()
              │             │               signIn(email, pw)
           valid         expired                    │
              │             │                       ▼
              │      refreshSession()     supabaseSession stored
              │             │             in chrome.storage
              │             │                       │
              ▼             ▼                       │
        Read activeSession  ◄───────────────────────┘
        from chrome.storage
              │
     ┌────────┴────────┐
     │                 │
  session exists    session null
     │                 │
     ▼                 ▼
┌──────────────┐  ┌──────────────────────────────────────┐
│ ACTIVE       │  │ SESSION SETUP                         │
│ SESSION VIEW │  │                                       │
│              │  │ ┌──────────────────────────────────┐  │
│ • Timer      │  │ │ Task input text area             │  │
│ • Stats      │  │ │ "What are you working on?"       │  │
│ • Task desc  │  │ │                                  │  │
│ • Block mode │  │ │ [Analyze Task] ─────────────────►│──┤──── POST /v1/session/analyze-task
│ • End button │  │ │                                  │  │
│ • Pomodoro   │  │ │ If not ready:                    │  │
│   indicator  │  │ │   Show follow-up questions       │  │
│              │  │ │   User answers                   │  │
│              │  │ │   [Submit Answers] ──────────────►│──┤──── POST /analyze-task (loop)
│              │  │ │                                  │  │
│              │  │ │ If ready:                        │  │
│              │  │ │   Show refined task + duration   │  │
│              │  │ └──────────────────────────────────┘  │
│              │  │                                       │
│              │  │ Block Mode: [Warn ◉] [Block ○]       │
│              │  │ Pomodoro:   [25/5] [50/10] [Custom]  │
│              │  │                                       │
│              │  │ [Start Session] ─────────────────────►│── POST /v1/session/start
│              │  │                                       │
│              │  └──────────────────────────────────────┘  │
└──────────────┘                                            │
       │                                                     │
       │ "End Session" clicked                               │
       │ POST /v1/session/end                                │
       │ clearActiveSession()                                │
       │ resetSessionStats()                                 │
       │ clearRecentUrls()                                   │
       │ chrome.runtime.sendMessage({type:'SESSION_ENDED'})  │
       ▼                                                     │
  Return to SESSION SETUP ◄──────────────────────────────────┘
```

### Task Analysis Loop — Exact Data Exchange

```
Round 1:
  popup.js → api.js:
    analyzeTask("Working on homework", [])
  
  api.js → HTTP:
    POST /v1/session/analyze-task
    { "taskDescription": "Working on homework", "previousResponses": [] }
  
  Backend → LLM:
    "Is 'Working on homework' specific enough?"
  
  LLM → Backend:
    { "ready": false, "followUpQuestions": [
        "What subject is your homework for?",
        "What specific topic or assignment?"
    ] }
  
  Backend → popup.js:
    { ready: false, followUpQuestions: ["What subject...", "What specific..."] }
  
  Popup renders: Two question fields for user to fill in

Round 2:
  popup.js → api.js:
    analyzeTask("Working on homework", [
        { question: "What subject is your homework for?", answer: "Machine Learning" },
        { question: "What specific topic?", answer: "Neural networks with PyTorch" }
    ])
  
  Backend → LLM:
    "User says ML homework on neural networks with PyTorch. Specific enough?"
  
  LLM → Backend:
    { "ready": true, 
      "refinedTask": "Working on neural network homework using PyTorch",
      "durationMinutes": 120 }
  
  Popup renders: Confirmation with suggested 2-hour duration
```

---

## 26.4 — Content Extraction Deep Dive

### Mozilla Readability — How It Works

Readability is a **DOM parsing algorithm** originally written for Firefox Reader View. It scores DOM nodes by heuristics to find the "article body" and strips everything else.

```javascript
// content.js: extractPageContent()
function extractPageContent() {
    // Step 1: Clone the DOM (Readability modifies the DOM it receives)
    const documentClone = document.cloneNode(true);
    //  ↑ This is a deep clone of the ENTIRE document:
    //    <html>, <head>, <body>, all children, all text nodes
    //    Estimated size for a medium page: 500KB - 5MB
    
    // Step 2: Parse with Readability
    const reader = new Readability(documentClone);
    const article = reader.parse();
    
    // article object (if successful):
    // {
    //   title:       "Understanding Transformers",     // extracted <h1> or <title>
    //   byline:      "John Smith",                     // detected author
    //   content:     "<div><p>Transformers are...</p>", // cleaned HTML
    //   textContent: "Transformers are a type of...",   // stripped plaintext
    //   length:      4523,                              // character count
    //   excerpt:     "An introduction to...",           // first paragraph
    //   siteName:    "r/programming"                    // site name
    // }
    
    // Step 3: Fallback if Readability fails (returns null)
    if (!article) {
        pageText = document.body.innerText;
        //  ↑ Raw text extraction — includes nav, footer, ads, etc.
    }
    
    // Step 4: Clean and truncate
    pageText = pageText
        .replace(/\s+/g, ' ')        // Collapse all whitespace to single spaces
        .trim()                       // Remove leading/trailing whitespace
        .substring(0, 15000);         // Hard cap at 15,000 characters
    
    // Step 5: Return to background service worker
    return {
        title: document.title || '',  // "Understanding Transformers - Reddit"
        pageText: pageText            // "Transformers are a type of neural..."
    };
}
```

**What Readability strips**:
- `<nav>` elements (navigation menus)
- `<footer>` elements
- `<aside>` elements (sidebars)
- Elements with class names like `sidebar`, `comment`, `footer`, `header`, `menu`, `nav`, `social`
- Elements with very low text-to-link ratio (link farms)
- `<script>` and `<style>` elements
- Ads detected via common ad container IDs/classes

**What this means for classification**: A Reddit page about neural networks might have its raw DOM be 60% navigation, 20% sidebar, 10% comments, and 10% article content. After Readability, it's 100% article content. This is **critical** for embedding quality — noise from unrelated DOM text would drag the cosine similarity away from the true topic.

### Fallback Content

If `sendMessage` to the content script fails entirely (tab closed, bfcache, etc.), the background creates minimal fallback content:

```javascript
// background.js, inside handleClassification()
catch (error) {
    pageContent = {
        title: tab.title || '',  // Chrome still has the tab's title
        pageText: '',            // No body text at all
    };
}
```

This means the classification will run against **just the page title**. The embedding of a title alone is surprisingly informative — "r/programming - Reddit" embeds very differently from "PyTorch Documentation."

---

## 26.5 — Reader Mode (Bonus Feature)

The content script includes an optional Reader Mode that transforms article pages into clean reading views:

```javascript
// content.js: checkAndEnableReaderMode()
async function checkAndEnableReaderMode() {
    // Check setting
    const setting = await chrome.storage.local.get(['readerModeEnabled']);
    if (!setting.readerModeEnabled) return;
    
    // Check if page is an article (not a web app like Gmail)
    if (!isProbablyReaderable(document)) return;
    
    // Don't double-render
    if (document.getElementById('siren-reader-view')) return;
    
    enableReaderMode();
}
```

**`isProbablyReaderable()`** (from Readability library): Uses heuristics to determine if a page is an article. It checks paragraph count, text density, and text-to-markup ratio. Gmail, Google Docs, and dashboards return `false`. Blog posts and news articles return `true`.

**Reader Mode DOM injection** (if enabled):

```html
<div id="siren-reader-view" class="siren-reader-view">
    <div class="siren-reader-content">
        <div class="siren-reader-header">
            <h1 class="siren-reader-title">Understanding Transformers</h1>
            <div class="siren-reader-meta">
                <span>John Smith</span> • 
                <span>12 min read</span>           <!-- ceil(wordCount/200) -->
                <span class="siren-reader-site">reddit.com</span>
            </div>
        </div>
        <div class="siren-reader-body">
            <!-- Readability's cleaned HTML content -->
            <p>Transformers are a type of neural network...</p>
        </div>
        <button class="siren-reader-close" title="Close Reader View">✕</button>
    </div>
</div>
```

**Data flow**:
- `readerModeEnabled` setting → `chrome.storage.local` (toggled by popup settings)
- Setting changes are **reactively detected** via `chrome.storage.onChanged` listener in the content script
- When disabled, the reader view div is removed and scroll is restored

---

## 26.6 — Frontend Authentication Architecture

### The Chrome Extension OAuth Problem

Standard web apps redirect to Google OAuth and get redirected back. Chrome extensions **can't be redirected to** because they don't have a normal URL. The solution: `chrome.identity.launchWebAuthFlow()`.

```
    User clicks "Sign in with Google"
                    │
                    ▼
    ┌─────────────────────────────────────────────────┐
    │  supabase.js: signInWithGoogle()                │
    │                                                 │
    │  STEP 1: Get redirect URL                       │
    │  chrome.identity.getRedirectURL()               │
    │  → "https://fgcmadhnkf...chromiumapp.org/"      │
    │  (Chrome generates this from extension ID)      │
    │                                                 │
    │  STEP 2: Build OAuth URL via Supabase           │
    │  supabase.auth.signInWithOAuth({                │
    │    provider: 'google',                          │
    │    options: {                                   │
    │      skipBrowserRedirect: true,  ← DON'T open  │
    │      redirectTo: "https://fgcm...org/",         │
    │      queryParams: {                             │
    │        access_type: 'offline',  ← get refresh   │
    │        prompt: 'consent'        ← always show   │
    │      }                                          │
    │    }                                            │
    │  })                                             │
    │  → Returns: URL to Google's OAuth consent page  │
    │                                                 │
    │  STEP 3: Open Chrome auth popup                 │
    │  chrome.identity.launchWebAuthFlow({            │
    │    url: "<google oauth url>",                   │
    │    interactive: true  ← show UI to user         │
    │  }, callback)                                   │
    │                                                 │
    │  User sees: Google account picker → consent     │
    │                                                 │
    │  STEP 4: Google redirects to chromiumapp.org    │
    │  Chrome intercepts this redirect               │
    │  Calls callback with the full redirect URL:     │
    │  "https://fgcm...org/#access_token=eyJ...       │
    │   &refresh_token=v1.MbQv...                    │
    │   &token_type=bearer                           │
    │   &expires_in=3600"                            │
    │                                                 │
    │  STEP 5: Parse tokens from URL hash             │
    │  const hashParams = new URLSearchParams(        │
    │    new URL(callbackUrl).hash.substring(1)       │
    │  );                                             │
    │  const accessToken = hashParams.get(            │
    │    'access_token'                               │
    │  );                                             │
    │  const refreshToken = hashParams.get(           │
    │    'refresh_token'                              │
    │  );                                             │
    │                                                 │
    │  STEP 6: Set Supabase session                   │
    │  await supabase.auth.setSession({               │
    │    access_token: accessToken,                   │
    │    refresh_token: refreshToken                  │
    │  });                                            │
    │  → Triggers chromeStorageAdapter.setItem()      │
    │  → Writes to chrome.storage.local.              │
    │    supabaseSession                              │
    └─────────────────────────────────────────────────┘
```

**Prerequisite configuration** (must be set up in Supabase dashboard):
- Supabase → Authentication → URL Configuration → Redirect URLs must include `https://<extension-id>.chromiumapp.org/`
- Google OAuth Client must allow the same redirect URI

### Token Lifecycle

```
    Token created (login/refresh)
    access_token: JWT, expires_in = 3600 seconds (1 hour)
    refresh_token: Opaque string, long-lived
                    │
                    ├── Every API call (api.js):
                    │   1. getAccessToken() → getSession()
                    │   2. supabase.auth.getSession()
                    │      → Checks if token expired
                    │      → If expired, auto-refreshes using refresh_token
                    │      → Writes new tokens to chrome.storage via adapter
                    │   3. Returns access_token
                    │   4. Sets Authorization: Bearer <access_token>
                    │
                    ├── On extension popup open:
                    │   1. Read supabaseSession from storage
                    │   2. If present, attempt getUser()
                    │   3. If token expired, refreshSession()
                    │   4. If refresh fails, show login screen
                    │
                    └── On signOut():
                        1. supabase.auth.signOut()
                        2. clearSupabaseSession() → removes from storage
                        3. chrome.runtime.sendMessage({type: 'LOGGED_OUT'})
                        4. Background clears: activeSession, stats, urls
```

---

## 26.7 — Complete Data Lifecycle: From User Input to PostgreSQL and Back

Here is the **complete, unbroken chain** of data transformations for a single focus session, following one datum through every layer and storage medium:

### Phase 1: Task "Working on ML homework" enters the system

```
USER KEYBOARD                              CHROME             NETWORK              RAILWAY             GOOGLE API           SUPABASE
    │                                      STORAGE                                                                         (PostgreSQL)
    │                                         │                    │                    │                    │                    │
    │ types "ML homework"                     │                    │                    │                    │                    │
    ├───popup textarea──────────────────►     │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │ clicks "Analyze Task"                   │                    │                    │                    │                    │
    ├───api.analyzeTask()──────────────────────────fetch()────────►│                    │                    │                    │
    │   {"taskDescription":"ML homework",     │                    │                    │                    │                    │
    │    "previousResponses":[]}              │                    │                    │                    │                    │
    │                                         │                    │  Zod validates     │                    │                    │
    │                                         │                    │  body shape        │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  analyzeTaskWithAI ───generateContent──►│                    │
    │                                         │                    │  (prompt: assess   │   {"ready":false,  │                    │
    │                                         │                    │   specificity)     │    "followUp":     │                    │
    │                                         │                    │                    │    ["What topic?"]}│                    │
    │                                         │                    │◄─────JSON──────────│◄───────────────────│                    │
    │◄──────────followUpQuestions─────────────────────────────────◄│                    │                    │                    │
    │   {ready:false, followUpQuestions:       │                    │                    │                    │                    │
    │    ["What topic?"]}                     │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │ types "neural networks PyTorch"         │                    │                    │                    │                    │
    │ clicks "Submit"                         │                    │                    │                    │                    │
    ├───api.analyzeTask(desc,[{Q,A}])──────────────────────────────►                    │                    │                    │
    │                                         │                    │  analyzeTaskWithAI ──────────────────────►                    │
    │                                         │                    │  (prev responses   │   {"ready":true,   │                    │
    │                                         │                    │   included)        │    "refinedTask":  │                    │
    │                                         │                    │                    │    "Working on     │                    │
    │                                         │                    │                    │     neural network │                    │
    │                                         │                    │                    │     homework using  │                    │
    │                                         │                    │                    │     PyTorch",      │                    │
    │                                         │                    │                    │    "durationMin":  │                    │
    │                                         │                    │                    │     120}           │                    │
    │◄──────────ready + refinedTask────────────◄───────────────────◄────────────────────◄────────────────────│                    │
    │                                         │                    │                    │                    │                    │
    │ clicks "Start Session"                  │                    │                    │                    │                    │
    ├───api.startSession(refined,warn,120)──────────────────────────►                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  expandTaskDesc────────────────────────────►                    │
    │                                         │                    │  "Working on       │                    │  "Working on neural│
    │                                         │                    │   neural network   │  gemini-2.0-flash  │  network homework  │
    │                                         │                    │   homework using   │  -lite expands:    │  using PyTorch...  │
    │                                         │                    │   PyTorch"         │  "...includes back │  backpropagation,  │
    │                                         │                    │                    │  propagation, CNN, │  CNN, RNN, LSTM,   │
    │                                         │                    │                    │  RNN, LSTM, arxiv, │  arxiv, Stack      │
    │                                         │                    │                    │  pytorch.org..."   │  Overflow, CUDA,   │
    │                                         │                    │◄─expanded string───◄────────────────────│  GPU computing..." │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  generateEmbedding─────────────────────────►                    │
    │                                         │                    │  (expanded string) │  text-embedding-   │                    │
    │                                         │                    │                    │  004 returns       │                    │
    │                                         │                    │◄──Float32[768]─────◄──[0.012,-0.056,...]│                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  createSession()───────────────────────────────────────────────►
    │                                         │                    │  INSERT INTO       │                    │  sessions table:   │
    │                                         │                    │  sessions (id,     │                    │  id: uuid          │
    │                                         │                    │   user_id,         │                    │  user_id: uuid     │
    │                                         │                    │   task_description,│                    │  task_description:  │
    │                                         │                    │   task_embedding,  │                    │  (expanded text)   │
    │                                         │                    │   block_mode,      │                    │  task_embedding:   │
    │                                         │                    │   expires_at)      │                    │  float8[768]       │
    │                                         │                    │                    │                    │  block_mode: warn  │
    │                                         │                    │◄─────────success───◄──────────────────────◄─────────────────────│
    │                                         │                    │                    │                    │                    │
    │◄────────{sessionId, expiresAt, ...}─────◄────JSON response──◄─                   │                    │                    │
    │                                         │                    │                    │                    │                    │
    │  popup.js stores in chrome.storage:     │                    │                    │                    │                    │
    ├──setActiveSession({sessionId,...})────────►activeSession      │                    │                    │                    │
    ├──resetSessionStats()─────────────────────►sessionStats={0s}  │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │  Notify background:                     │                    │                    │                    │                    │
    ├──sendMessage({type:'SESSION_STARTED'})   │                    │                    │                    │                    │
    │  → background clears recentUrls         │                    │                    │                    │                    │
    │                                         ├──recentUrls = {}   │                    │                    │                    │
```

### Phase 2: User visits reddit.com during the session

```
USER BROWSER                               CHROME             NETWORK              RAILWAY             GOOGLE API           SUPABASE
TAB NAV                                    STORAGE                                                                         (PostgreSQL)
    │                                         │                    │                    │                    │                    │
    │ navigates to reddit.com                 │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    ├───chrome.tabs.onUpdated fires──────────►│                    │                    │                    │                    │
    │   changeInfo.status = 'complete'        │                    │                    │                    │                    │
    │   tab.url = "https://reddit.com..."     │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   Guard checks:                         │                    │                    │                    │                    │
    │   ✓ status=complete                     │                    │                    │                    │                    │
    │   ✓ has URL                             │                    │                    │                    │                    │
    │   ✓ starts with http                    │                    │                    │                    │                    │
    │   ✓ not in SKIP patterns                │                    │                    │                    │                    │
    │   ✓ token exists                        │                    │                    │                    │                    │
    │   ✓ session exists & not expired ◄──────│ activeSession      │                    │                    │                    │
    │   ✓ not recently classified ◄───────────│ recentUrls = {}    │                    │                    │                    │
    │   ✓ not in Pomodoro break               │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   RACE CHECK #1: ✓                      │                    │                    │                    │                    │
    │   addRecentUrl(url, {decision:'pending'})│                    │                    │                    │                    │
    │                                    ─────► recentUrls[reddit] │                    │                    │                    │
    │                                         │  = {ts, pending}   │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   RACE CHECK #2: ✓                      │                    │                    │                    │                    │
    │   sendMessage(tabId, EXTRACT_CONTENT)   │                    │                    │                    │                    │
    │   → content.js:                         │                    │                    │                    │                    │
    │     document.cloneNode(true)            │                    │                    │                    │                    │
    │     new Readability(clone).parse()      │                    │                    │                    │                    │
    │     → {title:"r/programming", pageText} │                    │                    │                    │                    │
    │   ◄── {title, pageText}                 │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   RACE CHECK #4: ✓                      │                    │                    │                    │                    │
    │   classifyPage(sessionId, url,          │                    │                    │                    │                    │
    │                 title, pageText)         │                    │                    │                    │                    │
    │   → api.js truncates pageText to 10K    │                    │                    │                    │                    │
    │   → fetch(POST /v1/classify/web)─────────────────────────────►                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  authMiddleware:   │                    │                    │
    │                                         │                    │  jose.jwtVerify()  │                    │                    │
    │                                         │                    │  req.user = {sub}  │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  Zod validates body│                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  getSessionById()──────────────────────────────────────────────►
    │                                         │                    │  SELECT * FROM     │                    │  Returns session   │
    │                                         │                    │  sessions WHERE    │                    │  with task_embedding│
    │                                         │                    │  id='f47ac123-...'│                    │  Float8[768]       │
    │                                         │                    │◄─────SessionRow────◄──────────────────────◄────────────────────│
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  isProductivityTool──────────────────────►                    │
    │                                         │                    │  (url, title)      │  gemini-2.0-flash  │                    │
    │                                         │                    │                    │  -lite:            │                    │
    │                                         │                    │                    │  {"isTool": false} │                    │
    │                                         │                    │◄───false────────────◄────────────────────│                    │
    │                                         │                    │  (Reddit is NOT    │                    │                    │
    │                                         │                    │   a productivity   │                    │                    │
    │                                         │                    │   tool)            │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  generateEmbedding─────────────────────────►                    │
    │                                         │                    │  ("r/programming   │  text-embedding-   │                    │
    │                                         │                    │   Transformers..." │  004 returns       │                    │
    │                                         │                    │   10KB string)     │  Float32[768]      │                    │
    │                                         │                    │◄──contentEmbed─────◄────────────────────│                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  cosineSimilarity( │                    │                    │
    │                                         │                    │   taskEmbed[768],  │                    │                    │
    │                                         │                    │   contentEmbed[768]│                    │                    │
    │                                         │                    │  ) = 0.48          │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  getDecision(0.48) │                    │                    │
    │                                         │                    │  → "warn" (0.40≤   │                    │                    │
    │                                         │                    │    0.48<0.55)      │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  confidence = 0.32 │                    │                    │
    │                                         │                    │                    │                    │                    │
    │                                         │                    │  logClassification────────────────────────────────────────────►
    │                                         │                    │  Event (async, no  │                    │  INSERT INTO       │
    │                                         │                    │  await — fire &    │                    │  classification_   │
    │                                         │                    │  forget)           │                    │  events VALUES     │
    │                                         │                    │                    │                    │  (uuid, sessionId, │
    │                                         │                    │                    │                    │   'web', 'warn',   │
    │                                         │                    │                    │                    │   0.48, 0.32,      │
    │                                         │                    │                    │                    │   url, title,      │
    │                                         │                    │                    │                    │   reason, NOW())   │
    │                                         │                    │                    │                    │                    │
    │◄────{decision:"warn",confidence:0.32,───◄──JSON response─────◄                   │                    │                    │
    │      similarity:0.48, reason:"..."}     │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   RACE CHECK #5: ✓                      │                    │                    │                    │                    │
    │   addRecentUrl(url, result)──────────────► recentUrls[reddit] │                    │                    │                    │
    │                                         │  = {ts, {warn,...}}│                    │                    │                    │
    │   incrementStat('pagesChecked')──────────► pagesChecked: 48  │                    │                    │                    │
    │   incrementStat('pagesWarned')───────────► pagesWarned: 7    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   showOverlay(tabId, result, 'warn')    │                    │                    │                    │                    │
    │   → sendMessage(tabId, {                │                    │                    │                    │                    │
    │       type:'SHOW_OVERLAY',              │                    │                    │                    │                    │
    │       decision:'warn',                  │                    │                    │                    │                    │
    │       reason:'Content is loosely...',   │                    │                    │                    │                    │
    │       blockMode:'warn',                 │                    │                    │                    │                    │
    │       confidence: 0.32                  │                    │                    │                    │                    │
    │     })                                  │                    │                    │                    │                    │
    │   → content.js: showOverlay()           │                    │                    │                    │                    │
    │     Injects #siren-overlay into DOM     │                    │                    │                    │                    │
    │     body.style.overflow = 'hidden'      │                    │                    │                    │                    │
    │                                         │                    │                    │                    │                    │
    │   USER SEES: Full-screen overlay        │                    │                    │                    │                    │
    │   "This page may be off-topic"          │                    │                    │                    │                    │
    │   [← Go Back] [Continue Anyway →]       │                    │                    │                    │                    │
    │   [Challenge: improperly blocked?]      │                    │                    │                    │                    │
```

---

## 26.8 — Data Where All Data Lives (Summary Table)

| Data | Storage Medium | Format | Written By | Read By | Lifetime |
|------|---------------|--------|-----------|---------|----------|
| User credentials (Google OAuth) | Supabase `auth.users` | PostgreSQL row | Supabase Auth | Backend auth middleware | Permanent |
| JWT access token | `chrome.storage.local` | JSON string (1.5KB) | Supabase SDK via custom adapter | `api.js` every API call | 1 hour (auto-refreshed) |
| JWT refresh token | `chrome.storage.local` | JSON string | Supabase SDK | Supabase SDK on token refresh | ~30 days |
| Active session config | `chrome.storage.local` | JSON object (~500B) | `popup.js` on session start | `background.js` every tab event | Until session end/expiry |
| Session + embedding | Supabase `sessions` table | PostgreSQL row + float8[] | Backend `session.ts` | Backend `classify.ts` | Until session end (then deleted) |
| Raw task description | Supabase `sessions.task_description` | TEXT | Backend after LLM expansion | Backend classification pipeline | Session lifetime |
| Task embedding | Supabase `sessions.task_embedding` | float8[768] | Backend after `generateEmbedding()` | Backend cosine similarity | Session lifetime |
| Content embedding | **In-memory only** (never persisted) | Float32[768] | Backend `generateEmbedding()` | Backend `cosineSimilarity()` | ~1ms (single request) |
| Classification result | Supabase `classification_events` | PostgreSQL row | Backend fire-and-forget | Analytics (future) | Permanent |
| Classification cache | `chrome.storage.local.recentUrls` | JSON object (≤50KB) | `background.js` | `background.js` debounce check | 30-60 seconds (cleaned per interval) |
| Session statistics | `chrome.storage.local.sessionStats` | JSON object (~100B) | `background.js` per classification | `popup.js` for display | Session lifetime |
| Page content | **In-memory only** (content script) | `{title: string, pageText: string}` | `content.js` Readability | `background.js` → API | ~500ms (single request) |
| DOM clone for Readability | **In-memory only** (content script) | DOM tree (~500KB-5MB) | `document.cloneNode(true)` | Readability parser | ~200ms (parsing duration) |
| Overlay DOM elements | Host page DOM | HTML elements + CSS | `content.js` `showOverlay()` | User's eyes | Until dismissed/navigated |
| Tab URL tracker | Service worker memory | `Map<number, string>` | `background.js` on tab update | Race condition checks | Service worker lifetime |
| Pomodoro timer state | `chrome.storage.local.activeSession.pomodoro` | JSON sub-object | `popup.js` on session start | `background.js` break check | Session lifetime |

---

## 26.9 — What the User's Flow Diagram Envisions (vs. Current Implementation)

The user's architecture diagrams show several features that represent the **target vision** beyond the current implementation:

| Diagrammed Feature | Current Implementation | Gap |
|-------------------|----------------------|-----|
| "User explains project → LLM judges specificity → asks clarifying questions" | ✅ Fully implemented via `analyzeTask()` loop | None |
| "LLM enriches (lengthens) project answer and embeds it" | ✅ Fully implemented via `expandTaskDesc()` + `generateEmbedding()` | None |
| "Is it a tool? → Tool-judging agent" | ✅ Implemented via `isProductivityTool()` LLM check | None |
| "DOM embedded compared with project embedding" | ✅ Implemented via `cosineSimilarity()` | None |
| "BLOCKED - user has option to dispute → Appeals agent judges with LLM" | ✅ Implemented via `challenge` endpoint + `analyzeContentRelevance()` | None |
| "Periodic screenshot, DOM capture, and keystroke listener" | ❌ Not yet implemented | Future: continuous monitoring |
| "Distraction agent (judges if sites are being used in distracting way)" | ❌ Not yet implemented | Future: usage pattern analysis |
| "Activity log agent" | ⚠️ Partial: `classification_events` logs exist | Future: richer activity tracking |
| "Project agent (keeps running document understanding the user's project)" | ❌ Not yet implemented | Future: evolving task model |

The current system is a **faithful implementation of the core pipeline** (task input → expansion → embedding → classification → challenge). The diagrammed future vision adds continuous behavioral monitoring, which would require the `screen` and `app` event types already stubbed in the `ClassificationEventRow` type.

---

*These three chapters (24, 25, 26) provide an exhaustive, forensic-depth understanding of every data structure, transformation, and flow in the Siren/Ithaca system. You should now be able to whiteboard the entire architecture from memory.*
