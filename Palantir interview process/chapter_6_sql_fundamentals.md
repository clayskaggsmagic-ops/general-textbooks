# Chapter 6: SQL Fundamentals — The Core Query Language

---

## 6.1 Why This Matters for a Deployment Strategist

SQL is the **lingua franca of data**. Every system in Chapters 4 and 5 — PostgreSQL, Oracle, Snowflake, BigQuery, Redshift, Databricks — speaks SQL. The Palantir interview explicitly tests aggregate functions and joins, but a Deployment Strategist needs SQL fluency far beyond interview day:

- **Pipeline debugging** — tracing data quality issues back to source queries
- **Client conversations** — understanding how analysts and engineers describe their data
- **Ontology mapping** — SQL is often the first language used to explore datasets before modeling them in the Ontology
- **Code Workbook / Pipeline Builder** — Foundry's transformation tools are SQL-compatible

This chapter covers the foundations. Chapter 7 builds on this with window functions, advanced aggregation, and query optimization.

> **Interview tip:** The Palantir technical screen will test your ability to write GROUP BY queries with HAVING, multi-table JOINs, and possibly subqueries — all under time pressure. Master this chapter cold.

---

## 6.2 Core Statement Structure

### 6.2.1 The SELECT Statement

Every SQL query follows the same basic anatomy:

```sql
SELECT   columns / expressions / aggregates
FROM     table(s)
WHERE    row-level filter (before grouping)
GROUP BY grouping columns
HAVING   group-level filter (after grouping)
ORDER BY sort columns
LIMIT    max rows returned
```

### 6.2.2 Logical Execution Order

**This is the single most important concept for understanding SQL behavior.** The logical execution order is *different* from the written order:

```
Written Order              Logical Execution Order
─────────────              ───────────────────────
1. SELECT                  1. FROM / JOIN
2. FROM                    2. WHERE
3. WHERE                   3. GROUP BY
4. GROUP BY                4. HAVING
5. HAVING                  5. SELECT
6. ORDER BY                6. ORDER BY
7. LIMIT                   7. LIMIT
```

**Why this matters — three concrete consequences:**

| Consequence | Explanation | Example |
|---|---|---|
| **Can't use SELECT aliases in WHERE** | WHERE executes *before* SELECT, so aliases don't exist yet | `WHERE total_amount > 100` ✅ not `WHERE amt AS total_amount > 100` ❌ |
| **Can use SELECT aliases in ORDER BY** | ORDER BY executes *after* SELECT | `ORDER BY total_amount` ✅ (where `total_amount` is a SELECT alias) |
| **Can't use aggregates in WHERE** | Aggregation happens in GROUP BY/SELECT *after* WHERE | Use `HAVING COUNT(*) > 5` not `WHERE COUNT(*) > 5` |

### 6.2.3 The Execution Order Walkthrough

Consider this query step by step:

```sql
SELECT     department, COUNT(*) AS headcount, AVG(salary) AS avg_salary
FROM       employees e
JOIN       departments d ON e.dept_id = d.id
WHERE      e.hire_date >= '2020-01-01'
GROUP BY   department
HAVING     COUNT(*) >= 5
ORDER BY   avg_salary DESC
LIMIT      10;
```

| Step | Clause | What Happens |
|---|---|---|
| 1 | `FROM / JOIN` | Combine `employees` and `departments` into one virtual table |
| 2 | `WHERE` | Filter to employees hired on or after 2020-01-01 |
| 3 | `GROUP BY` | Collapse rows into groups by `department` |
| 4 | `HAVING` | Keep only groups with 5+ employees |
| 5 | `SELECT` | Compute `COUNT(*)` and `AVG(salary)` for each group; assign aliases |
| 6 | `ORDER BY` | Sort by `avg_salary` descending |
| 7 | `LIMIT` | Return only the top 10 rows |

---

## 6.3 Joins in Depth

Joins combine rows from two or more tables based on a related column. Understanding when to use each type is critical.

### 6.3.1 Join Types

```
         TABLE A                    TABLE B
    ┌──────────────┐           ┌──────────────┐
    │ id │  name    │           │ id │ dept     │
    │────│──────────│           │────│──────────│
    │  1 │ Alice    │           │  1 │ Eng      │
    │  2 │ Bob      │           │  3 │ Sales    │
    │  4 │ Carol    │           │  5 │ HR       │
    └──────────────┘           └──────────────┘
```

| Join Type | Result | Rows Returned (above) |
|---|---|---|
| **INNER JOIN** | Only matching rows from both tables | `(1, Alice, Eng)` |
| **LEFT (OUTER) JOIN** | All rows from A + matching from B (NULLs if no match) | `(1, Alice, Eng), (2, Bob, NULL), (4, Carol, NULL)` |
| **RIGHT (OUTER) JOIN** | All rows from B + matching from A (NULLs if no match) | `(1, Alice, Eng), (3, NULL, Sales), (5, NULL, HR)` |
| **FULL (OUTER) JOIN** | All rows from both (NULLs where no match) | `(1, Alice, Eng), (2, Bob, NULL), (4, Carol, NULL), (3, NULL, Sales), (5, NULL, HR)` |
| **CROSS JOIN** | Cartesian product — every row × every row | 3 × 3 = 9 rows |
| **SELF JOIN** | A table joined to itself (requires aliases) | Depends on condition |

