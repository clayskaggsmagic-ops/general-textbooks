# Chapter 23: Schema Design — From Blank Page to Production Database

> *"Give me six hours to chop down a tree and I will spend the first four sharpening the axe."*
> — Abraham Lincoln

---

## 23.1 Why This Chapter Exists

Every other chapter in this book teaches you *about* schemas. This chapter teaches you to **design one from scratch**.

If someone said "here's a business — design the database," could you do it right now? If the answer is "not really" or "I'd just start making tables and hope for the best," this chapter is for you. By the end, you'll have a repeatable process for turning any real-world scenario into a well-structured database.

Here's why this matters for a Deployment Strategist:

1. **Every Palantir engagement starts with schema.** When you integrate a client's data into Foundry, the first question is: "What does this data look like?" You need to read, evaluate, and sometimes redesign schemas.
2. **The interview tests this directly.** "Design a data model for X" is a core DS interview question. You'll be expected to make and defend schema decisions live.
3. **Wrong schema = wrong everything.** A bad schema leads to slow queries, wrong answers, painful maintenance, and frustrated users. A good schema makes everything downstream easier — analytics, dashboards, machine learning, and Ontology design.

> **What this chapter is NOT:** This is not a chapter about SQL syntax or query writing. This is about *designing the structure* that SQL queries will eventually run against. Think of it like architecture vs. construction — we're drawing the blueprints here, not swinging the hammer.

---

## 23.2 What Is a Schema? (The Real Explanation)

### The Word Itself

**Schema** (pronounced "SKEE-muh") comes from Greek, meaning "form" or "plan." In databases, a schema is the **blueprint** that defines:

- What **tables** exist
- What **columns** each table has
- What **data type** each column stores (text, numbers, dates, etc.)
- What **rules** govern the data (constraints)
- How tables **relate** to each other

Think of it this way:

```
  REAL WORLD ANALOGY:

  A schema is to a database what a floor plan is to a building.

  Floor plan says:                    Schema says:
  • 3 bedrooms, 2 bathrooms           • 5 tables
  • Master bedroom is 14x16           • "customers" table has 8 columns
  • Kitchen connects to dining room   • "orders" table connects to "customers"
  • Each room has a door (entry rule) • customer_id must be unique (constraint)

  The floor plan doesn't contain furniture.
  The schema doesn't contain data.

  The floor plan DEFINES THE SHAPE of the building.
  The schema DEFINES THE SHAPE of the database.
```

### A Concrete Example

Let's say you run a coffee shop. You need to track customers and their orders. Here's what a schema looks like in plain English vs. SQL:

**Plain English schema:**
- A **Customers** table with: customer ID (unique number), name (text), email (text), join date
- An **Orders** table with: order ID (unique number), which customer placed it, what they ordered, how much it cost, when they ordered

**The same thing as a SQL schema (DDL — Data Definition Language):**

```sql
CREATE TABLE customers (
    customer_id    INT PRIMARY KEY,          -- unique identifier, one per customer
    name           VARCHAR(100) NOT NULL,    -- text, up to 100 characters, required
    email          VARCHAR(255) UNIQUE,      -- text, must be unique across all customers
    joined_date    DATE DEFAULT CURRENT_DATE -- defaults to today if not specified
);

CREATE TABLE orders (
    order_id       INT PRIMARY KEY,
    customer_id    INT NOT NULL,             -- which customer placed this order
    item           VARCHAR(100) NOT NULL,
    amount         DECIMAL(10,2) NOT NULL,   -- dollars and cents (e.g., 4.50)
    order_date     TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    -- ^ THIS is the relationship. This line says:
    -- "Every customer_id in orders MUST exist in the customers table."
    -- The database will REFUSE to insert an order for customer 999
    -- if there is no customer 999 in the customers table.
);
```

**That's it.** That's a schema. Two tables, their columns, their data types, and the rules connecting them.

### The Three Things Every Schema Defines

Every schema answers exactly three questions:

| Question | Schema Element | Example |
|----------|---------------|---------|
| **What things exist?** | Tables | customers, orders, products |
| **What do we know about each thing?** | Columns (with data types) | name (text), price (decimal), created_at (timestamp) |
| **How do things relate?** | Foreign keys and constraints | orders.customer_id → customers.customer_id |

---

## 23.3 The Two Worlds: OLTP vs. OLAP

Before you can design a schema, you must answer one question: **What is this database FOR?**

Databases serve two fundamentally different purposes, and each one needs a completely different schema design. This is the single most important concept in this entire chapter.

### OLTP — The Operational System

**OLTP** stands for **Online Transaction Processing**. These are the databases that run businesses in real time.

```
  WHAT OLTP DOES:

  • A customer places an order on Amazon → OLTP database records it
  • A nurse updates a patient's chart → OLTP database stores the change
  • A bank processes a wire transfer → OLTP database moves the money
  • A barista rings up your latte → OLTP database logs the sale

  CHARACTERISTICS:
  • Lots of small, fast operations (thousands per second)
  • Mostly INSERT, UPDATE, DELETE (writing data)
  • Each operation touches very few rows (1 order, 1 patient, 1 transfer)
  • Must be FAST — customers are waiting
  • Must be CORRECT — you can't lose a bank transfer
  • Concurrent users — hundreds of people using it simultaneously
```

**Real-world OLTP systems you've used today:** your bank's app, Amazon's checkout, Uber's ride system, Starbucks' POS terminal, your hospital's patient records.

### OLAP — The Analytical System

**OLAP** stands for **Online Analytical Processing**. These are the databases that answer big questions about the business.

```
  WHAT OLAP DOES:

  • "What were total sales by region last quarter?" → OLAP answers it
  • "Which products have declining margins over 3 years?" → OLAP answers it
  • "How does patient readmission rate vary by hospital?" → OLAP answers it
  • "What's the average order size by customer segment?" → OLAP answers it

  CHARACTERISTICS:
  • Few queries, but each one is HUGE (scanning millions of rows)
  • Mostly SELECT (reading data, not changing it)
  • Each query touches many rows and many columns
  • Speed matters, but correctness of aggregations matters more
  • Few concurrent users (analysts, not customers)
  • Data is loaded in batches (nightly, hourly), not real-time
```

**Real-world OLAP systems:** Snowflake dashboards, Tableau reports, executive KPI dashboards, any "business intelligence" tool.

### Why They Need Different Schemas

Here's the key insight that confuses beginners: **the same data needs to be organized differently depending on what you're doing with it.**

Imagine you have a closet full of clothes:

```
  OLTP SCHEMA = Organized by TYPE (all shirts together, all pants together, all shoes together)
  
  Why? Because when you get dressed in the morning (a "transaction"), you need to quickly
  find ONE specific shirt. Organization by type makes individual item retrieval fast.
  
  But if someone asks "How many blue items do you own?" — you have to search EVERY section.
  Slow.

  ───────────────────────────────────────────────

  OLAP SCHEMA = Organized by OUTFIT (complete outfits laid out together with all attributes tagged)
  
  Why? Because when you're answering questions like "What colors do I wear most?" or
  "Which season has the most expensive outfits?" — you need everything pre-organized
  for analysis. Quick to answer big questions.
  
  But if you need to swap one shirt? You have to find it scattered across multiple outfits.
  Slow for individual changes.
```

Let me make this concrete with actual database schemas:

### The Same Business, Two Different Schemas

**Scenario:** An online bookstore sells books to customers.

**OLTP Schema (Normalized — Optimized for Transactions):**

```sql
-- Each piece of information stored ONCE, in ONE place
-- This eliminates redundancy but requires JOINs to reassemble

CREATE TABLE customers (
    customer_id    INT PRIMARY KEY,
    name           VARCHAR(100),
    email          VARCHAR(255)
);

CREATE TABLE addresses (
    address_id     INT PRIMARY KEY,
    customer_id    INT REFERENCES customers(customer_id),
    street         VARCHAR(200),
    city           VARCHAR(100),
    state          CHAR(2),
    zip            VARCHAR(10),
    address_type   VARCHAR(20)  -- 'billing', 'shipping'
);

CREATE TABLE books (
    book_id        INT PRIMARY KEY,
    title          VARCHAR(300),
    isbn           VARCHAR(13),
    publisher_id   INT REFERENCES publishers(publisher_id),
    price          DECIMAL(10,2),
    page_count     INT
);

CREATE TABLE publishers (
    publisher_id   INT PRIMARY KEY,
    name           VARCHAR(200),
    country        VARCHAR(100)
);

CREATE TABLE authors (
    author_id      INT PRIMARY KEY,
    name           VARCHAR(200),
    birth_year     INT
);

CREATE TABLE book_authors (          -- junction table for many-to-many
    book_id        INT REFERENCES books(book_id),
    author_id      INT REFERENCES authors(author_id),
    PRIMARY KEY (book_id, author_id)
);

CREATE TABLE orders (
    order_id       INT PRIMARY KEY,
    customer_id    INT REFERENCES customers(customer_id),
    order_date     TIMESTAMP,
    shipping_addr  INT REFERENCES addresses(address_id),
    status         VARCHAR(20)
);

CREATE TABLE order_items (
    order_id       INT REFERENCES orders(order_id),
    book_id        INT REFERENCES books(book_id),
    quantity       INT,
    unit_price     DECIMAL(10,2),    -- price at time of purchase
    PRIMARY KEY (order_id, book_id)
);
```

