# Flashcard Extraction Prompts

> Feed me these prompts one at a time. Each prompt produces a batch of flashcards tagged by chapter so you can filter out chapters you don't care about. I'll output the cards into `flashcards/` as JSON files, one per batch.

---

## Study Method Research Summary

Based on research into optimal study techniques, each chapter gets the card type that fits its material:

| Card Type | Best For | Used In Chapters |
|---|---|---|
| **Cloze (fill-in-blank)** | Definitions, acronyms, proper nouns, syntax | 1-7, 20 (glossary) |
| **"Explain It" (open recall)** | Trade-off reasoning, "when would you use X over Y" | 8-13, 28-30 |
| **Scenario → Response** | Interview scripts, what to say when asked X | 15-19, 25-27 |
| **Pattern Match** | "Given this problem, which algorithm/data structure?" | 8-11, 28-30 |
| **Diagram Recall** | Architecture components, data flow, system sketches | 3, 21-24, 28-30 |

**Key principles:**
- Every card tests ONE atomic fact or concept — never a paragraph
- Cards that ask "why" or "when" beat cards that ask "what"
- Acronyms get bidirectional cards (acronym → expansion AND expansion → acronym)
- Proper nouns (Palantir products, tech names) get "what is it + why does it matter" cards
- Chapter tags on every card so you can filter your study deck

---

## Batch 1: The Company & Products (Chapters 1-3)

```
Go through chapters 1-3 of the Palantir interview textbook and extract flashcards:

Chapter 1 (The Company): All proper nouns (founders, key executives, investors,
acquisitions, government contracts, revenue figures, key dates). Every Palantir-specific
term. The company history timeline. Card type: Cloze.

Chapter 2 (The Product Line): Every product name (Gotham, Foundry, AIP, Apollo, MetaConstellation).
What each product does, who uses it, how they differ. Key customer stories.
Card type: Cloze for names/definitions, "Explain It" for product comparisons.

Chapter 3 (The Ontology): Ontology definition, Object Types, Link Types, Actions,
how the Ontology differs from a traditional database schema, why Palantir considers
it their core IP. Card type: Cloze for terms, Diagram Recall for ontology structure.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 2: Databases & Cloud (Chapters 4-5)

```
Go through chapters 4-5 and extract flashcards:

Chapter 4 (Relational Databases): ACID properties, normalization forms (1NF-BCNF),
primary/foreign keys, JOIN types, indexing (B-Tree, Hash), transactions, isolation
levels, constraints. Card type: Cloze for definitions, "Explain It" for trade-offs
(e.g., "when would you denormalize?").

Chapter 5 (Cloud Data Ecosystem): AWS/GCP/Azure service names and what they do.
S3, Redshift, BigQuery, Snowflake, Databricks, Kafka, Airflow. Data lake vs.
data warehouse vs. lakehouse. ETL vs. ELT. Card type: Cloze for service names,
Pattern Match for "which tool for this job?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 3: SQL (Chapters 6-7)

```
Go through chapters 6-7 and extract flashcards:

Chapter 6 (SQL Fundamentals): SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY,
JOIN syntax, aggregate functions, NULL handling, DISTINCT, LIMIT, aliases, subqueries.
Card type: Cloze for syntax patterns, Pattern Match for "write a query that does X."

Chapter 7 (Advanced SQL): Window functions (ROW_NUMBER, RANK, DENSE_RANK, LEAD, LAG,
SUM OVER), CTEs, recursive CTEs, PARTITION BY, correlated subqueries, CASE expressions,
COALESCE, string functions. Card type: Cloze for syntax, "Explain It" for
"when would you use a CTE vs. a subquery?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 4: Data Structures (Chapters 8-9)

```
Go through chapters 8-9 and extract flashcards:

Chapter 8 (Linear Data Structures): Arrays, Linked Lists, Stacks, Queues, Hash Tables,
Heaps. Time complexity for every operation (insert, delete, search, access). When to
use each. Real-world applications. Card type: Pattern Match for "which DS for this
problem?", Cloze for Big-O complexities.

