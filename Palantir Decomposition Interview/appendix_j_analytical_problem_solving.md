# Appendix J: Analytical Problem Solving — The Third Interview Round

> *"They're not testing whether you can do math. They're testing whether you can structure ambiguity into something quantifiable — and then make a decision with imperfect numbers."*

---

## Why This Appendix Exists

The recruiter email lists three interview rounds: **Technical Comprehension** (Chapter 19), **Open-Ended Decomposition** (Chapters 21–22), and **Analytical Problem Solving** — which this appendix covers.

This round tests something distinct from the other two. It's not about designing a system (Decomposition) or learning a new concept (Technical Comprehension). It's about **quantitative reasoning with messy, real-world constraints**. Think: "back-of-the-envelope math meets consulting logic meets data intuition."

---

## J.1 What the Round Actually Looks Like

### The Format

```
Minutes 0-5:    Introductions, light context-setting
Minutes 5-15:   Interviewer gives you a scenario with a quantitative question
Minutes 15-35:  You structure the problem, estimate, and reason through it
Minutes 35-45:  Follow-up twists, "what would you do next?", wrap-up
```

### What They're Testing

| Signal | What It Looks Like |
|--------|-------------------|
| **Structured thinking** | You break a fuzzy question into concrete, estimable components |
| **Comfort with numbers** | You work with rough math without freezing or over-engineering |
| **Assumptions awareness** | You state your assumptions explicitly and know which ones matter |
| **Sensitivity analysis** | You can say "if I'm wrong about X, the answer changes dramatically" |
| **Decision-making** | You use your numbers to actually recommend something |

### What They're NOT Testing