### 6.3.2 Visualizing Joins (Venn Diagram Model)

```
    INNER JOIN            LEFT JOIN            RIGHT JOIN          FULL OUTER JOIN
  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │ A    ████  B │    │████████  B   │    │  A    ████████│    │████████████████│
  │      ████    │    │████████      │    │       ████████│    │████████████████│
  └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
    Only overlap       All of A +           All of B +          Everything
                         overlap               overlap
```

> **Caveat:** Venn diagrams are a useful mental model but technically imprecise — they suggest set-based operations, while JOINs operate on row-level conditions. They still help in interviews.

### 6.3.3 When to Use Each Join

| Scenario | Join Type | Example |
|---|---|---|
| "Show me orders WITH their customer info" | INNER JOIN | Only orders that have a matching customer |
| "Show ALL customers, even those with no orders" | LEFT JOIN | Customers without orders get NULL in order columns |
| "Find customers who have NEVER ordered" | LEFT JOIN + WHERE IS NULL | `LEFT JOIN orders ON ... WHERE orders.id IS NULL` |
| "Combine two complete datasets, keeping everything" | FULL OUTER JOIN | Merging data from two systems |
| "Generate all possible pairings" | CROSS JOIN | Scheduling (every team vs. every team) |
| "Find employees who report to other employees" | SELF JOIN | `employees e1 JOIN employees e2 ON e1.manager_id = e2.id` |

### 6.3.4 Self Join Example

```sql
-- Find each employee and their manager's name
SELECT
    e.name    AS employee,
    m.name    AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
-- LEFT JOIN ensures the CEO (manager_id IS NULL) is included
```

### 6.3.5 Join Algorithms

The database **query optimizer** chooses the join algorithm automatically. Understanding the options helps you diagnose slow queries:

| Algorithm | How It Works | Time Complexity | Best For |
|---|---|---|---|
| **Nested Loop** | For each row in outer table, scan inner table for matches | O(N × M) worst case. O(N log M) with index | Small tables, or inner table has an index on join column. Only option for non-equality joins (`>`, `<`, `BETWEEN`) |
| **Hash Join** | Build a hash table on the smaller table (build phase), then probe it with the larger table (probe phase) | O(N + M) ideal | Large unsorted tables with equality joins (`=`). No useful index. Smaller table fits in memory |
| **Merge Join** | Sort both tables on join key, then merge in a single pass | O(N + M) if pre-sorted. O(N log N + M log M) if sorting needed | Both tables already sorted on join key (e.g., via index). Large datasets |

**Practical rules:**
- **Index your join columns** — this is the #1 performance lever for joins
- **Smaller table as "build side"** — hash joins are faster when the hash table fits in memory
- **Pre-sorted data** → merge join is almost free
- The optimizer usually makes the right choice — focus on indexing and data design, not forcing algorithms

---

## 6.4 Aggregate Functions

Aggregate functions collapse multiple rows into a single summary value.

### 6.4.1 The Core Five

| Function | What It Does | NULL Handling |
|---|---|---|
| `COUNT(*)` | Counts all rows, including NULLs | **Includes** NULL rows |
| `COUNT(column)` | Counts non-NULL values in column | **Skips** NULLs |
| `COUNT(DISTINCT column)` | Counts unique non-NULL values | **Skips** NULLs, deduplicates |
| `SUM(column)` | Sum of non-NULL values | **Skips** NULLs. Returns NULL if all values are NULL |
| `AVG(column)` | Average of non-NULL values | **Skips** NULLs (divides by count of non-NULLs, not total rows!) |
| `MIN(column)` | Smallest non-NULL value | **Skips** NULLs |
| `MAX(column)` | Largest non-NULL value | **Skips** NULLs |

### 6.4.2 COUNT Variants — The Trap

This is a **common interview question**:

```sql
-- Given this table:
-- scores: | student | grade |
--         | Alice   |  90   |
--         | Bob     | NULL  |
--         | Carol   |  85   |
--         | Dave    |  90   |

SELECT
    COUNT(*)              AS total_rows,      -- 4
    COUNT(grade)          AS graded_students, -- 3 (skips Bob's NULL)
    COUNT(DISTINCT grade) AS unique_grades    -- 2 (90 and 85)
FROM scores;
```

### 6.4.3 AVG and NULLs — The Classic Pitfall

```sql
-- values: 10, NULL, 20, NULL, 30
SELECT AVG(value) FROM data;
-- Returns 20.0 (= (10 + 20 + 30) / 3)
-- NOT 12.0 (= 60 / 5)
-- AVG divides by the count of non-NULL values, not total rows!

-- If you want NULLs treated as 0:
SELECT AVG(COALESCE(value, 0)) FROM data;
-- Returns 12.0 (= (10 + 0 + 20 + 0 + 30) / 5)
```

