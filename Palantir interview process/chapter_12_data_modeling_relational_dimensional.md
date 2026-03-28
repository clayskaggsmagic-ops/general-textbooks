# Chapter 12: Data Modeling I — Relational and Dimensional Design

---

## 12.1 Why This Matters for a Deployment Strategist

Data modeling is where the DS role *lives*. If algorithms are the brain of the interview, data modeling is the heart — every Palantir engagement starts with answering one question: **"How does this organization's data actually fit together?"**

Here's why this chapter is essential:

1. **The Ontology IS a data model.** Every Object Type, Property, and Link in Palantir's Ontology has a direct analog to entities, attributes, and relationships in ER modeling. You literally cannot build an Ontology without understanding data modeling
2. **Clients' source systems are relational.** The Oracle, PostgreSQL, SAP, and Snowflake systems you integrate into Foundry all use relational schemas. You'll read ER diagrams, understand normalization choices, and explain why the client's schema looks the way it does
3. **The interview tests this explicitly.** The DS interview includes data modeling problems: "Design a schema for X," "What are the trade-offs of this normalization level?" You need to speak fluently about normal forms, dimensional models, and design patterns
4. **Star schemas appear everywhere in analytics.** When a client asks "Build me a dashboard," the underlying data model is almost always a star or snowflake schema. Understanding fact and dimension tables is prerequisite knowledge for any analytics deployment

> **Key insight:** Data modeling is the bridge between "what the business does" and "what the database stores." A great DS understands both sides of that bridge — and can walk clients across it.

---

## 12.2 Conceptual Modeling — Entity-Relationship Diagrams

### 12.2.1 What Is Conceptual Modeling?

Conceptual modeling is the **first step** in database design. It captures *what* data exists and *how* it relates — without worrying about tables, columns, or SQL. The primary tool is the **Entity-Relationship (ER) Diagram**.

```
  PURPOSE OF CONCEPTUAL MODELS:

  • Communicate with stakeholders (non-technical people can read them)
  • Establish shared vocabulary (what IS a "customer"? What IS an "order"?)
  • Serve as the blueprint for logical and physical design
  • Document business rules in visual form

  Audience: Business analysts, product owners, executives, AND engineers
  NOT about: Data types, indexes, storage engines, SQL syntax
```

### 12.2.2 The Building Blocks

**Entities** — Nouns. Things in the real world that the business cares about.

```
  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
  │  CUSTOMER   │   │   ORDER     │   │  PRODUCT    │
  └─────────────┘   └─────────────┘   └─────────────┘
       noun              noun              noun

  Strong entity: Exists independently (Customer, Product)
  Weak entity: Cannot exist without a parent entity
               (OrderLine only exists if an Order exists)
```

**Attributes** — Adjectives. Properties that describe an entity.

```
  ┌──────────────────────────┐
  │        CUSTOMER          │
  ├──────────────────────────┤
  │  customer_id (PK)        │   ← Primary key: uniquely identifies
  │  first_name              │   ← Simple attribute
  │  last_name               │
  │  email                   │   ← Candidate key (unique)
  │  address                 │   ← Composite: street + city + state + zip
  │  phone_numbers           │   ← Multi-valued: customer may have several
  │  age                     │   ← Derived: calculated from birth_date
  └──────────────────────────┘
```

**Relationships** — Verbs. How entities are connected.

```
  CUSTOMER ───< places >─── ORDER
  "A customer places an order"

  Verbs: places, contains, manages, belongs_to, enrolls_in
```

### 12.2.3 Cardinality — "How Many?"

Cardinality defines how many instances of one entity relate to another:

```
  ONE-TO-ONE (1:1)
  ┌──────────┐           ┌──────────┐
  │ EMPLOYEE │──── 1:1 ──│ PASSPORT │
  └──────────┘           └──────────┘
  Each employee has exactly one passport.
  Each passport belongs to exactly one employee.

  Real-world examples:
  • Person ↔ SSN
  • Country ↔ Capital City
  • User ↔ User Profile (in some designs)

  ─────────────────────────────────────────────

  ONE-TO-MANY (1:M)
  ┌──────────┐           ┌──────────┐
  │ CUSTOMER │──── 1:M ──│  ORDER   │
  └──────────┘           └──────────┘
  One customer can place MANY orders.
  Each order belongs to exactly ONE customer.

  This is the MOST COMMON relationship in relational databases.

  Real-world examples:
  • Department → Employees
  • Author → Books
  • Parent → Children

  ─────────────────────────────────────────────

  MANY-TO-MANY (M:N)
  ┌──────────┐           ┌──────────┐
  │ STUDENT  │──── M:N ──│  COURSE  │
  └──────────┘           └──────────┘
  A student enrolls in MANY courses.
  A course has MANY students.

  ⚠️ Cannot be directly represented in a relational database!
  → Must be decomposed using a JUNCTION TABLE (see §12.3.2)

  Real-world examples:
  • Doctors ↔ Patients
  • Actors ↔ Movies
  • Products ↔ Suppliers
```

### 12.2.4 Participation — "Must They?"

Participation defines whether every instance of an entity MUST participate in a relationship:

```
  TOTAL PARTICIPATION (mandatory)
  ═══════╡ EMPLOYEE ╞═══════ works_in ─── DEPARTMENT
  Double line: EVERY employee MUST work in a department.
  No employee can exist without a department assignment.

  PARTIAL PARTICIPATION (optional)
  ───────│ EMPLOYEE │─────── manages ─── DEPARTMENT
  Single line: An employee MAY manage a department.
  Not every employee manages one. Some do, some don't.

  ─────────────────────────────────────────────

  Combined example:

  EMPLOYEE ═══ works_in ─── DEPARTMENT
    Total                     Partial
  (every employee             (a department may exist
   must belong to              without employees, e.g.,
   a department)               a newly created dept.)
```

### 12.2.5 Supertype/Subtype — "Is-A" Relationships

Generalization and specialization allow hierarchical classification:

