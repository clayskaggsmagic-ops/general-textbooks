# Chapter 32: The Consulting Case Playbook — What to Do When It's "Save This Business"

> *This chapter assumes the interview is closer to a traditional consulting case than a deep technical system design. The question will sound like: "Here's a business with a problem. What do you do?" Your job is product thinking, business logic, and big-picture data strategy — not writing SQL or drawing database schemas.*

---

## 32.1 The Mindset Shift

Forget everything about WAL mode, 3NF, and ETL pipelines. If the interviewer gives you a business problem, they want to see:

1. **Can you listen?** Do you actually hear the problem, or do you start solving immediately?
2. **Can you structure ambiguity?** Can you take "this business is dying" and turn it into 3-4 clear areas to investigate?
3. **Can you prioritize?** You can't fix everything. What's the one thing that matters most, and why?
4. **Do you think about people?** Who uses this? Who pays for it? Who's resisting? Who benefits?
5. **Can you propose something concrete?** Not "leverage AI" — but "build a dashboard that shows X to Y people so they can make Z decision every morning"

### What They're NOT Testing

- Your ability to write code
- Your knowledge of specific databases or architectures
- Whether you can recite system design terminology
- Whether you know the "right" answer (there isn't one)

### What They ARE Testing

- Structured thinking under ambiguity
- Business intuition — do you understand how businesses actually work?
- Communication — can you explain your reasoning simply and persuasively?
- Prioritization — can you identify the 20% that drives 80% of the outcome?
- Product sense — can you imagine what a real person would actually use?

---

## 32.2 The 5-Step Framework — Use This Every Time

No matter what business problem they throw at you, walk through these five steps in order. Think of it as your anchor — you can always come back to it if you get lost.

### Step 1: LISTEN and RESTATE (1-2 minutes)

**What to do:**
- Let them finish talking. Don't interrupt.
- Restate the problem in your own words: "So if I'm understanding correctly, [company] is facing [problem], and the core question is [X]. Is that right?"
- Ask 2-3 clarifying questions:
  - "What's the timeline — are we talking about saving them this quarter, or building a 2-year strategy?"
  - "Who's the main decision-maker we'd be presenting to?"
  - "Are there any constraints I should know about — budget, headcount, regulatory?"

**Why this matters:** Interviewers say the #1 reason candidates fail is they start solving before they understand the problem. Restating buys you think time AND shows you're a good listener. FDEs spend half their time just understanding what the client actually needs.

### Step 2: MAP THE LANDSCAPE (2-3 minutes)

**What to do:**
Break the business into its major components. You don't need a fancy framework — just organize the universe of "things that could matter" into 3-5 buckets.

**The Universal Buckets (adapt to the specific case):**

| Bucket | What You're Asking |
|---|---|
| **Revenue** | How do they make money? What are the revenue streams? Which ones are growing/shrinking? |
| **Costs** | What are the biggest cost drivers? Are costs increasing faster than revenue? |
| **Customers** | Who are the customers? Are they churning? Are they satisfied? What do they actually want? |
| **Operations** | How does the business actually run day-to-day? Where are the bottlenecks? What's manual that should be automated? |
| **Competition** | Who else is in this space? What do competitors do better? What's the market trend? |

Say this out loud: "I'd like to think about this across a few dimensions — revenue, costs, customers, operations, and competitive landscape. Let me start with [whichever feels most promising]."

**Why this matters:** This shows structure. You've taken a vague "save this business" and turned it into a tractable investigation. The interviewer now knows you're organized.

### Step 3: DIG INTO THE PAIN (5-7 minutes)

**What to do:**
Pick the 1-2 buckets that seem most relevant and go deep. Ask questions. Hypothesize. Test your hypotheses.

**The pattern is always:**
1. "I'd hypothesize that [X is the problem] because [reasoning]"
2. "To test that, I'd want to know [specific data point]"
3. Interviewer gives you data (or says "let's say that's true")
4. "Okay, that confirms/contradicts my hypothesis. So the real issue is..."

**Example dialogue:**
> "My hypothesis is that they're losing customers because their product hasn't evolved with the market. To test that, I'd want to see their customer retention rate over the last 3 years, and ideally some exit survey data — why are people leaving? If they don't have exit surveys, that's itself a data gap we could fix."

**Key phrases to use:**
- "What does the data tell us about...?"
- "If I were the operator sitting in this seat every day, what would I need to see?"
- "What's the decision this person is making, and what information are they missing?"

**Why this matters:** This is the heart of the case. You're demonstrating analytical thinking — but notice it's not "technical" analysis. It's business analysis. You're reasoning about customers, markets, and operations, not about database schemas.

### Step 4: PROPOSE A SOLUTION (3-5 minutes)

**What to do:**
Based on what you've learned, propose something concrete. Not "use AI" — but a specific intervention that addresses the pain you identified.

**The Solution Formula:**
> "I would build [specific thing] that shows [specific information] to [specific person] so they can [specific action] every [time interval]."

**Good vs. Bad answers:**

| ❌ Bad (vague) | ✅ Good (concrete) |
|---|---|
| "They should leverage data analytics" | "I'd build a daily dashboard for store managers showing which products are trending down in their specific location, so they can adjust their floor display that morning" |
| "They need AI" | "I'd create an alert system that flags at-risk customers — anyone whose usage dropped 50% in the last month — and routes them to the retention team with a summary of their account history" |
| "Better data infrastructure" | "Right now their sales data and customer feedback live in two different systems. I'd set up a single view where the account manager can see both — revenue trend AND satisfaction scores — for each client, so they stop being blindsided by churn" |

**Why this matters:** This is where you show product sense. The interviewer is checking whether you can go from "I understand the problem" to "here's what I would actually build." Notice: every good answer names **who uses it**, **what they see**, and **what they do differently**.

### Step 5: CONSIDER TRADE-OFFS and NEXT STEPS (1-2 minutes)

**What to do:**
Show maturity. No solution is perfect. Name what could go wrong and what you'd do next.

- "The risk here is [X]. I'd mitigate that by [Y]."
- "The first version would be [simple thing]. Once we validate that works, I'd expand to [bigger thing]."
- "The biggest unknown is [data source/stakeholder buy-in/adoption]. I'd want to test that first before investing in the full build."
- "I'd also want to talk to [actual users] before finalizing — my hypothesis might be wrong, and the real pain might be something I haven't considered."

**Why this matters:** This separates good candidates from great ones. Anyone can propose a solution. Mature thinkers immediately ask "what could go wrong?" and "what would I test first?"

---

## 32.3 The Technical Dive — When They Say "Walk Me Through the Data"

At some point the interviewer will push you deeper: "Okay, I like the solution. Now tell me about the data." This is NOT system design. You're not drawing database schemas. You're thinking about **what data you need, where it comes from, what shape it's in, and what could go wrong.**

### The Four Questions You're Answering

**1. "What data sources exist?"**

Think about where the information *actually lives* in the real world. For every case, ask yourself:

- **Internal systems** — What does this organization already track? (Sales in a CRM, patients in an EHR, inventory in a warehouse system, etc.)
- **External feeds** — What comes from outside? (Market data, weather, competitor pricing, government databases, public records)
- **Generated data** — What data doesn't exist yet but we'd need to start collecting? (User behavior logs, survey responses, sensor readings, exit interviews)
- **Manual/tribal knowledge** — What's currently trapped in someone's head or in spreadsheets that should be structured? (The experienced analyst who "just knows" which accounts are risky)

**Example (Hospital wait times case):**
> "The core data sources would be the EHR system for patient records and visit times, the scheduling system for appointments and staff allocation, and probably badge-in data or shift logs for actual staff presence. We'd likely also want patient satisfaction surveys as a lagging indicator. One thing I'd check early: does the EHR actually timestamp each stage of the patient journey, or just check-in and checkout? If it only captures the endpoints, we'd need to add tracking at triage, diagnosis, and treatment start."

**2. "What shape is the data?"**

You're not designing tables — you're thinking about whether the data is structured, semi-structured, or messy. Big-picture considerations:

- **Is it tabular?** Clean rows and columns — customer name, purchase date, amount. Easy to work with.
- **Is it documents?** Free-text notes, PDFs, emails, case files. Needs some processing to extract structured fields from it.
- **Is it time-series?** Events with timestamps — sensor data, transaction logs, clickstreams. You care about patterns over time.
- **Is it relational?** Multiple entities that connect to each other — a patient has visits, each visit has tests, each test has results. The relationships matter as much as the data itself.
- **How clean is it?** Is there missing data? Duplicates? Conflicting records across systems? This is always the honest answer: "In reality, probably pretty messy."

**Example (Churn case):**
> "The customer data is relational — each customer has a subscription, each subscription has usage events, each event has a timestamp and action type. The tricky part is the product usage data is probably high-volume time-series — thousands of events per customer per month. We wouldn't analyze that raw; we'd aggregate it into features like 'sessions per week' or 'days since last login' that we can actually reason about."

**3. "What are the key data considerations?"**

This is where you show you've thought about the real-world messiness:

- **Freshness** — How current does this data need to be? Real-time? Daily batch? "For the shift supervisor dashboard, it needs to be real-time. For the monthly executive report, daily is fine."
- **Completeness** — What's missing? "They probably don't have exit survey data on every churned customer. We'd be working with a sample."
- **Access and permissions** — Who can see what? "Patient data has privacy constraints. The dashboard for administrators should show aggregate patterns, not individual patient records."
- **Integration** — Are these data sources connected today, or siloed? "The sales team uses Salesforce, the support team uses Zendesk. Those probably don't talk to each other. Connecting them is the first step."
- **Quality** — What about duplicates, inconsistencies, stale records? "If the address data hasn't been updated in 3 years, our geographic analysis is going to be wrong."

**4. "If you were to sketch the logic, what would it look like?"**

This is where very simple pseudocode is appropriate. Not real code — just the logic in plain English:

```
For each customer:
    Look at their usage over the last 30 days
    Compare it to their usage in the 30 days before that
    If usage dropped more than 50% → flag as "at-risk"
    Pull their top 3 most-used features → these are what they valued
    Check which of those features they stopped using → that's the likely pain point
    Route to retention team with: customer name, risk score, likely pain point
```

Or for the hospital case:

```
Every 15 minutes:
    For each facility:
        Count patients at each stage (waiting, triage, treatment, discharge)
        Compare to normal capacity for this hour and day of week
        If any stage exceeds 120% of normal → flag as bottleneck
        Show on dashboard with color coding: green (fine), yellow (building), red (critical)
        Suggest: "Consider redirecting scheduled patients to Facility B (currently at 60%)"
```

This level of detail is enough. Don't go deeper unless they ask.

### How to Handle "Go Deeper" Follow-Ups

| They say... | You say... |
|---|---|
| "What database would you use?" | "For something like this, I'd want a structured store for the transactional data — something relational. The key question is what needs to be queried together. I'd want to be able to join customer profiles with their usage events and support tickets in one query." |
| "How would you handle scale?" | "Start simple — everything in one place. If we're talking thousands of customers, any modern database handles that fine. If we're talking millions of events per day, I'd separate the raw event stream from the analytical layer." |
| "What about real-time?" | "It depends on who's using it. The floor supervisor needs real-time. The VP needs a daily summary. I'd build the real-time feed for the operator and aggregate it for the executive view." |
| "How would you connect these data sources?" | "In Palantir terms, this is basically the ontology problem — you've got data in different systems with different schemas, and you need to define the objects and relationships to link them. Practically: common keys like customer ID, deduplication, and a transformation layer that normalizes the different formats into one usable structure." |

## 32.4 The Five Question Archetypes — Know These Cold

Almost every FDE case falls into one of five archetypes. If you know the archetype, you know where to start digging.

### Archetype 1: "This Business Is Losing Money"
**Prompt:** "A mid-size retailer has seen profits decline 25% over 2 years. The CEO has asked us to figure out why and propose a turnaround plan."

**Where to start:** Revenue vs. Costs. Is revenue falling? Are costs rising? Both? Then drill into *why* — is it a customer problem (churn), a pricing problem, a market problem, or an operational efficiency problem?

**Data you'd want:**
- Revenue by product line / region / customer segment (trending over time)
- Customer acquisition cost vs. lifetime value
- Customer churn rate and any exit survey data
- Largest cost line items and how they've changed

**Palantir angle:** "I'd want to connect their sales data with their customer data in one place — right now it's probably in separate systems. A unified view would let us see which customer segments are profitable and which are actually costing money."

---

### Archetype 2: "Make This Operation More Efficient"
**Prompt:** "A hospital system processes 10,000 patients per month across 5 facilities. Wait times have increased 40%. How would you fix it?"

**Where to start:** Bottleneck analysis. Where in the patient journey is the delay? Is it intake, diagnosis, treatment, or discharge? Is it a capacity problem (not enough resources) or a scheduling problem (resources exist but aren't allocated well)?

**Data you'd want:**
- Average time at each stage of the patient journey
- Capacity utilization by department, day of week, hour
- Which patient types (emergency vs. scheduled) are impacted most
- Staff-to-patient ratios across facilities

**Palantir angle:** "I'd build an operational dashboard that shows real-time patient flow — where is every patient in their journey right now, and where are the bottlenecks forming. The shift supervisor could see this and reallocate staff before the backup gets bad, not after."

---

### Archetype 3: "Enter This New Market"
**Prompt:** "A SaaS company that sells to mid-market companies wants to move upmarket and sell to enterprises. Should they? How?"

**Where to start:** Market attractiveness and readiness. Is the enterprise market worth it? Can their current product handle enterprise needs? What would they need to change — in the product, the sales process, the support model?

**Data you'd want:**
- Enterprise market size and growth rate in their sector
- Current product gaps vs. enterprise requirements (security, SSO, audit logs, SLAs)
- Current sales cycle length vs. expected enterprise sales cycle
- Competitor positioning in the enterprise segment

**Palantir angle:** "The biggest risk isn't the product — it's the sales cycle. Enterprise deals take 6-18 months. I'd model their cash runway against different conversion scenarios before committing."

---

### Archetype 4: "Our Customers Are Leaving"
**Prompt:** "A subscription service has seen churn increase from 5% to 12% monthly over the last year. Why, and what should they do?"

**Where to start:** Segmentation. Not all churn is the same. Who specifically is leaving? When in their lifecycle do they leave? Did something change a year ago — new pricing, product change, competitor entry?

**Data you'd want:**
- Churn rate by customer segment (age, plan tier, geography, acquisition channel)
- Time-to-churn — do they leave in month 1, month 6, or month 12?
- Product usage patterns of churned vs. retained customers
- Any pricing or product changes that correlate with the churn increase

**Palantir angle:** "I'd build a churn prediction model — flag customers who look like they're about to leave based on engagement drop-off. Then surface those to the retention team with the top 3 reasons they're at risk, so the outreach is specific, not generic."

---

### Archetype 5: "Build Something for This Agency / Organization"
**Prompt:** "The VA wants to reduce veteran homelessness. You're deployed as a Palantir FDE. What do you do in your first 90 days?"

**Where to start:** Stakeholder mapping and data landscape. Who are the actors? (Caseworkers, housing providers, veterans, benefits administrators.) What data exists? (Intake records, benefits claims, housing availability, employment status.) What decisions are being made, and what information is missing?

**Data you'd want:**
- Veteran intake data — demographics, service history, current situation
- Housing availability and turnover rates by region
- Benefits eligibility and claims status
- Caseworker caseload and time allocation

**Palantir angle:** "The first thing I'd build is a caseworker tool — a single screen where they can see their full caseload, each veteran's status, what benefits they're eligible for that they haven't claimed, and what housing just became available in their area. Right now this probably requires opening 4 different systems."

---

## 32.5 Magic Phrases — Things That Make Interviewers Nod

Keep these in your pocket. Use them naturally when they fit.

| Phrase | When to use it |
|---|---|
| "Who's the person sitting in this seat every day?" | When you want to ground the discussion in a real user |
| "What decision does this enable that they can't make today?" | When proposing a solution — forces you to be specific |
| "What data exists vs. what data would we need to create?" | Shows you understand real-world data is messy and incomplete |
| "Day One vs. Day Ninety" | Shows you think in phases — MVP first, then iterate |
| "The highest-leverage thing we could do first is..." | Shows prioritization — you're not trying to boil the ocean |
| "I'd want to hear from the actual users before finalizing this" | Shows humility and real-world deployment awareness |
| "That's where I'd start — but I want to flag a risk..." | Shows mature, trade-off thinking |
| "In Palantir terms, this would look like an ontology with..." | Only use after you've established understanding — connects to Foundry naturally |

---

## 32.6 The Anti-Patterns — What Kills Candidates

### ❌ Going Technical Too Fast
If you start talking about databases, APIs, or data pipelines before you've established the business problem, you've lost. The interviewer asked you to save a business, not architect a system.

**Rule:** You should be able to explain your first 5 minutes of response to a smart person who has never written a line of code.

### ❌ Jumping to Solutions
"They should use machine learning" — before you've even asked who the customer is. First diagnose, then prescribe.

### ❌ Boiling the Ocean
Proposing 10 things instead of 1 great thing. The interviewer wants depth on one path, not a laundry list.

### ❌ Forgetting the Human
Every solution needs a user. "Build a predictive model" is incomplete. "Build a dashboard that shows caseworkers their highest-priority cases each morning, ranked by likelihood of falling through the cracks" — that's complete.

### ❌ Being Afraid to Say "I Don't Know"
"I'm not sure what the standard margin is for this industry — but I'd want to benchmark that. Let's assume it's around 15% and test the sensitivity." That's strength, not weakness.

---

## 32.7 How to Practice — Your Training Plan

### Solo Practice (20 minutes per case)

1. Pick a case archetype from Section 32.3
2. Set a timer for 15 minutes
3. Talk through the 5-Step Framework **out loud** — as if the interviewer is sitting across from you
4. Record yourself on your phone and play it back
5. Ask: Did I stay structured? Did I get too technical? Did I propose something concrete that a real person would use?

### The 3-Case Minimum

Before your interview, do at least 3 full practice cases:
- One **"business is losing money"** (Archetype 1)
- One **"make this operation efficient"** (Archetype 2)
- One **"build something for this organization"** (Archetype 5)

These cover the highest-probability question types for an FDE interview.

### The 30-Second Test

After finishing a practice case, explain your entire answer in 30 seconds. If you can't, you got lost in the details. The core logic should be: "The problem is X, I'd build Y, it helps person Z make decision W."

---

## 32.8 Connecting to Your Experience

When the case gives you an opening, connect to what you've already done:

| If the case involves... | Connect to... |
|---|---|
| Messy or siloed data | Your Pentagon experience — 3 weeks finding data that should have taken 1 day |
| Skeptical stakeholders | The Colonel story — building from the bottom up, earning the invite |
| Building a tool for resistant users | Telemus — "What's the worst part of your job?" approach |
| Data-driven decision making | The wargame engine — 100 simulations to build statistical distributions instead of one gut-feel run |
| AI / ML applications | Siren — intent-based classification instead of keyword matching |
| Working with ambiguity | Starting your own company after the DARPA deal fell through |

Don't force these. But when the conversation naturally touches one of these areas, a 20-second "I actually experienced something similar when..." makes the case answer personal and memorable.

---

## 32.9 Quick Reference Card — Tape This to Your Mirror

```
THE 5 STEPS:
1. LISTEN → Restate → Clarify
2. MAP → 3-5 buckets → Pick the hottest one
3. DIG → Hypothesize → Ask for data → Confirm/deny
4. PROPOSE → Who uses it? What do they see? What do they do differently?
5. TRADE-OFFS → What could go wrong? → What's the Day 1 MVP?

THE 5 ARCHETYPES:
1. Losing money → Revenue vs. Costs → Drill into why
2. Inefficient operations → Find the bottleneck → Build visibility
3. New market → Attractiveness + Readiness → Model the risk
4. Customers leaving → Segment the churn → Predict and intervene
5. Build for an org → Stakeholders + Data landscape → Day 1 tool

FORBIDDEN MOVES:
- Don't go technical before establishing the business problem
- Don't propose 10 things — go deep on 1
- Don't forget the human — name the user, the screen, the decision
- Don't be afraid to say "I'd want to test that assumption"
```
