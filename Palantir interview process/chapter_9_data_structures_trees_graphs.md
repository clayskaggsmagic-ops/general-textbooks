# Chapter 9: Data Structures II — Trees, Heaps, and Graphs

---

## 9.1 Why This Matters for a Deployment Strategist

Chapter 8 covered linear structures — data stored in sequences. This chapter moves into **hierarchical** and **network** structures, which model something far more interesting: *relationships*.

This is directly relevant to Palantir because:

1. **The Ontology is a graph.** Objects are nodes, Links are edges. Understanding graph theory means understanding the data model at the heart of Gotham and Foundry
2. **Database indexes are trees.** Every `WHERE` clause you write in SQL traverses a B+ tree. Understanding how indexes work lets you explain query performance to a client's DBA
3. **Entity resolution is graph matching.** When Palantir merges records representing the same person across multiple data sources, it's performing graph operations on a property graph
4. **Relationship analysis in Gotham is graph traversal.** Tracing connections between suspects, transactions, or events is BFS/DFS on a directed graph

> **Key insight:** In Chapter 8, the question was "what container do I need?" In this chapter, the question is "how are things *connected*?" — which is the fundamental question every Deployment Strategist answers for every client.

---

## 9.2 Binary Trees — The Foundation

### 9.2.1 Core Terminology

A **binary tree** is a hierarchical structure where each node has at most two children: a **left child** and a **right child**.

```
                    ┌────┐
                    │ A  │  ← Root (depth 0, height 3)
                    └──┬─┘
                 ┌─────┴─────┐
               ┌─┴──┐      ┌─┴──┐
               │ B  │      │ C  │  ← depth 1
               └─┬──┘      └─┬──┘
             ┌───┴───┐       └───┐
           ┌─┴──┐  ┌─┴──┐     ┌─┴──┐
           │ D  │  │ E  │     │ F  │  ← depth 2
           └─┬──┘  └────┘     └────┘
             │                ↑ Leaves (no children)
           ┌─┴──┐
           │ G  │  ← depth 3 (deepest leaf)
           └────┘
```

| Term | Definition |
|---|---|
| **Node** | A data container in the tree |
| **Edge** | Connection between a parent and child |
| **Root** | The topmost node (no parent) |
| **Leaf** | A node with no children |
| **Internal node** | A node with at least one child |
| **Depth** | Distance (in edges) from the root to a node. Root has depth 0 |
| **Height** | Maximum distance from a node to its deepest descendant leaf. Leaf height = 0 |
| **Height of tree** | Height of the root = depth of the deepest leaf |

### 9.2.2 Types of Binary Trees

```
  FULL                     COMPLETE                  PERFECT
  (every node has           (all levels full          (full + complete:
   0 or 2 children)          except maybe last;        all levels
                             last is left-filled)       completely filled)

       A                        A                        A
      / \                      / \                      / \
     B   C                    B   C                    B   C
    / \                      / \ / \                  / \ / \
   D   E                   D  E F  G                 D  E F  G
      / \                  /
     F   G                H
```

| Type | Property | Key Fact |
|---|---|---|
| **Full** | Every node has 0 or 2 children | If n leaves, then n−1 internal nodes |
| **Complete** | All levels full except last; last level filled left-to-right | Array representation is efficient (heaps use this) |
| **Perfect** | Every level is completely filled | Exactly 2ʰ⁺¹ − 1 nodes for height h |

### 9.2.3 Tree Traversals

All traversals visit every node exactly once → **O(n)** time complexity.

**Depth-First Traversals (DFS):**

Using this example tree:
```
         4
        / \
       2   6
      / \ / \
     1  3 5  7
```

| Traversal | Order | Result | Use Case |
|---|---|---|---|
| **In-order** | Left → Root → Right | 1, 2, 3, **4**, 5, 6, 7 | BST sorted output — visit nodes in ascending order |
| **Pre-order** | Root → Left → Right | **4**, 2, 1, 3, 6, 5, 7 | Copying/serializing a tree (root first) |
| **Post-order** | Left → Right → Root | 1, 3, 2, 5, 7, 6, **4** | Deleting a tree (children before parent) |

**Breadth-First Traversal (BFS):**

| Traversal | Order | Result | Use Case |
|---|---|---|---|
| **Level-order** | Level by level, left to right | 4, 2, 6, 1, 3, 5, 7 | Finding shortest path, printing tree by level |