```
              ┌────────────┐
              │  EMPLOYEE  │          ← SUPERTYPE
              │  emp_id    │            (shared attributes)
              │  name      │
              │  hire_date │
              └─────┬──────┘
                    │
           ┌────────┼────────┐
           ↓        ↓        ↓
    ┌───────────┐ ┌──────┐ ┌──────────┐
    │ FULL_TIME │ │ PART │ │ CONTRACT │  ← SUBTYPES
    │ salary    │ │ TIME │ │ hourly   │    (specialized
    │ benefits  │ │ hrs  │ │ _rate    │     attributes)
    └───────────┘ └──────┘ └──────────┘

  • Subtypes INHERIT all attributes from the supertype
  • Subtypes add their OWN unique attributes
  • Total specialization: every Employee MUST be one subtype
  • Partial specialization: an Employee MAY not be any subtype
  • Disjoint: each Employee can be ONLY ONE subtype
  • Overlapping: an Employee can be MULTIPLE subtypes
```

**Palantir Connection:** The Ontology supports subtypes directly. An Object Type "Vehicle" might have subtypes "Truck," "Sedan," "Motorcycle" — each inheriting base properties but adding their own.

---

## 12.3 Logical Modeling — ER to Relational Schema

### 12.3.1 Translation Rules

Converting a conceptual ER diagram into a relational schema follows systematic rules:

```
  ┌────────────────────────────────────────────────────────────────┐
  │  ER CONCEPT              →  RELATIONAL SCHEMA                  │
  ├────────────────────────────────────────────────────────────────┤
  │  Strong entity           →  Table with PK                      │
  │  Weak entity             →  Table with composite PK            │
  │                             (own partial key + owner's PK)     │
  │  Simple attribute        →  Column                             │
  │  Composite attribute     →  Multiple columns (flatten)         │
  │  Multi-valued attribute  →  Separate table with FK back        │
  │  Derived attribute       →  Computed column or don't store     │
  │  1:1 relationship        →  FK in either table (prefer the     │
  │                             side with total participation)     │
  │  1:M relationship        →  FK in the "many" side table        │
  │  M:N relationship        →  New junction table                 │
  │  Supertype/subtype       →  Multiple strategies (see below)    │
  └────────────────────────────────────────────────────────────────┘
```

### 12.3.2 Junction Tables for M:N Relationships

Many-to-many relationships **cannot** be directly stored in relational tables. They require a **junction table** (also called a bridge table, associative table, or link table):

```
  ER: STUDENT ──── M:N ──── COURSE

  Relational Schema:

  ┌───────────────┐     ┌──────────────────┐     ┌──────────────┐
  │   STUDENTS    │     │   ENROLLMENTS    │     │   COURSES    │
  ├───────────────┤     ├──────────────────┤     ├──────────────┤
  │ student_id PK │←─┐  │ student_id FK,PK │  ┌─→│ course_id PK │
  │ name          │  └──│ course_id  FK,PK │──┘  │ title        │
  │ email         │     │ enrollment_date  │     │ credits      │
  └───────────────┘     │ grade            │     └──────────────┘
                        └──────────────────┘
                          ↑ Junction table
                          • Composite PK: (student_id, course_id)
                          • Can have its own attributes (grade, date)
                          • Transforms M:N into two 1:M relationships
```

**Why junction tables matter:** Every many-to-many Link in the Palantir Ontology is backed by a junction table in the underlying dataset. Understanding junction tables = understanding how Ontology Links work at the data layer.

### 12.3.3 Composite Keys

A composite key is a primary key consisting of **two or more columns** together:

```
  ENROLLMENT table:
  ┌─────────────┬────────────┬──────────────────┬───────┐
  │ student_id  │ course_id  │ enrollment_date  │ grade │
  ├─────────────┼────────────┼──────────────────┼───────┤
  │ S001        │ CS101      │ 2024-01-15       │ A     │
  │ S001        │ MATH201    │ 2024-01-15       │ B+    │
  │ S002        │ CS101      │ 2024-01-16       │ A-    │
  └─────────────┴────────────┴──────────────────┴───────┘

  Neither student_id nor course_id alone is unique.
  But (student_id, course_id) together IS unique.
  → Composite primary key = (student_id, course_id)
```

| Key Type | Definition | Example |
|---|---|---|
| **Primary Key** | Uniquely identifies each row | `customer_id` |
| **Composite Key** | PK made of multiple columns | `(student_id, course_id)` |
| **Foreign Key** | References another table's PK | `order.customer_id → customer.customer_id` |
| **Candidate Key** | Any column(s) that could serve as PK | `email` (unique, not null) |
| **Natural Key** | Real-world identifier | `ssn`, `isbn` |
| **Surrogate Key** | System-generated identifier | `id` (auto-increment), UUID |

### 12.3.4 Supertype/Subtype Translation Strategies

```
  Strategy 1: TABLE-PER-TYPE (one table per supertype + one per subtype)
  ──────────────────────────────────────────────────────────────
  employees(emp_id PK, name, hire_date)
  full_time(emp_id PK/FK, salary, benefits)
  part_time(emp_id PK/FK, hourly_rate, max_hours)

  ✅ Clean separation, no NULLs
  ❌ Requires JOINs to get complete picture

  ──────────────────────────────────────────────────────────────
  Strategy 2: SINGLE TABLE (everything in one table)
  ──────────────────────────────────────────────────────────────
  employees(emp_id PK, name, hire_date, type,
            salary, benefits, hourly_rate, max_hours)

  ✅ No JOINs needed, simple queries
  ❌ Many NULLs (part-timers have NULL salary/benefits)

  ──────────────────────────────────────────────────────────────
  Strategy 3: TABLE-PER-SUBTYPE (no supertype table)
  ──────────────────────────────────────────────────────────────
  full_time(emp_id PK, name, hire_date, salary, benefits)
  part_time(emp_id PK, name, hire_date, hourly_rate, max_hours)

  ✅ No NULLs, no JOINs within a type
  ❌ Cannot query "all employees" without UNION
  ❌ Shared attributes duplicated across tables
```

---

## 12.4 Normalization — Eliminating Redundancy

### 12.4.1 Why Normalize?

Normalization eliminates **data redundancy** and prevents **anomalies** — situations where inserting, updating, or deleting data leads to inconsistencies:

