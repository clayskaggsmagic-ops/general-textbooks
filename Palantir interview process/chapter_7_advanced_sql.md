# Chapter 7: Advanced SQL — Window Functions, Optimization, and Interview Patterns

---

## 7.1 Why This Matters for a Deployment Strategist

Chapter 6 gave you the fundamentals: joins, aggregations, subqueries, CTEs. This chapter is what separates a competent SQL user from someone who can genuinely debug production pipelines and impress in a technical interview.

Three reasons this material is critical:

1. **Window functions** are the single most powerful feature in modern SQL — they let you compute rankings, running totals, and comparisons *without collapsing rows*, which is exactly what analysts need when building dashboards and reports in Foundry
2. **Query optimization** knowledge lets you diagnose why a client's 3-hour overnight ETL is slow and propose fixes — a high-visibility win for any DS
3. **Interview patterns** are the actual question archetypes you'll encounter — deduplication, top-N per group, gap detection — and knowing the templates cold lets you focus on problem-solving rather than syntax under pressure

> **Interview tip:** Palantir's SQL questions tend toward medium complexity. If you can do ROW_NUMBER deduplication, a multi-join aggregation with HAVING, and explain a basic EXPLAIN plan, you're well-prepared.

---

## 7.2 Window Functions — The OVER() Clause

### 7.2.1 The Core Concept

A window function performs a calculation across a *set of rows related to the current row* — but unlike GROUP BY, **it does not collapse the result set**. Every row stays; the window function adds a new column.

```
GROUP BY: Many rows → One row per group (aggregation collapses)
Window:   Many rows → Same rows, with new computed column added
```

**Syntax:**

```sql
function_name(arguments) OVER (
    [PARTITION BY partition_columns]
    [ORDER BY sort_columns]
    [frame_clause]
)
```

### 7.2.2 PARTITION BY

`PARTITION BY` divides the data into independent groups (partitions). The window function resets for each partition.

```sql
-- Without PARTITION BY → entire result set is one window
SUM(amount) OVER ()  -- total of ALL rows

-- With PARTITION BY → each department is its own window
SUM(amount) OVER (PARTITION BY department)  -- total per department
```

Think of `PARTITION BY` as GROUP BY's cousin: it creates groups, but doesn't eliminate rows.

### 7.2.3 ORDER BY Within Windows

`ORDER BY` inside the window defines the logical sequence for the computation. This determines what "previous," "next," "running total," and "rank" mean.

```sql
-- Running total: order matters — sum accumulates row by row
SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date)

-- Without ORDER BY, the sum would be the full partition total on every row
```

### 7.2.4 Frame Specifications

The **frame** defines exactly which rows the window function considers relative to the current row. This is the most nuanced part of window functions.

**Syntax:**

```sql
{ ROWS | RANGE } BETWEEN frame_start AND frame_end
```

**Frame Boundaries:**

| Boundary | Meaning |
|---|---|
| `UNBOUNDED PRECEDING` | First row of the partition |
| `n PRECEDING` | n rows (ROWS) or n value-units (RANGE) before current |
| `CURRENT ROW` | The current row |
| `n FOLLOWING` | n rows or value-units after current |
| `UNBOUNDED FOLLOWING` | Last row of the partition |

**ROWS vs. RANGE:**

| | ROWS | RANGE |
|---|---|---|
| **Operates on** | Physical row positions | Logical value ranges |
| **Tie handling** | Each tied row gets its own frame position | All rows with same ORDER BY value share position |
| **Use when** | You want exactly N rows | You want all rows within a value range |

**Default Frames (critical to know):**

| Situation | Default Frame |
|---|---|
| `OVER ()` — no ORDER BY | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (entire partition) |
| `OVER (ORDER BY col)` — with ORDER BY | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (start to current, including ties) |

> **Trap:** The default frame with ORDER BY is `RANGE ... CURRENT ROW`, which includes ties. This means a "running total" may appear to "jump" when there are duplicate ORDER BY values. Always use `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` if you want strict row-by-row accumulation.

---

## 7.3 Ranking Functions

All ranking functions require `ORDER BY` in the window clause.

### 7.3.1 Comparison Table

Given employees ordered by salary within each department (ties at $90K):

