# Chapter 9: Classified Work — Security Clearances, SCIFs, and the World Behind the Green Door

---

There's a phrase in defense circles: "behind the green door." It refers to the classified world—the one you can't see from outside, where the most sensitive national security work happens in windowless rooms, on air-gapped networks, by people who can't tell their families what they do all day.

For a startup, the classified world represents both the ultimate competitive moat and the ultimate barrier to entry. Once you're inside, the number of companies that can compete with you shrinks dramatically—because almost no one else can get in. But getting in requires infrastructure, overhead, and organizational changes that can reshape or even break your company.

This chapter is about that world: what classification actually means, what clearances are, what they cost, how long they take, what can disqualify you, how to set up the physical and IT infrastructure to do classified work, and—perhaps most importantly—whether you should even try.

---

## The Classification System

Before diving into clearances and facilities, you need to understand what classification actually is and how it's structured. The U.S. government classifies information at three levels based on the damage that unauthorized disclosure could cause to national security:

### The Three Levels

| Level | Standard | Informal Description |
|---|---|---|
| **Confidential** | Unauthorized disclosure could reasonably be expected to cause **damage** to national security | The lowest level. Think logistics data, routine military operational plans, some equipment specifications. |
| **Secret** | Unauthorized disclosure could reasonably be expected to cause **serious damage** to national security | The most common level. Most government day-to-day classified work happens at Secret. Think detailed weapons system data, intelligence reports, war plans, sensitive diplomatic communications. |
| **Top Secret** | Unauthorized disclosure could reasonably be expected to cause **exceptionally grave damage** to national security | The highest level. Think nuclear weapons design, signals intelligence methods, covert operations, the identities of intelligence sources. |

These three levels—Confidential, Secret, and Top Secret—are collectively known as **collateral classified information**. They're the baseline classification system, and having a security clearance at a given level (say, Secret) generally grants you access to information at that level and below, **provided you have a need-to-know**.

The "need-to-know" principle is critical. A Secret clearance doesn't give you access to all Secret information. It gives you the *eligibility* to access Secret information that is directly relevant to your work. If a project doesn't involve you, you don't get access—even if you hold the right clearance.

### Beyond Collateral: SCI and SAPs

Above and beyond the collateral classification levels sit two overlay systems that impose additional access restrictions:

**Sensitive Compartmented Information (SCI)**

SCI is not a classification *level*—it's a **control system** placed on top of classified information. Information at any level (Secret, Top Secret) can also be designated as SCI if it concerns or is derived from sensitive intelligence sources, methods, or analytical processes.

SCI is managed by the Director of National Intelligence (DNI) and is subdivided into **compartments** and **sub-compartments**, each representing a specific intelligence source or method. Access to SCI requires:

1. Typically a Top Secret clearance (the investigation for TS and SCI eligibility is essentially the same, which is why you see "TS/SCI" as a combined designation)
2. Formal authorization for each specific SCI compartment
3. A "need-to-know" for that specific compartment's information
4. Signing a non-disclosure agreement
5. Being formally "read in" (indoctrinated) to the compartment

SCI must be handled in a **SCIF** (Sensitive Compartmented Information Facility)—more on those below.

The key point: two people can both hold TS/SCI clearances and still be unable to share information with each other if they're read into different compartments. This is the "stovepipe" effect—information is tightly controlled even among cleared personnel.

**Special Access Programs (SAPs)**

SAPs provide safeguards and access restrictions that **exceed** those for regular classified information. SAPs are established when a program is so sensitive that standard classification protections are insufficient.

SAPs come in two types:
- **Acknowledged SAPs:** The program's existence is public, but its details are classified. Most weapons development programs are acknowledged SAPs.
- **Unacknowledged SAPs:** The program's existence itself is classified. These are the "black programs" that officially don't exist. The F-117 stealth fighter was an unacknowledged SAP before it was revealed in 1988.

SAPs are further categorized as:
- **Acquisition SAPs:** New weapons systems and technologies
- **Intelligence SAPs:** Sensitive intelligence operations
- **Operations and Support SAPs:** Sensitive military activities

Access to a SAP requires approval beyond what a standard clearance provides, including additional background investigation, special briefings, and acknowledgment of severe penalties for unauthorized disclosure.

**What this means for startups:** Most defense software startups will operate at the **Secret** level, if they do classified work at all. TS/SCI and SAP work is possible but dramatically increases complexity, cost, and overhead. Secret-level work is the realistic entry point for classified operations.

---

## Personal Security Clearances

A Personal Security Clearance (PCL) is an individual-level determination that a person is eligible to access classified information at a given level. The process is managed by the **Defense Counterintelligence and Security Agency (DCSA)**, with adjudication performed by the **DoD Consolidated Adjudication Facility (DODCAF)**.