```
  ❌ UNNORMALIZED TABLE (redundancy everywhere):
  ┌──────────┬──────────┬─────────┬────────────┬──────────────┐
  │ order_id │ cust_id  │ cust_   │ product    │ product_     │
  │          │          │ name    │            │ price        │
  ├──────────┼──────────┼─────────┼────────────┼──────────────┤
  │ 1001     │ C01      │ Alice   │ Widget     │ 9.99         │
  │ 1002     │ C01      │ Alice   │ Gadget     │ 19.99        │
  │ 1003     │ C02      │ Bob     │ Widget     │ 9.99         │
  │ 1004     │ C01      │ Alice   │ Widget     │ 9.99         │
  └──────────┴──────────┴─────────┴────────────┴──────────────┘

  PROBLEMS:
  • "Alice" stored 3 times → UPDATE ANOMALY: change her name in one
    row but forget the others → inconsistency
  • Can't add customer C03 without an order → INSERT ANOMALY
  • Delete order 1003 → lose the fact that Bob exists → DELETE ANOMALY
```

### 12.4.2 The Normal Forms

**First Normal Form (1NF)** — Atomic values, no repeating groups

```
  ❌ VIOLATES 1NF:
  ┌──────────┬──────────────────────────┐
  │ cust_id  │ phone_numbers            │
  ├──────────┼──────────────────────────┤
  │ C01      │ 555-1234, 555-5678       │  ← Multiple values in one cell!
  │ C02      │ 555-9012                 │
  └──────────┴──────────────────────────┘

  ✅ IN 1NF:
  ┌──────────┬──────────────┐    ┌──────────┬──────────────┐
  │ cust_id  │ name         │    │ cust_id  │ phone_number │
  ├──────────┼──────────────┤    ├──────────┼──────────────┤
  │ C01      │ Alice        │    │ C01      │ 555-1234     │
  │ C02      │ Bob          │    │ C01      │ 555-5678     │
  └──────────┴──────────────┘    │ C02      │ 555-9012     │
                                 └──────────┴──────────────┘

  Rule: Every cell contains exactly ONE value. No lists, no arrays,
  no comma-separated values. Each row is unique (has a PK).
```

**Second Normal Form (2NF)** — No partial dependencies

```
  ❌ VIOLATES 2NF (partial dependency):

  Table: ENROLLMENT (composite PK: student_id + course_id)
  ┌────────────┬───────────┬───────────────┬───────┐
  │ student_id │ course_id │ course_name   │ grade │
  ├────────────┼───────────┼───────────────┼───────┤
  │ S001       │ CS101     │ Intro to CS   │ A     │
  │ S002       │ CS101     │ Intro to CS   │ B+    │ ← "Intro to CS"
  │ S001       │ MATH201   │ Calculus II   │ A-    │    repeated!
  └────────────┴───────────┴───────────────┴───────┘

  course_name depends ONLY on course_id, not on the full key
  (student_id, course_id). This is a PARTIAL DEPENDENCY.

  ✅ IN 2NF (split into two tables):

  ENROLLMENTS                    COURSES
  ┌────────────┬───────────┬─────┐  ┌───────────┬───────────────┐
  │ student_id │ course_id │grade│  │ course_id │ course_name   │
  ├────────────┼───────────┼─────┤  ├───────────┼───────────────┤
  │ S001       │ CS101     │ A   │  │ CS101     │ Intro to CS   │
  │ S002       │ CS101     │ B+  │  │ MATH201   │ Calculus II   │
  │ S001       │ MATH201   │ A-  │  └───────────┴───────────────┘
  └────────────┴───────────┴─────┘

  Rule: Every non-key attribute depends on the WHOLE primary key,
  not just part of it. (Only relevant for COMPOSITE primary keys.)
```

**Third Normal Form (3NF)** — No transitive dependencies

```
  ❌ VIOLATES 3NF (transitive dependency):

  EMPLOYEES
  ┌────────┬──────────┬─────────────┬──────────────────┐
  │ emp_id │ name     │ dept_id     │ dept_name        │
  ├────────┼──────────┼─────────────┼──────────────────┤
  │ E01    │ Alice    │ D10         │ Engineering      │
  │ E02    │ Bob      │ D10         │ Engineering      │ ← repeated!
  │ E03    │ Carol    │ D20         │ Marketing        │
  └────────┴──────────┴─────────────┴──────────────────┘

  Dependency chain: emp_id → dept_id → dept_name
  dept_name depends on dept_id (non-key), NOT directly on emp_id.
  This is a TRANSITIVE DEPENDENCY.

  ✅ IN 3NF:

  EMPLOYEES                      DEPARTMENTS
  ┌────────┬──────────┬─────────┐  ┌─────────┬──────────────────┐
  │ emp_id │ name     │ dept_id │  │ dept_id │ dept_name        │
  ├────────┼──────────┼─────────┤  ├─────────┼──────────────────┤
  │ E01    │ Alice    │ D10     │  │ D10     │ Engineering      │
  │ E02    │ Bob      │ D10     │  │ D20     │ Marketing        │
  │ E03    │ Carol    │ D20     │  └─────────┴──────────────────┘
  └────────┴──────────┴─────────┘

  Rule: Non-key attributes depend ONLY on the primary key,
  not on other non-key attributes.
```

**Boyce-Codd Normal Form (BCNF)** — Stricter than 3NF

```
  BCNF adds one more constraint:
  "For every functional dependency X → Y, X must be a superkey."

  3NF allows: Non-prime attribute → another non-prime (if A→B, A doesn't
  need to be a superkey, as long as B is part of a candidate key)

  BCNF says: NO. If X → Y exists, X MUST be a superkey. Period.

  BCNF is sometimes called "3.5NF" — it's a stricter version of 3NF
  that handles edge cases with multiple overlapping candidate keys.

  In practice: Most 3NF schemas are already in BCNF.
  The distinction matters mainly in exam/interview settings.
```

### 12.4.3 The Normalization Ladder

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                                                                   │
  │  UNNORMALIZED DATA                                                │
  │       ↓  Eliminate repeating groups and multi-valued cells         │
  │  1NF — Atomic values only                                         │
  │       ↓  Eliminate partial dependencies                           │
  │  2NF — Full functional dependency on entire key                   │
  │       ↓  Eliminate transitive dependencies                        │
  │  3NF — No non-key → non-key dependencies                         │
  │       ↓  Every determinant is a superkey                          │
  │  BCNF — Strictest practical form                                  │
  │                                                                   │
  │  RULE OF THUMB: Design to 3NF for OLTP systems.                  │
  │  Most production databases target 3NF.                            │
  │  BCNF is ideal but rarely a practical concern.                    │
  │                                                                   │
  └───────────────────────────────────────────────────────────────────┘