### 6.4.4 GROUP BY with Multiple Columns

GROUP BY creates one group for each **unique combination** of the grouped columns:

```sql
-- How many orders per customer per status?
SELECT
    customer_id,
    status,
    COUNT(*)        AS order_count,
    SUM(amount)     AS total_amount
FROM orders
GROUP BY customer_id, status
ORDER BY customer_id, status;

-- Result:
-- customer_id | status    | order_count | total_amount
-- 1001        | delivered |           3 |       450.00
-- 1001        | pending   |           1 |       120.00
-- 1002        | delivered |           5 |       890.00
```

**Rule:** Every non-aggregated column in SELECT must appear in GROUP BY.

### 6.4.5 HAVING vs. WHERE

| | WHERE | HAVING |
|---|---|---|
| **When it runs** | Before GROUP BY (step 2) | After GROUP BY (step 4) |
| **Filters** | Individual rows | Groups (aggregate results) |
| **Can use aggregates?** | ❌ No | ✅ Yes |
| **Can use aliases?** | ❌ No | ❌ No (in most databases) |

```sql
-- WHERE: filter rows before grouping
--   "Only include orders from 2024"
-- HAVING: filter groups after grouping
--   "Only show departments with 10+ orders"

SELECT
    department,
    COUNT(*) AS order_count,
    SUM(amount) AS total
FROM orders
WHERE order_date >= '2024-01-01'     -- row filter (before grouping)
GROUP BY department
HAVING COUNT(*) >= 10                -- group filter (after grouping)
ORDER BY total DESC;
```

---

## 6.5 Subqueries

A subquery is a query nested inside another query. They can appear in `WHERE`, `FROM`, and `SELECT` clauses.

### 6.5.1 Four Types of Subqueries

| Type | Returns | Where Used | Example |
|---|---|---|---|
| **Scalar** | Single value (1 row, 1 column) | SELECT, WHERE, HAVING | `WHERE salary > (SELECT AVG(salary) FROM employees)` |
| **Column** | Single column, multiple rows | WHERE with IN, ANY, ALL | `WHERE dept_id IN (SELECT id FROM departments WHERE region = 'West')` |
| **Row** | Single row, multiple columns | WHERE with row constructors | `WHERE (dept_id, title) = (SELECT dept_id, title FROM ...)` |
| **Table** | Multiple rows and columns (derived table) | FROM clause | `FROM (SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id) AS dept_avg` |

### 6.5.2 Correlated vs. Uncorrelated

| | Uncorrelated | Correlated |
|---|---|---|
| **Dependency** | Independent — can run on its own | References outer query columns |
| **Execution** | Runs once, result cached | Re-executes for every outer row |
| **Performance** | Generally faster | Can be slow (O(N × subquery)) |
| **Rewritable?** | Usually fine as-is | Often rewritable as JOIN |

```sql
-- UNCORRELATED: subquery runs once
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- CORRELATED: subquery runs once per outer row
SELECT e.name, e.salary, e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id  -- references outer query
);
```

### 6.5.3 EXISTS vs. IN

| | IN | EXISTS |
|---|---|---|
| **Mechanism** | Builds a list of values, then checks membership | Checks for existence (returns TRUE on first match) |
| **Short-circuits?** | ❌ No — evaluates entire list | ✅ Yes — stops at first match |
| **NULL handling** | **Dangerous** — `NOT IN` fails silently if list contains NULL | Safe — handles NULLs correctly |
| **Best for** | Small, known value lists | Large subqueries, correlated checks |

**The NOT IN + NULL Trap** (very important):

```sql
-- This query returns ZERO rows if the subquery returns even ONE NULL:
SELECT name FROM employees
WHERE department_id NOT IN (SELECT dept_id FROM closed_departments);
-- If closed_departments.dept_id contains a NULL, the entire
-- NOT IN comparison becomes UNKNOWN for every row → nothing returned!

-- Safe alternative:
SELECT name FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM closed_departments cd
    WHERE cd.dept_id = e.department_id
);
```

> **Interview tip:** If asked about `IN` vs `EXISTS`, always mention the NULL trap with `NOT IN`. This demonstrates deep understanding.

---

## 6.6 Common Table Expressions (CTEs)

### 6.6.1 Basic Syntax

A CTE defines a temporary, named result set within a single query:

```sql
WITH active_customers AS (
    SELECT customer_id, name, email
    FROM customers
    WHERE status = 'active'
      AND last_order_date >= CURRENT_DATE - INTERVAL '90 days'
),
customer_orders AS (
    SELECT
        ac.customer_id,
        ac.name,
        COUNT(o.id) AS order_count,
        SUM(o.amount) AS total_spent
    FROM active_customers ac
    JOIN orders o ON ac.customer_id = o.customer_id
    GROUP BY ac.customer_id, ac.name
)
SELECT * FROM customer_orders
WHERE total_spent > 1000
ORDER BY total_spent DESC;
```

### 6.6.2 Why CTEs Over Subqueries?

