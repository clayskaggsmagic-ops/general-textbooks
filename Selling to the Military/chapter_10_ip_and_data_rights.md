# Chapter 10: Intellectual Property and Data Rights — Protecting Your Technology from the Government

---

Here is the single most important sentence in this book for a software startup founder:

**If you don't understand data rights before you sign your first government contract, you may give away the technology your entire company is built on—and there is no getting it back.**

IP in government contracting is nothing like commercial licensing. In the commercial world, you sell a product or service. You retain your IP. Your customer gets a license to use your software. End of story. In the government world, the rules are fundamentally different. The government doesn't just want to *use* your software—depending on how the contract is structured, it can demand the right to use, modify, reproduce, and give your source code to your competitors. And if you don't understand the rules, you'll hand over those rights without even realizing what you've done.

This chapter will walk you through the entire IP landscape of government contracting: what rights the government gets, what you can protect, what determines who owns what, and how to structure your company, your development process, and your contracts to keep your core technology in your hands.

---

## Why Government IP Is Fundamentally Different

In a commercial sale, you give your customer a license. That license is defined by your terms—your EULA, your SaaS agreement, your enterprise contract. You wrote those terms. You control them.

In a government contract, the government writes the rules. The license rights the government obtains in your technical data and software are defined by statute and regulation—primarily the **Defense Federal Acquisition Regulation Supplement (DFARS)**. These rules dictate the government's rights based on one overriding factor: **who paid for the development.**

This is the core principle. Everything else flows from it:

- If the **government paid** for the development → the government gets **Unlimited Rights**
- If **you paid** for the development → you retain **Limited Rights** (for data) or **Restricted Rights** (for software)
- If **both** of you paid → the government gets **Government Purpose Rights** (a middle ground)

The concept is simple. The execution is treacherous.

---

## The Spectrum of Data Rights

The DFARS defines a spectrum of license rights that the government can obtain. Understanding this spectrum is essential—it determines what the government can do with every piece of data and software you deliver.

### For Technical Data (DFARS 252.227-7013)

"Technical data" means recorded information of a scientific or technical nature—specifications, engineering drawings, processes, manuals, reports. It explicitly excludes computer software, which has its own clause.

| Rights Level | Who Funded Development | What the Government Can Do |
|---|---|---|
| **Unlimited Rights** | Government exclusively | Use, modify, reproduce, release, perform, display, or disclose the data **in any manner and for any purpose whatsoever.** This includes giving your data to your competitors for reprocurement. |
| **Government Purpose Rights (GPR)** | Mixed funding (government + private) | Use, modify, reproduce within the government **without restriction.** Release outside the government **for government purposes only**—not for commercial purposes. **GPR lasts 5 years, then automatically becomes Unlimited Rights.** |
| **Limited Rights** | Private expense exclusively | Government can use and duplicate the data **only within the government.** Cannot release outside the government without your permission, except to government support contractors under NDA, or in emergencies. **This is your strongest protection for technical data.** |

### For Noncommercial Computer Software (DFARS 252.227-7014)

"Computer software" means programs, source code, object code, algorithms, design details, and related material that would enable the software to be reproduced or recompiled.

| Rights Level | Who Funded Development | What the Government Can Do |
|---|---|---|
| **Unlimited Rights** | Government exclusively | Same as technical data—use, modify, reproduce, release for **any purpose.** The government can give your source code to anyone. |
| **Government Purpose Rights (GPR)** | Mixed funding | Same as GPR for technical data. Use within government without restriction; release outside government for government purposes only. **5-year clock, then Unlimited.** |
| **Restricted Rights** | Private expense exclusively | Government can use the software on **one computer at a time**, make **backup copies only**, modify for its own use or allow its contractors to maintain it. **Cannot distribute, disclose, or use for reprocurement.** This is your strongest protection for software. |

### The Critical Distinction: "Noncommercial" vs. "Commercial" Software

This is one of the most strategically important designations in all of government contracting.

