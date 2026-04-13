# Chapter 25: Palantir in the Real World — Use Cases and Customer Deployments

> *"When someone asks 'What does Palantir actually do?' — the answer isn't software features. It's what happened at the NHS, on the battlefield, on the factory floor, and inside the bank's compliance department."*

---

## Why This Chapter Matters

In a Palantir interview, you'll inevitably be asked to design a solution for a real-world problem. The candidates who stand out don't just describe abstract data pipelines — they reference *actual deployments* that demonstrate how Palantir's platforms create operational impact. Knowing 3–5 specific use cases with concrete metrics transforms you from "someone who read the product docs" into "someone who understands what this company actually does."

This chapter gives you that ammunition: sector-by-sector, deployment-by-deployment, with the specific numbers and outcomes that make your answers credible.

---

## 18.1 Government and Defense

Palantir's origin story is defense. Founded in 2003 with seed funding from the CIA's venture arm (In-Q-Tel), the company built its reputation by integrating intelligence data that saved lives. Today, defense remains its largest revenue segment and the source of its most consequential deployments.

---

### IED Detection — Where It All Started (2007–2012)

**The Problem:** In Iraq and Afghanistan, Improvised Explosive Devices (IEDs) were the leading cause of coalition casualties. The data needed to predict and prevent IED attacks existed — but it was scattered across bomb disposal reports, insurgent network maps, cell phone logs, satellite imagery, and patrol debriefs in incompatible systems.

**What Palantir Did:** Deployed Gotham to the Joint IED Defeat Organization. The platform integrated bomb data, insurgent network intelligence, signals intelligence, and geospatial data into a single analytical workspace — allowing analysts to identify patterns, map supply chains, and predict placement locations.

**Why It Matters for You:** This is the *foundational* Palantir use case. It established the template — take fragmented data, integrate it into a unified model, and enable humans to find patterns they couldn't see before — that drives every deployment today.

---

### Project Maven — Intelligence Fusion at Scale

**The Problem:** The Department of Defense generates vast amounts of Intelligence, Surveillance, and Reconnaissance (ISR) data — satellite imagery, drone feeds, sensor data, signals intelligence. Human analysts can't process it fast enough. The "kill chain" — the time from target detection to engagement — was measured in hours or days.

**What Palantir Did:** Palantir's Maven Smart System uses AI/ML to rapidly process ISR data from diverse sources, identify potential threats, and present actionable intelligence to analysts and commanders.

**Key Facts:**
- Initial contract: $480 million (2024) to expand Maven across global military users
- Expanded to $1.3 billion (May 2025)
- Formalized as an official Program of Record (March 2026) — meaning permanent integration across the U.S. military
- Compressed the kill chain from hours/days to minutes or seconds
- Humans remain responsible for all lethal decisions — Palantir emphasizes this consistently

**Interview Application:** When discussing intelligence fusion, sensor data integration, or real-time decision-making, Maven is the reference point. The key insight: Maven doesn't replace analysts — it gives them processed, prioritized information so they can make better decisions faster.

---

### TITAN — AI-Powered Targeting

**The Problem:** Ground forces need real-time targeting data from space-based, aerial, and terrestrial sensors fused into a single actionable picture — but existing systems couldn't integrate these sources fast enough for modern warfare timelines.

**What Palantir Did:** Won a $178.4 million contract (March 2024) to develop the Tactical Intelligence Targeting Access Node (TITAN) — a mobile ground station that fuses multi-domain sensor data using AI/ML. 10 prototypes are being delivered; field deployment expected by FY2026.

**Why It Matters:** TITAN represents Palantir moving from software into integrated hardware + software systems. It beat RTX (Raytheon) for the contract — a significant signal that Palantir competes (and wins) against legacy defense primes.

---

### The $10 Billion Army Enterprise Agreement (August 2025)

**The Problem:** The U.S. Army had 75+ separate software agreements, creating fragmentation, redundant procurement, and inconsistent data platforms across the service.

