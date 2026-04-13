# Chapter 22: Decomposition Practice Lab — Eight Complete Scenarios with Analysis

> *"You don't get good at decomposition by reading about it. You get good by doing 30 of them until the structure is automatic and the creativity is free."*

---

## Why This Chapter Matters

Chapter 21 taught you the mechanics — the timing, the moves, the pushback responses. This chapter is pure practice. Eight fully worked decomposition scenarios, each written as a complete simulated interview. For each scenario, you'll see:

1. The prompt
2. A model opening (restatement + questions)
3. A full decomposition with Ontology framing
4. A realistic pushback exchange
5. An analysis box explaining what made this answer strong

These are not summaries — they are detailed, talk-out-loud simulations at the level of specificity you'd need in the actual interview. Read them, then practice your own versions out loud before looking at the model answers.

---

## How to Use This Chapter

**Active practice method:**

1. Read only the prompt
2. Set a timer for 30 minutes
3. Talk through your decomposition out loud (record yourself)
4. Then read the model answer
5. Compare: What did you cover that the model didn't? What did you miss?
6. Do this for all eight scenarios over two weeks

**Passive study method (worse, but better than nothing):**

1. Read each scenario end-to-end
2. Pay attention to the structural patterns — how the answer opens, pivots, and closes
3. Note the level of specificity in data entity definitions
4. Study the pushback exchanges for response patterns

---

## Scenario 1: National Grocery Chain — Food Waste Reduction

### The Prompt

*"A national grocery chain with 1,200 stores wants to reduce food waste. They estimate they're discarding $400M worth of perishable goods annually. You're the DS assigned to this. Walk me through your approach."*

---

### Model Opening (Minutes 0-5)

"$400 million in discarded perishables across 1,200 stores — that's roughly $330K per store per year in waste. Before I design a system, let me understand the landscape.

**Questions:**

1. **Where in the store does waste concentrate?** Is it produce, bakery, deli, dairy, or all of the above? Because the spoilage dynamics are very different — produce deteriorates visibly, but dairy is a date-based discard.

2. **What's the current process when something is about to expire?** Do stores mark it down? Donate to food banks? Just throw it away? Because the answer tells me whether this is a *prediction* problem (forecast demand better) or a *process* problem (the food is there, we're just not moving it before it expires).

3. **What data systems exist today?** I'd expect a POS system tracking sales, an inventory management system tracking stock levels, and purchase orders from distribution centers. Do they have any of these at item-level, or is it category-level?

4. **Do stores have autonomy over ordering, or is it centralized?** If a store manager orders based on gut feeling vs. a central algorithm, the solution is very different.

Let me assume for now that they have POS data at SKU level, a basic inventory system, and centralized ordering through regional DCs."

---

### Decomposition (Minutes 5-15)

"I'd break this into four sub-problems:

```
┌──────────────────────────────────────────────────────────┐
│             REDUCE FOOD WASTE ($400M)                     │
│                                                           │
│  1. FORECAST DEMAND MORE ACCURATELY                       │
│     If we know how much of each item will sell, we        │
│     don't order too much.                                 │
│                                                           │
│  2. OPTIMIZE MARKDOWNS AND DONATIONS                       │
│     For items approaching expiry, trigger actions —        │
│     price reductions, food bank donations — before        │
│     they become waste.                                    │
│                                                           │
│  3. REDESIGN REPLENISHMENT                                │
│     Move from periodic bulk orders to dynamic,             │
│     demand-responsive ordering.                           │
│                                                           │
│  4. MEASURE AND LEARN                                      │
│     Track waste reduction by store, category, and SKU.    │
│     Close the feedback loop.                              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

Sub-problems 1 and 2 are the highest impact. If I can predict demand 10% better, that directly reduces over-ordering. And if I can trigger markdowns 48 hours before expiry instead of discarding, some of that $400M becomes revenue instead of waste. Let me focus there."

---

### Deep-Dive: Demand Forecasting (Minutes 15-25)

"**Data entities for the forecasting layer:**

| Entity | Grain | Key Attributes |
|---|---|---|
| SKU | One row per product | sku_id, category, subcategory, shelf_life_days, avg_retail_price |
| Store | One row per store | store_id, region, format (urban/suburban/rural), sq_footage |
| DailySales | One row per SKU per store per day | sku_id, store_id, date, units_sold, revenue, promotion_flag |
| Inventory | One row per SKU per store per day | sku_id, store_id, date, units_on_hand, units_received, units_wasted |
| Weather | One row per store per day | store_id, date, high_temp, precip, conditions |
| Events | One row per local event | event_id, date, location, type (sports, holiday, school_break) |

**The forecasting model:**

For each SKU at each store, I'd forecast daily demand 7 days out. Variables:
- Historical sales velocity (same day of week, same month, exponentially weighted)
- Day-of-week and seasonality patterns
- Weather impact (grilling items spike in warm weather; soup spikes in cold)
- Local events (sports games increase snack sales near stadiums)
- Promotion calendar (promoted items spike 2-5x, cannibalize similar SKUs)
- Store-level demographics (urban stores sell more prepared foods; suburban stores sell more bulk)

The model doesn't need to be fancy — even a well-tuned regression with these features would beat the current gut-based ordering at most stores. If the data supports it, we can upgrade to gradient-boosted trees, but accuracy gain beyond 'better than gut feeling' has diminishing returns.

**Key output:** A per-SKU, per-store order recommendation. The system tells the store manager: 'For week of March 15, order 120 units of organic strawberries. Last week you ordered 200 and wasted 60.'

**The user experience:**

The store manager opens a Workshop app every Monday morning. They see:

```
┌─────────────────────────────────────────────────────────┐
│  WEEKLY ORDER RECOMMENDATIONS — Store #0847              │
│                                                          │
│  Category: PRODUCE                                       │
│  ──────────────────────────────────────────────────       │
│  Item            Curr Ord   Rec Ord   Δ Waste Risk       │
│  Org Strawberry  200        120       ↓ High → Low       │
│  Bananas         500        480       ── Stable           │
│  Avocados        300        240       ↓ Med → Low        │
│  Baby Spinach    150        90        ↓ High → Med       │
│                                                          │
│  [Accept Recommendations]  [Adjust & Explain]            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