```

### 12.4.4 The Anomalies — Why We Normalize

| Anomaly | What Happens | Cause | Normal Form Fix |
|---|---|---|---|
| **Insert** | Can't add data without unrelated data (e.g., can't add a course without a student) | Entities mixed in one table | Split into separate tables (1NF/2NF) |
| **Update** | Changing data in one row but not matching rows → inconsistency | Same fact stored multiple times | Eliminate redundancy (2NF/3NF) |
| **Delete** | Removing a row loses unrelated data (e.g., deleting last student in a course loses the course) | Entities mixed in one table | Split into separate tables (2NF/3NF) |

### 12.4.5 When to Denormalize

Normalization optimizes for **writes** (consistency, integrity). But many real-world systems are **read-heavy**:

```
  ┌──────────────────────────────────────────────────────────────┐
  │  NORMALIZE (3NF) WHEN:          DENORMALIZE WHEN:            │
  │                                                              │
  │  • Write-heavy OLTP system      • Read-heavy OLAP/analytics │
  │  • Data consistency is critical  • Query speed is critical    │
  │  • Many concurrent writers       • Data loads in batches      │
  │  • Storage is limited            • JOINs are the bottleneck   │
  │  • Schema changes are frequent   • Dashboards/reports         │
  │  • Transaction integrity needed  • Pre-computed aggregates    │
  └──────────────────────────────────────────────────────────────┘
```

**Common denormalization techniques:**

| Technique | What It Does | Trade-off |
|---|---|---|
| **Pre-joined tables** | Store data from multiple tables in one | Eliminates JOINs; risks update anomalies |
| **Computed/cached columns** | Store calculated values (e.g., `total_order_value`) | Avoids re-computation; must keep in sync |
| **Materialized views** | Database stores query results as a table | Fast reads; stale data until refreshed |
| **Summary tables** | Pre-aggregate data (daily sales, monthly totals) | Instant reporting; extra storage + sync |
| **Star schema** | Denormalize dimensions around fact tables | Analytics-optimized; write complexity |

---

## 12.5 Dimensional Modeling — The Kimball Methodology

### 12.5.1 The Philosophy

Ralph Kimball's dimensional modeling methodology is the dominant approach for **analytical data warehouses**. Its core principle: **organize data around business processes and make it easy to query.**

```
  OLTP (Normalized)                OLAP (Dimensional)
  ─────────────────                ──────────────────
  Optimized for WRITES             Optimized for READS
  Minimize redundancy              Embrace strategic redundancy
  Many narrow tables               Few wide tables
  Complex JOINs for reports        Simple, predictable JOINs
  Entity-centric                   Event/process-centric
  Used by applications             Used by analysts & dashboards
```

### 12.5.2 Star Schema

The star schema is the foundation of dimensional modeling:

```
                    ┌──────────────┐
                    │  DIM_DATE    │
                    │  date_key PK │
                    │  date        │
                    │  month       │
                    │  quarter     │
                    │  year        │
                    │  day_of_week │
                    └──────┬───────┘
                           │
  ┌──────────────┐  ┌──────┴───────────────┐  ┌──────────────┐
  │ DIM_CUSTOMER │  │    FACT_SALES        │  │ DIM_PRODUCT  │
  │ cust_key  PK │──│ date_key     FK      │──│ prod_key  PK │
  │ name         │  │ cust_key     FK      │  │ product_name │
  │ email        │  │ prod_key     FK      │  │ category     │
  │ city         │  │ store_key    FK      │  │ brand        │
  │ state        │  │ quantity  (measure)  │  │ unit_price   │
  │ segment      │  │ unit_price (measure) │  └──────────────┘
  └──────────────┘  │ total_amt  (measure) │
                    │ discount   (measure) │  ┌──────────────┐
                    └──────┬───────────────┘  │ DIM_STORE    │
                           │                  │ store_key PK │
                           └──────────────────│ store_name   │
                                              │ city         │
                                              │ region       │
                                              └──────────────┘

  FACT TABLE (center):
  • Contains MEASURES (numbers you aggregate: sum, avg, count)
  • Contains FOREIGN KEYS to dimension tables
  • One row per EVENT/TRANSACTION
  • Typically very tall (millions/billions of rows), relatively narrow

  DIMENSION TABLES (surrounding):
  • Contain DESCRIPTIVE ATTRIBUTES (who, what, where, when)
  • Typically wide (many columns), relatively short (thousands of rows)
  • Usually DENORMALIZED (city, state, region all in one table)
```

**Why star schema works for analytics:**

```
  Query: "Total sales by product category and quarter for the West region"

  SELECT
    p.category,
    d.quarter,
    SUM(f.total_amt) AS total_sales
  FROM fact_sales f
  JOIN dim_product p ON f.prod_key = p.prod_key
  JOIN dim_date d    ON f.date_key = d.date_key
  JOIN dim_store s   ON f.store_key = s.store_key
  WHERE s.region = 'West'
  GROUP BY p.category, d.quarter;

  • Simple, predictable JOIN pattern (always fact → dimension)
  • Every query follows the same structure
  • Analysts can understand and write these queries intuitively
  • Query optimizers handle star joins very efficiently
```

### 12.5.3 Snowflake Schema

A snowflake schema **normalizes** the dimension tables:

```
  Star Schema (denormalized):          Snowflake Schema (normalized):

  ┌────────────┐                       ┌────────────┐
  │ DIM_PRODUCT│                       │ DIM_PRODUCT│
  │ prod_key   │                       │ prod_key   │
  │ name       │                       │ name       │
  │ category   │  ← redundant          │ cat_key FK │──→ ┌──────────┐
  │ dept       │  ← redundant          └────────────┘    │ CATEGORY │
  │ brand      │                                         │ cat_key  │
  └────────────┘                                         │ cat_name │
                                                         │ dept_key │──→ DEPARTMENT
                                                         └──────────┘