Chapter 9 (Trees & Graphs): BST, AVL, Red-Black, B-Tree, Trie, Binary Heap.
Graph representations (adjacency list vs matrix). BFS, DFS, Dijkstra, topological sort.
Card type: Cloze for definitions/complexities, "Explain It" for traversal trade-offs,
Pattern Match for "which algorithm for this graph problem?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 5: Algorithms (Chapters 10-11)

```
Go through chapters 10-11 and extract flashcards:

Chapter 10 (Algorithm Fundamentals): Sorting algorithms (Merge, Quick, Heap, Counting,
Radix) with time/space complexity. Binary search and variations. Two pointers, sliding
window, recursion, divide and conquer. Big-O notation rules. Card type: Cloze for
complexities, Pattern Match for "which sorting algorithm when?"

Chapter 11 (Advanced Algorithms): Dynamic programming (top-down vs bottom-up), greedy
algorithms, backtracking, graph algorithms (shortest path, MST, cycle detection).
When greedy works vs. when you need DP. Card type: "Explain It" for DP reasoning,
Pattern Match for "DP or greedy for this problem?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 6: Data Modeling (Chapters 12-13)

```
Go through chapters 12-13 and extract flashcards:

Chapter 12 (Relational & Dimensional Modeling): Star schema, snowflake schema,
fact tables, dimension tables, slowly changing dimensions (SCD Types 1-3), surrogate
keys, Kimball vs Inmon methodology. Card type: Cloze for terms, "Explain It" for
"star vs snowflake trade-offs", Diagram Recall for schema layouts.

Chapter 13 (Graph, NoSQL & Ontology Modeling): Document stores, key-value stores,
wide-column stores, graph databases. CAP theorem. Ontology mapping to Palantir's
object model. When to use graph vs. relational. Card type: Cloze for NoSQL families,
"Explain It" for CAP trade-offs, Pattern Match for "which DB model for this use case?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 7: Pipelines & DS Role (Chapters 14-15)

```
Go through chapters 14-15 and extract flashcards:

Chapter 14 (Data Pipelines): Batch vs stream processing, Kafka, Airflow, Spark,
Flink. Data quality (completeness, accuracy, consistency, timeliness). Idempotency.
Exactly-once vs at-least-once delivery. CDC. Schema evolution. Card type: Cloze for
tool names, "Explain It" for batch vs stream trade-offs.

Chapter 15 (The Deployment Strategist Role): What a DS does day-to-day, how it differs
from SWE/PM/consultant. The "Forward Deployed" model. Skills tested. What Palantir
values. Card type: Scenario → Response for "describe the role" interview questions,
Cloze for Palantir-specific role terminology.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 8: Interview Process & Case Study (Chapters 16-17)

```
Go through chapters 16-17 and extract flashcards:

Chapter 16 (The Interview Process): Interview stages (phone screen, Karat,
decomposition, case study, system design, behavioral, host match). What each tests.
Timeline. How to prepare for each. Card type: Cloze for stage names/descriptions,
Scenario → Response for "what to expect" preparation.

Chapter 17 (Case Study Mastery): Frameworks (MECE, issue trees, hypothesis-driven).
How to structure a case answer. Opening scripts. Closing scripts. Common case types.
Card type: Scenario → Response for scripts, Cloze for framework acronyms.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 9: Deployments & Philosophy (Chapters 18-19)

```
Go through chapters 18-19 and extract flashcards:

Chapter 18 (Real World Deployments): Specific Palantir deployment case studies.
Customer names, industries, problems solved, data types used, outcomes achieved.
Card type: Cloze for case study facts, Scenario → Response for "tell me about a
Palantir deployment."

Chapter 19 (The Philosophy): Palantir's founding philosophy, Alex Karp's worldview,
the tension between Silicon Valley and government work, Palantir's stance on data
ethics, the "Western values" framing. Card type: "Explain It" for philosophical
positions, Cloze for key quotes/positions.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 10: Glossary & Schema Design (Chapters 20-21)

```
Go through chapters 20-21 and extract flashcards:

Chapter 20 (Vocabulary Glossary): EVERY term in the glossary gets a card. Acronyms
get bidirectional cards (acronym → definition AND definition → acronym). Palantir-
specific terms get extra context on why they matter. Card type: Cloze (bidirectional
for acronyms).

