# Chapter 4: Getting In the Door — Registration, Certifications, and Set-Asides

Before a single dollar of government money can flow to your company—before you can bid on a contract, receive a purchase order, or even show up in the federal vendor database—you need to complete a series of administrative steps that have no equivalent in commercial sales. Miss any one of them and you are literally invisible to the federal buyer. This chapter walks through every gate you need to pass through, which ones are mandatory versus strategic, and—critically for software startups—which contract vehicles matter most and how to get on them.

Think of this chapter as building the infrastructure that makes everything in the rest of this book possible.

---

## Part 1: The Mandatory Registration Stack

There are four things every company must have before it can do business with the federal government. They are sequential—each depends on the one before it—and none of them is optional.

### SAM.gov: Your Front Door to the Federal Market

The System for Award Management (SAM.gov) is the federal government's master vendor database. If you are not registered in SAM.gov, you do not exist as far as the government is concerned. You cannot:

- Bid on any federal contract
- Receive any federal award
- Get paid by any federal agency
- Appear in any contracting officer's search results

Think of SAM.gov registration as the equivalent of getting a business license to operate in the government market. Without it, nothing else in this book matters.

**What SAM.gov registration requires:**

| Requirement | Details |
|---|---|
| **Login.gov account** | The government's single sign-on system; needed to access SAM.gov |
| **Legal business name** | Must *exactly* match your IRS records and state incorporation documents |
| **Physical street address** | P.O. boxes are generally not accepted for entity validation |
| **Employer Identification Number (EIN)** | Your federal tax ID; must match IRS records precisely |
| **Business formation documents** | Articles of Incorporation, LLC Agreement, or equivalent |
| **U.S. bank account information** | Routing and account numbers for Electronic Funds Transfer (EFT)—this is how the government pays you |
| **NAICS codes** | Industry classification codes (covered in detail below) |
| **Points of contact** | Accounts Receivable, Electronic, and Government contacts |
| **Representations & Certifications** | A long section of federal contracting compliance attestations |

**How long it takes:** The official estimate is 7-10 business days, but the realistic end-to-end timeline for most entities is **two to four weeks**. If there are errors—and there often are, especially discrepancies between your submitted business name and what the IRS has on file—it can stretch to six weeks or longer. Data must be 100% synchronized across the IRS, your state records, and your SAM submission.

**The trap nobody tells you about:** SAM.gov registration expires after exactly one year. If you let it lapse, you cannot receive payments, your bids become invalid, and any active contracts can be disrupted. Start the renewal process at least 60 days before your expiration date. You must be actively registered both at the time you submit an offer and at the time of contract award—a lapse at either moment is disqualifying.

> **Founder's Note:** The most common cause of SAM.gov delays is a mismatch between your legal business name in SAM and the name on file with the IRS. Before you start the process, pull your IRS confirmation letter and your state incorporation documents. Make sure the name, EIN, and address are identical down to capitalization and punctuation. One misplaced comma can add weeks to the process.

### Unique Entity Identifier (UEI): Your Federal Identity Number

The UEI is a 12-character alphanumeric code that serves as your company's unique identifier across all federal systems. It replaced the DUNS number (previously issued by Dun & Bradstreet) on April 4, 2022.

**Key facts about the UEI:**

- Automatically assigned during the SAM.gov registration process
- Free to obtain (unlike the old DUNS number, which sometimes required paid Dun & Bradstreet services)
- You can obtain a UEI without completing full SAM.gov registration if you only need it for sub-award reporting
- Required for all federal contracts, grants, and cooperative agreements
- Appears on every federal form, contract, and payment document

The practical implication: your UEI becomes the number the government uses to track your entire federal relationship—past performance, payments, debarments, everything. Guard it accordingly.

### CAGE Code: Your Defense-Specific Identifier

The CAGE (Commercial and Government Entity) code is a five-character alphanumeric identifier assigned by the Defense Logistics Agency (DLA). While UEI identifies you across all federal systems, the CAGE code is specifically used within the defense and NATO supply chain.

**Why CAGE codes matter for defense:**

| Use Case | CAGE Code Required? |
|---|---|
| DoD prime contracts | Yes |
| Subcontracting to DoD primes | Yes |
| Facility security clearances | Yes |
| NATO contracting | Yes |
| Non-DoD civilian agency work | Sometimes |
| Federal grants (non-DoD) | Usually no |