**What Palantir Did:** Won a 10-year, $10 billion enterprise agreement that consolidates these 75 agreements into a unified platform for real-time data and AI decision-making across the entire Army.

**Key Facts:**
- Largest single contract in Palantir's history
- Builds on the $618.9 million Army Vantage expansion (December 2024)
- Expected to streamline procurement and create significant cost savings
- Represents a fundamental shift: the Army choosing a commercial AI platform over building in-house

**Interview Application:** This contract is the ultimate proof point for the "platform, not project" thesis. The Army isn't buying a single application — it's buying an operating system for data-driven decision-making across the entire service.

---

### UK Ministry of Defence — £1.5 Billion Strategic Partnership (September 2025)

**The Problem:** The British Armed Forces needed AI-powered capabilities for data analysis, intelligence, decision support, and targeting — fast. Ukraine demonstrated that AI-augmented operations create decisive advantages.

**What Palantir Did:**
- Entered a strategic partnership worth up to £1.5 billion ($2 billion)
- Establishing London as Palantir's European defense headquarters
- Creating up to 350 new high-skilled UK jobs
- Building on technologies already trialed in Ukraine
- Followed by a £240.6 million contract (January 2026) for continued analytics support

**Interview Application:** This demonstrates Palantir's international defense expansion and the "Five Eyes" (US, UK, Australia, Canada, New Zealand) strategy — building allied interoperability through common platforms.

---

### Defense Deployment Reference Card

| Deployment | Date | Value | Key Metric |
|---|---|---|---|
| IED Detection (Gotham) | 2007–2012 | — | Integrated 5+ intelligence streams; predictive IED mapping |
| Project Maven | 2024–2026 | $1.3B | Kill chain compressed from hours to minutes |
| TITAN | 2024 | $178.4M | 10 mobile ground stations; multi-sensor AI fusion |
| Army Enterprise Agreement | 2025 | $10B / 10yr | Consolidates 75 software agreements |
| UK MoD Partnership | 2025 | £1.5B | 350 new UK jobs; Ukraine-tested technology |
| Army Vantage Expansion | 2024 | $618.9M | Expanded AI/analytics across Army |
| USSOCOM Expansion | 2024 | — | Advanced AI and mission manager capabilities |

---

## 18.2 Healthcare

Healthcare is Palantir's fastest-growing commercial vertical and its most compelling "public good" story. The COVID-19 pandemic was the inflection point — Palantir proved that integrated data platforms could literally save lives.

---

### NHS Federated Data Platform — The £330 Million "Digital Nervous System"

**The Problem:** The NHS — the world's fifth-largest employer — operated on fragmented, disconnected data systems. During COVID, this fragmentation made it nearly impossible to understand hospital capacity, PPE supply, or vaccine distribution at a national level in real time.

**What Palantir Did:**

*Phase 1: Pandemic Response (2020–2022)*
- Built a real-time data platform consolidating hospital capacity, PPE inventory, ICU availability, and staffing across England
- Powered the national vaccine rollout — described as "highly successful" by NHS leadership
- Demonstrated what integrated data infrastructure could do under extreme pressure

*Phase 2: Federated Data Platform (2023–present)*
- Awarded a £330 million, 7-year contract to build the NHS Federated Data Platform (FDP)
- Centralizes patient information, care coordination, supply chain management, and vaccination programs
- "Federated" design: data stays within each NHS trust; the platform provides a unified view without centralizing sensitive records

**Concrete Results:**

| NHS Trust | Metric | Improvement |
|---|---|---|
| Chelsea & Westminster | Inpatient waiting list | 28% reduction |
| Chelsea & Westminster | Operations canceled on the day | 50% reduction |
| 22 NHS Trusts (average) | Operating theatre utilization | 6.3% increase |