**Count: 8 tables.** Customer name is stored ONCE. Publisher name is stored ONCE. If a publisher changes their name, you update ONE row.

**That's the power of normalization.** But what does "normalization" actually mean? **Normalization is the process of organizing your data so that every piece of information is stored in exactly one place.** Instead of cramming everything into one giant table where the same customer name, publisher name, or address gets copied across hundreds of rows, you split the data into separate, focused tables — one for customers, one for publishers, one for books, etc. — and connect them with foreign keys. This eliminates redundancy (no duplicate data) and prevents nasty problems like updating a customer's email in one row but forgetting to update it in fifty others. We'll go deep on the specific rules of normalization (1NF, 2NF, 3NF) in section 21.4 below, but the core idea is simple: **one fact, one place.**

Now look at the OLAP version of the same data:

**OLAP Schema (Denormalized Star Schema — Optimized for Analytics):**

```sql
-- The FACT table: one row per book sold (the measurable event)
CREATE TABLE fact_sales (
    sale_id          INT PRIMARY KEY,
    date_key         INT,         -- FK → dim_date
    customer_key     INT,         -- FK → dim_customer
    book_key         INT,         -- FK → dim_book
    -- MEASURES (the numbers you aggregate):
    quantity         INT,
    unit_price       DECIMAL(10,2),
    total_amount     DECIMAL(10,2),
    discount_amount  DECIMAL(10,2)
);

-- DIMENSION tables: the context around each sale
CREATE TABLE dim_date (
    date_key         INT PRIMARY KEY,
    full_date        DATE,
    day_of_week      VARCHAR(10),   -- 'Monday', 'Tuesday'...
    month            INT,
    month_name       VARCHAR(10),   -- 'January', 'February'...
    quarter          INT,
    year             INT,
    is_weekend       BOOLEAN,
    is_holiday       BOOLEAN
);

CREATE TABLE dim_customer (
    customer_key     INT PRIMARY KEY,
    customer_id      INT,            -- original ID from OLTP
    name             VARCHAR(100),
    email            VARCHAR(255),
    city             VARCHAR(100),   -- DENORMALIZED: was in separate address table
    state            CHAR(2),        -- DENORMALIZED: was in separate address table
    segment          VARCHAR(50)     -- 'Premium', 'Standard', 'New'
);

CREATE TABLE dim_book (
    book_key         INT PRIMARY KEY,
    book_id          INT,
    title            VARCHAR(300),
    isbn             VARCHAR(13),
    author_name      VARCHAR(200),   -- DENORMALIZED: was in separate author table
    publisher_name   VARCHAR(200),   -- DENORMALIZED: was in separate publisher table
    genre            VARCHAR(50),
    price_tier       VARCHAR(20)     -- 'Budget', 'Mid-Range', 'Premium'
);
```

**Count: 4 tables.** Customer city is EMBEDDED in dim_customer (not a separate address table). Author name is EMBEDDED in dim_book (not a separate authors table). Redundant? Yes. But now watch what happens:

**Query comparison — "What are total sales by genre and quarter?"**

```sql
-- OLTP schema (normalized): requires 4 JOINs
SELECT 
    b.genre, 
    EXTRACT(QUARTER FROM o.order_date) AS quarter,
    SUM(oi.quantity * oi.unit_price) AS total_sales
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN books b ON oi.book_id = b.book_id
GROUP BY b.genre, EXTRACT(QUARTER FROM o.order_date);

-- OLAP schema (star): requires 2 simple JOINs  
SELECT 
    b.genre,
    d.quarter,
    SUM(f.total_amount) AS total_sales
FROM fact_sales f
JOIN dim_book b ON f.book_key = b.book_key
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY b.genre, d.quarter;
```

The star schema query is simpler, faster, and easier to understand. That's why analytics databases use denormalized schemas.

---

## 23.4 Normalization — The Rules of OLTP Schema Design

### What Is Normalization?

**Normalization** is a set of rules for organizing data to **eliminate redundancy** and **prevent data anomalies**. Think of it as Marie Kondo for databases — everything has exactly one place, and nothing is duplicated.

### Why Do We Care?

Let's see what happens WITHOUT normalization. Imagine you store everything in one big flat table:

```
  ┌──────────────────────────────────────────────────────────────────────────────────┐
  │                          BAD: One Giant "orders" Table                          │
  ├──────────┬───────────┬──────────────┬──────────┬──────────────┬────────────────┤
  │ order_id │ cust_name │ cust_email   │ book     │ author       │ publisher      │
  ├──────────┼───────────┼──────────────┼──────────┼──────────────┼────────────────┤
  │ 1001     │ Alice     │ alice@e.com  │ Dune     │ Frank Herbert│ Chilton Books  │
  │ 1002     │ Alice     │ alice@e.com  │ 1984     │ George Orwell│ Secker&Warburg │
  │ 1003     │ Bob       │ bob@e.com    │ Dune     │ Frank Herbert│ Chilton Books  │
  │ 1004     │ Alice     │ alice@e.com  │ Neuromcr │ William Gibs │ Ace Books      │
  └──────────┴───────────┴──────────────┴──────────┴──────────────┴────────────────┘
```

**Three terrible problems emerge:**

**1. Update Anomaly** — Alice changes her email. You have to update rows 1001, 1002, AND 1004. Miss one? Now Alice has TWO different emails in your database. Which is correct? Nobody knows.

**2. Insert Anomaly** — A new publisher (Penguin) joins your catalog, but nobody has ordered their books yet. You *can't add Penguin to the database* because every row requires an order_id. No order = no way to record the publisher exists.

**3. Delete Anomaly** — Bob's only order is #1003. If you delete that order (he returned it), you also lose the fact that "Dune" is published by "Chilton Books." Deleting a transaction destroys reference data.

**Normalization fixes ALL THREE problems** by splitting this one messy table into multiple clean tables, each storing one type of information.

### The Normal Forms — Step by Step

Normal forms are a ladder. Each level fixes a specific type of problem. You climb one rung at a time.

#### First Normal Form (1NF) — Atomic Values

**Rule:** Every cell contains ONE value. No lists, no repeating groups.

```
  ❌ VIOLATES 1NF:
  ┌──────────┬───────────┬──────────────────────────────┐
  │ order_id │ customer  │ items                        │
  ├──────────┼───────────┼──────────────────────────────┤
  │ 1001     │ Alice     │ Dune, 1984, Neuromancer      │  ← THREE values in one cell!
  │ 1002     │ Bob       │ Dune                         │
  └──────────┴───────────┴──────────────────────────────┘

  Problem: How do you query "all orders containing Dune"?
  You'd need string parsing: WHERE items LIKE '%Dune%'
  That's slow, fragile, and won't use indexes.

  ✅ FIXED (1NF):
  ┌──────────┬───────────┬──────────────┐
  │ order_id │ customer  │ item         │
  ├──────────┼───────────┼──────────────┤
  │ 1001     │ Alice     │ Dune         │  ← ONE value per cell
  │ 1001     │ Alice     │ 1984         │
  │ 1001     │ Alice     │ Neuromancer  │
  │ 1002     │ Bob       │ Dune         │
  └──────────┴───────────┴──────────────┘
  
  Now: WHERE item = 'Dune' — fast, clean, indexable.
```

#### Second Normal Form (2NF) — No Partial Dependencies

**Rule:** 1NF + every non-key column depends on the *entire* primary key, not just part of it.

This only matters when your primary key is **composite** (made of multiple columns).

```
  ❌ VIOLATES 2NF:
  
  Primary key: (order_id, book_id)  — two columns together
  
  ┌──────────┬─────────┬──────────┬─────────────┬───────┐
  │ order_id │ book_id │ quantity │ book_title   │ price │
  ├──────────┼─────────┼──────────┼─────────────┼───────┤
  │ 1001     │ 55      │ 2        │ Dune        │ 14.99 │
  │ 1001     │ 72      │ 1        │ 1984        │  9.99 │
  │ 1002     │ 55      │ 1        │ Dune        │ 14.99 │
  └──────────┴─────────┴──────────┴─────────────┴───────┘

  "quantity" depends on BOTH order_id AND book_id ✅ (how many of book X in order Y)
  "book_title" depends ONLY on book_id ❌ (the title is always "Dune" regardless of order)
  "price" depends ONLY on book_id ❌ (same problem)

  This is a PARTIAL DEPENDENCY — some columns depend on only PART of the key.

  ✅ FIXED (2NF): Split into two tables

  Table: order_items (PK: order_id, book_id)
  ┌──────────┬─────────┬──────────┐
  │ order_id │ book_id │ quantity │  ← depends on the full key
  └──────────┴─────────┴──────────┘

  Table: books (PK: book_id)
  ┌─────────┬─────────────┬───────┐
  │ book_id │ book_title   │ price │  ← depends only on book_id, so it gets its own table
  └─────────┴─────────────┴───────┘
```

