# Chapter 17: Case Study Mastery — Solving Open-Ended Data Problems

> *"The goal isn't the right answer — there isn't one. The goal is demonstrating how you'd actually work through this if a client called you tomorrow."*

---

## Why This Chapter Matters

The open-ended data problem is the *defining* element of the Palantir DS interview. It appears in the phone screen, the analytical/case round, and the decomposition round — three of the five onsite rounds involve it in some form. Unlike coding interviews that have verifiable solutions, these exercises have no single correct answer. What matters is your **process**, your **thinking**, and how you **communicate** both.

This chapter gives you a repeatable framework, six fully worked scenarios, and the communication techniques that separate "pretty good" candidates from "immediate hire" candidates.

---

## 17.1 The 7-Step Framework

Every open-ended data problem — regardless of industry, dataset, or complexity — can be approached with this framework. Memorize the structure; adapt the content.

```
┌─────────────────────────────────────────────────────────┐
│                THE 7-STEP FRAMEWORK                      │
│                                                          │
│  Step 1: UNDERSTAND THE SCENARIO                         │
│  ↓                                                       │
│  Step 2: ASK CLARIFYING QUESTIONS                        │
│  ↓                                                       │
│  Step 3: DEFINE THE DATA LANDSCAPE                       │
│  ↓                                                       │
│  Step 4: DESIGN THE APPROACH                             │
│  ↓                                                       │
│  Step 5: ADDRESS TRADE-OFFS                              │
│  ↓                                                       │
│  Step 6: CONSIDER EDGE CASES                             │
│  ↓                                                       │
│  Step 7: ITERATE                                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

### Step 1: Understand the Scenario

**Goal:** Confirm you know what you're solving and for whom.

Before you think about data or solutions, restate the problem in your own words and identify the stakeholder's *actual* need — which is often different from what they literally said.

**Technique: The Restatement**

```
INTERVIEWER: "A large city transit authority is experiencing a
40% increase in subway delays this year. They've asked for help."

YOU: "So the transit authority is dealing with a significant
spike in delays — nearly double what's expected. Before I dig
in, I want to make sure I understand the goal: are we trying
to identify WHY delays increased, or are we trying to reduce
them going forward, or both? And when you say 'delays' — do
you mean trains arriving late, trains being held at stations,
or service cancellations?"
```

**Why this works:**
- Shows you don't assume
- Distinguishes between diagnosis and prescription
- Reveals hidden complexity (what even counts as a "delay"?)
- Demonstrates client-facing communication skills

**Common mistake:** Jumping straight to "I would pull the delay data and look for patterns." Without understanding the problem, you might solve the wrong thing — exactly what bad DSs do in the field.

---

### Step 2: Ask Clarifying Questions

**Goal:** Scope the problem and uncover constraints before designing anything.

You should ask 3–5 targeted questions before proceeding. These fall into predictable categories:

| Category | Example Questions |
|---|---|
| **Scale** | How many subway lines? How many stations? How many trains per day? |
| **Data availability** | What data does the transit authority currently collect? Do they have sensor data? Maintenance logs? |
| **Success metrics** | How would we know if we've succeeded? Is it reducing average delay? Improving on-time percentage? Reducing customer complaints? |
| **Stakeholders** | Who will use the output? Operations managers? The transit board? The public? |
| **Constraints** | Is there a budget constraint? A timeline? Union rules about scheduling? |
| **History** | Has this been attempted before? What was tried? Why did it fail? |

**Pro tip:** You don't have to ask all of these. The interviewer may not have answers to all of them — and that's fine. The act of asking demonstrates your process even if they say "assume whatever you want."

**What the interviewer is thinking when you ask great questions:**

> *"This person would do well in a client scoping meeting. They're not going to build the wrong thing."*

---

### Step 3: Define the Data Landscape

**Goal:** Identify the entities, relationships, and grain of data you'd need.

This is where your data modeling knowledge (Chapters 12–13) meets the interview. You're essentially doing a mini-ER diagram verbally.

**Framework: The Four Questions**

1. **What are the entities?** (What things exist in this problem?)
2. **What are the relationships?** (How do they connect?)
3. **What is the grain?** (What does one row represent in each dataset?)
4. **What does each row represent?** (If you can't explain this, you don't understand the data.)

**Example — Subway Delays:**

```
ENTITIES:          RELATIONSHIPS:
┌──────────┐      ┌──────────────────────────────────┐
│ Trains   │──────│ A Train runs on a Line            │
│ Lines    │──────│ A Line has many Stations           │
│ Stations │──────│ A Train stops at many Stations     │
│ Delays   │──────│ A Delay occurs at a Station        │
│ Events   │──────│ An Event (weather, maintenance)    │
│ Crew     │      │   may cause a Delay                │
└──────────┘      └──────────────────────────────────┘

