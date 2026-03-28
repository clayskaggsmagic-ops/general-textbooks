# Chapter 11: Export Controls — ITAR, EAR, and Keeping Your Technology from Leaving the Country

---

Export controls are the laws that can put your founder in federal prison for showing a PowerPoint to the wrong person.

That is not hyperbole. Under U.S. export control law, transferring controlled technical data to a foreign national—even a foreign national sitting in your office in San Francisco—can constitute an illegal export. The penalties include criminal fines of up to $1 million per violation and imprisonment for up to 20 years. And unlike most business regulations, these are laws that the government actively enforces, with real people serving real prison sentences.

If you're building defense technology, export controls are not a "nice to have" compliance checkbox. They are the legal framework that determines who can see your technology, who can use it, and where in the world it can go. Getting this wrong doesn't just risk fines—it can shut down your company, bar you from government contracts forever, and send individuals to prison.

This chapter covers everything you need to know to stay on the right side of these laws.

---

## Why Export Controls Exist

The United States controls the export of military technology and sensitive dual-use technology for a simple reason: to prevent adversaries from acquiring capabilities that could threaten U.S. national security.

The legal foundation rests on two primary statutes:

1. **The Arms Export Control Act (AECA)** — authorizes the President to control the export and import of defense articles and services. Implemented through ITAR.

2. **The Export Control Reform Act of 2018 (ECRA)** — provides authority to control "dual-use" items (those with both commercial and military applications) and certain military items not covered by AECA. Implemented through EAR.

These laws create two parallel regimes, administered by two different agencies, covering two different categories of items. Understanding which regime applies to your technology is the first critical step.

---

## ITAR: The International Traffic in Arms Regulations

### What ITAR Covers

ITAR is administered by the **Directorate of Defense Trade Controls (DDTC)** within the U.S. Department of State. It controls:

- **Defense articles** — items specifically designed, developed, configured, adapted, or modified for military applications, listed on the U.S. Munitions List (USML)
- **Defense services** — the furnishing of assistance (including training) to foreign persons in the design, development, engineering, manufacture, production, assembly, testing, repair, maintenance, modification, operation, demilitarization, or destruction of defense articles
- **Technical data** — information required for the design, development, production, manufacture, assembly, operation, repair, testing, maintenance, or modification of defense articles. This includes blueprints, drawings, photographs, plans, instructions, and documentation

### The U.S. Munitions List (USML)

The USML is organized into 21 categories:

| Category | Description |
|---|---|
| I | Firearms, Close Assault Weapons, and Combat Shotguns |
| II | Guns and Armament |
| III | Ammunition/Ordnance |
| IV | Launch Vehicles, Guided Missiles, Ballistic Missiles, Rockets, Torpedoes, Bombs, and Mines |
| V | Explosives and Energetic Materials |
| VI | Surface Vessels of War and Special Naval Equipment |
| VII | Ground Vehicles |
| VIII | Aircraft and Related Articles |
| IX | Military Training Equipment and Training |
| X | Personal Protective Equipment |
| XI | Military Electronics |
| XII | Fire Control, Laser, Imaging, and Guidance Equipment |
| XIII | Materials and Miscellaneous Articles |
| XIV | Toxicological Agents, Including Chemical and Biological Agents, and Associated Equipment |
| XV | Spacecraft and Related Articles |
| XVI | Nuclear Weapons Related Articles |
| XVII | Classified Articles, Technical Data, and Defense Services |
| XVIII | Directed Energy Weapons |
| XIX | Gas Turbine Engines and Associated Equipment |
| XX | Submersible Vessels and Related Articles |
| XXI | Articles, Technical Data, and Defense Services Not Otherwise Enumerated |

**For software companies,** the most relevant categories are typically:
- **Category XI** (Military Electronics) — if your software is embedded in or controls military electronic systems
- **Category XII** (Fire Control, Laser, Imaging, and Guidance Equipment) — if your software processes targeting, guidance, or intelligence imagery data
- **Category XIII** (Materials and Miscellaneous Articles) — catch-all for items like cryptographic equipment interfaces
- **Category XXI** — the catch-all for defense articles not listed elsewhere

