# Appendix B: Consulting Case Study Fundamentals — The Tradition You're Inheriting


> ⚠️ **WARNING**: The Palantir recruiter explicitly states: "These are NOT consulting case studies." This appendix is background knowledge only — do NOT apply consulting frameworks in the interview. Use the decomposition approach from Chapters 20–22 instead.
> *"A case interview is not a trick question with a hidden answer. It is a window into how you think when the problem is bigger than you."*

---

## Why This Chapter Matters

When Palantir designed its Decomposition and Analytical rounds, it didn't invent the format from scratch. It borrowed — heavily — from the management consulting interview tradition pioneered by McKinsey, Bain, and BCG (the "MBB" firms) in the 1980s and refined over four decades. If you understand that tradition, you will understand *why* the Palantir interview works the way it does, what the interviewers are actually testing, and how to apply frameworks that have been battle-tested on millions of candidates.

This chapter covers the consulting case study from the ground up: its origins, its core logic, the universal frameworks, and the mental muscles it trains. Chapter 26 will teach you how to *execute* a case under interview conditions. Chapter 27 will map the consulting method onto the Palantir DS interview specifically, showing where it overlaps and where it diverges.

If you've never done a consulting case before, start here. If you have, skim the frameworks and focus on Section 25.5 — it reframes the entire methodology in terms you'll need for Palantir.

---

## 25.1 A Brief History — Where Case Studies Come From

### The Origin: McKinsey in the 1950s

Management consulting firms needed a way to evaluate candidates that went beyond GPA and credentials. The problem: consulting work is fundamentally *unstructurable* in advance. A consultant walks into a client's office on Monday morning, faces a problem they've never seen, in an industry they may not know, with incomplete data and a skeptical audience. No exam can simulate this.

In the 1950s, McKinsey & Company began presenting candidates with miniature versions of real client problems during interviews. The instruction was simple: *"Here's a problem. Think through it out loud."* There was no answer key. What mattered was how the candidate *attacked* the problem — the questions they asked, the structure they imposed, the logic they followed.

By the 1980s, Bain and BCG had adopted the format. By the 2000s, case interviews were standard across all strategy consulting firms and had spread to tech companies, private equity firms, and — eventually — data-focused companies like Palantir.

### Why Cases Survived for 70 Years

The case format endured because it tests something that resumes, grades, and behavioral questions cannot: **structured improvisation**. The ability to take an ambiguous, messy, real-world problem and — in real time, under pressure — organize it into tractable parts, identify what matters, and propose a defensible path forward.

This is exactly what a Deployment Strategist does. Every day.

---

## 25.2 The Anatomy of a Case — What You're Actually Being Given

Every case, regardless of firm or format, has the same underlying anatomy:

```
┌─────────────────────────────────────────────────────────────┐
│                    CASE ANATOMY                              │
│                                                              │
│  1. THE SITUATION                                            │
│     Context: industry, company, recent events                │
│     "Our client is a mid-size US airline..."                 │
│                                                              │
│  2. THE PROBLEM                                              │
│     The question they need answered                          │
│     "...and profits have declined 20% over 2 years."        │
│                                                              │
│  3. THE ASK                                                  │
│     What you're supposed to do                               │
│     "They've asked us to figure out why and what to do."     │
│                                                              │
│  4. THE HIDDEN STRUCTURE                                     │
│     Constraints, data, and curveballs the interviewer        │
│     will reveal as you ask the right questions               │
│     (You don't see this part up front)                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### The Four Case Archetypes

While every case is unique, almost all fall into one of four families:

| Archetype | Core Question | Example |
|---|---|---|
| **Profitability** | Why are profits declining (or, how do we increase them)? | "A hospital chain's margins dropped from 12% to 4% in two years. Diagnose and recommend." |
| **Market Entry** | Should the company enter a new market / launch a new product? | "A European luxury automaker is considering entering the Indian market. Should they?" |
| **Market Sizing** | How big is something? | "How many piano tuners are there in Chicago?" |
| **Mergers & Acquisitions** | Should the company acquire or merge with another? | "A national pharmacy chain is evaluating acquiring a telehealth startup. Advise." |

> **For Palantir purposes:** The Profitability and Market Sizing archetypes are less common (they're very McKinsey). Palantir cases tend to blend elements of all four into a *systems design* problem — which we'll cover in Chapter 27. But understanding the archetypes trains the muscle.

---

## 25.3 The Core Frameworks

Frameworks are pre-built thinking structures that help you organize a problem quickly. They are not formulas that generate answers — they are scaffolds that prevent you from missing critical dimensions.

**Critical warning:** Memorizing frameworks and force-fitting them onto every problem is the fastest way to fail a case. Good consultants *adapt* frameworks to the specific problem. Great ones build bespoke structures on the spot. Think of these as starting templates, not finished products.

---

### Framework 1: The Profit Tree

The most fundamental framework in consulting. Used for any profitability question.

```
                        PROFIT
                          │
              ┌───────────┴───────────┐
              │                       │
           REVENUE                  COSTS
              │                       │
       ┌──────┴──────┐        ┌──────┴──────┐
       │             │        │             │
     PRICE ×     VOLUME    FIXED         VARIABLE
                    │       COSTS          COSTS
             ┌──────┴──────┐   │              │
             │             │   │              │
          # CUSTOMERS  UNITS  Rent          COGS
                      PER     Salaries     Materials
                     CUSTOMER Overhead     Labor/unit
                              Equipment   Shipping