| Employee | Salary | ROW_NUMBER() | RANK() | DENSE_RANK() | NTILE(3) |
|---|---|---|---|---|---|
| Alice | 120K | 1 | 1 | 1 | 1 |
| Bob | 100K | 2 | 2 | 2 | 1 |
| Carol | 90K | 3 | 3 | 3 | 2 |
| Dave | 90K | 4 | 3 | 3 | 2 |
| Eve | 80K | 5 | 5 | 4 | 3 |
| Frank | 70K | 6 | 6 | 5 | 3 |

### 7.3.2 Function Details

| Function | Ties | Gaps | Use Case |
|---|---|---|---|
| `ROW_NUMBER()` | Assigns different numbers to ties (arbitrary order) | Never | **Deduplication**, pagination, unique numbering |
| `RANK()` | Same rank for ties | Yes — skips numbers after ties | Standard competition ranking (1st, 2nd, 2nd, 4th) |
| `DENSE_RANK()` | Same rank for ties | No gaps — consecutive ranks | When you need "the top 3 distinct salary levels" |
| `NTILE(n)` | N/A — divides into n buckets | N/A | Percentile buckets, quartile analysis |

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS drnk,
    NTILE(4)     OVER (PARTITION BY department ORDER BY salary DESC) AS quartile
FROM employees;
```

### 7.3.3 When to Use Which

| I need... | Use |
|---|---|
| Exactly one row per group (dedup) | `ROW_NUMBER()` |
| "Top 3 performers" (allow ties at the boundary) | `RANK()` or `DENSE_RANK()` |
| "Top 3 salary levels" (distinct values) | `DENSE_RANK()` |
| Split data into equal buckets | `NTILE(n)` |

---

## 7.4 Aggregate Window Functions

Any standard aggregate can become a window function by adding `OVER()`:

### 7.4.1 Running Total

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;

-- Result:
-- order_date | amount | running_total
-- 2025-01-01 |    100 |           100
-- 2025-01-02 |    250 |           350
-- 2025-01-03 |    180 |           530
-- 2025-01-04 |    320 |           850
```

### 7.4.2 Moving Average

```sql
-- 7-day moving average of daily revenue
SELECT
    order_date,
    daily_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7d
FROM daily_sales;
```

### 7.4.3 Percent of Total

```sql
-- Each order's percentage of its department's total
SELECT
    department,
    order_id,
    amount,
    SUM(amount) OVER (PARTITION BY department) AS dept_total,
    ROUND(
        amount * 100.0 / SUM(amount) OVER (PARTITION BY department),
        1
    ) AS pct_of_dept
FROM orders;
```

### 7.4.4 Partition-Level Aggregates Without GROUP BY

```sql
-- Show each employee alongside their department average
SELECT
    name,
    department,
    salary,
    AVG(salary)   OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg,
    COUNT(*)      OVER (PARTITION BY department) AS dept_size,
    MIN(salary)   OVER (PARTITION BY department) AS dept_min,
    MAX(salary)   OVER (PARTITION BY department) AS dept_max
FROM employees;
-- Every row is preserved — no GROUP BY needed
```

---

## 7.5 Offset Functions — LAG, LEAD, FIRST_VALUE, LAST_VALUE, NTH_VALUE

### 7.5.1 LAG and LEAD

| Function | What It Does | Syntax |
|---|---|---|
| `LAG(col, n, default)` | Gets value from `n` rows **before** current row | n defaults to 1; default returned for first row(s) |
| `LEAD(col, n, default)` | Gets value from `n` rows **after** current row | n defaults to 1; default returned for last row(s) |

```sql
-- Month-over-month revenue comparison
SELECT
    month,
    revenue,
    LAG(revenue, 1)  OVER (ORDER BY month) AS prev_month,
    LEAD(revenue, 1) OVER (ORDER BY month) AS next_month,
    revenue - LAG(revenue, 1) OVER (ORDER BY month) AS mom_change,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (ORDER BY month))
        * 100.0 / NULLIF(LAG(revenue, 1) OVER (ORDER BY month), 0),
        1
    ) AS mom_pct_change
FROM monthly_revenue;

-- Result:
-- month   | revenue | prev_month | next_month | mom_change | mom_pct_change
-- 2025-01 |  50000  |     NULL   |    62000   |     NULL   |      NULL
-- 2025-02 |  62000  |    50000   |    58000   |    12000   |      24.0
-- 2025-03 |  58000  |    62000   |    71000   |    -4000   |      -6.5
-- 2025-04 |  71000  |    58000   |      NULL  |    13000   |      22.4
```

