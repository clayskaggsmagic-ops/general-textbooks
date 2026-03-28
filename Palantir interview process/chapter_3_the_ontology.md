# Chapter 3: The Ontology — Palantir's Core Intellectual Framework

> *"The Ontology is the single most important concept in the Palantir ecosystem. If you understand nothing else, understand this."*

---

## Why This Chapter Matters

In Chapter 2 we surveyed four platforms — Gotham, Foundry, Apollo, AIP. Each appeared distinct, yet one abstraction threads through all of them like a spine: the **Ontology**. Alex Karp has called it the company's "durable competitive advantage." Shyam Sankar describes it as a "digital twin of the organization." Internal documentation positions it as "an API for your enterprise." A Deployment Strategist who cannot explain the Ontology — what it is, how it's built, why it matters — will not pass the interview. This chapter gives you everything you need.

---

## 3.1 What Is an Ontology?

### 3.1.1 The Computer Science Definition

In computer science, an **ontology** is a formal, explicit specification of a shared conceptualization. It defines:

| Element | Description | Example |
|---|---|---|
| **Individuals** | Specific instances of things | `Patient_42901` |
| **Classes** | Categories that group individuals | `Patient`, `Physician`, `Medication` |
| **Attributes** | Properties of individuals | `date_of_birth`, `dosage_mg` |
| **Relations** | How individuals connect | `prescribed_by`, `admitted_to` |
| **Axioms** | Logical rules that constrain the model | "Every `Prescription` must link to exactly one `Physician`" |

The Semantic Web community (W3C, OWL, RDF) popularized this idea: if systems agree on a shared vocabulary and its logical structure, they can interoperate without custom integrations.

### 3.1.2 The Palantir Definition

Palantir takes the academic idea and makes it **operational**. In Palantir's ecosystem, the Ontology is:

> **A live, editable, governed semantic layer that maps an organization's raw data to the real-world entities, relationships, and operations it represents — creating a "digital twin" of the business that humans, applications, and AI agents all interact with through the same interface.**

Key distinctions from the academic concept:

| Dimension | Academic Ontology | Palantir Ontology |
|---|---|---|
| **Purpose** | Knowledge representation & reasoning | Operational decision-making |
| **State** | Static schema | Live, continuously updated |
| **Writability** | Read-only inference | Read-write via Actions |
| **Users** | Researchers, knowledge engineers | Operators, analysts, executives, AI agents |
| **Deployment** | Standalone reasoning engines | Embedded in applications (Workshop, Slate, AIP) |
| **Governance** | Schema validation | RBAC, audit trails, row-level security |
| **Scale** | Thousands of triples | Billions of objects across petabytes |

### 3.1.3 The "Digital Twin" Analogy

Think of the Ontology as a fully interactive map of how your organization actually works:

- Every **customer**, **order**, **machine**, **patient**, **shipment**, and **employee** exists as a first-class digital object.
- Every **relationship** between them (Customer → places → Order; Machine → located_at → Factory) is an explicit, navigable link.
- Every **operation** that changes reality (Approve Claim, Dispatch Vehicle, Administer Medication) is a governed, auditable action.
- The map is **live** — backed by real data flowing from source systems via pipelines, updated on schedules measured in minutes or seconds.

This is what Palantir means when they say the Ontology creates a "digital twin of the organization."

---

## 3.2 The Four Building Blocks

The Ontology is composed of four interlocking primitives. Master these, and you can design an Ontology for any domain.

### 3.2.1 Object Types

**Definition:** An Object Type is the schema definition for a real-world entity or event. Each individual instance is called an **Object**; a collection of Objects is an **Object Set**.

**Analogy:**
| Database Concept | Ontology Concept |
|---|---|
| Table | Object Type |
| Row | Object |
| Result set | Object Set |

**Characteristics:**
- **Backed by datasets** — every Object Type maps to one or more underlying datasets (Parquet files, database tables, API feeds). The Ontology does not store data independently; it provides a semantic view over existing data.
- **Primary key required** — each Object must have a unique identifier (a `propertyId` + value pair). String-based primary keys are preferred over numeric IDs because they support UUIDs, composite keys, and systems where IDs may become alphanumeric.
- **Created in Ontology Manager** — the administrative interface within Foundry for defining Object Types, mapping properties, and configuring pipelines.