If the manager clicks 'Accept,' the order is placed. If they click 'Adjust & Explain,' they can override but must log a reason — this feeds back into the model ('manager overrode because of expected school event not in our calendar')."

---

### Deep-Dive: Markdown Optimization (Minutes 25-32)

"For items already on the shelf approaching expiry, I'd build a markdown trigger system.

**Logic:**

```
IF days_until_expiry <= 3
   AND current_inventory > predicted_demand_next_3_days × 1.2
THEN trigger markdown:
   - Suggest discount % based on price elasticity for this SKU
   - Notify store staff to move product to 'Quick Sale' section
   - If days_until_expiry = 1 AND quantity > donation_threshold:
     trigger food bank pickup request
```

This requires two things the chain may not have today: reliable expiry tracking per unit, and a connected food bank network.

**For expiry tracking:** If items have a 'best by' date on the package but it's not in the system, the simplest fix is scanning at receiving. When the store receives a shipment, they scan the product and enter the expiry date. Foundry ingests this and computes remaining shelf life daily.

**For food bank integration:** I'd model food banks as an Object Type — FoodBank(bank_id, name, location, accepts_categories, pickup_schedule). An Action called 'Request Donation Pickup' creates a donation record, notifies the nearest eligible food bank, and generates a tax-deductible donation receipt for the store.

**This is the type of answer that makes interviewers sit up** — you're not just analyzing data, you're designing an action loop that creates real-world value. The food bank integration turns waste into donation, which is good for the community AND the company's PR and tax position."

---

### Pushback Exchange (Minutes 32-38)

```
INTERVIEWER: "Store managers are resistant to algorithmic
ordering. They've been doing this for 20 years and trust
their judgment. How do you handle that?"

"That's the most common deployment challenge — user adoption.
I'd take three steps:

First, I'd frame it as assistance, not replacement. The
system makes recommendations; the manager decides. The
'Adjust & Explain' button is deliberate — it gives them
control and dignity.

Second, I'd start with the worst stores. If I pilot this
at the 50 stores with the highest waste rates, the
improvement will be dramatic and visible. When those
managers start talking to their peers about how their
waste dropped 30%, adoption spreads socially.

Third, I'd give managers transparency into WHY the system
recommends what it does. Not a black box — a readable
explanation: 'We recommend 120 strawberries because last
3 Wednesdays you sold 98, 107, and 112, and there's no
promotion this week.' If the reasoning is legible, managers
trust it faster.

What I would NOT do: make the system mandatory from day
one. Top-down tool mandates create resentment. Let the
data speak, let early adopters champion it, and let
adoption be organic with management support."
```

---

### Ontology Framing & Close (Minutes 38-45)

"**Ontology:**

Object Types: Store, SKU, InventoryRecord, OrderRecommendation, MarkdownEvent, FoodBank, DonationRecord

Links:
- Store → stocks → SKU
- OrderRecommendation → for → SKU at → Store
- MarkdownEvent → applied to → SKU at → Store
- DonationRecord → donated to → FoodBank, donated from → Store

Actions:
- 'Accept Order' → places PO with distribution center
- 'Trigger Markdown' → updates pricing system, notifies floor staff
- 'Request Donation Pickup' → creates DonationRecord, notifies FoodBank
- 'Override & Explain' → logs manager judgment for model feedback

**Deployment:**
- Phase 1 (weeks 1-4): Data audit + integration of POS + inventory at 50 pilot stores
- Phase 2 (weeks 5-8): Demand forecasting model trained, order recommendations go live at pilots
- Phase 3 (months 3-6): Roll to all 1,200 stores; add markdown optimization
- Phase 4 (month 6+): Food bank integration, feedback loop, model retraining

**What I'd explore further:** Whether the chain has cold chain temperature data — that would let us predict spoilage based on actual conditions, not just date labels."

---

### ✅ Analysis — What Made This Strong

- **Quantified the problem immediately** — $330K per store per year shows quantitative comfort
- **Distinguished prediction vs. process** — not just "build a model" but understanding there are two levers
- **Named specific fields** — sku_id, shelf_life_days, units_wasted — not vague "data"
- **Designed the user experience** — "store manager opens an app Monday morning" with a concrete mockup
- **Handled the adoption pushback** with empathy and a practical rollout strategy
- **Food bank integration** elevated the answer from "reduce waste" to "create value from waste"

---

## Scenario 2: Electric Utility — Wildfire Prediction

### The Prompt