### 7.5.2 FIRST_VALUE, LAST_VALUE, NTH_VALUE

| Function | Returns | Frame Caveat |
|---|---|---|
| `FIRST_VALUE(col)` | Value from the first row in the window frame | Usually works as expected |
| `LAST_VALUE(col)` | Value from the last row in the window frame | **Trap:** default frame ends at CURRENT ROW, not the partition end! |
| `NTH_VALUE(col, n)` | Value from the nth row in the window frame | Returns NULL if n exceeds frame size |

```sql
-- Compare each employee's salary to the highest and lowest in their department
SELECT
    name,
    department,
    salary,
    FIRST_VALUE(name) OVER (
        PARTITION BY department ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS highest_paid_employee,
    LAST_VALUE(name) OVER (
        PARTITION BY department ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        -- ^ Must specify full frame! Default would stop at CURRENT ROW
    ) AS lowest_paid_employee
FROM employees;
```

> **Critical:** Always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when using `LAST_VALUE()`. The default frame ends at the current row, so `LAST_VALUE()` will just return the current row's value!

---

## 7.6 Window Functions vs. GROUP BY — The Decision Framework

| | GROUP BY | Window Function |
|---|---|---|
| **Row Preservation** | ❌ Collapses to one row per group | ✅ Keeps all individual rows |
| **Use When** | You want *only* the summary | You want the summary *alongside* individual details |
| **Can Filter Groups?** | Yes, via HAVING | No (use surrounding CTE/subquery to filter) |
| **Multiple Aggregations** | All share the same grouping | Each can have its own PARTITION/ORDER |
| **Performance** | Often faster for pure aggregation | Can be slower — more data to return |