**Examples:**

| Domain | Object Types |
|---|---|
| Healthcare | `Patient`, `Physician`, `Medication`, `Encounter`, `Insurance_Claim` |
| Supply Chain | `Warehouse`, `Shipment`, `Supplier`, `Purchase_Order`, `SKU` |
| Defense | `Mission`, `Asset`, `Unit`, `Target`, `Intelligence_Report` |
| Financial Services | `Customer`, `Account`, `Transaction`, `Risk_Score`, `Alert` |

> **Interview tip:** When asked to "design an ontology," start by listing the **nouns** of the domain. Each noun is a candidate Object Type. Ask yourself: *What are the entities that people make decisions about?*

---

### 3.2.2 Properties

**Definition:** Properties are the attributes that describe an Object Type. They are analogous to columns in a database table.

**Data Types:**
| Type | Examples |
|---|---|
| **String** | `name`, `status`, `category` |
| **Numeric** | `quantity`, `cost`, `latitude` |
| **Boolean** | `is_active`, `is_hazardous` |
| **Timestamp** | `created_at`, `last_modified` |
| **Geospatial** | `location` (lat/long point or polygon) |
| **Time Series** | `sensor_readings` (array of timestamped values) |
| **Struct** | Nested JSON-like objects |

**Stored vs. Computed Properties:**

| Type | How it works | Example |
|---|---|---|
| **Stored** | Mapped directly from a backing dataset column | `order.total_price` ← `orders_table.total` |
| **Computed (Derived)** | Calculated at runtime from other properties or links | `customer.lifetime_value` = SUM of all linked `order.total_price` |

Derived properties (beta as of late 2024) are computed at query time and respect the security permissions of all involved objects. They are available via the TypeScript OSDK.

**Shared Properties:**
A property definition can be reused across multiple Object Types. For example, a `created_at_timestamp` shared property ensures consistent naming and data typing across `Order`, `Shipment`, `Invoice`, etc.

**Naming Conventions (Best Practice):**
- Use snake_case: `created_at_timestamp`, `total_cost_usd`
- Be specific: `delivery_date` not `date`
- Suffix with type hints where helpful: `_timestamp`, `_id`, `_count`
- Never version property names (`name_v2`); instead, add new properties and deprecate old ones

---

### 3.2.3 Links

**Definition:** A Link Type defines a relationship between two Object Types. A Link is a single instance of that relationship between two specific Objects. Links are analogous to joins between database tables.

**Cardinalities:**

| Cardinality | Description | Example |
|---|---|---|
| **One-to-One** | Each object on side A relates to at most one object on side B | `Employee` ↔ `Laptop` |
| **One-to-Many** | One object on side A relates to many objects on side B | `Customer` → `Order` (one customer, many orders) |
| **Many-to-Many** | Objects on both sides can relate to many | `Student` ↔ `Course` (via enrollment) |

**Self-Referencing Links:**
An Object Type can link to itself. Example: `Employee` → `manages` → `Employee`. This creates hierarchical or graph structures within a single type.

**Implementation:**
Technically, a Link Type maps to a foreign key relationship in the backing data. The Ontology Manager allows you to define which dataset columns serve as the join keys. Unlike traditional joins, however, Ontology links are:
- **Named and directional** — `order.customer` navigates from Order to Customer; `customer.orders` navigates back
- **Traversable at any depth** — you can walk from `Patient` → `Encounter` → `Physician` → `Department` in a single query
- **Security-aware** — if a user lacks permission to see a linked Object, the link returns nothing (not an error)

**Naming Conventions (Best Practice):**
- The API name on the **plural side** should be plural: `employee.subordinates.all()`, not `employee.subordinate.all()`
- Use meaningful names for self-referencing and multi-link scenarios: `manages` vs. `is_managed_by`
- Define foreign keys consistently: `{foreign_object_type}_id` (e.g., `customer_id` on `Order`)

---

### 3.2.4 Actions

**Definition:** An Action Type defines a structured operation that creates, modifies, or deletes Objects, Properties, and Links in a governed, auditable way. Each Action is a single atomic transaction.