| Benefit | Explanation |
|---|---|
| **Readability** | Name each step — reads top-to-bottom like a story |
| **Reusability** | Reference the same CTE multiple times in one query |
| **Debuggability** | Test each CTE independently by running it alone |
| **Recursion** | Only CTEs support recursive queries |

### 6.6.3 Recursive CTEs

Recursive CTEs traverse hierarchical data — org charts, category trees, bill-of-materials.

**Structure:**

```sql
WITH RECURSIVE cte_name AS (
    -- ANCHOR MEMBER: base case (non-recursive)
    SELECT ...
    
    UNION ALL
    
    -- RECURSIVE MEMBER: references cte_name (the recursion)
    SELECT ...
    FROM cte_name
    JOIN ...
)
SELECT * FROM cte_name;
```

**Org Chart Example:**

```sql
-- employees table: id | name    | manager_id
--                  1  | Alice   | NULL (CEO)
--                  2  | Bob     | 1
--                  3  | Carol   | 1
--                  4  | Dave    | 2
--                  5  | Eve     | 2

WITH RECURSIVE org_chart AS (
    -- Anchor: start with the CEO (no manager)
    SELECT
        id,
        name,
        manager_id,
        0 AS depth,
        name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: find direct reports of the current level
    SELECT
        e.id,
        e.name,
        e.manager_id,
        oc.depth + 1,
        oc.path || ' → ' || e.name
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT
    REPEAT('  ', depth) || name AS employee,
    depth,
    path
FROM org_chart
ORDER BY path;

-- Result:
-- employee      | depth | path
-- Alice         |     0 | Alice
--   Bob         |     1 | Alice → Bob
--     Dave      |     2 | Alice → Bob → Dave
--     Eve       |     2 | Alice → Bob → Eve
--   Carol       |     1 | Alice → Carol
```

### 6.6.4 CTE vs. Subquery vs. Temp Table

| | CTE | Subquery | Temp Table |
|---|---|---|---|
| **Scope** | Single query only | Single location in query | Entire session |
| **Materialized?** | Usually not (optimizer inlines it) | Not materialized | Physically stored on disk/memory |
| **Recursive?** | ✅ Yes | ❌ No | ❌ No |
| **Reusable in query?** | ✅ Yes (by name, multiple times) | ❌ No (must duplicate) | ✅ Yes (across queries) |
| **Indexable?** | ❌ No | ❌ No | ✅ Yes |
| **Best for** | Readable, medium-complexity queries | Simple, one-off nested logic | Large intermediate results, cross-query reuse, when indexing helps |

---

## 6.7 Set Operations

Set operations combine the results of two or more `SELECT` statements.

### 6.7.1 Operations

| Operation | What It Does | Duplicates? |
|---|---|---|
| `UNION` | Combine results, remove duplicates | No duplicates |
| `UNION ALL` | Combine results, keep duplicates | Keeps duplicates |
| `INTERSECT` | Rows present in both results | No duplicates |
| `EXCEPT` (or `MINUS`) | Rows in first result but not in second | No duplicates |

**Requirements:** Both SELECTs must have the **same number of columns** with **compatible data types**.

### 6.7.2 UNION vs. UNION ALL

```sql
-- UNION removes duplicates (requires sort/hash — slower)
SELECT city FROM customers
UNION
SELECT city FROM suppliers;

-- UNION ALL keeps everything (no dedup — faster)
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
```

> **Performance tip:** Always use `UNION ALL` unless you specifically need deduplication. `UNION` performs an implicit `DISTINCT`, which requires sorting or hashing the entire result set.

### 6.7.3 EXCEPT for "Not In" Logic

```sql
-- Find products that have never been ordered
SELECT product_id FROM products
EXCEPT
SELECT product_id FROM order_items;
-- Returns product_ids that exist in products but NOT in order_items
```

---

## 6.8 Filtering and Expressions

### 6.8.1 CASE Expressions

CASE is SQL's `if/else` construct. Two forms:

**Simple CASE** — equality tests only:

```sql
SELECT
    name,
    status,
    CASE status
        WHEN 'A' THEN 'Active'
        WHEN 'I' THEN 'Inactive'
        WHEN 'P' THEN 'Pending'
        ELSE 'Unknown'
    END AS status_label
FROM users;
```

**Searched CASE** — any boolean expression:

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 150000 THEN 'Senior'
        WHEN salary >= 100000 THEN 'Mid'
        WHEN salary >= 60000  THEN 'Junior'
        ELSE 'Entry'
    END AS band
FROM employees;
```

**CASE in aggregations** (pivot-style patterns):

```sql
-- Count orders by status in a single row per customer
SELECT
    customer_id,
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) AS delivered,
    COUNT(CASE WHEN status = 'pending'   THEN 1 END) AS pending,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled
FROM orders
GROUP BY customer_id;
```

### 6.8.2 COALESCE and NULLIF

| Function | What It Does | Example |
|---|---|---|
| `COALESCE(a, b, c, ...)` | Returns the first non-NULL argument | `COALESCE(nickname, first_name, 'Unknown')` |
| `NULLIF(a, b)` | Returns NULL if a = b, otherwise returns a | `NULLIF(denominator, 0)` — prevents division by zero |

```sql
-- Avoid division by zero:
SELECT
    total_revenue / NULLIF(total_orders, 0) AS avg_order_value
