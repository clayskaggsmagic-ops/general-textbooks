# The Big Picture: How to Think About Data & System Design (Without Being an Engineer)

> *Read this chapter first. It gives you the zoom-out mental model that makes every other chapter in this book click.*

---

## Why This Chapter Exists

You're not an engineer. You're not going to be writing code at Palantir. But in every interview round — Technical Comprehension, Decomposition, and Analytical — they will ask you to **design something**. Not code it. *Design* it.

That means you need to think about:
- What data exists (or should exist)
- How it's organized
- What a user sees
- What happens when they click a button
- How the whole thing gets rolled out

This chapter gives you the **five lenses** to look through when you hear any design prompt, and the **six tensions** you'll navigate in every answer. Internalize these and you'll never freeze when an interviewer says "walk me through your approach."

---

## The Palantir Toolbox — What Their Products Actually Are

Before anything else, you should know what Palantir's tools do — in plain English, not marketing language.

### Foundry — The Data Engine

**What it is:** Foundry is where all the data lives and gets cleaned up. Think of it as a massive, smart filing cabinet that can take messy data from dozens of different sources (spreadsheets, databases, sensors, government records) and organize them into one clean, connected view.

**What happens inside Foundry:**
1. **Raw data comes in** — from a client's existing systems (their sales database, their sensor feeds, their Excel reports)
2. **Pipelines clean and transform it** — fix typos, standardize formats, remove duplicates, fill in gaps
3. **The Ontology connects it** — raw tables become meaningful objects (a row in a spreadsheet becomes a "Store" with properties and connections)

**What to say in the interview:** "The raw data would be ingested into Foundry, where we'd build pipelines to clean, normalize, and join the data sources into a unified Ontology."

### Workshop — The User-Facing App Builder

**What it is:** Workshop is Palantir's tool for building the apps that real users interact with. If Foundry is the engine under the hood, Workshop is the dashboard and steering wheel.

**You don't write code to build Workshop apps.** You configure them — drag and drop tables, charts, maps, buttons, and action triggers. A DS designs what the Workshop app should show and do; an FDE builds it.

**What to say in the interview:** "The end-user would interact with this through a Workshop app — a map view showing risk scores by location, with action buttons to dispatch inspectors or escalate alerts."

### The Ontology — The Map of Everything

**What it is:** The Ontology is Palantir's name for the connected data model that sits on top of all the raw data. It's not a product you install — it's the *structure* you design.

Imagine all of a client's data is a pile of puzzle pieces from 20 different puzzles. The Ontology is the picture on the box that shows how all the pieces connect.

**Concretely, the Ontology has four parts:**

| Part | What It Is | Example |
|------|-----------|--------|
| **Object Types** | The real-world things | Store, Product, Customer, Order |
| **Properties** | What you know about each thing | Store → location, sqft, region, manager |
| **Links** | How things connect | Store → stocks → Product; Customer → places → Order |
| **Actions** | What a user can *do* | "Accept Order," "Dispatch Inspector," "Override Recommendation" |

When you say "I'd model this in the Ontology" in an interview, you mean: "I'd define the objects, their properties, how they connect, and what users can do with them." **That's it.**

### AIP — The AI Layer

**What it is:** AIP (Artificial Intelligence Platform) is Palantir's layer for running AI models on top of Foundry data. You don't need to explain how AI works — just know that AIP lets users ask questions in natural language, run predictions, and get automated insights without writing code.

**What to say in the interview:** "AIP could power the natural-language interface — letting a field commander ask 'which 5 bases have the lowest readiness?' and get an instant answer without building a custom query."

### FDE — Your Engineering Partner

**FDE** stands for **Forward Deployed Engineer**. They are the software engineers who work alongside Deployment Strategists.

**The relationship:** You (the DS) figure out *what to build* — talking to clients, designing the Ontology, defining the user experience. The FDE *builds it* — writing the code, constructing the pipelines, configuring Workshop.

**What to say in the interview:** "I'd partner with an FDE to build the data pipelines and Workshop app. My role would be designing the Ontology, defining the user workflows, and managing the client relationship."

---

## Concepts That Sound Scary But Aren't

These terms come up constantly. Here's what they actually mean.