Level-order traversal uses a **queue**: enqueue the root, then repeatedly dequeue a node, process it, and enqueue its children.

---

## 9.3 Binary Search Trees (BST)

### 9.3.1 The BST Property

A BST is a binary tree with an ordering constraint:

> For every node with key **k**: all keys in the **left** subtree < **k**, and all keys in the **right** subtree > **k**

This enables efficient searching by "cutting the search space in half" at each level — the same principle as binary search on a sorted array.

```
         50              Search for 35:
        /  \               50 → go left (35 < 50)
       30   70              30 → go right (35 > 30)
      /  \  / \              40 → go left (35 < 40)
    20  40 60  80             35 → FOUND!
       /
      35                   Only visited 4 nodes out of 8 → O(log n)
```

### 9.3.2 Operations and Complexity

| Operation | Average Case | Worst Case |
|---|---|---|
| Search | **O(log n)** | O(n) |
| Insert | **O(log n)** | O(n) |
| Delete | **O(log n)** | O(n) |
| In-order traversal | O(n) | O(n) |

### 9.3.3 The Degenerate Tree Problem

The worst case occurs when elements are inserted in sorted order. The tree becomes a chain — essentially a **linked list** — with height n instead of log n:

```
  Balanced BST (height = 3):       Degenerate BST (height = 7):
  Insert: 50, 30, 70, 20, 40       Insert: 10, 20, 30, 40, 50, 60, 70

           50                        10
          /  \                        \
        30    70                      20
       /  \                            \
     20    40                          30
                                        \
  Height ≈ log₂(n)                     40
  Search: O(log n)                      \
                                        50
                                         \
                                         60
                                          \
                                          70

                                   Height = n - 1
                                   Search: O(n) → just a linked list!
```

**This is why balanced BSTs exist.** Without active rebalancing, a BST offers no performance guarantees.

---

## 9.4 Balanced BSTs — AVL Trees and Red-Black Trees

### 9.4.1 The Core Idea

Balanced BSTs automatically restructure themselves after insertions and deletions to keep the tree height at **O(log n)**, guaranteeing efficient operations regardless of insertion order.

### 9.4.2 AVL Trees — Strict Balance

Named after Adelson-Velsky and Landis (1962). An AVL tree enforces:

> For every node, the height difference between left and right subtrees (the **balance factor**) is at most 1.

When an insertion or deletion violates this constraint, the tree performs **rotations** — local restructuring operations that restore balance in O(1) time.

```
  Unbalanced (BF = 2):          After Left Rotation:
       30                             20
      /                              /  \
    20                             10    30
   /
  10

  Four rotation types:
  • Left Rotation (LL imbalance)
  • Right Rotation (RR imbalance)
  • Left-Right (LR imbalance)
  • Right-Left (RL imbalance)
```

### 9.4.3 Red-Black Trees — Relaxed Balance

Red-Black trees use node **coloring** (red or black) and a set of rules instead of strict height balancing:

1. Every node is either red or black
2. The root is always black
3. No two consecutive red nodes (a red node cannot have a red child)
4. Every path from root to a null leaf has the same number of black nodes

This ensures the longest path is at most **2x the shortest path** — looser than AVL's strict ±1, but good enough for O(log n) guarantees with fewer rotations on insert/delete.

### 9.4.4 AVL vs. Red-Black — Decision Table

| Factor | AVL Tree | Red-Black Tree |
|---|---|---|
| **Balance strictness** | Strict (height difference ≤ 1) | Relaxed (longest path ≤ 2x shortest) |
| **Search speed** | Slightly faster (lower height) | Slightly slower |
| **Insert/delete speed** | Slower (more rotations) | Faster (fewer rotations, recoloring) |
| **Rotations per operation** | Up to O(log n) | At most 2–3 |
| **Best when** | Read-heavy workloads | Write-heavy or balanced read/write |
| **Used by** | Database indexes, dictionary lookups | C++ `std::map`/`std::set`, Java `TreeMap`, Linux kernel (CFS scheduler) |

### 9.4.5 When to Use a Balanced BST vs. a Hash Table