FROM summary;
-- If total_orders = 0, NULLIF returns NULL, and anything / NULL = NULL
-- Without NULLIF: division by zero error
```

### 6.8.3 Other Filtering Operators

| Operator | Usage | Notes |
|---|---|---|
| `BETWEEN x AND y` | Range inclusive | Equivalent to `>= x AND <= y` |
| `IN (a, b, c)` | Membership in list | Beware NULLs with NOT IN |
| `LIKE` | Pattern matching | `%` = any chars, `_` = single char |
| `ILIKE` | Case-insensitive LIKE | PostgreSQL-specific |
| `IS NULL` / `IS NOT NULL` | NULL test | **Never** use `= NULL` |

---

## 6.9 String, Date, and Numeric Functions

### 6.9.1 String Functions

| Function | What It Does | Example | Result |
|---|---|---|---|
| `CONCAT(a, b)` or `a \|\| b` | Concatenate strings | `CONCAT('Hello', ' ', 'World')` | `'Hello World'` |
| `SUBSTRING(s, start, len)` | Extract substring | `SUBSTRING('Palantir', 1, 3)` | `'Pal'` |
| `TRIM(s)` | Remove leading/trailing whitespace | `TRIM('  hello  ')` | `'hello'` |
| `LTRIM(s)` / `RTRIM(s)` | Trim left / right only | `LTRIM('  hello')` | `'hello'` |
| `UPPER(s)` / `LOWER(s)` | Change case | `UPPER('hello')` | `'HELLO'` |
| `REPLACE(s, old, new)` | Replace all occurrences | `REPLACE('2024-01-15', '-', '/')` | `'2024/01/15'` |
| `LENGTH(s)` or `LEN(s)` | String length | `LENGTH('Palantir')` | `8` |
| `LEFT(s, n)` / `RIGHT(s, n)` | First/last n characters | `LEFT('Palantir', 3)` | `'Pal'` |
| `POSITION(sub IN s)` | Find position of substring | `POSITION('ant' IN 'Palantir')` | `4` |

### 6.9.2 Date/Time Functions

| Function | What It Does | Example |
|---|---|---|
| `CURRENT_DATE` | Today's date | `2025-03-20` |
| `CURRENT_TIMESTAMP` / `NOW()` | Current date + time | `2025-03-20 14:30:00` |
| `EXTRACT(part FROM date)` | Pull out year, month, day, hour, etc. | `EXTRACT(YEAR FROM order_date)` → `2025` |
| `DATE_TRUNC(unit, date)` | Truncate to unit boundary | `DATE_TRUNC('month', '2025-03-20')` → `2025-03-01` |
| `DATE_PART(part, date)` | Same as EXTRACT (PostgreSQL) | `DATE_PART('dow', order_date)` → day of week |
| `AGE(end, start)` | Interval between two timestamps | `AGE('2025-03-20', '2020-01-01')` → `5 years 2 mons 19 days` |

### 6.9.3 Date Arithmetic

```sql
-- Add/subtract intervals
SELECT order_date + INTERVAL '30 days' AS due_date
FROM orders;

SELECT CURRENT_DATE - INTERVAL '7 days' AS one_week_ago;

-- Days between two dates
SELECT order_date, ship_date,
       ship_date - order_date AS days_to_ship
FROM orders;

-- Group by month
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

### 6.9.4 Timezone Handling

```sql
-- TIMESTAMP vs. TIMESTAMPTZ (PostgreSQL)
-- TIMESTAMP: stores without timezone (naive)
-- TIMESTAMPTZ: stores in UTC, converts on display

-- Convert between timezones
SELECT
    created_at AT TIME ZONE 'UTC'              AS utc_time,
    created_at AT TIME ZONE 'America/New_York'  AS eastern_time,
    created_at AT TIME ZONE 'Asia/Tokyo'        AS tokyo_time
FROM events;
```

> **Best practice:** Always store timestamps as `TIMESTAMPTZ` (timestamp with timezone). Store in UTC, convert on display. This prevents timezone-related bugs that are incredibly difficult to debug in production.

### 6.9.5 Type Casting

```sql
-- CAST syntax (ANSI standard)
SELECT CAST('42' AS INTEGER);
SELECT CAST(price AS DECIMAL(10,2));

-- :: shorthand (PostgreSQL)
SELECT '42'::INTEGER;
SELECT price::DECIMAL(10,2);

-- Common casts
SELECT
    CAST(order_date AS DATE),      -- timestamp → date
    CAST(amount AS VARCHAR(20)),    -- number → string
    CAST('2025-03-20' AS DATE)     -- string → date
FROM orders;
```

---

## 6.10 Data Integrity Constraints

Constraints are **rules enforced by the database** to guarantee data validity. They are the last line of defense against bad data.

### 6.10.1 Constraint Types