**If your software is classified as "noncommercial,"** DFARS 252.227-7014 applies, and the government's rights are determined by the funding source as described above. Even if you developed the software entirely at private expense, the government still gets "Restricted Rights"—which, while protective, are defined by the regulation, not by you.

**If your software is classified as "commercial,"** an entirely different clause applies: **DFARS 252.227-7202.** Under this clause, the government acquires only the rights **customarily provided to the public** under your standard commercial license. In other words, your commercial license terms govern—not the DFARS.

This is a massive difference. Under your commercial license, you might give the government the right to use your SaaS product with 500 seats for $X per month. Under DFARS 7014 for noncommercial software, the government might get the right to your source code.

**Proving your software is "commercial" is one of the single most important strategic decisions you can make.**

### How to Establish "Commercial" Status

Software qualifies as "commercial" under the FAR and DFARS if it:

1. Has been **sold, leased, or licensed to the public**, or
2. Has been **offered for sale, lease, or license to the public**, or
3. Has been developed and is **available for commercial use** in time to meet contract delivery requirements, or
4. Requires only **minor modifications** to meet contract requirements

The key strategies:

- **Sell commercially first.** Before you ever engage with the government, establish a commercial customer base (even a small one). This creates the evidentiary record that your software is a commercial product.

- **Maintain a public offering.** Keep your software available for commercial purchase—a pricing page, commercial contracts, public documentation. The government can't classify your software as noncommercial if the public can buy it.

- **Limit government-specific modifications.** The more you customize for the government, the harder it becomes to argue the result is still "commercial." If the government wants features, implement them as configuration options in your commercial product, not as a custom fork.

- **Document everything.** Keep records showing your software was developed at private expense for the commercial market, and that the government is receiving the same product available to other customers.

- **Push back on contracting officers.** Some contracting officers will default to inserting DFARS 7014 (noncommercial) into every software contract. If your software is commercial, insist on DFARS 7202. Bring documentation. Escalate if necessary.

---

## The Funding Source: Every Penny Matters

The determination of who "paid for" development is the single most consequential factor in data rights. This is where founders lose their IP—not through malice, but through ignorance of how funding source intersects with rights.

### What "Developed at Private Expense" Means

"Developed at private expense" means development was accomplished entirely with costs charged to indirect cost pools, not allocated to a government contract, or any combination of the contractor's own resources. Key points:

- **VC funding counts as private expense.** Money from your investors is private capital. Software you build with VC money, bootstrapped revenue, or your own savings is "developed at private expense."

- **Prior to the contract.** If you built your platform before the government contract, the pre-existing code was developed at private expense—regardless of what happens afterward.

- **IR&D (Independent Research and Development).** Under 31 CFR Part 31, contractors can charge certain research costs as indirect costs (IR&D). Software developed as IR&D is considered developed at private expense, even though the costs are eventually recovered through indirect rates charged to government contracts.

### What "Developed at Government Expense" Means

If the government pays for the development—through direct contract funding—the government gets Unlimited Rights to whatever was produced. This applies to:

- Features built specifically for a government contract under that contract's funding
- Modifications made to your platform at the government's direction and cost
- Any code, algorithms, or documentation developed during contract performance using contract funds

### The "Mixed Funding" Trap

Mixed funding is where most startups get burned. Here's the scenario:

You have an existing platform built at private expense. You win a government contract. Under that contract, the government pays you to add features to your platform. Now you have a product that was developed with **mixed funding**—some private, some government.

Under DFARS, the mixed-funding portions get **Government Purpose Rights.** And after 5 years, those GPR convert to **Unlimited Rights.** 

But here's the trap: if you don't carefully segregate which code was developed with which funding, the government can argue that the *entire* modified platform is mixed-funded—not just the new features. If you intermingle government-funded code with your proprietary codebase without documenting the boundaries, you risk the government claiming GPR (and eventually Unlimited Rights) to everything.

**How to protect yourself:**

1. **Segregate development.** Keep government-funded development in separate branches, modules, or repositories. Document exactly which lines of code were written under which funding.

