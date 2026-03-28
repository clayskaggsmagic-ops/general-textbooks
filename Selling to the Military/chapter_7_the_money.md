# Chapter 7: The Money — Pricing, Accounting, and Surviving DCAA Audits

---

In the commercial world, your pricing is simple: you charge what the market will bear. If customers pay it, the price is right. Your internal costs, profit margins, and overhead structure are nobody's business but yours.

In government contracting, the government makes your costs its business. Literally. There is a federal agency—the Defense Contract Audit Agency—whose sole purpose is to examine your books, question your expenses, and determine whether the price you charged was reasonable and the costs you claimed were allowable. There are 19 federal standards governing how you account for costs. There is a regulation—FAR Part 31—that tells you which of your legitimate business expenses the government will reimburse and which it won't. And if you get it wrong, the consequences range from having costs disallowed (you eat them) to suspension and debarment (you're banned from government work) to criminal prosecution under the False Claims Act.

This chapter is about building the financial infrastructure that keeps you out of trouble and keeps the money flowing. It's the unglamorous foundation that determines whether your company survives in this market.

---

## Why Government Accounting Is Fundamentally Different

In the commercial world, you price based on the market. You figure out what customers will pay, you set your price, and the margin between that price and your costs is your profit. Whether your costs are high or low, efficient or wasteful, is your problem.

In government contracting—particularly on cost-type contracts, which are common for R&D, services, and software development—the pricing model is inverted. Instead of pricing from the market down, you price from the costs up. The government pays you for your **allowable, allocable, and reasonable** costs, plus a negotiated fee or profit. Your costs aren't just your problem—they're the government's business, and the government gets to decide whether each of those costs is legitimate.

This cost-based pricing model means that your accounting system isn't just a tool for running your business. It's a compliance system that must produce auditable, defensible evidence that every dollar you charged the government was:

- **Allowable:** Permitted under FAR Part 31 and not on the list of unallowable costs.
- **Allocable:** Properly assigned to the contract that benefited from the work.
- **Reasonable:** Consistent with what a prudent person would pay in similar circumstances.

If a cost fails any of these three tests, it's disallowed—and you have to pay it back, sometimes with interest. This is why commercial accounting systems (QuickBooks, Xero, standard NetSuite) are inadequate for government contracting. They're not designed to track costs at this level of granularity, and they don't produce the documentation that DCAA auditors will demand.

---

## Allowable vs. Unallowable Costs: The FAR Part 31 Bible

FAR Part 31 is the regulation that defines which costs the government will reimburse and which it won't. Subpart 31.2 (FAR 31.201 through 31.205-52) is the specific section that applies to commercial organizations—which is what you are.

### The Express Lane to Trouble

Some costs are **expressly unallowable**, meaning there's no argument, no exception, no gray area. If you include these in your billing, the government will disallow them and potentially refer you for investigation:

| Cost Category | FAR Reference | Rule |
|---|---|---|
| **Alcoholic beverages** | 31.205-51 | Always unallowable. No exceptions. Even the wine at a client dinner. |
| **Entertainment** | 31.205-14 | All entertainment costs are unallowable: tickets, events, social activities, meals at entertainment venues, and any directly associated costs. |
| **Lobbying** | 31.205-22 | Costs to influence legislation, elections, or government officials are unallowable. Narrow exception for responding to documented technical information requests from Congress. |
| **First-class airfare** | 31.205-46 | Costs above the lowest available airfare are unallowable unless you document specific justifications (circuitous routing, excessively prolonged travel, inadequate accommodation). |
| **Fines and penalties** | 31.205-15 | Always unallowable. |
| **Donations and contributions** | 31.205-8 | Unallowable, with narrow exceptions for community service activities. |
| **Organization costs** | 31.205-27 | Costs of incorporating, forming partnerships, or reorganizing are unallowable. |
| **Interest and financing costs** | 31.205-20 | Generally unallowable for most contractors. |
| **Bad debts** | 31.205-3 | Unallowable. |
| **Selling and marketing** | 31.205-38 | Partially allowable, but there are significant restrictions. |

### The Subtlety: Unallowable Costs Still Exist in Your System

Here's what trips up nearly every new contractor: **you don't stop incurring unallowable costs.** You still take clients to dinner. Your CEO still flies business class sometimes. You still pay interest on a line of credit. These costs are perfectly legal business expenses. They're just not costs the government will reimburse.