### Data Pipeline ("How does raw data become useful?")

A **pipeline** is just a series of steps that take raw, messy data and turn it into clean, organized data. Think of it like a water treatment plant:

```
Raw data (dirty)  →  Clean  →  Standardize  →  Connect  →  Clean data (usable)
     ↑                ↑           ↑              ↑
  Excel files     Remove       Make all       Link Store
  with typos      duplicates   dates the      to its Orders
                               same format    and Products
```

You don't need to build pipelines. You need to know they exist and what they do: take messy inputs and produce clean outputs.

### ETL ("Extract, Transform, Load")

**ETL** is a three-letter acronym (say "E-T-L") for the pipeline process:
- **Extract:** Pull data out of the client's existing systems
- **Transform:** Clean it, fix it, reshape it
- **Load:** Put the clean version into Foundry

**What to say if it comes up:** "We'd build an ETL pipeline in Foundry — extracting data from their POS system and inventory database, transforming it to standardize formats and handle missing values, and loading it into the Ontology."

### API ("Application Programming Interface")

**API** is one of those terms that sounds intimidating but is actually simple. Think of it like a **restaurant menu:**

- The menu lists everything you can order (the available requests)
- You tell the waiter your order (make a request)
- The kitchen prepares it (the server processes it)
- The waiter brings your food (the response comes back)

You never go into the kitchen. You don't need to know how the food is cooked. You just need to know what's on the menu and how to order.

**In data terms:** An API is how one system talks to another. When a Workshop app needs to pull live weather data from a weather service, it calls the weather service's API: "give me the current temperature for zip code 10001." The weather service sends back "72°F." Done.

**Why it matters for Palantir:** APIs are how Foundry connects to external data sources (pulling data in), and how Workshop apps trigger actions in other systems (pushing data out). You don't build APIs — FDEs do. But you should know they exist and what they do.

**What to say:** "The Workshop app would call the prediction model via an API — the app sends the patient's vitals, and the model returns a risk score. The user never sees the API; they just see the risk score appear on their screen."

### Security and Access Control ("Who Can See What?")

For government and healthcare clients — Palantir's biggest markets — **not everyone should see everything.** This is called **RBAC** (Role-Based Access Control, say "R-back").

**The concept:** Different people get different permissions based on their role:

```
ROLE               CAN SEE                     CAN DO
─────────────────  ──────────────────────────  ─────────────────────
Field Analyst      Their own region's data      View, filter, export
Regional Manager   All regions' data            View, filter, reassign
Administrator      All data + user management   Everything + add users
Auditor            Read-only access to logs     View who accessed what
```

**Why it matters:** In government work, data is classified (Unclassified → CUI → Secret → Top Secret). A system that doesn't enforce "need-to-know" won't pass security review. In healthcare, HIPAA requires that only authorized staff can see patient records.

**What to say in the interview:** "I'd design the access model so caseworkers see only their assigned cases, supervisors see their team's full caseload, and the regional director gets an aggregated view without individual patient details. We'd also want an audit trail — a log of who accessed what, and when — for compliance."

**The one-liner:** "Not every user should see every data point. The system should enforce role-based access — field users see their scope, managers see the rollup, and everything is logged."

### Grain / Granularity ("How detailed is each row?")

**Grain** means: what does ONE row in your data represent?

| Grain | One row = | Detail level |
|-------|----------|-------------|
| **Fine** (transaction-level) | One strawberry purchase at one store at 2:47 PM | Very detailed, very large dataset |
| **Medium** (daily summary) | Total strawberry sales at Store #47 on Tuesday | Good balance of detail and size |
| **Coarse** (monthly summary) | Total strawberry sales at Store #47 in March | Small dataset, less detail |

**Why it matters:** If you have daily grain, you can answer "which day of the week sells the most strawberries." If you only have monthly grain, you can't — that detail is lost. Fine grain = more questions answered, bigger data. Coarse grain = fewer questions answered, faster queries.

**What to say:** "I'd want the data at daily grain — one row per store per product per day — so we can detect day-of-week patterns and weekly trends."

### Data Quality ("What does 'bad data' actually look like?")

When people say "bad data quality," they mean one or more of these concrete problems:

| Problem | What It Looks Like | Why It Matters |
|---------|-------------------|---------------|
| **Missing values** | A store has no sales recorded for Tuesday | Was the store closed, or did the system fail? You don't know. |
| **Duplicates** | The same order appears twice | Your revenue numbers are inflated by 2x. |
| **Inconsistent formats** | One system stores dates as "03/15/2026" and another as "2026-15-03" | The pipeline crashes or produces wrong results. |
| **Stale data** | Inventory counts are from last week, not today | You're making decisions on old information. |
| **Wrong values** | A store's lat/long coordinates place it in the ocean | Map views are broken; distance calculations are wrong. |

**What to say:** "Before building anything, I'd run a data quality audit — check for missing values, duplicates, and format inconsistencies. The model is only as good as the data it's trained on."

### API ("Application Programming Interface")

**Plain English:** An API is a *door* that lets one system talk to another. When your weather app shows today's forecast, it's calling the weather service's API: "Hey, give me the forecast for Austin, TX" → the API sends back the data.

**Why it matters:** When you say "we'd integrate weather data," what you really mean is "we'd call a weather API to pull forecast data into Foundry automatically."

### KPI ("Key Performance Indicator")

A **KPI** is just the number that tells you whether you're winning or losing. For a grocery chain trying to reduce waste, the KPI might be "percentage of perishable goods wasted per store per week." For a dispatch system, it might be "average response time in hours."

**What to say:** "The primary KPI would be waste percentage by store — if we reduce it from 30% to 15%, the project is working."

---

## The Five Lenses

Every system — whether it's a wildfire tracker, a grocery ordering tool, or a veteran homelessness dashboard — has the same five layers. When you hear a prompt, mentally walk through these in order:

```
┌─────────────────────────────────────────────────┐
│                                                  │
│   1. USERS         Who uses this? What decision  │
│                    do they make, how often?       │
│                                                  │
│   2. DATA          What information exists or     │
│                    needs to exist?                │
│                                                  │
│   3. LOGIC         What does the system compute,  │
│                    predict, or flag?              │
│                                                  │
│   4. SURFACES      What does the user actually    │
│                    see and interact with?         │
│                                                  │
│   5. DEPLOYMENT    How does this get rolled out   │
│                    in the real world?             │
│                                                  │
└─────────────────────────────────────────────────┘
```

Most candidates jump straight to #3 (the logic, the model, the algorithm). **You should start at #1 and work down.** That's what makes you sound like a Deployment Strategist instead of a data scientist.

---

### Lens 1: Users — "Who and What Decision?"

Before you touch data or systems, answer two questions:

1. **Who is the primary user?** (Not "the client" — the specific human. A store manager? A caseworker? A general on a base? An analyst at HQ?)
2. **What decision do they make, and how often?** (Daily dispatch? Weekly reorder? Real-time triage?)

**Why this matters:** The same data can power completely different systems depending on who's using it. A field technician needs a mobile task list with one-tap actions. An executive needs a quarterly trend dashboard. Design for the wrong user and you build the wrong thing.

**What to say in the interview:**

> "Before I design anything, I want to understand who's sitting in front of this system every day. A store manager checking orders on Monday morning needs a very different interface than a VP reviewing quarterly waste metrics."

---

### Lens 2: Data — "What Exists, What's Missing?"

You don't need to design databases. You need to think about data at three levels:

**Level A — What are the "things" in this problem?**

Every problem has real-world objects. List them. For a grocery chain: Stores, Products, Orders, Suppliers. For wildfire prediction: Power Lines, Weather Stations, Vegetation Zones, Inspection Records.

In Palantir language, these are **Object Types** — the nouns of the system.

**Level B — What do we know about each thing?**

Each object has attributes. A Store has a location, size, region, and manager. A Power Line has an age, material, last inspection date, and voltage.

In Palantir language, these are **Properties**.

**Level C — How do the things connect?**

A Store *stocks* Products. An Order is *placed by* a Customer *at* a Store. A Power Line *is inspected in* an Inspection Record.

In Palantir language, these are **Links**.

**What to say in the interview:**

> "The core data entities here are [list 4-5 nouns]. Each Store would have properties like location, sqft, and region. Stores are linked to Products through an inventory relationship, and to Orders through a sales relationship. This is the Ontology backbone."