*"A large electric utility in California wants to predict which power lines are most likely to cause wildfires so they can prioritize maintenance. They've had two catastrophic fires in the past 3 years that were traced to their infrastructure. Walk me through your approach."*

---

### Model Answer (Condensed)

**Opening questions:**
- How many miles of transmission and distribution lines? How many poles/towers?
- What maintenance data exists today — inspection schedules, repair records?
- Do they have weather station data along their corridors?
- What's the regulatory environment — is CPUC mandating specific actions?
- Is there vegetation management data — when were corridors last cleared?

**Decomposition:**

```
WILDFIRE RISK PREDICTION
├── 1. ASSET HEALTH SCORING
│   What's the physical condition of each line/pole?
│   Age, material, maintenance history, fault history
│
├── 2. ENVIRONMENTAL RISK OVERLAY
│   What external conditions increase fire risk?
│   Vegetation proximity, terrain, weather, soil moisture
│
├── 3. COMPOSITE RISK MODEL
│   Combine asset health + environmental risk into
│   a per-segment risk score
│
└── 4. PRIORITIZED ACTION SYSTEM
    Turn risk scores into maintenance work orders
    with optimal routing and resource allocation
```

**Data entities:**

| Entity | Grain | Key Attributes |
|---|---|---|
| LineSegment | One row per ~1 mile section | segment_id, voltage, material, install_year, last_inspection |
| Pole/Tower | One row per structure | structure_id, segment_id, material, height, age, condition_rating |
| InspectionRecord | One row per inspection | inspection_id, structure_id, date, inspector_id, findings[], photos |
| FaultEvent | One row per electrical fault | fault_id, segment_id, timestamp, type (arc/contact/equipment), weather_conditions |
| VegetationSurvey | One row per segment per survey | segment_id, survey_date, clearance_ft, species, growth_rate |
| WeatherForecast | One row per zone per hour | zone_id, timestamp, wind_speed, humidity, temp, red_flag_warning |
| IgnitionIncident | One row per fire event | incident_id, location, cause, segment_id, acreage, damage |

**Risk scoring model:**

```
SEGMENT RISK SCORE = f(
  Asset factors: age, material, fault_history, time_since_inspection
  Vegetation: clearance distance, species flammability, growth rate
  Environment: wind forecast, humidity, temperature, terrain_slope
  Historical: prior ignitions in the area, fire scar proximity
)

OUTPUT:
Each LineSegment gets a daily risk score from 0-100.
Anything above 75 goes into the RED ZONE queue.
```

**The critical insight:**

"The asymmetry of this problem is extreme. A false negative — missing a high-risk line that starts a fire — costs billions and kills people. A false positive — inspecting a line that was fine — costs a few thousand dollars. So the model should be tuned for HIGH RECALL (catch every risky line) even at the cost of false positives. This is the opposite of most prediction problems where precision matters more."

**Workshop app:**

```
┌─────────────────────────────────────────────────────┐
│  WILDFIRE RISK DASHBOARD — Next 72 Hours            │
│                                                      │
│  [MAP VIEW]                                         │
│  Lines colored by risk: 🔴 Red (>75) 🟡 Yellow (50-75) 🟢 Green (<50) │
│                                                      │
│  RED ZONE QUEUE (12 segments require action):       │
│  ─────────────────────────────────────────────       │
│  #1  Segment 4471  Risk: 94  Reason: High wind +    │
│      vegetation encroachment + 8yr old conductor    │
│      [Dispatch Inspection] [De-Energize Line]       │
│                                                      │
│  #2  Segment 2203  Risk: 89  Reason: Recent fault + │
│      red flag weather warning Thursday              │
│      [Dispatch Inspection] [De-Energize Line]       │
│                                                      │
│  PSPS DECISION SUPPORT:                              │
│  If Red Flag Warning activates, the system shows     │
│  which circuits to de-energize, how many customers   │
│  would be affected, and the medical baseline         │
│  vulnerability of those customers.                   │
│  [Simulate PSPS] [Generate Public Notification]     │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**Pushback:**

```
INTERVIEWER: "The utility executives are worried that
this system will generate so many maintenance work orders
that they can't afford to act on them all."

"That's the right worry — and it's why prioritization is
the entire point. The system doesn't say 'inspect everything.'
It says 'here are the 12 segments that have the highest
risk RIGHT NOW, given today's weather forecast, and here's
the estimated cost of inspecting each one vs. the estimated
expected loss if it ignites.'

I'd add a cost-benefit layer: for each red-zone segment,
estimate the cost of inspection ($X), the cost of
de-energization (lost revenue + customer impact), and the
expected loss of ignition (probabilistic damage estimate).