### The SF-86: Your Entire Life on Paper

The clearance process begins with **Standard Form 86** (SF-86)—the "Questionnaire for National Security Positions." The SF-86 is completed electronically through the e-QIP (Electronic Questionnaires for Investigations Processing) system, and it is one of the most comprehensive personal information forms in existence.

The SF-86 collects:

- **Residential history:** Every place you've lived for the past 10 years, with complete addresses
- **Employment history:** Every job for the past 10 years, with supervisor names and contact information
- **Education:** All schools attended
- **Personal relationships:** Spouse, cohabitants, relatives, foreign contacts
- **Foreign travel:** Every foreign country visited in the past 10 years
- **Foreign contacts:** Any close or continuing contact with foreign nationals
- **Financial records:** Debts, bankruptcies, tax issues, gambling losses
- **Criminal history:** Any arrests, charges, or convictions—even sealed or expunged records
- **Drug and alcohol use:** Any illegal drug use, ever. Any alcohol-related incidents.
- **Mental health:** Certain mental health conditions or treatments
- **Military service:** Complete military history
- **References:** People who can attest to your character in each location you've lived and worked

A new version of the SF-86 (dated February 2024) became available in May 2025, and all submissions must use the updated form after August 1, 2025.

The SF-86 is not a form you fill out casually. Errors, omissions, or inconsistencies will delay your investigation and can disqualify you. The number one rule: **be completely honest.** Investigators already have access to most of the information they're asking about. They're testing whether you'll disclose it voluntarily. Concealing information is almost always worse than whatever you're hiding.

### Investigation Types and Timelines

After the SF-86 is submitted, DCSA investigators conduct the background investigation:

| Clearance Level | Investigation Type | Timeline (Q4 FY2024) | Notes |
|---|---|---|---|
| **Confidential** | Tier 1 / NACLC | 60–90 days typical | Simplest investigation. National agency checks, local records checks, credit check. |
| **Secret** | Tier 3 / NACLC | ~138 days (90th percentile) | National agency checks, credit check, some interviews depending on complexity. |
| **Top Secret** | Tier 5 / SSBI | ~249 days (90th percentile) | Full investigation. In-person interviews with the applicant, neighbors, coworkers, supervisors. Field investigations in every location lived, worked, or attended school. |
| **TS/SCI** | Tier 5 + polygraph | 6–12+ months | Same as Top Secret plus a polygraph examination (counterintelligence scope or full-scope lifestyle). |

**The backlog:** DCSA has made significant progress reducing its investigation backlog, from 290,000 cases in September 2024 to approximately 222,000 cases by May 2025—a 24% reduction achieved through a "tiger team" initiative targeting bottlenecks. The inventory is projected to fall below 200,000 by the end of FY2025.

**Trusted Workforce 2.0:** The government is transitioning from periodic reinvestigations (every 5, 10, or 15 years depending on level) to **Continuous Vetting**—automated, ongoing review of cleared personnel through continuous monitoring of criminal records, financial data, travel, and other relevant information. This eliminates surprise reinvestigation gaps but may increase workload as the system reaches maturity.

### What Disqualifies You

There is no statutory list of automatic disqualifiers for a security clearance. Adjudicators evaluate 13 guidelines holistically, considering the whole person and whether concerns are mitigated. That said, certain factors commonly result in denial:

**1. Dishonesty and lack of candor (#1 fastest way to lose)**

If you lie on the SF-86 or during your investigation interview, you will almost certainly be denied. The government can forgive many things; it cannot forgive deception.

**2. Financial problems (#1 most common denial factor)**

Between 2019 and 2022, financial considerations were the top reason for security clearance denials. Unpaid debts, tax liens, unfiled returns, bankruptcies, and gambling problems all raise concerns about susceptibility to bribery or coercion. The government doesn't expect perfection—they expect responsibility. If you have debt, show a repayment plan.

**3. Foreign influence**

Unreported foreign relationships, financial interests abroad, dual citizenship, or foreign property ownership raise concerns about divided loyalties. Close relationships with foreign nationals from adversarial nations (China, Russia, Iran, North Korea) receive particular scrutiny.

**4. Criminal conduct**

Felony convictions are serious. Patterns of criminal behavior, even misdemeanors, indicate unreliability. For SCI and SAP access, conviction of a crime resulting in incarceration exceeding one year can be an additional disqualifier.

**5. Drug use**

Any illegal drug use is a concern. Recent drug use (within the last 1–2 years) is especially problematic. Marijuana use is still disqualifying under federal standards despite state legalization—federal law governs clearances.

**6. Non-U.S. citizenship**

You must be a U.S. citizen to hold a security clearance. No exceptions. Green card holders, visa holders, and dual citizens (who have not renounced foreign citizenship) face significant additional scrutiny.

**Important nuance:** None of these factors are automatic disqualifiers except non-U.S. citizenship. Adjudicators consider the whole person—the seriousness of the concern, how recent it is, the circumstances, and what mitigating factors exist. A bankruptcy from 10 years ago that you've since recovered from is treated very differently from current financial distress with no plan.

---

## Facility Security Clearances

While a PCL clears an *individual*, a **Facility Clearance (FCL)** clears your *company*. An FCL is an administrative determination by DCSA that your company is eligible to access, receive, and safeguard classified information.

No FCL, no classified contracts. Period.

### The Chicken-and-Egg Problem

This is the single most frustrating aspect of entering the classified world: **you cannot apply for an FCL on your own.** You need a **sponsor**.

Sponsorship must come from either:
- A **Government Contracting Activity (GCA)**—the government agency that wants to give you a classified contract, or
- A **cleared prime contractor** that needs you as a subcontractor on a classified program

The sponsorship request is submitted through the National Industrial Security System (NISS) and typically includes a **DD Form 254** (Contract Security Classification Specification) and a formal letter of sponsorship.

But here's the catch: the government or prime contractor needs a reason to sponsor you, which usually means a contract or at least a contract opportunity. And many contracts require an FCL to bid. So you need a contract to get a clearance, and you need a clearance to get a contract.

**How to break the cycle:**

1. **Win an unclassified contract first.** Build a relationship with a program office on unclassified work, then seek sponsorship from that same office when a classified requirement emerges.

2. **Team with a cleared prime.** Partner as a subcontractor to a company that already has an FCL. The prime can sponsor your FCL and provide you access to their SCIF while yours is being processed.

3. **DIU and innovation pathways.** Some innovation organizations (DIU, AFWERX, etc.) will sponsor FCLs for companies selected for programs that have classified components.

4. **SBIR Phase II.** Some SBIR Phase II contracts have classified components and can serve as the basis for FCL sponsorship.

5. **Pre-position.** Start the FCL process before you absolutely need it. The process takes 6–12+ months, and you don't want it on the critical path of a contract award.

### DD Form 441: The Security Agreement

The **DD Form 441** ("Department of Defense Security Agreement") is a legally binding document that formalizes the FCL. By executing it, your company agrees to comply with the NISPOM (discussed below) and to protect classified information according to government requirements. The DD Form 441 is accompanied by the **SF 328** ("Certificate Pertaining to Foreign Interests"), which discloses any foreign ownership, control, or influence.

### Key Management Personnel (KMP)

DCSA identifies specific senior leaders in your company as **Key Management Personnel**—individuals who must hold PCLs at the same level as the FCL. Who qualifies as KMP depends on your corporate structure, but typically includes:

- **Chief Executive Officer (CEO) or President** — Must be cleared. If your CEO is a foreign national or otherwise ineligible for a clearance, you have a fundamental problem.
- **Facility Security Officer (FSO)** — Must be cleared. This is the person responsible for your security program. More on this below.
- **Board members and senior executives** — Depending on governance structure, board members or other officers with authority over company operations may need clearances.
- **Insider Threat Program Senior Official (ITPSO)** — Required under the NISPOM.

All KMP must be **U.S. citizens.** Foreign nationals cannot be KMP. The government covers the cost of processing both FCLs and KMP clearances—you don't pay for the investigation itself, but you absorb the opportunity cost of your leaders' time during the process.

### The Facility Security Officer (FSO)

Every cleared company must designate a Facility Security Officer. The FSO is responsible for:

- Establishing, implementing, and maintaining the company's security program
- Ensuring compliance with the NISPOM and DCSA regulations
- Processing and managing personnel security clearances
- Conducting security briefings and debriefings for cleared employees
- Managing classified document control
- Reporting security incidents and suspicious contacts to DCSA
- Administering the Insider Threat Program
- Completing mandatory DCSA security training

**Hiring an FSO is often the first concrete step** toward obtaining an FCL. For a startup, this is a significant hire—either a full-time position (if you expect substantial classified work) or a part-time/outsourced role (if classified work is limited).

An experienced FSO typically costs $80,000–$120,000 annually if hired full-time. Outsourced FSO services are available for smaller companies at $2,000–$5,000 per month, though some DCSA regions prefer full-time FSOs.

The FSO must hold a PCL at the level of the FCL and must complete DCSA's required training courses, including the FSO Program Management course available through the Center for Development of Security Excellence (CDSE).

---

## FOCI: The Existential Threat to VC-Backed Startups

**Foreign Ownership, Control, or Influence (FOCI)** is the issue that has destroyed more defense startup clearance applications than any other single factor. If your startup has foreign founders, foreign investors, or a VC fund with foreign limited partners, FOCI is your most critical concern.

### What FOCI Is

FOCI exists when a foreign interest—any foreign government, agency, firm, or individual—has the power, directly or indirectly, to direct or decide matters affecting the management or operations of your company. DCSA evaluates FOCI based on several factors:

- Record of economic espionage against the U.S.
- Record of enforcement actions against the U.S.
- Type and sensitivity of the information to be accessed
- Source, nature, and extent of the foreign ownership or investment
- Nature of any bilateral and multilateral security agreements
- Ownership, control, or influence by foreign government entities

**Critical threshold:** FOCI review is no longer limited to classified contracts. A 2023 expansion extended FOCI scrutiny to **unclassified DoD contracts exceeding $5 million**. This means your cap table can affect your ability to win even unclassified defense contracts.

### How VC Funding Creates FOCI Problems

Silicon Valley startups are particularly vulnerable to FOCI because:

**1. Foreign LPs in VC funds.** Your VC firm may be entirely American, but if the fund includes foreign limited partners—sovereign wealth funds, family offices from the Gulf, Chinese or European investors—those upstream foreign interests create FOCI concerns. DCSA scrutinizes all upstream ownership entities, not just direct investors.

**2. Foreign co-founders or board members.** If your CTO is a Canadian citizen or your co-founder is an Israeli national, they cannot serve as KMP. If they have substantial equity or governance rights, they create FOCI that must be mitigated.

**3. SAFEs and convertible instruments with foreign holders.** While Simple Agreements for Future Equity aren't explicitly addressed in FOCI regulations, any foreign-held instrument that could convert to equity or grant influence is subject to FOCI scrutiny. A SAFE held by a foreign investor contributes to FOCI concerns, especially if it leads to significant future equity.

**4. Board seats and observer rights.** Even minority foreign investors who hold board seats or observer rights can trigger FOCI. Control and influence matter, not just ownership percentage.

**5. Debt instruments.** Foreign-held debt that creates leverage over the company can constitute FOCI, even if the foreign party doesn't own equity.

### FOCI Mitigation Instruments

If FOCI exists, it doesn't automatically disqualify you from obtaining an FCL—but it must be **mitigated** through formal agreements approved by DCSA. The instruments, from least restrictive to most:

| Instrument | Restriction Level | When Used | What It Does |
|---|---|---|---|
| **Board Resolution (BR)** | Lowest | Minor foreign influence, no ownership | The board passes a resolution committing to protect classified information and prevent foreign influence. |
| **Security Control Agreement (SCA)** | Low–Moderate | Minority foreign ownership with limited influence | Establishes specific security controls while allowing some foreign participation in governance. |
| **Special Security Agreement (SSA)** | Moderate–High | Foreign entity effectively owns or controls the company | Allows some foreign participation but requires appointing cleared U.S. citizens as "outside directors" who outnumber any foreign-appointed "inside directors." A Government Security Committee (GSC) composed solely of cleared U.S. citizens oversees all classified matters. |
| **Proxy Agreement (PA)** | High | Majority foreign ownership or effective control | The foreign owner retains economic ownership but transfers most operational control and voting rights to cleared U.S. proxy holders. No inside directors allowed—the proxies run the company for purposes of classified work. |
| **Voting Trust Agreement (VTA)** | Highest | Majority foreign ownership; most restrictive | The foreign owner transfers **legal title** of shares to cleared U.S. Voting Trustees. The trustees hold all voting rights and act as the owners. The original foreign shareholders retain only economic interests (dividends). No interlocking relationships between foreign owners and KMP are permitted. |

**The cost of FOCI mitigation is substantial:** legal fees for structuring agreements range from $50,000 to $250,000+, and ongoing compliance creates permanent governance overhead. VTAs and Proxy Agreements fundamentally alter your corporate governance—foreign investors effectively lose control.

### Structuring Your Cap Table to Avoid FOCI

If you think you might ever pursue classified defense work:

1. **Know your investors' LP base.** Before accepting VC money, ask whether the fund includes foreign LPs. If it does, understand the implications before closing.

2. **Restrict foreign board seats.** Don't give board seats or board observer rights to foreign investors or foreign-owned entities.

3. **Cap foreign equity.** Keep foreign ownership well below thresholds that require the more restrictive mitigation instruments. While there's no bright-line rule, lower foreign ownership percentage means simpler mitigation.

4. **Document everything.** Maintain a clear cap table that identifies the citizenship of every equity holder, the nationality of every LP in your VC funds, and any foreign debt holders.

5. **Consult a FOCI attorney early.** Before your Series A, if defense work is in your plan, engage a lawyer who specializes in FOCI and DCSA matters. Restructuring after the round closes is vastly more expensive than getting the structure right from the start.

6. **Consider a "clean" subsidiary.** Some companies create a wholly owned U.S. subsidiary with no foreign investors, foreign board members, or foreign KMP. The subsidiary holds the FCL and does all classified work. The parent company, with its foreign-influenced investor base, remains separate. This only works if the subsidiary is genuinely independent in its classified operations.

---

## The NISPOM: The Rulebook for Cleared Contractors

The **National Industrial Security Program Operating Manual** (NISPOM), now codified as **32 CFR Part 117**, is the foundational regulation governing how cleared contractors protect classified information. If you hold an FCL, the NISPOM is your bible.

The NISPOM covers:

- **Security clearances:** Requirements for PCL processing, initial briefings, debriefings, and continuous monitoring
- **Classified information marking and handling:** How to mark, store, transmit, and destroy classified documents
- **Physical security:** Requirements for secure storage (safes, vaults, closed areas), access controls, visitor control
- **NISPOM Change 2 requirements:** The Insider Threat Program—every cleared contractor must establish a program to detect, deter, and mitigate insider threats, including appointing an ITPSO (Insider Threat Program Senior Official)
- **IT security:** Requirements for classified information systems, including standalone classified workstations
- **International security:** Requirements for handling classified information in international programs (Foreign Military Sales, NATO, bilateral programs)
- **Reporting requirements:** Obligations to report adverse information about cleared employees, suspicious contacts, security violations, and changes to the company's ownership or governance structure

**Key point:** NISPOM compliance is not optional, and violations have consequences. DCSA conducts periodic security reviews of cleared facilities. A poor review can result in conditions, suspensions, or revocation of your FCL.

---

## SCIFs: The Physical Infrastructure of Classified Work

A **Sensitive Compartmented Information Facility (SCIF)** is a physically secure room or building where SCI can be stored, discussed, and processed. But even if you're only doing Secret-level work (not SCI), the physical security requirements for a classified work space are substantial.

### What a SCIF Requires

SCIFs are governed by **Intelligence Community Directive (ICD) 705**, which establishes stringent standards for construction, accreditation, and maintenance. Key requirements include:

**Construction:**
- **Walls, ceiling, and floor** must form a continuous security barrier ("six-sided box") that prevents penetration, visual observation, and acoustic eavesdropping
- Reinforced construction—often incorporating expanded metal mesh, plywood layers, or specialized drywall
- Sound Transmission Class (STC) rating of **45 to 50**, meaning speech within the SCIF should be barely audible (or inaudible) from outside
- No windows in most configurations, or shatter-resistant windows with opaque coverings if windows exist
- Steel or reinforced doors meeting specific thickness and security standards, with automatic closers, reinforced hinges, and alarms

**Access Control:**
- Badge readers, PIN pads, or biometric scanners
- Man-trap or controlled entry points
- Visitor logs and escort procedures
- No personal electronic devices (cell phones, smart watches, fitness trackers) inside the SCIF

**Intrusion Detection:**
- UL 2050-certified Intrusion Detection System (IDS)
- All access points monitored 24/7
- Monitoring station with response capability

**TEMPEST:**
- Protection against compromising electromagnetic emanations
- RF shielding to prevent electronic eavesdropping
- TEMPEST mitigations are one of the biggest cost drivers in SCIF construction

**IT Infrastructure:**
- Classified networks (SIPRNET, JWICS) with dedicated, accredited connections
- No Wi-Fi, no Bluetooth, no connection to unclassified networks
- Dedicated classified workstations, printers, and copiers
- Air-gapped systems—physically separated from all unclassified networks

### What a SCIF Costs

SCIF construction costs vary enormously based on size, location, and requirements:

| SCIF Size/Type | Cost Range | Notes |
|---|---|---|
| Small SCIF (<200 sq ft) | $200K–$400K+ | $1,000–$1,800 per sq ft. High fixed costs (door, IDS, ACS) amortized over small area. |
| Medium SCIF (500–2,000 sq ft) | $250K–$1M+ | $500–$1,000 per sq ft. Typical for small contractor. |
| Large SCIF (5,000+ sq ft) | $1.5M–$5M+ | Below $500/sq ft for larger facilities. Economies of scale on fixed costs. |
| Modular/portable SCIF | $150K–$500K | Pre-fabricated enclosures, faster to deploy, can be containerized. |
| Tenant improvement (retrofitting existing space) | $150K–$750K+ | Converting existing office space. Most economical if no major structural issues. |

Beyond construction, add costs for:
- Accreditation process: $25K–$75K+ (consulting, documentation, inspections)
- Annual maintenance and monitoring: $10K–$50K/year
- Classified IT equipment and network connections: $50K–$200K+ (SIPRNET circuits, classified workstations, safes)

### Alternatives to Building Your Own SCIF

Given the cost, most startups should **not** build their own SCIF as their first move into classified work. Alternatives:

**1. Use a partner's SCIF.** If you're subcontracting to a cleared prime contractor, you can often use their SCIF. Formalize this through a co-use or co-utilization agreement, where the "host" organization allows "tenant" organizations to use the facility.

**2. Government-provided space.** Some government organizations provide contractor workspace within government SCIFs, particularly for on-site support contracts. This is common for software development contracts where developers work embedded with the government team.

**3. Leased SCIF space.** Companies like Easterly Government Properties and others specialize in leasing government-suitable facilities, including pre-built SCIF space. Look for commercial office space near military installations that already has accredited SCIF rooms for lease.

**4. Shared contractor SCIF space.** In defense hubs (Northern Virginia, San Diego, Huntsville, Colorado Springs), some facilities offer shared or co-leased SCIF space for smaller contractors.

**5. Modular SCIFs.** Pre-fabricated, self-contained SCIF units that can be installed inside existing office buildings. Faster to deploy (weeks vs. months for traditional construction) and can be relocated if you move offices.

**The recommended approach for startups:** Start with a partner's SCIF or government-provided space. Only invest in your own SCIF when your classified workload justifies the capital expenditure—typically when you have multiple classified contracts and need regular access that a partner arrangement can't efficiently provide.

---

## Classified IT Systems: A Different World

Your commercial cloud infrastructure—AWS, Azure, Google Cloud—doesn't work for classified systems. The classified IT world operates on completely separate networks, physically isolated from the internet and all unclassified systems.

### The Major Classified Networks

**NIPRNet (Non-classified Internet Protocol Router Network)**
- Classification: Unclassified/CUI
- Purpose: The DoD's unclassified network for routine business. This is the network most defense contractors interact with for unclassified contracts.
- Connection: Not actually classified, but mentioned for context. Connects to the public internet through filtered gateways.

**SIPRNet (Secret Internet Protocol Router Network)**
- Classification: SECRET
- Purpose: The DoD's primary network for classified communications up to Secret. Used for classified email (SIPR email), file sharing, classified web-based applications, and interoperability between DoD commands and agencies.
- Access: Requires a Secret clearance, SIPRNET terminal in an accredited space, and a SIPRNet token (PKI certificate).
- Scale: The most widely used classified network. Most Secret-level classified work happens on SIPRNet.

**JWICS (Joint Worldwide Intelligence Communications System)**
- Classification: TOP SECRET/SCI
- Purpose: The Intelligence Community's primary network for TS/SCI communications. Used by NSA, CIA, DIA, NGA, and other IC agencies for intelligence gathering, analysis, and dissemination.
- Access: Requires TS/SCI clearance and access from within a SCIF with a JWICS terminal. No access from outside SCIFs.
- Scale: More restricted than SIPRNet. Terminals are primarily in SCIFs at intelligence agencies, combatant commands, and select cleared contractor facilities.

**NSANet (National Security Agency Network)**
- Classification: TOP SECRET/SCI (NSA-specific)
- Purpose: The NSA's internal network for its specific tools, programs, and operations.
- Access: Extremely restricted. Requires NSA-specific clearances and access from NSA facilities or NSA-accredited contractor spaces.

**What this means for software companies:**

If you're building software for the classified environment, everything changes:

1. **No cloud.** Your application runs on air-gapped servers in a SCIF, or on a classified network's existing infrastructure. No AWS, no containers automatically pulled from registries, no CI/CD from GitHub.

2. **Manual data transfer.** Getting code or data into and out of a classified network requires physical media (approved CDs, DVDs, or encrypted portable devices) with formal review and approval processes. Some organizations allow "cross-domain solutions" for automated one-way transfers, but these are complex and tightly controlled.

3. **Limited development tools.** Many commercial development tools don't work (or aren't approved) on classified networks. If your development workflow depends on a dozen SaaS tools, none of them will be available in a SCIF.

4. **Separate development environments.** You'll typically develop on an unclassified system, then transfer code to the classified environment for integration and testing. This creates a dual-development-environment challenge that slows your release cycle significantly.

5. **Different security standards.** Classified IS use NIST 800-53 controls with the classified overlay from CNSSI 1253, not the commercial-focused NIST 800-171. The control set is larger and more restrictive.

---

## Should Your Startup Pursue Classified Work?

This is the most important strategic question in this chapter. Classified work offers extraordinary competitive advantages, but the costs are real.

### Reasons to Pursue Classified Work

**1. Competitive moat.** Once you have an FCL, SCIF, and cleared personnel, 95%+ of potential competitors are eliminated. Most startups can't or won't make the investment. The cleared defense industrial base is small and well-defined.

**2. Higher contract values.** Classified programs tend to be larger, better-funded, and more strategically important to the DoD. They're less subject to budget cuts because they address the most critical national security missions.

**3. Longer relationships.** Transitioning classified work is extremely difficult. If you're performing well on a classified program, re-competing that work is expensive and risky for the government—they'd rather extend your contract. This creates "stickiness" that unclassified work often lacks.

**4. Access to information.** Being cleared gives you access to the classified requirements and threat data that inform what the military actually needs. Many of the best opportunities are invisible to companies without clearances because the requirements themselves are classified.

**5. Premium pricing.** The overhead of classified operations justifies higher rates, and the limited competition means less downward price pressure.

### Reasons Not to Pursue Classified Work (Yet)

**1. Capital requirements.** An FCL, SCIF, FSO, cleared personnel, and classified IT infrastructure represent $500K–$2M+ in upfront investment and $200K–$500K+ in annual overhead before you earn a dollar of classified revenue.

**2. Talent constraints.** Every employee who works on classified material needs a clearance. Cleared software engineers command a 15–30% salary premium over their uncleared peers. And you can't hire non-citizens—period—which eliminates a significant portion of the software engineering talent pool.

**3. Development velocity.** Classified environments are slower. Code transfer is manual. Development tools are limited. You can't work from home (you can only access classified material in a SCIF). Your commercial release pace won't survive contact with the classified world.

**4. FOCI constraints.** If you have or want foreign investors, classified work creates permanent governance complications. You may need to choose between the VC firms that will fund your growth and the classified contracts that will drive your revenue.

**5. Distraction.** The infrastructure and compliance burden of classified work can consume a small company. If you're a 20-person startup and 5 people are managing your security program instead of building your product, you've undermined your core value proposition.

### The Recommended Approach

**Phase 1: Build unclassified first.** Win contracts, deliver software, build relationships with program offices. Prove your technology works on unclassified programs. This is faster, cheaper, and establishes the credentials you'll need.

**Phase 2: Position for classified.** Once you have revenue and relationships, start laying groundwork: hire an FSO, clean up your cap table, identify a sponsor, begin the FCL process. Use your existing government customers as sponsors.

**Phase 3: Enter classified incrementally.** Start with a partner SCIF arrangement or government-provided space. Take one classified contract, not ten. Build your classified infrastructure gradually as revenue justifies the investment.

**Phase 4: Scale classified operations.** As classified revenue grows, invest in your own SCIF, grow your cleared workforce, and potentially pursue TS/SCI capabilities.

**The exception:** If your technology specifically addresses a classified-only mission—if the problem you solve only exists at the classified level—you may need to pursue classified work from the start. In that case, build the classified infrastructure into your initial business plan and fundraise accordingly.

---

## The Cost of Going Classified

| Item | Cost | Frequency |
|---|---|---|
| Facility Security Officer (full-time) | $80K–$120K/year | Ongoing |
| Facility Security Officer (outsourced) | $24K–$60K/year | Ongoing |
| FCL processing | No direct cost (government-funded) | One-time |
| FOCI mitigation (legal structuring) | $50K–$250K+ | One-time + ongoing compliance |
| SCIF construction (small, 200 sq ft) | $200K–$400K+ | One-time |
| SCIF construction (medium, 1,000 sq ft) | $350K–$1M+ | One-time |
| SCIF accreditation (consulting + documentation) | $25K–$75K | One-time + $10K–$30K for re-accreditation every 5 years |
| Partner SCIF co-use agreement | Variable, often $2K–$10K/month | Ongoing |
| Classified IT (SIPRNet circuit + equipment) | $50K–$200K | One-time + $20K–$50K/year ongoing |
| Security clearance salary premium (per engineer) | 15–30% above market | Ongoing |
| Annual DCSA security reviews and compliance | $15K–$40K/year | Ongoing |
| Insider Threat Program | $10K–$30K/year | Ongoing |

**Total first-year estimated cost for a small startup entering classified work:** $400K–$1.5M+, depending on whether you build your own SCIF or use a partner's.

---

## Common Mistakes

**1. Not understanding FOCI until it's too late.** A foreign investor writes a check, and two years later you discover it blocks your FCL. Check your cap table, your investors' LP bases, and your board composition before pursuing classified work.

**2. Hiring the wrong FSO.** An inexperienced FSO who doesn't know DCSA procedures can delay your FCL by months or years. Hire someone with DCSA experience or outsource to a firm that specializes in startup FCLs.

**3. Building a SCIF too early.** Capital spent on SCIF construction is capital not spent on product development. Use partner or leased SCIFs until you have the classified revenue to justify the investment.

**4. Assuming clearances are fast.** A Secret clearance takes 4–6 months. A TS takes 8–12+ months. If you need cleared developers for a contract starting in 60 days, you're two quarters too late. Start the clearance process well before you need it.

**5. Trying to develop classified software like commercial software.** Classified environments don't support your normal CI/CD pipeline, your SaaS dev tools, or your remote-work culture. Plan for fundamentally different workflows.

**6. Ignoring NISPOM reporting requirements.** Cleared employees have ongoing reporting obligations: foreign travel, foreign contacts, financial changes, arrests, and more. Failure to report can result in clearance revocation and, in some cases, criminal liability.

---

## Key Terms

**Collateral Classified:** Information classified at the standard Confidential, Secret, or Top Secret levels, without additional SCI or SAP controls.

**SCI (Sensitive Compartmented Information):** A control system (not a classification level) placed on classified information derived from sensitive intelligence sources or methods. Requires specific compartment authorization and must be handled in a SCIF.

**SAP (Special Access Program):** Programs with access restrictions exceeding those for standard classified information. Can be acknowledged (existence is public) or unacknowledged (existence is classified/"black programs").

**PCL (Personal Security Clearance):** Individual-level determination of eligibility to access classified information. Levels: Confidential, Secret, Top Secret.

**SF-86:** Standard Form 86, the comprehensive personal history questionnaire submitted through e-QIP to initiate a security clearance investigation.

**DCSA (Defense Counterintelligence and Security Agency):** The agency responsible for processing security clearance investigations, managing facility clearances, and overseeing industrial security.

**DODCAF (DoD Consolidated Adjudication Facility):** The entity that makes clearance eligibility determinations based on completed investigations.

**FCL (Facility Clearance):** Administrative determination that a company is eligible to access classified information. Requires sponsorship, KMP clearances, NISPOM compliance, and FOCI resolution.

**DD Form 441:** The Department of Defense Security Agreement executed between a cleared contractor and the government, formalizing the company's obligation to protect classified information.

**DD Form 254:** Contract Security Classification Specification. Identifies the classification levels, access requirements, and security specifications for a classified contract.

**KMP (Key Management Personnel):** Senior company officials who must hold personal security clearances at the level of the company's FCL. Must be U.S. citizens.

**FSO (Facility Security Officer):** The designated official responsible for managing a cleared contractor's security program. Required for all cleared facilities.

**ITPSO (Insider Threat Program Senior Official):** The official responsible for the company's Insider Threat Program, required under NISPOM.

**FOCI (Foreign Ownership, Control, or Influence):** A condition where a foreign interest has the power to direct or decide matters affecting a U.S. company. Must be mitigated for FCL eligibility.

**SSA (Special Security Agreement):** A FOCI mitigation instrument that allows foreign participation in a cleared company under governance restrictions, including cleared U.S. outside directors and a Government Security Committee.

**VTA (Voting Trust Agreement):** The most restrictive FOCI mitigation instrument, requiring foreign owners to transfer legal title of shares to cleared U.S. trustees.

**Proxy Agreement (PA):** A FOCI mitigation instrument where foreign owners retain economic ownership but U.S. proxy holders assume operational control.

**NISPOM (National Industrial Security Program Operating Manual):** Codified as 32 CFR Part 117, the regulatory manual governing cleared contractor security programs.

**SCIF (Sensitive Compartmented Information Facility):** A physically secure facility accredited under ICD 705 for the storage, processing, and discussion of SCI.

**ICD 705:** Intelligence Community Directive 705, the standard governing SCIF construction, accreditation, and maintenance.

**TEMPEST:** A program and set of standards for controlling compromising electromagnetic emanations from electronic equipment going into and out of secured facilities.

**SIPRNet (Secret Internet Protocol Router Network):** The DoD's classified network for information up to the SECRET level.

**JWICS (Joint Worldwide Intelligence Communications System):** The IC's classified network for TOP SECRET/SCI information.

**NSANet:** The NSA's classified internal network.

**Continuous Vetting:** The Trusted Workforce 2.0 initiative replacing periodic reinvestigations with automated, ongoing monitoring of cleared personnel.

**Cross-Domain Solution:** A hardware/software system designed to transfer information between networks at different classification levels, subject to strict policies and review.

**Need-to-Know:** The determination that access to specific classified information is necessary for the performance of an individual's duties. A clearance alone does not grant access—need-to-know must also exist.
