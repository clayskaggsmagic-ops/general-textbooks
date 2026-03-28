# Chapter 8: Cybersecurity and Compliance — CMMC, ATO, DevSecOps, and the Software Security Gauntlet

---

If there is a single chapter in this book that will determine whether your software company succeeds or fails in the defense market, this is it.

Government contracting has many barriers to entry—registration (Chapter 4), regulations (Chapter 5), accounting systems (Chapter 7). But for software companies specifically, cybersecurity compliance is the highest wall. It's also the most misunderstood, because it's actually **two separate walls** that most founders don't distinguish until it's too late:

**Wall 1: Organizational compliance.** Your *company* must demonstrate that it protects sensitive government information. This is the CMMC/NIST 800-171 world. It's about your corporate IT environment, your email, your laptops, your network.

**Wall 2: Product compliance.** Your *software* must be authorized to operate within DoD networks. This is the ATO/RMF world. It's about your application architecture, your code, your deployment environment, your security testing.

These are different compliance regimes, governed by different regulations, assessed by different organizations, on different timelines, at different costs. Conflating them—or worse, ignoring one while focusing on the other—is the single most common mistake software startups make in defense.

This chapter will walk you through both walls, explain how they interact, and give you a strategy for getting over them incrementally rather than all at once.

---

## Part I: Organizational Cybersecurity — Protecting Government Information

### Controlled Unclassified Information: The Data You Didn't Know You Had

Before you can understand the compliance requirements, you need to understand what you're protecting. The answer is **Controlled Unclassified Information (CUI)**.

CUI is government-created or government-owned information that, while not classified (not SECRET or TOP SECRET), still requires safeguarding. It's the vast middle ground between publicly releasable information and classified national security information. CUI categories include:

- Technical data about weapon systems and defense articles
- Export-controlled information
- Personally identifiable information (PII) of government personnel
- For Official Use Only (FOUO) documents
- Source selection information
- Contractor proprietary data submitted to the government
- Critical infrastructure information
- Vulnerability assessments

If you're building software for the DoD, you will almost certainly handle CUI. Your software's technical documentation, system architecture diagrams, vulnerability reports, user data from military personnel, test results—all of this is likely CUI. And once CUI touches your systems, your entire information handling environment becomes subject to federal cybersecurity requirements.

CUI is designated by the originating government agency using markings defined by the National Archives and Records Administration (NARA) CUI Registry. But here's the practical reality: many contracting officers don't explicitly mark every piece of CUI. The smart assumption is that virtually all non-public information you receive from or create for a DoD contract is CUI until proven otherwise.

---

### NIST SP 800-171: The 110 Controls

NIST Special Publication 800-171—"Protecting Controlled Unclassified Information in Nonfederal Systems and Organizations"—defines the security requirements that every company handling CUI must implement. These are the 110 controls organized into 14 families that form the baseline for organizational cybersecurity compliance in defense contracting.

The 14 control families:

| Family | Controls | What It Covers |
|---|---|---|
| **Access Control (AC)** | 22 controls | Who can access what. User authentication, remote access, account management, least privilege, session controls. |
| **Awareness and Training (AT)** | 3 controls | Security awareness training for all users and specialized training for users with security roles. |
| **Audit and Accountability (AU)** | 9 controls | Logging system events, protecting audit logs, review and analysis of logs, timestamp accuracy. |
| **Configuration Management (CM)** | 9 controls | Baseline configurations, change management, least functionality, software restrictions. |
| **Identification and Authentication (IA)** | 11 controls | Multi-factor authentication, password policies, device identification, replay-resistant authentication. |
| **Incident Response (IR)** | 3 controls | Incident handling capabilities, reporting, and testing/exercises. |
| **Maintenance (MA)** | 6 controls | System maintenance, maintenance tools, remote maintenance controls. |
| **Media Protection (MP)** | 9 controls | Media access, marking, storage, transport, sanitization. USB drives, hard drives, removable media. |
| **Personnel Security (PS)** | 2 controls | Personnel screening and personnel actions during transfers or terminations. |
| **Physical Protection (PE)** | 6 controls | Physical access controls, monitoring, visitor access, emergency power, fire protection. |
| **Risk Assessment (RA)** | 3 controls | Risk assessments, vulnerability scanning, remediation of vulnerabilities. |
| **Security Assessment (CA)** | 4 controls | Security assessments, plans of action, system authorization, monitoring. |
| **System and Communications Protection (SC)** | 16 controls | Boundary protection, cryptographic protections, network segmentation, CUI encryption in transit and at rest. |
| **System and Information Integrity (SI)** | 7 controls | Flaw remediation, malicious code protection, security alerts, system monitoring. |

**Important version note:** NIST published SP 800-171 Revision 3 in May 2024, which restructured and updated the controls. However, the DoD currently mandates compliance with **Revision 2** for DFARS 252.204-7012 and CMMC purposes. Monitor this carefully—the transition to Rev 3 will happen, but the timeline is still being determined.

---

### DFARS 252.204-7012: The Clause That Started Everything

Before CMMC existed, there was DFARS 252.204-7012—"Safeguarding Covered Defense Information and Cyber Incident Reporting." This clause, which appears in virtually every DoD contract involving CUI, requires contractors to:

1. **Implement NIST SP 800-171** security requirements on any system that processes, stores, or transmits Covered Defense Information (CDI)
2. **Report cyber incidents** to the DoD within 72 hours
3. **Preserve and provide** forensic evidence for 90 days following an incident
4. **Flow down** the requirement to all subcontractors