**The data question you should always raise:**

> "I'd want to know what data actually exists today vs. what we'd need to build. If they have POS data but no inventory tracking, that changes my approach — I'd start with what's available and layer in new data sources later."

---

### Lens 3: Logic — "What Does the System Actually Do?"

This is where most people's brains go first, but it should be third. There are really only **four types of logic** that systems do:

| Type | What It Does | Example |
|------|-------------|---------|
| **Describe** | Show what happened | "Last month, Store #47 had 30% food waste" |
| **Diagnose** | Explain why | "The waste concentrated in produce because orders were 2x actual demand" |
| **Predict** | Forecast what will happen | "Next week, Store #47 will sell ~120 strawberry units based on history + weather" |
| **Prescribe** | Recommend what to do | "Reduce the strawberry order from 200 to 130 units" |

**The key insight:** In the interview, you don't need to explain HOW a prediction model works (that's engineering). You need to explain WHAT it does and WHY that helps the user make a better decision.

**What to say in the interview:**

> "The system would do two things: first, a descriptive layer showing current waste rates by store and category — that's the dashboard. Second, a predictive layer that forecasts demand 7 days out so the system can recommend order quantities. The manager sees 'we predict you'll sell 120 strawberry units, but you're about to order 200 — here's why we recommend 130.'"

**You never need to say:** "We'd use a gradient-boosted tree with cross-validated hyperparameter tuning." Just say: "A forecasting model trained on historical sales, weather, and seasonality."

---

### Lens 4: Surfaces — "What Does the User See and Click?"

This is where Deployment Strategists shine vs. data scientists. A DS doesn't just build a model — they design the **experience**.

There are four surface types you should know:

| Surface | What It Is | When to Use It |
|---------|-----------|---------------|
| **Dashboard** | Read-only display of metrics and trends | Executive monitoring, KPI tracking |
| **Workflow App** | Interactive tool with buttons, lists, and actions | Daily operations — dispatch, triage, approvals |
| **Alert / Notification** | Push-based: the system comes to you | Time-critical events — equipment failure, risk threshold breach |
| **Report** | Periodic summary document | Compliance, board presentations, audits |

**The most impressive thing you can do in an interview** is describe what the user's screen looks like. Not in technical detail — just paint the picture:

> "The store manager opens a Workshop app Monday morning. They see a list of recommended orders by category. Each row shows: item name, current order quantity, recommended quantity, and a risk indicator (high/medium/low waste risk). They can accept the recommendation, adjust it, or override with a reason. That reason feeds back into the model."

**The word that matters most here: Actions.** What can the user *do*? Not just see — *do*. Click "Accept Order." Click "Dispatch Inspector." Click "Escalate to Supervisor." This is Palantir's core philosophy: data should drive **action**, not just **charts**.

---

### Lens 5: Deployment — "How Does This Actually Get Used?"

This is the secret weapon. Most candidates design a perfect system and then stop. Palantir interviewers are specifically testing whether you think about the messy reality of getting humans to actually use the thing.

**Three things to address:**

**Phased Rollout:**
> "I wouldn't deploy to all 1,200 stores at once. Phase 1: 50 pilot stores with the highest waste — prove the concept, learn what breaks. Phase 2: Roll to all stores in one region. Phase 3: National rollout."

**Change Management:**
> "The biggest risk isn't the technology — it's adoption. Store managers who've been ordering by gut for 20 years will resist an algorithm. I'd frame it as 'assistance, not replacement' — the system recommends, they decide. And I'd start with the worst-performing stores where the improvement is most visible."

**Feedback Loop:**
> "When a manager overrides the recommendation, the system should capture *why*. 'I know there's a school event this weekend' becomes a data point that improves the model. The system gets smarter from human judgment, not just historical data."

---

## The Six Recurring Design Tensions

Every design prompt forces you to navigate trade-offs. Interviewers aren't looking for the "right" answer — they're looking for you to **name the tension, explain both sides, and pick one with reasoning.**

Here are the six tensions that come up in almost every prompt:

---

### Tension 1: Predict & Suggest vs. Automate & Act

**The question:** Should the system show a human the answer and let them decide, or should it just do the thing automatically?