```

**How to use it:**

When told "profits are declining," your first move is to ask: *"Is this a revenue problem, a cost problem, or both?"* Then drill down the appropriate branch.

**Example dialogue:**

```
INTERVIEWER: "A mid-size hotel chain has seen profits drop 30%
over the past 18 months. The CEO hired us to figure out why."

YOU: "Before I dig into the analysis, I want to understand the
shape of the problem. Is this primarily a revenue decline, a
cost increase, or both moving in the wrong direction?"

INTERVIEWER: "Revenue has actually been flat. The issue is on
the cost side."

YOU: "Got it — so we need to understand where costs have grown.
Let me split costs into fixed and variable. On the fixed side:
has the chain expanded — new properties, new leases? On the
variable side: have operating costs per room-night increased —
labor, supplies, energy?"

INTERVIEWER: "Good question. They haven't added properties.
But labor costs per room have increased 25%."

YOU: "So the hypothesis is that per-unit labor cost is the
primary driver. I'd want to decompose that further: is it
wage increases, staffing levels, or overtime? And is this
across all properties equally, or concentrated in certain
markets?"
```

**Why it works:** You didn't guess. You didn't need domain expertise about hotels. You used a structure to systematically narrow a vague problem ("profits are down") into a specific, testable hypothesis ("per-room labor costs, possibly driven by overtime at specific locations").

---

### Framework 2: Market Sizing (Fermi Estimation)

Named after physicist Enrico Fermi, who was famous for estimating quantities with minimal data. The classic: "How many piano tuners are in Chicago?"

**The method:**

1. **Start from a known anchor** (population, households, etc.)
2. **Build a logical chain of assumptions**, each one reasonable
3. **Multiply through** to get an estimate
4. **Sanity check** the result

**Example — "How many gas stations are in the United States?"**

```
STEP 1: US population               ≈ 330 million people
STEP 2: Average people per car       ≈ 1.5 (220M cars)
STEP 3: Average miles driven/year    ≈ 12,000 miles
STEP 4: Average fuel efficiency      ≈ 25 MPG
STEP 5: Gallons per car per year     = 12,000 / 25 = 480 gallons
STEP 6: Fill-ups per year per car    = 480 / 12 gal per fill = 40
STEP 7: Total annual fill-ups        = 220M × 40 = 8.8 billion
STEP 8: Fill-ups per station per day = ~100 (busy suburban avg)
STEP 9: Fill-ups per station per year= ~36,500
STEP 10: Stations needed             = 8.8B / 36,500 ≈ 240,000

SANITY CHECK: Actual answer is ~150,000.
Our estimate is within 2x — acceptable for a Fermi estimate.
```

**Why consulting firms love this:**
- It tests your ability to *structure* an estimation without panicking
- It reveals your numeracy and comfort with math
- It shows whether you can identify reasonable assumptions vs. wild guesses
- It demonstrates your ability to *communicate* the chain of logic

**For Palantir:** Market sizing is less common in DS interviews, but the *underlying skill* — decomposing an unknown quantity into tractable sub-estimates — appears constantly in data questions like "How would you estimate the total number of fraudulent Medicare claims?" or "How many field technician hours are wasted on unnecessary dispatches?"

---

### Framework 3: The 3C's + P (Market Entry)

For "should we enter this market?" questions:

```
┌─────────────────────────────────────────────────────┐
│                  MARKET ENTRY                        │
│                                                      │
│  COMPANY          CUSTOMERS         COMPETITION      │
│  ─────────        ──────────        ────────────     │
│  • Capabilities   • Market size     • Who's there?   │
│  • Resources      • Growth rate     • Market share    │
│  • Brand fit      • Segments        • Barriers to    │
│  • Strategic      • Willingness       entry           │
│    alignment        to pay          • Competitive     │
│  • Risk           • Unmet needs       advantages     │
│    tolerance                                          │
│                                                      │
│  PRODUCT                                             │
│  ────────                                            │
│  • What offering?  • Pricing?  • Differentiation?    │
│  • Distribution?   • Timeline to market?             │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**How to use it:**

