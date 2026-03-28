# Chapter 5: The Legal Minefield — FAR, DFARS, and the Regulatory Universe

---

Every year, a few government contractors go to prison. Not metaphorically. Actual federal prison. They go because they lied on an invoice, or they slipped a government official an envelope, or they peeked at a competitor's pricing during a source selection. The federal government spends roughly $800 billion a year buying things from the private sector, and it has built an extraordinarily detailed legal apparatus to make sure that money is spent fairly. If you want to sell to the military, you need to understand this apparatus—not because you plan to break the rules, but because you can break them accidentally, and ignorance has never been a defense.

This chapter is the one most founders skip. It's the one they assume their lawyer will handle. But here's the problem: if you don't understand the basic legal architecture of government contracting, you won't know when you're walking into danger, and you won't know when your competitors are either. The regulatory landscape isn't just a set of constraints. It's a competitive moat. Companies that understand these rules better than their rivals can move faster, bid smarter, and avoid the landmines that blow up less prepared firms. The founders who treat compliance as a strategic capability—rather than a tax—are the ones who survive.

---

## The Bible: The Federal Acquisition Regulation

The Federal Acquisition Regulation—universally called the FAR—is the single most important document in government contracting. Codified in Title 48 of the Code of Federal Regulations, the FAR is the comprehensive rulebook that governs how every executive branch agency buys products and services from the private sector. If you sell to the government, the FAR is the law that governs the sale.

The FAR is enormous and exhaustive. It is organized into 53 numbered parts, grouped into eight subchapters, and it covers everything from how agencies publicize contract opportunities to how they terminate contracts that go wrong. Understanding its structure is essential because every contract you sign will reference specific FAR clauses by number, and you need to know what you're agreeing to.

### How the FAR Is Organized

The FAR's eight subchapters cover the full lifecycle of a government contract:

**Subchapter A — General (Parts 1–4).** This sets the ground rules: what the FAR is, who writes it, how terms are defined, and what constitutes improper business practices. Part 3 is especially important because it covers standards of conduct, anti-kickback rules, and the requirement for contractor codes of ethics.

**Subchapter B — Acquisition Planning (Parts 5–12).** These parts govern how the government plans its purchases and publicizes them. Part 8 covers required sources of supply (including the GSA Schedule). Part 10 covers market research. Part 12—the acquisition of commercial products and commercial services—is the most important part for most startups, because commercial item contracting is generally simpler and involves fewer regulatory burdens than traditional government contracting. The April 2025 Executive Order on streamlining procurement mandates that agencies increasingly use simplified acquisition procedures for commercial purchases up to $7.5 million.

**Subchapter C — Contracting Methods and Types (Parts 13–18).** This covers how contracts are awarded. Part 13 covers simplified acquisition procedures (for purchases under the Simplified Acquisition Threshold, currently $250,000). Part 15 covers contracting by negotiation—the formal competitive process used for larger contracts, including how proposals are evaluated and how best-value tradeoff decisions are made. Part 16 covers contract types: firm-fixed-price, cost-reimbursement, time-and-materials, and the various hybrids. For software companies, the distinction between firm-fixed-price (you eat overruns) and time-and-materials (the government pays your hourly rate plus expenses) has enormous financial implications.

**Subchapter D — Socioeconomic Programs (Parts 19–26).** This is the legal foundation for everything covered in Chapter 4: small business set-asides, the 8(a) program, HUBZone, SDVOSB, WOSB. Part 22 covers labor laws (Davis-Bacon, Service Contract Act, Buy American). Part 25 covers foreign acquisition restrictions.

**Subchapter E — General Contracting Requirements (Parts 27–33).** This is where things get dangerous for the unprepared. Part 27 covers patents, data rights, and copyrights—a chapter unto itself that we'll cover later. Part 31 covers cost principles: which costs the government will reimburse on cost-type contracts and which it won't. Part 32 covers contract financing, including the rules about advance payments that make SaaS billing complicated. Part 33 covers protests, disputes, and appeals.

**Subchapter F — Special Categories of Contracting (Parts 34–41).** Part 39 covers the acquisition of information technology, which has its own set of rules including Section 508 accessibility requirements and compliance with the Clinger-Cohen Act.

**Subchapter G — Contract Management (Parts 42–51).** Part 42 covers contract administration, including how the government assigns administrative contracting officers and quality assurance representatives to monitor your performance. Part 49 covers termination—both termination for convenience (the government can cancel your contract at any time for any reason, paying you for work completed) and termination for default (you failed to perform).