### The 2025 USML Revisions

A final rule effective **September 15, 2025** significantly updated the USML:

**Items removed from USML (transitioned to EAR/CCL):**
- Certain GNSS anti-jam/anti-spoofing systems and Controlled Reception Pattern Antennas (CRPAs) from Category XI
- Lead-free birdshot ammunition from Category III
- Various items no longer providing a critical military or intelligence advantage

**Items added to USML:**
- Large unmanned underwater vehicles (UUVs) with specific military features under Category XX(a)(10)
- The F-47 fighter aircraft
- Advanced aircraft parts, developmental exoskeletons, nerve agents, and new engine types

**New exemptions:**
- **ITAR §126.9(u)** — permits temporary export and reexport of certain large UUVs (over 3,000 lbs but not exceeding 8,000 lbs) for civil purposes including scientific research, oil and gas infrastructure, telecommunications cable work, and search and rescue. Strictly a contractor-owned, contractor-operated (COCO) model—no transfer of ownership to foreign persons.

**Existing DDTC licenses and approvals for transitioning items remain valid for up to three years,** giving companies time to adjust to new jurisdictional requirements.

### The "Deemed Export" Rule

This is the rule that catches most startups off guard:

**A "deemed export" occurs when controlled technical data or defense services are released to a foreign person within the United States.** Under ITAR, it doesn't matter that the foreign person is sitting in your office. It doesn't matter that they're your employee. If they're not a U.S. person (citizen, lawful permanent resident, or protected person), releasing ITAR-controlled technical data to them is an export—and it requires authorization.

**"U.S. person" means:**
- A U.S. citizen
- A lawful permanent resident (green card holder)
- A protected person (refugee or asylee)

It does **not** include H-1B visa holders, F-1 students, or other temporary residents.

**Practical implications:**
- If your company develops ITAR-controlled technology, you cannot let foreign national employees access that technical data without a license
- This applies to source code, design documents, architecture diagrams, test results, and any other "technical data" as defined by ITAR
- You need access controls—physical and digital—to prevent foreign nationals from viewing ITAR-controlled information
- Your office may need a **Technology Control Plan (TCP)** that segregates ITAR work from non-controlled work

**Recent proposed change:** DDTC is finalizing a rule that would restrict deemed export authorization to countries where the foreign person currently holds citizenship or permanent residency—a departure from the current rule that considers all countries where the person has ever held citizenship or permanent residency.

### DDTC Registration

**If you manufacture, export, or broker defense articles, you must register with DDTC.** This is not optional—it's a prerequisite for obtaining export licenses or using exemptions.

As of January 2025, the fee structure is tiered:

| Tier | Who | Annual Fee |
|---|---|---|
| **Tier 1** | First-time registrants, stand-alone broker renewals, registrants without an approved license in the past 90 days, tax-exempt nonprofits | $3,000 ($2,500 with pilot discount) |
| **Tier 2** | Registrants who submitted ≤10 license applications in the past 12 months | $4,000 |
| **Tier 3** | Registrants who submitted >10 license applications | $4,000 baseline + additional per application (capped at 3% of total license value) |

Key requirements:
- Registration must be renewed annually
- You must notify DDTC of material changes (CEO, senior officers, board members) within 5 days
- Failure to notify DDTC of material changes can result in criminal penalties and disruption of existing licenses

### Export Licenses and Agreements

When you need to share ITAR-controlled technical data or defense articles with a foreign person (whether overseas or through a deemed export), you generally need one of the following:

**Technical Assistance Agreement (TAA):** Required when you're providing defense services or ITAR-controlled technical data to a foreign person. This is the most common authorization for software companies sharing technical information with foreign partners or customers.

**Manufacturing License Agreement (MLA):** Required when a foreign person will manufacture defense articles using your technical data.

