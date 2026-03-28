# Chapter 1: The Landscape — How the Department of Defense Actually Buys Things

---

## Why This Chapter Exists

Before you pitch a single general, before you register on a single government website, before you write a single proposal, you need to understand the machine. The Department of Defense is the largest buyer on planet Earth. It spends more money annually than the GDP of most countries. And the way it spends that money is unlike anything you have ever encountered in the commercial world.

If you're coming from the startup ecosystem—where a deal can close over a Zoom call and a DocuSign in two weeks—the defense acquisition system is going to feel like you've been teleported to another planet. The vocabulary is different. The incentives are different. The timelines are measured in years, not quarters. And the rules aren't just corporate bureaucracy—they're *federal law*, backed by criminal penalties.

This chapter is the foundation for everything else in this book. It explains the basic architecture of how the DoD turns a military need into a funded contract. If you don't understand this system, nothing else will make sense. If you do understand it, you'll have an enormous advantage over the vast majority of startup founders who wander into this market thinking they can just "sell" their product to the Pentagon.

You can't. The Pentagon doesn't buy things the way normal organizations buy things. It runs an acquisition *system*—a bureaucratic machine with its own logic, its own language, and its own physics. Your job is to learn how the machine works so you can feed your technology into it.

---

## The Scale of the Machine

Let's start with the numbers, because the numbers are staggering.

### The Budget

The Department of Defense's budget request for Fiscal Year 2025 was **$849.8 billion**. The broader national defense budget—which includes the DoD plus the Department of Energy's nuclear weapons programs and other national security agencies—was approximately **$895 billion**.

For Fiscal Year 2026, the DoD budget request jumped to **$961.6 billion**, comprising $848.3 billion in discretionary funding and an additional $113.3 billion in mandatory funding secured through a reconciliation process. When you add in the National Nuclear Security Administration (NNSA) and other national security agencies, the total national defense topline breached **$1.01 trillion**.

One. Trillion. Dollars.

To put that in perspective: if the U.S. defense budget were a country's GDP, it would be roughly the 18th-largest economy in the world. It is larger than the entire GDP of Saudi Arabia, Switzerland, or Taiwan. Every year.

### How It Breaks Down by Service

The money doesn't flow through a single pipe. It's divided across the military services and defense agencies, each of which has its own acquisition infrastructure, its own priorities, and its own bureaucracies. Here's how the FY2026 request broke down:

| Component | FY2026 Budget Request |
|---|---|
| **Air Force** | $301.1 billion |
| **Navy** (includes Marine Corps) | $292.2 billion |
| **Army** | $197.4 billion |
| **Space Force** | $40 billion |
| **Missile Defense Agency** | ~$25.6 billion |
| **Cybersecurity investments** | $15.1 billion |
| **Defense Health Program** | $40.9 billion |

The Space Force's budget jumped more than 30% from FY2025 to FY2026—the fastest-growing service by percentage. That should tell you something about where the priorities are shifting.

### How It Breaks Down by Category

The defense budget isn't just one big pool of money. It's divided into distinct **appropriation titles**—categories of spending that cannot be freely moved between each other. This concept, called the **"color of money,"** is one of the most important things you will learn in this book, and it will come up repeatedly:

- **Military Personnel (MILPERS):** Pay and benefits for active-duty service members
- **Operations and Maintenance (O&M):** Day-to-day operating costs—fuel, maintenance, training, base operations
- **Procurement:** Buying actual things—aircraft, ships, vehicles, weapons, equipment
- **Research, Development, Test & Evaluation (RDT&E):** R&D, prototyping, testing
- **Military Construction (MILCON):** Building facilities, bases, infrastructure

Each color of money has its own rules. You cannot take RDT&E money (which funds prototypes and experiments) and use it to buy production units. You cannot take O&M money and use it for construction. This isn't a guideline—it is a legal constraint established by Congress. Violating the color of money rules is a criminal offense under the Antideficiency Act.

This matters enormously for startups because it means that even when a military unit loves your technology and wants to buy it, they may not have the right *kind* of money to do so. A program office that has RDT&E funds can pay you to do a prototype demonstration, but they literally cannot use that same money to buy 500 units for deployment. They need procurement money for that, and getting procurement money requires going through a completely separate process that is planned years in advance.

We'll come back to this concept many times. For now, just understand: not all DoD dollars are created equal, and the "color" of the money determines what it can be spent on.

---

## The Three Pillars: How Needs Become Funded Programs

The DoD acquisition system rests on three interlocking decision support systems. Think of them as three gears that must mesh together for anything to happen:

1. **JCIDS** — The Joint Capabilities Integration and Development System (defines *what* the military needs)
2. **PPBE** — Planning, Programming, Budgeting, and Execution (determines *how much money* is available and *where* it goes)
3. **DAS** — The Defense Acquisition System (manages *how* the thing actually gets built and delivered)

If any one of these gears stops turning, your technology goes nowhere. A brilliant capability without a validated requirement (JCIDS) can't get funded. A validated requirement without a budget line (PPBE) can't get built. A funded requirement without an acquisition program (DAS) has no mechanism to actually buy something. All three must align.

Let's take each one in turn.

---

### JCIDS: Where Requirements Come From

The Joint Capabilities Integration and Development System is the formal process by which the Department of Defense defines what capabilities it needs. It was created to replace the old service-specific requirements systems, which had a tendency to produce redundant, overlapping, and sometimes contradictory wish lists from each military branch.

The core idea behind JCIDS is **capabilities-based assessment**. Instead of starting with "we need a new tank," the process starts with "we have a gap in our ability to do X, and we need to figure out the best way to fill it." The answer might be a new tank. It might also be a drone, or a software system, or a change in doctrine, or nothing at all.

#### The JCIDS Document Chain

When the system works as intended, it produces a chain of documents that progressively define what the military needs:

**1. Initial Capabilities Document (ICD)**

The ICD is where it all starts. This document identifies a capability gap—something the military can't do, or can't do well enough—and proposes general approaches to fill it. The ICD doesn't specify a particular solution. It defines the problem. It says, essentially, "We need the ability to detect enemy submarines at greater ranges than we currently can. Here are some possible approaches."

The ICD supports the first major decision point in the acquisition process: **Milestone A**, which authorizes the start of technology development.

**2. Capability Development Document (CDD)**