**Interview Application:** The NHS FDP is the best example of large-scale public sector healthcare transformation. Key talking points: federated architecture (data sovereignty), pandemic-tested, measured outcomes, and the ethical tension around a private company managing public health data.

---

### HCA Healthcare — AI-Powered Nurse Scheduling

**The Problem:** HCA Healthcare — one of the largest U.S. private hospital groups — faced nursing workforce shortages exacerbated by post-pandemic burnout and complex scheduling requirements.

**What Palantir Did:** Built an AI-powered digital nursing scheduling platform that optimizes shift assignments based on patient acuity, nurse skills, availability, regulatory requirements, and historical patterns.

**Concrete Results:**
- 90% reduction in administrative scheduling burden
- $6 million in nursing cost savings at a single major hospital
- 90% reduction in scheduling issues across HCA Healthcare
- Improved nurse retention by reducing administrative friction

**Interview Application:** This is a perfect "operational AI" story — not flashy AI that makes headlines, but AI that solves a grinding daily problem (scheduling 200,000+ nurses) and produces measurable cost savings.

---

### CDC — The $443 Million Common Operating Picture

**The Problem:** The CDC managed disease surveillance through multiple disconnected programs. When COVID hit, there was no unified platform for tracking cases, hospitalizations, testing, and vaccination at the national level.

**What Palantir Did:**
- Powered the DCIPHER (Data Collation and Integration for Public Health Event Response) Program
- Built the HHS Protect platform — the federal government's COVID dashboard
- In December 2022, won a $443 million, 5-year contract to consolidate CDC surveillance into a single "Common Operating Picture"

**Interview Application:** The CDC deployment illustrates the "platform play" in government — start with crisis response, prove value, then expand to become the permanent infrastructure.

---

### NIH — COVID Cohort Research (N3C)

**What Palantir Did:** Powers the National COVID Cohort Collaborative (N3C) Data Enclave — one of the largest clinical research datasets in history. Enables researchers to analyze electronic health records from millions of patients to study COVID-19 and Long COVID.

**Key Facts:**
- $59.5 million NIH contract (2021)
- Integrates diverse datasets for Long COVID research
- Used by thousands of researchers across institutions

---

### Cleveland Clinic, Tampa General, Mount Sinai — Measured Impact

| Hospital System | Use Case | Result |
|---|---|---|
| Cleveland Clinic | Operating room optimization | 40% reduction in unused orthopedic OR time |
| Tampa General Hospital | Staffing and sepsis detection | 30% improvement in nurse staffing ratios; 50% reduction in early sepsis mortality |
| Mount Sinai | Hospital at Home program | 400% increase in admissions; $1.5M annualized ROI from reducing length of stay by 1.5 days |

---

### TeleTracking Partnership (2025)

**What Palantir Did:** Strategic partnership integrating TeleTracking's Operations IQ with Foundry and AIP for real-time hospital operations — predictive demand forecasting, staffing optimization, and resource management.

**Why It Matters:** This expands Palantir from analytics into operational command-and-control for hospitals — exactly the "software that runs things, not just analyzes things" positioning.

---

### Healthcare Deployment Reference Card

| Deployment | Value | Key Metric |
|---|---|---|
| NHS FDP | £330M / 7yr | 28% waiting list reduction, 50% fewer canceled ops |
| HCA Nurse Scheduling | — | 90% admin burden reduction, $6M savings per hospital |
| CDC Common Operating Picture | $443M / 5yr | Unified national disease surveillance |
| NIH N3C Research | $59.5M | Millions of patient records for COVID/Long COVID research |
| Cleveland Clinic | — | 40% reduction in unused OR time |
| Tampa General | — | 50% reduction in early sepsis mortality |
| Mount Sinai | — | 400% increase in Hospital at Home admissions |
| Joint Commission Partnership | — | AI for patient safety standards (2025) |
| TeleTracking | — | Real-time hospital command-and-control |

---

## 18.3 Commercial and Industrial