| Constraint | What It Enforces | Example |
|---|---|---|
| `PRIMARY KEY` | Uniqueness + NOT NULL. One per table. Auto-indexed | `employee_id INT PRIMARY KEY` |
| `FOREIGN KEY` | Referential integrity — value must exist in parent table | `FOREIGN KEY (dept_id) REFERENCES departments(id)` |
| `UNIQUE` | All values in column(s) must be distinct. Allows NULLs | `email VARCHAR(255) UNIQUE` |
| `CHECK` | Custom boolean condition must be true | `CHECK (age >= 18 AND age <= 150)` |
| `NOT NULL` | Column cannot contain NULL | `name VARCHAR(100) NOT NULL` |
| `DEFAULT` | Provide a value when none is specified | `status VARCHAR(20) DEFAULT 'active'` |

### 6.10.2 Full Example

```sql
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,                           -- PK: unique + not null
    customer_id INT NOT NULL,                                 -- required
    status      VARCHAR(20) DEFAULT 'pending'                 -- default value
                CHECK (status IN ('pending','shipped','delivered','cancelled')),
    amount      DECIMAL(12,2) NOT NULL CHECK (amount > 0),    -- must be positive
    email       VARCHAR(255),                                 -- optional
    order_date  DATE NOT NULL DEFAULT CURRENT_DATE,
    
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE,
    
    UNIQUE (customer_id, order_date)  -- composite uniqueness
);
```

### 6.10.3 Referential Actions: CASCADE vs. RESTRICT

When a referenced parent row is deleted or updated, what happens to the child rows?

| Action | ON DELETE | ON UPDATE |
|---|---|---|
| **CASCADE** | Delete all child rows automatically | Update all child FK values automatically |
| **RESTRICT** | Block the delete if child rows exist | Block the update if child rows exist |
| **SET NULL** | Set child FK to NULL | Set child FK to NULL |
| **SET DEFAULT** | Set child FK to its DEFAULT value | Set child FK to its DEFAULT value |
| **NO ACTION** | Similar to RESTRICT (check timing differs) | Similar to RESTRICT |

```sql
-- CASCADE: deleting a customer deletes all their orders
FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE CASCADE;

-- RESTRICT: can't delete a customer who has orders
FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE RESTRICT;
```

**When to use which:**
- **CASCADE** — parent → child ownership (deleting a blog post deletes its comments)
- **RESTRICT** — critical data that should never be accidentally orphaned (deleting a customer should fail if they have invoices)

### 6.10.4 Why Constraints Matter

| Without Constraints | With Constraints |
|---|---|
| Orphaned orders pointing to deleted customers | FK prevents orphans |
| Duplicate email addresses | UNIQUE prevents duplicates |
| Negative prices | CHECK prevents impossible values |
| Missing required fields | NOT NULL ensures completeness |
| Application bugs bypass validation | Database enforces rules regardless of application layer |

> **DS relevance:** When building Foundry pipelines from client databases, the presence (or absence) of constraints tells you a lot about data quality. A database with no foreign keys often means you'll find orphaned records, duplicate data, and missing relationships — exactly the problems Foundry is built to solve.

---

## 6.11 NULL Semantics

NULLs are the most error-prone aspect of SQL. Master this section and you'll avoid bugs that trip up most developers.

### 6.11.1 What NULL Is (and Isn't)

```
NULL ≠ 0
NULL ≠ ''  (empty string)
NULL ≠ FALSE

NULL = "I don't know" / "Not applicable" / "Missing"
```

### 6.11.2 Three-Valued Logic

Every boolean expression in SQL evaluates to one of three states: `TRUE`, `FALSE`, or `UNKNOWN`.

Any comparison involving NULL produces `UNKNOWN`:

```sql
NULL = NULL     → UNKNOWN  (not TRUE!)
NULL != NULL    → UNKNOWN  (not TRUE!)
NULL > 5        → UNKNOWN
NULL = 0        → UNKNOWN
5 = NULL        → UNKNOWN
```

**Truth Tables:**

```
AND Truth Table          OR Truth Table           NOT Truth Table
─────────────────        ─────────────────        ─────────────
         T    F    U              T    F    U          NOT
  T │    T    F    U      T │    T    T    T       T │  F
  F │    F    F    F      F │    T    F    U       F │  T
  U │    U    F    U      U │    T    U    U       U │  U
```

**Key insight:** `WHERE` only keeps rows where the condition is `TRUE`. Rows where the condition is `FALSE` or `UNKNOWN` are filtered out. This means:

```sql
-- This EXCLUDES rows where status IS NULL:
WHERE status != 'cancelled'
-- Because NULL != 'cancelled' → UNKNOWN → filtered out

-- To include NULL statuses:
WHERE status != 'cancelled' OR status IS NULL
-- Or: WHERE COALESCE(status, '') != 'cancelled'
```

### 6.11.3 NULLs in Aggregations

| Function | NULL Behavior |
|---|---|
| `COUNT(*)` | Counts NULL rows |
| `COUNT(column)` | Skips NULLs |
| `SUM / AVG / MIN / MAX` | Skip NULLs |
| `GROUP BY` | All NULLs grouped together as one group |