**Direct Commercial Sales (DCS) License:** For one-time exports of specific defense articles to specific end-users.

**Processing time** for ITAR licenses is typically 30-60 days, but complex cases can take significantly longer.

---

## EAR: The Export Administration Regulations

### What EAR Covers

EAR is administered by the **Bureau of Industry and Security (BIS)** within the U.S. Department of Commerce. It controls:

- **Dual-use items** — goods, software, and technology with both civilian and military applications
- **Some military items** that don't rise to the level of USML control
- **Commercial items** that could contribute to weapons of mass destruction programs or military capabilities of concern

EAR has **broader scope but generally less restrictive controls** than ITAR. Many EAR-controlled items can be exported without a license to most destinations, while ITAR-controlled items almost always require authorization.

### The Commerce Control List (CCL)

The CCL is organized into **10 categories** (numbered 0-9):

| Category | Description |
|---|---|
| 0 | Nuclear Materials, Facilities, and Equipment |
| 1 | Special Materials and Related Equipment |
| 2 | Materials Processing |
| 3 | Electronics |
| 4 | Computers |
| 5 | Telecommunications and Information Security |
| 6 | Sensors and Lasers |
| 7 | Navigation and Avionics |
| 8 | Marine |
| 9 | Aerospace and Propulsion |

Within each category, items are further classified by **product group**:

- **A** — Systems, equipment, and components
- **B** — Test, inspection, and production equipment
- **C** — Materials
- **D** — Software
- **E** — Technology

### Export Control Classification Numbers (ECCNs)

Every item on the CCL has a 5-character alphanumeric **ECCN**. For example:

- **5D002** — Encryption software (Category 5, Product Group D for Software, Control Number 002)
- **3A090** — Advanced computing integrated circuits
- **4E091** — AI model weights (new, January 2025)

**If your item is NOT on the CCL and not subject to any other agency's exclusive jurisdiction, it's classified as EAR99.** EAR99 items can generally be exported without a license to most destinations and end-users, though restrictions still apply for embargoed countries and prohibited end-users.

### Key Differences: ITAR vs. EAR

| Factor | ITAR | EAR |
|---|---|---|
| **Agency** | State Dept. (DDTC) | Commerce Dept. (BIS) |
| **Legal authority** | Arms Export Control Act | Export Control Reform Act |
| **What it controls** | Defense articles, services, technical data | Dual-use items, some military items |
| **Control list** | USML (21 categories) | CCL (10 categories + ECCNs) |
| **Default posture** | Restrictive — license required for most exports | Less restrictive — many items exportable without license |
| **Registration** | Required for manufacturers/exporters/brokers | Not required |
| **Jurisdiction determined by** | Whether item is "specifically designed" for military use | Whether item has dual-use capabilities |
| **Penalties** | Civil: up to $1.27M/violation; Criminal: $1M, 20 years | Civil: up to $374K/violation; Criminal: $1M, 20 years |
| **Deemed exports** | Yes — strictly enforced | Yes — applies to controlled technology |

---

## Commodity Jurisdiction: Which Regime Applies?

When it's not obvious whether your product falls under ITAR or EAR, you need a **Commodity Jurisdiction (CJ) determination.** This is a formal process to determine which agency has jurisdiction.

### How to Request a CJ Determination

1. **Submit Form DS-4076** through DDTC's online DECCS portal. You don't need to be registered with DDTC to submit a CJ request.
2. **Include detailed information:** product description, capabilities, components, end-use platform, military standards it meets (if any), and sales history.
3. DDTC forwards the request to BIS and the Defense Technology Security Administration (DTSA) for review.
4. **Processing time:** typically under 65 business days.

**Critical rule: While a CJ determination is pending, you must treat your item as ITAR-controlled.** If you want to export it before the determination is complete, you need ITAR authorization.

### When to Seek a CJ Determination