**Subchapter H — Clauses and Forms (Parts 52–53).** Part 52 is the clause library. Every contract clause the government uses lives in Part 52, organized by the FAR part it corresponds to. When your contract says "FAR 52.227-14 is incorporated by reference," that clause has a specific meaning, and you are legally bound by every word in it whether you've read it or not.

### Why the FAR Matters to You

The most important thing to understand about the FAR is that it creates a fundamentally different legal environment than commercial contracting. In a commercial transaction, the terms of the deal are whatever two private parties agree to. In government contracting, the terms are dictated by statute and regulation. The FAR isn't a suggestion. Many of its provisions derive from federal law, and violating them can trigger civil penalties, criminal prosecution, contract termination, and debarment from future government work.

The April 2025 Executive Order "Restoring Common Sense to Federal Procurement" directed the FAR Council to streamline the regulation by mid-October 2025, removing clauses not required by statute. This is a significant development: non-statutory clauses may be retained but with a four-year sunset provision requiring active renewal. If carried through, this could simplify the FAR meaningfully for commercial-item contractors. But as of this writing, the core statutory provisions—competition requirements, cost principles, anti-fraud rules—remain firmly in place.

---

## The Supplement: DFARS

If the FAR is the bible, the Defense Federal Acquisition Regulation Supplement—DFARS—is the book of additional commandments that apply only to Department of Defense contracts. DFARS doesn't replace the FAR; it layers on top of it. When you sign a DoD contract, both FAR and DFARS clauses apply.

DFARS exists because defense contracting has concerns that civilian agency contracting doesn't. National security. Classified information. The protection of military technology. Supply chain integrity. Cybersecurity. Foreign ownership restrictions. DFARS addresses all of these with additional rules that are, almost without exception, more stringent than the base FAR.

The DFARS clauses that appear most frequently in software contracts include:

**DFARS 252.204-7012 — Safeguarding Covered Defense Information and Cyber Incident Reporting.** This is the single most consequential DFARS clause for software companies. It requires every DoD contractor and subcontractor that processes, stores, or transmits Covered Defense Information (CDI) on non-federal systems to implement the 110 security controls specified in NIST Special Publication 800-171. It also requires reporting cyber incidents to the DoD's DIBNet portal within 72 hours of discovery, preserving forensic images for at least 90 days, and submitting any malicious software to the DoD Cyber Crime Center (DC3). If you use cloud services, they must meet security requirements equivalent to FedRAMP Moderate baseline. This clause appears in virtually every DoD software contract, and compliance is not optional—it is a condition of receiving the contract. The upcoming CMMC program (detailed in Chapter 8) is the enforcement mechanism that verifies compliance with the NIST 800-171 controls referenced by this clause.

**DFARS 252.204-7018 through 7021 — CMMC Requirements.** The Cybersecurity Maturity Model Certification clauses, with the final rule (DFARS 252.204-7021) published September 10, 2025, phase in over three years beginning November 10, 2025. These require contractors to hold an appropriate CMMC certification level as a condition of contract award, with third-party assessors verifying compliance for Level 2 and above.

**DFARS 252.227-7013 and 7014 — Technical Data and Computer Software Rights.** These clauses govern how the government obtains rights to use your software and technical data, and they differ significantly from commercial licensing norms. We'll cover them in detail in Chapter 10.

**DFARS 252.239-7010 — Cloud Computing Services.** This clause imposes specific requirements on cloud service providers supporting DoD, including data storage location requirements and incident response obligations.

### The Flow-Down Problem

A critical feature of both FAR and DFARS is the concept of flow-down clauses. Many contract clauses must be "flowed down" to subcontractors—meaning that if you're a subcontractor on a DoD prime contract, the prime is required to include these clauses in your subcontract, and you are required to comply with them. DFARS 252.204-7012, for example, flows down to all subcontractors whose subcontract performance involves covered defense information. This means you can inherit significant compliance obligations even when your contractual relationship is with a prime contractor rather than directly with the government.

---

## The Laws That Send People to Prison

The FAR and DFARS are administrative regulations. But underneath them sits a layer of federal criminal statutes that carry real prison sentences. These are the laws that every government contractor—especially every government contractor CEO—needs to understand cold.

### The False Claims Act

The False Claims Act (31 U.S.C. § 3729–3733) is the federal government's primary weapon against contractor fraud, and it is used aggressively. Originally enacted during the Civil War to combat suppliers selling defective equipment to the Union Army, it has evolved into one of the most powerful fraud-fighting tools in American law.

The FCA makes it illegal to knowingly submit a false or fraudulent claim for payment to the government. "Knowingly" is defined broadly: it includes actual knowledge, deliberate ignorance, and reckless disregard of truth. You don't have to intend to defraud the government; you just have to submit a claim you should have known was false.