This lets the executive make an informed decision: 'We can
afford to inspect 8 of these 12 today. The system ranks
them by risk-adjusted ROI so we inspect the right 8.'"
```

---

## Scenario 3: Veterans Affairs — Reducing Veteran Homelessness

### The Prompt

*"The Department of Veterans Affairs wants to reduce veteran homelessness. You're the DS. Where do you start?"*

---

### Model Answer (Condensed)

**Opening:**

"This is one of the most complex social problems you can work on, because veteran homelessness isn't one problem — it's a cluster of interconnected problems: housing affordability, mental health (PTSD, substance use), employment, benefits navigation, and social isolation. My approach would focus on connecting existing systems that currently don't talk to each other.

**Questions:**
- How many veterans are currently experiencing homelessness in the target area?
- What VA systems exist? (VistA for health records, HUD-VASH for housing vouchers, VR&E for employment)
- Do we have data on veterans who are *at risk* but not yet homeless?
- What community organizations (shelters, veteran service orgs) have data we could integrate?
- Who is the primary user — VA caseworkers? Regional coordinators?"

**Decomposition:**

```
REDUCE VETERAN HOMELESSNESS
├── 1. IDENTIFY AT-RISK VETERANS EARLY
│   Predict who is likely to become homeless before
│   it happens, using benefit gaps, healthcare
│   patterns, and life events.
│
├── 2. CONNECT VETERANS TO RESOURCES
│   Match homeless/at-risk veterans to the right
│   combination of services: housing, healthcare,
│   employment, benefits.
│
├── 3. COORDINATE ACROSS AGENCIES
│   The VA, HUD, DOL, and community orgs each have
│   a piece. Build a shared operating picture.
│
└── 4. TRACK OUTCOMES AND SUSTAIN HOUSING
    Once housed, are they staying housed? What
    predicts return to homelessness?
```

**Focus on Sub-Problem 1 — Early Warning:**

"The most impactful thing we can build is an early warning system. What signals precede homelessness?

- **Missed VA appointments** — a veteran who stops showing up for mental health care
- **Benefit lapses** — disability compensation stopped, housing voucher not renewed
- **Emergency department visits** — especially for substance use or psychiatric crisis
- **Address changes** — frequent moves or loss of a stable address
- **Discharge from inpatient care** — the transition period is highest risk
- **Recent separation from service** — the first 2 years post-military are peak risk

Each of these is already captured in VA systems but nobody is looking at them in combination. The system would aggregate these signals into a risk score per veteran, and surface a case list to outreach coordinators."

**Ontology:**

```
Object Types:
- Veteran (veteran_id, branch, separation_date, disability_rating,
  current_address_status [stable/unstable/unknown/homeless])
- Benefit (benefit_id, veteran_id, type [disability/housing/employment],
  status [active/lapsed/pending], expiry_date)
- ServiceEngagement (engagement_id, veteran_id, provider, type
  [healthcare/mental_health/substance_use], last_contact_date)
- HousingPlacement (placement_id, veteran_id, type [HUD-VASH/SSVF/
  transitional], start_date, current_status)
- Outreach (outreach_id, veteran_id, coordinator_id, date,
  type [phone/in_person/home_visit], outcome)

Links:
- Veteran → receives → Benefit
- Veteran → engaged with → ServiceEngagement
- Veteran → placed in → HousingPlacement
- Outreach → targets → Veteran
- Outreach → conducted by → Coordinator

Actions:
- 'Initiate Outreach' → assigns coordinator, creates Outreach record
- 'Renew Benefit' → triggers benefit renewal workflow
- 'Request Housing Placement' → creates referral to HUD-VASH
- 'Escalate' → flags veteran for clinical team review
```

**Pushback:**

```
INTERVIEWER: "The VA's data systems are notoriously
fragmented and outdated. VistA is from the 1980s. How
do you deal with that?"

"This is the most common reality in government deployments —
legacy systems that were never designed to interoperate.
My approach:

First, I would not try to replace VistA. That's a decade-long
project others have tried and failed. Instead, I'd build an
integration layer that reads FROM VistA and other systems and
writes to a unified data model in Foundry.

Second, I'd start with the easiest data first: benefit
enrollment data is relatively clean and structured. Healthcare
encounter data is messier. I'd build the risk model on
benefit lapses and appointment no-shows first — available
and reliable — then layer in clinical data as we establish
data pipelines.

Third, for data that doesn't exist digitally — like
community shelter records — I'd start with a simple intake
form that caseworkers fill out when they encounter a
veteran. That becomes a new data source we create."
```

---

## Scenario 4: Airline — Cascading Flight Delays

### The Prompt

*"A major US airline is experiencing 3x normal flight cancellations this summer. They want a system to predict and mitigate cascading delays. Walk me through your approach."*

---

### Model Answer (Condensed)

**Key insight in the opening:**

"Airline delays are a network problem, not an individual flight problem. When Flight 447 is delayed in Chicago, it doesn't just affect passengers on Flight 447 — it delays the connecting flights those passengers need to catch, it means the aircraft for the next flight out of that gate is late, and if the crew times out under FAA duty rules, the next flight gets cancelled entirely. One delay can cascade into 15 cancellations."

**Decomposition:**

```
REDUCE CASCADING DELAYS
├── 1. PREDICT INITIAL DELAYS
│   Weather, ATC flow control, maintenance MELs
│
├── 2. MODEL THE CASCADE
│   Given an initial delay, which downstream flights
│   are affected? Through aircraft rotation, crew
│   legality, and passenger connections.
│
├── 3. OPTIMIZE RECOVERY
│   When disruptions happen, what's the best set of
│   swaps, cancellations, and rebookings to minimize
│   total passenger impact?
│
└── 4. PREPOSITION RESILIENCE
    Before disruptions happen, schedule buffers,
    position spare aircraft, crew reserves at hubs.