For U.S. entities, a CAGE code is automatically assigned during SAM.gov registration if you don't already have one. Assignment typically takes 7-10 business days. International entities must obtain a NATO CAGE (NCAGE) code through their national codification bureau.

The CAGE code ties directly to your physical business location. If you have multiple locations doing government work, each location needs its own CAGE code. For a startup operating out of one office, this is straightforward. It becomes relevant when you open a SCIF or a separate cleared facility.

### NAICS Codes: The Classification System That Determines Your Eligibility

NAICS (North American Industry Classification System) codes are six-digit numbers that classify every business in the economy by industry. In government contracting, the NAICS code assigned to a solicitation determines two critical things:

1. **Which small business size standard applies** — whether your company qualifies as "small" for that particular contract
2. **Which set-aside categories you're eligible for** — certain contracts are reserved for businesses of a specific size under a specific NAICS code

Getting your NAICS codes right is not an administrative detail—it is a strategic decision that shapes the contracts you can pursue.

**NAICS Codes Most Relevant to Software Companies:**

| Code | Description | SBA Size Standard |
|---|---|---|
| **541511** | Custom Computer Programming Services — writing, modifying, testing, and supporting custom software | $34 million avg. annual receipts |
| **541512** | Computer Systems Design Services — planning and designing integrated systems (hardware + software + comms); includes IT consulting, systems integration | $34 million avg. annual receipts |
| **541519** | Other Computer Related Services — disaster recovery, software installation, technical support, IT value-added reselling | $34 million avg. annual receipts |
| **518210** | Computing Infrastructure Providers, Data Processing, Web Hosting, and Related Services — includes cloud hosting, SaaS infrastructure, data center services | $40 million avg. annual receipts |
| **541330** | Engineering Services — if your software is embedded in an engineering system | $25.5 million avg. annual receipts |
| **541715** | R&D in the Physical, Engineering, and Life Sciences — if you're doing research-oriented work | 1,000 employees |

**How size standards work:** The SBA calculates your average annual receipts by totaling your "total income" plus "cost of goods sold" over your **five most recently completed fiscal years**. For a startup with less than five years of history, it averages over the years you have. The critical trap: affiliation rules mean you must include the receipts of all domestic and foreign affiliates. If your startup took venture capital from a firm that controls other portfolio companies, the SBA may aggregate those revenues. This is a real issue for VC-backed companies and worth discussing with government contracting counsel early.

**Consequences of getting NAICS codes wrong:**

- **Too narrow:** You miss contracts where your capabilities are directly relevant because the contracting officer used a different NAICS code
- **Too broad:** You appear to be a generalist when the government is looking for a specialist
- **Wrong size standard:** You might qualify as small under one NAICS code ($34M threshold) but not under another ($25.5M threshold), which determines whether you're eligible for small business set-asides

In SAM.gov, you can list multiple NAICS codes. Most software startups should register under at least 541511 and 541512. If you do any cybersecurity work, add 541519. The contracting officer decides which NAICS code applies to each individual solicitation—you just need to make sure you've listed the right universe of codes in your profile.

---

## Part 2: Small Business Certifications and Set-Asides

The federal government has a statutory goal of awarding **23% of all prime contract dollars** to small businesses. This isn't aspirational—agencies are measured on it, and contracting officers actively look for ways to meet their targets. For a startup, this means the government is structurally incentivized to buy from you.

Beyond the general small business category, several socioeconomic certifications unlock **set-aside contracts**—competitions that are restricted to businesses with specific designations. If you qualify for any of these, they are among the most powerful competitive advantages available in government contracting.

### Federal Small Business Contracting Goals (2025)

| Category | Statutory Goal | Meaning |
|---|---|---|
| **Small Business (overall)** | 23% | Of all federal prime and subcontract dollars |
| **Small Disadvantaged Business (SDB)** | 5% | Includes 8(a) participants; reduced from a previous 15% target |
| **Women-Owned Small Business (WOSB)** | 5% | In industries where women are underrepresented |
| **Service-Disabled Veteran-Owned (SDVOSB)** | 5% | Increased from 3% in recent years—now >$31B annual target |
| **HUBZone** | 3% | Historically Underutilized Business Zones; government consistently struggles to meet this target |