**Why not just edit data directly?**
Actions enforce business logic. Instead of a user manually changing a status field, an Action encapsulates:
1. **Parameter collection** — what inputs does the user provide?
2. **Validation logic** — are the inputs valid? Does the user have permission?
3. **Side effects** — what else should happen? (Send notification, update linked objects, trigger a pipeline)
4. **Audit trail** — who did what, when, and why?

**Types of Actions:**

| Type | Description | Example |
|---|---|---|
| **Single-object** | Modifies one Object | "Update Patient Status" |
| **Bulk** | Modifies multiple Objects | "Mark 500 invoices as paid" |
| **Form-based** | Parameterized by user input (UI forms) | "Create New Shipment" dialog |
| **Function-backed** | Complex logic written in TypeScript or Python | "Calculate Optimal Route" using custom algorithm |

**Anatomy of an Action (Example: "Approve Insurance Claim"):**

```
Action Type: Approve Claim
├── Parameters:
│   ├── claim_id (required, string)
│   ├── approved_amount (required, numeric)
│   └── reviewer_notes (optional, string)
├── Validation Rules:
│   ├── claim.status must be "Pending Review"
│   ├── approved_amount ≤ claim.requested_amount
│   └── user must have "Claims_Approver" role
├── Mutations:
│   ├── SET claim.status = "Approved"
│   ├── SET claim.approved_amount = {approved_amount}
│   ├── SET claim.approved_by = {current_user}
│   └── SET claim.approved_at = NOW()
├── Side Effects:
│   ├── CREATE Payment object linked to claim
│   ├── NOTIFY claim.patient via email
│   └── TRIGGER payment_processing_pipeline
└── Audit:
    └── Full record in action log (immutable)
```

> **Interview tip:** When asked "how would users interact with this ontology?", your answer should center on Actions. **Actions are the verbs** of the Ontology — they are how work gets done.

---

## 3.3 The Three Layers

The four building blocks organize into three architectural layers, each serving a different audience and purpose.

```
┌─────────────────────────────────────────────────┐
│          DYNAMIC LAYER                          │
│   Rules · Policies · Permissions · Workflows    │
│   "How the organization governs and enforces"   │
├─────────────────────────────────────────────────┤
│          KINETIC LAYER                          │
│   Actions · Functions · Writeback · Pipelines   │
│   "How the organization operates and changes"   │
├─────────────────────────────────────────────────┤
│          SEMANTIC LAYER                         │
│   Object Types · Properties · Links             │
│   "What the organization IS"                    │
└─────────────────────────────────────────────────┘
              ▲                    ▲
         Raw Data              AI Agents,
       (ERP, CRM,            Users, Apps
      sensors, APIs)        (Workshop, AIP)
```

### 3.3.1 The Semantic Layer — "The Nouns"

**What it answers:** *What entities exist in this organization, and how do they relate to one another?*

The Semantic Layer is the conceptual model — the shared vocabulary that unifies fragmented data. It maps raw tables and columns from source systems to meaningful business objects.

**Key characteristics:**
- Defines Object Types, Properties, and Link Types
- Establishes a **shared language** across teams, departments, and data sources
- Powers ETL pipelines with traceable, auditable **data lineage** from source to insight
- Reconciles fragmented concepts — three departments calling the same entity by different names get mapped to a single Object Type

**Who builds it:** Data engineers and Deployment Strategists, in collaboration with domain experts.

**Analogy:** The Semantic Layer is the *dictionary and grammar* of the organization — what are the nouns, and how do they relate?

---

### 3.3.2 The Kinetic Layer — "The Verbs"

**What it answers:** *How does the organization take action and change state?*

The Kinetic Layer operationalizes the Ontology by introducing behaviors. It defines the "verbs" — the ways the organization interacts with and modifies its digital twin.

**Key components:**

| Component | Purpose | Example |
|---|---|---|
| **Action Types** | Define mutations (create, update, delete objects/links) | "Assign Driver to Route" |
| **Functions** | Encode business logic in TypeScript or Python | Calculate risk score, validate compliance |
| **Writeback** | Push changes from the Ontology back to source systems | Update SAP order status after approval in Foundry |
| **OSDK** | Ontology SDK for programmatic access (TypeScript, Python, Java) | External app reads/writes Ontology objects via API |