| Lean Toward Suggest | Lean Toward Automate |
|--------------------|--------------------|
| High stakes (child welfare, military targeting) | Low stakes (reorder paper towels, schedule maintenance window) |
| Low trust in the system (new deployment) | High trust (proven system, 6+ months of accuracy) |
| Legal/regulatory accountability required | No legal exposure |
| Complex judgment needed | Simple rule-based logic |

**Interview phrase:**
> "I'd start with decision support — show the recommendation and let the user act. Once the system proves itself over a few months, we can automate the low-risk actions while keeping high-stakes decisions human-in-the-loop."

---

### Tension 2: Real-Time vs. Batch

**The question:** Does the system need to react in seconds, or is a daily/weekly update fine?

**Simple rule of thumb:**
- If someone is *waiting* for the answer (dispatcher, operator, ER nurse) → **real-time**
- If someone *reviews* the answer at a scheduled time (executive, planner, analyst) → **batch**

**Interview phrase:**
> "The dispatch layer needs to be real-time — a technician can't wait until tomorrow to learn their next job. But the analytics dashboard? That can refresh overnight. I'd build both: a real-time operational layer and a batch reporting layer."

---

### Tension 3: Start With What Exists vs. Build What's Ideal

**The question:** Do we work with messy, incomplete data that exists today, or do we invest time building the perfect data pipeline first?

**Almost always start with what exists.** The Palantir philosophy is "just start with the data you have." You can layer in better data sources later.

**Interview phrase:**
> "Ideally we'd have IoT sensor data on every asset. But they probably don't have that yet. I'd start with the maintenance logs and inspection records they already keep — that gets us 70% of the signal. Then in Phase 2, we'd pilot sensors on the 50 highest-risk assets and integrate that data."

---

### Tension 4: One Platform vs. Tailored Views

**The question:** Do all users share one screen, or does each group get their own interface?

**Palantir's answer (and yours):**
> "One Ontology, multiple apps. The data model is shared — that's the source of truth. But the executive sees a scorecard, the field team sees a task list, and the analyst sees the full data pipeline. Same data, different surfaces."

This is almost always the right answer because it prevents data silos (everyone is working from the same facts) while giving each user exactly what they need.

---

### Tension 5: Precision vs. Recall

**Non-technical translation:**
- **Precision** = "Of the things we flagged, how many were actually real?" (Are we crying wolf?)
- **Recall** = "Of all the real cases out there, how many did we catch?" (Are we missing things?)