Your accounting system must **identify and segregate unallowable costs** so they're excluded from any billing, claim, or proposal submitted to the government—but they still need to flow through your books. They're not hidden; they're separated. This requires your chart of accounts to be structured with separate accounts or flags for unallowable costs from the very beginning. Retroactively reclassifying expenses after a DCAA auditor flags them is a much worse outcome than correctly classifying them upfront.

### The 2025 Streamlining Effort

Executive Order 14275 (April 15, 2025)—"Restoring Common Sense to Federal Procurement"—directed the FAR Council to streamline the FAR by eliminating non-statutory, redundant, or obsolete provisions. Class deviation RFO-2025-31 (June 29, 2025) began implementing this for FAR Part 31, aiming to keep only essential cost principles. However, the fundamental categories of unallowable costs are rooted in statute, not just regulation, and the core allowability rules remain fully in effect.

---

## Direct vs. Indirect Costs and Cost Pools

Every cost in your business is either direct or indirect:

**Direct costs** are expenses that can be identified specifically with a particular contract, project, or cost objective. If you're paying a developer to write code for Contract X, that developer's salary is a direct cost of Contract X. If you buy a server specifically for a contract deliverable, that's a direct cost.

**Indirect costs** are expenses that benefit multiple contracts or the company as a whole but can't be assigned to any single contract. Your office rent, your HR department, your company-wide software licenses, your CEO's salary—these benefit every contract your company performs, so they must be allocated across all contracts using a systematic methodology.

### The Three Core Cost Pools

Government contractors typically organize indirect costs into three pools:

**1. Fringe Benefits Pool**