#### Third Normal Form (3NF) — No Transitive Dependencies

**Rule:** 2NF + non-key columns don't depend on OTHER non-key columns.

```
  ❌ VIOLATES 3NF:

  Table: books (PK: book_id)
  ┌─────────┬────────────┬──────────────┬──────────────────┐
  │ book_id │ title      │ publisher_id │ publisher_name   │
  ├─────────┼────────────┼──────────────┼──────────────────┤
  │ 55      │ Dune       │ 10           │ Chilton Books    │
  │ 72      │ 1984       │ 20           │ Secker & Warburg │
  │ 88      │ Fahrenheit │ 10           │ Chilton Books    │  ← "Chilton Books" repeated!
  └─────────┴────────────┴──────────────┴──────────────────┘

  publisher_name depends on publisher_id (not on book_id).
  That's a TRANSITIVE dependency: book_id → publisher_id → publisher_name.

  Problem: If Chilton Books changes its name, you update multiple rows. Miss one? Inconsistency.

  ✅ FIXED (3NF): Split into two tables

  Table: books (PK: book_id)
  ┌─────────┬────────────┬──────────────┐
  │ book_id │ title      │ publisher_id │  ← publisher_id is a foreign key
  └─────────┴────────────┴──────────────┘

  Table: publishers (PK: publisher_id)
  ┌──────────────┬──────────────────┐
  │ publisher_id │ publisher_name   │  ← stored ONCE
  └──────────────┴──────────────────┘
```

### The Normalization Summary

| Form | Rule | What It Prevents |
|------|------|-----------------|
| **1NF** | Atomic values, no repeating groups | Can't query individual values in a list |
| **2NF** | No partial key dependencies | Redundancy when composite keys are used |
| **3NF** | No transitive dependencies | Redundancy from non-key → non-key chains |
| **BCNF** | Every determinant is a candidate key | Rare edge cases where 3NF still has redundancy |

> **Practical rule:** Most OLTP systems aim for **3NF**. Going beyond BCNF is rarely done in practice. The goal is: *every fact is stored once, in one place.*

---

## 23.5 Denormalization — Breaking the Rules on Purpose

### When Breaking Rules Is Smart

Normalization is beautiful in theory. In practice, highly normalized schemas can be **slow for analytical queries** because reassembling information requires many JOINs.

```
  THE FUNDAMENTAL TRADE-OFF:

  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │   MORE NORMALIZED                    LESS NORMALIZED            │
  │   (3NF, BCNF)                       (Denormalized)             │
  │                                                                 │
  │   ✅ No redundancy                   ❌ Redundant data          │
  │   ✅ Easy to update                  ❌ Updates touch many rows │
  │   ✅ Data integrity guaranteed       ❌ Risk of inconsistency   │
  │   ❌ Many JOINs for complex queries  ✅ Fewer JOINs needed     │
  │   ❌ Slower reads (many tables)      ✅ Faster reads            │
  │   ❌ Harder for analysts to use      ✅ Intuitive for analysts  │
  │                                                                 │
  │   Best for: OLTP                     Best for: OLAP            │
  │   (operational systems)              (analytical systems)       │
  └─────────────────────────────────────────────────────────────────┘
```

**Denormalization** means intentionally adding redundancy back into your schema to speed up reads. You're *choosing* to store data in multiple places because the performance benefit outweighs the maintenance cost.

### Common Denormalization Techniques

**1. Embedding foreign data directly:**
```sql
-- Normalized: separate tables
SELECT o.*, c.name, c.email 
FROM orders o JOIN customers c ON o.customer_id = c.customer_id;

-- Denormalized: customer info copied into orders
SELECT order_id, customer_name, customer_email, total
FROM orders_denormalized;
-- No JOIN needed! But if customer changes email, you must update every order row.
```

**2. Pre-computed aggregates:**
```sql
-- Normalized: calculate on the fly
SELECT customer_id, COUNT(*) AS total_orders, SUM(amount) AS lifetime_spend
FROM orders GROUP BY customer_id;
-- Scans every order, every time someone asks.

-- Denormalized: store the aggregate
ALTER TABLE customers ADD COLUMN total_orders INT DEFAULT 0;
ALTER TABLE customers ADD COLUMN lifetime_spend DECIMAL(10,2) DEFAULT 0.00;
-- Instant lookup! But you must update these every time a new order arrives.
```

**3. Redundant columns for common filters:**
```sql
-- Normalized: to filter orders by customer city, you JOIN to addresses
SELECT o.* FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN addresses a ON c.customer_id = a.customer_id
WHERE a.city = 'Austin';

-- Denormalized: add city directly to orders
SELECT * FROM orders WHERE customer_city = 'Austin';
-- No JOINs! But city is stored in two places now.
```

---

## 23.6 Star Schemas and Snowflake Schemas — OLAP Design Patterns

In section 21.3, you saw that OLAP databases need a different kind of schema than OLTP databases. Star schemas and snowflake schemas are the two standard blueprints for how to organize an OLAP database. They were invented specifically to make analytical queries — the kind that power dashboards, reports, and executive KPIs — run fast and be easy to write. Let's build them up from first principles.

### Why Does Analytics Need Its Own Schema Pattern?

Imagine an analyst at a pizza chain asks: *"What were our total sales by topping category and month for the last year?"*

In a normalized OLTP database, answering that question would require joining 5-6 tables together: orders → order_items → products → product_categories → dates. Each JOIN makes the query slower and harder to write. Analysts ask hundreds of questions like this every day, and they shouldn't need to understand the internal plumbing of a normalized database to get answers.

Star and snowflake schemas solve this by **pre-organizing the data around the questions people will ask.** Instead of optimizing for "store individual transactions safely" (OLTP's job), they optimize for "answer aggregate questions quickly" (OLAP's job).

### The Star Schema — The Most Important OLAP Pattern

The **star schema** gets its name from its shape when you draw it: a central table with several tables radiating outward like the points of a star.

```
                    ┌──────────────┐
                    │  dim_date    │
                    └──────┬───────┘
                           │
  ┌──────────────┐   ┌─────┴──────┐   ┌──────────────┐
  │ dim_customer ├───┤ fact_sales  ├───┤  dim_product  │
  └──────────────┘   └─────┬──────┘   └──────────────┘
                           │
                    ┌──────┴───────┐
                    │  dim_store   │
                    └──────────────┘

  The center = FACT table.   The points of the star = DIMENSION tables.
```

There are exactly two types of tables in a star schema, and understanding the difference between them is everything:

---

#### Fact Tables — "What Happened"

The **fact table** sits at the center of the star. It records **events** — things that happened in the business. Every row in the fact table represents one occurrence of that event.

**What's inside a fact table:**

1. **Measures** — These are the numbers you care about. They're the columns that analysts will SUM, COUNT, or AVERAGE. Think: revenue, quantity sold, discount amount, shipping cost, time spent. Measures answer the question *"how much?"* or *"how many?"*

2. **Foreign keys** — These are integer columns that point to the dimension tables. They answer the question *"what was the context of this event?"* — which customer, which product, which date, which store.

**What's NOT inside a fact table:** descriptive text. You won't find the customer's name, the product's category, or the store's city in the fact table. That information lives in the dimension tables. The fact table only stores the numeric keys that *point* to those descriptions.

**Why?** Because the fact table is usually *enormous* — millions or billions of rows. Storing "San Francisco, California, United States, Western Region" in every single row would waste massive amounts of space. Instead, you store a single integer (like `store_key = 42`) and look up the details in the much smaller dimension table when you need them.

Let's make this concrete. Here's a fact table for our pizza chain:

```sql
CREATE TABLE fact_pizza_sales (
    -- FOREIGN KEYS (the "who/what/where/when" pointers):
    sale_key         INT PRIMARY KEY,    -- unique ID for this row
    date_key         INT,                -- points to dim_date → WHEN was this sold?
    store_key        INT,                -- points to dim_store → WHERE was it sold?
    customer_key     INT,                -- points to dim_customer → WHO bought it?
    pizza_key        INT,                -- points to dim_pizza → WHAT was sold?

    -- MEASURES (the numbers analysts will aggregate):
    quantity         INT,                -- how many pizzas in this line item
    unit_price       DECIMAL(8,2),       -- price per pizza
    total_amount     DECIMAL(10,2),      -- quantity × unit_price
    discount_amount  DECIMAL(8,2),       -- any coupon or promo discount
    delivery_fee     DECIMAL(8,2)        -- $0 for pickup, otherwise delivery charge
);
```