**Key characteristics:**
- Every action is **traceable, governed, and executable at scale**
- Functions can range from simple data transformations to complex multi-step algorithms
- The SDK exposes the Ontology as a backend, enabling external applications to treat it as an API
- As of 2024–2025: Python Functions in Pipeline Builder (beta), AIP agents deployable via OSDK (beta), Foundry Branching for testing Actions without committing to production

**Who builds it:** Engineers and Deployment Strategists.

**Analogy:** If the Semantic Layer is the dictionary, the Kinetic Layer is the *set of allowed sentences* — the verbs, operations, and workflows that can be performed.

---

### 3.3.3 The Dynamic Layer — "The Rules"

**What it answers:** *How does the organization govern, enforce, and adapt?*

The Dynamic Layer introduces behavior, policies, and permissions. It ensures the Ontology is not a static model but an active system that enforces business rules across the platform.

**Key components:**

| Component | Purpose | Example |
|---|---|---|
| **Business Rules** | Conditional logic that fires automatically | "If inventory < threshold, trigger reorder" |
| **Policies** | Organizational constraints | "Only C-level can approve purchases > $500K" |
| **Permissions (RBAC)** | Row-level, column-level, object-level security | Analyst A sees patients in Region 1 only |
| **Workflows** | Multi-step orchestrated processes | Onboarding flow: create account → assign equipment → schedule training |
| **Dynamic Security** | Permissions that change based on object state | "Once a claim is submitted, the submitter can no longer edit it" |

**Key characteristics:**
- Coordinates read and write operations across diverse infrastructure while enforcing unified security
- Enables real-time actions and multi-step simulations
- End users interact with the Ontology through applications and dashboards powered by this layer
- Security policies propagate automatically — if a user can't see a `Patient` object, they can't see it in any application, any report, or any AI agent response

**Who builds it:** Deployment Strategists, security teams, and governance leads.

**Analogy:** The Dynamic Layer is the *law and governance* — the rules that determine who can say what, when, and under what conditions.

---

## 3.4 The Ontology as an "API for Your Organization"

This is Palantir's most powerful framing, and one you should memorize for interviews.

### 3.4.1 One Interface, Many Consumers

Traditional enterprises have dozens of data systems, each with its own API, its own schema, and its own access patterns. A supply chain manager queries SAP. A data scientist queries Snowflake. An AI model reads from a feature store. None of these interfaces share a common vocabulary.

The Ontology collapses this fragmentation:

```
                    ┌──────────────────────┐
                    │      ONTOLOGY        │
                    │  (Unified Semantic    │
                    │       Layer)          │
                    └──────────┬───────────┘
                               │
        ┌──────────┬───────────┼───────────┬───────────┐
        ▼          ▼           ▼           ▼           ▼
   Workshop    Slate App    AIP Agent   OSDK App    API Gateway
   (No-code)   (Low-code)  (LLM-based) (External)  (REST/GraphQL)
```

Every consumer — human or machine — interacts with the **same objects, same links, same actions, same permissions**. A Workshop dashboard showing "Customer X has 12 open orders" draws from the same Ontology that an AIP agent queries when a manager asks "Which customers are at risk of churn?"

### 3.4.2 Why This Matters for AIP

This is where the Ontology becomes a **strategic moat** for AI:

| Challenge with Generic LLMs | How the Ontology Solves It |
|---|---|
| Hallucinate facts | LLM queries are grounded in real Ontology objects with provenance |
| No access to live data | Ontology is backed by continuously updated pipelines |
| Can't take action | LLM proposes Actions defined in the Kinetic Layer; humans approve |
| No access control | Ontology enforces RBAC — the LLM only sees what the user can see |
| No audit trail | Every LLM-proposed action is logged and attributable |

This is why Palantir argues that shipping an LLM without an Ontology is like "shipping a spreadsheet without a database." The LLM needs structured, governed, real-time data to be useful in operations — and the Ontology provides exactly that.

### 3.4.3 The Ontology SDK (OSDK)