The CDD takes the vague need defined in the ICD and turns it into specific, measurable requirements. This is where **Key Performance Parameters (KPPs)** are established—the hard metrics that a system must meet. For example: "The system must detect a submarine at a range of at least 50 nautical miles with a probability of detection of 0.85."

The CDD is approved by the **Joint Requirements Oversight Council (JROC)**, a four-star general/flag officer body chaired by the Vice Chairman of the Joint Chiefs of Staff. Getting JROC approval for your system's requirements is a major milestone—and convincing the JROC to include requirements that your technology can meet is one of the most important (and ethically sensitive) things a defense tech company can do.

The CDD supports **Milestone B**, the decision to enter full engineering development.

**3. Capability Production Document (CPD)**

The CPD finalizes the requirements for production and deployment. It captures everything needed to ensure that the system being produced actually meets the needs originally defined back in the ICD. The CPD supports **Milestone C**, the decision to enter production.

#### Why JCIDS Matters for Startups

Here's the brutal reality: if your technology doesn't match a validated JCIDS requirement, there is no official mechanism to buy it at scale. The military might love it. Individual generals might be clamoring for it. But without a validated requirement—an ICD, a CDD, or at minimum a recognized capability gap—the acquisition system has no legal basis to commit procurement dollars.

This is why understanding requirements generation is not optional for defense tech founders. You need to know which Combatant Commands are generating requirements. You need to know which PEOs (Program Executive Offices) are managing the programs that align with your technology. And in many cases, you need to actively—and legally—help the government understand that a capability gap exists and that your technology addresses it.