- Your product has both military and commercial applications
- Your product was originally developed for commercial use but has defense applications
- Your product uses components or technology that appear on both the USML and CCL
- You're unsure whether recent USML revisions have moved your product to EAR jurisdiction
- A customer or partner asks which regime applies and you can't confidently answer

---

## Software-Specific Export Control Issues

Software companies face unique export control challenges because code, algorithms, and technical data can cross borders instantaneously through email, cloud platforms, and collaboration tools.

### When Is Software ITAR-Controlled?

Software is ITAR-controlled if it is:
- Listed on the USML (as a defense article in its own right)
- Technical data directly related to a defense article on the USML
- Part of a defense service (e.g., training a foreign person to use ITAR-controlled equipment)

**Examples of ITAR-controlled software:**
- Source code for a missile guidance algorithm
- Flight control software specifically designed for a military aircraft
- Image processing software designed for military ISR (Intelligence, Surveillance, Reconnaissance) systems
- Cybersecurity tools designed specifically to attack military-grade encryption

### When Is Software EAR-Controlled?

Software falls under EAR if it:
- Has dual-use capabilities and is classified under a CCL ECCN
- Contains encryption functionality (Category 5, Part 2)
- Involves advanced computing or AI technologies subject to new controls
- Was previously ITAR-controlled but transitioned to EAR through USML revision

### The "Publicly Available" and "Fundamental Research" Exclusions

Two important exclusions may apply to your software:

**Publicly Available Information:** Under both ITAR and EAR, information that is already in the public domain is generally not controlled. For ITAR, this means information published and generally accessible to the public through fundamental research in science and engineering, sales at newsstands and bookstores, libraries, patents, or public presentations.

**However — and this is critical for startups using open source:**
- Open-source software that is "published" and freely available may be excluded from some controls
- But if you modify open-source code and the modifications add controlled functionality (such as military-specific features), the modifications could be controlled even if the underlying open-source code is not
- The January 2025 BIS rule on AI model weights explicitly excluded "published" (open-source) model weights from ECCN 4E091 control, but closed-weight models were subject to it

**Fundamental Research:** Results of basic and applied research in science and engineering performed at accredited universities are generally exempt. But this exclusion applies to the *research results* — not to specific military applications of that research.

### SaaS and Cloud Deployment Challenges

SaaS delivery creates particularly thorny export control issues:

**Problem 1: Where is the cloud server?** If your SaaS application is hosted on a cloud server located outside the United States, you may have "exported" the software simply by deploying it. Even if the server is in an allied country, the deployment could require EAR authorization.

**Problem 2: Who is accessing the service?** If foreign nationals access your SaaS platform, you may be making a deemed export of the underlying technology—even if they're only interacting with the application through a user interface. The analysis depends on whether the access provides the foreign person with controlled "technology" or "technical data."

**Problem 3: Overseas DoD installations.** If the DoD deploys your software to a base in Germany, Japan, or CENTCOM's area of operations, the deployment itself could trigger export control requirements—even though the customer is the U.S. government. The software is physically located in a foreign country.

**Problem 4: Foreign allied access.** Many DoD programs involve coalition partners. If your software is used by the U.S. military alongside British, Australian, or Japanese partners, those foreign military personnel may be accessing controlled technology—triggering deemed export or standard export requirements.

**Best practices:**
- Determine the export classification of your software before offering it as SaaS
- Implement access controls that restrict use by foreign nationals unless proper authorization is in place
- Confirm the physical location of cloud servers and ensure it's acceptable under applicable regulations
- For DoD deployments overseas, work with the contracting officer and DDTC/BIS to confirm authorization
- Put geofencing and IP-based access restrictions in place for sensitive SaaS applications

---

## Encryption Export Controls

If your software includes encryption—and virtually all modern software does—you need to understand EAR Category 5, Part 2.

### What's Controlled

Category 5, Part 2 controls items that provide **confidentiality of information** through cryptographic means. This includes:

- Encryption hardware (ECCN 5A002)
- Encryption software (ECCN 5D002 for non-mass market; ECCN 5D992 for mass market)
- Encryption technology (ECCN 5E002)