For developers building external applications, the OSDK exposes the Ontology programmatically:

| Feature | Detail |
|---|---|
| **Languages** | TypeScript, Python, Java (2024+), OpenAPI spec generation |
| **Operations** | Read objects, filter object sets, execute actions, call functions |
| **Authentication** | OAuth2, integrated with Foundry permissions |
| **Developer Console** | Portal for creating and managing OSDK applications |
| **AIP Agent Deployment** | As of late 2024, AIP agents can be deployed to external apps via OSDK (beta) |

This means the Ontology is not locked inside Foundry. External websites, mobile apps, partner systems, and IoT devices can all read from and write to the Ontology through the SDK.

---

## 3.5 How the Ontology Enables Applications

The Ontology is not just a data model — it is the **backend** for every application built in Palantir.

### 3.5.1 Application Builder Comparison

| Tool | Code Required | Audience | Best For |
|---|---|---|---|
| **Workshop** | No-code / low-code | Business analysts, operators | Standardized operational apps with consistent design |
| **Slate** | Low-code (HTML/CSS/JS) | Developers, technical analysts | Custom dashboards, highly visual interfaces, landing pages |
| **Object Explorer** | No-code | Analysts | Ad-hoc data exploration, browsing objects and links |
| **Quiver** | No-code | Analysts | Charting, time-series analysis, metrics dashboards |

### 3.5.2 The "80/20" Principle

Here's why the Ontology is so powerful for application development:

**Without an Ontology (traditional approach):**
1. Gather requirements → 2. Design database schema → 3. Build API layer → 4. Build frontend → 5. Implement auth → 6. Write business logic → 7. Deploy → 8. Maintain

**With an Ontology:**
1. Define Object Types, Links, Actions in Ontology Manager → 2. Drag-and-drop widgets in Workshop

Steps 2–6 of the traditional approach are **already done** — the Ontology provides the data model, the API, the permissions, and the action logic. Workshop provides the UI. A Deployment Strategist who understands the customer's business can build a working operational application in **hours**, not months.

This is what Palantir means by empowering "citizen developers" — the people closest to the operational problem can build their own tools, without waiting for an engineering team to build a custom application from scratch.

### 3.5.3 Real-World Application Example

**Scenario:** A hospital network wants a patient flow dashboard.

1. **Ontology already contains:** `Patient`, `Bed`, `Ward`, `Physician`, `Encounter` (Object Types) with links like `Patient → admitted_to → Bed → located_in → Ward`
2. **In Workshop:** drag an Object Table widget, bind it to `Patient` objects, filter by `status = "admitted"`, add columns for `ward.name`, `physician.name`, `encounter.admission_date`
3. **Add an Action button:** "Discharge Patient" — calls the `Discharge` Action, which sets `patient.status = "discharged"`, frees the `Bed`, and triggers a notification to housekeeping
4. **Result:** A working patient flow dashboard, built in an afternoon, with full audit trail and RBAC inherited automatically from the Ontology

---

## 3.6 Best Practices for Ontology Design

These guidelines come directly from Palantir's own documentation and practitioner community. Expect to be tested on them.

### 3.6.1 Model Business Decisions, Not Source Systems

❌ **Wrong:** Create one Object Type per source table → `sap_orders_raw`, `salesforce_contacts`, `oracle_inventory`

✅ **Right:** Create Object Types that represent what the business cares about → `Customer`, `Order`, `Inventory_Item`

The Ontology should reflect how the organization thinks and makes decisions, not how its IT systems happen to store data. Multiple source tables may map to a single Object Type (data fusion), and a single source table may feed multiple Object Types (data splitting).

### 3.6.2 Meaningful Naming

| Rule | Good | Bad |
|---|---|---|
| Use natural-language names | `Purchase_Order` | `PO_TBL_v3` |
| No version suffixes | `Message` | `Message_v2` |
| No tag prefixes | Use Groups instead | `FIN_Transaction`, `OPS_Transaction` |
| Consistent property naming | `created_at_timestamp` | `date_created`, `creation_date`, `ts_created` |
| Descriptive link names | `manufactures`, `is_manufactured_by` | `link_1`, `rel_factory` |
| Primary key convention | `{object_type}_id` (string) | Autoincrement integer |
| Foreign key convention | `{foreign_object_type}_id` | `fk_ref_1` |