2. **Track every hour.** Your timekeeping system should identify which engineers worked on which tasks under which funding source. Defense contract accounting (discussed in Chapter 7) requires this anyway.

3. **Architecture for separation.** Design your software architecture with separation in mind. Put government-funded features in separate modules, plugins, or microservices. Keep your core platform—your proprietary algorithms, your ML models, your core engine—in a separate codebase that is never touched with government money.

4. **Maintain pristine records.** If you ever face a data rights challenge, the government will ask you to prove what was developed at private expense. If your records are sloppy, you lose.

---

## Marking: The Silent Killer of IP

Here is a rule that has destroyed more startup IP than perhaps any other:

**If you deliver technical data or computer software to the government without the proper restrictive marking, the government is presumed to have Unlimited Rights.**

This is not a technicality. It's an absolute rule. You can have developed your software entirely at private expense. You can have spent $10 million of VC money building it. If you deliver it to the government without marking it with the appropriate restrictive legend, **you have given the government Unlimited Rights to use, modify, reproduce, and give it to anyone.**

### Required Markings

Every piece of technical data and computer software you deliver must be conspicuously marked with the appropriate legend:

**For Limited Rights Technical Data:**
```
LIMITED RIGHTS

Contract No. [contract number]
Contractor Name: [your company name]
Contractor Address: [your address]

The Government's rights to use, modify, reproduce, release, perform,
display, or disclose this technical data are restricted by paragraph (b)(3)
of the Rights in Technical Data—Noncommercial Items clause (DFARS 252.227-7013).

Any reproduction of technical data or portions thereof marked with this
legend must also reproduce the markings.

(End of legend)
```

**For Restricted Rights Computer Software:**
```
RESTRICTED RIGHTS

Contract No. [contract number]
Contractor Name: [your company name]
Contractor Address: [your address]

The Government's rights to use, modify, reproduce, release, perform,
display, or disclose this software are restricted by paragraph (b)(3)
of the Rights in Noncommercial Computer Software and Noncommercial Computer
Software Documentation clause (DFARS 252.227-7014).

Any reproduction of software or portions thereof marked with this legend
must also reproduce the markings.

(End of legend)
```

There are analogous legends for Government Purpose Rights, SBIR Data Rights, and Specially Negotiated License Rights.

### The Assertion List: Your Pre-Contract IP Declaration

**DFARS 252.227-7017** requires that you submit a **"Technical Data and Computer Software Rights Assertion List"** as part of your proposal. This is your formal declaration of any restrictions you assert on the data and software you will deliver.

The assertion list must include:

1. **Specific identification:** Name each item of technical data or computer software for which you're asserting restrictions (document name, software name, version)
2. **Basis for assertion:** State the reason (e.g., "developed exclusively at private expense")
3. **Asserted rights category:** Specify the type of rights (Limited, Restricted, GPR, SBIR Data Rights, Specially Negotiated)
4. **Name of person asserting:** Identify who is making the assertion
5. **Signature and date:** Must be signed by an authorized official

**Critical rules:**
- The assertion list must be submitted **with your proposal.** Failure to submit it, or to sign and date it, can make your offer ineligible for award.
- If no restrictions are being asserted, you must explicitly state "None"—which means you're granting Unlimited Rights to everything.
- Once the contract is awarded, the assertion list becomes an attachment to the contract.
- The government retains the right to **challenge** your assertions after award.