The fringe pool captures the cost of employee benefits associated with labor. It includes:
- Health insurance premiums
- Dental and vision insurance
- Life insurance and disability insurance
- 401(k) or retirement plan employer contributions
- Payroll taxes (employer's share of FICA, FUTA, state unemployment)
- Paid time off (vacation, sick leave, holidays)
- Workers' compensation insurance

The fringe rate is calculated by dividing the total fringe costs by the total labor base (all direct labor, indirect labor, B&P labor, and G&A labor). A typical fringe rate for a small government contractor ranges from 25% to 45% of labor, depending on the richness of your benefits package.

**2. Overhead Pool**

The overhead pool captures the costs of operating the environments and functions that support direct contract work. It includes:
- Supervisory salaries for project management and technical leadership not charged directly
- Facilities costs (rent, utilities, maintenance) for project-related spaces
- IT infrastructure supporting contract performance
- Equipment and tools used across multiple projects
- Training and professional development
- Supplies and materials not charged directly

The overhead rate is typically calculated using direct labor dollars (or direct labor plus applicable fringe) as the allocation base. A small software contractor's overhead rate might range from 40% to 100% or more, depending on facility costs and staffing structure.

**3. General and Administrative (G&A) Pool**

The G&A pool captures the costs of managing the business as a whole:
- Executive management salaries
- Accounting and finance department
- Human resources
- Legal services
- Business insurance
- Corporate facilities (headquarters costs)
- Business development and proposal preparation
- Corporate software licenses (ERP, email, etc.)

The G&A rate is calculated using a broader allocation base—typically "Total Cost Input," which includes all direct costs, overhead costs, and fringe costs. G&A rates for small contractors commonly range from 10% to 25%.

### How the Rates Stack

Here's how indirect rates build on each other in a typical government contract proposal:

| Component | Example |
|---|---|
| **Direct Labor** (1 engineer, 1 year) | $150,000 |
| + Fringe (35% of direct labor) | $52,500 |
| + Overhead (60% of direct labor) | $90,000 |
| **Subtotal before G&A** | $292,500 |
| + G&A (15% of total cost input) | $43,875 |
| **Total Cost** | $336,375 |
| + Fee (8% of total cost) | $26,910 |
| **Total Price** | $363,285 |

This means a $150,000 engineer actually costs the government about $363,000 per year. Those indirect rates aren't profit—they're the real cost of running a compliant business. The fee (8% in this example) is the profit, and the government negotiates it.

---

## Provisional Billing Rates and the Incurred Cost Proposal

### Provisional Billing Rates (PBRs)

Because your actual indirect rates won't be known until the end of your fiscal year—when you know your total costs—you can't bill the government at actual rates during the year. Instead, you establish **provisional billing rates**: estimated indirect rates based on your budget for the upcoming fiscal year.

PBRs are typically submitted to the government (specifically DCMA—Defense Contract Management Agency—or the cognizant contracting officer) at the beginning of your fiscal year. They're calculated by dividing your budgeted indirect costs by your budgeted allocation base for each pool.

You bill the government using these provisional rates all year. If your provisional rates are too high, you over-bill and will owe the government money at true-up. If they're too low, you under-bill and the government will owe you. The goal is to set PBRs as close to actual as possible—the government does not like surprises in either direction.

### The Incurred Cost Proposal (ICP)

At the end of each fiscal year, if you hold any cost-type contracts containing FAR 52.216-7 (Allowable Cost and Payment), you must submit an **Incurred Cost Proposal** to the government. The ICP is the document that reconciles your provisional billing rates with your actual costs for the year—the "true-up."

The ICP is due **six months after your fiscal year ends**. If your fiscal year ends December 31, the ICP is due by June 30. If it ends September 30, it's due March 31.

The ICP is a substantial document. It typically includes:
- A schedule of actual direct costs by contract
- A schedule of actual indirect costs by pool
- Calculation of actual indirect rates
- A reconciliation of provisional billings to actual costs
- A listing of all contracts in your portfolio
- Executive compensation analysis
- A certificate of final indirect costs signed by a responsible official

Failure to submit a timely and adequate ICP can result in payment delays, withholding of up to 5% of billable amounts, increased audit scrutiny, and ultimately unilateral determination of your rates by the government—which will not be in your favor.

---

## Cost Accounting Standards: The 19 Commandments

Cost Accounting Standards (CAS) are a set of 19 standards that govern how government contractors measure, assign, and allocate costs. They exist to ensure consistency and prevent contractors from shifting costs between contracts to maximize reimbursement.

### When CAS Applies

CAS coverage is triggered by contract value:

| Scenario | Coverage Level |
|---|---|
| **Negotiated contract ≥ $7.5M** (the "trigger contract") and total CAS-covered awards < $50M | **Modified CAS coverage** — comply with CAS 401, 402, 405, and 406 only |
| **Single contract ≥ $50M** or total CAS-covered awards ≥ $50M | **Full CAS coverage** — comply with all 19 CAS standards |
| **Contracts < $7.5M**, sealed-bid, firm-fixed-price commercial items, small business | **Exempt** from CAS |

The $7.5M threshold is critical to understand. Once you receive a single negotiated (non-exempt) contract of $7.5 million or more, CAS is "triggered" for your business unit, and all subsequent non-exempt awards above the TINA threshold ($2 million) become CAS-covered, even if they're individually below $7.5 million.

### Modified vs. Full Coverage

**Modified CAS coverage** (the four standards) is a manageable lift for most companies:

- **CAS 401 — Consistency in Estimating, Accumulating, and Reporting Costs.** Your estimates must match your actual accounting. You can't estimate using one methodology and then record costs using another.
- **CAS 402 — Consistency in Allocating Costs for the Same Purpose.** A cost type must be treated the same way across all contracts. You can't charge travel as a direct cost on one contract and an indirect cost on another.
- **CAS 405 — Accounting for Unallowable Costs.** You must identify and exclude unallowable costs from billings. (You should be doing this regardless.)
- **CAS 406 — Cost Accounting Period.** You must use your fiscal year as your cost accounting period.

**Full CAS coverage** (all 19 standards) is a significantly heavier compliance burden. The additional 15 standards address topics including:
- Home office cost allocation (CAS 403)
- Capitalization of tangible assets (CAS 404)
- Depreciation methods (CAS 409)
- Indirect cost allocation (CAS 410, 418)
- Material cost accounting (CAS 411)
- Composition of cost pools (CAS 418)
- Independent R&D and bid proposal costs (CAS 420)

### The CAS Disclosure Statement (CASB DS-1)

If you're subject to full CAS coverage, you must submit a formal **Disclosure Statement** (CASB DS-1) detailing your cost accounting practices. This is an eight-section form covering general information, direct costs, indirect costs, depreciation, capitalization, and other categories. The DS-1 is reviewed by the Cognizant Federal Agency Official (CFAO) and DCAA for adequacy and compliance.

You must update your DS-1 anytime you change your accounting practices or business operations. Failure to update—or worse, operating inconsistently with your disclosed practices—is a CAS noncompliance that can trigger cost adjustments and penalties.

### Proposed Threshold Changes

The December 2025 NDAA compromise version included proposals to increase the CAS applicability threshold to $35 million (from $7.5M), the full CAS coverage threshold to $100 million (from $50M), and to eliminate the trigger contract mechanism. If enacted, these changes would free many mid-size contractors from CAS requirements entirely. As of early 2026, these provisions are still working through the legislative process. Monitor this—it could dramatically simplify your compliance burden.

---

## DCAA: The Auditors Who Matter

The Defense Contract Audit Agency exists for one purpose: to audit defense contractors. DCAA employs approximately 4,400 auditors across offices worldwide, and they conduct tens of thousands of audits annually. If you do cost-reimbursable work for the DoD, DCAA will eventually come calling.

### How Audits Are Triggered

DCAA doesn't decide on its own to audit you. Audits are triggered by a **request from a contracting officer** or administrative contracting officer who needs audit services to make a contract decision—a pre-award evaluation, an incurred cost settlement, a rate negotiation. DCAA also uses a risk-based approach, evaluating factors like contract dollars, internal controls, and prior audit findings.

Common triggers include:
- **Contract type.** Cost-reimbursable and time-and-materials contracts generate more audit activity than firm-fixed-price.
- **Contract size.** Larger contracts attract more scrutiny.
- **Prior findings.** If DCAA found problems before, they'll look harder next time.
- **Inadequate documentation.** If your records are messy, expect a closer look.
- **Timekeeping irregularities.** DCAA is extremely focused on labor charges—more on this below.

### The Major Audit Types

**Pre-Award Accounting System Audit**

Before you can receive a cost-type contract, the contracting officer may request DCAA to evaluate whether your accounting system is adequate to handle it. This is the audit that determines whether your system can properly segregate costs, identify direct costs by contract, and produce the data the government needs. It evaluates the **design** of your system—whether your methodology is sound—not necessarily whether you've been operating it perfectly.

This is where the SF 1408 comes in (more below).

**Incurred Cost Audit**

After you submit your annual ICP, DCAA may audit it to verify that the costs you claimed were actually incurred, properly classified, and allowable. DCAA aims to complete incurred cost audits within 12 months of receiving an adequate submission, though backlogs have historically stretched timelines longer. The outcomes can include adjustments to your rates—up or down—and questioning of specific cost items.

**Provisional Billing Rate Review**

DCAA reviews your provisional billing rates to ensure they reasonably approximate your actual rates and don't include unallowable costs. This is a lighter-touch review but can result in adjustments to your billing rates mid-year.

**Floor Checks**

Floor checks are often **unannounced**. A DCAA auditor shows up at your office, walks around, and verifies that the employees who charged time to government contracts actually exist, are actually present, and are actually working on what their timesheets say they're working on. The auditor may interview employees about their timekeeping practices, ask what they're working on, and compare their answers to the charges in your system.

Floor checks are why your timekeeping system and practices must be bulletproof. If the auditor finds employees who can't explain what contract they're charging, or find discrepancies between what people say they're doing and what their timesheets show, the consequences escalate quickly.

**Other Audit Types**

DCAA also conducts forward pricing rate audits (evaluating your proposed rates for future contracts), progress payment audits, closeout audits, and business system reviews.

---

## The SF 1408: Your Accounting System's Entrance Exam

Standard Form 1408—"Pre-Award Survey of Prospective Contractor Accounting System"—is the government's checklist for evaluating whether your accounting system is adequate for cost-type contracts. If you want a cost-reimbursable or time-and-materials contract, you'll need to pass this evaluation.

The SF 1408 assesses whether your system:

1. Is maintained in accordance with Generally Accepted Accounting Principles (GAAP)
2. Provides for proper segregation of direct costs from indirect costs
3. Identifies direct costs by contract, project, or task
4. Provides for accumulation of costs under general ledger control
5. Provides an adequate timekeeping system that identifies employees' labor by contract
6. Provides for consistent treatment of costs—same cost category treated the same way across all contracts
7. Provides for exclusion of unallowable costs from billings, claims, and proposals
8. Provides for identification of costs by contract line item
9. Provides for segregation of pre-production and production costs
10. Provides adequate internal controls

The SF 1408 evaluates the **design** of your system, not its operating history. This means a company with no government contracting history can pass the SF 1408 if its system is properly designed. You don't need five years of government contract experience—you need a well-structured accounting system, a compliant chart of accounts, and documented policies and procedures.

### How to Prepare

1. **Structure your chart of accounts** with separate account numbers for direct costs (by contract), indirect cost pools (Fringe, Overhead, G&A), and unallowable costs.
2. **Implement a compliant timekeeping system** (see next section).
3. **Write policies and procedures** that describe how you identify, classify, accumulate, and allocate costs.
4. **Demonstrate your indirect rate structure** with a sample rate calculation.
5. **Show your unallowable cost identification process**—how you flag and exclude costs like entertainment and alcohol.

Many companies engage a government contracts accounting consultant to review their system before the SF 1408 evaluation. This is money well spent—failing the SF 1408 delays or kills your contract award.

---

## Timekeeping: The Most Important System You Don't Think About

If there is a single system that determines whether you survive in government contracting, it's your timekeeping system. DCAA cares more about how you track labor than almost anything else, because labor is typically the largest direct cost on government contracts—and it's the cost most susceptible to fraud and misallocation.

### The Core Requirements

Your timekeeping system must satisfy these non-negotiable requirements:

**Total time accounting.** Every hour worked by every employee must be recorded—direct labor, indirect labor, paid time off, overtime, everything. You can't just track hours charged to government contracts; you must track all hours, including time spent on commercial work, overhead activities, B&P, and administrative tasks.

**Daily time entry.** Employees must record their time daily, as work is performed. Not weekly summaries. Not reconstructed from memory on Friday afternoon. Daily, real-time entries. This is one of the most commonly violated requirements, and DCAA auditors are trained to spot patterns that suggest backdating or batch entry.

**Specific project allocation.** Each time entry must identify the specific contract, task, or project the employee worked on. "General engineering" is not acceptable. "AWS infrastructure optimization for Contract W912HQ-25-C-0038, Task Order 3" is.

**Employee certification.** Employees must personally certify that their time entries are accurate. This typically means a digital sign-off on each timesheet—the employee affirming that the hours recorded reflect actual work performed.

**Supervisor approval.** A supervisor who has direct knowledge of the work must review and approve each timesheet. This isn't a rubber stamp—the supervisor should be able to verify that the employee was actually working on the projects shown.

**Audit trail.** Any changes to timesheet entries must be logged—who changed what, when, and why. Original entries must be preserved. The system must maintain immutable records that auditors can review.

**Separation of duties.** The person who enters time should not be the same person who approves it, and neither should be the person who processes payroll from it.

**Record retention.** Timekeeping records must be retained for at least three years after final contract payment. Many consultants recommend six years as best practice.

### The Consequences of Getting It Wrong

DCAA floor checks and timekeeping audits have ended companies. If auditors find that employees can't describe what they're working on, that timesheets are routinely entered late, that supervisors approve without reviewing, or that entries have been changed without documentation, the consequences escalate from questioned costs to system inadequacy determinations to potential fraud referrals.

The simplest preventative measure: implement a compliant timekeeping system before you touch your first government dollar. Not after. Before.

---

## ERP Systems for Government Contractors

No off-the-shelf ERP system is "DCAA certified"—DCAA doesn't certify software. However, several systems are purpose-built to support DCAA compliance and are widely used across the government contracting industry:

### Deltek Costpoint

Costpoint is the industry standard for mid-size and large government contractors. It provides comprehensive project accounting, job costing, indirect cost allocation, timekeeping, and reporting capabilities designed specifically for government contract compliance. It's the system DCAA auditors are most familiar with, which can be an advantage—auditors know where to find things in Costpoint, which makes audits smoother.

**Cost:** Enterprise pricing. Typically $50,000+ for implementation plus annual licensing fees. This is a system you grow into, not start with.

### Unanet

Unanet is designed for small to mid-size government contractors and professional services firms. It provides integrated time tracking, expense management, project accounting, and reporting with DCAA-supportive features. Its interface is more modern and accessible than Costpoint's, making it popular with smaller firms.

**Cost:** Mid-range. Cloud-based pricing starting around $25–$40 per user per month, with implementation costs.

### Procas

Procas is specifically designed for small government contractors just entering the market. It provides project accounting, timekeeping, expense tracking, and indirect cost allocation in an integrated package at a lower price point than Costpoint or Unanet. Its focus on simplicity and compliance makes it a common starting point.

**Cost:** Lower entry point, typically $15–$30 per user per month.

### The Progression

Many successful government contractors follow a progression:
1. **Start with Procas or Unanet** during your first few contracts, when the business is still small and you need compliance without enterprise complexity.
2. **Migrate to Costpoint** as you grow past $20M–$50M in annual revenue and the complexity of your contract portfolio demands more sophisticated cost allocation and reporting.

Do not try to run government contracts on QuickBooks, Xero, or standard commercial accounting software. These systems lack the project-level cost tracking, indirect cost pool management, and timekeeping integration that government contracting requires. You will fail your first DCAA audit.

---

## SaaS Pricing in Government: The Advance Payment Problem

If you're a software company—which, if you're reading this textbook, you likely are—the pricing model for your product creates a specific tension with government contracting rules.

### The Conflict

In the commercial world, SaaS pricing is simple: you charge per user per month (or per year), the customer pays upfront or on a recurring basis, and access is provided for the paid period. This is how every commercial SaaS company on earth operates.

Government contracting rules historically treated any payment made before the government received the "deliverable" as an impermissible advance payment. Since a SaaS subscription grants access over time, paying for a year's subscription upfront could be interpreted as an advance payment for services not yet rendered—violating federal acquisition law.

This interpretation made SaaS procurement extraordinarily difficult. Contracting officers either refused to buy SaaS, required month-by-month payments (which created 12x the administrative overhead), or structured awkward workarounds that satisfied no one.

### GSA Acquisition Letter MV-2024-01: The Fix

In March 2024, the GSA issued Acquisition Letter MV-2024-01, which clarified that upfront payments for SaaS licenses are **not** advance payments when specific conditions are met:

1. **Contemporaneous access.** Access to the software is granted at the same time as payment.
2. **Fixed-price basis.** The license is acquired at a fixed price (per-seat, per-unit, or subscription price).
3. **Fixed term.** The price covers a defined, limited period.
4. **No consumption metrics beyond quantity.** The pricing model doesn't vary based on usage, only based on the number of seats or units.
5. **No prerequisite upfront payment.** No payment beyond the fixed subscription cost is required for access.
6. **Continuous service.** The service is uninterrupted for the negotiated term.

If your SaaS pricing meets these criteria, the government can pay for your subscription upfront without violating advance payment rules. This is a significant development for software companies selling to the government.

### How to Structure Your Pricing

For government contracts, software companies typically need to present pricing in one of several structures:

**Per-seat subscription (annual).** Price per user per year, billed annually. This is the simplest model post-MV-2024-01 and aligns with how most government budgeting works (annual appropriations).

**Enterprise license.** A flat fee for organization-wide or unit-wide access. Simpler to administer but requires careful scoping so the government doesn't pay for access it doesn't use.

**Labor + license hybrid.** Common for software that requires customization or integration. The license fee covers the software; labor hours cover implementation, training, and support. This is often structured as a time-and-materials or cost-plus contract for the labor component and a firm-fixed-price CLIN for the license.

**Consumption-based pricing.** Per-API-call, per-GB, or per-transaction pricing. This model is harder to fit into government contracting because the total cost is unpredictable, which creates budget uncertainty for the contracting officer. If you use this model commercially, consider offering a committed-use discount or a fixed monthly tier for government customers.

### What to Avoid

- **Freemium-to-paid models.** The government can't easily upgrade from free to paid mid-contract. Structure the contract at the paid tier from the start.
- **Usage-based surprises.** If your pricing can spike based on usage, government customers will resist. Offer predictable tiers.
- **Automatic renewals.** Government contracts don't auto-renew. Structure for fixed terms with option years.

---

## When CAS and DCAA Don't Apply

Not every government contract requires full cost accounting compliance. Understanding the exemptions can save you enormous overhead:

### Commercial Item Contracts

If your product is a "commercial item" (or "commercial product/commercial service" under recent FAR revisions), many FAR/DFARS clauses—including most CAS requirements—don't apply. Commercial item contracts are typically firm-fixed-price, meaning the government pays your price and doesn't audit your costs. Your internal cost structure remains your business.

To qualify, your product must be "of a type" sold commercially or offered for sale to the general public. Most commercial SaaS platforms and off-the-shelf software qualify.

### Other Transactions (OTs)

As discussed in Chapter 6, OTs are explicitly not contracts and are not subject to FAR, CAS, or DCAA audit requirements. Your pricing under an OT is negotiated freely, your accounting system doesn't need to pass SF 1408 evaluation, and DCAA has no audit authority over the agreement.

This is a critical reason why OTs are so attractive for startups. You can enter the defense market, build revenue and relationships, and demonstrate your technology—all without building a DCAA-compliant accounting system from day one.

### The Catch

If a prototype OT transitions to a follow-on production **contract** (not an OT), that contract may be subject to FAR, and if it exceeds relevant thresholds, CAS and DCAA requirements may apply. Similarly, SBIR Phase III awards structured as cost-type contracts may trigger DCAA audit authority.

The strategic move: use the OT and SBIR phase to build revenue and relationships while incrementally building your compliance infrastructure. By the time you need a DCAA-adequate system, you should have the revenue to pay for one.

---

## Building Compliance Incrementally: A Practical Roadmap

You don't need a $100,000 ERP system and a DCAA-compliant accounting infrastructure before you get your first government dollar. You need to build it in stages:

### Stage 1: Before Your First Contract ($0 investment required)

- **Understand your NAICS codes and small business size standards** (Chapter 4).
- **Register in SAM.gov** (Chapter 4).
- **Choose an accounting system** that supports project-level cost tracking, even if you start small. Many firms start with Procas or Unanet at $15–$40/user/month.

### Stage 2: First OT, SBIR, or Fixed-Price Contract ($5K–$15K)

- **Set up your chart of accounts** with separate accounts for direct costs (by contract), Fringe, Overhead, and G&A pools, and unallowable costs.
- **Implement a compliant timekeeping system** with daily entry, employee certification, supervisor approval, and audit trails.
- **Write your basic accounting policies and procedures**: how you classify costs, how you handle timekeeping, how you identify unallowable costs.
- **Start tracking indirect rates**, even if no one is auditing them yet. Understanding your real rates now prevents sticker shock later.

This stage is manageable for most startups. The accounting system costs $200–$500/month. The timekeeping system is built into the ERP. The policies take a few days to write (or hire a consultant for $5K–$10K to draft them).

### Stage 3: First Cost-Type Contract ($15K–$50K)

- **Prepare for the SF 1408 evaluation.** Have a government contracts accounting consultant review your system before the DCAA assessment.
- **Calculate and submit provisional billing rates.**
- **Establish processes for the annual Incurred Cost Proposal.**
- **Train all employees on timekeeping compliance**—not just the rules, but why they matter and what happens if they're violated.

### Stage 4: CAS Trigger ($7.5M+ contract) ($50K–$150K)

- **Ensure compliance with the four modified CAS standards** (401, 402, 405, 406).
- **Consider engaging a CAS-specialist consultant** or adding a government contracts accounting professional to your staff.
- **Review your cost allocation methodology** with a CAS expert to ensure consistency across your portfolio.

### Stage 5: Full CAS Coverage ($50M+ awards) ($100K–$300K+)

- **Prepare and submit the CAS Disclosure Statement** (CASB DS-1).
- **Migrate to an enterprise ERP** (Costpoint or equivalent) if you haven't already.
- **Hire or contract a dedicated government contracts finance team.**
- **Establish a compliance review program** for ongoing CAS and FAR adherence.

The key insight: you don't build the entire infrastructure before you start. You build it incrementally, matched to the complexity of your contract portfolio. A company with a single $200K OT doesn't need Costpoint. A company with $75M in CAS-covered contracts does. The progression is gradual, and the revenue from each stage funds the compliance investment for the next.

---

## Common Financial Mistakes

### Charging Unallowable Costs to Government Contracts

This is the single most common finding in DCAA audits. Companies charge entertainment, alcohol, or other unallowable costs to indirect pools that are then allocated to government contracts. The fix is structural: design your chart of accounts to flag unallowable costs from the start, and review every expense before it enters a billable pool.

### Inconsistent Cost Treatment

Charging a cost as direct on one contract and indirect on another violates CAS 402 and is a common finding. Develop a written policy that defines which costs are direct and which are indirect, and apply it consistently across all contracts.

### Weak Timekeeping

Late time entries, missing supervisor approvals, no audit trail for changes—these are the findings that lead to system inadequacy determinations. Implement your timekeeping system correctly from the beginning and enforce compliance relentlessly.

### Failing to Submit the ICP on Time

Missing the six-month deadline for your Incurred Cost Proposal can result in withholding of payments, adverse rate determinations, and increased audit scrutiny. Put the deadline on your calendar the day your fiscal year ends.

### Not Understanding Your Rates

Many startups enter government contracting without understanding their real indirect rates. They bid competitively low rates, win the contract, and then discover their actual costs are 40% higher than what they're being reimbursed. Calculate your real rates before you bid, and don't lowball—the government expects realistic rates, and artificially low rates can raise concerns about your ability to perform.

---

## Key Terms

**Allowable cost:** A cost that is permitted under FAR Part 31 and can be reimbursed under a government contract. Must also be allocable and reasonable.

**Unallowable cost:** A cost that may not be recovered under a government contract, regardless of its reasonableness. Includes alcohol, entertainment, lobbying, fines, most interest costs, and others.

**Allocable cost:** A cost that is chargeable to a particular contract or cost objective based on the benefit received.

**Direct cost:** A cost that can be identified specifically with a particular contract, project, or task.

**Indirect cost:** A cost incurred for common objectives that benefits more than one contract and cannot be identified to a single contract.

**Fringe rate:** Indirect rate covering employee benefits costs, allocated across all labor.

**Overhead rate:** Indirect rate covering costs that support direct project work, typically allocated on direct labor.

**G&A rate:** Indirect rate covering general management and administrative costs, typically allocated on total cost input.

**Cost pool:** An accumulation of indirect costs grouped for allocation to cost objectives via an allocation base.

**Provisional Billing Rate (PBR):** Estimated indirect rate used for interim billing during the contract performance period.

**Incurred Cost Proposal (ICP):** Annual submission reconciling provisional billing rates with actual costs. Due six months after fiscal year end.

**CAS (Cost Accounting Standards):** 19 federal standards governing cost measurement, assignment, and allocation for government contractors.

**Modified CAS coverage:** Compliance with CAS 401, 402, 405, and 406 only. Triggered by a $7.5M+ contract with total CAS-covered awards below $50M.

**Full CAS coverage:** Compliance with all 19 CAS standards. Triggered by a $50M+ single contract or $50M+ total CAS-covered awards.

**CASB DS-1 (Disclosure Statement):** Formal document detailing a contractor's cost accounting practices. Required under full CAS coverage.

**DCAA (Defense Contract Audit Agency):** Federal agency responsible for auditing defense contractors' costs, systems, and compliance.

**SF 1408:** Pre-Award Survey of Prospective Contractor Accounting System. The government's checklist for evaluating accounting system adequacy.

**Floor check:** Often unannounced DCAA audit verifying employee presence, timekeeping practices, and accuracy of labor charges.

**GAAP (Generally Accepted Accounting Principles):** The standard framework of accounting guidelines used in the United States.

**FAR Part 31:** Federal Acquisition Regulation section governing contract cost principles, including allowable vs. unallowable costs.

**FAR 52.216-7:** Allowable Cost and Payment clause. Inclusion in a contract triggers the ICP requirement.

**GSA AL MV-2024-01:** March 2024 GSA Acquisition Letter clarifying that SaaS subscription payments meeting specific conditions are not impermissible advance payments.

**Deltek Costpoint:** Enterprise ERP system for government contractors. Industry standard for mid-size and large firms.

**Unanet:** ERP system for small to mid-size government contractors.

**Procas:** Integrated project accounting system designed for small government contractors.

**Total Cost Input:** The broadest allocation base for G&A, including all direct costs, overhead, and fringe applied to direct costs.

**TINA (Truth in Negotiations Act):** Requires contractors to submit certified cost or pricing data for negotiated contracts above $2M (now called "Truthful Cost or Pricing Data").