These percentages translate to hundreds of billions of dollars in contracts that are either set aside exclusively for qualified businesses or where qualified businesses receive evaluation preferences.

### The 8(a) Business Development Program

The 8(a) program is widely considered the single most powerful small business certification in government contracting. It provides access to sole-source contracts, mentoring, and reduced competition that can transform a startup's trajectory.

**Eligibility Requirements:**

- At least 51% unconditionally owned and controlled by U.S. citizens who are socially and economically disadvantaged
- Owner's personal net worth ≤ $850,000 (excluding primary residence, business equity, and retirement accounts)
- Owner's adjusted gross income averaged over three years ≤ $400,000
- Owner's total personal assets ≤ $6.5 million
- Business must demonstrate "potential for success" (typically two-year operating history, though the SBA now allows greater flexibility, including using income tax returns showing revenues from any industry, not just the primary one)
- Must meet SBA size standards for primary NAICS code

**What 8(a) gives you:**

- **Sole-source contracts** up to **$4.5 million** for services and **$7 million** for manufacturing—meaning the government can award the contract directly to you without competition
- Exclusive access to 8(a) set-aside competitive procurements
- Business development assistance: mentoring, training, technical support
- A nine-year program participation window (four-year developmental stage, five-year transitional stage)

**The 8(a) sole-source power:** For many startups, the single most important benefit of 8(a) certification is the sole-source threshold. A government Program Manager who wants your specific technology can direct a contract to you without opening it to competition, as long as it's under the dollar threshold and you're the only 8(a) firm they're directing it to. This bypasses the entire competitive procurement process for small to mid-sized contracts and can compress procurement timelines from months to weeks.

> **Strategic Note for Founders:** 8(a) certification is a nine-year clock. Every day you're eligible but not certified is a day of program time you're leaving on the table. If there is any chance you qualify, apply early. The program is most valuable in its first few years, when you can build past performance and relationships while enjoying reduced competition.

### Service-Disabled Veteran-Owned Small Business (SDVOSB)

SDVOSB certification has become dramatically more valuable in recent years. The federal spending goal was increased from 3% to 5%, representing an annual target of **over $31 billion**—and unlike HUBZone, the government actively meets this target.

**Requirements:**

- At least 51% owned and controlled by one or more service-disabled veterans
- Service-connected disability verified by the VA
- Veteran(s) must control daily operations and long-term decision-making
- Must meet SBA size standards
- Principal owner must be a U.S. citizen
- **Mandatory SBA VetCert certification** (as of December 22, 2024—only SBA-certified firms count toward agency goals)

**Process and Timeline:** Applications go through the SBA's VetCert portal. The timeline has improved dramatically: after the SBA cleared a significant backlog, processing now averages approximately **12 days** (down from 90-180 days historically).

**Benefits:**

- Exclusive SDVOSB set-aside contracts
- Sole-source awards up to **$4 million** without competition
- Price evaluation credits in some procurements
- Subcontracting opportunities (prime contractors actively seek SDVOSB partners to meet their own subcontracting goals)

### Women-Owned Small Business (WOSB) and EDWOSB

**Requirements:**

- At least 51% unconditionally and directly owned by one or more women who are U.S. citizens
- Women must manage daily operations and make long-term business decisions
- Must meet SBA size standards for primary NAICS code
- EDWOSB (Economically Disadvantaged WOSB) adds financial criteria: net worth < $850K, AGI averaged over three years < $400K, total assets < $6.5M

**Process:** Apply through the MySBA Certifications portal (certify.sba.gov). Third-party certifiers like WBENC are approved but final certification runs through SBA. Processing typically takes 30-90 days. Certification lasts three years with annual attestation requirements.

**Benefits:** Access to contracts set aside in industries where women-owned businesses are underrepresented. EDWOSB certification opens access to additional NAICS codes.

### HUBZone Certification

HUBZone (Historically Underutilized Business Zones) certification is available to businesses located in designated areas where 35% of their employees reside in a HUBZone. The 3% federal goal has historically been the hardest to meet, meaning there is **unmet demand** for HUBZone firms.