This is where Palantir's growth story lives. U.S. commercial revenue surged 93% year-over-year in Q2 2025, with the company exceeding $1 billion in quarterly revenue for the first time. The "Bootcamp" sales strategy — immersive workshops where potential customers build solutions on AIP in days — converts nearly 75% of participants into multi-year contracts.

---

### Airbus — Skywise: The Aviation Data Platform

**The Problem:** Airbus needed to integrate data from across its global operations — design, manufacturing, supply chain, and airline customers — into a single platform to improve production efficiency and help airlines optimize operations.

**What Palantir Did:** Co-built Skywise, now the aviation industry's largest data platform, running on Foundry. Partnership ongoing for over a decade.

**Concrete Results:**
- 33% increase in A350 aircraft production delivery rate
- 10,500+ connected aircraft
- 50,000+ daily users across Airbus and its airline customers
- Integrates supply chain, quality, and operational data into a unified view

**Interview Application:** Skywise is the gold standard for "platform as competitive advantage." Airbus doesn't just use Skywise internally — they sell access to airline customers, making it a *ecosystem platform* built on Palantir infrastructure.

---

### bp — A Decade-Long Digital Twin

**The Problem:** bp's global energy operations span offshore platforms, refineries, gas fields, and trading desks across dozens of countries. Optimizing production, maintenance, and safety requires integrating data from thousands of sensors, systems, and sites.

**What Palantir Did:** Built a digital twin of bp's global operations over a 10+ year partnership (since 2014). Expanded in September 2024 to integrate AIP for enhanced AI capabilities.

**Concrete Results:**
- 30,000 additional barrels of production per day
- Hundreds of millions of dollars in additional annual revenue
- Operations span North Sea, Gulf of Mexico, Oman's Khazzan gas fields
- At AIPCon 8 (Sept 2025): presented "One Ontology for a Connected bp" — a single unified data model across the entire company

**Interview Application:** The bp deployment demonstrates long-term platform value. The key insight: the Ontology grows over time. What started as production optimization now spans trading, maintenance, safety, and procurement — all connected through one model.

---

### GE Aerospace — Agentic AI for Military Readiness

**The Problem:** GE Aerospace maintains and produces jet engines for both commercial airlines and the U.S. military. Managing parts demand, maintenance schedules, and production across thousands of engines requires coordinating complex supply chains.

**What Palantir Did:** Expanded multi-year partnership (March 2026) to deploy agentic AI systems for production, sustainment, and maintenance across GE Aerospace's operations.

**Concrete Results:**
- Improved visibility into parts demand and shortages for T-38 trainer jets and J85 engines
- AIP managing supply chain functions: fulfillment, sourcing, allocation, and customer service
- Focus on U.S. Air Force readiness — keeping aircraft mission-capable

**Interview Application:** This is the "agentic AI" reference point. Unlike traditional dashboards, the system actively manages — recommending purchase orders, flagging shortages, and routing work — with human oversight.

---

### American Airlines — AI-Powered Network Planning

**The Problem:** Airline schedule optimization involves millions of variables — aircraft assignments, crew schedules, gate availability, passenger demand, weather patterns, and regulatory constraints. Small improvements create outsized value.

**What Palantir Did:** Implemented Foundry and a planning tool called Vector for operational scheduling and management.

**Concrete Results:**
- "Tens of millions of dollars" in value unlocked in a single year
- At AIPCon 8: demonstrated a single schedule change that saved $400,000+ after a supervisor validated the AI's recommendation
- System integrates multiple data sources to identify optimization opportunities humans would miss

**Interview Application:** The $400K example is perfect for interviews — it's specific, concrete, and demonstrates the "human-in-the-loop" model. The AI identified the opportunity; a human approved it.

---

### Novartis — Drug Discovery Acceleration with Data42

**The Problem:** Pharmaceutical drug development is slow and expensive. Predicting the right human dose for new compounds requires analyzing vast amounts of preclinical data — a process that historically took a team of scientists about a week per compound.