```

**Data entities:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Flight | One row per scheduled flight | flight_id, origin, dest, sched_depart, sched_arrive, aircraft_id, status |
| Aircraft | One row per tail number | aircraft_id, type, maintenance_status, current_location, next_scheduled_flight |
| CrewMember | One row per crew member | crew_id, role, base, duty_start, duty_hours_remaining, legal_rest_required |
| CrewAssignment | One row per crew-flight pair | crew_id, flight_id, role |
| Passenger | One row per PNR | pnr, passenger_name, itinerary_flights[], connection_mins, rebooking_priority |
| WeatherForecast | One row per airport per hour | airport_code, timestamp, wind, visibility, convective_activity, FAA_ground_stop |
| ATCAction | One row per ATC directive | action_id, type [ground_stop/ground_delay/miles_in_trail], airport, start_time, end_time |

**The cascade model — crucial detail:**

"The key data structure is the **aircraft rotation chain**. Each aircraft flies a sequence of flights throughout the day: ORD→DEN, DEN→SFO, SFO→LAX, LAX→ORD. If the first flight delays 45 minutes, every subsequent flight in that chain shifts — UNLESS we swap in a different aircraft.

Similarly, the **crew chain**: a crew flies 3-4 legs per day. After 10-12 hours of duty, they legally cannot fly. If a delay pushes them past their duty limit on leg 3, leg 4 is cancelled unless a reserve crew is available.

I'd model both chains as linked graphs:

```
AIRCRAFT ROTATION:
Flight 101 (ORD→DEN, N12345) ──→ Flight 202 (DEN→SFO, N12345) ──→ Flight 303 (SFO→LAX, N12345)
    ↑ 45 min delay here           ↑ now 45 min late              ↑ now 90 min late (gate conflict)

CREW CHAIN:
Crew #772: Flt 101 ──→ Flt 202 ──→ Flt 303 ──→ Flt 404
    Duty start: 0600    +3hrs       +3hrs       +3hrs = 12hrs
                                               ↑ if Flt 303 delays 30 min,
                                                  crew times out before Flt 404
```

The system would simulate these cascades forward: 'If Flight 101 delays 45 minutes, here are the 8 downstream flights affected through aircraft rotation and the 3 affected through crew legality.'"

**Workshop app — Operations Control Center:**

"The airline's Operations Control Center (OCC) would have a real-time view:

1. **Disruption feed:** incoming weather, ATC actions, and maintenance issues — each tagged with estimated impact (# flights, # passengers affected)
2. **Cascade simulation:** for each disruption, a forward simulation showing all affected flights, with severity color-coding
3. **Recovery options:** for each cascade, the system proposes recovery scenarios:
   - Option A: Swap aircraft N12345 with spare at DEN → recovers flights 202-303, costs $15K in repositioning
   - Option B: Cancel flight 303, rebook 180 passengers → passenger cost high, but protects the crew for flight 404
   - Option C: Call reserve crew at SFO → costs overtime premium, recovers all flights
4. **Actions:** 'Execute Swap,' 'Cancel and Rebook,' 'Call Reserve Crew,' 'Issue Passenger Notification'"

---

## Scenario 5: Child Welfare — Abuse Risk Assessment

### The Prompt

*"A state child welfare agency receives 200,000 calls to its abuse hotline annually. They want to improve how they prioritize which reports to investigate first. Walk me through your approach."*

---

### Model Answer (Condensed)

**Opening — this requires ethical framing immediately:**

"This is a problem where the stakes of getting it wrong in either direction are catastrophic. A false negative — under-prioritizing a report of real abuse — means a child stays in danger. A false positive — over-prioritizing and sending investigators to safe homes — wastes scarce resources, traumatizes families, and disproportionately affects communities of color.

Before I discuss any model, I need to understand the current process. How are calls prioritized today?"

**Decomposition:**

```
IMPROVE HOTLINE PRIORITIZATION
├── 1. STRUCTURED INTAKE ASSESSMENT
│   What information is captured on the call?
│   Is it standardized? Can we improve the form?
│
├── 2. RISK-SIGNAL ENRICHMENT
│   Cross-reference the report with existing records:
│   prior reports, prior removals, service history,
│   parent criminal history (carefully), substance
│   use treatment records.
│
├── 3. DECISION SUPPORT (NOT AUTOMATION)
│   Surface relevant history and risk signals to
│   the screener, without making the decision for them.
│   The human decides. The system informs.
│
├── 4. FAIRNESS AUDIT
│   Continuously monitor: are outcomes equitable
│   across demographics? Does the system replicate
│   existing biases in reporting patterns?
│
└── 5. OUTCOME TRACKING
    For investigated reports: was abuse confirmed?
    For uninvestigated reports: did a subsequent
    report come in? This is the feedback loop.
```

**Key design decision — Decision Support vs. Decision Making:**

"I would NOT build a system that assigns a risk score and says 'investigate this one, don't investigate that one.' The consequences are too high for full automation, and the public accountability question is serious: if a child is harmed because an algorithm deprioritized their case, who is responsible?

Instead, I'd build a system that shows the screener everything relevant:

```
┌──────────────────────────────────────────────────────┐
│  INCOMING REPORT — Case #20260327-4491               │
│                                                       │
│  REPORT SUMMARY:                                      │
│  Caller: teacher (mandated reporter)                 │
│  Child: age 7, female                                │
│  Allegation: physical abuse, visible bruising         │
│                                                       │
│  RELEVANT HISTORY:                                    │
│  ⚠️ 2 prior reports on this address (2024, 2025)     │
│  ⚠️ Prior report was screened out — not investigated  │
│  ⚠️ Mother received substance use services (2024)     │
│  ✅ No prior removals                                 │
│  ✅ No domestic violence record                       │
│                                                       │
│  CONTEXTUAL FACTORS:                                  │
│  📍 ZIP code has above-average poverty rate            │
│     (NOTE: poverty is NOT an abuse indicator —          │
│      included for resource referral, not risk)         │
│                                                       │
│  SCREENER DECISION:                                   │
│  [Priority 1 — Investigate within 24hrs]              │
│  [Priority 2 — Investigate within 72hrs]              │
│  [Screen Out — Provide Reason]                        │
│                                                       │
└──────────────────────────────────────────────────────┘
```

The system surfaces *information*, the human makes the *judgment*. But it flags patterns a screener might miss — like two prior reports at the same address that were screened out."

**Pushback:**

```
INTERVIEWER: "Allegheny County in Pennsylvania built
exactly this — a predictive model for child welfare
screening. It was heavily criticized. How would you
address those criticisms?"