**Requirements:**

- Principal office located in a qualified HUBZone
- At least 35% of employees reside in a HUBZone
- At least 51% owned and controlled by U.S. citizens, a Community Development Corporation, an agricultural cooperative, an Indian tribe, or an Alaska Native Corporation
- Must meet SBA size standards

**Benefits:** Set-aside contracts and **price evaluation preferences** in full-and-open competitions—meaning even when you're competing against larger firms, you get a pricing advantage.

> **Practical Note:** For software companies, which can locate anywhere and whose employees can work remotely, HUBZone certification may be more accessible than it appears. If a significant percentage of your team lives in qualifying zones (check the SBA HUBZone map), and you can establish a principal office in one, you can unlock an underserved certification category.

### Stacking Certifications

A critical insight that many founders miss: these certifications are **not mutually exclusive**. A woman-owned, service-disabled veteran-owned small business located in a HUBZone with 8(a) certification can access multiple pools of set-aside contracts simultaneously. The more certifications you hold, the more procurement pathways open up.

| Certification | Sole-Source Ceiling | Set-Aside Access | Processing Time |
|---|---|---|---|
| **8(a)** | $4.5M services / $7M manufacturing | Yes | Several months |
| **SDVOSB** | $4M | Yes | ~12 days (2025) |
| **WOSB/EDWOSB** | Varies by NAICS | Yes | 30-90 days |
| **HUBZone** | $4M | Yes + price preferences | Variable |

---

## Part 3: Contract Vehicles — The Highways of Federal Procurement

Understanding contract vehicles is essential because in many cases, getting on the right vehicle is more important than winning any individual contract. A contract vehicle is a pre-competed agreement that allows government buyers to purchase from a pre-approved pool of vendors. If you're on the vehicle, procurement is fast and easy. If you're not, the government buyer has to go through a more burdensome process—and they often won't bother.

### GSA Multiple Award Schedule (MAS)

The GSA Schedule is the largest and most widely used federal contract vehicle, accounting for over **$51.5 billion in sales in FY2024**. Approximately 35% of those sales go to small businesses.

**What it is:** A pre-negotiated, government-wide contract that allows any federal agency to purchase your products or services at pre-approved prices. Think of it as the government's Amazon—a catalog of approved vendors with pre-negotiated terms.