| | Balanced BST | Hash Table |
|---|---|---|
| **Lookup** | O(log n) guaranteed | O(1) average, O(n) worst |
| **Ordering** | ✅ In-order traversal gives sorted data | ❌ No ordering |
| **Range queries** | ✅ "All keys between X and Y" efficiently | ❌ Must scan all keys |
| **Min/Max** | ✅ O(log n) | ❌ O(n) |
| **Predictability** | ✅ Guaranteed worst-case O(log n) | ⚠️ Depends on hash function quality |
| **Use when** | You need sorted data, range queries, or worst-case guarantees | You need fastest possible lookup and don't need ordering |

---

## 9.5 B-Trees and B+ Trees — Database Index Structures

### 9.5.1 The Problem They Solve

Binary trees have at most 2 children per node, which means a tree with 1 million keys has height ≈ 20. Each level requires a **disk I/O** operation, and disk reads are ~100,000x slower than memory reads. Twenty disk reads per lookup is too slow.

**B-trees** solve this by allowing hundreds of keys per node (matching the disk page size), dramatically reducing tree height. A B-tree with 1 million keys might have height = 3, meaning only 3 disk reads per lookup.

### 9.5.2 B-Tree Structure

A B-tree of order **m** (maximum children per node = m):
- Each internal node has between ⌈m/2⌉ and m children
- All leaves are at the same depth
- Keys within a node are sorted
- Both internal nodes and leaf nodes can store data pointers

```
  B-Tree of order 4 (max 3 keys, max 4 children per node):

              ┌──────────────┐
              │  30  │  60   │            ← Root (2 keys)
              └──┬───┬───┬──┘
         ┌───────┘   │   └───────┐
    ┌────┴────┐ ┌────┴────┐ ┌────┴────┐
    │ 10 │ 20 │ │ 40 │ 50 │ │ 70 │ 80 │  ← Leaf level
    └─────────┘ └─────────┘ └─────────┘

  Height = 2 for 8 keys
  A BST would need height ≈ 3 for same data
  With 1000 keys per node, height stays tiny even for billions of rows
```

### 9.5.3 B+ Tree — The Database Standard

The B+ tree is a variant of the B-tree with two key modifications:

1. **Data only in leaf nodes:** Internal nodes store only keys (for navigation), not data pointers. This makes internal nodes smaller, allowing more keys per node, which means higher fan-out and lower tree height
2. **Linked leaf nodes:** All leaf nodes are connected in a doubly-linked list, enabling efficient sequential scanning

```
┌────────────────────────────────────────────────────────────┐
│  B+ TREE                                                    │
│                                                              │
│  Internal nodes (keys only, for navigation):                 │
│                    ┌──────────┐                              │
│                    │ 30 │ 60  │                              │
│                    └─┬──┬──┬─┘                              │
│               ┌──────┘  │  └──────┐                          │
│                                                              │
│  Leaf nodes (data + linked list):                            │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                    │
│  │10│20│30 │⇄│40│50│60 │⇄│70│80│90 │                    │
│  │↓ │↓ │↓  │   │↓ │↓ │↓  │   │↓ │↓ │↓  │                    │
│  │d │d │d  │   │d │d │d  │   │d │d │d  │                    │
│  └─────────┘   └─────────┘   └─────────┘                    │
│       ↑                                                      │
│  Each ↓ points to the actual data row on disk                │
│                                                              │
│  RANGE QUERY: SELECT * WHERE key BETWEEN 30 AND 70          │
│    1. Navigate tree to find leaf with key 30                 │
│    2. Follow linked list → 30, 40, 50, 60, 70               │
│    3. No need to go back up the tree!                         │
└────────────────────────────────────────────────────────────┘
```

### 9.5.4 Why B+ Trees Dominate Database Indexing

| Advantage | Explanation |
|---|---|
| **Low height** | Hundreds of keys per node → 3–4 disk reads for billions of rows |
| **Matches disk pages** | Node size = disk page size (8KB in PostgreSQL) → one I/O per node |
| **Efficient range queries** | Linked leaf nodes enable sequential scanning without tree re-traversal |
| **Sorted output** | Leaf scan gives sorted order — supports `ORDER BY` without extra sorting |
| **High fan-out** | Internal nodes hold only keys → more keys per node → fewer levels |

### 9.5.5 Connection to PostgreSQL and Oracle