GRAIN:
• Delays table: one row per delay event
  (train_id, line_id, station_id, delay_minutes,
   delay_type, timestamp, cause_code)
• Trains table: one row per train
  (train_id, model, last_maintenance_date, age_years)
• Schedules table: one row per scheduled stop
  (train_id, station_id, scheduled_arrival, actual_arrival)
```

**Why this matters:** Defining the data landscape shows the interviewer you think in terms of *data structures*, not just abstract ideas. This is the bridge between "smart person with ideas" and "person who can actually build this in Foundry."

---

### Step 4: Design the Approach

**Goal:** Propose a concrete pipeline from raw data to decision support.

Structure your approach as a flow:

```
DATA SOURCES → INTEGRATION → ANALYSIS → OUTPUT → DECISION SUPPORT

┌─────────────┐    ┌──────────────┐    ┌──────────────┐
│ Data Sources │───→│ Integration  │───→│  Analysis    │
│              │    │              │    │              │
│ • Schedules  │    │ • Clean      │    │ • Descriptive│
│ • Sensors    │    │ • Normalize  │    │   (what      │
│ • Maint logs │    │ • Join       │    │    happened) │
│ • Weather    │    │ • Validate   │    │ • Diagnostic │
│ • Ridership  │    │ • Resolve    │    │   (why)      │
│              │    │   entities   │    │ • Predictive │
└─────────────┘    └──────────────┘    │   (what next)│
                                       └──────┬───────┘
                                              │
                          ┌───────────────────┴───────────────┐
                          │                                    │
                   ┌──────▼──────┐              ┌──────────────▼──┐
                   │   Output    │              │  Decision       │
                   │             │              │  Support        │
                   │ • Dashboard │              │                 │
                   │ • Alerts    │              │ • Prioritized   │
                   │ • Reports   │              │   action list   │
                   │ • Scores    │              │ • What-if       │
                   │             │              │   scenarios     │
                   └─────────────┘              └─────────────────┘