Don't mechanically walk through every box. Instead, identify which dimension is the *binding constraint* and focus there.

```
INTERVIEWER: "A premium coffee chain based in Scandinavia wants
to enter the US market. Should they?"

YOU: "I'd structure this around three questions. First,
Company: do they have the capital, supply chain, and brand
positioning to compete in the US? Second, Customers: is there
a segment of the US coffee market that's underserved by
existing players? Third, Competition: what's the competitive
landscape — they'd be going up against Starbucks, Blue Bottle,
and hundreds of independents. Let me start with Competition,
since that feels like the highest-risk dimension."
```

---

### Framework 4: M&A Evaluation

For "should Company A acquire Company B?" questions:

```
┌──────────────────────────────────────────────────────────┐
│                M&A EVALUATION                             │
│                                                           │
│  1. STRATEGIC RATIONALE                                    │
│     Why does this acquisition make sense?                  │
│     • Revenue synergies (cross-sell, new market access)    │
│     • Cost synergies (shared infrastructure, headcount)    │
│     • Capability acquisition (tech, talent, IP)            │
│     • Defensive move (block competitor)                    │
│                                                           │
│  2. TARGET ASSESSMENT                                      │
│     Is the target worth buying?                            │
│     • Financial health (revenue, margins, growth)          │
│     • Customer base (overlap? complementary?)              │
│     • Technology / IP (unique? Replicable?)                │
│     • Culture fit (can you actually integrate?)            │
│                                                           │
│  3. VALUATION                                              │
│     What's the right price?                                │
│     • Revenue multiples vs. comparable companies           │
│     • Discounted cash flow (DCF) simplified                │
│     • Synergy value (what's the combined entity worth?)    │
│                                                           │
│  4. INTEGRATION RISKS                                      │
│     What could go wrong?                                   │
│     • Technology integration complexity                    │
│     • Key person risk (will founders/leaders stay?)        │
│     • Customer churn during transition                     │
│     • Regulatory/antitrust risk                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

### Framework 5: MECE — The Meta-Framework

**MECE (Mutually Exclusive, Collectively Exhaustive)** is not a framework itself — it's the quality standard that all frameworks must meet.

```
MUTUALLY EXCLUSIVE: Categories don't overlap.
COLLECTIVELY EXHAUSTIVE: Categories cover everything.

✅ MECE: Revenue vs. Costs
   (Nothing is both revenue and cost; everything is one or the other)

❌ NOT MECE: "Marketing problems" vs. "Customer problems"
   (A marketing problem IS a customer problem → they overlap)

✅ MECE: Domestic vs. International
   (Every sale is one or the other)

❌ NOT MECE: "Product issues" vs. "Quality issues" vs. "Brand issues"
   (Quality is part of Product; Brand is influenced by both → overlaps)
```

**Why MECE matters:** If your breakdown isn't MECE, you'll either double-count something or miss something entirely. In consulting, this leads to wrong recommendations. In a Palantir Ontology design, this leads to a broken data model. MECE is the discipline that prevents both.

---

## 25.4 The Mental Muscles — What Cases Actually Train

Behind the frameworks, cases develop five mental capabilities:

### Muscle 1: Hypothesis-Driven Thinking

Instead of boiling the ocean, you form an educated guess early and then test it.

```
UNDIRECTED (BAD):
"Let me look at everything and see what patterns emerge."

HYPOTHESIS-DRIVEN (GOOD):
"My hypothesis is that the profit decline is driven by
rising variable costs in the Southeast region, based on the
fact that they've expanded aggressively there. Let me check
the data to confirm or revise."
```

**Why it matters for Palantir:** When you arrive at a client site with millions of rows of data, you can't explore everything. You need a hypothesis that guides your first analysis. If the hypothesis is wrong, you revise it. But you *start with a direction*. DSs call this "leading with a point of view."

---

### Muscle 2: Structured Decomposition

The ability to take a single big question and break it into smaller, answerable sub-questions — recursively, until each sub-question is tractable.

```
BIG QUESTION:
"Why is our customer satisfaction declining?"