### 6.11.4 NULLs in JOINs

```sql
-- NULLs never match in JOIN conditions
-- This will NOT join rows where both dept_ids are NULL:
FROM employees e
JOIN departments d ON e.dept_id = d.id
-- Because NULL = NULL → UNKNOWN → not joined

-- To handle: use COALESCE or special IS NOT DISTINCT FROM
FROM employees e
JOIN departments d ON e.dept_id IS NOT DISTINCT FROM d.id
-- IS NOT DISTINCT FROM treats NULL = NULL as TRUE (PostgreSQL)
```

### 6.11.5 NULLs in DISTINCT

`DISTINCT` treats all NULLs as identical — unlike comparisons, where NULL ≠ NULL.

```sql
SELECT DISTINCT status FROM orders;
-- If status has values: 'active', NULL, 'active', NULL, 'pending'
-- Returns: 'active', NULL, 'pending'  (only one NULL)
```

### 6.11.6 The Eight NULL Traps

| # | Trap | Wrong | Right |
|---|---|---|---|
| 1 | Comparing with `=` | `WHERE x = NULL` | `WHERE x IS NULL` |
| 2 | `NOT IN` with NULLs | `NOT IN (1, 2, NULL)` → always empty | `NOT EXISTS (...)` |
| 3 | AVG skips NULLs | `AVG(score)` on sparse data | `AVG(COALESCE(score, 0))` if NULLs mean zero |
| 4 | String concatenation | `'Hello' \|\| NULL` → `NULL` | `'Hello' \|\| COALESCE(name, '')` |
| 5 | Math with NULLs | `price * NULL` → `NULL` | `price * COALESCE(discount, 0)` |
| 6 | `!=` excludes NULLs | `WHERE status != 'X'` drops NULL rows | Add `OR status IS NULL` |
| 7 | ORDER BY NULLs | NULLs sort first or last (varies by DB) | `ORDER BY col NULLS LAST` (PostgreSQL) |
| 8 | UNIQUE allows multiple NULLs | `UNIQUE(email)` allows many NULL emails | Add `NOT NULL` if needed |

---

## 6.12 Putting It All Together — Practice Scenarios

### Scenario 1: Revenue Report (Aggregation + JOIN + Filter)

```sql
-- Top 5 product categories by revenue in Q1 2025,
-- excluding cancelled orders

SELECT
    c.category_name,
    COUNT(DISTINCT o.order_id)   AS order_count,
    SUM(oi.quantity * oi.price)  AS total_revenue,
    AVG(oi.quantity * oi.price)  AS avg_line_item
FROM order_items oi
JOIN orders o    ON oi.order_id = o.order_id
JOIN products p  ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
WHERE o.order_date BETWEEN '2025-01-01' AND '2025-03-31'
  AND o.status != 'cancelled'
GROUP BY c.category_name
HAVING SUM(oi.quantity * oi.price) > 10000
ORDER BY total_revenue DESC
LIMIT 5;
```

### Scenario 2: Customers Without Orders (LEFT JOIN + IS NULL)

```sql
-- Find customers who registered but never placed an order

SELECT c.customer_id, c.name, c.email, c.signup_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL
ORDER BY c.signup_date;
```

### Scenario 3: Running Org Chart (Recursive CTE)

```sql
-- Find all employees under a specific VP, with depth

WITH RECURSIVE team AS (
    SELECT id, name, manager_id, 0 AS depth
    FROM employees
    WHERE name = 'VP Engineering'
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, t.depth + 1
    FROM employees e
    JOIN team t ON e.manager_id = t.id
    WHERE t.depth < 10  -- safety: prevent infinite recursion
)
SELECT
    REPEAT('  ', depth) || name AS employee,
    depth
FROM team
ORDER BY depth, name;
```

### Scenario 4: Year-over-Year Comparison (CTE + Self Join)

```sql
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        EXTRACT(YEAR FROM order_date) AS year,
        EXTRACT(MONTH FROM order_date) AS month_num,
        SUM(amount) AS revenue
    FROM orders
    WHERE status != 'cancelled'
    GROUP BY 1, 2, 3
)
SELECT
    curr.month,
    curr.revenue AS current_year,
    prev.revenue AS previous_year,
    ROUND(
        (curr.revenue - prev.revenue) / NULLIF(prev.revenue, 0) * 100,
        1
    ) AS yoy_growth_pct
FROM monthly_revenue curr
LEFT JOIN monthly_revenue prev
    ON curr.month_num = prev.month_num
    AND curr.year = prev.year + 1
WHERE curr.year = 2025
ORDER BY curr.month;
```

---

## 6.13 Glossary