**Rule of Thumb:**
- "I need a report with one row per category" → **GROUP BY**
- "I need each row enriched with category-level stats" → **Window function**
- "I need each row's rank within its group" → **Window function** (can't do this with GROUP BY)

---

## 7.7 Indexing Concepts

Indexes are lookup data structures that the database maintains alongside the table data. They speed up reads at the cost of write overhead.

### 7.7.1 Index Types

| Index Type | Structure | Best For | Not Suited For |
|---|---|---|---|
| **B-tree** (default) | Balanced tree, sorted keys | `=`, `<`, `>`, `<=`, `>=`, `BETWEEN`, `LIKE 'prefix%'`, ORDER BY, GROUP BY | `LIKE '%suffix'` (can't use sorted order) |
| **Hash** | Hash table | Equality (`=`) only — O(1) lookup | Range queries, sorting, pattern matching. Many DBs convert to B-tree internally |
| **GIN** | Generalized Inverted Index | Full-text search, JSONB containment, array operations | Simple scalar comparisons |
| **GiST** | Generalized Search Tree | Geometric/spatial data, range types, full-text search | Simple equality/range on scalars |
| **BRIN** | Block Range Index | Very large, naturally ordered tables (timestamps, sequential IDs) | Small tables, randomly ordered data |

### 7.7.2 Composite (Multi-Column) Indexes

**Column order matters.** The leftmost prefix rule governs which queries can use the index:

```sql
CREATE INDEX idx_orders_cust_date ON orders (customer_id, order_date);

-- ✅ Uses the index (matches leftmost prefix):
WHERE customer_id = 100
WHERE customer_id = 100 AND order_date > '2025-01-01'

-- ❌ Cannot use this index (skips the leftmost column):
WHERE order_date > '2025-01-01'
-- (Needs a separate index on order_date alone)
```

**Column ordering heuristic:** Put the most **selective** (highest cardinality) column first, unless your queries always filter on a specific column.

### 7.7.3 Covering Indexes

A covering index includes *all columns* needed by a query, so the database never touches the table — it reads everything from the index (an "index-only scan"):

```sql
-- Query:
SELECT customer_id, order_date, amount
FROM orders
WHERE customer_id = 100 AND order_date > '2025-01-01';

-- Covering index (includes all selected columns):
CREATE INDEX idx_orders_covering
ON orders (customer_id, order_date)
INCLUDE (amount);
-- PostgreSQL syntax: INCLUDE adds payload columns to the index leaf
```

### 7.7.4 Partial (Filtered) Indexes

Index only a subset of rows — smaller, faster, cheaper:

```sql
-- Only index active orders (90% of queries filter to active)
CREATE INDEX idx_active_orders
ON orders (customer_id, order_date)
WHERE status = 'active';

-- Only index non-NULL emails (most are NULL)
CREATE INDEX idx_user_emails
ON users (email)
WHERE email IS NOT NULL;
```

### 7.7.5 When to Index — and When NOT To

| ✅ Index When | ❌ Don't Index When |
|---|---|
| Column is used frequently in WHERE, JOIN, ORDER BY | Table is small (<1,000 rows) — sequential scan is faster |
| Column has high selectivity (many distinct values) | Column has low selectivity (few distinct values, e.g., `gender`) |
| Read-heavy workload | Write-heavy workload — every INSERT/UPDATE/DELETE updates the index |
| Foreign key columns (join performance) | Column is rarely queried |
| Large tables where full scans are expensive | You already have too many indexes (maintenance overhead) |

### 7.7.6 The Read/Write Trade-Off

```
┌──────────────────────────────────────────────────┐
│  INDEXES: THE FUNDAMENTAL TRADE-OFF              │
│                                                    │
│  More indexes → Faster reads, slower writes        │
│  Fewer indexes → Slower reads, faster writes       │
│                                                    │
│  Every INSERT: must update ALL indexes on table    │
│  Every UPDATE: must update indexes on changed cols │
│  Every DELETE: must update ALL indexes on table    │
│                                                    │
│  Rule: Index what you query. Don't index           │
│  everything "just in case."                        │
└──────────────────────────────────────────────────┘
```

---

## 7.8 Query Optimization — Reading EXPLAIN Plans

### 7.8.1 EXPLAIN vs. EXPLAIN ANALYZE

| | EXPLAIN | EXPLAIN ANALYZE |
|---|---|---|
| **Runs the query?** | ❌ No — estimates only | ✅ Yes — executes and measures |
| **Shows** | Estimated cost, rows, width | Estimated + actual time, rows, loops |
| **Safe for** | Any query (read-only analysis) | SELECT queries (be careful with INSERT/UPDATE/DELETE — they execute!) |

```sql
EXPLAIN ANALYZE
SELECT customer_id, COUNT(*)
FROM orders
WHERE order_date >= '2025-01-01'
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

### 7.8.2 Reading the Output

```
                                                 QUERY PLAN
───────────────────────────────────────────────────────────────────────────────────
 HashAggregate  (cost=245.00..247.50 rows=200 width=12)
                (actual time=3.215..3.312 rows=47 loops=1)
   Group Key: customer_id
   Filter: (count(*) > 5)
   Rows Removed by Filter: 153
   -> Bitmap Heap Scan on orders  (cost=12.45..230.00 rows=1200 width=4)
                                  (actual time=0.215..2.113 rows=1247 loops=1)
        Recheck Cond: (order_date >= '2025-01-01')
        Heap Blocks: exact=89
        -> Bitmap Index Scan on idx_orders_date  (cost=0.00..12.15 rows=1200 width=0)
                                                 (actual time=0.142..0.142 rows=1247 loops=1)
              Index Cond: (order_date >= '2025-01-01')
 Planning Time: 0.189 ms
 Execution Time: 3.412 ms
```

**Key metrics to look at:**

| Metric | What It Means | What to Watch For |
|---|---|---|
| **cost** (startup..total) | Planner's estimated cost in arbitrary units | High cost → expensive operation |
| **rows** | Estimated row count | Compare to actual rows — big discrepancy means stale statistics |
| **actual time** | Real execution time (ms) | The actual bottleneck |
| **actual rows** | Real row count | If `rows` says 10 but `actual rows` is 100,000 → run `ANALYZE` |
| **loops** | Times this node was executed | Nested loops execute the inner side many times |
| **width** | Average row width in bytes | Wide rows = more I/O |

### 7.8.3 Scan Types

| Scan Type | How It Works | When Used |
|---|---|---|
| **Sequential Scan** (Seq Scan) | Reads every page of the table | No useful index; large portion of table needed (>5-10%); small table |
| **Index Scan** | Traverse B-tree → fetch rows from table (heap) by TID | Small number of rows match; index exists on filtered column |
| **Index-Only Scan** | Read everything from the index — never touch the table | All required columns are in the index (covering index); visibility map is current |
| **Bitmap Index Scan → Bitmap Heap Scan** | Build bitmap of matching pages → read those pages sequentially | Moderate selectivity (1-20% of rows); combines multiple indexes with BitmapAnd/BitmapOr |

**Scan Type Decision Spectrum:**

```
Few rows matched ◀─────────────────────────────────▶ Many rows matched

Index Scan    Index-Only Scan    Bitmap Scan          Sequential Scan
(< 1-5%)      (covering idx)    (1-20%)              (> 10-20%)
```

### 7.8.4 Optimization Checklist

When a query is slow, work through this checklist:

1. **Run EXPLAIN ANALYZE** — find the node with the highest `actual time`
2. **Check estimated vs. actual rows** — large discrepancy → run `ANALYZE tablename`
3. **Look for Seq Scans on large tables** — may need an index
4. **Check for Nested Loop Joins** — acceptable for small inner tables, bad for large ones
5. **Look for Sort nodes** — if not needed, the query might benefit from an index matching the ORDER BY
6. **Check for Filter:** removing many rows → the WHERE could use an index instead
7. **Run `EXPLAIN (ANALYZE, BUFFERS)`** — see I/O details (shared hits vs. reads)

---

## 7.9 Practical Interview Patterns

These are the actual archetypes that appear in SQL interviews. Learn the pattern, not just the query.

### 7.9.1 Pattern: Deduplication (Keep Latest)

**Problem:** Table has duplicate rows; keep only the most recent per entity.

```sql
-- Keep only the latest address per customer
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC
        ) AS rn
    FROM customer_addresses
)
SELECT * FROM ranked WHERE rn = 1;
```

**Why ROW_NUMBER?** It guarantees exactly one row per partition (unlike RANK, which keeps ties).

### 7.9.2 Pattern: Top-N Per Group

**Problem:** Find the top 3 products by revenue in each category.

```sql
WITH ranked_products AS (
    SELECT
        category,
        product_name,
        revenue,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue DESC
        ) AS rn
    FROM product_sales
)
SELECT category, product_name, revenue
FROM ranked_products
WHERE rn <= 3
ORDER BY category, rn;
```

**Variant:** Use `RANK()` if you want to include ties at the boundary (e.g., two products tied for 3rd place).

### 7.9.3 Pattern: Running Totals

**Problem:** Show cumulative revenue over time.

```sql
SELECT
    order_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_revenue