### Mass Market vs. Non-Mass Market

**Mass market encryption software** (ECCN 5D992) is designed for individual consumers or small businesses, sold at retail, and generally available to the public. Think: consumer messaging apps, basic VPN tools, standard TLS implementations. Mass market items can typically be exported without a license (**NLR — No License Required**), except to embargoed countries.

**Non-mass market encryption software** (ECCN 5D002) requires classification and may require a license, depending on destination and end-use. However, **License Exception ENC** (EAR §740.17) provides a streamlined path for many encryption exports.

### License Exception ENC

License Exception ENC allows export of many encryption items without an individual license, subject to classification, reporting, and end-use/end-user requirements:

- **ENC (b)(1):** Applies to mass market items. Broad authorization, minimal restrictions.
- **ENC (b)(2):** Applies to network infrastructure items, source code, items customized for government use, and other more sensitive categories. Requires a classification request. Has restrictions for government end-users in certain countries.
- **ENC (b)(3):** Applies to more sensitive items like cryptographic chips, libraries, and products with "non-standard cryptography." Requires a classification request.

**Practical steps for software startups:**
1. Determine whether your software uses encryption
2. If yes, classify the encryption components (mass market vs. non-mass market, ECCN)
3. File a classification request with BIS if required for License Exception ENC
4. Implement end-user screening to ensure you're not exporting to prohibited destinations or end-users
5. Maintain records of all encryption exports

---

## AI and Machine Learning Export Controls

The export control of AI technology is the most rapidly evolving area of the regulatory landscape.

### The January 2025 AI Diffusion Rule (and Its Reversal)

In January 2025, BIS issued an interim final rule creating a new ECCN **4E091** for AI model weights—specifically targeting the weights of advanced closed-weight dual-use AI models. The rule imposed a worldwide license requirement with a presumption of denial for most destinations outside of close U.S. allies.

**However, on May 13, 2025, BIS announced it would rescind this rule and directed non-enforcement of controls on AI model weights under ECCN 4E091.** A formal rescission and replacement rule is expected.

**What this means for startups:**
- Direct ECCN-based control on model weights is currently paused
- But heightened due diligence expectations remain—you're still expected to screen customers and end-uses
- Advanced computing chips (ECCNs 3A090, 4A090) used for AI training remain strictly controlled
- The regulatory landscape will continue to shift—build flexibility into your compliance program

### AI-Specific Exported Control Considerations

Even with the model weights rule paused, several AI-related export control issues remain active:

1. **Training data.** If training data contains ITAR-controlled technical data (e.g., military specifications, classified information), any model trained on that data may inherit the classification of the training data.

2. **Algorithmic specifications.** Detailed descriptions of your AI algorithm—architecture, hyperparameters, training methodology—could constitute controlled "technology" or "technical data" under either ITAR or EAR.

3. **Model outputs.** If your AI model processes controlled inputs and generates outputs that reveal information about controlled technical data, those outputs could be controlled.

4. **Fine-tuning for military applications.** A commercially available AI model that is fine-tuned for a specific military application could transition from EAR jurisdiction (or no jurisdiction) to ITAR jurisdiction if the fine-tuning creates a "defense article."

5. **Inference API access.** Providing foreign persons with API access to a model that processes military-relevant queries could constitute a "defense service" under ITAR if the model is providing assistance related to defense articles.

---

## Penalties: What Happens When You Get It Wrong

### ITAR Penalties (as of January 2025)

| Type | Maximum Penalty |
|---|---|
| **Civil** | Greater of **$1,271,078 per violation** or twice the transaction value (adjusted annually for inflation) |
| **Criminal** | **$1,000,000 per violation** and/or **up to 20 years imprisonment** |
| **Debarment** | Statutory debarment for 3 years upon conviction; bars all ITAR-regulated activities |
| **License revocation** | Loss of all existing export licenses and inability to obtain new ones |