**What Palantir Did:** Co-developed Data42, a platform consolidating data from 700 million patient records and 3,000 clinical trials to accelerate drug discovery decisions.

**Concrete Results:**
- 90–98% reduction in time for human dose prediction (one week → two hours)
- Now used in nearly 100% of Novartis's small molecule projects
- Increased consistency and reproducibility of predictions
- Earlier project de-risking — kill failing compounds sooner, invest in winners faster

**Interview Application:** This is the best pharma use case. The metric — "from one week to two hours" — is unforgettable. It also illustrates Palantir beyond operations: accelerating *scientific discovery*.

---

### Waste Management (Valoriza) — Smart Environmental Services

**What Palantir Did:** Partnership (November 2025) deploying Foundry and AIP for waste management optimization. Integrates data from IoT-enabled waste containers and vehicle fleets to optimize routes, reduce emissions, and improve service quality.

**Interview Application:** A clean example of IoT + Ontology: waste containers as Object Types with fill-level Properties, trucks as Objects with route Links, and "Optimize Route" as an Action.

---

### Commercial Deployment Reference Card

| Deployment | Sector | Key Metric |
|---|---|---|
| Airbus Skywise | Aerospace | 33% increase in A350 production; 50K daily users |
| bp Digital Twin | Energy | 30K additional barrels/day; hundreds of millions in revenue |
| GE Aerospace | Defense/Aerospace | Agentic AI for engine sustainment and production |
| American Airlines | Aviation | $400K saved from a single schedule change |
| Novartis Data42 | Pharma | 90-98% reduction in dose prediction time |
| Waste Mgmt (Valoriza) | Environmental | IoT-powered route optimization |

---

## 18.4 Financial Services and Anti-Fraud

Financial services has emerged as one of Palantir's fastest-growing verticals, with anti-money laundering (AML) and fraud detection as the primary entry points.

---

### Foundry for AML — The Financial Crime Platform

**The Problem:** Banks are legally required to detect and report suspicious activity, but legacy AML systems generate overwhelming numbers of false positive alerts. Compliance teams spend 90%+ of their time investigating alerts that turn out to be legitimate transactions.

**What Palantir Did:** Built Foundry for AML — a platform that uses entity resolution, behavioral risk models, and network analysis to detect actual financial crime while dramatically reducing false positives.

**Concrete Results (across 12+ global financial institutions):**
- 90% reduction in compliance costs
- 4–4.5x improvement in true positive rates (some sources report up to 40x)
- 50% reduction in investigative time
- 70+ use cases on a single platform
- Full audit trails with data lineage for regulatory compliance

**How It Works:**
1. **Entity Resolution:** Reconciles the same customer across accounts, aliases, and institutions
2. **Behavioral Models:** Detects anomalous patterns — not just rule-based thresholds
3. **Network Analysis:** Maps relationships between accounts, beneficiaries, and counterparties to identify rings and shell structures
4. **Case Management:** Compliance officers review AI-flagged cases, investigate, and file SARs with full audit trails

**Types of Fraud Detected:**
- Transaction fraud — unusual patterns, velocity, or geographies
- Trade-based money laundering — over/under invoicing, false descriptions of goods
- Mortgage fraud — inflated appraisals, straw buyers, identity fraud
- Payment fraud — unauthorized transactions, account takeover
- Sanctions evasion — hidden beneficial ownership, layered transactions

---

### Société Générale — European Banking AML (March 2025)

**What Palantir Did:** Deployed anti-financial crime technology at Société Générale to bolster defenses against money laundering and fraud across its international retail banking operations.

**Interview Application:** Demonstrates Palantir's credibility with major European banks — important because European regulators (especially post-Wirecard) have become extremely demanding about AML compliance.

---

### Fannie Mae — AI-Powered Mortgage Fraud Detection (May 2025)