```

| | Star Schema | Snowflake Schema |
|---|---|---|
| **Dimension tables** | Denormalized (flat, wide) | Normalized (hierarchical, narrow) |
| **JOINs** | Fewer (fact → dimension only) | More (fact → dim → sub-dim → ...) |
| **Query speed** | ✅ Faster (fewer JOINs) | ❌ Slower (more JOINs) |
| **Storage** | ❌ More redundancy | ✅ Less redundancy |
| **Simplicity** | ✅ Easier to understand | ❌ More complex |
| **Best for** | Most analytics use cases | Storage-constrained or complex hierarchies |

**In practice:** Star schemas are used in ~90% of analytical warehouses. Snowflake schemas are used when dimension tables are extremely large and storage/update cost matters.

### 12.5.4 Fact Table Types

```
  ┌───────────────────────────────────────────────────────────────────┐
  │  TRANSACTIONAL FACT TABLE                                         │
  │  ───────────────────────                                          │
  │  One row per EVENT as it happens.                                 │
  │  Most common type. Grain = individual transaction.                │
  │                                                                   │
  │  Example: Every sale, every click, every shipment                 │
  │  ┌──────────┬──────────┬──────────┬─────────┬──────────┐         │
  │  │ date_key │ cust_key │ prod_key │ qty     │ amount   │         │
  │  ├──────────┼──────────┼──────────┼─────────┼──────────┤         │
  │  │ 20240115 │ C001     │ P005     │ 2       │ 19.98    │         │
  │  │ 20240115 │ C003     │ P012     │ 1       │ 49.99    │         │
  │  │ 20240116 │ C001     │ P005     │ 3       │ 29.97    │         │
  │  └──────────┴──────────┴──────────┴─────────┴──────────┘         │
  │  Rows: INSERT only (append). Never updated.                       │
  │  Volume: Very high (could be billions of rows)                    │
  ├───────────────────────────────────────────────────────────────────┤
  │  PERIODIC SNAPSHOT FACT TABLE                                     │
  │  ───────────────────────────                                      │
  │  One row per ENTITY per TIME PERIOD.                              │
  │  Captures state at regular intervals.                             │
  │                                                                   │
  │  Example: Daily account balance, monthly inventory level          │
  │  ┌──────────┬──────────┬──────────┬──────────┐                   │
  │  │ date_key │ acct_key │ balance  │ num_txns │                   │
  │  ├──────────┼──────────┼──────────┼──────────┤                   │
  │  │ 20240115 │ A001     │ 5432.10  │ 3        │                   │
  │  │ 20240115 │ A002     │ 12876.50 │ 0        │                   │
  │  │ 20240116 │ A001     │ 5200.00  │ 1        │                   │
  │  └──────────┴──────────┴──────────┴──────────┘                   │
  │  Rows: INSERT new snapshots. Previous rows never modified.        │
  │  Use when: Need to track state over time                          │
  ├───────────────────────────────────────────────────────────────────┤
  │  ACCUMULATING SNAPSHOT FACT TABLE                                 │
  │  ─────────────────────────────                                    │
  │  One row per PROCESS INSTANCE. Updated as milestones are reached. │
  │  Tracks the lifecycle of a process from start to end.             │
  │                                                                   │
  │  Example: Order fulfillment (ordered → shipped → delivered)       │
  │  ┌──────────┬────────────┬──────────────┬──────────────┐         │
  │  │ order_id │ order_date │ ship_date    │ deliver_date │         │
  │  ├──────────┼────────────┼──────────────┼──────────────┤         │
  │  │ O001     │ 2024-01-15 │ 2024-01-17   │ 2024-01-20   │         │
  │  │ O002     │ 2024-01-16 │ 2024-01-18   │ NULL         │         │
  │  │ O003     │ 2024-01-17 │ NULL         │ NULL         │         │
  │  └──────────┴────────────┴──────────────┴──────────────┘         │
  │  Rows: UPDATED as milestones complete. NULL = not yet reached.    │
  │  Use when: Tracking multi-step processes with defined stages      │
  └───────────────────────────────────────────────────────────────────┘
```

### 12.5.5 Slowly Changing Dimensions (SCD)

Dimension data changes over time. How you handle those changes determines what historical analysis is possible:

```
  SCENARIO: Customer "Alice" moves from New York to Chicago on March 1.

  ─────────────────────────────────────────────────────────────────

  SCD TYPE 1 — OVERWRITE (no history)
  Simply update the row in place.

  BEFORE:                          AFTER:
  │ cust_key │ name  │ city     │  │ cust_key │ name  │ city     │
  │ C001     │ Alice │ New York │  │ C001     │ Alice │ Chicago  │

  ✅ Simple, minimal storage
  ❌ History is LOST — can't answer "Where did Alice live in January?"
  Use when: Corrections, data cleansing, or history doesn't matter

  ─────────────────────────────────────────────────────────────────

  SCD TYPE 2 — ADD NEW ROW (full history)
  Create a new row with version tracking.

  │ cust_key │ name  │ city     │ eff_start  │ eff_end    │ current │
  │ C001_v1  │ Alice │ New York │ 2020-01-01 │ 2024-02-28 │ N       │
  │ C001_v2  │ Alice │ Chicago  │ 2024-03-01 │ 9999-12-31 │ Y       │

  ✅ Full history preserved — can answer ANY historical question
  ✅ Sales before March 1 still link to "New York"
  ❌ Table grows over time (one row per change per entity)
  ❌ Need surrogate keys (C001_v1 ≠ C001_v2 in the PK)
  Use when: Historical accuracy is critical (most common in practice)

  ─────────────────────────────────────────────────────────────────

  SCD TYPE 3 — ADD COLUMN (limited history)
  Add columns for previous values.

  │ cust_key │ name  │ current_city │ previous_city │
  │ C001     │ Alice │ Chicago      │ New York      │

  ✅ Simple before/after comparison
  ❌ Only tracks ONE prior value — can't see two moves ago
  ❌ Schema changes required for each tracked attribute
  Use when: Only need "before and after" analysis