DECOMPOSITION:
├── Is satisfaction declining across all segments, or specific ones?
│   ├── By product line?
│   ├── By geography?
│   └── By customer tenure (new vs. long-term)?
├── What dimension of satisfaction is declining?
│   ├── Product quality?
│   ├── Service / support?
│   ├── Pricing perception?
│   └── Delivery / logistics?
├── When did the decline start?
│   ├── Was there a triggering event?
│   └── Is it gradual or sudden?
└── How are we measuring satisfaction?
    ├── Survey methodology unchanged?
    └── Response rates stable?
```

**This is the exact skill tested in Palantir's Decomposition round.** The difference: in consulting, you decompose into *analyses*. In a Palantir context, you decompose into *data entities, pipelines, and operational workflows*.

---

### Muscle 3: 80/20 Prioritization

The Pareto Principle applied to problem-solving: 80% of the value comes from 20% of the analysis. Good caseists know what to skip.

```
COMPLETE BUT IMPRACTICAL:
"I'd want to analyze all 47 product lines across 12 regions
over 5 years with 8 demographic segments."

80/20 FOCUSED:
"The interviewer mentioned the decline started 18 months ago
and is concentrated in the Southeast. I'll start there — if
3 of 47 products drive most of the revenue, those 3 products
in the Southeast in the last 18 months is my focus."
```

**For Palantir:** This translates directly to deployment planning. You don't build the entire Ontology on day one. You identify the highest-impact Object Types, build those first, and iterate.

---

### Muscle 4: Quantitative Comfort

Cases require you to do mental math, interpret charts, and make numerical arguments — not with a calculator, but with logic and estimation.

Key skills:
- **Rounding appropriately:** 327 × 41 ≈ 330 × 40 = 13,200. Close enough.
- **Percentage reasoning:** "Revenue is $500M and declined 20% — that's $100M in lost revenue."
- **Unit economics:** "If each truck costs $200/day to operate and we have 400 trucks, that's $80K/day or ~$29M/year."
- **Sensitivity analysis:** "If our assumption is off by 50%, does the conclusion change? If not, the assumption is safe enough."

---

### Muscle 5: Communication Under Pressure

The meta-skill. You can have perfect analysis, but if you can't communicate it clearly, concisely, and in real time — you fail the case.

```
BAD COMMUNICATION:
"So, um, I was thinking about the costs and I think maybe
we should look at the fixed costs because, well, variable
costs might also be important but I think fixed costs are
probably the issue, or maybe not, let me think..."

GOOD COMMUNICATION:
"I'd like to structure this in three parts. First, I'll
break down where the cost increase is coming from. Second,
I'll dig into the largest driver. Third, I'll propose
next steps. Let me start with the cost breakdown."
```

**This is the communication technique called "signposting"** — and it's directly referenced in Palantir's Chapter 17. The consulting tradition is where it originates.

---

## 25.5 The Consulting–Palantir Rosetta Stone

Now that you understand the consulting tradition, here's how it maps to what Palantir tests:

| Consulting Concept | Palantir Equivalent | How It Appears in the DS Interview |
|---|---|---|
| **Case prompt** | Scenario / client problem | "A transit authority is experiencing a 40% increase in delays..." |
| **Profit tree / framework** | 7-Step Framework (Ch. 17) | You impose structure on an ambiguous problem |
| **MECE decomposition** | Ontology design (Object Types, Links) | Entities must be mutually exclusive; the model must be exhaustive |
| **Hypothesis-driven thinking** | "Leading with a point of view" | You propose an approach before seeing all the data |
| **Clarifying questions** | Scoping (stakeholders, data, constraints) | "Who's the end user? What data exists today?" |
| **80/20 prioritization** | MVP scoping / phased deployment | "Phase 1 focuses on the top 10 stations by delay volume" |
| **Market sizing** | Data estimation | "How many fraudulent claims exist in the Medicaid system?" |
| **Recommendation** | Actions in the Ontology | "The operator clicks 'Dispatch Backup Train'" |
| **Slide deliverable** | Workshop app / dashboard | The output the client actually *uses* |
| **Implementation plan** | Deployment plan | How you roll it out, train users, iterate |

### The Key Difference

In a consulting case, the output is a **recommendation** — a PowerPoint deck telling the CEO what to do.

In a Palantir case, the output is an **operating system** — a live application that lets the operator *do* it themselves, every day, with data updating in real time.

This is the philosophical gap between consulting and Palantir. Consulting delivers *advice*. Palantir delivers *capability*. Both start with the same analytical rigor, but they end in fundamentally different places.

```
CONSULTING ENDPOINT:         PALANTIR ENDPOINT:
─────────────────────        ─────────────────────
"We recommend reducing       "Here's a Workshop app where
headcount by 15% in the      the regional manager sees
Southeast region to align    staffing levels, costs, and
labor costs with revenue     utilization by location in
targets."                    real time. When a location
                             is over-staffed relative