### EAR Penalties (as of January 2025)

| Type | Maximum Penalty |
|---|---|
| **Civil** | Greater of **$374,474 per violation** or twice the transaction value |
| **Criminal** | Up to **$1,000,000 per violation** and/or **up to 20 years imprisonment** |
| **Denial of export privileges** | Prohibition from participating in any export transaction |
| **Seizure/forfeiture** | Government can seize articles involved in the violation |

### Real-World Examples

- **Seagate Technologies**: $300 million civil penalty for exporting hard drives to Huawei in violation of EAR
- **3D Systems**: $20 million settlement for ITAR violations related to sharing controlled technology with foreign nationals
- **Individual case**: A person received 60 months imprisonment for conspiring to export AK-47-type rifles to Mexico without a license

### The Denied Parties Lists

Multiple government agencies maintain lists of individuals and entities with whom U.S. companies are prohibited from doing business:

- **Denied Persons List (BIS)** — persons denied export privileges
- **Entity List (BIS)** — foreign companies and organizations subject to specific license requirements
- **Unverified List (BIS)** — parties whose bona fides could not be verified
- **Specially Designated Nationals (OFAC)** — persons owned or controlled by sanctioned countries, or associated with terrorism or narcotics trafficking
- **Debarred Parties (DDTC)** — persons prohibited from ITAR-regulated activities

**You are required to screen all parties in every transaction against these lists.** This includes customers, end-users, consignees, freight forwarders, and any other party involved in the transaction.

---

## Voluntary Self-Disclosure

Both DDTC and BIS strongly encourage **voluntary self-disclosure (VSD)** of potential violations. If you discover a possible violation:

**Benefits of VSD:**
- Considered a significant mitigating factor in determining penalties
- Can result in reduced penalties, no action, or a warning letter
- For EAR violations, VSD can cap the maximum civil penalty at 50% of the prescribed maximum
- Demonstrates good faith and a commitment to compliance

**Consequences of NOT disclosing:**
- Failure to disclose a known violation is treated as an aggravating factor
- Can result in significantly higher penalties
- May lead to criminal referral

**When VSD is mandatory:**
- Under ITAR, certain violations involving proscribed countries (Iran, North Korea, Syria, Cuba, etc.) must be disclosed
- Unreturned temporary exports that exceed their authorization period must be disclosed

**Process:**
1. Investigate the potential violation thoroughly
2. Submit a formal VSD to the appropriate agency (DDTC for ITAR, BIS for EAR)
3. Include a description of the violation, the corrective actions taken, and any mitigating factors
4. Cooperate fully with any government investigation

---

## Export Controls and Hiring

This is where export controls intersect with one of your most critical business functions: building your team.

### The Deemed Export Problem with Foreign National Employees

If your company works on ITAR-controlled technology:

- **H-1B visa holders** are foreign persons under ITAR. Giving them access to ITAR-controlled technical data is a deemed export to their country of citizenship.
- **F-1/OPT students** are foreign persons. Same rules apply.
- **Green card holders** are U.S. persons. They can access ITAR data without restriction.
- **Dual citizens** — the analysis depends on the specific countries of citizenship and the applicable regulations.

### What This Means in Practice

1. **Before hiring foreign nationals,** determine whether your work involves ITAR-controlled technical data or EAR-controlled technology.

2. **If it does,** you have three options:
   - **Obtain a license.** Apply for an ITAR license (TAA) or EAR authorization that covers the deemed export to the individual's country of citizenship.
   - **Segregate the work.** Keep the foreign national employee away from all controlled technical data. This requires physical and IT access controls, separate work areas, and a Technology Control Plan.
   - **Don't hire.** In some cases—particularly for citizens of embargoed or sanctioned countries—no license will be available, and the only compliant option is not to employ that person on controlled work.