### 3.6.3 Tight Project Scoping

| Principle | Explanation |
|---|---|
| **One project, one purpose** | Don't mix unrelated Object Types in the same project |
| **Separate concerns** | Consider separate projects for data sources, Ontology definitions, and workflow logic |
| **Meaningful folder names** | Organize resources within projects for clarity |
| **Avoid tangential resources** | Don't add "nice to have" Object Types that aren't needed for the current use case |
| **Cross-project references OK** | Object Types in one project can link to Object Types in another |

### 3.6.4 Design for Iteration

Palantir recommends an iterative development approach:

1. **Draft with placeholder data** — define Object Types and Links before the data pipeline is complete
2. **Split the team** — a Front End Team builds Objects, Actions, and Applications; a Data Engineering Team integrates backing datasets
3. **Gather feedback early** — domain experts should validate Object Types and Link names before the pipeline is built
4. **Only make Object Types editable when necessary** — writeable Object Types require more governance overhead
5. **Configure health checks** — every backing dataset should have scheduled builds and health monitoring

---

## 3.7 Interview Exercise: Design an Ontology

In the Deployment Strategist interview, you may be asked to design an Ontology from scratch for a given domain. Here is a step-by-step framework and a worked example.

### 3.7.1 The Five-Step Framework

```
Step 1: DISCOVER    →  "What decisions do users make?"
Step 2: MODEL       →  "What entities and relationships support those decisions?"
Step 3: INSTRUMENT  →  "What data sources back these entities?"
Step 4: ACTIVATE    →  "What actions do users need to perform?"
Step 5: GOVERN      →  "Who can see and do what?"
```

### 3.7.2 Worked Example: Hospital System

**Scenario:** A large hospital network asks Palantir to help them manage patient flow, bed capacity, and clinical operations across 15 hospitals.

---

**Step 1: DISCOVER — What Decisions?**
- Which patients need beds? Which beds are available?
- Which physicians are on call? Are they over capacity?
- Which surgeries are scheduled? Is the OR available?
- When should a patient be discharged? Who needs to approve?

**Step 2: MODEL — Object Types and Links**

| Object Type | Key Properties | Primary Key |
|---|---|---|
| `Patient` | `name`, `date_of_birth`, `mrn`, `status` (admitted/discharged/ER), `insurance_id` | `patient_id` |
| `Physician` | `name`, `specialty`, `department_id`, `on_call_status`, `license_number` | `physician_id` |
| `Bed` | `ward_id`, `room_number`, `bed_number`, `status` (available/occupied/cleaning), `bed_type` | `bed_id` |
| `Ward` | `hospital_id`, `name`, `floor`, `capacity`, `specialty` | `ward_id` |
| `Hospital` | `name`, `location`, `total_beds`, `icu_beds`, `er_capacity` | `hospital_id` |
| `Encounter` | `patient_id`, `physician_id`, `admission_date`, `discharge_date`, `diagnosis_codes`, `status` | `encounter_id` |
| `Surgery` | `encounter_id`, `operating_room`, `surgeon_id`, `scheduled_time`, `actual_start`, `procedure_code` | `surgery_id` |
| `Medication_Order` | `encounter_id`, `medication_name`, `dosage`, `frequency`, `prescribing_physician_id`, `status` | `med_order_id` |

**Link Map:**

```
Hospital ──1:M──► Ward ──1:M──► Bed
                                 │
Patient ──1:M──► Encounter ──M:1─┘ (Bed ◄──1:1── Encounter)
                    │
                    ├──M:1──► Physician (attending)
                    ├──1:M──► Medication_Order
                    └──1:M──► Surgery
                                │
                          Surgery ──M:1──► Physician (surgeon)
```

**Step 3: INSTRUMENT — Data Sources**