"I'm familiar with the AFST [Allegheny Family Screening
Tool] debate. Three main criticisms:

First, the model used government service utilization
data — families who interact with more government services
(Medicaid, SNAP, public housing) scored higher risk.
But these are actually POVERTY indicators, not ABUSE
indicators. The model learned to flag poor families.

I'd address this by excluding features that are proxies
for poverty. Use BEHAVIORAL signals — prior reports,
report characteristics, reporter type — not SOCIOECONOMIC
signals.

Second, the model was opaque — families couldn't know or
challenge their score. I'd make the contributing factors
transparent to both the screener and, where appropriate,
the family.

Third, the model was applied uniformly across demographics
without checking for disparate impact. I'd build continuous
demographic parity monitoring: if Black families are
referred for investigation at disproportionate rates
relative to confirmed abuse rates, something in the
system is producing bias, and we need to find it."
```

---

## Scenario 6: Fast-Food Chain — Labor Scheduling

### The Prompt

*"A national fast-food chain with 8,000 locations wants to optimize labor scheduling. They're spending $3.2B on labor annually and believe 15-20% is misallocated. Walk me through how you'd build this."*

---

### Model Answer (Key Elements)

**Quantification:** "$3.2B × 15% misallocation = $480M in potential savings. Even capturing 1/3 of that is $160M. This is a high-impact problem."

**Key tension:** "Optimizing labor scheduling in fast food is not purely a math problem. These are hourly workers, many with childcare constraints, transportation limitations, and second jobs. A system that optimizes for cost but creates unpredictable schedules will increase turnover — which costs $3-5K per employee to replace. So the optimization function isn't minimize(labor_cost). It's minimize(labor_cost + turnover_cost + customer_wait_time_cost) subject to constraints like minimum shift length, maximum schedule variability, and regulatory requirements."

**Data entities:**

| Entity | Grain | Key Attributes |
|---|---|---|
| Location | One row per store | location_id, format, seating_capacity, drive_thru_flag, region |
| Employee | One row per worker | employee_id, location_id, role, hire_date, availability[], certifications |
| Shift | One row per scheduled shift | shift_id, employee_id, date, start_time, end_time, station |
| Transaction | One row per order | transaction_id, location_id, timestamp, item_count, total, drive_thru_flag |
| DemandForecast | One row per location per 15min | location_id, timestamp, predicted_transactions, predicted_items |

**Core model:** "Predict transaction volume in 15-minute increments for each location, 2 weeks out. Then convert transactions → required labor hours by station (counter, drive-thru, kitchen, prep) using a staffing ratio. Then generate a schedule that matches labor to demand curves while respecting employee availability and labor laws."

**The user — general manager's weekly view:**

```
NEXT WEEK'S SCHEDULE — Store #2847

                Mon   Tue   Wed   Thu   Fri   Sat   Sun
Demand Fcst:    $4.2K $3.8K $4.0K $4.1K $5.8K $6.2K $3.5K
Staff Needed:   8     7     8     8     11    12    7
Staff Sched:    8     7     8     8     10    11    7
Gap:            0     0     0     0     -1    -1    0

⚠️ Friday and Saturday are under-staffed by 1.
   Recommended: Offer overtime to Maria G (available)
   or Request part-time from nearby Store #2903.

[Accept Schedule] [Request Additional Staff] [Adjust Manually]
```

---

## Scenario 7: Bank — Money Laundering Detection

### The Prompt

*"A large bank wants to detect money laundering patterns in transaction data. They currently generate 50,000 alerts per month, but 98% are false positives. The compliance team can only review 5,000. Redesign the system."*

---

### Model Answer (Key Elements)

**Opening insight:** "The problem isn't detection — they're detecting too much. The problem is PRECISION. 98% false positive rate means the compliance team is drowning in noise. My goal is to increase precision from 2% to 20%+ while maintaining recall — not missing fewer real cases."

**Decomposition:**

```
IMPROVE AML DETECTION
├── 1. REDUCE NOISE
│   Better rules, smarter thresholds, contextual filtering
│
├── 2. ENRICH ALERTS
│   Before a human sees an alert, attach context that
│   helps them decide faster
│
├── 3. RISK-STRATIFY ALERTS
│   Not all alerts are equal. Rank by likelihood of
│   true positive so reviewers see the best cases first
│
├── 4. NETWORK ANALYSIS
│   Look beyond individual transactions to patterns
│   across connected accounts and entities
│
└── 5. FEEDBACK LOOP
    When reviewers dismiss or escalate alerts, feed
    that decision back into the scoring model