**What Palantir Did:** Partnered with Fannie Mae to launch an AI-powered Crime Detection Unit targeting mortgage fraud. Projected to save the U.S. housing market millions in future fraud losses.

**Interview Application:** A government-sponsored enterprise (GSE) adopting Palantir for fraud detection sends a powerful signal about platform reliability and regulatory trust.

---

### Financial Services Reference Card

| Capability | Metric |
|---|---|
| False positive reduction | 90% cost reduction |
| True positive improvement | 4–4.5x (or up to 40x) |
| Investigative time | 50% reduction |
| Use cases | 70+ on a single platform |
| Institutions served | 12+ of the world's largest banks |

---

## 18.5 Supply Chain

Supply chain is the cross-cutting use case — it appears in defense (munitions), healthcare (PPE), manufacturing (parts), and retail (products). Palantir's "control tower" model has consistently demonstrated measurable results.

---

### The Control Tower Model

Palantir's supply chain control tower provides end-to-end visibility from raw materials to customer delivery:

```
SUPPLY CHAIN CONTROL TOWER
──────────────────────────

Tier 2         Tier 1         Manufacturing    Distribution    Customer
Suppliers  →   Suppliers  →   Plants       →   Centers      →  Delivery
   ↕              ↕              ↕               ↕              ↕
   └──────────────┴──────────────┴───────────────┴──────────────┘
                              │
                    ┌─────────▼──────────┐
                    │  FOUNDRY ONTOLOGY   │
                    │  (Unified Model)    │
                    │                     │
                    │  • Supplier Risk    │
                    │  • Inventory Levels │
                    │  • Transit Status   │
                    │  • Demand Forecast  │
                    │  • Disruption Alerts│
                    └─────────┬──────────┘
                              │
                    ┌─────────▼──────────┐
                    │   WORKSHOP APP      │
                    │                     │
                    │  • Map View         │
                    │  • Risk Dashboard   │
                    │  • Alert Feed       │
                    │  • What-If Sim      │
                    │  • Action Buttons   │
                    └────────────────────┘
```

### Pandemic-Era Supply Chain Resilience

During COVID, Palantir powered supply chain operations for:
- **HHS Protect:** Real-time PPE inventory and distribution across the U.S.
- **Vaccine Distribution:** Cold-chain logistics tracking, allocation optimization, and equity modeling
- **NHS Supply Chain:** Medical supply visibility across England's healthcare system

### Reported Impact Metrics

| Metric | Result |
|---|---|
| Out-of-stock reduction | Up to 50% reduction in stockouts for consumer goods clients |
| Demand forecast accuracy | Significant improvement through ML-driven demand sensing |
| Lead time visibility | Real-time tracking across multi-tier supplier networks |
| Disruption response time | Hours instead of days to identify and mitigate supply disruptions |

---

## 18.6 Energy and Emerging Sectors

---

### Centrus Energy — Classified/Unclassified Integration for Nuclear Supply Chain

**The Problem:** Centrus Energy is expanding U.S. uranium enrichment capacity — a multi-billion-dollar infrastructure project with unique challenges: classified and unclassified operational environments, stringent regulatory compliance, complex manufacturing, and national security implications.

**What Palantir Did:** Deployed AIP and Foundry across Centrus's operations (announced March 2026) to optimize project controls, engineering, manufacturing execution, supply chain management, and regulatory compliance — integrating across classified and unclassified environments.

**Concrete Results:**
- Nearly $300 million in identified potential cost savings and operational efficiencies
- Expected to shorten manufacturing lead times
- Accelerate timeline for bringing additional enrichment capacity online
- Operations centered at Piketon, Ohio expansion facility

**Why This Matters:** This is the purest "sovereign AI" case. The U.S. is rebuilding domestic nuclear fuel supply to reduce dependence on Russian and Chinese enrichment services. Palantir's platform enables that by integrating across security boundaries that would otherwise prevent unified optimization.