```

**The key insight:** Don't just describe the analysis. Describe the *output* — who sees it, what format, and what decisions it enables. DSs don't produce analyses; they produce *operating capabilities*.

---

### Step 5: Address Trade-offs

**Goal:** Show that you understand real-world constraints.

Every design decision involves trade-offs. Explicitly naming them demonstrates maturity and experience.

**The Big Five Trade-offs:**

| Trade-off | Description | Example |
|---|---|---|
| **Accuracy vs. Speed** | A more accurate model takes longer to build and run | Real-time fraud detection needs speed; quarterly risk assessment can optimize for accuracy |
| **Completeness vs. Timeliness** | Waiting for all data delays decisions | Do we wait for every hospital to report, or act on 80% coverage now? |
| **Centralized vs. Distributed** | One system of record vs. federated data ownership | Central HQ dashboard vs. regional operations teams with their own views |
| **Automated vs. Human-in-the-Loop** | Full automation vs. human review at decision points | Auto-assign field technicians vs. let a manager review and approve |
| **Precision vs. Recall** | Catching more true positives vs. reducing false positives | Fraud detection: flag everything suspicious (high recall, many false alarms) vs. only flag high-confidence fraud (high precision, miss some cases) |

**How to discuss trade-offs in the interview:**

```
"There's a trade-off here between speed and accuracy. If we
build a real-time alerting system, we'll catch delays as they
happen but might generate false alarms from noisy sensor data.
If we batch-process daily, we'll have cleaner insights but
the operations team can't act on them in real time. Given that
the goal is reducing delays — which requires real-time
intervention — I'd lean toward the real-time approach with a
confidence threshold to filter noise. We can always tune the
threshold as we learn."
```

---

### Step 6: Consider Edge Cases

**Goal:** Demonstrate the thoroughness that separates junior from senior thinking.

| Edge Case Category | Questions to Ask Yourself |
|---|---|
| **Missing data** | What if key fields are null? What if a data source goes offline? |
| **Dirty data** | What if timestamps are in different time zones? What if IDs are inconsistent? |
| **Bias** | Is the data representative? Does it undercount certain populations? |
| **Scale** | What if data volume 10x's? Does the approach still work? |
| **Adversarial inputs** | Could someone game the system? (Fraud detection: fraudsters adapt) |
| **Temporal effects** | Does the pattern change over time? Seasonality? COVID-era anomalies? |
| **Dependencies** | What if an upstream data source changes schema? |

**You don't need to solve every edge case.** You just need to *name* them. The interviewer wants to see that you're aware of what could go wrong.

```
"One thing I'd want to flag — if we're using historical delay
data to predict future delays, we need to account for the fact
that COVID disrupted ridership patterns dramatically. Training
a model on 2020-2021 data could give us misleading baselines.
I'd probably segment the training data and weight post-pandemic
periods more heavily."
```

---

### Step 7: Iterate

**Goal:** Adapt when the interviewer challenges you or adds complexity.

The interviewer *will* push back. This is not a sign you're wrong — it's a deliberate test of flexibility.

**Common interviewer moves and how to respond:**

| Interviewer Move | What They're Testing | How to Respond |
|---|---|---|
| "What if we don't have that data?" | Adaptability | "Then I'd look for a proxy. If we don't have sensor data, maintenance logs might show patterns." |
| "The client says that's too complicated." | Simplification ability | "Fair point. Let me simplify. The MVP would focus on just the top 10 stations by delay volume." |
| "What would you do first?" | Prioritization | "I'd start with the data audit — understand what we actually have before designing anything." |
| "That wouldn't work because..." | Resilience, listening | "Good point — I hadn't considered that. Let me revise: instead of X, we could Y." |
| "What if the scope doubles?" | Scalability thinking | "If we designed the Ontology well, adding new entities is incremental — it's the initial structure that matters." |

**The meta-skill:** Treat the pushback as *information*, not criticism. Every redirect from the interviewer is giving you new constraints that help you build a better answer. This is *exactly* what working with clients is like.

---

## 17.2 Worked Scenarios

Let's apply the 7-Step Framework to six realistic scenarios. Each one demonstrates different aspects of the framework.

---

### Scenario 1: Subway System Optimization

**The Prompt:** *"A major city's transit authority is experiencing a 40% increase in subway delays. They've asked Palantir for help. Walk me through your approach."*

**Step 1 — Understand:**
"The transit authority wants to reduce delays. I need to understand: are we diagnosing root causes, building a real-time monitoring system, or optimizing schedules? I'd ask the client what outcome they'd value most."

**Step 2 — Clarify:**
- How many lines and stations? (~25 lines, ~470 stations for a system like NYC)
- What data currently exists? (Schedules, real-time GPS, maintenance logs, ridership, weather, incident reports)
- What's the current definition of "delay"? (>5 min behind schedule? Service advisories?)
- Who's the primary user? (Operations control center staff? Executive leadership?)

**Step 3 — Data Landscape:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Trains | One row per train | train_id, model, age, last_maintenance |
| Lines | One row per line | line_id, name, length_km, station_count |
| Stations | One row per station | station_id, name, line_ids, passenger_volume |
| Scheduled Stops | One row per scheduled arrival | train_id, station_id, scheduled_time |
| Actual Arrivals | One row per actual arrival | train_id, station_id, actual_time, delay_min |
| Incidents | One row per incident | incident_id, type, station_id, timestamp, duration |
| Weather | One row per hour per station area | timestamp, precip, temp, conditions |

**Step 4 — Approach:**

*Phase 1: Descriptive Analysis*
- Calculate delay distributions by line, station, time of day, day of week
- Identify the top 20 delay-producing stations (Pareto principle — likely 20% of stations cause 80% of delays)
- Cross-reference delays with incidents, weather, and maintenance events

*Phase 2: Diagnostic Analysis*
- Root cause categorization: infrastructure failures, crew issues, passenger incidents, weather, cascading delays
- Temporal analysis: are delays clustered at peak hours? After specific maintenance events?
- Cascading effect mapping: when one train delays, how does it propagate?

*Phase 3: Operational Dashboard*
- Real-time delay monitoring in a Workshop app
- Predictive alerts: "Based on weather forecast and current maintenance state, Line 7 has a 73% probability of delay exceeding 15 minutes in the next 3 hours"
- Action buttons: "Dispatch backup train," "Reroute via Line 4," "Issue public advisory"

**Step 5 — Trade-offs:**
- Real-time alerts are useful but require clean, low-latency data feeds. If sensor data is unreliable, start with batch analysis and layer in real-time later.
- A simpler dashboard (top-line KPIs) ships faster but lacks the granularity operators need. Build the detailed view first for operators, then aggregate for executives.

**Step 6 — Edge Cases:**
- GPS gaps in underground tunnels → use scheduled-vs-actual at station endpoints instead
- COVID-era data → segment pre/post pandemic for accurate baselines
- Weekend vs. weekday patterns differ fundamentally → separate models

**Ontology Framing:**
- **Object Types:** Train, Line, Station, Delay Event, Incident, Crew Assignment
- **Properties:** train_id, model, age, last_maintenance (Train); delay_minutes, cause_code (Delay Event)
- **Links:** Train → runs on → Line; Delay Event → occurred at → Station; Incident → caused → Delay Event
- **Actions:** "Dispatch Backup Train" (creates a new Crew Assignment, notifies Operations); "Issue Advisory" (triggers public notification pipeline)

---

### Scenario 2: The Classic Taxi Ride Decomposition

**The Prompt:** *"Here's a dataset of 8,000 taxi rides — ID, fare, pickup location, dropoff location, pickup time, dropoff time, distance. Design a system around this data. You have one week."*

> This is Palantir's most well-known interview exercise. Here's how to nail it.

**Step 1 — Understand:**
"I'm given a structured dataset and asked to build something useful in a week. Before I design, I need to know: who's the end user — the taxi company? City regulators? Drivers? Each leads to a very different product."

**Step 2 — Clarify:**
- Is this one city or multiple? (Affects geographic scope)
- Is the goal to optimize operations, detect fraud, understand demand, or something else?
- Do we have driver information? (Driver ID, rating, experience?)
- Is this a one-time analysis or an ongoing operational tool?

**Step 3 — Data Landscape:**

```
EXISTING DATASET (8,000 rows):
┌──────────┬────────────┬────────────┬────────────┬────────┬─────────┐
│ ride_id  │ pickup_loc │ dropoff_loc│ pickup_time│ fare   │ distance│
│ (PK)     │ (lat, lon) │ (lat, lon) │ (datetime) │ (USD)  │ (miles) │
└──────────┴────────────┴────────────┴────────────┴────────┴─────────┘