**Simple decision rule:**
- If **missing a real case is catastrophic** (wildfire, child abuse, fraud) → optimize for **recall** (catch everything, accept some false alarms)
- If **false alarms cause fatigue** and people start ignoring the system → optimize for **precision** (only flag what you're confident about)

**Interview phrase:**
> "The cost asymmetry here is extreme. Missing a real wildfire-risk line could kill people; inspecting a line that turns out to be fine costs a few thousand dollars. So I'd tune for high recall — catch every risky segment — and accept the false positive cost."

---

### Tension 6: Speed to Deploy vs. Completeness

**The question:** Do we ship something imperfect in 4 weeks, or build the full solution in 6 months?

**Palantir's answer (and yours): Ship fast, iterate.**

**Interview phrase:**
> "I'd aim for a usable MVP in 4-6 weeks — even if it only covers 3 of the 5 sub-problems. Get it in front of real users, learn what they actually need vs. what we assumed, and iterate. The worst outcome is spending 6 months building a perfect system that nobody uses because we got the user experience wrong."

---

## The 30-Second Framework — Use This for Any Prompt

When you hear any design prompt, mentally walk through this:

```
1. RESTATE the problem (10 sec)
   → "So they want to reduce food waste across 1,200 stores..."

2. ASK 2-3 questions (2 min)
   → Who uses this? What data exists? How is this handled today?

3. NAME THE OBJECTS (1 min)
   → "The core entities are Stores, Products, Orders, and Suppliers."

4. DESCRIBE THE LOGIC (3 min)
   → "The system would predict demand and recommend order quantities."

5. PAINT THE SCREEN (2 min)
   → "The manager opens the app Monday, sees a recommended order list..."

6. NAME AN ACTION (30 sec)
   → "They click 'Accept' to place the order, or 'Override' to adjust."

7. DESCRIBE ROLLOUT (1 min)
   → "Phase 1: 50 pilot stores. Phase 2: regional. Phase 3: national."

8. NAME A TRADE-OFF (30 sec)
   → "The tension is between automated ordering vs. human approval..."
```

**Total time: ~10 minutes.** That leaves 35 minutes for the interviewer to push back, drill deeper, and test your flexibility — which is exactly what they're evaluating.

---

## The Vocabulary Cheat Sheet

You don't need to understand these deeply. You just need to use the right words at the right time so the interviewer hears "this person speaks our language."

| When You Mean | Say This (Palantir Term) |
|--------------|-------------------------|
| The real-world things in the system (stores, people, assets) | **Object Types** |
| What you know about each thing (name, location, date) | **Properties** |
| How things connect to each other | **Links** |
| What a user can *do* in the system (approve, dispatch, escalate) | **Actions** |
| The entire data model — objects, properties, links, actions together | **The Ontology** |
| The data processing and cleaning pipeline | **Foundry pipeline** |
| The user-facing app where people see data and take actions | **Workshop app** |
| Deploying in stages to reduce risk | **Phased rollout** |
| Starting with the simplest version and improving | **MVP** (Minimum Viable Product) |
| Checking if the system output makes sense | **Validation** |
| The system learning from user corrections | **Feedback loop** |

---

## The Data Quality Playbook — What Goes Wrong With Real Data (And What to Say About It)

Here's a truth nobody puts in the marketing brochure: **80% of a real Palantir deployment is cleaning up messy data.** The Ontology, the Workshop apps, the AI recommendations — none of it matters if the underlying data is garbage. Interviewers know this. If you can speak to data quality issues fluently, you instantly sound like someone who's worked with real data — not just studied it.

### The 5 Data Problems You'll See Everywhere

| # | Problem | What It Looks Like | Interview Example |
|---|---------|-------------------|--------------------|
| 1 | **Missing values** | Critical fields are blank — no address, no timestamp, no ID | "30% of patient records have no discharge date. Do we exclude them or impute?" |
| 2 | **Duplicates** | The same entity appears multiple times with slight variations | "John Smith, J. Smith, and Jonathan Smith at the same address — is that one customer or three?" |
| 3 | **Format inconsistency** | Dates as "01/15/2025" in one system, "2025-01-15" in another, "Jan 15th" in a third | "Before we can join these tables, we need to standardize the date format and timezone" |
| 4 | **Stale data** | Information that was accurate 6 months ago but isn't now | "The inventory system says Store #42 has 200 units, but the last sync was 3 weeks ago" |
| 5 | **Schema mismatches** | Two systems use different names for the same thing, or the same name for different things | "System A calls it 'revenue,' System B calls it 'gross_sales' — are these the same field?" |

### How to Talk About Data Quality in the Interview

When you're designing a system, **always mention data quality early** — don't wait for the interviewer to poke at it. Here's how:

> *"Before we build anything, the first thing I'd do is a data audit. What systems exist? What format is the data in? How fresh is it? In my experience, the biggest risk isn't the model or the dashboard — it's that the underlying data has gaps, duplicates, or format issues that make the output unreliable."*

Then, when the interviewer asks "what could go wrong?":

> *"Three likely data quality issues: (1) duplicate records — if two systems track the same customer differently, we'll double-count. I'd build a deduplication step in the pipeline using email + address as a matching key. (2) Missing values — if 30% of records lack a critical field, we need to decide: exclude them, impute, or flag for manual review? That's a conversation with the client. (3) Staleness — if data only refreshes weekly, our 'real-time' alerts are actually 7 days behind."*

### The Golden Rule of Data Quality

> **Never assume the data is clean. Never assume the client knows it's dirty.**

In real deployments, the client will tell you "our data is great." It will not be great. The first two weeks of every engagement are spent discovering exactly how not-great it is. If you can articulate this in the interview, you sound like a veteran.

---

## What NOT to Say

| Don't Say | Why It's Bad | Say This Instead |
|-----------|-------------|-----------------|
| "We'd use machine learning" (alone) | Too vague. ML is a tool, not a solution. | "A forecasting model trained on historical sales and weather would predict demand 7 days out." |
| "We'd build a dashboard" (alone) | Passive. Dashboards are charts, not operating systems. | "A Workshop app where the manager sees recommendations and can approve, adjust, or override them." |
| "I'd collect all the data and analyze it" | No structure. No specificity. | "I'd start by understanding what data exists — POS records, inventory logs, weather — and build the Ontology around Stores, Products, and Orders." |
| "That's a hard problem" | Doesn't show you how to deal with hard problems. | "That's complex because [specific reason]. I'd simplify by starting with [specific scope] and expanding." |
| "I'm not sure, I'm not technical" | Undersells yourself. You're applying for a technical-adjacent role. | "I'd want to partner with an FDE to validate the technical approach, but at a high level, here's how I'd structure it..." |

---

## Worked Mini-Example: All Five Lenses On One Prompt

Let's apply everything to a single prompt so you can see the full flow.

**Prompt:** *"A city fire department wants to reduce response times. They currently average 8 minutes, and the target is 5 minutes. Walk me through your approach."*

---

**Lens 1 — Users:**
> "The primary user is the dispatch coordinator — the person who decides which fire station sends which truck to which call. They make this decision dozens of times per day, in real time, under pressure. There's also a secondary user: the fire chief, who reviews weekly performance reports."

**Lens 2 — Data:**
> "The core Object Types are: **Station** (location, truck count, staffing level), **Truck** (type, current status, GPS location), **Call** (location, severity, timestamp, type — fire vs. medical vs. hazmat), and **Response** (which truck dispatched, departure time, arrival time, response time)."
>
> "Links: a Station → houses → Trucks. A Call → dispatched → Truck. A Response → resolves → Call."
>
> "I'd want to know: do they have GPS on trucks? Do they have historical call data? Do they have traffic data or real-time road conditions? That changes how sophisticated the recommendations can be."

**Lens 3 — Logic:**
> "Two layers. First, **descriptive**: show which stations and neighborhoods have the longest response times — where are the bottlenecks? Maybe Station #12 covers a huge area, or the bridge to the east side creates a 3-minute delay. Second, **prescriptive**: when a call comes in, the system recommends which truck to send based on distance, traffic, truck type, and current availability — not just 'send the closest station' but 'send the truck that will arrive fastest given real-time conditions.'"

**Lens 4 — Surfaces:**
> "The dispatcher's Workshop app shows a map with all active trucks (green = available, yellow = en route, red = on scene). When a new 911 call comes in, the system highlights the recommended truck with an estimated arrival time: 'Truck 7 from Station #3 — ETA 4 min 20 sec.' The dispatcher clicks 'Dispatch' to confirm, or picks a different truck with one click."
>
> "The fire chief gets a weekly dashboard: average response time by neighborhood, busiest hours, stations that are consistently over target. Actions available: 'Propose Station Relocation Study,' 'Request Additional Truck.'"

**Lens 5 — Deployment:**
> "Phase 1: integrate historical call data and build the descriptive dashboard — show where the problems are. No fancy optimization yet, just visibility. Phase 2: add real-time GPS tracking and the dispatch recommendation engine at 5 pilot stations. Phase 3: city-wide rollout."
>
> "Change management risk: dispatchers with 20 years of experience trust their gut. I'd frame it as: 'the system handles the math — distance, traffic, availability — so you can focus on judgment calls like severity and crew fatigue.' And I'd show them their own data: 'Last month, the system's recommendation would have beaten manual dispatch on response time in 73% of cases.'"

**Trade-off named:**
> "The tension is real-time vs. batch. Dispatch recommendations need to be real-time — someone's house is on fire. But the performance analytics for the chief can be a weekly batch report. I'd build the real-time layer first because that's where the life-saving impact is."

---

**That entire answer takes about 8-10 minutes.** It covers all five lenses, names specific Object Types and Properties, paints the user's screen, describes Actions, handles deployment and change management, and names a trade-off. That's a strong decomposition answer — and you didn't need to mention a single algorithm, programming language, or database technology.

---

*This chapter is your mental anchor. Before reading any other chapter, before doing any practice prompt, internalize these five lenses and six tensions. Everything else in this book is detail work within this framework.*