```

**Key data structure — the entity resolution problem:**

"The hardest technical challenge here is entity resolution. A single person might have 3 accounts at the bank, 2 businesses, and move money through 5 intermediary accounts. The transaction-level system sees 10 unrelated accounts. The entity-level system sees one person with a web of accounts.

I'd build a Person entity in the Ontology that links:
- Personal accounts (matched by SSN, address, phone)
- Business accounts (matched by beneficial ownership)
- Connected accounts (accounts that frequently transact with the person)

Then re-run the alert rules at the ENTITY level, not the account level. A $9,500 deposit (just below the $10K reporting threshold) looks innocent for one account. But if the same person makes 6 deposits of $9,500 across 3 accounts in one week — that's textbook structuring."

**The reviewer's workflow:**

```
┌──────────────────────────────────────────────────────────┐
│  ALERT REVIEW — Alert #AML-2026-44721                     │
│  Priority: HIGH (risk score: 87/100)                      │
│                                                           │
│  ENTITY: [John Smith] — 3 accounts, 1 business            │
│                                                           │
│  PATTERN DETECTED: Structuring                            │
│  ─────────────────────────────────────────                │
│  6 cash deposits of $9,400-$9,800 across 3 accounts       │
│  over 5 business days (total: $57,200)                    │
│                                                           │
│  ENTITY CONTEXT:                                          │
│  • Account opened 3 months ago                            │
│  • Business is a car wash (cash-intensive — EXPECTED?)    │
│  • 2 prior SARs filed on associated entities              │
│  • Entity linked to 4 other alerts in past 90 days        │
│                                                           │
│  [File SAR]  [Dismiss — False Positive]  [Escalate]       │
│  [Request More Info]                                       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

**Pushback and response:**

```
INTERVIEWER: "The bank's legal team is worried that
machine learning models for AML are hard to explain
to regulators. FinCEN wants to know WHY you flagged
something."

"Absolutely right — AML is one of the most heavily
regulated areas, and 'the model said so' is not an
acceptable explanation to FinCEN or OCC examiners.

This is why I'd use an interpretable approach:
1. The initial screening uses rule-based detection
   (structuring thresholds, velocity checks, geographic
   risk ratings) — fully explainable.
2. The risk-stratification layer uses a model, but with
   SHAP values or similar explainability: 'This alert
   scored 87 because: structuring pattern (contributes 40
   points), entity has prior SARs (25 points), new account
   (12 points), high-risk geography (10 points).'
3. Every SAR filing includes a human-written narrative.
   The system pre-populates a draft narrative based on
   the contributing factors, but the reviewer edits and
   approves it.

The model augments the reviewer. It does not replace
the reviewer. And every decision is traceable."
```

---

## Scenario 8: Humanitarian — Refugee Camp Placement

### The Prompt

*"A humanitarian NGO needs to decide where to establish refugee camps in a conflict zone. Populations are displaced, resources are scarce, and the situation changes daily. Design a system to support this decision."*

---

### Model Answer (Key Elements)

**Opening:** "This is a multi-constraint optimization problem under extreme uncertainty. The stakes are life and death, the data is sparse and unreliable, and the situation changes hourly. My design needs to work with *minimal* data and gracefully incorporate new information."

**Decomposition:**

```
REFUGEE CAMP PLACEMENT
├── 1. POPULATION TRACKING
│   Where are displaced populations? How many?
│   Which direction are they moving?
│
├── 2. SITE SUITABILITY SCORING
│   Which locations satisfy the minimum requirements
│   for a camp? (water, access roads, distance from
│   conflict, terrain, political neutrality)
│
├── 3. RESOURCE MATCHING
│   Given available supplies, vehicles, and staff,
│   which sites can we actually operationalize?
│
└── 4. DYNAMIC REPLANNING
    As the conflict shifts, populations move, and
    resources are consumed — continuously update
    the plan.
```

**Data — working with scarcity:**

"In a conflict zone, we don't have clean census data or satellite-linked sensors. We have:
- **UNHCR registration data** — incomplete but structured
- **Satellite imagery** — can estimate population clusters by counting shelters/tents
- **Partner reports** — field teams filing daily situation reports (unstructured text)
- **Open-source intelligence** — social media, news, radio intercepts
- **Geospatial layers** — elevation, water sources, road networks, mine fields

I'd design the system to work at LOW DATA FIDELITY. Every data point gets a confidence rating: 'Satellite imagery from 3 days ago shows ~2,000 structures in grid square X (confidence: medium — cloud cover obscured 30% of the area).' Decision-makers need to know not just the data but how reliable the data is."

**Ontology:**

```
Object Types:
- PopulationCluster (cluster_id, location, estimated_count,
  confidence, source, last_updated, movement_direction)
- CandidateSite (site_id, location, water_access [yes/no/partial],
  road_access [paved/unpaved/none], distance_from_conflict_km,
  terrain, political_status, capacity_estimate)
- Resource (resource_id, type [water_purification/shelters/medical/
  food], quantity, current_location, deployable_within_hours)
- Camp (camp_id, site_id, population, status [planning/establishing/
  operational/closing], capacity, current_resources)

Actions:
- 'Designate Site' → creates Camp, begins resource allocation
- 'Deploy Resources' → assigns Resources to Camp, generates logistics plan
- 'Issue Movement Advisory' → communicates safe routes to field teams
- 'Trigger Evacuation' → if conflict encroaches, initiate camp movement
```