**Interview Application:** When discussing classified environments, data sovereignty, or critical infrastructure, Centrus is the reference. It demonstrates Palantir's ability to operate where security requirements are non-negotiable.

---

### Sovereign AI — The Emerging Theme

Palantir increasingly positions itself as a "sovereign AI" platform — software that nations and companies can trust with their most sensitive data because it deploys within their own infrastructure (on-premises or within their own cloud tenants).

**Evidence:**
- UK MoD partnership explicitly focused on UK data sovereignty
- NATO and Five Eyes allied interoperability programs
- Centrus cross-classification boundary integration
- FedRAMP and IL-5/IL-6 (Impact Level) certifications for U.S. government classified environments

---

## 18.7 AIPCon 8 — The Commercial Coming-Out Party (September 4, 2025)

AIPCon 8 was Palantir's biggest annual conference, featuring over 70 U.S. commercial customers — many sharing their Palantir deployment publicly for the first time. Key presentations:

| Company | Title / Focus | Key Takeaway |
|---|---|---|
| **bp** | "One Ontology for a Connected bp" | A single unified data model spanning the entire company — production, trading, maintenance |
| **American Airlines** | "Transforming Network Planning for Aviation" | $400K savings from a single AI-recommended schedule change |
| **Novartis** | Drug Discovery Acceleration | 90-98% reduction in human dose prediction time |
| **MaineHealth** | Precision Healthcare Delivery | "Right care, right patient, right time" — AI-optimized patient routing |
| **TWG Motorsports / Andretti Global** | "RaceOS" | Ultra-fast data integration for real-time racing strategy; predictive car performance modeling |
| **Lumen** | AI and Time-to-Value | Accelerating ROI realization from AI investments |
| **Nebraska Medicine** | Intelligent Automation | Medical necessity validation and reimbursement optimization |

**Why AIPCon Matters for Interviews:** It proves that Palantir has crossed the commercial adoption chasm. 70+ companies — not defense agencies — publicly endorsing Palantir on a livestream represents a fundamental shift in the company's perception.

---

## 18.8 Deployment Patterns — What to Notice

Across all these sectors, there are five recurring patterns. These are what your interviewer wants to hear you articulate:

### Pattern 1: Crisis → Platform → Expansion

Nearly every major deployment follows this arc:

```
CRISIS                → PROVE VALUE           → BECOME INFRASTRUCTURE
(IEDs, COVID, fraud)    (rapid deployment,       (multi-year contract,
                         measurable impact)        platform expansion)

Examples:
• IED crisis     → Gotham intelligence    → $10B Army agreement
• COVID pandemic → NHS vaccine rollout    → £330M Federated Data Platform
• AML compliance → Foundry for AML pilot  → 12+ global bank deployments
```

### Pattern 2: Ontology as Operating System

Every deployment ends up building an Ontology — a unified model of the organization's real-world entities and relationships. Over time, the Ontology becomes the *operating system* for decision-making:

```
Year 1: "We use Palantir for supply chain visibility."
Year 3: "We use Palantir for supply chain, maintenance, and safety."
Year 5: "We run our operations on the Ontology."
```

bp's "One Ontology" presentation at AIPCon 8 is the clearest articulation of this pattern.

### Pattern 3: Actions > Dashboards

The highest-value deployments aren't analytics platforms — they're *operational* platforms with **Actions**:

| Dashboard (Low Value) | Action (High Value) |
|---|---|
| "Show me which providers are suspicious" | "Open Investigation" → assigns case, tracks progress |
| "Display hospital capacity by region" | "Request Transfer" → initiates patient move, notifies receiving hospital |
| "Map delay patterns by subway line" | "Dispatch Backup Train" → creates assignment, notifies crew |

### Pattern 4: Human-in-the-Loop

In every deployment, humans make the final call:
- Maven identifies targets → humans authorize engagement
- AML flags suspicious accounts → compliance officers investigate
- American Airlines AI recommends schedule change → supervisor validates