**Draft your assertion list carefully.** Over-asserting (claiming restrictions you can't support) will generate challenges and erode trust. Under-asserting (failing to list proprietary items) means you've surrendered rights you can never reclaim.

---

## SBIR/STTR Data Rights: The 20-Year Shield

If your company wins an SBIR or STTR contract, you get special data rights protection under **DFARS 252.227-7018**—and this protection is one of the most valuable features of the SBIR/STTR programs.

### What You Get

**SBIR/STTR data** means all technical data and computer software developed or generated during the performance of a Phase I, II, or III SBIR/STTR contract. For this data:

- The government's rights are **restricted for 20 years** from the date of contract award
- During the protection period, the government **cannot release or disclose** your data outside the government (with limited exceptions for emergency use or government support contractors under NDA)
- You retain full commercial rights throughout

### What Happens After 20 Years

When the 20-year protection period expires, the government acquires **perpetual Government Purpose Rights**—not Unlimited Rights. This is a critical improvement from the previous system. Under GPR, the government can use your data for government purposes but **cannot use it for commercial purposes or give it to competitors for commercial exploitation.**

### Marking Requirements for SBIR/STTR

All SBIR/STTR data must be marked with the SBIR Data Rights legend:

```
SBIR DATA RIGHTS

Contract No. [contract number]
Contractor Name: [your company name]
Contractor Address: [your address]
Expiration of SBIR Data Rights Period: [date—20 years from award]

The Government's rights to use, modify, reproduce, release, perform,
display, or disclose technical data or computer software marked with this
legend are restricted during the period shown as provided in paragraph (b)(4)
of the Rights in Noncommercial Technical Data and Computer Software—Small
Business Innovation Research (SBIR) Program clause (DFARS 252.227-7018).

(End of legend)
```

**Failure to mark = loss of protection.** The same rule applies: if you deliver SBIR data without the SBIR Data Rights legend, the government gets Unlimited Rights. Train every engineer, every technical writer, every person who touches a deliverable to understand this.

---

## The Bayh-Dole Act: Patents and Subject Inventions

While DFARS clauses govern rights in data and software, the **Bayh-Dole Act** (35 U.S.C. § 200-212) governs rights in **patentable inventions** conceived or reduced to practice under a government contract.

### Key Provisions

1. **You can retain title.** Small businesses and nonprofits can elect to retain ownership of "subject inventions"—inventions developed under federally funded research. This is a powerful right: you own the patent, not the government.

2. **Government gets a license.** Even when you retain title, the government automatically receives a **nonexclusive, nontransferable, irrevocable, paid-up, worldwide license** to practice the invention for government purposes. This means the government can use your patented technology without paying royalties—but it cannot sublicense it commercially or give it to commercial competitors.

3. **Disclosure obligations.** You must disclose subject inventions to the funding agency within 2 months of the inventor disclosing it to you, and elect to retain title within 2 years.

4. **Domestic manufacturing.** Products embodying a subject invention sold in the U.S. must be manufactured substantially in the U.S., unless you get a waiver.

5. **March-in rights.** The government can compel you to license your invention to others if:
   - You haven't taken effective steps to commercialize it within a reasonable time
   - Action is needed to meet health, safety, or public use needs not being satisfied
   - You're not meeting domestic manufacturing requirements

**In practice,** march-in rights have historically never been exercised against a contractor that was actively commercializing its technology. The August 2025 Commerce Department action against Harvard for non-compliance with disclosure and title election requirements was unprecedented but involved administrative failures—not active commercialization disputes. If you're building a product and bringing it to market, march-in rights are a theoretical rather than practical concern.

---

## Software-Specific IP Challenges

### SaaS and Cloud Delivery: When You Don't Deliver Code

Traditional DFARS data rights are built around the concept of **deliverable** software—code you hand over to the government. But what if your software is delivered as a service?

When you sell SaaS, you're providing access to your application over the internet. You're not delivering source code, object code, or installation packages. This creates interesting IP dynamics:

**Advantages of SaaS for IP protection:**
- You never deliver source code, so there's nothing to mark and nothing for the government to redistribute
- Your proprietary algorithms stay on your servers
- The government has a subscription, not a license to your codebase
- Updates and improvements remain under your control

**Risks of SaaS for IP protection:**
- The government may **insist on source code escrow**—a third-party holds your code in case you go bankrupt or stop supporting the product. DFARS PGI 227.7203-2 explicitly acknowledges escrow arrangements.
- Some contracts require **deployment to government infrastructure** (GovCloud, on-premises DoD systems). Once your code is deployed inside a government environment, the data rights conversation changes fundamentally—you're now delivering code.
- The government may demand **technical data** about your software's architecture, APIs, data schemas, and integration points. Even if you don't deliver source code, this technical data is subject to DFARS data rights.
- If your SaaS contract is terminated, the government may require all its data back in machine-readable formats—and may argue that any government-funded customizations should be delivered as code.

**Best practices for SaaS contracts:**
1. Clearly define in the contract that the delivery is a service (subscription), not a software deliverable
2. If source code escrow is required, negotiate tight restrictions on when and how the escrow is released
3. Keep government-funded customizations modular and separable from your core platform
4. Define data ownership and exportability clearly—the government's data is the government's, but your algorithms and code are yours

### AI Models and Training Data: The Emerging Frontier

The question of who owns AI model weights, training data, and model outputs is the most active and unresolved IP issue in defense contracting today.

**The current regulatory environment (as of early 2026):**

The GSA proposed **GSAR 552.239-7001** ("Basic Safeguarding of Artificial Intelligence Systems") in March 2026, which would dramatically expand government rights in AI systems:

- **Government owns all data inputs and outputs.** Anything the government feeds into your AI system (prompts, queries, source data) and anything the system produces during contract performance belongs to the government.
- **Government owns "custom developments."** Any modifications, fine-tuning, configurations, or enhancements made to your AI system—including model weights resulting from training on government data—are owned by the government.
- **Contractors cannot use government data to train models for others.** If the government provides training data, you cannot use that data (or models trained on it) for non-government customers.
- **Pre-existing IP is protected.** Your foundational models, base architectures, and pre-training developed before the contract are explicitly excluded—the government doesn't claim ownership of your underlying technology.

The FY2025 NDAA also addresses government procurement of AI/ML, signaling congressional intent to ensure the government has sufficient rights in AI systems while acknowledging the need to protect contractor base models.

**Practical strategies for AI companies:**

1. **Isolate government fine-tuning.** Keep government-trained model weights separate from your commercial model weights. Never merge government-funded fine-tuning into your commercial product.

2. **Document your pre-existing model.** Before starting a government contract, document the state of your model—architecture, training data sources, performance benchmarks. This establishes what existed before government funding.

3. **Define "custom development" narrowly.** In contract negotiations, push for a narrow definition that covers only government-specific fine-tuning, not improvements to your underlying platform that happen to benefit the government.

4. **Address data rights for training data explicitly.** If the government provides training data, clarify in the contract whether that data can be used for other purposes, whether models trained on it are "custom developments," and what happens to model weights at contract termination.

5. **Consider offering inference-only access.** Instead of delivering models, offer API access to your models. This is the AI equivalent of SaaS—the government gets the outputs without accessing the model weights.

### Open-Source Software Considerations

Open-source software (OSS) creates additional IP complexity in government contracts:

- **You can't assert restricted rights on OSS.** If your product incorporates open-source components, those components come with their own licenses (MIT, Apache, GPL, etc.). You can't claim Limited or Restricted Rights to code that's already publicly available under an open-source license.

- **GPL "copyleft" risk.** If your proprietary code links to GPL-licensed code in ways that trigger the GPL's copyleft provisions, your proprietary code could be considered a "derivative work" that must also be released under the GPL. This could inadvertently give the government (and everyone else) unlimited access to your code.

- **SBOM requirements.** As discussed in Chapter 8, the government increasingly requires Software Bills of Materials (SBOMs) that disclose all open-source components. This is primarily a security concern, but it also has IP implications—the government knows exactly what's proprietary and what's open source.

- **Strategic architecture.** Use clean separation between open-source components and proprietary code. Keep your proprietary algorithms, models, and business logic in separate modules that don't trigger copyleft obligations.

---

## Data Rights Negotiations: What's Negotiable and What's Not

Despite the regulatory framework, data rights in government contracts are **more negotiable than most founders realize.** Here's what you need to know:

### What's Generally Not Negotiable

- **The basic framework.** The principle that government-funded development yields government rights is statutory. You can't negotiate around the fundamental DFARS structure.
- **Form, fit, and function data.** Data describing the form, fit, and function of delivered items—interface specifications, dimensions, performance envelopes—always goes to the government with Unlimited Rights. This data is essential for maintenance, repair, and interoperability.
- **Marking requirements.** You can't negotiate out of the obligation to mark your data properly.

### What IS Negotiable

- **The definition of deliverables.** This is your most powerful lever. You get to control **what's on the deliverables list.** If your core algorithm isn't a deliverable, the government doesn't get data rights to it. Negotiate to deliver only the outputs—reports, specifications, interface documentation—not the underlying proprietary technology.

- **Specifically Negotiated License Rights.** DFARS explicitly allows for **"specifically negotiated license rights"**—custom terms agreed upon between you and the government. These can be more permissive than Restricted Rights but more restrictive than Unlimited Rights. However, they cannot be *more* restrictive than Limited or Restricted Rights.

- **The GPR period.** While the default is 5 years before GPR converts to Unlimited, this period is negotiable. You can push for longer periods, up to the maximum the contracting officer will accept.

- **Scope of "government purpose."** In some contracts, you can negotiate a narrower definition of what constitutes a "government purpose" for GPR data.

- **OTA terms.** In Other Transaction (OT) agreements, IP rights are **fully negotiable.** OTAs are not subject to FAR/DFARS unless the parties agree to incorporate those clauses. This is one of the biggest advantages of the OTA pathway for startups.

### Negotiation Strategies

1. **Start the IP conversation early.** Don't wait until contract award. Raise data rights in your initial meetings with the program office, during proposal Q&A, and in any pre-solicitation engagement.

2. **Know your BATNA (Best Alternative to Negotiated Agreement).** If the data rights terms are unacceptable, be willing to walk away. Some contracts aren't worth the IP you'd surrender.

3. **Propose alternative delivery structures.** Instead of delivering your proprietary source code, offer to deliver compiled binaries, API access, or SaaS subscriptions. Instead of delivering your ML model, offer to deliver an inference endpoint.

4. **Engage a data rights attorney.** Government IP law is specialized. Don't rely on your general counsel or your startup attorney. Hire someone who negotiates DFARS data rights regularly.

5. **Educate the contracting officer.** Many contracting officers use default DFARS clauses without fully understanding the implications for commercial and startup companies. A clear, professional explanation of why certain terms don't work for your business model can be surprisingly effective.

---

## Practical IP Protection Strategies

### Architecture for IP Protection

Design your software architecture with data rights in mind from day one:

**1. Core/Edge Separation**
- **Core:** Your proprietary algorithms, ML models, business logic, and competitive differentiators. Developed at private expense. Never delivered to the government. Never touched with government money.
- **Edge:** Government-specific integrations, adapters, interfaces, and configurations. Developed under contract. Delivered to the government with appropriate (possibly broader) rights.

This is not a theoretical exercise. This is how you design your system:

```
┌─────────────────────────────────────────┐
│  Government-Specific Layer (Delivered)  │
│  - DoD integrations, connectors         │
│  - Government-specific UI components    │
│  - Data format converters               │
│  - Classification marking tools         │
│  Rights: GPR or Unlimited               │
├─────────────────────────────────────────┤
│  API / Interface Layer                  │
│  - Published APIs (form/fit/function)   │
│  - Integration specifications           │
│  Rights: Unlimited (it's F/F/F data)    │
├─────────────────────────────────────────┤
│  Proprietary Core (NOT Delivered)       │
│  - Core algorithms and ML models        │
│  - Proprietary business logic           │
│  - Private training data                │
│  - Competitive differentiators          │
│  Rights: Not applicable (not delivered) │
└─────────────────────────────────────────┘
```

**2. Separate Repositories**
Maintain separate code repositories for privately funded and government-funded development. Use clear branch management and access controls. Tag every commit with its funding source.

**3. Modular Design**
Build government-specific features as modules, plugins, or microservices that can be cleanly separated from your core platform. If the government eventually gets Unlimited Rights to a feature, you want to be able to extract that feature without exposing your core technology.

### Development Process Controls

**1. Timekeeping Discipline**
Every engineering hour must be tracked against a specific funding source. Your timekeeping system should distinguish between:
- Direct labor charged to a specific government contract
- IR&D (Independent Research and Development)
- Privately funded development (not charged to any government contract)

**2. Funding Source Documentation**
For every feature, module, data set, and algorithm, maintain a record of:
- What funding source was used
- When development occurred
- Which engineers worked on it
- What the deliverable status is

**3. IP Review Process**
Before delivering anything to the government, conduct an IP review:
- Is this item on the deliverables list? If not, don't deliver it.
- Is it properly marked with the appropriate restrictive legend?
- Does the assertion list cover this item?
- Has the funding source been documented?

**4. Training**
Every engineer, project manager, and business development person in your company should understand the basics of data rights. Many IP losses occur because an engineer attaches a document to an email without thinking about markings, or a PM includes proprietary technical information in a status report.

---

## Why VCs Care About Data Rights

If you're raising money for a defense tech company, understand this: **your approach to data rights is a make-or-break factor for investors.**

Defense tech VCs evaluate IP risk obsessively because:

1. **Valuation depends on IP ownership.** If the government has Unlimited Rights to your core technology, your technology has no competitive moat. Why would anyone pay a premium for a company whose core IP can be freely given to competitors?

2. **Exit depends on IP.** Whether you're pursuing an acquisition or an IPO, acquirers and public market investors need assurance that your IP is defensible. "The government has Unlimited Rights to our source code" is not a sentence that supports a $500M exit.

3. **Competition depends on IP.** If the government can give your code to incumbent defense contractors, those contractors—with their existing relationships, cleared employees, and contract vehicles—can undercut you using your own technology.

4. **Revenue durability depends on IP.** If the government doesn't need you to maintain or modify your software (because they have unlimited access to the source code), your recurring revenue stream evaporates.

**What VCs want to see:**
- A clear IP strategy that separates proprietary core technology from government deliverables
- Proper use of commercial software designation where applicable
- Meticulous marking and assertion practices
- Contracts that preserve Restricted or Limited Rights on proprietary technology
- An architecture designed to protect core IP
- Legal counsel experienced in DFARS data rights

---

## The IP Decision Matrix

When evaluating any government contract opportunity, run it through this decision matrix:

| Question | If Yes | If No |
|---|---|---|
| Is the contract asking me to deliver source code? | Red flag. Negotiate alternatives (APIs, SaaS, compiled binaries). If delivery is mandatory, ensure proper markings and assertions. | Good. You maintain physical control of your IP. |
| Will I be developing new features under this contract? | Document funding source meticulously. Architecture must separate new features from core IP. | Good. Delivering existing commercial product preserves existing rights. |
| Can I establish my software as "commercial"? | Insist on DFARS 252.227-7202. Your commercial license terms govern. | You're under DFARS 7014 (noncommercial). Understand the rights spectrum and mark everything. |
| Does the contract include SBIR/STTR funding? | Use DFARS 252.227-7018 markings. You get 20-year protection. | Standard DFARS data rights apply. |
| Is this an OTA (Other Transaction)? | IP terms are fully negotiable. Push for commercial-like terms. Don't accept boilerplate DFARS clauses in an OTA. | Standard DFARS clauses likely apply. Focus on assertions and markings. |
| Does the government want to deploy my software on their infrastructure? | You're effectively delivering code. Negotiate escrow terms, deployment restrictions, and clear ownership boundaries. | SaaS delivery protects your code. Ensure the contract reflects the service model. |
| Am I using AI/ML models? | Clarify ownership of model weights, training data, and outputs. Document pre-existing models. Isolate government fine-tuning. | Standard data rights framework applies. |

---

## Common Mistakes

**1. Not marking deliverables.** This is the most catastrophic and most avoidable mistake. Every deliverable must be marked with the appropriate restrictive legend. Build this into your delivery process as a mandatory checklist item.

**2. Accepting default DFARS clauses without reading them.** Many contracts include DFARS 7014 (noncommercial software) when your software should be treated as commercial under DFARS 7202. Don't sign until the right clauses are in place.

**3. Not submitting an assertion list.** If DFARS 252.227-7017 is in the solicitation and you don't submit an assertion list, you've failed to assert any restrictions. The government gets Unlimited Rights to everything.

**4. Co-mingling funding sources.** If government-funded code is developed in the same repository, the same module, or even the same file as your proprietary code, separating the two becomes exponentially harder. Maintain clean separation from day one.

**5. Assuming OTAs automatically protect your IP.** OTAs allow for negotiation—they don't guarantee protection. If you don't negotiate favorable IP terms in an OTA, the government may insert unfavorable terms by default.

**6. Ignoring the 5-year GPR sunset.** Government Purpose Rights seem like a reasonable compromise—until you realize that in 5 years, they convert to Unlimited Rights. If your core technology is under GPR, the government gets unrestricted access to it 5 years from now. That clock starts ticking at contract award.

**7. Treating data rights as a legal problem.** Data rights are an **engineering and business problem** as much as a legal one. Your architecture, your development process, your timekeeping, and your delivery procedures all determine whether your IP is protected. Lawyers can draft great contracts, but if your engineers deliver unmarked source code, the contracts don't matter.

---

## Key Terms

**Technical Data:** Recorded information of a scientific or technical nature, excluding computer software. Governed by DFARS 252.227-7013.

**Computer Software:** Programs, source code, object code, algorithms, and related material. Governed by DFARS 252.227-7014 (noncommercial) or DFARS 252.227-7202 (commercial).

**Unlimited Rights:** The broadest government license—use, modify, reproduce, distribute for any purpose, including giving to competitors.

**Government Purpose Rights (GPR):** Middle-ground license for mixed-funding development—broad government use, but not for commercial purposes. Converts to Unlimited Rights after 5 years.

**Limited Rights:** Most restrictive license for privately developed technical data—government use only within the government, no external release.

**Restricted Rights:** Most restrictive license for privately developed noncommercial software—single computer use, backup copies, no distribution.

**DFARS 252.227-7013:** Rights in Technical Data—Noncommercial Items. The core clause governing government rights in technical data.

**DFARS 252.227-7014:** Rights in Noncommercial Computer Software. The core clause governing government rights in software that isn't commercial.

**DFARS 252.227-7202:** Rights in Commercial Computer Software. Provides that the government gets only the license terms customarily offered to the public.

**DFARS 252.227-7017:** Identification and Assertion of Use, Release, or Disclosure Restrictions. Requires the contractor to submit an assertion list with proposals.

**DFARS 252.227-7018:** Rights in SBIR/STTR data. Provides 20-year protection, with perpetual GPR after expiration.

**Form, Fit, and Function Data:** Data describing the external physical and functional characteristics of an item. Always delivered with Unlimited Rights.

**Bayh-Dole Act:** Federal law allowing small businesses and nonprofits to retain title to inventions developed with federal funding. Government retains nonexclusive license.

**March-in Rights:** Government's right under Bayh-Dole to compel licensing of a subject invention under specific circumstances (failure to commercialize, health/safety needs).

**Subject Invention:** Any invention conceived or first reduced to practice under a federal contract.

**IR&D (Independent Research & Development):** Contractor research and development not directly charged to a specific government contract. Treated as private expense for data rights purposes.

**Specifically Negotiated License Rights:** Custom data rights terms negotiated between the contractor and the government, subject to certain limits.

**Source Code Escrow:** Arrangement where source code is held by a third party and released to the government under specified conditions (bankruptcy, failure to support).

**GSAR 552.239-7001:** Proposed GSA clause (March 2026) establishing government ownership of AI system data inputs, outputs, and custom developments.