**Example rows in this fact table:**

```
  sale_key │ date_key │ store_key │ customer_key │ pizza_key │ qty │ unit_price │ total  │ discount │ del_fee
  ─────────┼──────────┼───────────┼──────────────┼───────────┼─────┼────────────┼────────┼──────────┼────────
  1        │ 20240115 │ 3         │ 501          │ 12        │ 2   │ 14.99      │ 29.98  │ 5.00     │ 3.99
  2        │ 20240115 │ 3         │ 501          │ 7         │ 1   │ 11.99      │ 11.99  │ 0.00     │ 0.00
  3        │ 20240116 │ 1         │ 802          │ 12        │ 1   │ 14.99      │ 14.99  │ 0.00     │ 0.00
```

Notice: row 1 tells you that *something* was sold on date 20240115 at store 3 by customer 501 and it was pizza 12, quantity 2, totaling $29.98 with a $5 discount and $3.99 delivery fee. But it doesn't tell you that date 20240115 is a Monday in January, or that store 3 is in Chicago, or that customer 501 is "Alice Martinez," or that pizza 12 is a "Large Pepperoni." All of those descriptive details live in the dimension tables.

---

#### Dimension Tables — "The Context Around What Happened"

**Dimension tables** are the points of the star. They contain the **descriptive attributes** — the words, labels, and categories that give meaning to the numbers in the fact table. When an analyst says "show me sales **by region**" or "filter **by pizza category**" or "break it down **by month**," they're using dimension attributes.

Each dimension table answers one of the classic journalist questions:
- **dim_date** → WHEN did it happen?
- **dim_store** → WHERE did it happen?
- **dim_customer** → WHO was involved?
- **dim_pizza** → WHAT was sold?

Here are the dimension tables for our pizza chain:

```sql
-- WHEN: The date dimension (every analytics schema has one)
CREATE TABLE dim_date (
    date_key         INT PRIMARY KEY,       -- e.g., 20240115 (often YYYYMMDD format)
    full_date        DATE,                  -- 2024-01-15
    day_name         VARCHAR(10),           -- 'Monday'
    day_of_week      INT,                   -- 2 (Monday=2 in ISO)
    day_of_month     INT,                   -- 15
    week_of_year     INT,                   -- 3
    month_number     INT,                   -- 1
    month_name       VARCHAR(10),           -- 'January'
    quarter          INT,                   -- 1
    year             INT,                   -- 2024
    is_weekend       BOOLEAN,              -- FALSE
    is_holiday       BOOLEAN,              -- FALSE (TRUE for July 4th, etc.)
    fiscal_quarter   INT,                   -- company's fiscal calendar (may differ from calendar)
    season           VARCHAR(10)            -- 'Winter'
);
```

**Why is dim_date so detailed?** Because analysts constantly slice data by time in ways you can't easily compute from a raw date. "Show me weekend vs. weekday sales" requires knowing which dates are weekends. "Compare Q4 to Q1" requires quarter labels. "What's our holiday performance?" requires a holiday flag. Pre-computing all of this into the dimension table means analysts never have to write date-math functions — they just filter: `WHERE d.is_weekend = TRUE`.

```sql
-- WHERE: The store dimension
CREATE TABLE dim_store (
    store_key        INT PRIMARY KEY,
    store_id         INT,                   -- original ID from the OLTP system
    store_name       VARCHAR(100),          -- 'Downtown Chicago'
    address          VARCHAR(200),          -- '123 Michigan Ave'
    city             VARCHAR(100),          -- 'Chicago'
    state            VARCHAR(50),           -- 'Illinois'
    state_abbrev     CHAR(2),              -- 'IL'
    zip_code         VARCHAR(10),           -- '60601'
    region           VARCHAR(50),           -- 'Midwest'
    country          VARCHAR(50),           -- 'United States'
    store_type       VARCHAR(30),           -- 'Dine-in', 'Delivery-only', 'Express'
    square_footage   INT,                   -- 2400
    open_date        DATE,                  -- when did this store open?
    manager_name     VARCHAR(100)           -- 'Dave Johnson'
);

-- Notice: city, state, region, and country are ALL in one table.
-- In a normalized OLTP schema, these would be in separate tables
-- (cities → states → regions → countries). Here we deliberately
-- DENORMALIZE them into one flat table for simpler queries.
-- "Show me sales by region" is now a one-table lookup, not a 4-table JOIN chain.
```

```sql
-- WHO: The customer dimension
CREATE TABLE dim_customer (
    customer_key     INT PRIMARY KEY,
    customer_id      INT,                   -- original ID from the OLTP system
    first_name       VARCHAR(50),           -- 'Alice'
    last_name        VARCHAR(50),           -- 'Martinez'
    email            VARCHAR(255),
    phone            VARCHAR(20),
    city             VARCHAR(100),          -- DENORMALIZED from address table
    state            CHAR(2),
    zip_code         VARCHAR(10),
    signup_date      DATE,
    loyalty_tier     VARCHAR(20),           -- 'Gold', 'Silver', 'Bronze', 'None'
    age_bracket      VARCHAR(20),           -- '25-34', '35-44' (for demographic analysis)
    preferred_order  VARCHAR(20)            -- 'Delivery', 'Pickup', 'Dine-in'
);

-- WHAT: The pizza/product dimension
CREATE TABLE dim_pizza (
    pizza_key        INT PRIMARY KEY,
    pizza_id         INT,                   -- original ID from OLTP
    pizza_name       VARCHAR(100),          -- 'Large Pepperoni'
    size             VARCHAR(20),           -- 'Small', 'Medium', 'Large', 'XL'
    crust_type       VARCHAR(30),           -- 'Thin', 'Hand-Tossed', 'Deep Dish', 'Stuffed'
    category         VARCHAR(30),           -- 'Classic', 'Specialty', 'Veggie', 'Premium'
    is_vegetarian    BOOLEAN,              -- TRUE/FALSE
    is_gluten_free   BOOLEAN,
    base_price       DECIMAL(8,2),          -- standard menu price
    calorie_count    INT,                   -- 1200
    primary_topping  VARCHAR(50)            -- 'Pepperoni' (the dominant topping)
);
```

**Key thing to notice about dimension tables:** they're **wide** (lots of columns) but **short** (relatively few rows). Your pizza chain might have 50 stores, 200 products, and 100,000 customers — but you could have 50 million rows in fact_pizza_sales. The dimension tables are small lookup references. The fact table is the massive data warehouse.

---

#### How They Work Together — A Query in Action

Now watch the payoff. An analyst asks: *"What were total sales by pizza category and quarter for 2024, sorted by revenue?"*

```sql
SELECT
    p.category,                             -- from dim_pizza
    d.quarter,                              -- from dim_date
    d.year,                                 -- from dim_date
    SUM(f.total_amount) AS total_revenue,   -- MEASURE from fact table
    SUM(f.quantity)     AS pizzas_sold,     -- MEASURE from fact table
    COUNT(*)            AS num_transactions -- count of fact rows
FROM fact_pizza_sales f
JOIN dim_pizza p    ON f.pizza_key = p.pizza_key       -- link fact → pizza details
JOIN dim_date d     ON f.date_key  = d.date_key        -- link fact → date details
WHERE d.year = 2024
GROUP BY p.category, d.quarter, d.year
ORDER BY total_revenue DESC;
```

**Result:**

```
  category   │ quarter │ year │ total_revenue │ pizzas_sold │ num_transactions
  ───────────┼─────────┼──────┼───────────────┼─────────────┼─────────────────
  Classic    │ 4       │ 2024 │ $2,340,128    │ 156,008     │ 142,300
  Premium    │ 4       │ 2024 │ $1,890,445    │ 94,522      │ 88,100
  Specialty  │ 3       │ 2024 │ $1,670,332    │ 111,355     │ 101,200
  Classic    │ 3       │ 2024 │ $1,650,200    │ 110,013     │ 98,400
  ...
```

**Why this is powerful:** the analyst wrote a simple query with only 2 JOINs, and they never had to think about individual transactions, address normalization, or complex date-math. They just said "give me category, quarter, and sums" — and the star schema made it easy.

Compare that to the same question on a normalized OLTP schema: you'd need to join orders → order_items → products → product_categories, then use `EXTRACT(QUARTER FROM order_date)` for the date logic with no pre-built quarter column. More JOINs, more complexity, slower execution on large datasets.

---

#### The Grain — The Most Important Decision