The penalties are severe:

- **Treble damages:** The government recovers three times its actual losses.
- **Per-claim civil penalties:** In FY2024, these ranged from $13,946 to $27,894 per false claim submitted—and on a large contract with monthly invoices, the per-claim penalties alone can reach tens of millions.
- **Criminal liability:** While primarily a civil statute, FCA violations can lead to criminal charges, particularly when fraud is systematic.

In fiscal year 2025, the DOJ recovered over $6.8 billion under the False Claims Act—more than double the recovery of any of the preceding three years. The majority of these cases originated from whistleblowers, called "qui tam relators," who can file lawsuits on behalf of the government and receive 15–30% of the recovery. Your disgruntled former employee isn't just a personnel problem; they're a potential plaintiff with a financial incentive to report every billing irregularity they've ever witnessed.

**What this means for software companies:** If you certify that your software meets a contractual requirement and it doesn't, that's potentially a false claim. If you bill for developer hours that weren't actually worked, that's a false claim. If you certify compliance with NIST 800-171 when you haven't implemented the required controls, that certification—which is a condition for getting paid—could be a false claim. The FCA doesn't require that you intended to steal; it requires only that you submitted a claim you should have known was inaccurate.

### The Anti-Kickback Act

The Anti-Kickback Act of 1986 (41 U.S.C. § 8702) prohibits anyone involved in a government contract from offering, soliciting, or receiving kickbacks in connection with the award of subcontracts. A "kickback" is any money, fee, commission, gift, or thing of value provided to improperly obtain or reward favorable treatment.

The penalties include criminal fines, imprisonment, contract termination, and debarment. The act applies to prime contractors, subcontractors, and their employees at all levels.

**What this means for software companies:** The most common kickback scenario for startups involves teaming arrangements. If you agree to give a prime contractor a percentage of your subcontract value as a "teaming fee" or "business development fee" that isn't tied to actual services they're providing, that can be a kickback. If a government official steers a contract your way and you hire their spouse as a "consultant," that can be a kickback. The lines here are brighter than most founders expect, and crossing them is a felony.

### The Procurement Integrity Act

The Procurement Integrity Act (41 U.S.C. § 2101–2107) protects the integrity of the competitive procurement process by prohibiting two categories of conduct:

1. **Disclosure of source selection information.** No one may disclose or obtain contractor bid or proposal information or source selection information before a contract is awarded. If a government evaluator tells you what your competitor bid, and you use that information, both of you may have committed a federal crime.

2. **Post-employment restrictions.** Government officials who serve in certain decision-making roles on contracts exceeding $10 million are barred for one year from receiving compensation from the contractor that won the award.

Violations carry criminal penalties of up to five years imprisonment and civil penalties including contract cancellation, profit disgorgement, suspension, and debarment.

**What this means for software companies:** If you hire a former government program manager who recently oversaw your contract, you need to verify they've completed their cooling-off period. If a government contact shares competitive information with you during a procurement, you need to report it to the contracting officer—not use it. Failing to self-report these situations dramatically increases your legal exposure.

### The Truth in Negotiations Act (TINA)

Originally called the Truth in Negotiations Act and now formally referred to under FAR Part 15.4 as "Truthful Cost or Pricing Data," this law (10 U.S.C. § 3702) requires contractors to submit cost or pricing data that is current, accurate, and complete when negotiating certain contracts and modifications. The threshold—currently $2 million—triggers the requirement.

If you submit cost or pricing data and it turns out to be inaccurate, the government has what's called a "defective pricing" claim: it can reduce the contract price by the amount of the overpricing, plus interest. And if the inaccuracy was intentional, the False Claims Act comes into play.

**What this means for software companies:** Most commercial-item contracts are exempt from TINA because commercial pricing is based on market forces rather than cost data. This is one of many reasons why maintaining your commercial sales channel and establishing commercial pricing is strategically important—it keeps you in the commercial-item lane and out of the cost-data-submission lane. But if you do a sole-source, cost-type contract above the threshold, you enter TINA territory, and your internal cost records become discoverable.

---

## Software Licensing and SaaS: Where Old Rules Meet New Business Models

The FAR was written in an era when the government bought things—physical things with physical delivery dates. Software, especially software-as-a-service, doesn't fit neatly into this framework, and the resulting friction creates traps for startups that don't understand the disconnect.

### The Advance Payment Problem

Federal law (31 U.S.C. § 3324) generally prohibits agencies from making advance payments—paying for goods or services before they're delivered. This creates an immediate conflict with the standard SaaS business model, where customers pay an annual subscription fee upfront in exchange for continuous access to the software.