DERIVED FIELDS I'd compute:
• trip_duration = dropoff_time - pickup_time
• avg_speed = distance / trip_duration
• fare_per_mile = fare / distance
• hour_of_day = extract(hour from pickup_time)
• day_of_week = extract(dow from pickup_time)
• pickup_zone = geospatial bucket (neighborhood, grid cell)
• dropoff_zone = geospatial bucket
```

**Step 4 — Approach (One-Week Sprint):**

*Day 1–2: Data Exploration and Quality*
- Profile the data: nulls, outliers, distributions
- Flag anomalies: $0 fares, zero-distance trips, impossibly fast speeds
- Build derived columns

*Day 3–4: Core Analyses*
- **Demand heatmap:** Where are pickups concentrated? When? → helps fleet positioning
- **Pricing analysis:** Fare per mile by zone, time of day, day of week → identify pricing anomalies
- **Route efficiency:** Compare actual distance to straight-line distance → identify detour-heavy routes
- **Anomaly detection:** Trips with unusually high fare-to-distance ratios → potential fraud indicators

*Day 5: Workshop App*
- Map view of demand hotspots (filterable by time of day)
- Table of flagged anomalous rides for manual review
- Dashboard: avg fare, avg distance, avg duration, peak hours, busiest zones
- Action: "Investigate Ride" → opens detail view with all ride attributes

*Day 6-7: Polish, Test, Present*
- Validate edge cases, add documentation
- Prepare 10-minute demo for stakeholder

**Step 5 — Trade-offs:**
- Building a map view is visually compelling but takes more time than a table. For a one-week sprint, a filterable table with latitude/longitude is the MVP; add the map if time allows.
- 8,000 rows is small enough that performance isn't a concern, but I'd design the pipeline to handle 8 million rows — the architecture shouldn't change.

**Step 6 — Edge Cases:**
- Trips with pickup_time = dropoff_time → zero duration, likely data error
- Trips where distance is 0 but fare is positive → cancelled ride? Minimum fare?
- Rides crossing midnight → ensure duration calculation handles day boundary
- Coordinates outside the expected city → data quality issue or inter-city rides

---

### Scenario 3: Supply Chain Control Tower

**The Prompt:** *"A global consumer goods company wants real-time visibility into their supply chain — from raw material sourcing to retail shelf. How would you build a supply chain control tower?"*

**Step 1 — Understand:**
"A control tower is a centralized monitoring and decision-support system. The goal is to give supply chain leadership a single view of the entire chain, with the ability to identify disruptions, predict shortages, and take corrective action."

**Step 2 — Clarify:**
- How many suppliers, factories, warehouses, and retail locations?
- What systems currently exist? (ERP, WMS, TMS?)
- What's the biggest pain point today? (Stockouts? Excess inventory? Late deliveries?)
- Who will use this daily? (VP Supply Chain? Regional planners? Warehouse managers?)

**Step 3 — Data Landscape:**

| Entity | Source System | Grain |
|---|---|---|
| Suppliers | Procurement system | One row per supplier |
| Raw Materials | ERP (SAP, Oracle) | One row per material per supplier |
| Purchase Orders | ERP | One row per PO line item |
| Shipments | TMS (Transportation) | One row per shipment leg |
| Inventory | WMS (Warehouse) | One row per SKU per location per day |
| Sales / Demand | POS / Retail systems | One row per SKU per store per day |
| Disruptions | External APIs (weather, news) | Event-driven, one row per event |

**Step 4 — Approach:**

```
CONTROL TOWER ARCHITECTURE

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Suppliers   │  │  Factories   │  │ Warehouses   │
│  (Tier 1,2,3)│  │  (Production │  │ (Inventory   │
│              │  │   schedules) │  │   levels)    │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └────────┬────────┴────────┬────────┘
                │                 │
         ┌──────▼─────────────────▼──────┐
         │     ONTOLOGY (Foundry)         │
         │                                │
         │  Supplier ←→ Material ←→ PO    │
         │  PO ←→ Shipment ←→ Warehouse   │
         │  Warehouse ←→ Inventory ←→ SKU │
         │  SKU ←→ Store ←→ Sales         │
         └──────────────┬─────────────────┘
                        │
              ┌─────────▼─────────┐
              │   CONTROL TOWER   │
              │   (Workshop App)  │
              │                   │
              │ • Global Map View │
              │ • Risk Scorecard  │
              │ • Alert Feed      │
              │ • What-If Sim     │
              └───────────────────┘