Chapter 21 (Schema Design): Entity-relationship modeling, schema design patterns,
normalization decisions, Palantir-specific schema considerations, object type mapping.
Card type: Cloze for patterns, Diagram Recall for ER diagrams, "Explain It" for
design decisions.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 11: Wargame Project (Chapters 22-24)

```
Go through chapters 22-24 and extract flashcards:

Chapter 22 (Wargame Project Overview): What the wargame project is, its purpose as a
portfolio piece, how it demonstrates Palantir-relevant skills.

Chapter 23 (Wargame Data Architecture): The data model, entity types, relationships,
data sources, pipeline design. Technical decisions and trade-offs.

Chapter 24 (Wargame Interview Talking Points): How to present the project in an
interview, key technical decisions to highlight, anticipated questions and answers.
Card type: Scenario → Response for talking points, Cloze for architecture facts.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 12: Case Studies & Decomposition (Chapters 25-27)

```
Go through chapters 25-27 and extract flashcards:

Chapter 25 (Consulting Case Fundamentals): MECE principle, issue trees, frameworks
(profitability, market entry, M&A), hypothesis-driven structure, the 4-step case
method. Opening scripts and closing scripts for case interviews. Card type:
Scenario → Response for scripts, Cloze for framework acronyms.

Chapter 26 (Decomposition Playbook): The 5 decomposition moves (Stakeholder Map,
Data Inventory, Decision Architecture, Progressive Disclosure, Ontology Close).
Interviewer pushback patterns and responses. Card type: Scenario → Response for
each move's script, Cloze for move names.

Chapter 27 (Decomposition Practice Lab): Practice scenarios with model answers.
Key patterns that repeat across scenarios. Card type: Scenario → Response for
the scripted answers, Pattern Match for "which move for this situation?"

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## Batch 13: System Design (Chapters 28-30)

```
Go through chapters 28-30 and extract flashcards:

Chapter 28 (System Design Fundamentals): The 45-minute structure, requirements
scoping template, back-of-envelope numbers to memorize, the signpost technique,
CAP/PACELC theorem, fan-out on read vs write, Snowflake IDs. Card type: Cloze
for numbers/acronyms, "Explain It" for CAP trade-offs, Scenario → Response for
interview scripts.

Chapter 29 (System Design Building Blocks): SQL vs NoSQL decision framework, Redis
vs Memcached, cache invalidation patterns (TTL, event-driven, write-through),
Kafka vs RabbitMQ vs SQS, delivery guarantees, L4 vs L7 load balancing, REST vs
GraphQL vs gRPC, consistency patterns, ACID isolation levels. Card type: Pattern
Match for "which tool for this job?", Cloze for definitions, "Explain It" for
every trade-off comparison.

Chapter 30 (System Design Practice): Key design patterns (fan-out hybrid, Bloom
filter dedup, quorum consensus, consistent hashing, heartbeat presence). The core
design for each of the 6 practice problems — URL shortener, rate limiter, chat,
crawler, notifications, distributed KV store. Card type: Pattern Match for
"which pattern for this problem?", Diagram Recall for architectures.

Tag all cards with their chapter number. Output as JSON array with fields:
{ chapter, card_type, front, back, tags[] }
```

---

## After All Batches: Build the App

```
Now take all the JSON flashcard files in flashcards/ and build a single-file
offline HTML flashcard app with these features:

1. FSRS-lite spaced repetition engine (built-in, no server needed)
2. Chapter filter — toggle chapters on/off to customize your study deck
3. Card type filter — study only "Explain It" cards, or only Cloze, etc.
4. Progress tracking — cards due, cards mastered, streak counter
5. Keyboard shortcuts (Space=flip, 1-4=difficulty rating, arrow keys=navigate)
6. Dark mode premium UI — must work beautifully offline on a plane
7. LocalStorage persistence — progress survives browser refresh
8. Export/import — backup progress as JSON
9. EASY CARD DELETION — every card gets a prominent delete/trash button visible
   while studying. One tap and it's gone, no confirmation modal. Also add a
   "Manage Cards" view where you can see all cards in a list and bulk-select
   cards to delete. Deleted cards are permanently removed from LocalStorage.
   Keyboard shortcut: 'D' or 'Delete' key to nuke the current card instantly.

The app must be a SINGLE .html file with all CSS/JS inline so it works
completely offline with no internet connection.
```