There is, of course, a significant caveat: JCIDS has been widely criticized as slow, bureaucratic, and poorly suited to rapidly evolving technologies like software and AI. Reform efforts have streamlined some of these requirements, and alternative acquisition pathways (which we'll cover in Chapter 6) can bypass the full JCIDS process. But the traditional system remains the primary pathway for large programs of record, and understanding it is essential.

**The critical exception for software companies:** If your product is software, you need to know that JCIDS was updated in FY2022 specifically to accommodate the Software Acquisition Pathway (DoDI 5000.87). Under this pathway, the full ICD → CDD → CPD document chain is *not required*. Instead, software programs use a **Capability Need Statement (CNS)**—a much simpler, higher-level document that captures the mission deficiency and desired capabilities without locking in detailed technical requirements years in advance. This single change eliminates one of the most hostile aspects of the traditional system for software companies: the requirement to freeze specifications before development begins. We'll cover the Software Acquisition Pathway in detail later in this chapter.

---

### PPBE: Where the Money Comes From

The Planning, Programming, Budgeting, and Execution system is how the Department of Defense allocates resources. It is the financial engine that determines which programs get funded, how much they receive, and for how long.

PPBE is a cyclical process that runs continuously. At any given moment, the DoD is simultaneously executing the current year's budget, defending next year's request to Congress, and planning the budget for *two to five years in the future*. This means that the money for your technology isn't decided today—it was decided two to three years ago, and the money for two years from now is being decided right now.

#### The Four Phases

**Phase 1: Planning**

The planning phase sets strategic direction. The Secretary of Defense and senior civilian/military leaders review national-level guidance—the National Security Strategy, the National Defense Strategy, the National Military Strategy—and translate it into defense-specific priorities. The output is the **Defense Planning Guidance (DPG)**, which tells each military department and defense agency what the priorities are and what trade-offs they should be making.

For startups, what matters here is that the DPG reflects *political* priorities as much as military ones. If autonomous systems are a priority this year, money will flow toward autonomous systems. If the focus shifts to shipbuilding or nuclear modernization, other areas get squeezed. Knowing where the strategic winds are blowing gives you a sense of where the money will go.

**Phase 2: Programming**

This is where strategic goals get converted into specific program line items with attached dollar amounts. Each DoD component develops a **Program Objective Memorandum (POM)**—a detailed, multi-year resource plan that says "Here is everything we want to spend money on over the next five years, and here's how much we need."

The POM is where the real fights happen. Services compete against each other. Programs within services compete against each other. If your technology is associated with a program that gets cut in the POM, your contract evaporates—even if everyone agrees the technology is great. The POM is managed by the **Director of Cost Assessment and Program Evaluation (CAPE)**, one of the most powerful and least-known offices in the Pentagon.

**Phase 3: Budgeting**

The budgeting phase translates the POM into the immediate fiscal year's budget request. Each component submits a **Budget Estimate Submission (BES)**, which scrutinizes the first one to two years of the POM in exhaustive detail. The result becomes the DoD's portion of the President's annual budget request, which is submitted to Congress.

**Phase 4: Execution**

Execution is what happens after Congress appropriates the money. Programs spend their allocated funds, and the DoD monitors whether the money is being used effectively. If significant changes occur—a program goes over budget, a technology fails testing, or a new threat emerges—adjustments require going back through the process or seeking congressional approval for reprogramming.

#### Why PPBE Matters for Startups

The critical insight is the **timeline**. The PPBE cycle plans budgets two to three years in advance. If it is March 2026 and your technology is ready to sell today, the money to buy it was already allocated in the FY2026 budget, which was planned during the FY2024 POM cycle—meaning the decisions about what to fund were being made in 2023.

If your technology wasn't on anyone's radar in 2023, it is almost certainly not in the FY2026 budget. To get into the FY2028 budget, you need to be influencing the POM discussions that are happening *right now*.

This is why the Valley of Death (Chapter 12) is so deadly. Even when the military wants your technology, the money to buy it may not exist for another two to three years because the budget was locked in before anyone knew your company existed. Alternative pathways like SBIR, OTAs, and the Rapid Defense Experimentation Reserve exist specifically to short-circuit this timeline—but they have their own limitations.

---

### DAS: The Defense Acquisition System

The Defense Acquisition System is the formal process that manages how the DoD develops, tests, produces, and sustains military systems. It's governed by **DoD Directive 5000.01** and **DoD Instruction 5000.02**, and it has been reformed and reorganized more times than anyone can count.

#### The Acquisition Lifecycle

The traditional acquisition lifecycle is a phased, milestone-driven process. Each phase has specific exit criteria that must be met before a program can advance to the next phase. Here are the phases:

**Phase 1: Materiel Solution Analysis (MSA)**

Before anything gets built, the DoD conducts an analysis of alternatives. What are the possible solutions to the capability gap identified through JCIDS? Should the military modify an existing system? Buy something off the shelf? Develop something new? The MSA phase evaluates options and recommends a path forward.

This phase ends at **Milestone A**, which authorizes the program to enter technology development.

**Phase 2: Technology Maturation & Risk Reduction (TMRR)**

This is where the hard engineering questions get answered. Can the technology actually work? What are the risks? During TMRR, the program develops prototypes, conducts tests, and reduces the technical uncertainty to a level where the DoD is comfortable committing to full development.

This phase ends at **Milestone B**—arguably the most important decision point in the entire process, because it's where the DoD decides to commit serious money. After Milestone B, the program becomes a **Program of Record** with dedicated funding in the Future Year Defense Program. Milestone B is when a program goes from "interesting experiment" to "real."

**Phase 3: Engineering & Manufacturing Development (EMD)**

This is full-scale development. The system is engineered, tested, and prepared for production. Developmental and operational testing verifies that the system meets its KPPs. If it doesn't, the program can be sent back for more development or, in the worst case, terminated.

This phase ends at **Milestone C**, the decision to begin production.

**Phase 4: Production & Deployment**

The system goes into production—initially as **Low-Rate Initial Production (LRIP)** to validate the manufacturing process, and then **Full-Rate Production (FRP)** once operational testing confirms the system works. The system is fielded to military units.

**Phase 5: Operations & Support (O&S)**

The system is in the field. This phase covers everything from maintenance and upgrades to eventual disposal. For technology companies, this phase matters because sustainment contracts—keeping the system running, updated, and supported—can be worth more than the original development contract and can last decades.

---

## The Gatekeepers: Milestones, the DAB, and ACAT Categories

### Milestones

Milestones A, B, and C are not just formalities. They are formal decision points where a **Milestone Decision Authority (MDA)** reviews all available evidence—cost estimates, test results, risk assessments, schedule status—and decides whether the program is allowed to proceed to the next phase.

The MDA can approve the program, approve it with conditions, or kill it. These reviews involve extensive documentation, briefings, and often months of preparation. For major programs, the milestone review process alone can take six months to a year.

### The Defense Acquisition Board (DAB)

For the largest and most important programs, the MDA is the **Under Secretary of Defense for Acquisition and Sustainment (USD(A&S))**, and the milestone review is conducted by the **Defense Acquisition Board**. The DAB is the senior advisory board for defense acquisitions, and its membership reads like the Pentagon's starting lineup:

- The Vice Chairman of the Joint Chiefs of Staff
- The Service Secretaries
- Multiple Under Secretaries of Defense
- The Director of CAPE
- The Director of Operational Test & Evaluation (DOT&E)

Getting past the DAB is a major accomplishment. Being killed by the DAB is a major career and corporate event. Entire programs have been restructured, delayed, or terminated based on DAB reviews.

### ACAT Categories

Not every program goes through the full DAB gauntlet. The level of oversight depends on the program's **Acquisition Category (ACAT)**:

**ACAT I — Major Defense Acquisition Programs (MDAPs)**

These are the big ones. A program is designated ACAT I if its estimated RDT&E cost exceeds approximately **$525 million** or its estimated procurement cost exceeds approximately **$3.065 billion** (in FY2020 constant dollars). Think fighter jets, aircraft carriers, missile defense systems.

ACAT I programs are reviewed by the DAB. They receive the highest level of scrutiny and oversight. They also receive the most congressional attention, the most press coverage, and the most political protection.

ACAT I is further subdivided:
- **ACAT ID:** Milestone Decision Authority is the Defense Acquisition Executive (the USD(A&S))
- **ACAT IB:** Milestone Decision Authority is the Service Acquisition Executive
- **ACAT IC:** Milestone Decision Authority is the Component Acquisition Executive

**ACAT II — Major Systems**

These are significant programs that don't meet the ACAT I cost thresholds but still have substantial costs and importance. They are typically reviewed by the **Component Acquisition Executive (CAE)**—the senior acquisition official within a military service or defense agency.

**ACAT III — Lower-cost Programs**

These are smaller programs with lower costs and less complexity. They are reviewed at the component level or below. For startups, ACAT III programs are often the most accessible entry point—they have less oversight, faster timelines, and more flexibility for the program manager to make decisions.

### Why ACAT Matters for Startups

ACAT level determines how long everything takes, how many people must approve every decision, and how much flexibility program managers have to work with new suppliers. An ACAT I program manager is surrounded by oversight bodies, review boards, and congressional reporting requirements. An ACAT III program manager has far more autonomy.

For a startup, targeting an ACAT III program or an activity below the formal ACAT threshold (such as a rapid prototyping project or an innovation initiative) is dramatically faster and less bureaucratic than trying to get onto a major ACAT I program of record.

---

## Programs of Record vs. Everything Else

This distinction is critical, and most startup founders don't understand it.

### What Is a Program of Record?

A **Program of Record (POR)** is an acquisition program that has received official approval—typically at Milestone B—and has dedicated, multi-year funding allocated in the Future Year Defense Program (FYDP). It's a "line item" in the defense budget. It has a program manager, a program office, a budget line, congressional oversight, and a mandate to develop and deliver a specific capability.

Becoming part of a POR is the gold standard for a defense contractor. It means you have a funded, long-term relationship with the government. It means your technology is on a trajectory toward production and deployment. It means the government has invested institutional credibility in your program.

It also means the program is subject to the full weight of the acquisition system—ACAT reviews, milestone decisions, JCIDS requirements, Congressional reporting. Getting to POR status typically takes 5-12 years from initial concept.

### What Isn't a Program of Record

Everything else. And there's a lot of "everything else":

- **Prototypes** — Technology demonstrations, proof-of-concept projects, engineering experiments. Funded with RDT&E money but with no commitment to production.
- **Pilot programs** — Limited-scope evaluations where a unit tries out a technology in an operational environment. Often funded with O&M money or through innovation programs.
- **Experiments and demos** — Science & technology (S&T) activities run by research labs or organizations like DARPA. These are early-stage investigations, not procurement.
- **Quick-reaction capabilities** — Technologies fielded rapidly in response to urgent operational needs. These can move fast but often lack the sustained funding of a POR.
- **SBIR Phase I and Phase II awards** — Small research contracts that fund technology development but carry no commitment to future acquisition.
- **Middle Tier Acquisition programs** — A relatively new category (created by Section 804 of the FY2016 NDAA) that sits between prototypes and full PORs.

### The Middle Tier: Section 804 and the Adaptive Acquisition Framework

Recognizing that the traditional JCIDS-PPBE-DAS pipeline was too slow for rapidly evolving technologies, Congress created the **Middle Tier of Acquisition (MTA)** through Section 804 of the FY2016 NDAA. MTA programs are designed to be completed within **2-5 years**—dramatically faster than traditional acquisitions.

MTA has two sub-pathways:

**Rapid Prototyping:** Develop and field a prototype that demonstrates new capabilities within five years. The prototype should provide a *residual operational capability*—meaning it's not just a lab demo, it's something troops can actually use.

**Rapid Fielding:** Take proven technology and field production quantities with minimal additional development within five years, with production beginning within six months of approved requirements.

MTA programs are intentionally exempt from many of the traditional JCIDS and DAS requirements, giving program managers much more flexibility. Successful MTA prototypes can transition into:
- Operational use directly
- The Rapid Fielding pathway for faster production
- A traditional Program of Record for sustained production and support

The broader reform that MTA sits within is the **Adaptive Acquisition Framework (AAF)**, which replaced the old one-size-fits-all acquisition pipeline with **six distinct pathways**:

1. **Urgent Capability Acquisition** — For capabilities needed in less than 2 years
2. **Middle Tier Acquisition** — For capabilities that can be delivered in 2-5 years
3. **Major Capability Acquisition** — The traditional milestone-driven process for large, complex systems
4. **Software Acquisition** — A pathway specifically designed for software-intensive systems, emphasizing agile development and continuous delivery
5. **Defense Business Systems** — For IT systems that support business operations (finance, HR, logistics)
6. **Acquisition of Services** — For service contracts

For startups, the AAF is good news. It means you're not locked into the traditional 10-15 year pipeline. Depending on your technology and the urgency of the need, there are faster pathways available. But you need to know which pathway fits your situation—and more importantly, you need to help your government customer navigate the system to get your technology on the right pathway.

**If you're a software company, the most important thing in the list above is #4.** The Software Acquisition Pathway (DoDI 5000.87) was designed explicitly to solve the problems that make the traditional system hostile to software. It was issued in October 2020, and by March 2025, a SecDef memorandum mandated it as the *preferred* method for all DoD software programs. It deserves its own deep dive.

---

## The Software Acquisition Pathway: DoDI 5000.87

If you are building software for the military, this section may be the most important pages in this entire book.

DoDI 5000.87 created the first acquisition pathway designed specifically for software-intensive systems. It represents a fundamental philosophical break with the traditional acquisition system described above. Where the traditional system is linear, milestone-driven, and requirements-locked, the Software Acquisition Pathway is iterative, user-driven, and designed for continuous delivery.

Let's break down how it works and why it matters.

### Why Software Needed Its Own Pathway

The traditional acquisition system was designed to buy hardware—aircraft, tanks, ships, missiles. These are systems where you spend years designing, years building, and then decades maintaining a fixed product. The sequential phase model (MSA → TMRR → EMD → Production → O&S) makes sense when you're building something that takes years to manufacture and decades to operate.

Software doesn't work this way. Software can be updated in minutes. Requirements evolve continuously. The concept of a "final" software product is meaningless—software is either being actively developed and improved, or it's dying. Forcing software through a hardware acquisition pipeline produces absurd results: by the time a software system completes the traditional 12-year acquisition cycle, the technology it was built on is two generations obsolete, the threat environment has changed, and the operational need has evolved beyond recognition.

DoDI 5000.87 was created to fix this. And as of March 2025, the Secretary of Defense has directed that it is no longer just an option—it is the **default and preferred pathway** for acquiring software capabilities across the entire Department of Defense.

### The Two-Phase Structure

The Software Acquisition Pathway has only **two phases**—a dramatic simplification from the traditional five-phase lifecycle:

**Phase 1: Planning**

During the Planning phase, the program defines what problem it's solving and how it will organize to solve it. The key outputs are:

- **Capability Need Statement (CNS):** A high-level document that captures the mission deficiency and desired capabilities—*without* locking in detailed technical requirements. The CNS replaces the ICD/CDD/CPD document chain from JCIDS. It focuses on *what* the warfighter needs to accomplish, not *how* the software should accomplish it. This is a massive difference. The CNS is reviewed periodically (at least as often as value assessments) to reflect evolving operational needs—meaning requirements *can change* without triggering a bureaucratic nightmare.

- **User Agreement (UA):** A formal agreement between the sponsor (the organization that needs the capability) and the program manager. The UA commits both parties to continuous user involvement throughout development. It defines who has decision-making authority for capability releases, how trade-offs will be managed, and what the roles and responsibilities are. Think of it as a lightweight charter that replaces the elaborate Acquisition Strategy documents required by traditional programs.

- **Cost estimate and acquisition strategy:** The program develops an initial cost estimate and determines how it will be organized, funded, and contracted.

Planning is designed to be fast. The entire purpose is to get to execution as quickly as possible.

**Phase 2: Execution**

Execution is where the software is built, delivered, and continuously improved. This is an **ongoing, iterative process**—not a phase that you "complete" and exit. The Execution phase continues for the life of the software.

During Execution:
- Software teams use **Agile and Lean development methodologies** to deliver capability in regular iterations (sprints, typically 2-4 weeks)
- Software is delivered to users continuously—the expectation is that users are receiving working software, not PowerPoint briefings about software that will eventually be ready
- Teams operate within a **DevSecOps pipeline** that integrates development, security, and operations so that security testing happens automatically with every code commit rather than as a separate, months-long gate review
- **No traditional milestones.** There is no Milestone A, B, or C. There is no Defense Acquisition Board review. The Decision Authority (which can be as low as the Component Acquisition Executive, rather than the USD(A&S)) approves entry into Execution and then monitors progress through Value Assessments rather than milestone reviews.

### Value Assessments: How Software Programs Are Monitored

Instead of milestone reviews, the Software Acquisition Pathway uses **Value Assessments**—conducted at least annually after initial fielding. Value Assessments evaluate:

- Whether the software is delivering useful capability to users
- Whether the mission improvements justify the investment
- Whether the program should continue, be expanded, be reduced, or be terminated

This is a radical concept within the DoD acquisition system. Instead of asking "Did you meet the 200 requirements we specified three years ago?", Value Assessments ask "Is this software actually helping warfighters do their jobs better?" The focus shifts from compliance with specifications to *delivered value*—which is exactly how software is evaluated in the commercial world.

Value Assessments incorporate feedback from actual end users—the soldiers, sailors, airmen, and Marines who use the software daily. This is a stark contrast to the traditional system, where the people defining requirements (JROC generals) and the people reviewing milestones (DAB members) may be several organizational layers removed from the people who actually use the system.

### What This Pathway Eliminates

To appreciate how significant DoDI 5000.87 is, consider what it removes from the traditional process:

| Traditional Acquisition | Software Acquisition Pathway |
|---|---|
| Full JCIDS document chain (ICD → CDD → CPD) | Capability Need Statement (one document) |
| Requirements locked before development | Requirements evolve continuously |
| Three milestone gates (A, B, C) | No traditional milestones |
| DAB review for large programs | Value Assessments (annual) |
| Sequential phases over 5-15 years | Two phases; Execution is continuous |
| Testing as a separate phase (DT&E, OT&E) | Testing integrated into DevSecOps pipeline |
| ATO as a point-in-time gate | Continuous ATO (cATO) as the target model |
| Delivery at the end | Continuous delivery throughout |

### Why the March 2025 SecDef Memorandum Is a Game-Changer

When DoDI 5000.87 was first issued in October 2020, it was just one pathway among six. Program offices could use it, but they could also default to the traditional Major Capability Acquisition pathway for software—and many did, because the traditional system was what they knew.

The March 2025 Secretary of Defense memorandum changed the calculus. The memo directed that:

1. The Software Acquisition Pathway is the **preferred and default method** for acquiring all software capabilities across the DoD
2. Software programs should use **Commercial Solutions Openings (CSOs) and Other Transaction Authorities (OTAs)** as the default contracting mechanisms—not traditional FAR-based contracts
3. Services and agencies must provide justification for *not* using the Software Acquisition Pathway for software-intensive programs

This is extraordinary. For the first time, the burden of proof has shifted: instead of needing to justify why you're using the new pathway, program offices now need to justify why they're *not* using it. For software startups, this means the entire DoD acquisition system is officially being directed toward the processes and contracting mechanisms (OTAs, CSOs) that are most accessible to commercial and nontraditional companies.

### What This Means for Software Startups

The Software Acquisition Pathway creates several specific advantages for you:

**1. Shorter timelines.** Without three milestone gates and a DAB review, programs can move from concept to fielded capability in months rather than years. Software teams can start delivering working software to users almost immediately.

**2. Requirements that match reality.** The CNS model means your software doesn't need to meet 200 frozen requirements from three years ago. It needs to deliver value to actual users—something that Agile teams do naturally.

**3. Contracting mechanisms you can actually use.** The SecDef memo's default to OTAs and CSOs means program offices are being pushed toward the contracting mechanisms that don't require you to have a cost accounting system, DCAA-approved rates, or years of government-specific past performance.

**4. Continuous delivery is the expectation.** The traditional system expected you to deliver a finished product after years of development. The Software Acquisition Pathway expects continuous iterations—which is how every modern software company operates.

**5. Value-based evaluation.** Value Assessments measure whether your software helps warfighters, not whether you checked compliance boxes. If you build software that users love, you'll pass Value Assessments. If you build software that meets every requirement but nobody uses, you'll fail them.

**The catch:** You still need an Authority to Operate (ATO) to deploy your software on DoD networks. The ATO process remains a significant challenge—taking 9-18 months through traditional channels—and is covered extensively in Chapter 8. However, the Software Acquisition Pathway's emphasis on DevSecOps and Continuous ATO (cATO) is designed to transform the ATO from a one-time gate into an ongoing, automated process that integrates with your development pipeline.

You also still need to navigate the PPBE cycle for sustained funding, find a champion within the program office, and build the cybersecurity compliance infrastructure (CMMC, etc.) described in later chapters. DoDI 5000.87 changes *how* the DoD buys software—it doesn't change the fact that the government still takes its time spending money.

---

## The Contract Types: How Money Actually Changes Hands

Once a program office decides to buy something, it issues a contract. But not all contracts are alike. The type of contract determines how risk is shared between the government and the contractor, how you get paid, and what accounting systems you need.

### Firm-Fixed-Price (FFP)

The simplest concept, and the government's preferred contract type. The price is set before work begins and does not change, regardless of your actual costs. If you bid $2 million for a deliverable and it costs you $1.5 million to produce, you pocket $500,000 in profit. If it costs you $3 million, you eat the $1 million loss.

**Who bears the risk:** The contractor. You are responsible for all cost overruns.

**When it's used:** When the requirements are well-defined and the costs can be accurately estimated. Common for commercial items, manufacturing contracts, and services with clear deliverables.

**What it means for startups:** FFP contracts are the most straightforward, but they can be dangerous if you underestimate costs. The government will not bail you out if you bid too low. On the other hand, FFP contracts require the least burdensome accounting systems—you don't need to track every hour and every dollar against government-approved cost categories.

### Cost-Plus (Cost-Reimbursement)

The government pays you for all **allowable** costs incurred in performing the contract, plus a fee. The key word is "allowable"—not all business costs qualify. Alcohol, entertainment, most lobbying expenses, first-class travel, and many other costs the government deems unreasonable are unallowable under FAR Part 31 and will not be reimbursed.

There are several flavors:

**Cost-Plus-Fixed-Fee (CPFF):** The government reimburses your allowable costs and pays a fixed fee (profit) that doesn't change even if costs increase. This is the most common cost-reimbursement type. The fee is set at contract award and stays constant.

**Cost-Plus-Incentive-Fee (CPIF):** The fee varies based on whether you hit cost or performance targets. Beat the target cost, and your fee increases. Miss it, and your fee decreases. This creates a financial incentive to control costs while still protecting you from catastrophic overruns.

**Cost-Plus-Award-Fee (CPAF):** The fee is determined by the government's subjective evaluation of your performance. A fee evaluation board reviews your work and assigns a score. Higher scores mean higher fees. This gives the government a powerful tool to reward (or punish) contractor performance, but the subjectivity can be frustrating.

**Who bears the risk:** The government bears most of the cost risk. You get reimbursed for actual costs, so cost overruns don't come out of your profit.

**When it's used:** When the work is complex, uncertain, or research-oriented—situations where it's genuinely impossible to estimate costs accurately in advance. R&D contracts are almost always cost-reimbursement.

**What it means for startups:** Cost-plus contracts require a fully compliant accounting system. You need to track direct costs, indirect costs, overhead rates, fringe rates, and G&A rates. You need a timekeeping system. You will be audited by DCAA. This accounting infrastructure is expensive to build (see Chapter 7), and it's one of the biggest barriers to entry for startups in defense.

### Time & Materials (T&M)

You get paid for actual labor hours at pre-negotiated hourly rates, plus the actual cost of materials. There is usually a **not-to-exceed (NTE)** ceiling on the total contract value, but you're paid based on hours worked, not on deliverables completed.

**Who bears the risk:** The government. T&M contracts offer the least incentive for contractor efficiency because you get paid for every hour worked, regardless of productivity. The FAR actually states that T&M contracts should be used only when no other contract type is suitable.

**When it's used:** When the scope of work is too uncertain to define deliverables or estimate fixed prices. Common for IT support, consulting, maintenance, and emergency response.

**What it means for startups:** T&M contracts can be a useful entry point because they're relatively simple and don't require deliverables to be precisely specified upfront. However, they also tend to be lower-value and don't build toward a sustainable, scalable business.

### Indefinite Delivery/Indefinite Quantity (IDIQ)

An IDIQ isn't really a contract type—it's a **contract vehicle**. It establishes an umbrella agreement with a specified minimum and maximum dollar value over a multi-year period (typically 5-10 years). The actual work is ordered through individual **task orders** (for services) or **delivery orders** (for supplies), each of which can be priced as FFP, cost-plus, or T&M.

IDIQs are enormously important in the defense world. Many of the largest contract vehicles—DISA's SETI, GSA's Alliant 2, the Navy's SeaPort—are IDIQs. If you hold a position on a major IDIQ, you have a hunting license to compete for task orders across the government for years.

**When it's used:** When the government knows it needs a category of goods or services but can't predict exact quantities or timing. The IDIQ provides a pre-competed, pre-approved pool of vendors from which the government can quickly order specific work.

**What it means for startups:** Getting on an IDIQ vehicle can be transformative, but it's also a significant investment of time and proposal effort. Winning a position on an IDIQ merely means you're eligible to bid on task orders—you still have to compete for the actual work. But the barrier to entry for individual task orders is much lower than competing for standalone contracts.

### The Contract Type Spectrum

Think of contract types as a spectrum of risk:

```
                    RISK →

     ┌──────────┐    ┌──────────┐    ┌──────────┐
     │   FFP    │    │   T&M    │    │ Cost-Plus │
     │          │    │          │    │           │
     │Contractor│    │  Shared  │    │Government │
     │bears risk│    │   risk   │    │bears risk │
     └──────────┘    └──────────┘    └──────────┘
```

The government always prefers FFP because it shifts risk to the contractor and provides cost certainty. You will see this repeatedly: the pressure to accept FFP even when the work is genuinely uncertain. Resisting this pressure when appropriate—and having enough expertise to explain why a cost-reimbursement structure is more appropriate for R&D work—is one of the most important skills a defense tech founder can develop.

---

## Why the System Is Slow (and Why Reform Keeps Failing)

You now have a basic understanding of how the machine works. But here's the uncomfortable truth: the machine is broken. Everyone knows it's broken. And it has been broken—in basically the same ways—for over forty years.

### The Fundamental Problems

**1. The 12-Year Problem**

According to the Government Accountability Office (GAO), the average time to deliver the first version of a major weapon system is nearly **12 years**. Twelve years from "we need this" to "here it is." In the technology world, 12 years is an eternity. The iPhone didn't exist 12 years before the iPhone. Companies that receive an SBIR Phase I today may not see production contracts until the 2030s.

**2. Rigid, Sequential Processes**

The traditional acquisition lifecycle is linear and sequential. You complete one phase, get milestone approval, and enter the next phase. This made sense in the 1960s when the DoD was buying aircraft carriers and nuclear submarines—systems so complex that extensive upfront planning was essential. It makes no sense for software, AI, or rapidly evolving digital technologies that are obsolete by the time they complete testing. The Software Acquisition Pathway (DoDI 5000.87, covered earlier in this chapter) was created specifically to address this problem for software—but cultural inertia means many program offices still default to the traditional process.

**3. Requirements Lock-In**

JCIDS requires detailed requirements to be defined early and approved by senior leaders before development begins. Once approved, changing requirements is an enormous bureaucratic undertaking. This means that by the time a system is built to spec and fielded, the requirements it was built to meet may no longer reflect the actual operational need. Technology has moved on. Threats have evolved. But the system delivers what was specified, whether anyone still needs it or not.

**4. Cost Overruns and Schedule Slips**

Major defense acquisition programs have historically experienced cost growth of **40-60%** over their original estimates. The F-35 Joint Strike Fighter—the most expensive weapons program in history—is over $180 billion above its original cost estimate. These overruns consume funds that could go to other programs and erode confidence in the acquisition system.

**5. The Bureaucracy Tax**

FAR and DFARS together span thousands of pages. Compliance with these regulations requires armies of lawyers, accountants, and compliance specialists. For large prime contractors with dedicated compliance infrastructure, this is a cost of doing business. For startups, it's a barrier to entry that swallows resources that should go to building technology.

**6. Risk Aversion**

The system punishes failure more than it rewards success. A program manager who takes a risk on new technology and fails faces career consequences. A program manager who sticks with the proven-but-inferior approach and delivers on time faces no consequences. This asymmetry creates an institutional bias toward the status quo—exactly the opposite of what the DoD needs in an era of rapid technological change.

### The Reform Parade

The problems are well-documented, and the reform efforts are equally well-documented:

**The Packard Commission (1986)**

Formally known as the President's Blue Ribbon Commission on Defense Management, the Packard Commission was established by President Reagan to study management dysfunction within the DoD. Its core finding: the problem wasn't fraud or individual bad actors. It was **"overcomplicated organization and rigid procedure."**

The Commission recommended creating an Under Secretary of Defense for Acquisition—a "procurement czar" with centralized authority over the buying process. That position was created and still exists today (now the USD(A&S)). But studies have shown that the Packard Commission's reforms did not consistently reduce cost overruns, and in some cases, cost performance on development contracts actually worsened.

**The Goldwater-Nichols Act (1986)**

Passed the same year as the Packard Commission report, Goldwater-Nichols was a sweeping reorganization of the entire DoD—not just acquisition. It strengthened the authority of the Chairman of the Joint Chiefs, empowered Combatant Commanders, and attempted to fix the "inefficient, unwieldy, and at times corrupt" weapons acquisition system.

Goldwater-Nichols is generally considered the most important defense reorganization since the National Security Act of 1947. But its impact on acquisition specifically has been debated. Some argue it created an unintended split between requirements (controlled by the military through JCIDS) and acquisition (controlled by civilians through the DAS), leading to disconnects that persist to this day.

**Better Buying Power (2010-2015)**

Introduced by Under Secretary of Defense for Acquisition Ashton Carter (who later became Secretary of Defense), Better Buying Power was a series of initiatives aimed at getting more capability per dollar. BBP 1.0, 2.0, and 3.0 emphasized affordability, competition, cost control, and innovation.

BBP was more pragmatic than previous reform efforts—it focused on specific, actionable changes rather than grand reorganizations. But it was also implemented unevenly, and some offices treated it as a rigid directive rather than flexible guidance.

**Section 804 and the Adaptive Acquisition Framework (2016-present)**

The most recent—and arguably most significant—reform wave. Congress created the Middle Tier of Acquisition through Section 804, and the DoD subsequently developed the Adaptive Acquisition Framework with six distinct pathways. This is the first time the system has genuinely acknowledged that different types of acquisitions need different processes.

The AAF is still being implemented and refined. Early signs are promising—programs using MTA and the Software Acquisition Pathway have moved significantly faster than traditional programs. But cultural resistance remains strong, and many program offices default to the traditional major capability acquisition pathway even when faster alternatives exist. The March 2025 SecDef memorandum (discussed earlier) is the most aggressive push yet to break this inertia for software specifically—mandating the Software Acquisition Pathway as the default and directing the use of OTAs and CSOs as the preferred contracting mechanisms.

**2025 and Beyond: The DOGE Effect and Executive Pressure**

The current political environment in 2025-2026 has added new pressure on acquisition reform. Executive orders emphasizing speed, commercial solutions, and reduced bureaucracy are pushing the system to move faster. The creation of the Department of Government Efficiency (DOGE) has intensified scrutiny of Pentagon spending. How these pressures ultimately reshape the acquisition system remains to be seen—but the direction is clear: faster, leaner, more commercial.

---

## How Requirements Flow from Warfighter to Solicitation: The Practical Pipeline

Let's put all of this together into a practical sequence—the real journey from "a soldier has a problem" to "a contractor gets paid to solve it."

### Step 1: A Military Unit Identifies a Problem

A platoon leader in the 82nd Airborne notices that enemy drones are a persistent threat during operations and current countermeasures are inadequate. This is not yet a formal requirement—it's an operational observation. It might be captured in an after-action report, mentioned at a Capabilities Development Workshop, or raised through a Combatant Command's annual requirements review.

### Step 2: The Problem Gets Formalized

The problem works its way up through military channels. Perhaps the Army's Training and Doctrine Command (TRADOC) conducts a formal Capabilities-Based Assessment and determines that the Army has a gap in low-altitude counter-UAS (C-UAS) capability. An **Initial Capabilities Document** is drafted.

### Step 3: Requirements Are Validated

The ICD is reviewed through JCIDS. The Joint Requirements Oversight Council examines it and validates that, yes, this is a real capability gap that affects the joint force. A requirement is formally established.

### Step 4: Funding Is Programmed

The Army's POM submission for the upcoming PPBE cycle includes a request for RDT&E funding to explore C-UAS solutions. After extensive review by CAPE and OSD, the funding is approved and incorporated into the President's budget request to Congress.

### Step 5: Congress Appropriates the Money

Congress reviews the budget request, conducts hearings, marks up the NDAA, and eventually passes an appropriations bill that includes funding for C-UAS research. The money is now available—but only in the specific appropriation title and for the specific purpose Congress authorized.

### Step 6: The Acquisition System Engages

A Program Executive Office—in this case, perhaps PEO Missiles & Space—is assigned responsibility for the C-UAS program. A program manager is appointed. The program enters the acquisition lifecycle, starting with Materiel Solution Analysis.

### Step 7: Industry Is Engaged

The program office conducts market research, possibly issuing a **Request for Information (RFI)** to learn what solutions are available. It might hold an Industry Day where companies can learn about the requirement and ask questions. It might also coordinate with innovation organizations like DIU or the Army Applications Lab to identify commercial technologies that could address the gap quickly.

### Step 8: A Solicitation Is Issued

Based on the acquisition strategy and the type of contract selected, the program office issues a formal solicitation—an RFP (Request for Proposal), a BAA (Broad Agency Announcement), a CSO (Commercial Solutions Opening), or an SBIR topic. Companies respond with proposals.

### Step 9: Evaluation and Award

Proposals are evaluated by a Source Selection Evaluation Board. For competitive solicitations, the government evaluates technical approach, management plan, past performance, and cost/price. The contracting officer awards the contract to the company or companies that offer the best value.

### Step 10: Performance and Delivery

The contractor performs the work, delivers the system, and gets paid. If the contract is cost-reimbursement, the contractor bills actual costs periodically. If it's FFP, payment is tied to deliverables or milestones.

This entire process—from Step 1 to Step 10—can take **3 to 15 years** for a traditional acquisition. For programs using MTA, the timeline might be 2-5 years. For urgent capabilities, it might be under 2 years. For SBIR Phase I awards, the initial solicitation-to-award cycle might be 3-6 months.

---

## What This Means for You

If you've read this far, you should be developing a few realizations:

**1. The system is not designed for you.** It was designed for Lockheed Martin, Raytheon, and Boeing—companies with thousands of employees, dedicated government compliance infrastructure, and relationships that span decades. Every simplification, every alternative pathway, every innovation initiative is an attempt to make the system less hostile to smaller, newer companies. But the underlying architecture still favors incumbents.

**2. Speed is your enemy and your friend.** The system is slow, which means you'll burn cash waiting for it. But if you can navigate the alternative pathways—OTAs, SBIRs, Middle Tier Acquisition—you can move faster than the traditional system allows. And if you can survive the timeline, you can build a position that incumbents find hard to dislodge.

**3. Understanding the system IS the competitive advantage.** Most startup founders who enter defense contracting learn about the acquisition system by running headfirst into it and getting confused, frustrated, and eventually broke. The founders who succeed are the ones who study the system, understand its logic, identify the pressure points, and navigate deliberately. That's what the rest of this book is for.

**4. The money is real—and it's massive.** A trillion dollars a year, every year. The opportunity is not theoretical. But accessing it requires a level of patience, strategic sophistication, and regulatory compliance that is completely alien to the startup world.

**5. The system is changing—and for software companies, it's changing faster than for anyone else.** The Adaptive Acquisition Framework, Middle Tier Acquisition, the Replicator Program—these are real reforms creating genuine openings for new companies. But the Software Acquisition Pathway (DoDI 5000.87) and the March 2025 SecDef memorandum represent something more: a top-down mandate that fundamentally restructures how the DoD buys software. OTAs and CSOs as default contracting mechanisms. Value Assessments instead of milestone reviews. Continuous delivery instead of waterfall development. For the first time, the acquisition system is officially being bent toward how modern software companies actually work. If there was ever a time for a software startup to enter defense, it's now.

**6. But "now" still means navigating two gauntlets.** Even with a modernized acquisition pathway, you still face two parallel compliance challenges: *organizational* cybersecurity compliance (CMMC, NIST 800-171) and *product-level* cybersecurity compliance (Authority to Operate). The first determines whether your company is trustworthy enough to handle defense data. The second determines whether your software is secure enough to run on defense networks. Both take time and money. Both are covered in Chapter 8. And both must be planned for from day one—not bolted on after you've won your first contract.

---

## Key Terms to Remember

| Term | Definition |
|---|---|
| **PPBE** | Planning, Programming, Budgeting, and Execution — the DoD's resource allocation system |
| **JCIDS** | Joint Capabilities Integration and Development System — the requirements generation process |
| **DAS** | Defense Acquisition System — the formal process for managing acquisitions |
| **ICD** | Initial Capabilities Document — defines a capability gap |
| **CDD** | Capability Development Document — specifies measurable requirements |
| **CPD** | Capability Production Document — finalizes production requirements |
| **JROC** | Joint Requirements Oversight Council — validates requirements |
| **DAB** | Defense Acquisition Board — senior review body for major acquisitions |
| **ACAT** | Acquisition Category — classifies programs by size and determines oversight level |
| **POM** | Program Objective Memorandum — multi-year funding plan |
| **POR** | Program of Record — officially approved, funded acquisition program |
| **MTA** | Middle Tier Acquisition — 2-5 year accelerated acquisition pathway |
| **AAF** | Adaptive Acquisition Framework — six-pathway acquisition system |
| **FFP** | Firm-Fixed-Price — set price, contractor bears risk |
| **CPFF** | Cost-Plus-Fixed-Fee — costs reimbursed plus fixed profit |
| **T&M** | Time & Materials — hourly rates plus materials at cost |
| **IDIQ** | Indefinite Delivery/Indefinite Quantity — umbrella contract with task orders |
| **MSA** | Materiel Solution Analysis — first phase, evaluates options |
| **TMRR** | Technology Maturation & Risk Reduction — prototype and risk reduction phase |
| **EMD** | Engineering & Manufacturing Development — full-scale development |
| **LRIP** | Low-Rate Initial Production — initial, small-batch production |
| **FRP** | Full-Rate Production — full-scale manufacturing |
| **Color of Money** | Legal restrictions on how different appropriation categories can be spent |
| **FYDP** | Future Year Defense Program — the 5-year funding plan |
| **CAPE** | Cost Assessment and Program Evaluation — the OSD office that manages programming |
| **USD(A&S)** | Under Secretary of Defense for Acquisition and Sustainment |
| **DoDI 5000.87** | DoD Instruction governing the Software Acquisition Pathway |
| **CNS** | Capability Need Statement — simplified requirements document for software programs |
| **UA** | User Agreement — formal agreement for continuous user involvement in software programs |
| **DevSecOps** | Development, Security, and Operations — integrated software development methodology |
| **cATO** | Continuous Authority to Operate — ongoing security authorization for software |
| **ATO** | Authority to Operate — formal security certification to deploy on DoD networks |

---

## Chapter Summary

The Department of Defense spends roughly $1 trillion per year through an acquisition system that was designed for a different era. That system rests on three pillars: JCIDS (which defines what the military needs), PPBE (which determines how much money is available and where it goes), and the Defense Acquisition System (which manages how things get built and bought).

The traditional acquisition lifecycle runs through five phases—from analyzing alternatives to sustaining fielded systems—with three major milestone decision points that act as gates. Programs are categorized by size (ACAT I, II, III), which determines how much oversight they receive and who has the authority to make milestone decisions. The largest programs go before the Defense Acquisition Board, a senior Pentagon body with the power to approve, restructure, or kill programs.

For software companies, the most significant development is the Software Acquisition Pathway (DoDI 5000.87), which replaces the traditional five-phase lifecycle with a two-phase model (Planning → Execution) built around Agile development, continuous delivery, and Value Assessments instead of milestone reviews. The March 2025 SecDef memorandum mandated this as the default pathway for all DoD software, with OTAs and CSOs as the default contracting mechanisms—a directive that officially bends the acquisition system toward how modern software companies operate.

The system has been criticized for decades as too slow (12 years average), too expensive (40-60% cost growth), and too hostile to innovation. Reform efforts—from the Packard Commission to the Adaptive Acquisition Framework—have made incremental progress. The Software Acquisition Pathway represents the most targeted reform yet for software-intensive systems, though significant challenges remain—particularly the Authority to Operate process and organizational cybersecurity compliance.

For defense tech startup founders, the critical takeaway is this: the system is knowable. It has rules, logic, and patterns. The founders who learn those rules—and who identify the alternative pathways that bypass the slowest parts of the system—are the ones who survive and thrive. The rest of this textbook will teach you how.