**"Grain"** (also called **"granularity"**) means: **what does ONE ROW in the fact table represent?** This is the single most important decision you make when designing a star schema, because it determines what questions the schema can and cannot answer.

Think of it like the resolution on a camera:
- **Fine grain** (high resolution) = one row per individual pizza sold → you can zoom in on any specific transaction, but the table is huge
- **Coarse grain** (low resolution) = one row per store per day → the table is smaller and faster, but you can't see individual transactions anymore

**Examples of different grains for the same business:**

```
  GRAIN LEVEL        │ WHAT ONE ROW REPRESENTS                │ FACT TABLE SIZE     │ WHAT YOU CAN ANSWER
  ────────────────────┼────────────────────────────────────────┼─────────────────────┼──────────────────────────────
  Transaction-level   │ One pizza in one order                 │ 50 million rows/yr  │ "What did customer X buy on Jan 15?"
  Order-level         │ One complete order (may have 3 pizzas) │ 15 million rows/yr  │ "What's the average order value?"
  Daily store summary │ One store's total for one day          │ ~18,000 rows/yr     │ "Which store had the best Tuesday?"
  Monthly summary     │ One store's total for one month        │ 600 rows/yr         │ "What's the monthly trend by store?"
```

**The rule:** always define the grain FIRST, before designing any other part of the schema. If you don't, you'll end up with a fact table where some rows represent individual sales and other rows represent daily totals — and then every query produces garbage.

**How to choose the right grain:** ask "What is the most detailed question anyone will ever ask?" If someone might ask "Show me Alice's exact order on January 15th," you need transaction-level grain. If the most detailed question is "What do daily sales look like by store?" then daily grain is fine and your fact table will be much smaller and faster.