```

**SCD Decision Table:**

| Type | History | Storage | Complexity | Best For |
|---|---|---|---|---|
| **Type 1** | ❌ None | Minimal | Simplest | Error corrections, unimportant attributes |
| **Type 2** | ✅ Full | Grows over time | Moderate | Most analytical dimensions (default choice) |
| **Type 3** | ⚠️ Limited (1 prior) | Minimal | Moderate | "Before and after" analysis only |

### 12.5.6 Conformed Dimensions

Conformed dimensions are **shared across multiple fact tables**, ensuring consistent definitions across the organization:

```
  ┌──────────────────────┐         ┌──────────────────────┐
  │   FACT_SALES         │         │   FACT_INVENTORY     │
  │   date_key FK ───────┼────┐    │   date_key FK ───────┼──┐
  │   prod_key FK ───────┼──┐ │    │   prod_key FK ───────┼┐ │
  │   cust_key FK        │  │ │    │   warehouse_key FK   ││ │
  │   quantity           │  │ │    │   qty_on_hand        ││ │
  │   revenue            │  │ │    │   qty_on_order       ││ │
  └──────────────────────┘  │ │    └──────────────────────┘│ │
                            │ │                            │ │
                            │ └──── DIM_DATE ──────────────┘ │
                            │      (shared!)                 │
                            └──── DIM_PRODUCT ───────────────┘
                                   (shared!)

  Benefits:
  • "Product X" means the SAME thing across sales and inventory
  • Can "drill across" — compare sales vs. inventory for same product
  • Enables cross-functional analysis (marketing + sales + finance)
  • Prevents "same metric, different number" problem

  This is exactly what Palantir's Ontology does:
  One "Customer" Object Type used across all applications.
```

---

## 12.6 Schema Design Patterns

### 12.6.1 Audit Trails

Track **who** changed **what** and **when** for compliance and debugging:

```
  Pattern 1: METADATA COLUMNS (lightweight)
  ──────────────────────────────────────────
  Every table gets these columns:

  CREATE TABLE orders (
    order_id     SERIAL PRIMARY KEY,
    customer_id  INT NOT NULL,
    total        DECIMAL(10,2),
    status       VARCHAR(20),
    -- Audit columns:
    created_at   TIMESTAMP DEFAULT NOW(),
    updated_at   TIMESTAMP DEFAULT NOW(),
    created_by   VARCHAR(100),
    updated_by   VARCHAR(100)
  );

  ✅ Simple, low overhead
  ❌ Only captures the LATEST change, not full history

  ──────────────────────────────────────────
  Pattern 2: AUDIT LOG TABLE (heavyweight)
  ──────────────────────────────────────────
  Separate table tracking every change:

  CREATE TABLE audit_log (
    audit_id     SERIAL PRIMARY KEY,
    table_name   VARCHAR(100),
    record_id    VARCHAR(100),
    action       VARCHAR(10),     -- INSERT, UPDATE, DELETE
    old_values   JSONB,           -- previous state
    new_values   JSONB,           -- new state
    changed_by   VARCHAR(100),
    changed_at   TIMESTAMP DEFAULT NOW()
  );

  ✅ Complete history of every change
  ✅ Required for HIPAA, SOX, GDPR compliance
  ❌ Significant storage overhead
  ❌ Must be implemented via triggers or application logic
```

### 12.6.2 Soft Deletes

Mark records as deleted instead of physically removing them:

```
  APPROACH 1: Boolean flag
  ──────────────────────────────────────────
  ALTER TABLE customers ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;

  -- "Delete" a customer:
  UPDATE customers SET is_deleted = TRUE WHERE customer_id = 123;

  -- Query active customers:
  SELECT * FROM customers WHERE is_deleted = FALSE;

  ──────────────────────────────────────────
  APPROACH 2: Timestamp (preferred)
  ──────────────────────────────────────────
  ALTER TABLE customers ADD COLUMN deleted_at TIMESTAMP NULL;

  -- "Delete" a customer:
  UPDATE customers SET deleted_at = NOW() WHERE customer_id = 123;

  -- Query active customers:
  SELECT * FROM customers WHERE deleted_at IS NULL;

  -- Know WHEN something was deleted:
  SELECT * FROM customers WHERE deleted_at IS NOT NULL
  ORDER BY deleted_at DESC;  -- most recently "deleted" first
```

| | Hard Delete | Soft Delete |
|---|---|---|
| **Data recovery** | ❌ Gone forever | ✅ Can restore |
| **Referential integrity** | Can cascade break FKs | FK references still valid |
| **Query complexity** | Simple | ⚠️ Must add `WHERE deleted_at IS NULL` everywhere |
| **Storage** | ✅ Freed | ❌ Accumulates |
| **GDPR compliance** | ✅ Actually deleted | ⚠️ May not satisfy "right to be forgotten" |
| **Audit trail** | ❌ No record of deletion | ✅ Know what was deleted and when |

### 12.6.3 Temporal Data

Temporal design tracks data across two independent time axes:

```
  VALID TIME (business time):
  "When was this fact TRUE in the real world?"
  Example: Employee salary was $80K from Jan–June, then $90K from July onward.

  TRANSACTION TIME (system time):
  "When was this fact RECORDED in the database?"
  Example: The salary change was entered into the system on July 3rd.

  ──────────────────────────────────────────

  UNI-TEMPORAL: tracks one time axis (usually valid time)

  ┌─────────┬────────┬──────────────┬──────────────┐
  │ emp_id  │ salary │ valid_from   │ valid_to     │
  ├─────────┼────────┼──────────────┼──────────────┤
  │ E01     │ 80000  │ 2024-01-01   │ 2024-06-30   │
  │ E01     │ 90000  │ 2024-07-01   │ 9999-12-31   │
  └─────────┴────────┴──────────────┴──────────────┘

  ──────────────────────────────────────────

  BITEMPORAL: tracks BOTH axes independently

  ┌─────────┬────────┬────────────┬──────────┬────────────┬──────────┐
  │ emp_id  │ salary │ valid_from │ valid_to │ txn_from   │ txn_to   │
  ├─────────┼────────┼────────────┼──────────┼────────────┼──────────┤
  │ E01     │ 80000  │ 2024-01-01 │ 9999-12-31│ 2024-01-01│ 2024-07-03│
  │ E01     │ 80000  │ 2024-01-01 │ 2024-06-30│ 2024-07-03│ 9999-12-31│
  │ E01     │ 90000  │ 2024-07-01 │ 9999-12-31│ 2024-07-03│ 9999-12-31│
  └─────────┴────────┴────────────┴──────────┴────────────┴──────────┘

  Now can answer:
  • "What was E01's salary in March 2024?" → $80K (valid time query)
  • "What did the database THINK E01's salary was on Jan 15?" → $80K (txn time)
  • "When was the raise RECORDED in the system?" → July 3 (txn time)

  Use cases: Financial systems, insurance, regulatory compliance,
  any system where "what did we know and when did we know it" matters