| Object Type | Backing Source | Sync Frequency |
|---|---|---|
| `Patient` | Epic EHR via FHIR API | Near real-time (5 min) |
| `Physician` | HR system (Workday) + credentialing DB | Daily |
| `Bed` | Bed management system (TeleTracking) | Real-time (1 min) |
| `Ward`, `Hospital` | Static reference data (manual CSV + config) | Weekly |
| `Encounter` | Epic ADT feed | Near real-time (5 min) |
| `Surgery` | OR scheduling system (Epic Optime) | Hourly |
| `Medication_Order` | CPOE system (Epic) | Near real-time |

**Step 4: ACTIVATE — Actions**

| Action | Trigger | Side Effects |
|---|---|---|
| **Admit Patient** | ER triage nurse | Creates `Encounter`, links to `Bed`, updates `Bed.status` to occupied, notifies assigned `Physician` |
| **Transfer Patient** | Charge nurse | Updates `Encounter.bed_id`, frees old `Bed`, occupies new `Bed`, logs transfer reason |
| **Discharge Patient** | Attending physician | Sets `Encounter.discharge_date`, frees `Bed`, triggers housekeeping workflow, generates discharge summary |
| **Schedule Surgery** | Surgeon | Creates `Surgery` object, links to `Encounter` and OR, validates OR availability, checks surgeon schedule conflicts |
| **Order Medication** | Physician | Creates `Medication_Order`, validates against allergy list, checks drug interactions, routes to pharmacy |

**Step 5: GOVERN — Permissions**

| Role | Can See | Can Do |
|---|---|---|
| **Nurse (Ward A)** | Patients in Ward A only | Admit, Transfer (within Ward A) |
| **Attending Physician** | Own patients + department | Discharge, Order Medication, Schedule Surgery |
| **Hospital Administrator** | All patients, all wards, all hospitals | View dashboards, run reports (no clinical actions) |
| **System Administrator** | Everything | Manage Object Types, configure pipelines |

---

### 3.7.3 Common Interview Domains

Be prepared to sketch an Ontology for any of these:

| Domain | Key Object Types | Key Actions |
|---|---|---|
| **Supply Chain** | `Warehouse`, `Shipment`, `Supplier`, `Purchase_Order`, `Vehicle`, `Driver` | Request Supply, Dispatch Shipment, Record Delivery |
| **Fleet Management** | `Vehicle`, `Driver`, `Route`, `Maintenance_Record`, `Fuel_Log` | Assign Route, Log Maintenance, Report Incident |
| **Financial Services** | `Customer`, `Account`, `Transaction`, `Alert`, `Investigation` | Flag Transaction, Escalate Alert, Close Investigation |
| **Military Operations** | `Unit`, `Asset`, `Mission`, `Target`, `Intel_Report`, `Airspace` | Plan Mission, Allocate Asset, Deconflict Airspace |
| **Manufacturing** | `Machine`, `Work_Order`, `Quality_Check`, `Material`, `Shift` | Start Work Order, Log Defect, Schedule Maintenance |

---

## 3.8 The Ontology Across Platforms

The Ontology is not exclusive to Foundry. It is the unifying abstraction across the entire Palantir ecosystem.

| Platform | How the Ontology Appears |
|---|---|
| **Foundry** | Objects, Links, Actions defined in Ontology Manager; consumed by Workshop, Slate, Pipeline Builder, Object Explorer |
| **Gotham** | Intelligence entities (persons, organizations, events, locations) modeled as objects with relationship graphs |
| **AIP** | LLM queries grounded in Ontology objects; agents propose Actions; hallucination prevention via ontological provenance |
| **Apollo** | Deploys and updates the software infrastructure that hosts the Ontology across cloud, on-prem, edge, and air-gapped environments |

The Ontology is the **semantic bridge** between platforms. A `Vehicle` object defined in Foundry is the same `Vehicle` that an AIP agent reasons about, the same `Vehicle` that appears in a Gotham operational picture, and the same `Vehicle` whose update pipeline is managed by Apollo.

---

## 3.9 Glossary of Key Terms