```

**Ontology Framing:**
- **Object Types:** Supplier, Material, PurchaseOrder, Shipment, Warehouse, Product (SKU), Store
- **Links:** Supplier → supplies → Material; Material → ordered via → PO; PO → fulfilled by → Shipment; Shipment → delivered to → Warehouse; Product → stocked at → Warehouse; Product → sold at → Store
- **Properties:** Supplier: reliability_score, lead_time_days, country_risk | Shipment: status, ETA, carrier | Product: current_inventory, reorder_point, days_of_supply
- **Actions:** "Expedite Shipment" (updates priority, notifies carrier); "Trigger Reorder" (creates new PO for low-stock SKUs); "Switch Supplier" (reassigns material sourcing)

---

### Scenario 4: Healthcare Fraud Detection

**The Prompt:** *"A state Medicaid program suspects significant fraud among healthcare providers. They have 5 years of claims data. How would you approach this?"*

**Step 1 — Understand:**
"We're looking for providers (doctors, clinics, pharmacies) who are billing inappropriately — phantom patients, upcoding, unbundling, kickback schemes. The goal is to identify high-risk providers for investigation, not to adjudicate fraud directly."

**Step 2 — Clarify:**
- How many providers? How many claims per year?
- What does a claims record look like? (Provider, patient, procedure codes, diagnosis codes, amount billed, amount paid)
- Is there a known fraud rate? (Government estimates ~10% in Medicaid)
- Are there existing fraud detection systems? What are they missing?
- What constitutes a "referral for investigation" — what evidence threshold?

**Step 3 — Data Landscape:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Provider | One row per provider | NPI, specialty, location, enrollment_date |
| Patient | One row per patient | member_id, demographics, eligibility_dates |
| Claim | One row per claim line | claim_id, provider, patient, procedure_code, diagnosis_code, date, amount_billed, amount_paid |
| Pharmacy Claim | One row per prescription | rx_id, prescriber, patient, drug, quantity, refill_count |
| Exclusion List | One row per excluded entity | NPI, exclusion_date, reason |

**Step 4 — Approach:**

*Phase 1: Profiling and Peer Comparison*
- For each provider, compute: claims volume, avg claim amount, unique patients, procedure mix, billing pattern (time of day, day of week)
- Compare each provider to their specialty peers. A dermatologist billing 3x the average is suspicious; one billing at the average is not.
- Z-score analysis: flag providers >2 standard deviations from specialty mean on key metrics

*Phase 2: Network Analysis (Link Analysis)*
- Build a provider-patient-pharmacy graph
- Identify rings: groups of providers and patients that refer exclusively to each other
- Detect unusual referral patterns: a general practitioner referring 80% of patients to a single specialist
- Cross-reference with exclusion lists: are any providers billing through excluded entities?

*Phase 3: Temporal Pattern Detection*
- Sudden spikes in billing volume → new fraud scheme?
- Billing on weekends/holidays → unusual for certain specialties
- Patient impossible travel: same patient visiting providers in different cities on the same day

*Phase 4: Risk Scoring and Case Management*
- Aggregate signals into a composite fraud risk score per provider
- Workshop app: ranked list of high-risk providers with evidence summaries
- Investigation workflow: "Open Case" → assign investigator → track progress → "Close Case"

**Ontology Framing:**
- **Object Types:** Provider, Patient, Claim, Prescription, Investigation
- **Links:** Claim → submitted by → Provider; Claim → for → Patient; Provider → referred to → Provider; Investigation → targets → Provider
- **Actions:** "Open Investigation" (creates Investigation object, assigns to investigator); "Flag Claim" (marks claim for review, adds to investigation evidence); "Suspend Provider" (triggers enrollment freeze)

---

### Scenario 5: Fleet Management

**The Prompt:** *"An energy company has 2,000 field technicians serving 50,000 customer sites. They want to optimize dispatching and reduce response times. How would you approach this?"*

**Step 2 — Clarify:**
- What types of service calls? (Emergency repairs, scheduled maintenance, installations?)
- What's the current average response time? What's the target?
- Do technicians have specialized skills? (Some can handle gas, others electric?)
- What data exists today? (Work orders, GPS locations, skill certifications, drive time?)

**Step 3 — Data Landscape:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Technician | One row per tech | tech_id, location (GPS), skills, certification, shift_start, shift_end |
| Customer Site | One row per site | site_id, address, lat/lon, equipment_installed, service_history |
| Work Order | One row per work order | wo_id, site_id, type, priority, skills_required, created_time, desired_completion |
| Assignment | One row per assignment | assignment_id, wo_id, tech_id, dispatched_time, arrived_time, completed_time |
| Equipment | One row per installed unit | equip_id, site_id, model, install_date, last_service, warranty_status |

**Step 4 — Approach:**

*Optimization Model:*
- Match work orders to available technicians based on: skills match, proximity (drive time), priority, and schedule availability
- This is a variant of the vehicle routing problem (VRP) — NP-hard in the general case, but solvable with heuristics at this scale
- Score each potential tech-to-WO assignment and dispatch the best match

*Workshop App:*
- Map view: technician locations (live GPS) + open work orders (priority-colored pins)
- Dispatch console: recommended assignments with explanations ("Tech #127 is 12 miles away, has gas certification, and is available in 45 minutes")
- Actions: "Accept Dispatch," "Reassign," "Escalate Priority"
- Self-service: customer can see estimated arrival time

**Step 5 — Trade-offs:**
- Optimal routing (shortest total distance for all techs) vs. fair workload distribution → optimize for response time, then balance workloads as a secondary objective
- Fully automated dispatch vs. dispatcher review → start with human-in-the-loop to build trust, automate later

---

### Scenario 6: Pandemic Patient Flow

**The Prompt:** *"During a pandemic, the Department of Health wants to understand hospital capacity in real time — beds, ICU availability, ventilator supply, and staffing — across 600 hospitals in the state. Design this system."*

**Step 2 — Clarify:**
- How frequently does data need to refresh? (Hourly? Daily? Real-time?)
- Who's the primary user? (State emergency management? Hospital administrators? Both?)
- What's the primary decision this enables? (Where to send new patients? Where to deploy National Guard medical teams? Where to ship ventilators?)

**Step 3 — Data Landscape:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Hospital | One row per hospital | hospital_id, name, county, total_beds, ICU_beds, surge_capacity |
| Daily Census | One row per hospital per day | hospital_id, date, occupied_beds, ICU_occupied, ventilators_in_use, covid_patients, staff_available |
| Equipment | One row per equipment type per hospital | hospital_id, equip_type, total_count, available_count |
| Transfers | One row per patient transfer | transfer_id, from_hospital, to_hospital, patient_status, transfer_date |
| Regional Data | One row per county per day | county_id, new_cases, positivity_rate, vaccination_rate |

**Step 4 — Approach:**

```
PANDEMIC PATIENT FLOW SYSTEM
─────────────────────────────