**Critical design decision:**

"The system must support decision-making under uncertainty, not wait for certainty. I'd build a scenario comparison view:

```
SITE COMPARISON — Northern Corridor

                     Site Alpha    Site Bravo    Site Charlie
Water access:        River (2km)   Well (on-site) River (5km)
Road access:         Paved highway Unpaved trail  Paved road
Dist from conflict:  45km          80km           30km ⚠️
Terrain:             Flat          Hilly          Flat
Est. pop nearby:     ~8,000        ~3,000         ~12,000
Setup time:          3 days        7 days         2 days
Risk assessment:     MEDIUM        LOW            HIGH

RECOMMENDATION: Site Alpha offers the best balance of
accessibility, safety, and proximity to displaced populations.
Site Bravo is safer but harder to reach and serves fewer people.
Site Charlie is closest to population but dangerously near
the conflict front.
```

The decision-maker sees the trade-offs explicitly and decides. The system doesn't choose — it illuminates."

---

## 22.1 Pattern Library — Recurring Structures

After working through these eight scenarios, you'll notice recurring patterns. Here are the ones to internalize:

### Pattern 1: The Three-Tier Stakeholder Stack

Almost every problem has three user tiers:
- **Front-line operators** → need action lists and alerts
- **Middle managers** → need dashboards and resource views
- **Executives** → need aggregate metrics and trend lines

Design for the front line first. That's where operational value lives.

### Pattern 2: The Data Quality Hedge

Always acknowledge that data won't be perfect:
- "The system is designed to work even if [data source] is unavailable"
- "I'd start with the data we know we have and architect for data we might get later"
- "Every data input gets a confidence rating"

### Pattern 3: The Ethical Checkpoint

For any scenario involving people — healthcare, criminal justice, education, child welfare — stop and address:
- **Bias:** Does the data reflect systemic inequities?
- **Transparency:** Can affected people understand how the system works?
- **Human-in-the-loop:** Is a human making the final decision?
- **Accountability:** If the system fails, who is responsible?

### Pattern 4: The Phased Deployment

Every answer should end with phases:
- **Phase 1:** Data integration + core MVP (2-4 weeks)
- **Phase 2:** Full feature set for primary users (month 2-3)
- **Phase 3:** Scale to additional users/regions (month 4-6)
- **Phase 4:** Advanced features — prediction, optimization (month 6+)

### Pattern 5: The Action Signature

End every Ontology design with at least 3 Actions that change the real world:
- An Action that **creates** something (work order, outreach record, referral)
- An Action that **modifies** something (adjust staffing, reassign resource, change priority)
- An Action that **triggers** a workflow (notify stakeholder, escalate, generate report)

---

## 22.2 Self-Assessment Rubric

After each practice decomposition, grade yourself:

| Dimension | Did I... | Score (1-5) |
|---|---|---|
| **Restate & Clarify** | Restate the problem? Ask 3+ clarifying questions? | ___ |
| **Stakeholders** | Identify who uses the system and what decisions they make? | ___ |
| **Decomposition** | Break into 3-5 MECE sub-problems? Prioritize? | ___ |
| **Data Specificity** | Name entities, fields, grain, and data quality issues? | ___ |
| **User Experience** | Describe what someone sees on screen and what they click? | ___ |
| **Ontology** | Define Object Types, Links, Properties, and Actions? | ___ |
| **Trade-offs** | Name at least 2 real trade-offs? | ___ |
| **Edge Cases** | Name at least 2 things that could go wrong? | ___ |
| **Pushback Response** | Accept feedback gracefully and revise concretely? | ___ |
| **Communication** | Signpost? Check in? Remain concise? | ___ |
| **Phased Plan** | Propose a phased deployment, not a Big Bang? | ___ |
| **Time Management** | Complete the core answer in ~30 min, leaving room for discussion? | ___ |

**Target: 4+ on every dimension before your interview.**

---

## 22.3 Chapter Summary

This chapter provided eight fully worked decomposition scenarios:

1. **Grocery — Food Waste** → demand forecasting + markdown optimization
2. **Utility — Wildfire Prediction** → asset health scoring + asymmetric risk
3. **Veterans Affairs — Homelessness** → early warning system + cross-agency coordination
4. **Airline — Cascading Delays** → aircraft rotation chains + crew legality graphs
5. **Child Welfare — Abuse Risk** → decision support (not decision making) + fairness audit
6. **Fast Food — Labor Scheduling** → demand-responsive staffing + turnover vs. cost optimization
7. **Bank — Money Laundering** → entity resolution + precision over recall + regulatory explainability
8. **Humanitarian — Refugee Camps** → multi-constraint optimization under data scarcity

### Recurring patterns to internalize:
- Three-tier stakeholder stack (front-line → managers → executives)
- Data quality hedge (design for imperfect data)
- Ethical checkpoint (bias, transparency, human-in-the-loop)
- Phased deployment (MVP → features → scale → advanced)
- Action signature (create, modify, trigger)

---

*With these three chapters — consulting fundamentals (Ch. 25), the decomposition playbook (Ch. 26), and this practice lab (Ch. 27) — you have the complete methodology for the Palantir DS interview's most challenging rounds. Practice daily. Record yourself. Review with the self-assessment rubric. The structure becomes automatic; the creativity flows from there.*