3. **Technology Control Plans (TCPs):** If you employ foreign nationals and work on controlled technology, you need a TCP that documents:
   - What technology is controlled
   - Who has access (and who doesn't)
   - Physical access controls (locked doors, clean desks, escort requirements)
   - IT access controls (network segmentation, access permissions, encryption)
   - Training requirements for all employees
   - Incident reporting procedures

### The Startup Hiring Crisis

This creates a genuine crisis for defense tech startups, particularly in Silicon Valley and other tech hubs with large populations of foreign-born engineers:

- You may have to exclude talented engineers from specific projects
- You may need separate physical spaces and network segments for controlled and uncontrolled work
- Visa sponsorship decisions need to account for export control implications
- Your HR team needs to understand export controls—not just immigration law

---

## Building an Export Compliance Program

Every defense tech startup needs an **Export Compliance Program (ECP)** built into its operations from day one. Here's what it should include:

### 1. Management Commitment

- Written policy statement from the CEO committing the company to export compliance
- Appointment of an **Empowered Official** (required for ITAR registrants) — a senior person with authority to sign export licenses and make binding compliance decisions
- Allocation of budget and resources for compliance activities

### 2. Product Classification

- Determine whether each product, service, and technology is ITAR-controlled (and which USML category), EAR-controlled (and which ECCN), or EAR99
- Request Commodity Jurisdiction determinations for ambiguous items
- Re-evaluate classifications when products change or regulations are updated
- Maintain a classification matrix documenting the status of every product

### 3. Screening

- Screen all parties in every transaction against the consolidated denied/restricted party lists
- Use automated screening software (Visual Compliance, Descartes, OCR Solutions, etc.)
- Screen at multiple points: when a customer first engages, when an order is placed, and when shipment occurs
- Screen employees and job applicants where relevant to deemed export issues

### 4. Licensing

- Identify when a license or agreement is required before any export, including deemed exports
- Maintain a tracking system for all license applications, approvals, and conditions
- Monitor compliance with license conditions (quantities, end-uses, timeframes)
- Have a process for emergency situations where time is critical

### 5. Training

- All employees should receive export control awareness training at hiring and annually
- Engineers and technical staff working on controlled technology need detailed training on what they can and cannot share
- Business development and sales teams need training on screening requirements and red flags
- International shipping and logistics personnel need detailed process training

### 6. Record-Keeping

- Maintain records of all export transactions for a minimum of **5 years** (ITAR requires 5 years; EAR may require longer in some cases)
- Records should include: classification documentation, license applications and approvals, screening results, shipping documents, and correspondence
- Implement a document retention policy that meets regulatory requirements

### 7. Internal Audits

- Conduct regular internal audits of your compliance program (at least annually)
- Use the results to identify gaps, update procedures, and improve training
- Consider periodic external audits by qualified export compliance counsel

### 8. Incident Response

- Establish procedures for reporting and investigating potential violations
- Define clear escalation paths from individual contributor → compliance officer → legal counsel → management
- Know when and how to file voluntary self-disclosures
- Document everything

---

## The Export Control Decision Tree for Startups

When evaluating any technology, product, or transaction, work through this decision tree:

```
1. Is my product on the USML?
   ├── YES → ITAR applies. Register with DDTC. Get licenses for ALL exports.
   └── NOT SURE →
       ├── Request a CJ determination from DDTC
       └── Treat as ITAR-controlled until determination is received

2. If not USML → Is my product on the CCL (has an ECCN)?
   ├── YES → EAR applies. Check license requirements based on
   │         destination, end-user, and end-use.
   └── NO → Product is EAR99. Generally exportable without license,
            but still check for:
            ├── Prohibited end-users (denied parties lists)
            ├── Prohibited end-uses (WMD, military, etc.)
            └── Embargoed destinations

3. Does my product contain encryption?
   ├── YES → Classify under Category 5, Part 2
   │         ├── Mass market → Likely NLR (ECCN 5D992)
   │         └── Non-mass market → ECCN 5D002, use License Exception ENC
   └── NO → Not applicable

4. Am I sharing technical data or source code with a foreign person?
   ├── YES → Deemed export analysis required
   │         ├── ITAR data → License (TAA) required
   │         └── EAR data → Check if license required based on ECCN,
   │                        country, and end-use
   └── NO → Not a deemed export concern

5. Am I hiring foreign national engineers?
   ├── YES → Determine if they will access controlled technology
   │         ├── YES → Obtain license OR segregate work OR don't assign
   │         └── NO → Document the access controls
   └── NO → Not a deemed export concern for hiring
```

---

## Common Mistakes Startups Make

**1. Assuming ITAR doesn't apply because you're a "software company."** Software that is specifically designed for military applications is a defense article. Source code for ITAR-controlled systems is ITAR-controlled technical data. Period.

**2. Not registering with DDTC.** If you manufacture or export defense articles—including ITAR-controlled software—you must register with DDTC. This is a legal obligation, not a suggestion. And failure to register doesn't excuse you from ITAR compliance; it just adds another violation.

**3. Sharing technical data in open collaboration tools.** If your engineers share ITAR-controlled information via Slack, GitHub, Google Docs, or email with foreign nationals—even co-workers—that's an illegal deemed export. Lock down your collaboration tools.

**4. Ignoring deemed export rules when hiring.** Many startups hire the best available talent without considering export control implications. If you're working on controlled technology, you must factor export controls into your hiring and access control processes.

**5. Failing to classify products.** "We never bothered to figure out if we're ITAR or EAR" is not a defense. Classification is your responsibility. Not knowing your product's classification doesn't shield you from penalties.

**6. Assuming your cloud provider handles compliance.** AWS GovCloud, Azure Government, and Google Cloud for Government provide infrastructure that meets certain compliance requirements—but the export control classification and licensing obligations remain yours. Your cloud provider doesn't get ITAR licenses on your behalf.

**7. Treating export controls as a one-time exercise.** Products change. Regulations change. The USML is revised regularly. Customer bases evolve. You need to continuously monitor and update your classifications, screening, and compliance procedures.

---

## Key Terms

**ITAR (International Traffic in Arms Regulations):** U.S. State Department regulations controlling the export of defense articles, services, and technical data. Administered by DDTC.

**EAR (Export Administration Regulations):** U.S. Commerce Department regulations controlling the export of dual-use items. Administered by BIS.

**DDTC (Directorate of Defense Trade Controls):** The State Department office that administers ITAR.

**BIS (Bureau of Industry and Security):** The Commerce Department agency that administers EAR.

**USML (U.S. Munitions List):** The list of defense articles and services subject to ITAR control. Organized into 21 categories.

**CCL (Commerce Control List):** The list of dual-use items subject to EAR control. Organized into 10 categories.

**ECCN (Export Control Classification Number):** A 5-character alphanumeric code identifying an item's position on the CCL and the reasons for its control.

**EAR99:** Items subject to EAR but not specifically listed on the CCL. Generally exportable without a license.

**Deemed Export:** The release of controlled technology or technical data to a foreign person within the United States, which is treated as an export to that person's country of citizenship.

**CJ (Commodity Jurisdiction) Determination:** The formal process to determine whether an item falls under ITAR or EAR.

**TAA (Technical Assistance Agreement):** An ITAR authorization for providing defense services or technical data to foreign persons.

**License Exception ENC:** An EAR provision allowing export of certain encryption items without an individual license, subject to classification, reporting, and end-use requirements.

**VSD (Voluntary Self-Disclosure):** A formal disclosure to DDTC or BIS of a potential export control violation. Treated as a mitigating factor in enforcement.

**TCP (Technology Control Plan):** A documented plan for controlling access to ITAR-controlled or EAR-controlled technology within a facility, particularly when foreign nationals are present.

**Debarment:** Prohibition from participating in ITAR-regulated activities, typically for 3 years following a criminal conviction under the AECA.

**Entity List:** A BIS list of foreign companies and organizations subject to specific license requirements due to national security or foreign policy concerns.