DATA FEEDS              ANALYSIS                  OUTPUTS
────────────           ────────────               ──────────
Hospital daily    →    Capacity                →   STATE MAP
  reporting             utilization %               (traffic light
                                                     by hospital)
ICU census        →    Surge                   →   ALERT FEED
  reports                prediction                 (hospitals at
                                                     >85% capacity)
Ventilator        →    Resource                →   TRANSFER
  inventory              allocation                  RECOMMENDATIONS
                         optimization               (send patients
                                                     from Hospital A
County case       →    Trend                        to Hospital B)
  data                   projection
                                                   SUPPLY DISPATCH
Staff              →   Staffing                →   (ship ventilators
  availability           risk                        from reserve to
                                                     Hospital C)
```

**Ontology Framing:**
- **Object Types:** Hospital, DailyCensus, Equipment, Transfer, County
- **Links:** Hospital → located in → County; DailyCensus → reported by → Hospital; Transfer → from → Hospital, Transfer → to → Hospital
- **Actions:** "Request Transfer" (creates Transfer object, notifies receiving hospital); "Deploy Equipment" (creates shipment order, updates Equipment counts); "Issue Surge Alert" (triggers notification to state emergency management)

---

## 17.3 Connecting to the Ontology

Throughout these scenarios, notice the pattern: every good answer ends with an Ontology framing. This is not optional in a Palantir interview — it's what separates a generic consulting answer from a DS answer.

### The 4-Part Ontology Pattern

Every solution you design should end with:

```
┌────────────────────────────────────────────────────────┐
│  1. OBJECT TYPES — What are the real-world things?     │
│     "The core entities are Trains, Stations, Lines,    │
│      and Delay Events."                                │
│                                                        │
│  2. PROPERTIES — What do we need to know about them?   │
│     "Each Train has an age, model, and maintenance     │
│      history. Each Delay has a duration, cause code,   │
│      and affected station."                            │
│                                                        │
│  3. LINKS — How do they relate?                        │
│     "A Train runs on a Line. A Delay Event occurred    │
│      at a Station. An Incident may have caused a       │
│      Delay Event."                                     │
│                                                        │
│  4. ACTIONS — What can the user DO?                    │
│     "An operator can 'Dispatch Backup Train,' which    │
│      creates a new assignment and updates the line's   │
│      active train count. Or 'Issue Advisory,' which    │
│      triggers the public notification system."         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### Why Actions Matter Most