| Term | Definition |
|---|---|
| **Object Type** | Schema definition for a real-world entity or event |
| **Object** | A single instance of an Object Type |
| **Object Set** | A filtered collection of Objects |
| **Property** | An attribute of an Object Type (analogous to a column) |
| **Derived Property** | A property computed at runtime from other properties or links (beta) |
| **Shared Property** | A property definition reused across multiple Object Types |
| **Link Type** | A defined relationship between two Object Types |
| **Link** | A single instance of a relationship between two Objects |
| **Action Type** | A governed operation that creates, modifies, or deletes Objects/Links |
| **Action** | A single execution of an Action Type (atomic transaction) |
| **Side Effect** | An additional consequence triggered by an Action (e.g., notification, pipeline trigger) |
| **Function** | Custom business logic written in TypeScript or Python |
| **Backing Dataset** | The raw data source that feeds an Object Type |
| **Primary Key** | A unique identifier for an Object (string preferred) |
| **Semantic Layer** | The conceptual model — what entities exist and how they relate |
| **Kinetic Layer** | The operational layer — how the organization takes action and changes state |
| **Dynamic Layer** | The governance layer — rules, policies, permissions, and workflows |
| **Digital Twin** | A live, operational model of the organization mirroring real-world state |
| **Ontology Manager** | The Foundry interface for creating and managing Object Types, Links, and Actions |
| **OSDK** | Ontology SDK — programmatic access to the Ontology (TypeScript, Python, Java) |
| **Developer Console** | Portal for creating and managing OSDK applications |
| **Writeback** | Pushing Ontology changes back to source systems |
| **Native Federation** | Incorporating external datasets without duplicating data |
| **Foundry Branching** | Testing Actions on branches without committing to production (beta, 2025) |

---

## 3.10 Chapter Summary Card

```
┌─────────────────────────────────────────────────────────┐
│  CHAPTER 3 SUMMARY: THE ONTOLOGY                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  WHAT: A live semantic layer creating a "digital twin"  │
│  of the organization — same interface for humans,       │
│  applications, and AI agents.                           │
│                                                         │
│  FOUR BUILDING BLOCKS:                                  │
│  ┌─────────┐ ┌──────────┐ ┌───────┐ ┌─────────┐       │
│  │ Object  │ │Properties│ │ Links │ │ Actions │       │
│  │ Types   │ │          │ │       │ │         │       │
│  │ (nouns) │ │ (adj.)   │ │ (rel.)│ │ (verbs) │       │
│  └─────────┘ └──────────┘ └───────┘ └─────────┘       │
│                                                         │
│  THREE LAYERS:                                          │
│  Dynamic  → Rules, Policies, Permissions                │
│  Kinetic  → Actions, Functions, Writeback, OSDK         │
│  Semantic → Object Types, Properties, Links             │
│                                                         │
│  KEY INSIGHT: The Ontology is an "API for your org" —   │
│  one semantic framework consumed by Workshop, Slate,    │
│  AIP agents, OSDK apps, and API gateways alike.         │
│                                                         │
│  INTERVIEW FRAMEWORK:                                   │
│  DISCOVER → MODEL → INSTRUMENT → ACTIVATE → GOVERN     │
│                                                         │
│  BEST PRACTICES:                                        │
│  • Model decisions, not source systems                  │
│  • Consistent naming (snake_case, no versions)          │
│  • Tight project scoping                                │
│  • Iterate with domain experts                          │
│  • String primary keys (not autoincrement integers)     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3.11 Five Key Takeaways

1. **The Ontology is Palantir's deepest competitive moat.** It is what turns raw data into operational decisions. Competitors can copy UIs and dashboards; replicating a deeply integrated, governed, real-time semantic layer over an enterprise's entire data estate is extraordinarily difficult.

2. **Four primitives, infinite models.** Object Types (nouns), Properties (adjectives), Links (relationships), and Actions (verbs) — with just these four building blocks, you can model any organization's operations.

3. **Three layers, three audiences.** Semantic (data engineers), Kinetic (developers and Deployment Strategists), Dynamic (security and governance teams). A Deployment Strategist works primarily across Kinetic and Dynamic.

4. **The Ontology enables "citizen development."** Because the Ontology provides the data model, API, permissions, and business logic, non-engineers can build operational applications in Workshop in hours instead of months.

5. **In the interview, you may be asked to design one.** Use the five-step framework: DISCOVER → MODEL → INSTRUMENT → ACTIVATE → GOVERN. Start with the decisions users need to make, work backward to the entities and data sources, then define actions and permissions.