```

### 12.6.4 Multi-Tenant Design

Multiple organizations ("tenants") sharing a single database:

```
  Strategy 1: SHARED SCHEMA (tenant_id column)
  ──────────────────────────────────────────
  ┌──────────────────────────────────────────────────┐
  │                    ORDERS                         │
  │ order_id │ tenant_id │ customer_id │ total       │
  │ 1001     │ T_ACME    │ C001        │ 150.00      │
  │ 1002     │ T_GLOBEX  │ C001        │ 275.00      │
  │ 1003     │ T_ACME    │ C002        │ 89.50       │
  └──────────────────────────────────────────────────┘
  Every query MUST filter: WHERE tenant_id = 'T_ACME'
  Often enforced with Row-Level Security (RLS)

  ✅ Cheapest, simplest to manage
  ❌ Risk of data leakage if tenant_id filter forgotten
  ❌ "Noisy neighbor" — one tenant's heavy queries slow others

  ──────────────────────────────────────────
  Strategy 2: SEPARATE SCHEMAS (one per tenant)
  ──────────────────────────────────────────
  Database: app_db
  Schema: acme.orders, acme.customers
  Schema: globex.orders, globex.customers

  ✅ Better logical isolation
  ❌ Schema migrations across N tenants = operational complexity

  ──────────────────────────────────────────
  Strategy 3: SEPARATE DATABASES (maximum isolation)
  ──────────────────────────────────────────
  Database: acme_db  →  orders, customers
  Database: globex_db → orders, customers

  ✅ Maximum isolation, security, per-tenant customization
  ❌ Most expensive, hardest to manage at scale