Many candidates can identify Object Types and Properties — that's essentially data modeling. What separates strong candidates is describing **Actions**: the things users can *do* within the system that trigger real-world outcomes.

Actions demonstrate that you're thinking about **operational systems**, not just analytical dashboards. This is the core Palantir philosophy: data should drive decisions and actions, not just charts.

**Weak answer:** "We'd build a dashboard showing delay patterns."

**Strong answer:** "We'd build a Workshop app where the operations controller sees real-time delay status. When they see a cascading delay developing, they click 'Dispatch Backup' — which creates a new crew assignment, notifies the backup driver, and updates the passenger information system. The dashboard is the *surface*; the Actions are the *value*."

---

## 17.4 Communication Techniques

Knowing the framework isn't enough if you can't *communicate* it clearly in a live interview. Here are the techniques that matter:

### Technique 1: Verbal Diagramming

When you're on a video call without a shared whiteboard, you can still communicate structure by describing what you'd draw.

```
"Imagine a pipeline flowing left to right. On the far left,
we have four data sources feeding in: schedules, sensor data,
maintenance logs, and weather. They flow into an integration
layer where we clean, normalize, and join them — think of it
as a funnel. On the other side, we have three outputs branching
out: a real-time dashboard for operators, predictive alerts
going to mobile devices, and a weekly summary report for
leadership. The Ontology sits in the middle as the connective
tissue."
```

If you *do* have access to a whiteboard or screen share, **use it.** Research consistently shows that candidates who diagram score higher on communication. Even simple boxes and arrows help.

### Technique 2: Signposting

Tell the interviewer what you're going to cover before you cover it. This lets them follow your structure and redirect you if needed.

```
"I'm going to approach this in three phases. First, I'll define
the data we'd need and where it comes from. Second, I'll walk
through the analysis pipeline. Third, I'll describe the end-user
experience. Stop me at any point if you want to go deeper on
something."
```

### Technique 3: Acknowledging Unknowns

Palantir values intellectual honesty over false confidence. When you don't know something, name it and propose how you'd resolve it.

| Don't Say | Do Say |
|---|---|
| "I'm sure the data would be in their ERP system." | "My assumption is this data lives in their ERP, but I'd want to verify — some companies keep it in separate systems." |
| "We'd just use machine learning for that." | "An ML approach could work here, but I'd want to understand the data volume first. With only a few thousand records, a rule-based approach might be more interpretable and just as effective." |
| "That's easy." | "That's straightforward on the surface, but there are a few hidden complexities — for example, how we handle historical data during schema changes." |

### Technique 4: The "What I Don't Know" Close

When you finish your answer, proactively name the gaps:

```
"There are a few things I'd want to explore further before
committing to this design. First, I'm not sure how the sensor
data is transmitted — if it's batch, my real-time approach
needs rethinking. Second, I don't know the union rules around
crew scheduling, which could constrain the dispatch
optimization. And third, I'd want to partner with an FDE
to assess the data quality before designing the pipeline."
```