> **Practical rule for interviews:** When in doubt, go with the finest grain (transaction-level). You can always aggregate fine-grained data upward (SUM daily into monthly), but you can NEVER disaggregate coarse data downward (you can't split a monthly total back into individual transactions). It's better to have too much detail than too little.

---

#### Surrogate Keys vs. Natural Keys

You may have noticed that the dimension tables use keys like `pizza_key` instead of just using the original `pizza_id` from the OLTP system. This is deliberate.

- A **natural key** is the key from the original source system, like `pizza_id = 42` or `customer_id = 501`. It has real-world meaning.
- A **surrogate key** is a new, meaningless integer generated by the warehouse, like `pizza_key = 1, 2, 3...`. It has no real-world meaning — it's just an internal pointer.

**Why use surrogate keys?** Three reasons:

1. **Source system keys can change.** If the pizza chain switches POS systems, the old `pizza_id = 42` might become `product_code = 'PZ-0042'` in the new system. The surrogate key (`pizza_key = 12`) stays the same regardless.

2. **Multiple source systems.** If the chain acquires another pizza company, both companies might have a `customer_id = 501` — but they're different customers. Surrogate keys avoid this collision.

3. **Tracking history.** If a customer moves from Chicago to Miami, you might want to keep BOTH versions — "Customer 501 when they were in Chicago" and "Customer 501 when they're in Miami" — with different surrogate keys so that old sales are attributed to the old location and new sales to the new location. (This is called a **Slowly Changing Dimension**, covered below.)

> **In practice:** OLAP schemas almost always use surrogate keys. The original natural key is still stored in the dimension table (that's the `customer_id` or `pizza_id` column) so you can trace back to the source system, but the primary key of the dimension and all foreign keys in the fact table use surrogates.

---

#### Slowly Changing Dimensions (SCD) — When Dimension Data Changes Over Time

Here's a problem you wouldn't think about until it bites you: **dimension data changes.** Customers move cities. Stores switch regions. Products get reclassified.

When a customer moves from Chicago to Miami, what should your dim_customer table do? There are three common approaches, called **SCD Types**:

**Type 1 — Overwrite:** Just update the row. Chicago becomes Miami. Simple, but you lose history — all of that customer's old Chicago sales now look like Miami sales.

```
  BEFORE:   customer_key=501, city='Chicago'
  AFTER:    customer_key=501, city='Miami'     ← Chicago is gone forever
```

**Type 2 — Add a new row:** Create a new row with a new surrogate key. Now you have two versions of the customer. Old sales point to the Chicago row, new sales point to the Miami row.

```
  customer_key=501, customer_id=501, city='Chicago', valid_from='2020-01-01', valid_to='2024-06-30', is_current=FALSE
  customer_key=937, customer_id=501, city='Miami',   valid_from='2024-07-01', valid_to='9999-12-31', is_current=TRUE
```

**Type 3 — Add a column:** Store both the old and new value in the same row.

```
  customer_key=501, current_city='Miami', previous_city='Chicago'
```

> **What to know for interviews:** Type 2 is the most common in practice because it preserves full history. Just know that SCDs exist, and know that Type 2 "adds a new row for each change" — that's usually enough.

---

### The Snowflake Schema — When Dimensions Get Split Up

A **snowflake schema** starts as a star schema but then takes the dimension tables and **normalizes** them — splitting them into sub-tables to remove redundancy. The result looks like a snowflake because each dimension "branches" outward into multiple smaller tables.

#### Star vs. Snowflake — A Concrete Comparison

Let's look at dim_pizza in our pizza chain:

**STAR SCHEMA (dim_pizza is one flat table):**

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                          dim_pizza                             │
  ├──────────┬────────────────────┬────────┬─────────┬─────────────┤
  │ pizza_key│ pizza_name         │ size   │ category│ crust_type  │
  ├──────────┼────────────────────┼────────┼─────────┼─────────────┤
  │ 1        │ Small Margherita   │ Small  │ Classic │ Thin        │
  │ 2        │ Medium Margherita  │ Medium │ Classic │ Thin        │
  │ 3        │ Large Margherita   │ Large  │ Classic │ Thin        │
  │ 4        │ Small Pepperoni    │ Small  │ Classic │ Hand-Tossed │
  │ 5        │ Large BBQ Chicken  │ Large  │ Premium │ Deep Dish   │
  │ 6        │ Medium Veggie      │ Medium │ Veggie  │ Thin        │
  └──────────┴────────────────────┴────────┴─────────┴─────────────┘

  Notice: "Classic" is repeated 3 times. "Thin" is repeated 3 times.
  That's REDUNDANCY — but it's intentional in a star schema.
```

**SNOWFLAKE SCHEMA (dim_pizza is split into sub-tables):**

```
  dim_pizza (main):           dim_category:          dim_crust:
  ┌──────────┬────────────┐   ┌─────────┬─────────┐  ┌─────────┬─────────────┐
  │ pizza_key│ pizza_name │   │ cat_key │ category│  │crust_key│ crust_type  │
  │          │ cat_key    │   ├─────────┼─────────┤  ├─────────┼─────────────┤
  │          │ crust_key  │   │ 1       │ Classic │  │ 1       │ Thin        │
  │          │ size       │   │ 2       │ Premium │  │ 2       │ Hand-Tossed │
  ├──────────┼────────────┤   │ 3       │ Veggie  │  │ 3       │ Deep Dish   │
  │ 1        │ SM Margh   │   └─────────┴─────────┘  └─────────┴─────────────┘
  │          │ cat_key=1  │
  │          │ crust_key=1│   "Classic" stored ONCE.   "Thin" stored ONCE.
  │          │ Small      │   No redundancy!           No redundancy!
  └──────────┴────────────┘

  But now to get the full pizza info, you JOIN dim_pizza → dim_category AND dim_pizza → dim_crust.
  That's MORE JOINs for every query.
```

#### Why Would You Choose Snowflake Over Star?

Honestly? **Most of the time, you wouldn't.** Star schemas are the default for good reason. But snowflake schemas have their place:

**Choose Snowflake when:**
- A dimension table is **very large** (millions of rows) and redundancy wastes significant storage
- Your warehouse tool handles JOINs efficiently (many modern cloud warehouses like Snowflake/BigQuery optimize this automatically)
- You need strict data governance — normalized sub-tables enforce consistency (e.g., you can't accidentally type "Clasic" instead of "Classic" because category names are only in one table)

**Choose Star when (the default ~90% of the time):**
- You want **simpler queries** — analysts don't have to think about joining sub-dimensions
- You want **faster queries** — fewer JOINs means less compute time
- Dimension tables are small enough that redundancy is trivial (storing "Classic" 200 times costs essentially nothing)
- You're building a data warehouse for a team of analysts who aren't SQL experts

**Side-by-side query comparison:**

```sql
-- STAR SCHEMA: "What are total sales by pizza category?"
-- Simple: two JOINs, all category info is in dim_pizza
SELECT p.category, SUM(f.total_amount) AS revenue
FROM fact_pizza_sales f
JOIN dim_pizza p ON f.pizza_key = p.pizza_key
GROUP BY p.category;

-- SNOWFLAKE SCHEMA: same question, but now category is in a sub-table
-- More complex: THREE JOINs required
SELECT c.category, SUM(f.total_amount) AS revenue
FROM fact_pizza_sales f
JOIN dim_pizza p    ON f.pizza_key = p.pizza_key
JOIN dim_category c ON p.cat_key   = c.cat_key        -- extra JOIN!
GROUP BY c.category;
```

Both queries give the same answer. But the star schema version is simpler. Multiply that by hundreds of analyst queries per day, and simplicity wins.

#### Complete Side-by-Side Summary

| Aspect | Star Schema | Snowflake Schema |
|--------|-------------|-----------------|
| **Dimension tables** | Denormalized — flat, wide tables with all attributes | Normalized — split into sub-tables connected by foreign keys |
| **Number of tables** | Fewer (1 fact + a few dimensions) | More (same fact + dimensions + sub-dimension tables) |
| **Redundancy** | Yes — "Classic" might appear 200 times in dim_pizza | No — "Classic" appears once in dim_category |
| **Query complexity** | Simpler — analysts JOIN fact to dimension, done | More complex — analysts must JOIN through sub-dimensions too |
| **Query speed** | Faster — fewer JOINs to execute | Slower — more JOINs adds compute time |
| **Storage** | More disk space (repeated text in dimensions) | Less disk space (no repeated text) |
| **Ease of use for analysts** | Easy — intuitive, flat tables | Harder — must understand the sub-table relationships |
| **Data governance** | Weaker — typos in repeated values possible | Stronger — canonical values in one location |
| **Best for** | 90% of analytics use cases | Very large dimensions or strict governance needs |

> **Interview advice:** Default to the star schema. If an interviewer asks about snowflake, explain that it normalizes the dimension tables to reduce redundancy but adds query complexity, and that you'd only do it for very large dimensions or strict data governance. That's a complete answer.

---

## 23.7 The Decision Framework: How to Design a Schema for Any Project

Now the real question: someone hands you a business scenario. How do you decide what schema to build?

### Step 1: Ask "What Is This Database FOR?"

```
  DECISION TREE:

  What will this database primarily do?
      │
      ├── Run a live application (process transactions, serve users)
      │   └── → OLTP → Normalized schema (3NF)
      │
      ├── Answer analytical questions (reports, dashboards, KPIs)
      │   └── → OLAP → Denormalized schema (Star/Snowflake)
      │
      └── Both (operational + analytics)
          └── → Build BOTH: OLTP source → ETL pipeline → OLAP warehouse
              (This is what most companies actually do)
```

### Step 2: Identify the Entities

Entities are the **nouns** in the business scenario. Ask: "What *things* does this business track?"

**Trick:** Listen for nouns in the problem statement.
- "Customers **place** orders for products" → Entities: Customer, Order, Product
- "Doctors **treat** patients at hospitals" → Entities: Doctor, Patient, Hospital, Treatment
- "Drivers **deliver** packages to addresses" → Entities: Driver, Package, Address, Delivery

### Step 3: Identify the Relationships

How do entities connect?
- Can a customer have many orders? → **One-to-Many (1:M)**
- Can a book have many authors AND an author write many books? → **Many-to-Many (M:N)** → needs a junction table
- Does each employee have exactly one badge? → **One-to-One (1:1)**

### Step 4: Define the Grain (for OLAP)

If you're building an analytical schema, define the grain before anything else:
- "What is the most atomic event we want to analyze?"
- One row per sale? Per day? Per customer-product combination?

### Step 5: Assign Primary Keys and Foreign Keys

- Every table needs a **primary key** (unique identifier for each row)
- Relationships are expressed through **foreign keys** (a column in one table pointing to the PK of another)

### Step 6: Normalize or Denormalize Based on Purpose

- OLTP → normalize to 3NF (eliminate redundancy)
- OLAP → denormalize into star schema (optimize for reads)

---

## 23.8 Worked Example #1: Hospital Patient System

Let's walk through the complete design process.

### The Scenario

> "A hospital needs a database to manage patient visits. Patients see doctors in specific departments. Each visit has a diagnosis, prescribed medications, and a billing amount."

### Step 1: What is this for?

This is an **operational system** — it runs the hospital day to day. Nurses input data, doctors look up records, billing processes charges. → **OLTP → Normalized (3NF)**

### Step 2: Identify entities

Reading the scenario for nouns: **Patient, Doctor, Department, Visit, Diagnosis, Medication, Bill**

### Step 3: Identify relationships

- A patient has many visits; a visit belongs to one patient → **1:M**
- A doctor has many visits; a visit is handled by one doctor → **1:M**
- A department has many doctors; a doctor belongs to one department → **1:M**
- A visit has one diagnosis (simplified) → **1:1** (embed in visit)
- A visit can involve multiple medications; a medication can be prescribed in many visits → **M:N** → junction table
- A visit has one bill → **1:1** (embed or separate table)

### Step 4: Primary and Foreign Keys

```sql
CREATE TABLE departments (
    department_id   INT PRIMARY KEY,
    name            VARCHAR(100) NOT NULL,    -- 'Cardiology', 'Orthopedics', etc.
    floor           INT,
    phone           VARCHAR(20)
);

CREATE TABLE doctors (
    doctor_id       INT PRIMARY KEY,
    name            VARCHAR(100) NOT NULL,
    specialty       VARCHAR(100),
    department_id   INT NOT NULL REFERENCES departments(department_id),
    hire_date       DATE
);

CREATE TABLE patients (
    patient_id      INT PRIMARY KEY,
    name            VARCHAR(100) NOT NULL,
    date_of_birth   DATE NOT NULL,
    gender          CHAR(1),
    phone           VARCHAR(20),
    insurance_id    VARCHAR(50)
);

CREATE TABLE visits (
    visit_id        INT PRIMARY KEY,
    patient_id      INT NOT NULL REFERENCES patients(patient_id),
    doctor_id       INT NOT NULL REFERENCES doctors(doctor_id),
    visit_date      TIMESTAMP NOT NULL,
    diagnosis       VARCHAR(500),           -- embedded (1:1 with visit)
    diagnosis_code  VARCHAR(10),            -- ICD-10 code
    billing_amount  DECIMAL(10,2),          -- embedded (1:1 with visit)
    notes           TEXT
);

CREATE TABLE medications (
    medication_id   INT PRIMARY KEY,
    name            VARCHAR(200) NOT NULL,
    dosage_form     VARCHAR(50),            -- 'tablet', 'injection', 'liquid'
    manufacturer    VARCHAR(200)
);

CREATE TABLE visit_medications (           -- junction table for M:N
    visit_id        INT REFERENCES visits(visit_id),
    medication_id   INT REFERENCES medications(medication_id),
    dosage          VARCHAR(100),           -- '500mg twice daily'
    duration_days   INT,
    PRIMARY KEY (visit_id, medication_id)
);
```

**Result: 6 tables, 3NF.** Each fact stored once. Patient name → patients table only. Doctor name → doctors table only. Department name → departments table only. If the cardiology department moves to floor 4, you update ONE row.

### Now: The OLAP Version

The hospital's analytics team wants a dashboard: "Show me visit volume, average billing, and medication costs by department, month, and patient demographics."

**Star schema design:**

```sql
CREATE TABLE fact_visits (
    visit_key        INT PRIMARY KEY,
    date_key         INT REFERENCES dim_date(date_key),
    patient_key      INT REFERENCES dim_patient(patient_key),
    doctor_key       INT REFERENCES dim_doctor(doctor_key),
    -- MEASURES:
    billing_amount   DECIMAL(10,2),
    medication_count INT,
    medication_cost  DECIMAL(10,2),
    length_of_stay   INT              -- minutes
);

CREATE TABLE dim_date (
    date_key         INT PRIMARY KEY,
    full_date        DATE,
    day_of_week      VARCHAR(10),
    month            INT,
    month_name       VARCHAR(10),
    quarter          INT,
    year             INT,
    is_weekend       BOOLEAN
);

CREATE TABLE dim_patient (
    patient_key      INT PRIMARY KEY,
    patient_id       INT,               -- from OLTP
    name             VARCHAR(100),
    age_group        VARCHAR(20),        -- '18-30', '31-45', '46-60', '60+'
    gender           CHAR(1),
    insurance_type   VARCHAR(50)         -- 'Private', 'Medicare', 'Medicaid', 'Uninsured'
);

CREATE TABLE dim_doctor (
    doctor_key       INT PRIMARY KEY,
    doctor_id        INT,               -- from OLTP
    name             VARCHAR(100),
    specialty        VARCHAR(100),
    department_name  VARCHAR(100),       -- DENORMALIZED from departments table
    department_floor INT                 -- DENORMALIZED from departments table
);
```

**Notice:** `department_name` and `department_floor` live directly in `dim_doctor`. In the OLTP version, they were in a separate `departments` table. Here, they're denormalized for fast querying — an analyst can filter by department WITHOUT a JOIN.

---

## 23.9 Worked Example #2: Ride-Sharing App (Uber-Style)

### The Scenario

> "Design a database for a ride-sharing service. Riders request rides, drivers accept them, rides have pickup/dropoff locations, fares, ratings, and payment methods."

### Step 1: What is this for?

This runs the live app — riders are requesting rides RIGHT NOW. → **OLTP → Normalized (3NF)**

### Step 2: Entities

Riders, Drivers, Rides, Locations (pickup/dropoff), Payments, Ratings

### Step 3: Relationships

- A rider has many rides → 1:M
- A driver has many rides → 1:M
- A ride has one payment → 1:1 (embed or separate)
- A ride has one rating (by rider) → 1:1

### Step 4: Design

```sql
CREATE TABLE riders (
    rider_id         INT PRIMARY KEY,
    name             VARCHAR(100) NOT NULL,
    email            VARCHAR(255) UNIQUE NOT NULL,
    phone            VARCHAR(20),
    created_at       TIMESTAMP DEFAULT NOW()
);

CREATE TABLE drivers (
    driver_id        INT PRIMARY KEY,
    name             VARCHAR(100) NOT NULL,
    email            VARCHAR(255) UNIQUE NOT NULL,
    phone            VARCHAR(20),
    license_plate    VARCHAR(20),
    vehicle_type     VARCHAR(50),         -- 'Sedan', 'SUV', 'Luxury'
    rating_avg       DECIMAL(3,2),        -- pre-computed for fast display
    status           VARCHAR(20),         -- 'available', 'on_trip', 'offline'
    created_at       TIMESTAMP DEFAULT NOW()
);

CREATE TABLE rides (
    ride_id          INT PRIMARY KEY,
    rider_id         INT NOT NULL REFERENCES riders(rider_id),
    driver_id        INT REFERENCES drivers(driver_id),  -- NULL until accepted
    status           VARCHAR(20) NOT NULL,  -- 'requested','accepted','in_progress','completed','cancelled'
    pickup_lat       DECIMAL(10,7),
    pickup_lng       DECIMAL(10,7),
    pickup_address   VARCHAR(300),
    dropoff_lat      DECIMAL(10,7),
    dropoff_lng      DECIMAL(10,7),
    dropoff_address  VARCHAR(300),
    requested_at     TIMESTAMP NOT NULL,
    accepted_at      TIMESTAMP,
    started_at       TIMESTAMP,
    completed_at     TIMESTAMP,
    distance_miles   DECIMAL(6,2),
    duration_minutes INT,
    fare_amount      DECIMAL(10,2),
    surge_multiplier DECIMAL(3,2) DEFAULT 1.00,
    payment_method   VARCHAR(20),          -- 'credit_card', 'debit', 'paypal'
    rating           INT CHECK (rating BETWEEN 1 AND 5),  -- rider's rating of driver
    rating_comment   TEXT
);
```

**Design decisions explained:**

1. **Why is `rating_avg` on the drivers table?** It's denormalized. Technically, you could calculate it from all ride ratings every time. But when a rider opens the app and sees "4.87 ★" next to a driver's name, that needs to load *instantly*. So we pre-compute it and store it on the driver. We update it after each ride. This is a deliberate denormalization in an OLTP system for performance.

2. **Why aren't pickup/dropoff in a separate locations table?** Because each ride's locations are unique — nobody else shares that exact pickup point. A separate table would add JOINs without reducing redundancy. If we had *named locations* (airports, stadiums) shared across rides, we'd normalize those into a separate table.

3. **Why is payment_method a column, not a separate table?** Simplicity. If we needed to store multiple payment methods per rider (like Venmo and 3 credit cards), we'd need a separate `payment_methods` table. But per-ride, there's exactly one payment method, so it embeds cleanly.

### Now: The OLAP Version

The product team asks: "Show us average fare by city, time of day, vehicle type, and whether surge pricing was active."

```sql
CREATE TABLE fact_rides (
    ride_key           INT PRIMARY KEY,
    date_key           INT REFERENCES dim_date(date_key),
    time_key           INT REFERENCES dim_time(time_key),
    rider_key          INT REFERENCES dim_rider(rider_key),
    driver_key         INT REFERENCES dim_driver(driver_key),
    geography_key      INT REFERENCES dim_geography(geography_key),
    -- MEASURES:
    fare_amount        DECIMAL(10,2),
    distance_miles     DECIMAL(6,2),
    duration_minutes   INT,
    surge_multiplier   DECIMAL(3,2),
    rating             INT,
    is_completed       BOOLEAN
);

CREATE TABLE dim_time (
    time_key           INT PRIMARY KEY,
    hour               INT,              -- 0-23
    minute             INT,
    time_of_day        VARCHAR(20),       -- 'Morning Rush', 'Midday', 'Evening Rush', 'Late Night'
    is_rush_hour       BOOLEAN
);

CREATE TABLE dim_geography (
    geography_key      INT PRIMARY KEY,
    city               VARCHAR(100),
    neighborhood       VARCHAR(100),
    zip_code           VARCHAR(10),
    state              VARCHAR(50),
    region             VARCHAR(50)        -- 'Northeast', 'West Coast', etc.
);

CREATE TABLE dim_driver (
    driver_key         INT PRIMARY KEY,
    driver_id          INT,
    name               VARCHAR(100),
    vehicle_type       VARCHAR(50),       -- denormalized
    tenure_months      INT,
    rating_tier        VARCHAR(20)        -- 'Gold (4.8+)', 'Silver (4.5-4.8)', 'Standard'
);
```

**Notice what changed:** In the OLTP schema, pickup_lat and pickup_lng were raw coordinates. In the OLAP schema, geography is a *dimension* with human-readable labels (city, neighborhood, region). This is because analysts don't ask "show me rides at latitude 30.2672"; they ask "show me rides in Austin, TX." The ETL pipeline geocodes the raw coordinates into the dimension values.

---

## 23.10 Worked Example #3: E-Commerce Analytics (OLAP Focus)

### The Scenario

> "A company sells products online. Management wants a data warehouse to analyze sales trends by product category, customer segment, time period, and sales channel (website vs. mobile app)."

### Step 1: What is this for?

This is explicitly for analytics → **OLAP → Star Schema**

### Step 2: Define the Grain FIRST

The most atomic event is: **one product sold in one order**. One row per line item.

> This is the most critical decision. If we set the grain to "one row per order," we lose the ability to analyze individual products within orders. If we set it to "one row per day," we lose order-level detail. Always choose the finest grain — you can always aggregate up, but you can never drill down past your grain.

### Step 3: Identify Measures (numbers to aggregate)

- quantity_sold
- unit_price
- discount_amount
- total_revenue (quantity × unit_price - discount)
- shipping_cost

### Step 4: Identify Dimensions (context for those numbers)

- **When?** → dim_date
- **What?** → dim_product (product name, category, subcategory, brand)
- **Who?** → dim_customer (name, segment, region, acquisition channel)
- **How?** → dim_channel (website, mobile app, phone order, marketplace)
- **Where?** → dim_geography (shipping destination)

### Step 5: Build the Schema

```sql
-- FACT TABLE: one row per product sold per order
CREATE TABLE fact_order_items (
    order_item_key   INT PRIMARY KEY,
    order_id         INT,                 -- degenerate dimension (no separate dim table)
    date_key         INT REFERENCES dim_date(date_key),
    product_key      INT REFERENCES dim_product(product_key),
    customer_key     INT REFERENCES dim_customer(customer_key),
    channel_key      INT REFERENCES dim_channel(channel_key),
    geography_key    INT REFERENCES dim_geography(geography_key),
    -- MEASURES:
    quantity         INT,
    unit_price       DECIMAL(10,2),
    discount_pct     DECIMAL(5,2),
    discount_amount  DECIMAL(10,2),
    net_revenue      DECIMAL(10,2),
    shipping_cost    DECIMAL(10,2),
    cost_of_goods    DECIMAL(10,2),       -- enables margin analysis
    profit           DECIMAL(10,2)        -- net_revenue - cost_of_goods
);

CREATE TABLE dim_product (
    product_key      INT PRIMARY KEY,
    product_id       VARCHAR(20),         -- original SKU from OLTP
    product_name     VARCHAR(200),
    category         VARCHAR(100),        -- 'Electronics', 'Clothing', 'Books'
    subcategory      VARCHAR(100),        -- 'Laptops', 'Smartphones', 'Accessories'
    brand            VARCHAR(100),
    supplier         VARCHAR(200),
    price_tier       VARCHAR(20),         -- 'Budget (<$25)', 'Mid ($25-100)', 'Premium ($100+)'
    is_active        BOOLEAN
);

CREATE TABLE dim_customer (
    customer_key     INT PRIMARY KEY,
    customer_id      INT,
    name             VARCHAR(100),
    email            VARCHAR(255),
    city             VARCHAR(100),
    state            VARCHAR(50),
    country          VARCHAR(50),
    signup_date      DATE,
    customer_segment VARCHAR(30),         -- 'New', 'Returning', 'VIP', 'At-Risk'
    lifetime_orders  INT,
    lifetime_revenue DECIMAL(12,2),
    acquisition_source VARCHAR(50)        -- 'Google Ads', 'Organic', 'Referral', 'Social'
);

CREATE TABLE dim_channel (
    channel_key      INT PRIMARY KEY,
    channel_name     VARCHAR(50),         -- 'Website', 'Mobile App', 'Phone', 'Marketplace'
    platform         VARCHAR(50),         -- 'iOS', 'Android', 'Web', 'Amazon'
    is_owned         BOOLEAN              -- Do we own this channel or is it 3rd party?
);

CREATE TABLE dim_date (
    date_key         INT PRIMARY KEY,     -- format: YYYYMMDD (e.g., 20250321)
    full_date        DATE,
    day_name         VARCHAR(10),
    day_of_month     INT,
    day_of_year      INT,
    week_of_year     INT,
    month            INT,
    month_name       VARCHAR(10),
    quarter          INT,
    year             INT,
    fiscal_quarter   INT,                 -- may differ from calendar quarter
    fiscal_year      INT,
    is_weekend       BOOLEAN,
    is_holiday       BOOLEAN,
    holiday_name     VARCHAR(50)
);

CREATE TABLE dim_geography (
    geography_key    INT PRIMARY KEY,
    zip_code         VARCHAR(10),
    city             VARCHAR(100),
    state            VARCHAR(50),
    country          VARCHAR(50),
    region           VARCHAR(50)          -- 'Northeast', 'West Coast', 'EU', 'APAC'
);
```

**Now the powerful queries become simple:**

```sql
-- Total revenue by product category and quarter
SELECT p.category, d.quarter, d.year, SUM(f.net_revenue) AS total_revenue
FROM fact_order_items f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY p.category, d.quarter, d.year
ORDER BY d.year, d.quarter;

-- Top 10 customers by lifetime spend, filtered to VIPs
SELECT c.name, c.customer_segment, SUM(f.net_revenue) AS total_spend
FROM fact_order_items f
JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE c.customer_segment = 'VIP'
GROUP BY c.name, c.customer_segment
ORDER BY total_spend DESC
LIMIT 10;

-- Revenue by sales channel, comparing mobile vs. web
SELECT ch.channel_name, SUM(f.net_revenue) AS revenue, COUNT(*) AS items_sold
FROM fact_order_items f
JOIN dim_channel ch ON f.channel_key = ch.channel_key
GROUP BY ch.channel_name;
```

**Each query: 1-2 JOINs, clean and readable.** In a normalized OLTP schema, these queries would require 5-8 JOINs and be much harder to write and understand.

---

## 23.11 The Complete Decision Cheat Sheet

When you're in an interview or on a project and someone says "design a schema for X," use this:

```
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │                      SCHEMA DESIGN DECISION FRAMEWORK                      │
  ├─────────────────────────────────────────────────────────────────────────────┤
  │                                                                             │
  │  1. WHAT IS THE PRIMARY USE CASE?                                          │
  │     • Transactional (app, service) → OLTP → Normalize (3NF)               │
  │     • Analytical (reports, BI)     → OLAP → Denormalize (Star Schema)     │
  │     • Both                         → Build both, connect with ETL         │
  │                                                                             │
  │  2. WHAT ARE THE ENTITIES? (Nouns in the problem)                          │
  │     • List every thing the business tracks                                 │
  │     • Each entity usually becomes a table                                  │
  │                                                                             │
  │  3. WHAT ARE THE RELATIONSHIPS?                                            │
  │     • 1:1 → embed in same table or separate                              │
  │     • 1:M → FK on the "many" side                                        │
  │     • M:N → junction table with two FKs                                   │
  │                                                                             │
  │  4. FOR OLAP: WHAT IS THE GRAIN?                                          │
  │     • Define what ONE ROW represents                                       │
  │     • Choose the finest grain you'll need                                  │
  │                                                                             │
  │  5. ASSIGN KEYS                                                            │
  │     • Every table gets a PK                                               │
  │     • Relationships → FK on the child/dependent side                      │
  │     • OLAP: use surrogate integer keys (not business keys)                │
  │                                                                             │
  │  6. NORMALIZE / DENORMALIZE                                                │
  │     • OLTP → split until 3NF (no redundancy)                             │
  │     • OLAP → flatten dimensions (embed related attributes)               │
  │                                                                             │
  │  7. ADD CONSTRAINTS                                                        │
  │     • NOT NULL for required fields                                        │
  │     • UNIQUE for natural uniqueness (email, ISBN)                         │
  │     • CHECK for value validation (rating BETWEEN 1 AND 5)                 │
  │     • DEFAULT for sensible defaults (created_at = NOW())                  │
  │                                                                             │
  └─────────────────────────────────────────────────────────────────────────────┘
```

---

## 23.12 Common Mistakes (and How to Avoid Them)

| Mistake | Why It's Bad | Fix |
|---------|-------------|-----|
| **No primary key** | Can't uniquely identify rows; duplicates creep in | Every table gets a PK, always |
| **Storing lists in a column** | Violates 1NF; can't query, index, or validate | Use a separate table with one row per item |
| **Using the wrong grain** | "One row per order" loses product-level detail | Always define grain explicitly and pick the finest level you need |
| **Normalizing an OLAP schema** | Forces analysts to write 8-table JOINs for basic questions | Use star schema for analytics |
| **Denormalizing an OLTP schema** | Update anomalies — change a customer name, miss 5 rows | Keep OLTP at 3NF (small denormalizations OK for hot paths) |
| **Skipping foreign keys** | Orphaned records (orders pointing to deleted customers) | Always define FK constraints |
| **VARCHAR(MAX) for everything** | Wastes storage, prevents optimization, no validation | Size your VARCHARs realistically |
| **No timestamps** | Can't audit, can't debug, can't replicate | Add created_at and updated_at to every table |

---

## 23.13 How This Connects to Palantir

Every concept in this chapter maps directly to what you'll do as a Deployment Strategist:

| Schema Design Concept | Palantir Equivalent |
|----------------------|---------------------|
| Entities (tables) | Object Types in the Ontology |
| Columns (attributes) | Properties on Object Types |
| Foreign keys (relationships) | Links between Object Types |
| Grain (what a row represents) | The semantic meaning of an Object |
| Star schema fact table | The operational event you're tracking in Foundry |
| Dimension tables | The contextual Object Types that enrich the fact |
| ETL from OLTP → OLAP | Foundry's Data Connection → Transforms → Ontology pipeline |
| Denormalization for speed | Pipeline Builder materializing data for Workshop apps |

When a client says "we use Oracle for our ERP" — that's an OLTP system with a normalized schema. Your job is to connect Foundry to it, understand the schema, and build an Ontology (which is essentially a purpose-built analytical layer) on top of it.

> **Interview application:** If asked "Design a data model for tracking hospital readmissions" — you now have the framework. You'd identify entities (Patient, Admission, Hospital, Diagnosis), define relationships, choose grain, and sketch either a normalized OLTP schema or a star-schema OLAP model based on the use case. You'd articulate *why* you made each choice.

---

## 23.14 Chapter Summary

| Concept | One-Line Definition |
|---------|-------------------|
| **Schema** | The blueprint defining a database's tables, columns, types, and relationships |
| **OLTP** | Operational database optimized for fast transactions (normalized, 3NF) |
| **OLAP** | Analytical database optimized for complex queries (denormalized, star schema) |
| **Normalization** | Eliminating redundancy by splitting data into focused tables |
| **1NF** | No lists in cells — one value per cell |
| **2NF** | No partial key dependencies |
| **3NF** | No transitive dependencies (non-key → non-key) |
| **Denormalization** | Intentionally adding redundancy for read speed |
| **Star Schema** | Central fact table surrounded by dimension tables |
| **Snowflake Schema** | Star schema with normalized (split) dimension tables |
| **Fact Table** | Stores measurable events (the numbers you aggregate) |
| **Dimension Table** | Stores context (who, what, where, when) around facts |
| **Grain** | What one row in a fact table represents — define it FIRST |
| **Foreign Key** | A column pointing to a primary key in another table, enforcing relationships |
| **Junction Table** | Resolves many-to-many relationships with two foreign keys |

---

*You started this chapter not knowing what a schema is. You now know how to design one for any scenario — operational or analytical — and, more importantly, you know WHY you'd design it that way. That "why" is what interviewers want to hear.*