```

| Strategy | Isolation | Cost | Complexity | Best For |
|---|---|---|---|---|
| **Shared schema** | Low | Low | Low | SaaS with many small tenants |
| **Separate schemas** | Medium | Medium | Medium | Moderate customization needs |
| **Separate databases** | High | High | High | Enterprise, compliance-critical |

---

## 12.7 Data Quality Dimensions

Schema design is your first line of defense for data quality. These six dimensions define what "quality" means:

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                    THE SIX DIMENSIONS OF DATA QUALITY           │
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                 │
  │  1. ACCURACY     — Does the data reflect reality?               │
  │     Schema tool: CHECK constraints, data type validation        │
  │     Example: age CHECK (age >= 0 AND age <= 150)                │
  │                                                                 │
  │  2. COMPLETENESS — Is all required data present?                │
  │     Schema tool: NOT NULL constraints                           │
  │     Example: email VARCHAR(255) NOT NULL                        │
  │                                                                 │
  │  3. CONSISTENCY  — Does the same fact match across locations?   │
  │     Schema tool: FOREIGN KEY constraints, normalization         │
  │     Example: FK ensures customer_id in orders EXISTS in         │
  │              customers; 3NF prevents redundant storage          │
  │                                                                 │
  │  4. TIMELINESS   — Is the data available when needed?           │
  │     Schema tool: Timestamps (created_at, updated_at)            │
  │     Example: updated_at reveals stale records                   │
  │                                                                 │
  │  5. VALIDITY     — Does the data conform to rules/formats?     │
  │     Schema tool: ENUM types, CHECK constraints, REGEX           │
  │     Example: status IN ('active','inactive','suspended')        │
  │                                                                 │
  │  6. UNIQUENESS   — Is each entity represented exactly once?     │
  │     Schema tool: UNIQUE constraints, PRIMARY KEY                │
  │     Example: UNIQUE(email) prevents duplicate customers         │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

**How schema constraints enforce data quality:**

| Constraint | Quality Dimension | What It Prevents |
|---|---|---|
| **PRIMARY KEY** | Uniqueness | Duplicate rows |
| **NOT NULL** | Completeness | Missing required data |
| **UNIQUE** | Uniqueness | Duplicate values in a column |
| **FOREIGN KEY** | Consistency | Orphan references (order → nonexistent customer) |
| **CHECK** | Accuracy, Validity | Invalid values (negative prices, impossible dates) |
| **DEFAULT** | Completeness | Missing values where a sensible default exists |
| **ENUM / domain types** | Validity | Arbitrary values where only specific ones are allowed |

**Palantir Connection:** Data quality is a core DS concern. When integrating a client's messy data into Foundry, you assess these six dimensions to build cleaning pipelines. The Ontology itself enforces consistency and validity through typed Properties and Link constraints.

---

## 12.8 Connecting It All — Data Modeling at Palantir

| Palantir Concept | Data Modeling Equivalent | Chapter Section |
|---|---|---|
| **Object Type** (Customer, Order) | Entity in ER diagram; Table in schema | §12.2 |
| **Property** (name, status, price) | Attribute in ER; Column in table | §12.2 |
| **Link** (Customer → Orders) | Relationship in ER; FK or junction table | §12.2, 12.3 |
| **Ontology composition** | Logical model (translated from conceptual) | §12.3 |
| **Pipeline transforms** | Normalization/denormalization decisions | §12.4 |
| **Workshop dashboard** | Star schema query consumer | §12.5 |
| **SCD Type 2 in Ontology** | Versioned Object Types with effective dates | §12.5.5 |
| **Conformed dimensions** | Shared Object Types across applications | §12.5.6 |
| **Data quality checks** | Pipeline rules enforcing the 6 dimensions | §12.7 |

---

## 12.9 Glossary

| Term | Definition |
|---|---|
| **1NF** | First Normal Form — atomic values, no repeating groups |
| **2NF** | Second Normal Form — no partial dependencies; every non-key depends on the whole key |
| **3NF** | Third Normal Form — no transitive dependencies; non-key depends only on the key |
| **Accumulating snapshot** | Fact table tracking process milestones; rows are updated as stages complete |
| **Audit trail** | Chronological record of all data changes for compliance and debugging |
| **BCNF** | Boyce-Codd Normal Form — every determinant is a superkey; stricter than 3NF |
| **Bitemporal** | Tracking both valid time and transaction time independently |
| **Cardinality** | The numerical relationship between entities: 1:1, 1:M, or M:N |
| **Composite key** | Primary key made of two or more columns |
| **Conformed dimension** | Dimension shared across multiple fact tables for consistent analysis |
| **Conceptual model** | High-level data model (ER diagram) focused on entities and relationships |
| **Delete anomaly** | Removing a row unintentionally deletes unrelated data |
| **Denormalization** | Intentionally adding redundancy to optimize read performance |
| **Dimension table** | Descriptive attributes providing context to facts (who, what, where, when) |
| **ER diagram** | Entity-Relationship diagram — visual representation of data structure |
| **Fact table** | Central table storing measurable business events and foreign keys to dimensions |
| **Foreign key** | Column referencing another table's primary key to enforce relationships |
| **Insert anomaly** | Can't insert data without also adding unrelated information |
| **Junction table** | Bridge table resolving M:N relationships using composite foreign keys |
| **Kimball methodology** | Bottom-up dimensional modeling approach for data warehouses |
| **Logical model** | Table/column-level schema derived from the conceptual model |
| **Multi-tenant** | Architecture serving multiple organizations from shared infrastructure |
| **Natural key** | Real-world identifier used as primary key (SSN, ISBN) |
| **Normalization** | Process of structuring tables to eliminate redundancy and anomalies |
| **Partial dependency** | Non-key attribute depends on only part of a composite key (violates 2NF) |
| **Participation** | Whether entity instances must (total) or may (partial) join a relationship |
| **Periodic snapshot** | Fact table capturing state at regular intervals (daily balances, monthly totals) |
| **SCD** | Slowly Changing Dimension — strategy for handling changes to dimension data over time |
| **Snowflake schema** | Star schema with normalized dimension tables |
| **Soft delete** | Marking records as deleted (via flag/timestamp) instead of physically removing them |
| **Star schema** | Central fact table connected to denormalized dimension tables |
| **Supertype/subtype** | "Is-a" hierarchy in ER modeling; inheritance of attributes |
| **Surrogate key** | System-generated identifier (auto-increment, UUID) with no business meaning |
| **Temporal data** | Data tracked with time dimensions (valid time, transaction time) |
| **Transactional fact** | Fact table with one row per individual business event |
| **Transitive dependency** | Non-key attribute depends on another non-key attribute (violates 3NF) |
| **Update anomaly** | Changing data in one row but not matching rows causes inconsistency |

---

## 12.10 Chapter Summary — The "Cheat Sheet"

```
┌─────────────────────────────────────────────────────────────────┐
│  CHAPTER 12 SUMMARY CARD                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPTUAL MODELING (ER Diagrams):                              │
│  • Entities (nouns) → Attributes (adjectives) → Relationships    │
│  • Cardinality: 1:1, 1:M, M:N                                   │
│  • Participation: Total (must) vs. Partial (may)                 │
│  • Supertype/subtype: "is-a" with inheritance                    │
│                                                                  │
│  LOGICAL MODELING (ER → Tables):                                 │
│  • Entities → tables; Attributes → columns; PKs, FKs            │
│  • 1:1 → FK in either table                                     │
│  • 1:M → FK in the "many" table                                 │
│  • M:N → junction table with composite FK/PK                    │
│                                                                  │
│  NORMALIZATION:                                                  │
│  • 1NF: atomic values, no repeating groups                       │
│  • 2NF: no partial dependencies (whole-key dependency)           │
│  • 3NF: no transitive dependencies (key-only dependency)         │
│  • BCNF: every determinant is a superkey                         │
│  • Why: prevent insert/update/delete anomalies                   │
│  • When to denormalize: read-heavy analytics, dashboards         │
│                                                                  │
│  DIMENSIONAL MODELING (Kimball):                                 │
│  • Star schema: fact table + denormalized dimensions             │
│  • Snowflake schema: normalized dimensions (more JOINs)          │
│  • Fact types: transactional, periodic snapshot, accumulating     │
│  • SCD: Type 1 (overwrite), Type 2 (new row), Type 3 (new col)  │
│  • Conformed dimensions: shared across fact tables               │
│                                                                  │
│  SCHEMA PATTERNS:                                                │
│  • Audit: created_at/updated_at + audit_log table                │
│  • Soft deletes: deleted_at timestamp (vs. hard delete)          │
│  • Temporal: valid_time + transaction_time → bitemporal           │
│  • Multi-tenant: shared schema < separate schemas < separate DBs │
│                                                                  │
│  DATA QUALITY (6 dimensions):                                    │
│  • Accuracy (CHECK), Completeness (NOT NULL),                    │
│    Consistency (FK + normalization), Timeliness (timestamps),    │
│    Validity (ENUM/CHECK), Uniqueness (PK/UNIQUE)                 │
│                                                                  │
│  PALANTIR CONNECTION:                                            │
│  • Object Types = entities; Properties = attributes              │
│  • Links = relationships (FKs or junction tables)                │
│  • Ontology = logical model of the business                      │
│  • Conformed dims = shared Object Types across apps              │
│  • Data quality dims = pipeline cleaning rules                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12.11 What's Next

With relational and dimensional modeling as the foundation, **Chapter 13: Data Modeling II — The Ontology, Graph Thinking, and Modern Paradigms** goes beyond traditional schemas. You'll learn how Palantir's Ontology extends these concepts, how graph databases model complex networks, and how modern paradigms like event sourcing and Data Vault bring new flexibility to data architecture.

---

*"Every Palantir deployment begins with a data model. Before a single pipeline is built, before a single dashboard is rendered, someone has to answer: 'What are the entities? How do they relate? What questions does the business need to answer?' That someone is the Deployment Strategist — and the tools in this chapter are the first tools you'll reach for."*