This shows: humility, awareness of unknowns, and knowledge that DSs work *with* FDEs — not in isolation.

### Technique 5: Inviting Collaboration

The interview should feel like a working session, not a presentation. Regularly check in:

- "Does this direction seem right to you?"
- "Is there a data source I'm missing?"
- "Should I go deeper on the analysis, or move to the output?"
- "What does the client care about most — speed to deploy, or completeness?"

This transforms the dynamic from "candidate performing" to "colleagues brainstorming" — which is exactly how DSs work.

---

## 17.5 The Meta-Lesson — What Interviewers Actually Evaluate

Across all case study rounds, here's the truth: **you will not finish.** The problem is deliberately larger than the allotted time. This is by design. What matters is NOT coverage — it's depth of thinking at each step.

### What Gets You a "Strong Hire":

| Signal | How to Demonstrate It |
|---|---|
| **Structured thinking** | Use the 7-Step Framework visibly — announce each step |
| **Data intuition** | Name specific fields, types, and grain — not vague "data" |
| **Palantir fluency** | Frame in Ontology terms — Object Types, Links, Actions |
| **Real-world awareness** | Name trade-offs, edge cases, and stakeholder constraints |
| **Communication** | Signpost, diagram, check in, and narrate your reasoning |
| **Adaptability** | Welcome pushback and iterate gracefully |
| **Honesty** | Acknowledge unknowns instead of bluffing |
| **Operational mindset** | Design systems people *use*, not analyses people *read* |

### What Gets You Rejected:

| Anti-Signal | What It Looks Like |
|---|---|
| **No structure** | Rambling without a clear framework |
| **Abstraction without substance** | "We'd leverage AI and machine learning" without specifics |
| **Ignoring the user** | Designing a system without asking who uses it |
| **Fragility under pushback** | Getting defensive when the interviewer redirects |
| **No Palantir awareness** | Generic consulting answer with no Ontology framing |
| **Monologuing** | Talking for 10+ minutes without checking in |

---

## 17.6 Practice Approach

### How to Practice the 7-Step Framework

1. **Pick a headline** — any news story about a large-scale problem (flight cancellations, water contamination, school overcrowding, wildfire response)
2. **Set a timer for 20 minutes**
3. **Talk through the 7 steps out loud** — record yourself if possible
4. **Review your recording** — did you signpost? Did you define entities? Did you address trade-offs?
5. **Do this once per day for two weeks** — the framework should become automatic

### Practice Prompts

| Prompt | Key Challenge |
|---|---|
| "A national retailer is experiencing 15% shrinkage (theft/loss). Help." | Balancing loss prevention with customer experience; POS data + inventory + camera data |
| "A city wants to reduce gun violence. They have 911 data, arrest records, and social services data." | Ethical considerations, bias in policing data, community trust |
| "An airline is having 3x normal cancellations this summer. Investigate." | Multiple interlocking systems (weather, crew, maintenance, ATC), cascading failures |
| "A pharmaceutical company wants to optimize clinical trial site selection." | Geographic access, demographics, patient recruitment, regulatory constraints |
| "A military logistics command needs to track equipment and supply readiness across 50 bases." | Classification, data sovereignty, rugged/disconnected environments, Palantir Gotham relevance |

---

## 17.7 Chapter Summary — Quick Reference

### The 7-Step Framework
1. **Understand** — Restate the problem, identify the real need
2. **Clarify** — Ask about scale, data, metrics, stakeholders, constraints
3. **Define Data** — Entities, relationships, grain, what each row represents
4. **Design Approach** — Sources → Integration → Analysis → Output → Decisions
5. **Trade-offs** — Accuracy/speed, completeness/timeliness, automated/human
6. **Edge Cases** — Missing data, dirty data, bias, scale, adversarial inputs
7. **Iterate** — Adapt when the interviewer pushes back or adds constraints

### The Ontology Pattern
- **Object Types** → the real-world things
- **Properties** → what you need to know about them
- **Links** → how they relate
- **Actions** → what users can DO (this is the key differentiator)

### Communication Checklist
- [ ] Signpost before you dive in ("I'll cover three areas...")
- [ ] Diagram verbally or on the whiteboard
- [ ] Check in every 3-5 minutes ("Does this direction make sense?")
- [ ] Acknowledge unknowns explicitly
- [ ] Invite collaboration ("What am I missing?")
- [ ] Name specific fields and data types, not abstract "data"
- [ ] Close with what you'd explore further

---

*Next Chapter: Chapter 18 — Decomposition Deep-Dive: Structured Problem-Solving Under Pressure →*