For years, this prohibition created genuine confusion. Some contracting officers refused to approve annual subscription payments, insisting on monthly billing tied to service delivery. Others treated SaaS licenses as products (delivered upon activation) rather than services (delivered over time), which resolved the problem differently.

The GSA settled the question with Acquisition Letter MV-2024-01, issued March 15, 2024. The guidance clarifies that an upfront payment for SaaS is **not** an advance payment—and is therefore permissible—if the following conditions are met:

1. Access to the software is granted contemporaneously with the payment (delivery occurs at the time of payment).
2. The license is acquired on a fixed-price basis or a fixed-price with economic price adjustment.
3. The license is priced for a defined, limited period (annual subscription, per-seat pricing).
4. Pricing depends only on quantity, not on utilization or consumption metrics.
5. No upfront payment beyond the fixed subscription cost is required as a prerequisite for access.
6. The service is continuous and uninterrupted for the negotiated term.

This guidance is enormously important for SaaS companies because it aligns federal procurement rules with standard commercial practice. But it's still guidance, not regulation—individual contracting officers may still need to be educated about it. Having a copy of AL MV-2024-01 ready to share with a skeptical contracting officer can save months of negotiation.

### Structuring Software Licenses for Government Customers

Beyond the advance payment question, several other FAR-driven issues arise when licensing software to the government:

**Government purpose rights vs. unlimited rights.** The government's default position is that it should get broad rights to use, modify, and redistribute software it pays for. Your default position as a software company is that you want to license your software, not give it away. DFARS 252.227-7014 distinguishes between software developed exclusively with government funds (government gets unlimited rights), software developed exclusively at private expense (government gets restricted rights), and software developed with mixed funding (government gets government purpose rights). Understanding this framework before you sign a contract is essential.

**Section 508 accessibility.** If your software will be used by federal employees or members of the public, it must comply with Section 508 of the Rehabilitation Act, which requires electronic and information technology to be accessible to people with disabilities. Section 508 compliance is often an evaluation factor in competitive procurements, and non-compliance is a legitimate basis for a bid protest.

**FedRAMP for cloud services.** If your SaaS product is hosted in the cloud (which it almost certainly is), the government will want to know your FedRAMP authorization status. FedRAMP Moderate authorization is the baseline for most unclassified government workloads. FedRAMP High is required for sensitive but unclassified data. Each authorization level requires implementing and documenting hundreds of specific security controls.

---

## Bid Protests: The Weapons Your Competitors Will Use Against You

A bid protest is a challenge to the award—or proposed award—of a government contract. It's the mechanism by which a losing bidder (or a company that believes the solicitation was unfairly structured) can contest the government's decision. Bid protests are simultaneously a safeguard for fair competition and a source of program delay that drives program managers to distraction.

### Where Protests Are Filed

There are three venues for bid protests:

**The Government Accountability Office (GAO).** The GAO is, by far, the most common venue for bid protests. It handles roughly 1,600–1,800 protests per year. GAO protests are relatively fast (100-day decision timeline), relatively inexpensive (you don't technically need a lawyer, though you absolutely should have one), and carry an automatic stay of contract performance—meaning the government generally cannot proceed with the contested contract while the protest is pending.

**The U.S. Court of Federal Claims (COFC).** The COFC is a federal court with jurisdiction over contract disputes and bid protests. Unlike GAO protests, COFC protests do not carry an automatic stay (protesters must request injunctive relief). The process is more formal, follows federal rules of procedure, involves discovery, and takes longer. But COFC decisions are legally binding, whereas GAO recommendations technically are not (though agencies almost always follow them). A party unhappy with a GAO decision can re-file at the COFC for a fresh judicial review.

**The Agency Itself.** Protesters can also file agency-level protests with the contracting officer. These are less common because they lack the independence and enforcement mechanisms of GAO and COFC proceedings.

### Protest Statistics

The numbers tell an important story about the protest system. In FY2025:

- **1,688 new cases** were filed at the GAO (1,617 of which were bid protests), a 6% decrease from FY2024.
- The GAO **sustained** 14% of protests decided on the merits—meaning it found the agency violated procurement law in a way that prejudiced the protester.
- The **effectiveness rate** was 52%—meaning that in more than half of all protests, the protester received some form of relief, either through a GAO sustainment or through the agency voluntarily taking corrective action before a decision was issued.

That 52% effectiveness rate should command your attention. It means that when a losing bidder files a protest, there is a better-than-even chance that the award will be disrupted. The most common grounds for sustained protests are unreasonable technical evaluations, unreasonable cost/price evaluations, and unreasonable rejection of proposals.

### What Protests Mean for You

Bid protests affect you in two directions:

**You might protest.** If you believe your technically superior proposal was unfairly evaluated, or that the government failed to follow its own evaluation criteria, a GAO protest is a relatively low-cost way to challenge the decision. The 100-day timeline and automatic stay create meaningful leverage.

**You might be protested.** If you win a contract, your competitor may protest the award. This can delay your start of work by months. If the GAO sustains the protest, the agency may be directed to reevaluate proposals, and you could lose the contract you thought you'd won. This is worth understanding not because it should discourage you from competing, but because you should build protest risk into your timeline and financial projections.

---

## Contract Disputes: When Things Go Wrong After Award

Bid protests happen before work begins. Contract disputes happen during or after performance, when the contractor and the government disagree about what the contract requires, how much the government owes, or whether performance has been adequate.

### The Contract Disputes Act

The Contract Disputes Act of 1978 (CDA), codified at 41 U.S.C. § 7101–7109, is the statutory framework for resolving contract disputes between contractors and the federal government.

The process begins with a **claim**—a written demand from the contractor to the contracting officer (or from the government to the contractor) for money or other relief. Claims must be submitted within six years of when the claim accrues. For claims exceeding $100,000, the contractor must certify that the claim is made in good faith, the supporting data is accurate, and the amount requested accurately reflects the contract adjustment for which the contractor believes the government is liable.

The contracting officer must issue a **Contracting Officer's Final Decision (COFD)** within 60 days for claims of $100,000 or less. For larger claims, the contracting officer has 60 days to either decide or notify the contractor when a decision will be issued. If the contracting officer fails to act within 60 days, the contractor can treat the claim as "deemed denied" and proceed to appeal.

### Appeal Paths

If you're unhappy with the COFD, you have two choices:

**Board of Contract Appeals.** You must appeal within 90 days of receiving the COFD. For DoD contracts, this goes to the Armed Services Board of Contract Appeals (ASBCA). For civilian agency contracts, it goes to the Civilian Board of Contract Appeals (CBCA). Both boards hear cases de novo—meaning they review the entire factual record fresh, not just whether the contracting officer was reasonable. Both boards also encourage alternative dispute resolution as a faster path to settlement.

**U.S. Court of Federal Claims.** You must file within 12 months of receiving the COFD. This is a full judicial proceeding with all the formality, cost, and timeline that implies.

**What this means for software companies:** The most common disputes in software contracts involve scope creep (the government wants functionality beyond what the contract specified and refuses to issue a modification), acceptance testing (the government won't accept software deliverables, claiming they don't meet specifications), and payment delays (the government withholds payment pending resolution of a performance disagreement). Understanding the CDA process means you know that your recourse begins with a formal claim to the contracting officer—not a phone call to your program manager—and that you have specific deadlines for escalating it.

---

## Allowable Costs, Unallowable Costs, and the CAS Trap

If you do any cost-reimbursement or time-and-materials work, you need to understand FAR Part 31—the cost principles that determine which of your costs the government will pay for and which it won't. This is one of the most technically dense areas of government contracting, and mistakes here trigger False Claims Act liability.

### The Basic Framework

Under a cost-reimbursement contract, the government pays your actual, allowable, allocable, and reasonable costs, plus a fee. Each of those four adjectives has a specific legal meaning:

- **Actual:** You actually incurred the cost.
- **Allowable:** The cost is not on the government's list of unallowable expenses (FAR 31.205 has 52 specific cost categories, each with its own allowability rules).
- **Allocable:** The cost is properly chargeable to the contract (not to a different contract or to your commercial business).
- **Reasonable:** The cost is the kind a prudent business person would incur in a competitive environment.

### Costs That Are Always Unallowable

Some costs can never be charged to a government contract, regardless of how reasonable they seem:

- **Alcohol:** No exceptions, even if you're entertaining a customer.
- **Entertainment:** Tickets, club memberships, sporting events.
- **Fines and penalties:** If you get fined by a regulator, you can't pass it to the government.
- **Lobbying costs:** Money spent trying to influence legislation or contract awards.
- **Bad debts:** Losses on other contracts or from other customers.
- **Donations and contributions:** Charitable giving is unallowable.
- **First-class travel:** The government pays for coach.
- **Interest on borrowings:** With narrow exceptions for facilities capital.
- **Organizational restructuring costs:** M&A transaction fees, golden parachutes.

### Why This Matters Even if You're Fixed-Price

You might think: "I'm a software company. I'll do fixed-price contracts. Cost allowability doesn't apply to me." And you'd be mostly right—on a firm-fixed-price contract, the government pays the agreed price and doesn't audit your internal costs. But there are two catches:

First, if you ever do a time-and-materials contract (common for software development and integration work), your billing rates must be supported by your actual cost structure, and unallowable costs cannot be included in those rates. If DCAA (Defense Contract Audit Agency) audits your labor rates and finds that your overhead pool includes entertainment expenses or lobbying costs, they'll reduce your rates retroactively—and if you've already been paid at the higher rate, you owe the money back.

Second, even on fixed-price contracts, your proposed pricing must be "fair and reasonable." If a contracting officer asks for cost backup to establish reasonableness and your cost data includes unallowable items, you've got a defective pricing problem.

### Cost Accounting Standards (CAS)

If your government contracts exceed $50 million in a single fiscal year, you become subject to full Cost Accounting Standards (CAS), which impose rigorous requirements on how you accumulate and report costs. Most startups won't hit this threshold early, but it's worth knowing it exists because it fundamentally changes your accounting requirements—and many companies that cross it discover, painfully, that their accounting systems need to be rebuilt from scratch.

---

## The Secure Software Development Framework: The Newest Compliance Layer

Executive Order 14028, "Improving the Nation's Cybersecurity," signed May 12, 2021, fundamentally changed what the government expects from software producers. In response to catastrophic supply chain attacks like SolarWinds, the EO directed NIST to develop secure software development standards—resulting in the Secure Software Development Framework (SSDF), published as NIST Special Publication 800-218.

### What the SSDF Requires

The SSDF isn't a single checklist; it's a set of practices organized into four groups:

1. **Prepare the Organization (PO):** Ensure your people, processes, and technology are prepared to perform secure software development.
2. **Protect the Software (PS):** Protect all components of your software from tampering and unauthorized access.
3. **Produce Well-Secured Software (PW):** Produce software with minimal security vulnerabilities in its releases.
4. **Respond to Vulnerabilities (RV):** Identify residual vulnerabilities in your software releases and respond appropriately.

### The Attestation Requirement

On March 18, 2024, the Cybersecurity and Infrastructure Security Agency (CISA) released a standardized attestation form that software producers selling to the federal government must complete. This is a self-attestation—you're certifying that your development practices align with the SSDF. But because it's a certification submitted to the government, a false attestation could implicate the False Claims Act. The attestation isn't aspirational; it's a legal statement.

NIST released an initial public draft of SSDF Version 1.2 (SP 800-218r1) on December 17, 2025, introducing new practices and refining existing ones. The framework is evolving, and staying current is part of the obligation.

### What This Means for Software Companies

The SSDF attestation requirement means that your software development practices are now a compliance issue, not just an engineering preference. You need documented evidence that you:

- Maintain a secure development environment
- Conduct regular code reviews and security testing
- Manage your software supply chain (third-party libraries, open-source components)
- Have a vulnerability disclosure and response process
- Generate and maintain a Software Bill of Materials (SBOM)

If you're building software for DoD, these requirements interact with and reinforce the CMMC requirements discussed in Chapter 8 and the DFARS 252.204-7012 obligations discussed earlier in this chapter. The government is building a layered compliance architecture, and each layer references the others.

---

## Ethics, Compliance, and the Mandatory Disclosure Rule

### FAR 52.203-13: Contractor Code of Business Ethics and Conduct

Any federal contract exceeding $6 million with a performance period over 120 days triggers FAR 52.203-13, which requires the contractor to:

1. **Establish a written code of business ethics and conduct** and make it available to every employee engaged in contract performance.
2. **Implement an ethics awareness and compliance program** within 90 days of contract award, including training for principals, employees, and relevant agents and subcontractors.
3. **Create an internal control system** that includes:
   - A senior-level officer responsible for ethics oversight
   - Periodic reviews of business practices for compliance
   - A confidential reporting mechanism for employees to report suspected violations
   - Internal or external auditing, as appropriate
   - Disciplinary action for violations

### The Mandatory Disclosure Rule

The most consequential part of FAR 52.203-13 is the mandatory disclosure requirement. You are required to disclose, in writing, to the agency's Office of Inspector General (with a copy to the contracting officer), credible evidence of:

- Violations of federal criminal law involving fraud, conflict of interest, bribery, or gratuity violations (Title 18, U.S. Code)
- Violations of the civil False Claims Act
- Significant overpayments on the contract

This disclosure obligation has no small business exemption and no commercial-item exemption. If your contract exceeds $5 million and has a performance period over 120 days, you must disclose—and knowing failure to do so is itself a basis for suspension or debarment.

The mandatory disclosure rule creates a real tension for contractors: you're essentially required to turn yourself in. But the DOJ's Corporate Enforcement and Voluntary Self-Disclosure Policy offers meaningful incentives—including potentially reduced penalties—for companies that proactively disclose, cooperate with investigations, and remediate misconduct. The companies that get destroyed are the ones that hide problems; the ones that survive are usually the ones that disclose early, fix fast, and demonstrate good faith.

### Suspension and Debarment

The government maintains a list—the System for Award Management (SAM) exclusions list—of companies and individuals who are prohibited from receiving government contracts. Being placed on this list is called "debarment" (a formal, usually three-year prohibition) or "suspension" (an interim measure pending investigation).

You can be suspended or debarred for:

- Commission of fraud or a criminal offense in connection with a government contract
- Violation of federal antitrust law
- Commission of embezzlement, theft, forgery, bribery, or making false statements
- Knowing failure to disclose as required under the mandatory disclosure rule
- Any other cause so serious as to affect your present responsibility as a contractor

Debarment is functionally a death sentence for a government contractor. It's worth noting that debarment isn't limited to the company—individual executives can be debarred personally, which follows them even if they leave the company and join a new one.

---

## Organizational Conflicts of Interest: The Invisible Disqualifier

Organizational conflicts of interest (OCI) are one of the most underappreciated risks in government contracting, particularly for software and advisory companies. FAR Subpart 9.5 defines an OCI as a situation where a contractor's existing work creates an unfair competitive advantage or impairs its objectivity in performing another contract.

There are three types of OCI:

1. **Unequal access to information.** You performed a contract that gave you access to non-public information that could provide an advantage in a later competition. Example: You helped write the requirements for a system, and now you have insight into evaluation criteria that other bidders don't.

2. **Biased ground rules.** You helped set the rules for a competition in which you're also competing. Example: You wrote the statement of work for a software system, and now you're bidding on the contract to build it.

3. **Impaired objectivity.** You're evaluating or recommending solutions in an area where you have a financial interest. Example: You're advising the government on which software platform to adopt while also selling a competing platform.

The consequence of an OCI finding is disqualification—you can be eliminated from a competition or have a contract terminated. What makes OCIs dangerous for startups is that they're often invisible until a competitor raises them in a protest. If you've been doing advisory work for a program office, think carefully before competing on the production contract that flows from your advice.

---

## The Regulatory Landscape as a Competitive Moat

Most founders see the legal and regulatory framework of government contracting as a burden—a cost of doing business that has to be endured. The smart ones see it differently.

Every FAR clause, every DFARS requirement, every compliance obligation is a barrier to entry. Every time a competitor skips the SBOM, ignores the SSDF attestation, or fails to implement NIST 800-171, they're creating a vulnerability that will eventually catch up with them—in an audit, in a bid protest, or in a False Claims Act investigation.

If you build compliance into your company from day one—if your accounting system is CAS-ready before you need it, if your development practices are SSDF-compliant before the attestation is required, if your cybersecurity posture meets CMMC Level 2 before the assessment is scheduled—you can move faster than competitors who are scrambling to retrofit compliance onto systems that weren't designed for it.

The companies that consistently win in government contracting are not necessarily the ones with the best technology. They're the ones that combine good technology with bulletproof compliance. They're the ones whose proposals check every box, whose past performance files are clean, and whose legal exposure is managed rather than ignored.

Here's a framework for thinking about regulatory knowledge as a competitive asset:

**Speed.** If you already understand the FAR clauses that apply to your contract type, you can respond to solicitations faster. You don't waste weeks trying to figure out what "FAR 52.212-4(c)" means.

**Accuracy.** If you understand cost principles, you can price your proposals correctly the first time. You won't underbid because you forgot about unallowable costs, and you won't overbid because you included excessive contingencies for regulatory risk you don't understand.

**Risk management.** If you understand bid protest mechanics, you can evaluate whether to protest a loss—or whether your own win is likely to survive one. You can structure your teaming arrangements to avoid kickback issues. You can manage your OCI exposure proactively rather than discovering it when a competitor challenges your award.

**Reputation.** Government contracting is a reputation business. Contracting officers talk to each other. Program managers remember which companies deliver clean, compliant proposals and which ones create administrative nightmares. A reputation for compliance is a business development asset that compounds over time.

---

## Building Compliant Systems from Day One

You don't need to be a lawyer to sell to the government. But you do need a lawyer who knows government contracting—not a generalist who's "pretty sure" they can figure it out, but a specialist who works with FAR and DFARS every day. Government contracting law is a specialty, like patent law or securities law, and the consequences of getting it wrong are too severe to wing it.

Here's what to build before your first contract:

### Compliance Checklist

**Immediate (before your first proposal):**

1. Retain a government contracts attorney (not a general business attorney).
2. Establish a code of business ethics and conduct.
3. Set up an accounting system that can track costs by contract and distinguish allowable from unallowable expenses. Your startup QuickBooks setup won't be adequate; you need contract-level cost segregation at minimum.
4. Create a document retention policy (most government contract records must be kept for three years after final payment, some longer).
5. Implement the basics of NIST 800-171 if you'll handle CUI.

**Before your first award:**

6. Establish a mandatory disclosure process—know who in your company is responsible for identifying and reporting potential violations.
7. Designate a senior ethics officer (this can be the CEO in a small company).
8. Complete the SSDF self-assessment and prepare your attestation.
9. Brief all employees on anti-kickback rules, procurement integrity, and conflict-of-interest policies.

**Within 90 days of your first award:**

10. Formalize your ethics compliance program as required by FAR 52.203-13.
11. Implement a confidential hotline or reporting mechanism for employees.
12. Conduct initial compliance training for all employees involved in contract performance.
13. Schedule your first internal compliance review.

**Ongoing:**

14. Conduct annual compliance training.
15. Perform periodic internal audits of billing, cost accounting, and cybersecurity controls.
16. Update your compliance program when regulations change (which they do, frequently).
17. Monitor the Federal Register and acquisition.gov for regulatory updates.

---

## Key Terms

**FAR (Federal Acquisition Regulation):** The primary regulation governing all federal agency acquisitions. Organized into 53 parts across 8 subchapters.

**DFARS (Defense Federal Acquisition Regulation Supplement):** DoD-specific regulations layered on top of FAR for defense contracts.

**Flow-down clause:** A contract clause that must be incorporated into subcontracts, binding subcontractors to the same requirements.

**False Claims Act (FCA):** Federal statute imposing treble damages and civil penalties for submitting false claims for payment to the government. Primary enforcement tool for government contract fraud.

**Anti-Kickback Act:** Federal statute criminalizing paying or receiving kickbacks in connection with government contracts.

**Procurement Integrity Act:** Federal statute prohibiting the disclosure of source selection information and imposing post-employment restrictions on government officials.

**Truth in Negotiations Act (TINA):** Requires submission of current, accurate, and complete cost or pricing data for certain contract negotiations above the $2 million threshold.

**Bid protest:** A formal challenge to a government contract award, filed at GAO, COFC, or the agency.

**GAO (Government Accountability Office):** Primary venue for bid protests; issues recommendations within 100 days.

**COFC (Court of Federal Claims):** Federal court with jurisdiction over bid protests and contract disputes; decisions are legally binding.

**Contract Disputes Act (CDA):** Statutory framework for resolving disputes between contractors and the government, beginning with a contracting officer's final decision and proceeding through appeal.

**ASBCA (Armed Services Board of Contract Appeals):** Adjudicates contract disputes for DoD contracts.

**CBCA (Civilian Board of Contract Appeals):** Adjudicates contract disputes for civilian agency contracts.

**COFD (Contracting Officer's Final Decision):** The government's formal response to a contractor claim; triggers appeal rights and deadlines.

**DCAA (Defense Contract Audit Agency):** The agency that audits defense contractors' accounting systems, billing rates, and cost claims.

**OCI (Organizational Conflict of Interest):** A situation where a contractor's work creates an unfair competitive advantage or impairs objectivity.

**Debarment:** Formal exclusion from government contracting, typically lasting three years.

**Suspension:** Interim exclusion from government contracting while an investigation is pending.

**Mandatory disclosure:** Legal obligation to report credible evidence of fraud, criminal conduct, or FCA violations to the agency Inspector General.

**CAS (Cost Accounting Standards):** Requirements for consistent cost measurement and allocation, applicable when government contract revenue exceeds $50 million annually.

**SSDF (Secure Software Development Framework):** NIST SP 800-218; a set of secure development practices that federal software producers must attest to.

**SBOM (Software Bill of Materials):** A comprehensive inventory of software components, dependencies, and their sources.

**FedRAMP (Federal Risk and Authorization Management Program):** Standardized security assessment and authorization framework for cloud services used by federal agencies.

**Section 508:** Requires federal technology to be accessible to people with disabilities.

**Qui tam:** A provision of the False Claims Act allowing private individuals to file lawsuits on behalf of the government and receive a share of recovered funds.