- Exact mathematical precision (±50% is fine for Fermi questions)
- Whether you know the "right answer" (there often isn't one)
- Advanced statistics or probability theory
- Your ability to do arithmetic quickly in your head

---

## J.2 Fermi Estimation — "How Many X Are There in Y?"

### What It Is

Fermi estimation (named after physicist Enrico Fermi) is the practice of making reasonable estimates for quantities that seem impossible to calculate. The key insight: **you don't need exact numbers. You need a reasonable order of magnitude.**

### The Framework: Break → Estimate → Multiply → Sanity-Check

Every Fermi question follows the same structure:

1. **Break** the question into smaller, estimable pieces
2. **Estimate** each piece using common sense and anchoring
3. **Multiply** the pieces together
4. **Sanity-check** by approaching from a different angle

### Worked Example: "How many piano tuners are there in Chicago?"

**Step 1 — Break it down:**
- How many pianos are in Chicago?
- How often does a piano need tuning?
- How many tunings can one tuner do per year?
- → Number of tuners = Total tunings needed ÷ Tunings per tuner

**Step 2 — Estimate each piece:**
- Chicago population: ~2.7 million people → ~1 million households
- % of households with a piano: ~5% → 50,000 pianos
- Add commercial pianos (schools, churches, concert halls): ~10,000
- Total pianos: ~60,000
- Each piano tuned ~1.5 times per year → 90,000 tunings needed
- A tuner does ~4 tunings/day × 250 working days = 1,000 tunings/year

**Step 3 — Divide:**
- 90,000 ÷ 1,000 = **~90 piano tuners**

**Step 4 — Sanity check:**
- Is that reasonable for a city of 2.7M? About 1 per 30,000 people. Sounds plausible for a niche trade. ✓

### The Critical Interview Move: State Your Assumptions

After each estimate, say out loud:

> *"I'm estimating 5% of households have a piano. That could be high — it might be 2-3% in a city like Chicago. But even if I'm off by 2x, our answer would be ~45-90 tuners, which is the same order of magnitude."*

This shows **sensitivity awareness** — the most important Fermi skill. You're not just guessing; you're telling the interviewer which assumptions drive the answer.

### Common Fermi Prompts

| Question | How to Break It Down |
|----------|---------------------|
| "How many gas stations in the US?" | Cars → fill-ups per week → capacity per station |
| "How much revenue does a McDonald's make?" | Meals per hour × hours × price × days |
| "How many flights are in the air right now?" | US airports × departures/hour × flight duration |
| "How many Uber rides happen in NYC daily?" | Population × % who ride × rides/rider/day |
| "How many data points does a hospital generate daily?" | Patients × systems × events per patient |

---

## J.3 Metrics and KPIs — "How Do You Know It's Working?"

### Why This Matters

After every system design or decomposition, someone will ask: "How do you measure whether this is successful?" If you can't answer, your entire solution sounds academic. **Metrics are how you prove the thing you built actually matters.**

### The Metrics Stack

For any Palantir deployment, think about metrics at three levels:

```
LEVEL 3: OUTCOME METRICS  (Did the business improve?)
    ↑ driven by
LEVEL 2: USAGE METRICS     (Are people using the tool?)
    ↑ driven by
LEVEL 1: DATA METRICS      (Is the system working correctly?)
```

**Example — Grocery waste reduction system:**

| Level | Metric | Example |
|-------|--------|---------|
| Data (L1) | Pipeline freshness | "Data updated every 15 minutes" |
| Data (L1) | Coverage | "95% of stores reporting" |
| Usage (L2) | DAU (Daily Active Users) | "85% of store managers log in daily" |
| Usage (L2) | Action completion rate | "60% of waste alerts result in markdowns" |
| Outcome (L3) | Waste reduction | "22% less spoilage vs. last quarter" |
| Outcome (L3) | Revenue impact | "$88M in recovered revenue annually" |

### The North Star Metric

Every deployment should have **one metric** that captures the ultimate goal. If the interviewer asks "how would you measure success?" — give them the North Star first, then the supporting metrics.

> *"The North Star Metric is waste reduction — pounds of perishable goods discarded per store per week. If that number drops consistently, the system is working. Supporting that, I'd track three things: pipeline freshness to make sure the data is current, manager action rate to make sure humans are actually responding to alerts, and markdown-to-spoilage ratio to see if we're acting early enough."*

### Leading vs. Lagging Indicators

| Type | What It Tells You | When You See It | Example |
|------|------------------|----------------|---------|
| **Leading** | Will we succeed? | Early | "App logins are up 3x this week" |
| **Lagging** | Did we succeed? | After the fact | "Food waste dropped 22% this quarter" |

**Interview move:** Always mention both. Leading indicators are how you course-correct during a deployment; lagging indicators are how you report results to the client's leadership.

### The Dangerous Vanity Metrics

These sound impressive but don't prove impact:

| Vanity Metric | Why It's Misleading | Better Alternative |
|---------------|--------------------|--------------------|
| "10,000 data points ingested" | Volume ≠ value | "95% of data passes quality checks" |
| "200 users have accounts" | Accounts ≠ usage | "85% DAU rate among registered users" |
| "Dashboard loads in 2 seconds" | Speed ≠ impact | "Decision time reduced from 4 hours to 20 mins" |
| "We integrated 12 data sources" | Integration ≠ insight | "Cross-referencing 3 sources revealed 2,000 duplicate records" |

---

## J.4 "Interpret This Data" — Analytical Reasoning Questions

### The Format

The interviewer presents a dataset, chart, or table and asks: "What do you see? What would you recommend?" This tests whether you can extract signal from noise and make decisions under uncertainty.

### The 4-Step Analysis Framework

**Step 1: Describe What You See (don't jump to conclusions)**

> *"The chart shows monthly revenue over 24 months. There's a clear upward trend from months 1-18, then a sharp decline in months 19-24. The decline starts around October 2024."*

**Step 2: Generate Hypotheses (at least 3)**

> *"Three possible explanations: (1) Seasonal pattern — if this is a retail business, post-holiday decline would be normal. (2) External shock — something changed in the market or regulatory environment around October. (3) Internal issue — maybe a product change, pricing change, or key customer churn."*

**Step 3: Identify What Data You'd Need to Distinguish**

> *"To distinguish between these, I'd want: (a) year-over-year comparison to check for seasonality, (b) customer-level data to see if the decline is broad or concentrated in a few accounts, and (c) a timeline of product or pricing changes."*

**Step 4: Make a Provisional Recommendation**

> *"Based on what I can see, my initial hypothesis is [X], and I'd recommend [action] while we confirm with additional data. If my hypothesis is wrong, I'd pivot to [alternative]."*

### Common Pitfalls

| Mistake | What It Looks Like | Better Approach |
|---------|--------------------|-----------------| 
| **Jumping to conclusions** | "Revenue dropped because of the recession" | "Revenue dropped — let me think about why. Three hypotheses..." |
| **Only one hypothesis** | "It must be seasonality" | "It could be seasonality, or customer churn, or a pricing issue" |
| **No recommendation** | "I'd need more data" | "My best guess is X, and here's what I'd check to confirm" |
| **Ignoring context** | Looking only at the chart | "What industry is this? What happened in October externally?" |

---

## J.5 "How Would You Measure the Success Of X?" Questions

These are the most common analytical questions in the DS interview. Here's a bank of worked examples:

### Example 1: "How would you measure whether a predictive maintenance system is working?"

> *"North Star: Unplanned downtime hours per facility per month — that's what the client ultimately cares about.*
>
> *Supporting metrics:*
> *1. Prediction accuracy: What % of failures did we predict at least 24 hours in advance?*
> *2. False positive rate: How often do we flag a machine that's actually fine? Too many false alarms and operators stop trusting the system.*
> *3. Action rate: When we flag a machine, does someone actually schedule preventive maintenance? If not, the model is useless.*
> *4. Time-to-repair: If we predict a failure and act on it, is the repair faster/cheaper than an unplanned breakdown?"*

### Example 2: "How would you measure whether a fraud detection system is working?"

> *"This one's tricky because you have to balance catching fraud vs. annoying legitimate customers.*
>
> *Primary metrics:*
> *1. Detection rate (recall): What % of actual fraud do we catch?*
> *2. False positive rate (precision): What % of flagged transactions are legitimate?*
> *3. Dollar impact: How much fraud did we prevent in dollars? That's the ROI story.*
>
> *Secondary metrics:*
> *4. Customer friction: How many legitimate customers are blocked or delayed?*
> *5. Time to detection: How quickly do we flag fraud after it occurs?*
>
> *The key tension is recall vs. precision. Catch more fraud = more false positives = more angry customers. I'd work with the client to define an acceptable false positive rate before deploying."*

### Example 3: "How would you measure whether a hospital bed allocation system is working?"

> *"North Star: Average patient wait time for an available bed.*
>
> *Supporting metrics:*
> *1. Bed utilization rate: Are beds sitting empty or at capacity? Target: 85-90% (100% means no surge capacity).*
> *2. Transfer delay: Time between 'patient ready to move' and 'patient in new bed.'*
> *3. Prediction accuracy: When we forecast bed availability for tomorrow, how close were we?*
> *4. Staff satisfaction: Do nurses and administrators trust the system? If they override it constantly, it's not working."*

---

## J.6 Practice Problems — Try These Before Your Interview

### Problem 1: Fermi Estimation

*"A major airline wants to estimate how many meals they serve per year across all flights. Walk me through your estimation."*

**Hints to structure your thinking:** Routes → flights per day → meals per flight → domestic vs. international.

### Problem 2: Metrics Design

*"A city government deployed a pothole detection app — citizens photograph potholes, the system uses computer vision to classify severity, and repair crews are dispatched. The mayor asks: 'Is this working?' What do you tell her?"*

**Think about:** Response time, repair rate, citizen satisfaction, cost efficiency vs. old system.

### Problem 3: Data Interpretation

*"You're looking at employee attrition data for a tech company. Attrition was 12% for the past 3 years, then jumped to 22% last quarter. What do you investigate?"*

**Think about:** Department-level breakdown, tenure distribution, exit interview themes, external market (competitors hiring?), internal events (reorg, layoffs, policy changes).

### Problem 4: Trade-Off Analysis

*"A hospital has $500K to spend on data infrastructure. Option A: a real-time patient monitoring dashboard that alerts nurses. Option B: a predictive model that forecasts patient deterioration 6 hours in advance but requires 6 months to build. What do you recommend?"*

**Think about:** Time-to-value, clinical impact, staff adoption, lives at stake.

---

## J.7 Quick Reference Card

```
FERMI ESTIMATION:
1. Break → smaller pieces
2. Estimate → each piece
3. Multiply → get your answer
4. Sanity-check → approach differently
• ALWAYS state assumptions
• ALWAYS note which assumptions drive the answer

METRICS DESIGN:
• Start with the North Star (one metric that captures the goal)
• Layer: Data metrics → Usage metrics → Outcome metrics
• Mention leading AND lagging indicators
• Call out vanity metrics and why you're avoiding them

DATA INTERPRETATION:
1. Describe what you see (no conclusions yet)
2. Generate 3+ hypotheses
3. Say what data you'd need to distinguish them
4. Make a provisional recommendation

UNIVERSAL MOVES:
• "The key tension here is..."
• "My estimate could be off by 2x, but even then..."
• "The metric I'd watch most closely is..."
• "If my assumption about X is wrong, the answer changes to..."
```

---

*Return to: Chapter 18 — The Interview Process for the full interview structure.*