This is deliberate Palantir philosophy: AI augments human judgment; it doesn't replace it.

### Pattern 5: Measurable Outcomes

Every deployment produces quantifiable metrics — and Palantir is aggressive about publicizing them:

| Type | Examples |
|---|---|
| **Time savings** | Kill chain: hours → minutes; Dose prediction: 1 week → 2 hours |
| **Cost savings** | $6M per hospital (HCA); $400K single change (AA); $300M potential (Centrus) |
| **Efficiency gains** | 33% production increase (Airbus); 30K barrels/day (bp); 90% admin reduction (HCA) |
| **Quality improvements** | 50% fewer canceled ops (NHS); 50% sepsis mortality reduction (Tampa General) |
| **Productivity** | 4–4.5x true positive improvement (AML); 400% admission increase (Mount Sinai) |

---

## 18.9 Your "Ready to Reference" Cheat Sheet

For any interview, be prepared to reference 3–5 of these deployments. Here are the five highest-impact, most versatile examples:

### The Essential Five

| # | Deployment | Sector | One-Line Impact | When to Use |
|---|---|---|---|---|
| 1 | **NHS Federated Data Platform** | Healthcare | 28% wait list reduction; pandemic to platform | Public sector, healthcare, data privacy, federated architecture |
| 2 | **bp "One Ontology"** | Energy | 30K barrels/day; decade-long platform expansion | Ontology design, long-term value, industrial operations |
| 3 | **Project Maven** | Defense | Kill chain: hours → minutes; $1.3B | Real-time AI, human-in-the-loop, intelligence fusion |
| 4 | **Novartis Data42** | Pharma | 1 week → 2 hours for dose prediction | Scientific applications, data consolidation, ML |
| 5 | **American Airlines** | Aviation | $400K from one schedule change | Operational optimization, concrete ROI, human validation |

### How to Reference a Deployment in an Interview

**Don't do this:**
> "Palantir works with the military and hospitals and stuff."

**Do this:**
> "One example that illustrates this well is bp. They've been working with Palantir for over a decade — building what they call 'One Ontology' that connects production, trading, maintenance, and safety data across the entire company. It started with production optimization, where they unlocked an additional 30,000 barrels per day. But the real value has been the compounding effect — once the Ontology was in place, they could layer on new use cases without rebuilding the data infrastructure each time. That's the pattern I'd want to replicate here."

---

## 18.10 Chapter Summary

### By Sector

| Sector | Flagship Deployments | Largest Contract |
|---|---|---|
| **Defense** | Maven, TITAN, Army Enterprise, UK MoD | $10B / 10yr (Army) |
| **Healthcare** | NHS FDP, HCA, CDC, Cleveland Clinic | £330M / 7yr (NHS) |
| **Commercial** | Airbus, bp, American Airlines, Novartis | Decade-long (bp, Airbus) |
| **Financial** | Foundry for AML, Société Générale, Fannie Mae | $443M (CDC fraud) |
| **Energy** | bp, Centrus Energy | £1.5B (UK MoD) |

### Five Patterns to Articulate
1. **Crisis → Platform → Expansion** — Enter during a crisis, prove value, become infrastructure
2. **Ontology as Operating System** — The unified model grows and becomes the organization's decision backbone
3. **Actions > Dashboards** — The highest-value systems let users *do* things, not just *see* things
4. **Human-in-the-Loop** — AI augments; humans decide
5. **Measurable Outcomes** — Every deployment produces quantifiable impact

### The Meta-Point

When you reference these deployments in an interview, you're demonstrating three things simultaneously:
1. **You've done your homework** — you know what Palantir actually does
2. **You think in concrete terms** — not abstractions, but specific metrics and outcomes
3. **You understand the DS role** — because these deployments were designed and delivered by people in the exact role you're interviewing for

---

*Next Chapter: Chapter 26 — Wargame Case Study: Crisis Simulation Architecture →*