- **PostgreSQL's default index type is B-tree** (actually a B+ tree variant). When you `CREATE INDEX idx ON users(email)`, PostgreSQL builds a B+ tree
- **Oracle's B-tree indexes** are also B+ tree implementations with branch blocks for navigation and leaf blocks for data storage
- When `EXPLAIN ANALYZE` shows "Index Scan" or "Index Only Scan," it's traversing a B+ tree
- The SQL concepts from Chapters 6–7 (index scan types, covering indexes, composite indexes) all rely on B+ tree mechanics

---

## 9.6 Tries (Prefix Trees)

### 9.6.1 The Concept

A trie stores strings by breaking them into characters, with each edge representing a character. Words sharing a common prefix share the same path from the root.

```
  Storing: "car", "card", "care", "cat", "do", "dog"

                    (root)
                   /      \
                  c        d
                 /          \
                a            o
               / \            \
              r   t            g
             / \
            d   e

  Searching "care": traverse c → a → r → e → FOUND (4 steps)
  Searching "cap": traverse c → a → p → NOT FOUND (3 steps)

  Lookup time: O(m) where m = length of the search string
  Independent of how many strings are stored!
```

### 9.6.2 Complexity

| Operation | Time | Notes |
|---|---|---|
| Insert | **O(m)** | m = length of key |
| Search | **O(m)** | Independent of total number of stored strings |
| Delete | **O(m)** | Traverse then clean up |
| Prefix search | **O(p + k)** | p = prefix length, k = number of matching results |
| Space | **O(ALPHABET_SIZE × m × n)** | Can be memory-heavy — each node may hold 26+ pointers |

### 9.6.3 Applications

| Application | How Tries Help |
|---|---|
| **Autocomplete** | Type "prog" → traverse trie → find all words starting with "prog" |
| **Spell checking** | Check if typed word exists in trie dictionary |
| **IP routing** | Longest prefix matching on IP addresses in routers |
| **Dictionary lookup** | O(m) lookup regardless of dictionary size |
| **Search engine suggestions** | Match typed query against stored queries |
| **Phone directories (T9)** | Map number sequences to word completions |

### 9.6.4 Trie vs. Hash Table for String Lookups

| | Trie | Hash Table |
|---|---|---|
| **Exact lookup** | O(m) | O(1) average, but O(m) to hash the string |
| **Prefix search** | ✅ Efficient — native operation | ❌ Not possible without scanning all keys |
| **Sorted output** | ✅ DFS gives alphabetical order | ❌ Unordered |
| **Memory** | Higher (many null pointers) | Lower (compact) |
| **Best when** | Prefix matching is needed | Only exact key lookup needed |

---

## 9.7 Heaps

### 9.7.1 What is a Heap?

A heap is a **complete binary tree** that maintains the **heap property**: in a max-heap, every parent is ≥ its children; in a min-heap, every parent is ≤ its children.

```
  MAX-HEAP:                    MIN-HEAP:
       90                          10
      /  \                        /  \
    75    60                    25    30
   / \   /                    / \   /
  50 55 40                   40 35 50

  Parent ≥ Children            Parent ≤ Children
  Root = maximum               Root = minimum
```

### 9.7.2 Array Representation

Because heaps are complete binary trees, they can be stored efficiently in a simple array with no pointers:

```
  Max-heap:  90, 75, 60, 50, 55, 40

  Index:      0   1   2   3   4   5

  For node at index i:
    Parent:       (i - 1) / 2
    Left child:   2i + 1
    Right child:  2i + 2

  Example: node 75 at index 1
    Parent: (1-1)/2 = 0 → 90 ✓
    Left child: 2(1)+1 = 3 → 50
    Right child: 2(1)+2 = 4 → 55
```

### 9.7.3 Operations and Complexity

| Operation | Time | How |
|---|---|---|
| **Peek** (get min/max) | **O(1)** | Always at the root (index 0) |
| **Insert** | **O(log n)** | Add to end, "bubble up" (sift up) to restore heap property |
| **Extract** (remove min/max) | **O(log n)** | Replace root with last element, "bubble down" (sift down) |
| **Build heap** (heapify array) | **O(n)** | Bottom-up sift down — faster than n individual inserts |
| **Heap sort** | **O(n log n)** | Build max-heap, repeatedly extract max |