The problem with DFARS 7012 was enforcement. Contractors self-attested to compliance—they assessed themselves against the 110 controls and submitted a score. There was no independent verification. Predictably, many contractors claimed compliance without actually implementing the controls. Studies estimated that a significant majority of the defense industrial base was not actually compliant.

CMMC was created to solve this enforcement gap.

---

### CMMC 2.0: The Three Levels

The Cybersecurity Maturity Model Certification (CMMC) 2.0 is the DoD's program to verify that defense contractors actually implement the cybersecurity controls they're required to have. The final CMMC rule was published in the Federal Register in September 2025, with implementation beginning November 10, 2025.

**Level 1 — Foundational (17 controls)**

Level 1 applies to contractors handling Federal Contract Information (FCI)—basic contract-related information that isn't CUI. It requires compliance with 17 practices derived from FAR 52.204-21 (basic safeguarding). Level 1 is assessed through **annual self-assessment**, with results affirmed by a senior company official and entered into the Supplier Performance Risk System (SPRS).

Think of Level 1 as basic IT hygiene: use antivirus, lock your screen, change default passwords, limit access. Most technology companies already do most of this.

**Level 2 — Advanced (110 controls = NIST SP 800-171)**

Level 2 applies to contractors handling CUI—which is most defense software contractors. It requires implementation of all 110 NIST SP 800-171 Rev 2 controls. Level 2 has two assessment tracks:

- **Self-assessment:** For contracts involving CUI that is not considered critical to national security. The contractor conducts its own assessment, scores itself, and submits results to SPRS. A senior official must affirm the results.
- **C3PAO certification:** For contracts involving CUI that is critical to national security (or at the contracting officer's discretion). A Certified Third-Party Assessment Organization (C3PAO), accredited by the CMMC Accreditation Body (the Cyber AB), conducts an independent on-site assessment. This is the version most people mean when they say "CMMC certification."

**Level 3 — Expert (110+ controls = NIST SP 800-172 enhancements)**

Level 3 applies to contractors handling the most sensitive CUI associated with the highest-priority programs. It requires all 110 NIST 800-171 controls plus additional enhanced security requirements from NIST SP 800-172. Level 3 assessments are conducted by the **government itself**—specifically the Defense Industrial Base Cybersecurity Assessment Center (DIBCAC).

Level 3 is rare and currently applies to only the most sensitive programs. Most software startups will need Level 2.

### CMMC Phased Rollout Timeline

The rollout is occurring in four phases:

| Phase | Date | What Happens |
|---|---|---|
| **Phase 1** | November 10, 2025 | Level 1 and Level 2 self-assessments required in applicable new contracts. DoD may also include Level 2 C3PAO certification requirements at its discretion. Applies to ~65% of the defense industrial base. |
| **Phase 2** | November 10, 2026 | Level 2 C3PAO certification assessments become mandatory for applicable contracts requiring third-party assessment. |
| **Phase 3** | November 10, 2027 | Level 3 government-led assessments are introduced. |
| **Phase 4** | November 10, 2028 | Full implementation across all applicable DoD contracts. |

**What this means for you:** If you plan to bid on DoD contracts involving CUI in 2026 or later, you will need CMMC Level 2 certification. Start preparing now—the assessment process takes 6-12 months from the time you engage a C3PAO.

---

### The CMMC Assessment Process

**C3PAO Assessment (Level 2)**

1. **Readiness.** Before engaging a C3PAO, get your house in order. Implement the 110 controls, document them in a System Security Plan (SSP), and conduct an internal assessment. Many companies hire a CMMC Registered Practitioner (RP) or consultant ($15K–$50K) to conduct a readiness assessment first.

2. **Engage a C3PAO.** Select an accredited C3PAO from the Cyber AB marketplace. Assessment costs typically range from **$35,000 to $75,000**, but can exceed $100,000 for complex environments. The DoD estimates total triennial costs (initial assessment + two annual affirmations) of **$105,000–$118,000** for small entities.

3. **Assessment.** The C3PAO team conducts an on-site assessment over several days, examining your technical controls, policies, procedures, and evidence artifacts. They interview staff, inspect configurations, and verify that controls are not just documented but actually implemented.

4. **Scoring.** The assessor scores you against all 110 requirements. You achieve one of three outcomes:
   - **Level 2 Final** — All controls met. Certification valid for 3 years.
   - **Conditional Level 2** — At least 88 of 110 controls met (80%), with remaining gaps documented in POA&Ms. You have 180 days to close all POA&Ms and pass a closeout assessment.
   - **Not Met** — Below 88 controls. No certification. You must remediate and re-engage.

5. **Affirmation.** After certification, a senior company official must submit an annual affirmation confirming continued compliance.

---

### SPRS Scores: Your Cybersecurity Credit Score

The Supplier Performance Risk System (SPRS) score is the numerical representation of your NIST 800-171 compliance. Contracting officers check SPRS scores before awarding contracts—if your score isn't posted or is too low, you won't win the contract.

**How it's calculated:**
- Start with a maximum of **110 points**
- Each unimplemented control deducts 1, 3, or 5 points based on the severity of the gap
- No partial credit—if a control isn't fully implemented, you lose the full point value
- Scores range from **110** (perfect) to **-203** (nothing implemented)

**Minimum score for Conditional Level 2:** **88 out of 110**

Your SPRS score must be submitted through the SPRS portal, must be based on an assessment conducted within the last three years, and must be current at the time of contract award.

---

### POA&Ms: What You Can and Can't Defer

Plans of Action and Milestones (POA&Ms) document security gaps you haven't closed yet—a remediation plan with timelines.

Under CMMC 2.0, POA&Ms are allowed under strict conditions:

- **Level 1:** No POA&Ms permitted. All 17 controls must be met.
- **Level 2:** POA&Ms are permitted **only for 1-point requirements**. High-impact 3-point and 5-point requirements cannot be on a POA&M. Additionally, certain critical controls—including the System Security Plan itself, controls related to external system connections, and certain physical access controls—are explicitly prohibited from POA&Ms regardless of point value.
- **Closeout window:** All POA&M items must be fully remediated and verified through a closeout assessment within **180 days** of the conditional certification. If you miss this deadline, your certification expires.

The narrow exception: The CUI encryption requirement (SC.L2-3.13.11) may be placed on a POA&M if encryption is in place but is not using FIPS-validated cryptographic modules.

---

### The Enclave Strategy: Reducing Your Compliance Surface

Here's the insight that saves most startups: **you don't have to secure your entire company to NIST 800-171 standards.** You only have to secure the systems that process, store, or transmit CUI.

The **enclave strategy** creates a defined, isolated boundary—a CUI enclave—where all government-related work happens. Everything inside the enclave meets the full 110 controls. Everything outside the enclave is your normal commercial IT environment, which doesn't need to meet NIST 800-171.

How to implement an enclave:

1. **Isolate CUI systems.** Create a separate network segment, use separate cloud accounts (e.g., a dedicated AWS GovCloud or Azure Government tenant), or use dedicated workstations for CUI processing.

2. **Separate identities.** Users accessing the CUI enclave should use dedicated credentials, not their normal corporate login.

3. **Restrict data flow.** CUI should never leave the enclave boundary. Implement data loss prevention (DLP) controls, disable USB ports in the enclave, and restrict file sharing to approved channels.

4. **Minimize scope.** Only users who need to access CUI should have enclave access. If you have 50 employees but only 15 work on government contracts, only those 15 need enclave access—and only their enclave-facing systems are in scope.

5. **Document the boundary.** Your SSP must clearly define the enclave boundary—which systems, networks, and users are in scope and which are out.

The enclave approach dramatically reduces the cost and complexity of CMMC compliance. Instead of hardening 50 laptops, a full corporate network, and your commercial cloud infrastructure, you're hardening 15 laptops and a dedicated government cloud environment. The controls still apply in full—you're not reducing the requirements, you're reducing the area of application.

**Managed CUI enclave services.** Several companies now offer fully managed CUI enclave solutions—hosted environments pre-configured to meet NIST 800-171 requirements. Companies like PreVeil, Summit 7, and others provide cloud-based enclaves where you can process CUI without building the infrastructure yourself. Costs typically range from $20–$50 per user per month, which is dramatically less than building your own compliant infrastructure.

---

## Part II: Product Cybersecurity — Getting Your Software Authorized (ATO)

If CMMC is the wall for your *company*, the Authority to Operate (ATO) is the wall for your *product*. No software deploys to a DoD network without an ATO. Period.

### What an ATO Actually Is

An Authority to Operate is a formal authorization from a senior DoD official—the **Authorizing Official (AO)**—that your software system has been assessed, that the risks have been evaluated, and that the AO accepts those risks and authorizes the system to operate within a DoD environment.

The AO is typically a senior military or civilian official (often a general officer or senior executive) who has the authority—and the personal accountability—to accept risk on behalf of the organization. This isn't a rubber stamp. The AO's career is on the line. If they authorize a system that gets hacked, they're held accountable.

This means the ATO decision is fundamentally a **risk acceptance decision**, not a pass/fail test. The AO reviews the assessment results, weighs the residual risks against the mission need, and decides whether the system should operate. Some AOs are risk-averse and demand near-perfect compliance. Others will accept more risk if the mission need is urgent.

---

### The NIST Risk Management Framework: Seven Steps

The ATO process is governed by the **NIST Risk Management Framework (RMF)**, defined in NIST SP 800-37 Revision 2. The RMF follows seven steps:

**Step 1 — Prepare**

Establish context for managing security and privacy risk at both the organizational and system levels. Define roles, identify stakeholders, determine the system boundary, and establish the governance structure for risk decisions.

**Step 2 — Categorize**

Classify the information system based on the potential impact of a security breach. Using FIPS 199 ("Standards for Security Categorization of Federal Information and Information Systems"), you assess the potential impact on **confidentiality, integrity, and availability** as Low, Moderate, or High. Most DoD systems are categorized as Moderate or High.

The categorization determines which security controls apply and how stringent the assessment will be.

**Step 3 — Select**

Based on the system's categorization, select security controls from **NIST SP 800-53 Rev 5** ("Security and Privacy Controls for Information Systems and Organizations"). NIST 800-53 contains over 1,000 individual controls across 20 control families. You select the baseline that matches your categorization (Low, Moderate, or High) and then tailor it to your specific system.

For a Moderate system, expect approximately 325 applicable controls. For a High system, approximately 421.

**Step 4 — Implement**

Implement the selected controls in your system and document how each control is satisfied. This is where the work happens—configuring systems, writing code to meet security requirements, establishing procedures, and creating the evidence that proves each control is met.

**Step 5 — Assess**

An independent assessor evaluates whether the controls are implemented correctly, operating as intended, and producing the desired security outcomes. For DoD systems, this assessment is typically conducted by a Security Control Assessor (SCA) from the Defense Information Systems Agency (DISA) or a qualified assessment team.

**Step 6 — Authorize**

The Authorizing Official reviews the complete authorization package—the System Security Plan (SSP), Security Assessment Report (SAR), Plans of Action and Milestones (POA&Ms), and risk determination—and makes the risk acceptance decision. If the AO accepts the risk, they issue the ATO.

**Step 7 — Monitor**

After authorization, continuously monitor the system's security posture. Track changes, reassess controls, report incidents, and update the authorization as the system evolves.

---

### Why Traditional ATO Kills Software Deals

The traditional RMF/ATO process, as practiced by most DoD organizations, is the single biggest obstacle to deploying software in the defense environment. Here's why:

**Timeline: 9–18+ months.** The traditional ATO process for a new system takes 9 to 18 months from start to finish. Some programs have taken over two years. For a startup shipping software updates weekly, waiting 18 months for permission to deploy is a death sentence.

**Cost: $500K–$2M+.** The documentation, assessment, and remediation costs for a traditional ATO are enormous. You need to produce a System Security Plan (typically 200+ pages), complete security control assessments, conduct penetration testing, generate a Security Assessment Report, and address findings. Many companies hire specialized ATO consultants or Assessment & Authorization (A&A) firms at $150–$300/hour.

**Point-in-time snapshot.** A traditional ATO authorizes a specific version of your software at a specific point in time. Every significant change—a new feature, a library update, an infrastructure change—theoretically requires re-assessment or at minimum a formal change request through the AO. This fundamentally conflicts with modern software development, where code changes happen daily.

**Paper-oriented.** Traditional ATO produces binders (sometimes literally) of documentation that prove compliance at the moment of assessment but say nothing about the system's security posture six months later.

**The result:** Software companies win contracts, build products, and then watch those products sit idle for 12–18 months while the ATO process grinds through. By the time the software is authorized, the technology has evolved, requirements have changed, and the original value proposition has degraded. Many deals die in the ATO queue.

---

### Continuous ATO: The Game Changer

In February 2022, the DoD CIO issued a memorandum that fundamentally changed the ATO landscape: the directive establishing **continuous Authorization to Operate (cATO)**.

The 2022 memorandum acknowledged that traditional RMF implementations, while establishing the framework for continuous risk management, had devolved into "paper-oriented authorizations" that failed to continuously monitor risk after the initial authorization. The memo set out to "raise the bar" by requiring three core competencies for cATO:

**1. Continuous Monitoring (CONMON)**

Real-time or near-real-time visibility into the cybersecurity activities within the system boundary. This means automated vulnerability scanning, continuous compliance monitoring of RMF controls, and active dashboard reporting—not quarterly manual reviews.

**2. Active Cyber Defense (ACD)**

The ability to detect and respond to cyber threats in real time. This includes intrusion detection, endpoint detection and response (EDR), security information and event management (SIEM) integration, and incident response automation.

**3. Approved DevSecOps Reference Design**

The adoption of an approved DevSecOps pipeline with a secure software supply chain. This is the requirement that ties cATO to modern software development practices (more on this below).

**The transformative benefit:** A DoD CISO-approved cATO **has no expiration date**. Unlike a traditional 3-year ATO that requires periodic re-authorization, a cATO remains in effect as long as the system maintains its required real-time risk posture. This means your software can be continuously updated, deployed, and improved without going through a full re-authorization cycle for each change.

The April 2024 **DevSecOps Continuous Authorization Implementation Guide**, released by the DoD CIO, provided further direction on achieving cATO for DevSecOps platforms and applications produced by software factories, including requirements for a Secure Software Supply Chain (SSSC) and Software Bills of Materials (SBOMs).

---

### The DevSecOps Pipeline: What cATO Actually Requires

cATO isn't just a new acronym for the same process. It requires a fundamentally different approach to software security—one where security is embedded in your development pipeline, not bolted on at the end.

A compliant DevSecOps pipeline for cATO includes:

**Automated Security Testing in CI/CD**

- **Static Application Security Testing (SAST):** Automated code analysis that identifies vulnerabilities in source code before deployment. Tools like SonarQube, Checkmarx, or Fortify scan every code commit.
- **Dynamic Application Security Testing (DAST):** Automated testing of running applications to find vulnerabilities that only manifest in execution. Tools like OWASP ZAP or Burp Suite run against deployed versions.
- **Software Composition Analysis (SCA):** Automated scanning of third-party libraries and open-source components for known vulnerabilities. Tools like Snyk, Black Duck, or Dependabot.
- **Container Scanning:** If you deploy in containers (and for DoD, you almost certainly will), automated scanning of container images for vulnerabilities, misconfigurations, and compliance issues. Tools like Trivy, Anchore, or Prisma Cloud.
- **Infrastructure as Code (IaC) Scanning:** Automated compliance checks on your infrastructure definitions (Terraform, CloudFormation, Helm charts). Tools like Checkov, cfn-lint, or Terrascan.

**Policy-as-Code Enforcement**

NIST 800-53 controls codified as automated policy checks. Instead of manually verifying that encryption is enabled on your database, your pipeline includes a policy check that blocks deployment if encryption is not configured. Tools like Open Policy Agent (OPA) or HashiCorp Sentinel enable this.

**Automated Evidence Collection**

Every security scan, every policy check, every vulnerability assessment generates evidence artifacts that are automatically collected and stored. When the AO asks for proof that you tested for SQL injection, you don't dig through email—your pipeline produces the evidence automatically.

**Real-Time Compliance Dashboards**

A continuously updated view of your system's compliance posture: which controls are met, which are degraded, what vulnerabilities exist, what the current risk level is. This replaces the 200-page static SSP with a living, real-time representation of your security posture.

**Secure Software Supply Chain**

- All dependencies tracked and assessed
- Software Bill of Materials (SBOM) generated automatically with each build
- Provenance verification for all third-party components
- Approved artifact repositories (like Iron Bank) for container images

---

### DoD Software Factories: Your ATO Accelerator

As covered briefly in Chapter 6, DoD software factories are centralized platforms that provide pre-authorized development and deployment environments. For software companies seeking ATO, software factories are the single most important accelerator available.

**Platform One (Air Force)**

The Air Force's enterprise DevSecOps platform. Platform One operates under a cATO, meaning applications deployed through Platform One's pipeline can inherit significant portions of the platform's authorization. Instead of authorizing your entire stack from scratch, you only need to authorize the delta—your application code and configuration on top of Platform One's already-authorized infrastructure.

**Iron Bank**

A component of Platform One, Iron Bank is the DoD's trusted repository of vetted and hardened container images. Every container in Iron Bank undergoes daily vulnerability scanning, and comprehensive risk assessments and compliance reports are available for each image.

For software companies, Iron Bank is critical in two ways:
1. **Use Iron Bank images as your base.** If you build your containers on Iron Bank-approved base images, you inherit the security posture of those base images and dramatically reduce your own assessment scope.
2. **Get your containers into Iron Bank.** If your software is approved for inclusion in Iron Bank, any DoD organization can deploy your containers with significantly reduced ATO overhead. This is a powerful distribution channel.

**Black Pearl (Navy)**

The Navy's equivalent of Platform One. Black Pearl provides secure, Navy-accredited DevSecOps services and partners with Platform One for products like Iron Bank and Repo One (source code repository). Deploying through Black Pearl gives you access to Navy-accredited infrastructure.

**Army Software Factory (ASWF)**

The Army's initiative launched in 2021, focused on training soldiers in commercial software development practices and building Army-specific applications. The ASWF uses agile development, CI/CD pipelines, and soldier-centered design.

**The Strategy:** If you can deploy your software through a DoD software factory, you can potentially reduce your ATO timeline from 12–18 months to weeks or a few months. The factory's cATO covers the platform; you only need to authorize your application on that platform. This is the single most effective ATO acceleration strategy available to startups.

---

### FIPS 140-2/140-3: Cryptographic Requirements

FIPS 140 (Federal Information Processing Standard 140) governs the use of cryptographic modules in federal systems. If your software handles CUI or operates in a DoD environment, you must use **FIPS-validated encryption**.

**FIPS 140-2 vs. 140-3:**
- FIPS 140-2 has been the standard for decades. All new validations must now use FIPS 140-3, and FIPS 140-2 certificates will be moved to a historical list by September 21, 2026.
- FIPS 140-3 became the active standard for new validations in 2022.

**What this means in practice:**

1. **Use FIPS-validated cryptographic libraries.** Don't implement your own encryption. Use validated modules like OpenSSL (with FIPS provider), AWS-LC, BoringCrypto (Go), or the Windows CNG provider. Check the NIST Cryptographic Module Validation Program (CMVP) database to verify that the specific version you're using has a current validation certificate.

2. **Enable FIPS mode.** Many operating systems and runtimes have a "FIPS mode" that restricts cryptographic operations to validated algorithms and modules. In practice, this means:
   - TLS 1.2 or higher (TLS 1.0 and 1.1 are prohibited)
   - AES-128 or AES-256 for symmetric encryption
   - SHA-256 or higher for hashing
   - RSA-2048+ or ECDSA with approved curves for digital signatures
   - No MD5, no DES, no RC4, no SHA-1 for security-relevant operations

3. **Don't break your application.** FIPS mode can break dependencies that use non-FIPS algorithms (common in JavaScript libraries, Python packages, and older Java code). Test your entire application in FIPS mode early in development, not as an afterthought.

4. **Cloud FIPS.** Major cloud providers offer FIPS endpoints: AWS GovCloud, Azure Government, and Google Cloud FIPS endpoints all use FIPS-validated modules for data in transit and at rest.

---

## Part III: Cloud and SaaS Compliance

### FedRAMP: The Cloud Authorization Framework

FedRAMP (Federal Risk and Authorization Management Program) is the federal government's standardized approach to security assessment and authorization for cloud service providers. If you're selling a SaaS product to the government, FedRAMP is likely in your future—but maybe not in the way you think.

**The Three Authorization Levels:**

| Level | ~Controls | Typical Cost | Timeline | Use Case |
|---|---|---|---|---|
| **FedRAMP Low** | 125 | $250K–$500K | ~12 months | Low-risk data, limited PII |
| **FedRAMP Moderate** | 325 | $500K–$1.5M | 12–18 months | Most federal SaaS (CUI, financial, HR data). This is the most common level. |
| **FedRAMP High** | 421 | $1M–$3M+ | 18–36 months | Most sensitive unclassified data: national security, critical infrastructure, law enforcement |

**Authorization Paths:**
- **Agency Authorization:** A single federal agency sponsors your authorization. Faster and less expensive but initially scoped to that agency.
- **JAB Authorization (Joint Authorization Board):** The multi-agency authorization endorsed by the JAB (DHS, DoD, GSA). More rigorous and expensive but allows government-wide reuse.

**Annual Maintenance:** $200K–$500K+ per year for continuous monitoring, annual assessments, and maintaining your authorization.

**FedRAMP 20x:** A new automation-focused authorization program launched in 2025. The Phase 1 Low Impact pilot concluded in September 2025, with Phase 2 (Moderate) launching in November 2025. Wide-scale adoption is projected for FY2026 Q3-Q4. This may significantly reduce timelines and costs for future authorizations.

**The Bottom Line:** FedRAMP is expensive and time-consuming. Most startups should not pursue FedRAMP as their first move into government. It makes sense when you have multiple federal customers across agencies and need a single reusable authorization. For DoD-only sales, a DoD-specific ATO (especially through a software factory) is often faster and cheaper.

---

### DoD Cloud Computing SRG: Impact Levels Explained

The DoD Cloud Computing Security Requirements Guide (CC SRG), managed by the Defense Information Systems Agency (DISA), defines **Impact Levels** that determine what kind of data your cloud environment can handle. FedRAMP serves as the minimum baseline, with the SRG adding DoD-specific requirements on top.

**IL2 — Public and Non-Critical Information**

- Data: Publicly releasable information and non-critical mission data
- Baseline: FedRAMP Moderate authorization is generally sufficient
- Personnel: Standard background checks
- Infrastructure: Commercial cloud (no dedicated DoD infrastructure required)
- Most commercial SaaS products can meet IL2

**IL4 — Controlled Unclassified Information (CUI)**

- Data: CUI that requires safeguarding
- Baseline: FedRAMP Moderate + DoD-specific enhancements (additional access management, logging, and audit controls)
- Personnel: Background investigation for cloud service personnel
- Infrastructure: Can be on commercial cloud infrastructure with proper controls
- **This is where most defense software startups should target**

**IL5 — Mission-Critical CUI and National Security Systems**

- Data: Higher-sensitivity CUI and mission data where compromise could be devastating
- Baseline: FedRAMP High + additional DoD enhancements (421+ controls)
- Personnel: **U.S. citizens only** for all personnel with access
- Infrastructure: **Dedicated infrastructure**—physically and logically separated from commercial cloud customers
- Cloud providers: AWS GovCloud (IL5), Azure Government (IL5), Google Cloud IL5 environments

**IL6 — Classified (SECRET)**

- Data: Classified information up to SECRET
- Baseline: Beyond FedRAMP; dedicated classified cloud enclaves
- Infrastructure: Connected to SIPRNet, physically isolated
- Most startups will never need IL6

**How to decide:** If your software handles CUI (which most defense software does), target **IL4** as your initial deployment level. If your customer is a combatant command or a highly sensitive program, you may need **IL5**. Discuss the required Impact Level with your government customer early—it determines your entire cloud architecture.

---

### JWCC: Where Your Software Lives

The Joint Warfighting Cloud Capability (JWCC) is the DoD's multi-cloud contract vehicle, replacing the failed single-vendor JEDI program. JWCC contracts are held by AWS, Microsoft Azure, Google Cloud, and Oracle, providing cloud services across all classification levels.

**Why it matters for startups:** If your software runs in the cloud (and it almost certainly does), the DoD will want it deployed on JWCC-authorized cloud infrastructure. Your cloud hosting needs to connect to—or run within—one of these four providers' DoD-authorized environments.

This doesn't mean you need your own JWCC contract. It means your software needs to be architected to run on one of these cloud platforms at the appropriate Impact Level. If you're already on AWS or Azure, you're halfway there—you just need to deploy in the GovCloud or Government region at the right IL.

---

### The "FedRAMP or Equivalent" Trap

DFARS 252.204-7012 requires that contractors using external cloud service providers to store CUI must ensure those CSPs meet requirements "equivalent to FedRAMP Moderate." For years, what "equivalent" meant was unclear, creating ambiguity that contractors and cloud providers exploited.

A **December 2023 DoD CIO memorandum** clarified the standard. "FedRAMP Moderate equivalent" now means:

1. **3PAO assessment** at 100% compliance with the FedRAMP Moderate baseline (all 325 controls)
2. **Zero open POA&Ms** from the assessment
3. **Full body of evidence** (SSP, SAP, SAR, POA&Ms) provided to the contractor and DoD
4. **Annual reassessment** by a 3PAO
5. **Compliance with DFARS 7012** requirements for cyber incident reporting, forensic evidence preservation, and access for damage assessment

In practical terms: if you're a SaaS company and your customers are DoD contractors, either get FedRAMP authorized or prove equivalence through a rigorous 3PAO assessment. Using a commercial cloud deployment and calling it "FedRAMP equivalent" without the 3PAO assessment is no longer defensible.

---

## Part IV: Secure Software Development

### The Secure Software Development Framework (SSDF)

NIST SP 800-218 defines the Secure Software Development Framework—a set of secure development practices that federal agencies now require software producers to **attest to**.

This requirement flows from Executive Order 14028 (May 2021, "Improving the Nation's Cybersecurity") and is operationalized through OMB Memoranda M-22-18 and M-23-16. Software producers selling to federal agencies must complete a **Secure Software Development Attestation Form** (released by CISA in March 2024) affirming that they follow SSDF practices.

The SSDF organizes practices into four groups:
- **Prepare the Organization (PO):** Establish policies, define roles, implement tooling
- **Protect the Software (PS):** Secure development environments, protect code integrity, verify provenance
- **Produce Well-Secured Software (PW):** Design for security, review code, test for vulnerabilities, configure defaults securely
- **Respond to Vulnerabilities (RV):** Identify and remediate vulnerabilities, analyze root causes

For software companies, this means:
- You need a documented secure development policy
- Your CI/CD pipeline must include automated security testing
- You must track and respond to vulnerabilities in your software
- You must be able to attest to these practices formally

---

### Software Bill of Materials (SBOM)

Executive Order 14028 also mandated that federal agencies obtain SBOMs for acquired software. An SBOM is essentially a detailed inventory of every component in your software—every library, framework, module, and dependency, including version numbers, license information, and cryptographic hashes.

**Why it matters:** When a new vulnerability is discovered (like Log4Shell), the government needs to know immediately which of its systems are affected. An SBOM provides that visibility. Without one, you're searching through thousands of applications hoping to find where affected components are deployed.

**NTIA Minimum Elements for an SBOM:**
- Component name and version
- Supplier/vendor name
- Unique identifier (e.g., CPE, PURL, SWID tag)
- Dependency relationships
- Author of the SBOM
- Timestamp

**How to generate SBOMs:**
- **CycloneDX** and **SPDX** are the two standard SBOM formats
- Most modern build tools can generate SBOMs: `syft`, `cdxgen`, `trivy`, or native tooling in npm, pip, Maven
- Integrate SBOM generation into your CI/CD pipeline so every build automatically produces an updated SBOM
- Store SBOMs as artifacts alongside your releases

---

### Open-Source Software in DoD

The DoD doesn't prohibit open-source software. In fact, the DoD CIO has issued guidance supporting the use of open source. But open-source components must be managed and vetted:

- **Iron Bank model:** Containerize open-source components and submit them for hardening and vetting through Iron Bank. Once approved, they're available to the entire DoD.
- **Vulnerability management:** You are responsible for tracking vulnerabilities in every open-source component you use and patching or mitigating them promptly.
- **License compliance:** Some open-source licenses (particularly copyleft licenses like GPL) may create complications for government use. Verify that your open-source components' licenses are compatible with your delivery model.
- **Supply chain integrity:** Verify the provenance of open-source packages. Use signed packages, verified registries, and dependency pinning to prevent supply chain attacks.

---

## Part V: Practical Guidance for Software Startups

### The Six Most Common Mistakes

**1. Confusing CMMC and ATO**

CMMC certifies your *organization's* cybersecurity. ATO authorizes your *product's* deployment. You need both, and they're different compliance regimes with different requirements, different assessors, and different timelines. A CMMC Level 2 certification does not give your software an ATO. An ATO does not satisfy CMMC requirements.

**2. Treating compliance as an afterthought**

If you architect your software without considering DoD security requirements, you'll face months of rework when ATO assessment reveals that your application doesn't support FIPS encryption, can't generate audit logs at the required granularity, or doesn't implement role-based access controls that map to NIST 800-53.

Design for compliance from day one. Use FIPS-validated crypto libraries. Implement comprehensive audit logging. Build role-based access controls. Generate SBOMs automatically. These are easier to build in than to bolt on.

**3. Not understanding your ATO path**

"We'll just get an ATO" is not a plan. You need to know: Which AO will authorize your system? Which program office is sponsoring the ATO? What Impact Level do you need? Can you deploy through a software factory? What assessment team will evaluate your controls? Answer these questions before you write your first line of government-facing code.

**4. Going straight to FedRAMP**

Many startups assume they need FedRAMP to sell to DoD. Usually, they don't—at least not initially. A DoD-specific ATO through a software factory or a program-level authorization is faster, cheaper, and sufficient for most early contracts. Pursue FedRAMP when you have multiple agency customers and need government-wide authorization.

**5. Failing to separate CUI environments**

If CUI sprawls across your entire corporate network—developer laptops, Slack channels, Google Drive—your entire company is in scope for CMMC. Use the enclave strategy from day one to isolate CUI and keep your CMMC scope manageable.

**6. Underestimating timelines and costs**

Traditional ATO: 9-18 months and $500K-$2M. CMMC Level 2 certification: 6-12 months and $100K-$250K+ total. FedRAMP Moderate: 12-18 months and $500K-$1.5M plus $200K-$500K/year maintenance. Budget for these. Plan for these. Don't discover them after you've won the contract.

---

### The Cost of Compliance

| Compliance Item | Estimated Cost | Timeline |
|---|---|---|
| CMMC Level 1 (self-assessment) | $5K–$15K (consultant review) | 1–3 months |
| CMMC Level 2 readiness assessment | $15K–$50K | 2–4 months |
| CMMC Level 2 C3PAO certification | $35K–$75K (assessment only) | 3–6 months |
| CMMC Level 2 total first-year (remediation + assessment) | $70K–$250K+ | 6–12 months |
| CMMC Level 2 triennial (3-year cycle with affirmations) | $105K–$118K | 3 years |
| Traditional ATO (from scratch) | $500K–$2M+ | 9–18+ months |
| ATO via software factory (Platform One, Black Pearl) | $50K–$200K | 2–6 months |
| FedRAMP Low | $250K–$500K + $100K–$200K/yr | ~12 months |
| FedRAMP Moderate | $500K–$1.5M + $200K–$500K/yr | 12–18 months |
| FedRAMP High | $1M–$3M+ + $500K–$1M/yr | 18–36 months |
| Managed CUI enclave service | $20–$50/user/month | Ongoing |
| FIPS 140-3 validation (own module) | $100K–$500K+ | 6–18 months |

---

### A Phased Roadmap for Software Startups

**Stage 1: Before Your First DoD Engagement ($0–$15K)**

- Implement basic IT hygiene (MFA on all accounts, encrypted devices, endpoint protection)
- Begin mapping your software to NIST 800-53 controls at a high level
- Use FIPS-validated encryption libraries in your software from day one
- Integrate SAST, DAST, and SCA into your CI/CD pipeline
- Generate SBOMs automatically with each build
- Start your NIST 800-171 self-assessment to understand your current gaps

**Stage 2: First OT, SBIR, or Commercial-Item Contract ($15K–$75K)**

- Implement the enclave strategy to isolate CUI processing
- Complete CMMC Level 1 self-assessment (required for contracts with FCI)
- Begin CMMC Level 2 preparation: implement NIST 800-171 controls within the enclave
- Submit initial SPRS score
- Identify your ATO path: can you deploy through a software factory?
- Engage a CMMC RP or consultant for a readiness assessment

**Stage 3: First CUI Contract Requiring CMMC Level 2 ($75K–$250K)**

- Complete CMMC Level 2 self-assessment or engage a C3PAO for certification
- Begin the ATO process: identify your AO, program office sponsor, and assessment team
- If using a software factory: containerize your application, submit containers to Iron Bank, deploy through Platform One or Black Pearl
- If pursuing independent ATO: engage an A&A consultant, begin SSP documentation, prepare for security control assessment

**Stage 4: Scaling Government Revenue ($250K–$1M+)**

- Pursue cATO through a mature DevSecOps pipeline
- Consider FedRAMP if you have multiple agency customers
- Target IL4 or IL5 cloud deployment environments
- Build an internal compliance team or retain ongoing compliance support
- Pursue reciprocity across programs and services (one ATO accepted by multiple AOs)

---

## Key Terms

**CUI (Controlled Unclassified Information):** Government-created or -owned information that requires safeguarding but is not classified. Encompasses dozens of categories including technical data, export-controlled information, and PII.

**NIST SP 800-171:** The 110 security controls in 14 families that govern protection of CUI in non-federal systems. Currently Rev 2 is mandated; Rev 3 was published May 2024.

**DFARS 252.204-7012:** The DFARS clause requiring implementation of NIST 800-171, cyber incident reporting within 72 hours, and FedRAMP equivalence for cloud services handling CDI.

**CMMC 2.0 (Cybersecurity Maturity Model Certification):** DoD program to verify contractor compliance with cybersecurity requirements. Three levels: L1 (17 controls, self-assessment), L2 (110 controls, self or C3PAO assessment), L3 (110+ controls, government assessment).

**C3PAO (Certified Third-Party Assessment Organization):** Organizations accredited by the Cyber AB to conduct CMMC Level 2 certification assessments.

**SPRS (Supplier Performance Risk System):** The portal where contractors submit their NIST 800-171 compliance scores. Scores range from 110 (perfect) to -203. Minimum 88 required for Conditional Level 2.

**POA&M (Plan of Action and Milestones):** Documents identifying security gaps and remediation timelines. Under CMMC Level 2, limited to 1-point requirements with a 180-day closeout window.

**Enclave:** An isolated computing environment configured to meet NIST 800-171 requirements for CUI processing, limiting the compliance scope to a defined boundary.

**ATO (Authority to Operate):** Formal authorization from an Authorizing Official permitting a system to operate in a DoD environment. Based on risk acceptance after security assessment.

**Authorizing Official (AO):** The senior DoD official with authority—and accountability—to accept risk and authorize a system to operate.

**RMF (Risk Management Framework):** The NIST 800-37 seven-step process governing information security risk management: Prepare, Categorize, Select, Implement, Assess, Authorize, Monitor.

**NIST SP 800-53:** The comprehensive catalog of security and privacy controls used in the RMF. Rev 5 contains 1,000+ controls across 20 families.

**FIPS 199:** Standards for categorizing information systems by potential impact (Low/Moderate/High) on confidentiality, integrity, and availability.

**cATO (continuous ATO):** The modernized ATO approach established by the February 2022 DoD CIO memo. Requires continuous monitoring, active cyber defense, and an approved DevSecOps pipeline. Has no expiration date.

**DevSecOps:** Integration of security practices into the software development lifecycle. Includes automated SAST, DAST, SCA, container scanning, and IaC scanning in CI/CD pipelines.

**Platform One:** The Air Force's enterprise DevSecOps platform operating under a cATO. Deploying through Platform One can dramatically accelerate ATO timelines.

**Iron Bank:** DoD trusted repository of vetted and hardened container images. Part of Platform One. Daily vulnerability scanning, compliance reports, and risk assessments for each image.

**Black Pearl:** The Navy's DevSecOps platform, analogous to Platform One. Partners with Platform One for Iron Bank and Repo One.

**Army Software Factory (ASWF):** The Army's initiative for training soldiers in software development and building Army applications using commercial practices.

**FIPS 140-2/140-3:** Federal standards for cryptographic module validation. FIPS 140-3 is the current standard for new validations; FIPS 140-2 certificates move to historical status by September 2026.

**FedRAMP:** Federal cloud security authorization program. Levels: Low (~125 controls), Moderate (~325), High (~421). Costs range from $250K to $3M+ for initial authorization.

**FedRAMP 20x:** A new automation-focused authorization pathway launched in 2025, designed to reduce timelines and costs.

**DoD CC SRG (Cloud Computing Security Requirements Guide):** DISA-managed guide defining Impact Levels for DoD cloud services.

**Impact Level (IL):** DoD classification of cloud data sensitivity. IL2 (public/non-critical), IL4 (CUI), IL5 (mission-critical CUI, dedicated infrastructure, US-only), IL6 (classified/SECRET).

**JWCC (Joint Warfighting Cloud Capability):** DoD's multi-cloud contract with AWS, Azure, Google Cloud, and Oracle, providing cloud services across all classification levels.

**SSDF (Secure Software Development Framework):** NIST SP 800-218 practices for secure software development. Federal software suppliers must attest to compliance via the CISA attestation form.

**SBOM (Software Bill of Materials):** An inventory of all components in your software, including versions, suppliers, and dependencies. Required by EO 14028. Standard formats: CycloneDX and SPDX.

**SAST (Static Application Security Testing):** Automated analysis of source code for vulnerabilities before deployment.

**DAST (Dynamic Application Security Testing):** Automated testing of running applications for vulnerabilities in execution.

**SCA (Software Composition Analysis):** Automated scanning of third-party and open-source dependencies for known vulnerabilities.