FROM daily_sales;
```

**Variant — Running total per partition:**

```sql
SUM(daily_revenue) OVER (
    PARTITION BY region
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

### 7.9.4 Pattern: Gap Analysis (Finding Missing Sequences)

**Problem:** Find gaps in an invoice number sequence.

```sql
SELECT
    invoice_number,
    LEAD(invoice_number) OVER (ORDER BY invoice_number) AS next_invoice,
    LEAD(invoice_number) OVER (ORDER BY invoice_number) - invoice_number AS gap_size
FROM invoices
WHERE LEAD(invoice_number) OVER (ORDER BY invoice_number) - invoice_number > 1;

-- More correct: use a CTE since you can't use window functions in WHERE
WITH sequenced AS (
    SELECT
        invoice_number,
        LEAD(invoice_number) OVER (ORDER BY invoice_number) AS next_invoice
    FROM invoices
)
SELECT
    invoice_number AS gap_starts_after,
    next_invoice AS gap_ends_before,
    next_invoice - invoice_number - 1 AS missing_count
FROM sequenced
WHERE next_invoice - invoice_number > 1;
```

### 7.9.5 Pattern: Sessionization

**Problem:** Group user events into sessions where a new session starts after 30 minutes of inactivity.

```sql
WITH events_with_gap AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS prev_event_time,
        CASE
            WHEN event_time - LAG(event_time) OVER (
                PARTITION BY user_id ORDER BY event_time
            ) > INTERVAL '30 minutes'
            OR LAG(event_time) OVER (
                PARTITION BY user_id ORDER BY event_time
            ) IS NULL
            THEN 1
            ELSE 0
        END AS is_new_session
    FROM user_events
),
sessions AS (
    SELECT
        *,
        SUM(is_new_session) OVER (
            PARTITION BY user_id ORDER BY event_time
        ) AS session_id
    FROM events_with_gap
)
SELECT
    user_id,
    session_id,
    MIN(event_time) AS session_start,
    MAX(event_time) AS session_end,
    COUNT(*)        AS event_count,
    MAX(event_time) - MIN(event_time) AS session_duration
FROM sessions
GROUP BY user_id, session_id
ORDER BY user_id, session_start;
```

**How it works:**
1. `LAG()` gets the previous event time per user
2. Mark a new session if the gap exceeds 30 minutes (or it's the first event)
3. `SUM()` over the flags creates an incrementing session counter
4. GROUP BY session to get session-level metrics

### 7.9.6 Pattern: Pivot (Rows → Columns)

**Problem:** Convert monthly revenue rows into a single row with columns for each month.

```sql
-- Pivot using CASE + aggregate
SELECT
    product,
    SUM(CASE WHEN month = 'Jan' THEN revenue ELSE 0 END) AS jan_revenue,
    SUM(CASE WHEN month = 'Feb' THEN revenue ELSE 0 END) AS feb_revenue,
    SUM(CASE WHEN month = 'Mar' THEN revenue ELSE 0 END) AS mar_revenue,
    SUM(CASE WHEN month = 'Apr' THEN revenue ELSE 0 END) AS apr_revenue
FROM monthly_sales
GROUP BY product;
```

**Unpivot (Columns → Rows) using LATERAL:**

```sql
-- PostgreSQL LATERAL approach
SELECT
    product,
    month_name,
    revenue
FROM quarterly_report,
LATERAL (VALUES
    ('Jan', jan_revenue),
    ('Feb', feb_revenue),
    ('Mar', mar_revenue)
) AS unpivoted(month_name, revenue);
```

### 7.9.7 Pattern: Percent of Total

**Problem:** Show each row's percentage contribution to the group total.

```sql
SELECT
    region,
    product,
    revenue,
    SUM(revenue) OVER (PARTITION BY region) AS region_total,
    ROUND(revenue * 100.0 / SUM(revenue) OVER (PARTITION BY region), 1) AS pct_of_region,
    SUM(revenue) OVER () AS grand_total,
    ROUND(revenue * 100.0 / SUM(revenue) OVER (), 1) AS pct_of_grand
FROM product_sales
ORDER BY region, revenue DESC;
```

### 7.9.8 Pattern: Year-over-Year Comparison

**Problem:** Compare each month's revenue to the same month last year.

```sql
-- Method 1: Using LAG with 12-month offset
WITH monthly AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    WHERE status != 'cancelled'
    GROUP BY 1
)
SELECT
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS prev_year_revenue,
    ROUND(
        (revenue - LAG(revenue, 12) OVER (ORDER BY month))
        * 100.0 / NULLIF(LAG(revenue, 12) OVER (ORDER BY month), 0),
        1
    ) AS yoy_growth_pct
FROM monthly
ORDER BY month;

-- Method 2: Self-join on month number (from Chapter 6)
WITH monthly AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        EXTRACT(YEAR FROM order_date) AS year,
        EXTRACT(MONTH FROM order_date) AS month_num,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY 1, 2, 3
)
SELECT
    curr.month,
    curr.revenue AS current_year,
    prev.revenue AS previous_year,
    ROUND(
        (curr.revenue - prev.revenue)
        * 100.0 / NULLIF(prev.revenue, 0),
        1
    ) AS yoy_growth_pct
FROM monthly curr
LEFT JOIN monthly prev
    ON curr.month_num = prev.month_num
    AND curr.year = prev.year + 1
WHERE curr.year = 2025
ORDER BY curr.month;
```

---

## 7.10 Performance Anti-Patterns

These are the things that make queries slow. Recognizing them is a valuable skill in any DS role.

### 7.10.1 The Big Five Anti-Patterns

| # | Anti-Pattern | Why It's Bad | Fix |
|---|---|---|---|
| 1 | **SELECT *** | Fetches all columns → more I/O, more network, prevents index-only scans | List only the columns you need |
| 2 | **Functions on indexed columns in WHERE** | `WHERE YEAR(order_date) = 2025` → index on `order_date` becomes useless (must evaluate function on every row) | Rewrite: `WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01'` |
| 3 | **Implicit type conversions** | `WHERE varchar_col = 12345` → the DB casts the column, not the literal → index can't be used | Ensure types match: `WHERE varchar_col = '12345'` |
| 4 | **Correlated subqueries on large tables** | Re-executes inner query for every outer row → O(N×M) | Rewrite as JOIN or use EXISTS/CTE |
| 5 | **Missing indexes on join/filter columns** | Forces sequential scan on large tables | Add indexes on FK columns, WHERE predicates, JOIN columns |

### 7.10.2 Anti-Pattern Deep Dives

**Functions in WHERE (Index Killer):**

```sql
-- ❌ BAD: function prevents index use
WHERE UPPER(email) = 'ALICE@EXAMPLE.COM'
WHERE EXTRACT(YEAR FROM order_date) = 2025
WHERE CAST(user_id AS VARCHAR) = '12345'

-- ✅ GOOD: keep the column "naked"
WHERE email = 'alice@example.com'  -- use case-insensitive collation or ILIKE
WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01'
WHERE user_id = 12345

-- ✅ ALTERNATIVE: create a function-based index (PostgreSQL)
CREATE INDEX idx_email_upper ON users (UPPER(email));
-- Now UPPER(email) = 'VALUE' CAN use the index
```

**Implicit Type Conversion:**

```sql
-- ❌ BAD: comparing VARCHAR to INT
-- The DB must CAST the entire column → full scan
SELECT * FROM products
WHERE product_code = 12345;  -- product_code is VARCHAR

-- ✅ GOOD: match the type
SELECT * FROM products
WHERE product_code = '12345';
```

**N+1 Query Pattern:**

```sql
-- ❌ BAD: application loops, one query per customer
-- Python pseudocode:
-- for customer in customers:
--     orders = db.query("SELECT * FROM orders WHERE customer_id = ?", customer.id)

-- ✅ GOOD: single JOIN
SELECT c.*, o.*
FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

### 7.10.3 Quick Performance Checklist

```
┌──────────────────────────────────────────────────┐
│  PERFORMANCE CHECKLIST                            │
│                                                    │
│  □ SELECT only needed columns (never *)            │
│  □ WHERE clause: columns are "naked" (no functions)│
│  □ JOIN columns: all indexed                       │
│  □ JOIN types: confirm they match intent           │
│      (INNER vs LEFT — wrong join = wrong results)  │
│  □ Subqueries: uncorrelated or rewritten as JOIN   │
│  □ Data types: no implicit conversions             │
│  □ EXPLAIN ANALYZE: estimated ≈ actual rows?       │
│  □ Statistics current? (ANALYZE table recently?)   │
│  □ UNION ALL instead of UNION (when dedup not      │
│      needed)                                        │
│  □ LIMIT: present for exploratory queries          │
└──────────────────────────────────────────────────┘
```

---

## 7.11 Named Window Definitions (WINDOW Clause)

When you reuse the same window definition multiple times, name it once:

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER()  OVER w AS rn,
    RANK()        OVER w AS rnk,
    DENSE_RANK()  OVER w AS drnk,
    AVG(salary)   OVER w AS dept_avg,
    salary - AVG(salary) OVER w AS diff_from_avg
FROM employees
WINDOW w AS (PARTITION BY department ORDER BY salary DESC);
-- ^ Define once, use many times
```

This is cleaner, less error-prone, and signals to the optimizer that the same sort/partition can be reused.

---

## 7.12 Glossary

| Term | Definition |
|---|---|
| **B-tree index** | Balanced tree structure for sorted key lookups. Default index type. Supports equality, range, and ordering |
| **Bitmap scan** | Two-phase scan that builds a page bitmap from indexes, then reads matching pages sequentially. Middle ground between index and sequential scan |
| **Composite index** | Index on multiple columns. Leftmost prefix rule determines query eligibility |
| **Covering index** | Index containing all columns a query needs, enabling index-only scans without touching the table |
| **DENSE_RANK()** | Assigns consecutive ranks with no gaps; tied rows share the same rank |
| **EXPLAIN ANALYZE** | Runs a query and reports estimated vs. actual execution metrics for each plan node |
| **FIRST_VALUE()** | Returns the first value in the window frame |
| **Frame clause** | ROWS BETWEEN or RANGE BETWEEN — specifies which rows relative to current row the window function considers |
| **Hash index** | Hash-table-based index for equality lookups only. O(1) for exact matches |
| **Index-only scan** | Reading all needed data from the index without touching the table |
| **LAG()** | Offset function returning a value from N rows before the current row |
| **LAST_VALUE()** | Returns the last value in the window frame. Requires explicit frame specification to work intuitively |
| **LEAD()** | Offset function returning a value from N rows after the current row |
| **NTILE(n)** | Distributes rows into n approximately equal-sized buckets |
| **NTH_VALUE()** | Returns the value from the Nth row in the window frame |
| **OVER()** | Clause that turns an aggregate or ranking function into a window function |
| **Partial index** | Index built on a subset of rows matching a WHERE condition. Smaller and faster |
| **PARTITION BY** | Divides the result set into independent groups for window function computation |
| **RANK()** | Assigns ranking with gaps after ties (1, 2, 2, 4) |
| **ROW_NUMBER()** | Assigns unique sequential integers; no ties, no gaps |
| **Sequential scan** | Reads every page of a table. Used when no suitable index exists or when a large fraction of rows are needed |
| **Sessionization** | Grouping sequential events into logical sessions based on time gaps |
| **Window function** | A function that computes a value across a set of rows related to the current row without collapsing the result set |

---

## 7.13 Chapter Summary — The "Cheat Sheet"

```
┌───────────────────────────────────────────────────────────────┐
│  CHAPTER 7 SUMMARY CARD                                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  WINDOW FUNCTIONS:                                            │
│  • OVER() turns aggregate/ranking into window function        │
│  • PARTITION BY = GROUP BY without collapsing rows            │
│  • ORDER BY = defines sequence for computation                │
│  • Frame default WITH ORDER BY:                               │
│    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW          │
│    (includes ties! Use ROWS for strict row-by-row)            │
│                                                               │
│  RANKING:                                                     │
│  • ROW_NUMBER: unique, no gaps → deduplication                │
│  • RANK: ties share rank, gaps after → competition ranking    │
│  • DENSE_RANK: ties share rank, no gaps → "3 tiers"          │
│  • NTILE(n): split into n equal buckets                       │
│                                                               │
│  OFFSET:                                                      │
│  • LAG(col, n): look back n rows                              │
│  • LEAD(col, n): look forward n rows                          │
│  • FIRST_VALUE / LAST_VALUE: first/last in frame              │
│    (LAST_VALUE needs explicit UNBOUNDED FOLLOWING!)            │
│                                                               │
│  INDEXES:                                                     │
│  • B-tree: default, range + equality + ORDER BY               │
│  • Composite: leftmost prefix rule, column order matters      │
│  • Covering: all cols in index → index-only scan              │
│  • Partial: WHERE on index → smaller, faster                  │
│  • Trade-off: more indexes = faster reads, slower writes      │
│                                                               │
│  EXPLAIN ANALYZE:                                             │
│  • Cost (startup..total), Rows, Width = estimates             │
│  • Actual time, Actual rows = reality                         │
│  • Big discrepancy = stale stats → run ANALYZE                │
│  • Seq Scan on big table + few rows = missing index           │
│                                                               │
│  INTERVIEW PATTERNS (memorize these templates):               │
│  1. Dedup: ROW_NUMBER() PARTITION BY key ORDER BY ts → rn=1   │
│  2. Top-N: ROW_NUMBER/RANK PARTITION BY group → rn<=N         │
│  3. Running total: SUM() OVER (ORDER BY date ROWS...)         │
│  4. Gaps: LEAD(val) - val > 1                                 │
│  5. Sessions: LAG → flag gaps > 30min → SUM flags             │
│  6. Pivot: CASE in aggregate + GROUP BY                       │
│  7. % of total: val / SUM(val) OVER (PARTITION BY group)      │
│  8. YoY: LAG(val, 12) or self-join on month_num              │
│                                                               │
│  ANTI-PATTERNS:                                               │
│  • SELECT * → list specific columns                           │
│  • Functions in WHERE → keep column "naked"                   │
│  • Type mismatch → match literal type to column type          │
│  • Correlated subquery → rewrite as JOIN                      │
│  • Missing index on JOIN/WHERE cols → add targeted indexes    │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 7.14 What's Next

In **Chapter 8: Data Structures I — Fundamentals**, we shift from databases to computer science theory. We'll cover arrays, linked lists, stacks, queues, hash maps, and trees — the building blocks of every system you'll encounter as a Deployment Strategist, and essential knowledge for Palantir's technical interview.

---

*"The difference between a junior and a senior is that the senior can read an EXPLAIN plan and knows when to reach for ROW_NUMBER()."*