**Insert — Bubble Up:**
```
  Insert 85 into max-heap [90, 75, 60, 50, 55, 40]:

  Step 1: Add to end          Step 2: Bubble up (85 > 60)
  [90, 75, 60, 50, 55, 40, 85]  [90, 75, 85, 50, 55, 40, 60]

       90                           90
      /  \                         /  \
    75    60                     75    85  ← 85 swapped with 60
   / \   / \                    / \   /
  50 55 40  85←new            50 55 40  60

  85 < 90, so stop. Heap property restored.
```

### 9.7.4 Priority Queue = Heap

The **priority queue** (introduced in Chapter 8, §8.5.3) is typically implemented using a binary heap:

| Priority Queue Operation | Heap Operation | Complexity |
|---|---|---|
| Insert with priority | Heap insert + bubble up | O(log n) |
| Get highest priority | Peek at root | O(1) |
| Remove highest priority | Extract root + bubble down | O(log n) |

### 9.7.5 Applications

| Application | Heap Type | Why |
|---|---|---|
| **Dijkstra's shortest path** | Min-heap | Always process the closest unvisited vertex |
| **Top-K problems** | Min-heap of size K | Keep K largest — if new element > min, replace |
| **Task scheduling (OS)** | Max/min-heap | Highest priority process runs next |
| **Median maintenance** | Two heaps (max + min) | Max-heap for lower half, min-heap for upper half |
| **Event-driven simulation** | Min-heap | Process earliest event next |
| **Heap sort** | Max-heap | O(n log n) in-place sorting algorithm |
| **Merge K sorted lists** | Min-heap of size K | Always extract the smallest head element |

---

## 9.8 Graphs

### 9.8.1 Core Concepts

A **graph** G = (V, E) consists of a set of **vertices** (nodes) and **edges** (connections between nodes).

```
  UNDIRECTED GRAPH:              DIRECTED GRAPH (DIGRAPH):
  (edges go both ways)           (edges have direction)

    A --- B                        A → B
    |   / |                        ↑   ↓
    |  /  |                        |   ↓
    | /   |                        D ← C
    C --- D

  WEIGHTED GRAPH:                UNWEIGHTED GRAPH:
  (edges have costs)             (all edges equal)

    A -5- B                        A --- B
    |     |                        |     |
    3     2                        |     |
    |     |                        |     |
    C -1- D                        C --- D
```

### 9.8.2 Graph Terminology

| Term | Definition |
|---|---|
| **Vertex (node)** | A point in the graph |
| **Edge** | A connection between two vertices |
| **Degree** | Number of edges connected to a vertex. In directed graphs: in-degree (incoming) + out-degree (outgoing) |
| **Path** | Sequence of vertices connected by edges |
| **Cycle** | A path that starts and ends at the same vertex |
| **Connected** | Every vertex is reachable from every other (undirected) |
| **Strongly connected** | Every vertex reachable from every other following directed edges |
| **DAG** | Directed Acyclic Graph — directed graph with no cycles |
| **Sparse graph** | Few edges relative to vertices (E << V²) |
| **Dense graph** | Many edges relative to vertices (E ≈ V²) |

### 9.8.3 Representations

**Adjacency List** — Each vertex stores a list of its neighbors:

```
  Graph:   A --- B         Adjacency List:
           |   / |         A: [B, C]
           |  /  |         B: [A, C, D]
           | /   |         C: [A, B, D]
           C --- D         D: [B, C]
```

**Adjacency Matrix** — A V×V matrix where `matrix[i][j] = 1` if edge exists:

```
  Graph:   A --- B         Adjacency Matrix:
           |   / |            A  B  C  D
           |  /  |         A  0  1  1  0
           | /   |         B  1  0  1  1
           C --- D         C  1  1  0  1
                           D  0  1  1  0
```

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| **Space** | O(V + E) | O(V²) |
| **Check edge exists?** | O(degree) — scan neighbor list | **O(1)** — direct lookup |
| **Find all neighbors** | **O(degree)** — iterate list | O(V) — scan entire row |
| **Add vertex** | O(1) | O(V²) — resize matrix |
| **Add edge** | O(1) | O(1) |
| **Best for** | **Sparse graphs** (most real-world graphs) | **Dense graphs** (many edges) |

**Rule of thumb:** Almost all real-world graphs (social networks, supply chains, entity graphs) are sparse → **use adjacency lists**.

### 9.8.4 Graph Traversals — BFS vs. DFS

**BFS (Breadth-First Search)** — Level by level, using a **queue**:

```
  Start at A:

  Queue: [A]                   Visit: A
  Queue: [B, C]                Visit: B, C       (A's neighbors)
  Queue: [D]                   Visit: D           (B and C's unvisited neighbors)

  BFS Order: A → B → C → D

  Properties:
  • Explores all neighbors at distance 1, then distance 2, etc.
  • Finds SHORTEST PATH in unweighted graphs
  • Time: O(V + E)
  • Space: O(V) for the queue and visited set
```

**DFS (Depth-First Search)** — Go deep, then backtrack, using a **stack** (or recursion):

```
  Start at A:

  Stack: [A]                   Visit: A
  Stack: [B]                   Visit: B           (go deeper)
  Stack: [C]                   Visit: C           (go deeper)
  Stack: [D]                   Visit: D           (go deeper)
  Stack: []                    Backtrack — all visited

  DFS Order: A → B → C → D (path depends on neighbor ordering)

  Properties:
  • Explores as deep as possible before backtracking
  • Uses less memory than BFS on wide graphs
  • Time: O(V + E)
  • Space: O(V) for the stack and visited set
```

**BFS vs. DFS — When to Use Each:**

| Use Case | BFS | DFS |
|---|---|---|
| Shortest path (unweighted) | ✅ Guaranteed | ❌ Not guaranteed |
| Cycle detection | ✅ Works | ✅ Works (often preferred) |
| Topological sort | Via Kahn's algorithm | ✅ Natural fit |
| Connected components | ✅ Works | ✅ Works |
| Web crawling | ✅ Crawl breadth-first | Risky — may go too deep |
| Maze solving | ✅ Finds shortest path | Finds *a* path, not necessarily shortest |
| Memory (wide graphs) | ⚠️ Queue can grow large | ✅ Stack stays smaller |
| Memory (deep graphs) | ✅ Queue stays manageable | ⚠️ Stack/recursion can overflow |

### 9.8.5 Topological Sort

A **topological sort** orders the vertices of a DAG so that for every directed edge u → v, vertex u comes before v. Only possible on **Directed Acyclic Graphs** (DAGs).

```
  Course prerequisites:
     Math 101 → Math 201 → Stats 301
     CS 101 → CS 201 ↗

  Topological order: Math 101, CS 101, Math 201, CS 201, Stats 301
  (many valid orderings exist)

  Applications:
  • Build systems (compile dependencies in order)
  • Task scheduling (prerequisites before dependents)
  • Pipeline execution (ETL pipelines, Palantir Pipeline Builder)
  • Package managers (install dependencies first)
```

### 9.8.6 Cycle Detection

| Graph Type | Method | How |
|---|---|---|
| **Undirected** | DFS | If you visit an already-visited node that isn't the parent → cycle |
| **Directed** | DFS with "in-progress" state | If you visit a node currently in the recursion stack → cycle |
| **Directed** | Topological sort | If sort can't complete (not all nodes emitted) → cycle exists |
| **Undirected** | Union-Find | If two endpoints of an edge are already in the same set → cycle |

### 9.8.7 Connected Components

- **Undirected graph:** Run BFS/DFS from an unvisited node → all nodes reached form one connected component. Repeat for remaining unvisited nodes
- **Directed graph:** **Strongly Connected Components (SCCs)** — maximal subsets where every node reaches every other node. Found using Tarjan's or Kosaraju's algorithm in O(V + E)

---

## 9.9 Real-World Graph Applications

| Domain | Vertices | Edges | Graph Type |
|---|---|---|---|
| **Social networks** | Users | Friendships/follows | Undirected (Facebook) / Directed (Twitter) |
| **Supply chains** | Warehouses, factories, retailers | Shipping routes | Directed, weighted (cost/distance) |
| **Transportation** | Airports, cities | Flights, roads | Directed/undirected, weighted |
| **Knowledge graphs** | Entities (people, places, concepts) | Relationships | Directed, labeled |
| **Entity resolution** | Records | "Same entity" links | Undirected |
| **Dependency management** | Packages, tasks | Dependencies | DAG |
| **The Internet** | Web pages | Hyperlinks | Directed |

---

## 9.10 Why This All Matters for Palantir — The Ontology as a Graph

The Palantir Ontology (Chapter 3) is fundamentally a **property graph**:

```
┌────────────────────────────────────────────────────────────────┐
│  THE ONTOLOGY IS A GRAPH                                        │
│                                                                  │
│  Object Types = Vertex Types (labeled nodes)                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │  Customer     │    │   Order      │    │  Product     │       │
│  │ • name        │    │ • date       │    │ • SKU        │       │
│  │ • email       │    │ • status     │    │ • price      │       │
│  │ • segment     │    │ • total      │    │ • category   │       │
│  └──────┬───────┘    └───┬──┬───────┘    └──────┬───────┘       │
│         │                │  │                    │               │
│         │   Link Types = Edges (labeled relationships)           │
│         │                │  │                    │               │
│         └────PLACED──────┘  └────CONTAINS────────┘               │
│                                                                  │
│  Graph Operations in Palantir:                                   │
│  • Entity resolution = graph matching                            │
│  • Relationship analysis = BFS/DFS traversal                     │
│  • Supply chain optimization = shortest/cheapest path            │
│  • Intelligence analysis (Gotham) = pattern detection in graphs  │
│  • Fraud detection = cycle detection in transaction graphs       │
│  • Influence mapping = connected components analysis             │
│                                                                  │
│  When an analyst in Gotham traces connections from a suspect     │
│  to associates to organizations to bank accounts, they are       │
│  performing BFS on a directed property graph.                    │
│                                                                  │
│  When Foundry identifies duplicate customer records across       │
│  three source systems, it's performing entity resolution —       │
│  matching subgraphs across the Ontology.                         │
└────────────────────────────────────────────────────────────────┘
```

---

## 9.11 The Grand Comparison Table

| Structure | Lookup | Insert | Delete | Space | Ordered? | Best Use Case |
|---|---|---|---|---|---|---|
| **BST (unbalanced)** | O(log n)* | O(log n)* | O(log n)* | O(n) | ✅ | Simple sorted structure *if balanced* |
| **AVL Tree** | O(log n) | O(log n) | O(log n) | O(n) | ✅ | Read-heavy, needs strict balance |
| **Red-Black Tree** | O(log n) | O(log n) | O(log n) | O(n) | ✅ | General-purpose balanced BST (std library default) |
| **B+ Tree** | O(log n) | O(log n) | O(log n) | O(n) | ✅ | Database indexes, disk-based storage |
| **Trie** | O(m)† | O(m)† | O(m)† | O(Σ × n × m) | ✅ | Prefix search, autocomplete, IP routing |
| **Min/Max Heap** | O(n)‡ | O(log n) | O(log n) | O(n) | ❌ | Priority queues, Top-K, scheduling |
| **Graph (Adj. List)** | O(V+E) | O(1) | O(E) | O(V+E) | ❌ | Relationships, networks, the Ontology |
| **Graph (Adj. Matrix)** | O(1)§ | O(1) | O(1) | O(V²) | ❌ | Dense graph, fast edge lookups |

*\* Average case; worst case O(n) for degenerate trees*
*† m = length of the string key*
*‡ O(1) for peek at min/max; O(n) for arbitrary element search*
*§ For checking edge existence between two specific vertices*

---

## 9.12 Glossary

| Term | Definition |
|---|---|
| **AVL tree** | Self-balancing BST where every node's left and right subtree heights differ by at most 1 |
| **B-tree** | Self-balancing multi-way search tree optimized for disk I/O. Nodes can hold multiple keys |
| **B+ tree** | B-tree variant where data resides only in linked leaf nodes. The standard database index |
| **Balance factor** | In an AVL tree: height(left subtree) − height(right subtree). Must be −1, 0, or 1 |
| **BFS** | Breadth-First Search — level-by-level traversal using a queue |
| **Binary Search Tree (BST)** | Binary tree where left child < parent < right child |
| **Complete binary tree** | All levels full except possibly the last, which is left-filled |
| **Connected component** | Maximal subset of vertices where every pair is reachable |
| **Cycle** | A path in a graph that starts and ends at the same vertex |
| **DAG** | Directed Acyclic Graph — a directed graph with no cycles |
| **Degenerate tree** | A tree where each node has only one child — essentially a linked list |
| **DFS** | Depth-First Search — go deep, then backtrack, using a stack or recursion |
| **Full binary tree** | Every node has either 0 or 2 children |
| **Heap** | Complete binary tree satisfying heap property (parent ≥ children or ≤ children) |
| **Heapify** | O(n) operation to convert an arbitrary array into a valid heap |
| **In-order traversal** | Left → Root → Right. Produces sorted output for BSTs |
| **Level-order traversal** | BFS on a tree — visit nodes level by level using a queue |
| **Perfect binary tree** | All internal nodes have 2 children and all leaves at same depth |
| **Post-order traversal** | Left → Right → Root. Useful for tree deletion |
| **Pre-order traversal** | Root → Left → Right. Useful for tree serialization |
| **Property graph** | A graph where both vertices and edges can have key-value attributes |
| **Red-Black tree** | Self-balancing BST using node coloring for looser balance guarantees |
| **Rotation** | A local restructuring operation on a BST that restores balance |
| **SCC** | Strongly Connected Component — maximal vertex subset where all nodes reach all others |
| **Topological sort** | Linear ordering of DAG vertices such that all edges go from earlier to later in the order |
| **Trie** | Tree where each edge represents a character; paths from root to nodes spell prefixes |