Delivered: 80-page deck      to demand, they click
                             'Optimize Staffing' and
Lives: in a filing cabinet   the system generates a
                             rebalancing recommendation
                             they can approve or modify."

                             Delivered: live application
                             Lives: in daily operations
```

---

## 25.6 Common Mistakes in Consulting Cases

Before you start practicing cases (next chapter), internalize these failure patterns:

### Mistake 1: Framework Vomiting

Reciting a framework without adapting it to the specific problem.

```
❌ "I'll use the 3C's framework. Company — the company is an
airline. Customers — the customers are passengers. Competition
— the competitors are other airlines."

This tells the interviewer nothing. You just labeled things.

✅ "Given that this is a profitability question, I'll start with
the revenue/cost split. But before I get there — airlines have
a unique cost structure where fuel is 30-40% of operating costs
and is largely uncontrollable. So I suspect the answer lives
in the cost structure, specifically in the areas the airline
CAN control."
```

### Mistake 2: Diving Into Details Too Early

Going deep on one sub-problem before understanding the whole landscape.

```
❌ INTERVIEWER: "Our retail client's profits are declining."
   YOU: "Interesting, let me think about their pricing strategy.
   If they were to implement a dynamic pricing model..."

   (You don't even know if pricing is the issue!)

✅ YOU: "Before I zoom in, I want to map the full picture.
   Is this a revenue problem, a cost problem, or both? And
   is it across all stores or concentrated in specific regions?"
```

### Mistake 3: Ignoring the "So What?"

Producing analysis without connecting it to a decision.

```
❌ "The data shows that 62% of delays occur between 7-9 AM
   and 18% involve signal failures."

   (So what? What should the client DO?)

✅ "62% of delays cluster in the 7-9 AM rush, with signal
   failures as the leading cause. This tells me two things:
   first, we should prioritize signal infrastructure investment
   at the top 10 rush-hour bottleneck stations. Second, for
   immediate relief, we should increase service frequency by
   10% during peak hours to add buffer capacity."
```

### Mistake 4: Not Doing the Math

Consulting cases often require simple calculations. Candidates who dodge the math or handwave lose credibility.

```
❌ "The revenue impact would be significant."

✅ "If we reduce wait times by 15 minutes per patient across
   2,000 daily visits, that's 30,000 patient-minutes saved
   per day — or 500 patient-hours. At an average revenue of
   $200 per visit, being able to see even 50 more patients
   per day translates to $10,000 per day, or roughly
   $3.6M additional annual revenue."
```

### Mistake 5: Forgetting the Human Element

Cases aren't pure logic puzzles. Real problems involve people — employees, customers, regulators, unions, communities. The best candidates acknowledge this.

```
✅ "One thing I'd want to flag: if the recommendation involves
   significant layoffs, the implementation plan needs to account
   for severance costs, potential union negotiations, and the
   impact on morale among remaining staff. The financial model
   might say 'cut 200 positions,' but the organizational reality
   is more complex."
```

---

## 25.7 Chapter Summary — Quick Reference

### The Four Case Archetypes
1. **Profitability** — Why are profits declining?
2. **Market Entry** — Should we enter this market?
3. **Market Sizing** — How big is something?
4. **M&A** — Should we acquire this company?

### The Five Core Frameworks
1. **Profit Tree** — Revenue (Price × Volume) vs. Costs (Fixed + Variable)
2. **Market Sizing** — Anchor → Assumptions → Multiply → Sanity Check
3. **3C's + P** — Company, Customers, Competition, Product
4. **M&A Evaluation** — Strategic Rationale → Target Assessment → Valuation → Integration Risk
5. **MECE** — Mutually Exclusive, Collectively Exhaustive (the quality standard)

### The Five Mental Muscles
1. **Hypothesis-driven thinking** — Start with a guess, test it
2. **Structured decomposition** — Big question → tractable sub-questions
3. **80/20 prioritization** — Focus on the 20% that drives 80% of value
4. **Quantitative comfort** — Mental math, estimation, unit economics
5. **Communication under pressure** — Signpost, structure, adapt

### The Consulting → Palantir Translation
- Consulting delivers **advice** (slide decks)
- Palantir delivers **capability** (live operational applications)
- Both require the same analytical rigor; they differ in the **output format**

---

*Next: Appendix C — Consulting Case Playbook (Background Reference Only) →*