| Term | Definition |
|---|---|
| **Aggregate function** | A function that operates on a set of rows and returns a single value (COUNT, SUM, AVG, MIN, MAX) |
| **CASE expression** | SQL's conditional (if/else) logic. Returns a value based on evaluated conditions |
| **CASCADE** | Referential action — automatically propagate deletes/updates to child rows |
| **COALESCE** | Returns the first non-NULL argument from a list |
| **Correlated subquery** | A subquery that references columns from the outer query; re-executed for every outer row |
| **Cross join** | Cartesian product of two tables — every row paired with every other row |
| **CTE** | Common Table Expression — a named, temporary result set defined with `WITH` |
| **Derived table** | A subquery in the `FROM` clause acting as a temporary table |
| **DISTINCT** | Removes duplicate rows from a result set. Treats NULLs as identical |
| **EXISTS** | Returns TRUE if a subquery returns at least one row. Short-circuits on first match |
| **FOREIGN KEY** | A column referencing a PRIMARY KEY in another table; enforces referential integrity |
| **FULL OUTER JOIN** | Returns all rows from both tables; NULLs where no match |
| **GROUP BY** | Collapses rows into groups for aggregation |
| **Hash join** | Join algorithm using an in-memory hash table. Best for large unsorted equality joins |
| **HAVING** | Filters groups after aggregation (vs. WHERE which filters rows before) |
| **INNER JOIN** | Returns only rows with matches in both tables |
| **LEFT JOIN** | Returns all rows from the left table; NULLs for non-matching right rows |
| **Logical execution order** | The order SQL processes clauses: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT |
| **Merge join** | Join algorithm that sorts both tables then merges. Best when data is pre-sorted |
| **Nested loop join** | Join algorithm that iterates outer rows, scans inner for matches. Best for small/indexed joins |
| **NOT NULL** | Constraint preventing NULL values in a column |
| **NULL** | Represents unknown, missing, or not applicable data. Not equal to zero or empty string |
| **NULLIF** | Returns NULL if two arguments are equal; prevents division by zero |
| **PRIMARY KEY** | Unique identifier for each row. Combines UNIQUE + NOT NULL. One per table |
| **Recursive CTE** | A CTE that references itself to traverse hierarchical data |
| **Referential integrity** | Guarantee that foreign key values always point to existing parent records |
| **RESTRICT** | Referential action — block parent delete/update if child rows exist |
| **Scalar subquery** | A subquery returning exactly one value (one row, one column) |
| **Self join** | A table joined to itself using aliases |
| **Three-valued logic** | SQL's logic system: TRUE, FALSE, UNKNOWN. Any comparison with NULL yields UNKNOWN |
| **UNION / UNION ALL** | Set operation combining results. UNION deduplicates; UNION ALL keeps all rows |

---

## 6.14 Chapter Summary — The "Cheat Sheet"

```
┌───────────────────────────────────────────────────────────────┐
│  CHAPTER 6 SUMMARY CARD                                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  EXECUTION ORDER (memorize this):                             │
│  FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT│
│                                                               │
│  JOINS:                                                       │
│  • INNER: only matches                                        │
│  • LEFT: all left + matches (NULLs for no match)              │
│  • FULL OUTER: everything (NULLs both sides)                  │
│  • CROSS: cartesian product                                   │
│  • SELF: table joined to itself (aliased)                     │
│  • Algorithms: Nested Loop (small), Hash (large unsorted),    │
│    Merge (pre-sorted)                                         │
│                                                               │
│  AGGREGATIONS:                                                │
│  • COUNT(*) includes NULLs, COUNT(col) skips them             │
│  • AVG() divides by non-NULL count, not total rows            │
│  • WHERE filters rows, HAVING filters groups                  │
│                                                               │
│  SUBQUERIES & CTEs:                                           │
│  • Uncorrelated = runs once, Correlated = runs per row        │
│  • EXISTS > IN for large sets (short-circuits)                │
│  • NOT IN + NULL = disaster. Use NOT EXISTS instead            │
│  • CTEs: readable, reusable, support recursion                │
│                                                               │
│  NULL RULES:                                                  │
│  • NULL = NULL → UNKNOWN (not TRUE!)                          │
│  • Use IS NULL / IS NOT NULL (never = NULL)                   │
│  • Aggregates skip NULLs (except COUNT(*))                    │
│  • DISTINCT treats NULLs as identical                         │
│  • WHERE excludes UNKNOWN rows                                │
│                                                               │
│  CONSTRAINTS:                                                 │
│  • PK = UNIQUE + NOT NULL (one per table)                     │
│  • FK = referential integrity (CASCADE / RESTRICT)            │
│  • CHECK = custom validation rules                            │
│                                                               │
│  INTERVIEW MANTRA: Read the question → identify the join      │
│  type → write FROM/JOIN → add WHERE → add GROUP BY/HAVING     │
│  → SELECT columns → ORDER BY → verify NULL edge cases         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 6.15 What's Next

In **Chapter 7: Advanced SQL — Window Functions, Optimization, and Analytics Patterns**, we build on these fundamentals with window functions (ROW_NUMBER, RANK, LAG/LEAD, running totals), query optimization (EXPLAIN ANALYZE, indexing strategies), and advanced patterns (pivoting, gaps and islands, sessionization) that separate competent SQL users from experts.

---

*"SQL is the bicycle of data. You never forget it, and it takes you everywhere." — Every data engineer, eventually.*