---

## 9.13 Chapter Summary — The "Cheat Sheet"

```
┌───────────────────────────────────────────────────────────────┐
│  CHAPTER 9 SUMMARY CARD                                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  BINARY TREES:                                                │
│  • Max 2 children per node; root, leaves, internal nodes      │
│  • Full (0 or 2 kids), Complete (left-filled), Perfect (all)  │
│  • Traversals: In/Pre/Post-order (DFS), Level-order (BFS)     │
│                                                               │
│  BST:                                                         │
│  • Left < Root < Right → O(log n) avg search/insert/delete    │
│  • Degenerate (sorted insert) → O(n) → linked list            │
│  • Balancing essential for guarantees                          │
│                                                               │
│  BALANCED BSTs:                                               │
│  • AVL: strict balance (|BF| ≤ 1), more rotations, faster reads│
│  • Red-Black: relaxed balance, fewer rotations, faster writes  │
│  • Both guarantee O(log n) for all operations                  │
│  • Use BST over hash table when you need: ordering, range      │
│    queries, sorted traversal, or worst-case guarantees         │
│                                                               │
│  B-TREES / B+ TREES:                                          │
│  • Multi-way trees: hundreds of keys per node                  │
│  • Designed for disk I/O — node = disk page                    │
│  • B+ tree: data in linked leaf nodes → fast range queries     │
│  • PostgreSQL/Oracle default index = B+ tree variant           │
│                                                               │
│  TRIES:                                                       │
│  • Characters on edges; prefix sharing = space efficiency      │
│  • O(m) lookup where m = key length                            │
│  • Autocomplete, spell-check, IP routing, search suggest       │
│                                                               │
│  HEAPS:                                                       │
│  • Complete binary tree with parent ≥ or ≤ children            │
│  • Stored as array: parent=(i-1)/2, left=2i+1, right=2i+2     │
│  • Peek O(1), Insert O(log n), Extract O(log n)               │
│  • Implements priority queues, Top-K, Dijkstra's               │
│                                                               │
│  GRAPHS:                                                      │
│  • G = (V, E); directed/undirected; weighted/unweighted        │
│  • Adjacency list (sparse, O(V+E)) vs matrix (dense, O(V²))   │
│  • BFS: queue, level-by-level, shortest path (unweighted)      │
│  • DFS: stack/recursion, go deep, topo sort, cycle detection   │
│  • Connected components, SCCs, topological sort, cycle detect  │
│                                                               │
│  PALANTIR CONNECTION:                                          │
│  • Ontology = property graph (Objects = nodes, Links = edges)  │
│  • Entity resolution = graph matching                          │
│  • Relationship analysis = BFS/DFS traversal                   │
│  • Database indexes (SQL Chapters 6-7) = B+ tree traversal     │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 9.14 What's Next

In **Chapter 10: Algorithms I — Sorting, Searching, and Complexity Analysis**, we shift from *structures* to *processes* — the algorithms that operate on these data structures. We'll cover Big-O analysis rigorously, essential sorting algorithms, binary search patterns, and the algorithmic thinking that powers efficient system design.

---

*"At Palantir, everything is a graph. Customers are connected to orders, orders to products, products to supply chains, supply chains to geopolitical events. Your job as a Deployment Strategist is to see these connections — and the data structures in this chapter are how you model them."*