**Why it matters:**
- Agencies can buy from GSA Schedule holders with minimal paperwork
- You appear in GSA Advantage! (the government's online shopping portal)
- Provides credibility—being a Schedule holder signals that GSA has vetted your pricing and capabilities
- Simplifies repeat purchases; once an agency finds you on the Schedule, reordering is easy

**Eligibility requirements:**

| Requirement | Details |
|---|---|
| **Business history** | Minimum 2 years in business (but see Startup Springboard below) |
| **Annual revenue** | At least $25,000 |
| **Financial statements** | Auditable financials for 2 years |
| **Past performance** | Typically 3 references |
| **Trade Agreements Act** | Products must comply with TAA (manufactured in the U.S. or designated countries) |

**The Startup Springboard Program:** For companies with less than two years of corporate experience, GSA offers the Startup Springboard alternative pathway. Instead of demonstrating two years of company history, you can qualify through the experience of key personnel, affiliated companies, or demonstrated commercial success. This is specifically designed for tech startups entering the federal market and is worth pursuing if you're early-stage.

**Timeline and cost:**
- Application process: **6-12 months** from start to award
- The application itself is free, but the internal cost of preparing the offer package—commercial sales practices disclosure, financial statements, pricing spreadsheets, technical narratives—typically runs **$25,000-$75,000** in staff time
- Many companies use GSA Schedule consultants ($15,000-$24,000 for full-service acquisition)
- **Minimum sales requirements:** Approximately $25,000 in the first two years, $25,000 annually thereafter. GSA is actively "rightsizing" the Schedule in 2025-2026, letting contracts expire for vendors that don't meet thresholds

**Should a startup bother?** Maybe. The GSA Schedule is most valuable if you have a product that multiple agencies will buy repeatedly—a SaaS platform, a cybersecurity tool, a data analytics product. If you're selling custom development services to a single customer, the ROI is less clear. Consider your GSA Schedule a medium-term investment: it takes 6-12 months to get, but once you have it, it opens a permanent sales channel.

**2025-2026 changes to watch:** GSA is implementing mandatory Transactional Data Reporting (TDR) for most product and cloud SINs by FY2026, meaning GSA will have visibility into your exact prices and quantities for every transaction. This increases compliance burden but also creates more transparency.

### Government-Wide Acquisition Contracts (GWACs)

GWACs are pre-competed, multiple-award contracts that allow all federal agencies to purchase IT products and services. They sit above the GSA Schedule in complexity and prestige. For a software company, GWACs are often the most important contract vehicles in the federal market.

#### Alliant 3

Alliant 3 is the most significant IT services GWAC in the federal government, managed by GSA. Phase 1 awards were announced in February 2026 and the contract officially went live for orders on **March 10, 2026**.

**Scope:** Integrated IT services and service-based solutions—cloud services, cybersecurity, software development, AI/ML, data analytics, and emerging technologies.

**Why it matters:** Alliant is the go-to vehicle for large IT service procurements across the federal government. Being on Alliant gives you access to task orders worth billions collectively.

**Reality for startups:** Getting on a GWAC like Alliant typically requires substantial past performance, financial resources, and technical certifications. Most startups will not hold a GWAC position directly. Instead, the strategy is to **partner with a GWAC holder** as a subcontractor or teaming partner. When a prime contractor on Alliant 3 wins a task order, they often need specialized subcontractors—and that's your opening.

#### SEWP (Solutions for Enterprise-Wide Procurement)

SEWP is NASA's IT products GWAC, widely used across all federal agencies. It is known for three things: speed, low prices, and simplicity.

**SEWP V status:** The original ordering period has been extended through **April 30, 2026**, while NASA reviews proposals for **SEWP VI** (the follow-on contract).

**How SEWP works:**
1. Agency defines IT need
2. Agency uses the SEWP Quote Request Tool (QRT) to solicit quotes from contract holders
3. Contract holders respond (often within 24 hours)
4. Agency evaluates and selects
5. SEWP Program Management Office processes and tracks the order

**Key characteristics:**
- Firm-fixed-price, indefinite-delivery/indefinite-quantity (IDIQ) structure
- Very low surcharge: **0.34%** (compared to GSA's 0.75% Industrial Funding Fee)
- Often delivers prices below GSA Schedule
- Covers software products, cloud services, and related support services

**Strategy for startups:** Like Alliant, you're unlikely to be a SEWP contract holder as a startup. The path is to become a partner or reseller through an existing SEWP holder. If your software product can be delivered through an established IT reseller that holds a SEWP contract, your product can reach agencies through the SEWP channel without you holding the contract directly.

#### Other IT Contract Vehicles Worth Knowing

| Vehicle | Manager | Scope | Status |
|---|---|---|---|
| **CIO-SP3** | NIH | IT services for health and biomedical agencies | Active but aging |
| **CIO-SP4** | NIH | Intended successor to CIO-SP3 | **Canceled February 2026** after years of protests |
| **Alliant 2** | GSA | IT services GWAC (IC focus, TS FCL required) | Being superseded by Alliant 3 |
| **ITES-3S** | Army | IT services for Army and other DoD components | Active |
| **NETCENTS-3** | Air Force | Network-centric IT solutions | Active |
| **SeaPort-NxG** | Navy | Engineering, technical, and professional services | Active |

> **The CIO-SP4 Cautionary Tale:** CIO-SP4 was supposed to be the government's premier IT services GWAC, with a potential ceiling of $50 billion. After years of development, a flood of bid protests caused NIH to cancel the entire program in February 2026. This is a reminder that government procurement is never certain—even GWACs that seem inevitable can collapse. Always have multiple vehicle strategies.

### Blanket Purchase Agreements (BPAs)

A BPA is a simplified method of filling anticipated repetitive needs. An agency establishes a BPA with one or more vendors under an existing contract (usually a GSA Schedule) for specific products or services.

**Why BPAs matter:** They are the closest thing the government has to a recurring subscription. Once you have a BPA, the agency can place orders against it with minimal paperwork—often a single-page order form. For SaaS companies, a BPA under a GSA Schedule is the ideal structure because it allows the agency to make annual license renewals without re-competing each year.

**How to get one:** You typically need to be on a GSA Schedule first, then respond when an agency issues a Request for Quotation (RFQ) to establish a BPA for specific products or services.

---

## Part 4: Subcontracting — The Realistic First Step

For most software startups, the first federal revenue will come not from winning a prime contract but from subcontracting to a company that already holds one. This is not a consolation prize—it's a deliberate strategy that builds the three things you need most: past performance, customer relationships, and an understanding of how to work inside the government.

### Why Subcontracting Comes First

1. **Prime contractors have subcontracting requirements.** Any prime contract over $750,000 ($1.5M for construction) requires a subcontracting plan with small business goals. The prime is legally obligated to make good-faith efforts to subcontract to small businesses.

2. **The government tracks it.** Primes report their subcontracting spend to the government in the Electronic Subcontracting Reporting System (eSRS). Failing to meet subcontracting goals can affect a prime's future bid evaluations.

3. **You build citable past performance.** After performing on a subcontract, you have demonstrated experience in a government environment that you can reference in future proposals.

4. **You learn the customer.** Working inside a government program as a subcontractor teaches you the mission, the people, the processes, and the pain points—intelligence that is invaluable when you later pursue prime contracts.

### How to Find Prime Contractors Looking for Subcontractors

| Resource | What it does |
|---|---|
| **SubNet (SBA)** | SBA's Subcontracting Network where primes post subcontracting opportunities |
| **SAM.gov Contract Awards** | Search recent awards by NAICS code to find primes working in your space |
| **DoD Subcontracting Directory** | Lists DoD prime contractors and their small business liaison officers |
| **OSBP Small Business Events** | Each service branch holds "Industry Day" events connecting primes with small businesses |
| **GSA OSDBU** | GSA's Office of Small and Disadvantaged Business Utilization facilitates introductions |

### The Approach

When approaching a prime contractor as a potential subcontractor:

1. **Lead with your NAICS codes and certifications.** If you're 8(a), SDVOSB, or WOSB-certified, say so immediately. Primes need small business subcontractors to meet their goals, and your certifications make you more valuable.

2. **Identify the contract.** Don't approach with a generic "we'd like to subcontract." Research which contracts the prime holds, which programs they support, and where your capabilities fit.

3. **Propose specific work.** Show the prime exactly which piece of the program you can own. For a software startup, this might be: "We can provide the machine learning inference pipeline for the Phase 2 data analytics component you're delivering under contract [X]."

4. **Understand the prime's economics.** The prime typically takes a management fee (often 10-15% overhead wrap) on subcontract work. Know this going in—your pricing needs to account for the prime's margin.

5. **Build the relationship before the re-compete.** If you perform well as a subcontractor, you become part of the prime's team for the next contract. You may also learn enough about the customer to eventually compete as a prime yourself.

---

## Part 5: The DoD Mentor-Protégé Program

The DoD Mentor-Protégé Program (MPP) is a formal mechanism for large defense contractors to help small businesses develop their capabilities. Made permanent by the 2023 NDAA, it creates structured partnerships that go far beyond typical prime-sub relationships.

### How It Works

A large defense contractor (the mentor) enters a formal agreement with a qualifying small business (the protégé) to provide developmental assistance. Agreements typically last **up to three years** and can include:

- Technology transfer
- Technical enhancement and engineering support
- Business development training
- Cybersecurity readiness assistance
- Business infrastructure improvements
- Software development and manufacturing customization (added in recent rule changes)

### Types of Agreements

| Type | How the Mentor Benefits |
|---|---|
| **Reimbursable** | Mentor receives reimbursement from DoD for approved costs of assistance |
| **Credit** | Mentor receives credit toward its small business subcontracting goals |
| **Hybrid** | Combination of reimbursement and credit |

### What This Means for Startups

The mentor-protégé relationship solves several startup problems simultaneously:

1. **Capability building:** The mentor provides resources, training, and technology that you'd otherwise have to develop or purchase independently
2. **Past performance proxy:** Work performed under a mentor-protégé agreement helps establish your track record
3. **Clearance facilitation:** A mentor with existing classified programs can help sponsor your facility clearance
4. **Business pipeline:** Mentors often bring protégés onto their contract teams, creating direct revenue opportunities

### How to Get Started

The DoD OSBPs (Offices of Small Business Programs) do not pair mentors and protégés—you are responsible for finding your own mentor. Start with:

1. Companies you already have a business relationship with
2. The DoD Subcontracting Directory
3. Industry conferences and small business matchmaking events
4. Ask the OSBP at the service branch most relevant to your technology

**Eligibility:** To qualify as a protégé, your company must meet the SBA size standard for the applicable NAICS code. A previous rule requiring protégés to be less than half the size standard has been removed—as long as you're small under the relevant NAICS code, you're eligible.

**Recent changes (2025-2026):**
- Mentor eligibility threshold lowered from $100M to $25M in DoD contracts
- Agreement periods extended to three years
- Pilot program allows protégés to receive up to 25% of the mentor's reimbursement for implementing engineering, software development, or manufacturing customization to integrate technology with a DoD program
- Deadline for mentor reimbursement and subcontracting credit extended to September 30, 2026

### SBA's Mentor-Protégé Program (Separate from DoD)

The SBA also runs its own Mentor-Protégé Program, distinct from the DoD version. The key additional benefit: under the SBA program, a mentor and protégé can form a **joint venture** that qualifies as small for government contract purposes, even if the mentor is a large business. This lets you bid on small business set-aside contracts with the resources and past performance of a large company behind you—a powerful structural advantage.

---

## Part 6: The Startup's Priority Checklist

Not everything in this chapter is equally urgent. Here's the sequence that matters for a software startup entering the federal market, broken into what you should do immediately and what can wait.

### Phase 1: Foundation (Do This Now — Week 1-4)

These are mandatory. You cannot pursue any government business without them.

| Step | Action | Timeline | Cost |
|---|---|---|---|
| 1 | **Get your EIN** from the IRS (if you don't already have one) | 1 day (online) | Free |
| 2 | **Verify your legal business name** matches your IRS records exactly | Same day | Free |
| 3 | **Register on SAM.gov** — provides UEI, CAGE code, and federal vendor status | 2-4 weeks | Free |
| 4 | **Select your NAICS codes** — at minimum, 541511 and 541512 for software | During SAM registration | Free |
| 5 | **Complete Representations & Certifications** in SAM — includes small business self-certification | During SAM registration | Free |

After Phase 1, you are a registered federal vendor. Contracting officers can find you, you can respond to solicitations, and you can receive payments.

### Phase 2: Certifications (Do This Soon — Month 1-3)

These are not mandatory but are strategically powerful if you qualify.

| Step | Action | Timeline | Cost |
|---|---|---|---|
| 6 | **Apply for any applicable small business certifications** (8(a), SDVOSB, WOSB, HUBZone) | 2 weeks – several months | Free |
| 7 | **Self-certify as a small business** under your primary NAICS code in SAM.gov | Already done in Phase 1 | Free |
| 8 | **Research applicable DoD mentor-protégé opportunities** — identify potential mentors in your technology area | Ongoing | Time |

### Phase 3: Market Access (Do This When Ready — Month 3-12)

These expand your addressable market and create procurement channels.

| Step | Action | Timeline | Cost |
|---|---|---|---|
| 9 | **Evaluate whether a GSA Schedule makes sense** for your product/service model | Decision point | N/A |
| 10 | **If yes, begin GSA Schedule application** (consider Startup Springboard if <2 years old) | 6-12 months | $25K-$75K internal |
| 11 | **Identify and approach prime contractors** for subcontracting opportunities under existing GWACs | Ongoing | Time |
| 12 | **Attend agency Industry Days** and OSBP matchmaking events | As scheduled | Travel |

### Phase 4: Scale (Do This After First Contracts — Year 1+)

| Step | Action | Timeline | Cost |
|---|---|---|---|
| 13 | **Evaluate GWAC participation** — either direct (if you have scale) or through teaming agreements | Depends on vehicle timeline | Significant |
| 14 | **Establish BPAs** with agencies that use your product regularly | After GSA Schedule or direct contract | Time |
| 15 | **Build past performance portfolio** from sub and prime contracts for future pursuits | Continuous | N/A |

---

## Part 7: The Things Nobody Tells You

### The Annual Renewal Problem

Your SAM.gov registration, small business certifications, and GSA Schedule all require periodic renewal. Miss a renewal deadline and you can lose contracting eligibility mid-project. Build a compliance calendar from day one. Put every renewal date in a shared calendar with 90-day and 60-day advance reminders.

### The Affiliation Trap

The SBA's affiliation rules can disqualify VC-backed startups from small business status. If your investors have the power to control your company—through board seats, protective provisions, or ownership percentages—the SBA may aggregate the revenues of all companies in the investor's portfolio. A $5M startup can be deemed "large" if its VC fund also controls a $500M portfolio company. This is a real and common problem that requires careful corporate governance structuring, often before you take your first institutional investment.

> **Critical for VC-backed founders:** Get small business affiliation advice from a government contracts attorney *before* you close your next funding round. The structure of your investor rights, board composition, and protective provisions can determine whether you qualify as a small business for federal contracting purposes. Once the investment is structured, it's much harder to fix.

### The Past Performance Catch-22

Many federal solicitations require "relevant past performance" as an evaluation factor—but you can't get past performance without winning contracts, and you can't win contracts without past performance. The practical solutions:

1. **Subcontracting** — builds citable experience under someone else's prime contract
2. **8(a) sole-source** — lets the government award you a contract without competitive past performance requirements
3. **SBIR/STTR** — Phase I and Phase II awards count as past performance
4. **OTAs and CSOs** — Other Transaction agreements through DIU, AFWERX, etc. build demonstrated capability
5. **Commercial references** — Many solicitations will accept commercial past performance as relevant experience, especially for COTS software products

### Price Reasonableness

The government has a legal obligation to pay "fair and reasonable" prices. For your GSA Schedule, this means your government prices generally need to be equal to or better than what you offer your most favored commercial customers. This has implications for your commercial pricing strategy—if you offer a Fortune 500 company a 50% discount, the government will expect at least that discount on your Schedule.

### The 24-Month Clock Is Ticking

From the day you know you want to sell to the government, the 24-month clock starts. It takes 2-4 weeks to register in SAM. It takes months to get certified. It takes 6-12 months to get a GSA Schedule. The first contract can take 6-18 months from initial bid to award. If you wait until you "need" government revenue to start this process, you're already too late by at least a year.

---

## Key Terms

| Term | Definition |
|---|---|
| **SAM.gov** | System for Award Management — the federal government's master vendor registration database |
| **UEI** | Unique Entity Identifier — 12-character code replacing the DUNS number as of April 2022 |
| **CAGE Code** | Commercial and Government Entity Code — 5-character identifier assigned by DLA for defense supply chain |
| **NAICS** | North American Industry Classification System — 6-digit codes classifying businesses by industry |
| **SBA** | Small Business Administration — sets size standards and manages certification programs |
| **8(a)** | SBA's Business Development Program for socially and economically disadvantaged businesses |
| **SDVOSB** | Service-Disabled Veteran-Owned Small Business |
| **WOSB / EDWOSB** | Women-Owned Small Business / Economically Disadvantaged Women-Owned Small Business |
| **HUBZone** | Historically Underutilized Business Zone — location-based certification |
| **GSA MAS** | General Services Administration Multiple Award Schedule — pre-negotiated contract catalog |
| **GWAC** | Government-Wide Acquisition Contract — pre-competed IT contract vehicles available to all agencies |
| **SEWP** | Solutions for Enterprise-Wide Procurement — NASA's IT products GWAC |
| **BPA** | Blanket Purchase Agreement — simplified repeat purchase mechanism |
| **IDIQ** | Indefinite Delivery/Indefinite Quantity — contract type with no guaranteed minimum order |
| **TDR** | Transactional Data Reporting — GSA's requirement for detailed transaction-level reporting |
| **eSRS** | Electronic Subcontracting Reporting System — where primes report small business subcontracting |
| **OSBP** | Office of Small Business Programs — exists within each military department and defense agency |
| **MPP** | Mentor-Protégé Program — formal DoD program pairing large contractors with small businesses |
| **Past Performance** | A company's track record of successfully completing previous government contracts |
| **Affiliation** | SBA rules that may aggregate the size of related companies when determining small business eligibility |
