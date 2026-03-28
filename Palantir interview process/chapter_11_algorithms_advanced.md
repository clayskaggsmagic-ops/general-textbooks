# Chapter 11: Algorithms II — Dynamic Programming, Greedy, Graph Algorithms, and Backtracking

---

## 11.1 Why This Matters for a Deployment Strategist

Chapter 10 introduced the building blocks — complexity analysis, sorting, searching, recursion. This chapter covers the **paradigms** — the high-level strategies for attacking problems you've never seen before.

This matters for Palantir because:

1. **The DS interview tests decomposition, not coding.** You'll be given vague, multi-part problems and judged on *how* you break them down. The paradigms in this chapter — DP, greedy, graph algorithms, backtracking — are the classification system for that decomposition
2. **Foundry pipelines are DAGs.** Every data pipeline in Foundry is a Directed Acyclic Graph of transforms. Topological sort determines execution order. Understanding graph algorithms means understanding how your client's data flows
3. **Optimization is a daily conversation.** "What's the fastest way to route shipments?" (shortest path). "What's the cheapest way to connect these warehouses?" (MST). "What's the best way to allocate budget across projects?" (knapsack). These are the exact conversations DSes have with clients
4. **Knowing *when* greedy fails saves you.** A DS who recommends a greedy heuristic for a problem that requires DP will build a solution that works on test data and fails in production. Understanding the distinction is critical

> **Key insight:** The interview doesn't ask you to implement Dijkstra's. It asks you to *recognize* that a client's routing problem is a shortest-path problem, *identify* the right paradigm, and *reason* about trade-offs. This chapter gives you that recognition toolkit.

---

## 11.2 Dynamic Programming — Optimization Through Memory

### 11.2.1 The Two Required Properties

A problem can be solved with DP if and only if it has **both** of these properties:

**1. Optimal Substructure** — An optimal solution contains optimal solutions to its subproblems.

```
  Shortest path from A to C through B:

  A ──3──> B ──2──> C     Total: 5

  If this is the shortest path A→C, then:
  • A→B (cost 3) must be the shortest path from A to B
  • B→C (cost 2) must be the shortest path from B to C

  If either sub-path weren't optimal, we could swap it for
  a shorter one and get a shorter total — contradiction.

  ✅ Has optimal substructure → DP candidate
```

**2. Overlapping Subproblems** — The same subproblems appear multiple times during computation.

```
  Fibonacci:                          Shortest path (no overlaps):
  fib(5)                              A → B → C (each edge visited once)
  ├── fib(4)                          No repeated subproblems
  │   ├── fib(3) ← repeated!
  │   └── fib(2) ← repeated!         Without overlap, we use
  └── fib(3) ← repeated!             divide-and-conquer instead
      ├── fib(2) ← repeated!         (like Merge Sort)
      └── fib(1)

  ✅ Has overlapping subproblems → DP candidate
```

**The key distinction:**

| Both properties? | Strategy |
|---|---|
| ✅ Optimal substructure + ✅ Overlapping subproblems | **Dynamic Programming** |
| ✅ Optimal substructure + ❌ No overlapping subproblems | **Divide and Conquer** (Merge Sort) |
| ❌ No optimal substructure | DP won't work — try other approaches |

### 11.2.2 Top-Down (Memoization) vs. Bottom-Up (Tabulation)

There are two ways to implement DP, and they produce **identical results**:

```
  TOP-DOWN (Memoization)           BOTTOM-UP (Tabulation)
  ─────────────────────            ──────────────────────
  Start from the big problem       Start from the smallest subproblems
  Recurse down to base cases       Build up to the answer iteratively
  Cache results in a hash map      Fill a table row by row

  fib(n):                          fib(n):
    if n in memo: return memo[n]     dp = [0, 1]
    if n ≤ 1: return n               for i in 2..n:
    memo[n] = fib(n-1) + fib(n-2)      dp[i] = dp[i-1] + dp[i-2]
    return memo[n]                    return dp[n]

  Direction: n → 0 (top-down)      Direction: 0 → n (bottom-up)
  Structure: Recursion + cache     Structure: Loop + table
```

| | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| **Direction** | Big problem → base cases | Base cases → big problem |
| **Implementation** | Recursive + hash map/array | Iterative + table/array |
| **Subproblems solved** | Only the ones needed | All subproblems in order |
| **Stack overflow risk** | ⚠️ Yes, for deep recursion | ❌ None |
| **Space optimization** | Harder (full cache) | Easier (often only need last few values) |
| **Intuition** | More natural (follows recurrence) | Requires figuring out fill order |
| **When to prefer** | Problem has many unreachable subproblems | Need to optimize space; iterative order is clear |

### 11.2.3 How to Recognize DP Problems

Look for these **signal phrases** in problem statements:

```
  ┌───────────────────────────────────────────────────────────────┐
  │  "How many ways can you..."        → Count paths/combinations │
  │  "What is the minimum cost to..."  → Optimization problem     │
  │  "What is the maximum value..."    → Optimization problem     │
  │  "Is it possible to..."           → Decision (yes/no)         │
  │  "Find the longest/shortest..."   → Optimization on sequence  │
  │  "Find all ways to..."            → Enumeration               │
  └───────────────────────────────────────────────────────────────┘
```

**The DP Problem-Solving Framework:**

```
  Step 1: DEFINE THE STATE
          What variables describe a unique subproblem?
          → dp[i] = "answer for the first i elements"
          → dp[i][j] = "answer using items 1..i with capacity j"

  Step 2: WRITE THE TRANSITION (recurrence relation)
          How does dp[i] relate to smaller subproblems?
          → dp[i] = dp[i-1] + dp[i-2]  (Fibonacci)
          → dp[i] = max(dp[i-1], dp[j] + value[i])  (LIS)

  Step 3: IDENTIFY BASE CASES
          What are the trivial subproblems?
          → dp[0] = 0, dp[1] = 1  (Fibonacci)

  Step 4: DETERMINE COMPUTATION ORDER
          Which subproblems must be solved first?
          → Usually left-to-right or small-to-large

  Step 5: EXTRACT THE ANSWER
          Where in the table is the final answer?
          → dp[n], max(dp[...]), dp[m][n]
```

### 11.2.4 Classic DP Problems

**Climbing Stairs** — "How many ways to climb n stairs, taking 1 or 2 steps at a time?"

```
  State:     dp[i] = number of ways to reach step i
  Transition: dp[i] = dp[i-1] + dp[i-2]
  Base cases: dp[0] = 1, dp[1] = 1
  Answer:    dp[n]

  This IS the Fibonacci sequence in disguise!

  n=5: dp = [1, 1, 2, 3, 5, 8] → 8 ways
  Time: O(n) | Space: O(1) with optimization
```

**Coin Change** — "Minimum coins to make amount A from denominations {c₁, c₂, ..., cₖ}"

```
  State:      dp[a] = minimum coins to make amount a
  Transition: dp[a] = min(dp[a - cⱼ] + 1) for each coin cⱼ ≤ a
  Base case:  dp[0] = 0 (zero coins for zero amount)
  Answer:     dp[A]

  Example: coins = {1, 3, 4}, amount = 6

  dp[0] = 0
  dp[1] = dp[1-1] + 1 = 1            (use coin 1)
  dp[2] = dp[2-1] + 1 = 2            (use coin 1)
  dp[3] = min(dp[3-1]+1, dp[3-3]+1)
        = min(3, 1) = 1              (use coin 3)
  dp[4] = min(dp[4-1]+1, dp[4-3]+1, dp[4-4]+1)
        = min(2, 2, 1) = 1           (use coin 4)
  dp[5] = min(dp[4]+1, dp[2]+1, dp[1]+1)
        = min(2, 3, 2) = 2           (use coin 4 + coin 1)
  dp[6] = min(dp[5]+1, dp[3]+1, dp[2]+1)
        = min(3, 2, 3) = 2           (use coin 3 + coin 3) ✓

  Time: O(A × k) | Space: O(A)
```

**0/1 Knapsack** — "Maximize value of items in a knapsack with weight limit W"

```
  State:      dp[i][w] = max value using first i items with capacity w
  Transition: dp[i][w] = max(
                dp[i-1][w],                    ← skip item i
                dp[i-1][w - wᵢ] + vᵢ          ← take item i (if wᵢ ≤ w)
              )
  Base case:  dp[0][w] = 0 for all w
  Answer:     dp[n][W]

  Items: {(weight=2, value=6), (weight=3, value=10), (weight=4, value=12)}
  Capacity W = 5

       w→  0  1  2  3  4  5
  i=0      0  0  0  0  0  0
  i=1      0  0  6  6  6  6     (item 1: w=2, v=6)
  i=2      0  0  6 10 10 16     (item 2: w=3, v=10)
  i=3      0  0  6 10 12 16     (item 3: w=4, v=12)

  Answer: dp[3][5] = 16 (take items 1 and 2: 6+10=16, weight 2+3=5)

  Time: O(n × W) | Space: O(n × W), can optimize to O(W)
```

**Longest Common Subsequence (LCS)** — "Find the longest subsequence common to both strings"

```
  State:      dp[i][j] = LCS length of first i chars of s1 and first j chars of s2
  Transition: if s1[i] == s2[j]: dp[i][j] = dp[i-1][j-1] + 1
              else:              dp[i][j] = max(dp[i-1][j], dp[i][j-1])
  Base case:  dp[0][j] = 0, dp[i][0] = 0
  Answer:     dp[m][n]

  s1 = "ABCBDAB", s2 = "BDCAB"

       ""  B  D  C  A  B
  ""    0  0  0  0  0  0
  A     0  0  0  0  1  1
  B     0  1  1  1  1  2
  C     0  1  1  2  2  2
  B     0  1  1  2  2  3
  D     0  1  2  2  2  3
  A     0  1  2  2  3  3
  B     0  1  2  2  3  4

  LCS = "BCAB" (length 4)
  Time: O(m × n) | Space: O(m × n), can optimize to O(min(m, n))
```

**Edit Distance (Levenshtein)** — "Minimum insertions, deletions, and substitutions to transform s1 into s2"

```
  State:      dp[i][j] = edit distance between s1[0..i-1] and s2[0..j-1]
  Transition: if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]        ← characters match, no edit
              else:
                dp[i][j] = 1 + min(
                  dp[i-1][j],                   ← delete from s1
                  dp[i][j-1],                   ← insert into s1
                  dp[i-1][j-1]                  ← substitute
                )
  Base case:  dp[i][0] = i, dp[0][j] = j
  Answer:     dp[m][n]

  "kitten" → "sitting": edit distance = 3
  (substitute k→s, substitute e→i, insert g)

  Time: O(m × n) | Space: O(m × n)
```

**Longest Increasing Subsequence (LIS)** — "Find the longest strictly increasing subsequence"

```
  State:      dp[i] = length of LIS ending at index i
  Transition: dp[i] = max(dp[j] + 1) for all j < i where arr[j] < arr[i]
  Base case:  dp[i] = 1 for all i (each element is an LIS of length 1)
  Answer:     max(dp[0..n-1])

  arr = [10, 9, 2, 5, 3, 7, 101, 18]

  dp  = [1,  1, 1, 2, 2, 3, 4,   4]

  LIS = [2, 5, 7, 101] or [2, 3, 7, 101] — length 4

  Time: O(n²) | Space: O(n)
  Note: Can be optimized to O(n log n) using binary search + patience sorting
```

### 11.2.5 The DP Classification Table

| Problem | State | Dimensions | Time | Space |
|---|---|---|---|---|
| **Fibonacci / Climbing Stairs** | dp[i] = ways to reach position i | 1D | O(n) | O(1)* |
| **Coin Change** | dp[a] = min coins for amount a | 1D | O(A·k) | O(A) |
| **0/1 Knapsack** | dp[i][w] = max value, items 1..i, capacity w | 2D | O(n·W) | O(W)* |
| **LCS** | dp[i][j] = LCS of s1[0..i], s2[0..j] | 2D | O(m·n) | O(m·n) |
| **Edit Distance** | dp[i][j] = edits for s1[0..i] → s2[0..j] | 2D | O(m·n) | O(m·n) |
| **LIS** | dp[i] = LIS ending at index i | 1D | O(n²) | O(n) |

*\* With space optimization*

---

## 11.3 Greedy Algorithms — Locally Optimal, Globally Optimal

### 11.3.1 The Greedy Choice Property

A greedy algorithm makes the **locally optimal choice** at each step, hoping to find a **globally optimal** solution. It never reconsiders a choice once made.

```
  DP:     "Consider ALL options at each step → guarantee global optimum"
  Greedy: "Pick the BEST option at each step → hope for global optimum"

  Greedy works ONLY when the greedy choice property holds:
  "A globally optimal solution can be assembled from locally optimal choices."
```

**When greedy works:**

| Problem | Greedy Strategy | Why It Works |
|---|---|---|
| Activity selection | Pick the activity that finishes earliest | Leaves the most remaining time for other activities |
| Fractional knapsack | Take items by highest value/weight ratio | Can take fractions, so no "all or nothing" trade-off |
| Huffman coding | Merge the two lowest-frequency nodes | Most frequent symbols naturally get shortest codes |
| Coin change (canonical coins) | Use the largest denomination first | Standard denominations (25, 10, 5, 1) have the greedy property |

**When greedy FAILS:**

| Problem | Greedy Attempt | Why It Fails |
|---|---|---|
| **0/1 Knapsack** | Take highest value/weight ratio first | Can't take fractions — may miss a better combination |
| **Coin change (non-canonical)** | Take largest coin first | Coins {1, 3, 4} for amount 6: greedy → 4+1+1 (3 coins); optimal → 3+3 (2 coins) |
| **Traveling Salesman** | Visit nearest unvisited city | Can paint yourself into a corner with long final edges |
| **Negative edge shortest path** | Dijkstra's greedy relaxation | Negative edges can make previously "settled" nodes wrong |

### 11.3.2 Classic Greedy Problems

**Activity Selection / Interval Scheduling** — "Select the maximum number of non-overlapping activities"

```
  Activities (sorted by finish time):
  ┌─────────────────────────────────────────────────────────────┐
  │ A1: [1────3]                                                │
  │ A2:   [2──────5]                                            │
  │ A3:      [3──4]                                             │
  │ A4:         [4────6]                                        │
  │ A5:              [6──7]                                     │
  │ A6:           [5───────8]                                   │
  └─────────────────────────────────────────────────────────────┘

  Greedy: Sort by finish time. Pick first. Skip overlapping. Repeat.

  Pick A1 [1,3] → skip A2 (overlaps) → pick A3 [3,4] →
  pick A4 [4,6] → pick A5 [6,7] → skip A6 (overlaps)

  Result: {A1, A3, A4, A5} = 4 activities ← OPTIMAL

  Why it works: Picking the earliest-finishing activity always leaves
  the most room for subsequent activities. Provably optimal.

  Time: O(n log n) for sort + O(n) for selection = O(n log n)
```

**Fractional Knapsack** — "Maximize value, fractions allowed"

```
  Items: {(w=10, v=60), (w=20, v=100), (w=30, v=120)}
  Capacity: W = 50

  Value/weight ratios: 6.0, 5.0, 4.0

  Greedy: Sort by ratio. Take fully or partially.

  Take item 1 fully  (w=10, v=60)   → remaining capacity: 40
  Take item 2 fully  (w=20, v=100)  → remaining capacity: 20
  Take 2/3 of item 3 (w=20, v=80)  → remaining capacity: 0

  Total value: 60 + 100 + 80 = 240 ← OPTIMAL

  Time: O(n log n)
```

**Huffman Coding** — "Build optimal prefix code for data compression"

```
  Characters: a(45), b(13), c(12), d(16), e(9), f(5)

  Greedy: Repeatedly merge the two lowest frequencies.

  Step 1: Merge f(5) + e(9) → [fe](14)
  Step 2: Merge c(12) + b(13) → [cb](25)
  Step 3: Merge [fe](14) + d(16) → [fed](30)
  Step 4: Merge [cb](25) + [fed](30) → [cbfed](55)
  Step 5: Merge a(45) + [cbfed](55) → [root](100)

  Result:
    a → 0         (1 bit — most frequent)
    b → 101       (3 bits)
    c → 100       (3 bits)
    d → 111       (3 bits)
    e → 1101      (4 bits)
    f → 1100      (4 bits — least frequent)

  Most frequent characters get shortest codes → OPTIMAL compression
```

### 11.3.3 Greedy vs. DP — The Decision Guide

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  CAN I BE GREEDY?                                            │
  │                                                              │
  │  Ask: "Does the locally optimal choice always lead           │
  │        to the globally optimal solution?"                    │
  │                                                              │
  │  YES                              NO                         │
  │   ↓                                ↓                         │
  │  Use GREEDY                       Does it have               │
  │  O(n) or O(n log n)              overlapping subproblems?    │
  │  Simple, fast                         ↓                      │
  │                                  YES         NO              │
  │                                   ↓           ↓              │
  │                                 Use DP     Use Divide        │
  │                                              & Conquer       │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

| | Greedy | Dynamic Programming |
|---|---|---|
| **Choices** | Make the best choice NOW, never look back | Consider ALL choices, pick the best overall |
| **Guarantee** | Only when greedy choice property holds | Always finds the optimal (if applicable) |
| **Speed** | Usually faster (O(n) or O(n log n)) | Usually slower (often O(n²) or O(n·W)) |
| **Simplicity** | Simpler to implement | More complex (define state, transition, base cases) |
| **Example** | Fractional knapsack ← greedy works | 0/1 knapsack ← greedy fails, DP needed |

---

## 11.4 Graph Algorithms — Navigating Networks

### 11.4.1 Shortest Path Algorithms

**BFS for Unweighted Graphs** — O(V + E)

```
  In unweighted graphs, BFS naturally finds shortest paths because
  it explores all vertices at distance d before distance d+1.

  From Chapter 9: BFS uses a queue and visits level by level.

  Use when: All edges have equal weight (or weight = 1)
```

**Dijkstra's Algorithm** — "Shortest path from source, non-negative weights"

```
  Strategy: Greedy — always process the unvisited vertex with smallest
  known distance. Once a vertex is "settled," its distance is final.

  Algorithm:
  1. Set dist[source] = 0, dist[all others] = ∞
  2. Add source to a min-heap (priority queue)
  3. While heap not empty:
     a. Extract vertex u with minimum distance
     b. For each neighbor v of u:
        If dist[u] + weight(u,v) < dist[v]:
          dist[v] = dist[u] + weight(u,v)     ← "relaxation"
          Add/update v in the min-heap

  Example:
        A ──1──> B ──2──> D
        |        ↑        ↑
        4        3        1
        |        |        |
        ↓        |        |
        C ──1──> E ──1──> F

  Process A (dist=0):  B=1, C=4
  Process B (dist=1):  D=3
  Process D (dist=3):  (no unvisited neighbors improve)
  Process C (dist=4):  E=5
  Process E (dist=5):  B already settled, F=6
  Process F (dist=6):  D already settled

  Time: O((V + E) log V) with binary min-heap
        O(V² + E) with simple array (better for dense graphs)
  Space: O(V + E)

  ⚠️ DOES NOT WORK with negative edge weights!
```

**Bellman-Ford Algorithm** — "Shortest path from source, handles negative weights"

```
  Strategy: Relax ALL edges, V-1 times. If still improving after
  V-1 iterations, there's a negative cycle.

  Algorithm:
  1. Set dist[source] = 0, dist[all others] = ∞
  2. Repeat V-1 times:
     For each edge (u, v, weight):
       If dist[u] + weight < dist[v]:
         dist[v] = dist[u] + weight            ← relaxation
  3. Check for negative cycles:
     For each edge (u, v, weight):
       If dist[u] + weight < dist[v]:
         → Negative cycle detected! No shortest path exists.

  Time: O(V × E) — significantly slower than Dijkstra's
  Space: O(V)

  Use when: Graph has negative edge weights
  Bonus: Can detect negative cycles (Dijkstra's cannot)
```

**The Shortest Path Decision Table:**

| Algorithm | Weights | Negative edges? | Negative cycle detection? | Time | Best for |
|---|---|---|---|---|---|
| **BFS** | Unweighted (all = 1) | N/A | N/A | O(V + E) | Unweighted graphs |
| **Dijkstra's** | Non-negative | ❌ Fails | ❌ | O((V+E) log V) | Most weighted graphs |
| **Bellman-Ford** | Any | ✅ Handles | ✅ Detects | O(V × E) | Negative weights |

### 11.4.2 Topological Sort — Ordering Dependencies

Topological sort produces a linear ordering of vertices in a **Directed Acyclic Graph (DAG)** such that for every directed edge u → v, vertex u comes before v.

```
  Course prerequisites:
  CS101 → CS201 → CS301
  CS101 → CS202
  MATH101 → CS202 → CS301

  Valid topological orders:
  [MATH101, CS101, CS201, CS202, CS301]  ✓
  [CS101, MATH101, CS202, CS201, CS301]  ✓

  Invalid:
  [CS301, CS101, ...]  ✗ (CS301 requires CS101)

  Key requirement: Graph must be a DAG (no cycles).
  If there IS a cycle → no valid topological order exists.
```

**Kahn's Algorithm (BFS-Based):**

```
  1. Compute in-degree for every vertex
  2. Add all vertices with in-degree 0 to a queue
  3. While queue is not empty:
     a. Remove vertex u from queue → add to result
     b. For each neighbor v of u:
        Decrement in-degree of v
        If in-degree of v becomes 0 → add v to queue
  4. If result has fewer than V vertices → cycle exists!

  Time: O(V + E) | Space: O(V)
  Bonus: Naturally detects cycles
```

**DFS-Based Topological Sort:**

```
  1. For each unvisited vertex, run DFS
  2. After visiting all neighbors, push current vertex to stack
  3. Pop all vertices from stack → topological order

  Time: O(V + E) | Space: O(V)
```

| | Kahn's (BFS) | DFS-Based |
|---|---|---|
| **Style** | Iterative, uses queue | Recursive, uses stack |
| **Cycle detection** | ✅ Natural (result size < V) | Requires extra "in progress" tracking |
| **Intuitiveness** | Very intuitive (process ready items) | Less intuitive but more general |

**Real-World Applications:**

| Application | What it models |
|---|---|
| **Build systems** (Make, Gradle, npm) | Compile dependencies in correct order |
| **Course scheduling** | Ensure prerequisites are completed first |
| **Foundry pipeline execution** | Execute data transforms in dependency order |
| **Task scheduling** | Order tasks respecting precedence constraints |
| **Package managers** | Install dependencies before dependents |

### 11.4.3 Minimum Spanning Trees — Cheapest Connectivity

A Minimum Spanning Tree (MST) connects all vertices with the **minimum total edge weight** and **no cycles**.

**Prim's Algorithm** — "Grow a tree from a starting vertex"

```
  Strategy: Start at any vertex. Repeatedly add the cheapest
  edge connecting the tree to a non-tree vertex.

  Like Dijkstra's, but instead of tracking total distance from source,
  track the cheapest edge to join the growing tree.

        A
       / \
      1   4
     /     \
    B───3───C
    |       |
    2       5
    |       |
    D───6───E

  Start at A:
  Add edge A-B (1) → tree: {A, B}
  Add edge B-D (2) → tree: {A, B, D}
  Add edge B-C (3) → tree: {A, B, D, C}
  Add edge C-E (5) → tree: {A, B, D, C, E}

  MST edges: {A-B, B-D, B-C, C-E}, total weight = 1+2+3+5 = 11

  Time: O(E log V) with binary min-heap
        O(V²) with adjacency matrix (better for dense graphs)
```

**Kruskal's Algorithm** — "Sort edges, add cheapest non-cycle edges"

```
  Strategy: Sort ALL edges by weight. Add edges one by one,
  skipping any edge that would create a cycle.

  Uses Union-Find to efficiently detect cycles.

  Same graph:
  Sorted edges: A-B(1), B-D(2), B-C(3), A-C(4), C-E(5), D-E(6)

  Add A-B(1) → no cycle → {A,B}
  Add B-D(2) → no cycle → {A,B,D}
  Add B-C(3) → no cycle → {A,B,D,C}
  Add A-C(4) → ✗ CYCLE (A and C already connected) → skip
  Add C-E(5) → no cycle → {A,B,D,C,E} → DONE (V-1 = 4 edges)

  Same MST, total weight = 11

  Time: O(E log E) for sorting ≈ O(E log V) since E ≤ V²
```

**Union-Find (Disjoint Set Union)** — The backbone of Kruskal's:

```
  Two operations:
  • Find(x):   Which set does x belong to? (returns the root/representative)
  • Union(x,y): Merge the sets containing x and y

  Optimizations:
  • Path Compression: During Find, point every node directly to root
  • Union by Rank: Attach smaller tree under larger tree's root

  With both optimizations: amortized O(α(n)) per operation
  α(n) = inverse Ackermann function ≈ effectively O(1)

  Initial:  {A} {B} {C} {D} {E}  ← each vertex is its own set

  Union(A,B):  {A,B} {C} {D} {E}
  Union(B,D):  {A,B,D} {C} {E}
  Union(B,C):  {A,B,D,C} {E}

  Find(A) == Find(C)?  → Yes! Both in same set → adding A-C would cycle
```

| | Prim's | Kruskal's |
|---|---|---|
| **Approach** | Vertex-based (grow one tree) | Edge-based (sort and merge forests) |
| **Data structure** | Min-heap / priority queue | Sort + Union-Find |
| **Better for** | **Dense graphs** (many edges) | **Sparse graphs** (few edges) |
| **Disconnected graphs** | ❌ Requires connected graph | ✅ Produces MST forest |
| **Time** | O(E log V) or O(V²) | O(E log E) ≈ O(E log V) |

**Real-World MST Applications:**

| Application | What it models |
|---|---|
| **Network cable routing** | Minimum cable to connect all offices |
| **Power grid design** | Minimum wiring to reach all substations |
| **Road network planning** | Minimum roads to connect all towns |
| **Cluster analysis** | Remove longest edges from MST → natural clusters |

### 11.4.4 Cycle Detection and Connected Components

**Cycle Detection:**

| Method | Graph Type | Approach |
|---|---|---|
| **DFS with coloring** | Directed graphs | Track "in progress" (gray) nodes; back edge = cycle |
| **Union-Find** | Undirected graphs | If Find(u) == Find(v) when adding edge (u,v) → cycle |
| **Kahn's algorithm** | Directed graphs | If topological sort result has < V vertices → cycle exists |

**Connected Components:**

```
  Undirected graph: Run BFS/DFS from each unvisited vertex.
  Each run discovers one connected component.

  Directed graph: Use Kosaraju's or Tarjan's algorithm to find
  Strongly Connected Components (SCCs).

  Union-Find: Process all edges. Vertices with same Find() root
  are in the same component.
```

---

## 11.5 Backtracking — Systematic Search with Pruning

### 11.5.1 The Core Concept

Backtracking systematically explores all possible solutions by building candidates incrementally and **abandoning** ("pruning") a candidate as soon as it determines the candidate cannot lead to a valid solution.

```
  Backtracking = DFS on a decision tree + pruning

  ┌──────────────────────────────────────────────────────────────┐
  │                         ROOT                                 │
  │                      ╱       ╲                               │
  │                 Choice A    Choice B                         │
  │                ╱    ╲      ╱    ╲                            │
  │             A1       A2  B1     B2                           │
  │            ╱  ╲     ✗   ╱  ╲    ✗                           │
  │          A1a  A1b      B1a  B1b                              │
  │          ✓     ✗       ✗    ✓                                │
  │                                                              │
  │  ✓ = valid solution found                                    │
  │  ✗ = pruned (constraint violated, no need to explore further)│
  │                                                              │
  │  Without pruning: explore ALL leaves (brute force)           │
  │  With pruning: skip entire subtrees → much faster            │
  └──────────────────────────────────────────────────────────────┘
```

**The Backtracking Template:**

```
  function backtrack(candidate, constraints):
    if candidate is a complete solution:
      record/return it
      return

    for each possible next choice:
      if choice violates constraints:
        PRUNE — skip this entire branch
        continue

      make the choice                    ← "choose"
      backtrack(updated candidate)       ← "explore"
      undo the choice                    ← "un-choose" (backtrack!)
```

### 11.5.2 Classic Backtracking Problems

**N-Queens** — "Place N queens on an N×N board so no two attack each other"

```
  4-Queens solution:

  . Q . .     Place queens row by row.
  . . . Q     At each row, try each column.
  Q . . .     If column is safe (no conflict with
  . . Q .     previous queens), recurse to next row.
              If no column works, BACKTRACK to previous row.

  Constraints to check:
  • Same column? → track used columns
  • Same diagonal? → track row-col and row+col values
  • Same row? → placing one per row prevents this

  Time: O(N!) worst case, much less with pruning
```

**Sudoku Solver** — "Fill a 9×9 grid following Sudoku rules"

```
  1. Find the next empty cell
  2. Try digits 1-9
  3. For each digit:
     - Check if valid (no repeat in row, column, or 3×3 box)
     - If valid, place digit and recurse to next empty cell
     - If recursion fails, remove digit (backtrack) and try next
  4. If no digit works → return false (trigger backtracking in caller)
  5. If no empty cells remain → SOLVED!

  Pruning: Skip digits that violate Sudoku constraints immediately
```

**Permutations** — "Generate all orderings of [1, 2, 3]"

```
  backtrack([], [1,2,3]):
    choose 1 → backtrack([1], [2,3])
      choose 2 → backtrack([1,2], [3])
        choose 3 → result: [1,2,3] ✓
      choose 3 → backtrack([1,3], [2])
        choose 2 → result: [1,3,2] ✓
    choose 2 → backtrack([2], [1,3])
      choose 1 → [2,1,3] ✓
      choose 3 → [2,3,1] ✓
    choose 3 → backtrack([3], [1,2])
      choose 1 → [3,1,2] ✓
      choose 2 → [3,2,1] ✓

  6 permutations = 3! ✓
  Time: O(n × n!)
```

**Subset Sum** — "Find all subsets that sum to a target"

```
  set = {3, 34, 4, 12, 5, 2}, target = 9

  For each element: either INCLUDE it or EXCLUDE it (binary choice tree)

  Include 3 → remaining target: 6
    Include 4 → remaining: 2
      Include 2 → remaining: 0 → FOUND: {3, 4, 2} ✓
    Include 5 → remaining: 1
      Include 2 → remaining: -1 → PRUNE (exceeded target)
  ...

  Pruning: If current sum > target, stop exploring that branch

  Time: O(2ⁿ) worst case, pruning helps significantly
```

### 11.5.3 When to Use Backtracking

```
  ┌──────────────────────────────────────────────────────────────┐
  │  USE BACKTRACKING WHEN:                                      │
  │                                                              │
  │  1. You need ALL solutions (permutations, combinations)      │
  │  2. There are CONSTRAINTS to satisfy (N-Queens, Sudoku)      │
  │  3. No greedy or DP pattern exists                          │
  │  4. The search space is finite and can be pruned             │
  │  5. You need to "try everything" but intelligently           │
  │                                                              │
  │  DO NOT USE WHEN:                                            │
  │                                                              │
  │  1. Greedy gives optimal solution → much faster              │
  │  2. DP applies → avoids re-exploring the same states         │
  │  3. Search space is too large even with pruning              │
  └──────────────────────────────────────────────────────────────┘
```

---

## 11.6 Problem-Solving Frameworks — The Meta-Algorithm

### 11.6.1 The Paradigm Selection Framework

When facing an unfamiliar problem, classify it:

```
  ┌────────────────────────────────────────────────────────────────────┐
  │                    GIVEN AN UNKNOWN PROBLEM                        │
  │                          ↓                                         │
  │              Does it involve a GRAPH?                              │
  │             ╱                       ╲                              │
  │           YES                        NO                            │
  │            ↓                          ↓                            │
  │    What do you need?          Is it an OPTIMIZATION problem?       │
  │    ╱      |       ╲           ╱                    ╲               │
  │ Shortest Ordering Cheapest  YES                    NO              │
  │  path     (topo   connect    ↓                      ↓              │
  │    ↓     sort)    (MST)   Does greedy             Enumeration?     │
  │ Dijkstra   ↓       ↓     choice property          (all solutions) │
  │ or BF   Kahn's  Prim's   hold?                       ↓            │
  │         or DFS  or         ╱      ╲              BACKTRACKING      │
  │                Kruskal's YES       NO                              │
  │                           ↓        ↓                               │
  │                        GREEDY     DP                               │
  │                                   (overlapping subproblems?)       │
  │                                   ╱              ╲                 │
  │                                 YES               NO               │
  │                                  ↓                 ↓               │
  │                              Use DP          Divide & Conquer      │
  └────────────────────────────────────────────────────────────────────┘
```

### 11.6.2 Palantir's Decomposition Approach

The DS interview evaluates your ability to apply **computational thinking** to vague problems:

```
  THE FOUR PILLARS OF COMPUTATIONAL THINKING:

  1. DECOMPOSITION
     Break a complex problem into smaller, manageable sub-problems.
     "How would you build a fraud detection system?"
     → Data ingestion → Feature engineering → Model training
       → Real-time scoring → Alert generation → Case management

  2. PATTERN RECOGNITION
     Identify similarities to known problems.
     "Optimize delivery routes" → Shortest path problem → Dijkstra's
     "Schedule non-overlapping meetings" → Interval scheduling → Greedy
     "Allocate limited budget to maximize ROI" → Knapsack → DP

  3. ABSTRACTION
     Focus on what matters, ignore what doesn't.
     A supply chain is really a weighted DAG.
     Customer segmentation is really a clustering (graph) problem.
     Pipeline scheduling is really a topological sort.

  4. ALGORITHM DESIGN
     Choose the right paradigm, reason about trade-offs.
     "We could use a greedy heuristic (fast, good enough) or DP (optimal, slower)."
     "This is NP-hard, so we need an approximation algorithm."
```

### 11.6.3 How to Approach a Problem You've Never Seen

```
  Step 1: CLASSIFY
          • Graph problem? → Which graph algorithm?
          • Optimization? → Greedy or DP?
          • Enumeration? → Backtracking?
          • Sorting/searching variant?

  Step 2: SIMPLIFY
          • Solve for n=1, n=2, n=3 by hand
          • Can you see a pattern?
          • Can you build solutions for n from solutions for n-1?

  Step 3: IDENTIFY SUBPROBLEMS
          • What are the choices at each step?
          • What information do you need to make each choice?
          • That information = your DP state

  Step 4: CONSIDER TRADE-OFFS
          • Greedy: faster but may not be optimal
          • DP: optimal but slower and more memory
          • Backtracking: complete but potentially exponential
          • Approximation: when exact solution is intractable

  Step 5: VALIDATE
          • Test with small examples
          • Check edge cases
          • Verify complexity matches constraints
```

---

## 11.7 Connecting It All — Algorithms at Palantir

| Palantir Scenario | Algorithm Paradigm | Specific Algorithm |
|---|---|---|
| "Order pipeline transforms in Foundry" | Graph → ordering | Topological Sort |
| "Find shortest supply chain route" | Graph → shortest path | Dijkstra's Algorithm |
| "Connect all warehouses with minimum cable" | Graph → minimum connectivity | Prim's or Kruskal's (MST) |
| "Allocate limited budget to maximize impact" | Optimization → choices with constraints | 0/1 Knapsack (DP) |
| "How many ways can data flow through the system?" | Counting → overlapping paths | Dynamic Programming |
| "Recommend the best set of alerts to fire" | Optimization → locally optimal | Greedy (if valid) or DP |
| "Schedule non-overlapping analyst tasks" | Optimization → interval selection | Activity Selection (Greedy) |
| "Find all possible entity resolution matches" | Enumeration → all valid combinations | Backtracking |
| "Detect circular dependencies in a pipeline" | Graph → cycle detection | DFS coloring or Kahn's |
| "Find communities in a relationship graph" | Graph → connected components | Union-Find or DFS/BFS |

---

## 11.8 Glossary

| Term | Definition |
|---|---|
| **Activity selection** | Greedy problem: select max non-overlapping activities by choosing earliest finish time |
| **Backtracking** | Systematic exploration of all candidates with pruning of invalid branches |
| **Bellman-Ford** | Shortest path algorithm handling negative weights; O(V×E); detects negative cycles |
| **Coin change** | DP problem: find minimum coins to make a target amount |
| **Connected components** | Maximal subgraphs where every pair of vertices is reachable from each other |
| **Cycle detection** | Identifying cycles in graphs using DFS coloring, Union-Find, or Kahn's |
| **DAG** | Directed Acyclic Graph — a directed graph with no cycles |
| **Dijkstra's algorithm** | Greedy shortest path algorithm for non-negative weights; O((V+E) log V) |
| **Dynamic programming** | Optimization technique using overlapping subproblems and optimal substructure |
| **Edit distance** | DP problem: minimum single-character operations to transform one string to another |
| **Fractional knapsack** | Knapsack variant where items can be divided; solvable by greedy |
| **Greedy algorithm** | Makes locally optimal choices at each step; works only when greedy choice property holds |
| **Greedy choice property** | Locally optimal choices lead to globally optimal solution |
| **Huffman coding** | Greedy data compression algorithm building optimal prefix codes |
| **Kahn's algorithm** | BFS-based topological sort using in-degree computation |
| **Knapsack (0/1)** | DP problem: maximize value of items without exceeding weight capacity; items are indivisible |
| **Kruskal's algorithm** | MST algorithm: sort edges, add cheapest non-cycle edges using Union-Find |
| **LCS** | Longest Common Subsequence — DP problem finding longest shared subsequence of two strings |
| **LIS** | Longest Increasing Subsequence — DP problem finding longest strictly increasing subsequence |
| **Memoization** | Top-down DP: recursion with cached results in a hash map or array |
| **MST** | Minimum Spanning Tree — tree connecting all vertices with minimum total edge weight |
| **Negative cycle** | A cycle in a graph whose total edge weight is negative; makes shortest path undefined |
| **N-Queens** | Backtracking problem: place N queens on N×N board with no mutual attacks |
| **Optimal substructure** | Property where optimal solution contains optimal solutions to subproblems |
| **Overlapping subproblems** | Property where same subproblems are solved repeatedly in recursion |
| **Path compression** | Union-Find optimization: point nodes directly to root during Find |
| **Prim's algorithm** | MST algorithm: grow tree from a vertex, always add cheapest connecting edge |
| **Pruning** | Eliminating branches of the search tree that cannot lead to valid solutions |
| **Tabulation** | Bottom-up DP: iteratively fill a table from base cases to the answer |
| **Topological sort** | Linear ordering of DAG vertices where u comes before v for all edges u→v |
| **Union-Find** | Data structure supporting Find (which set?) and Union (merge sets) in near O(1) amortized |
| **Union by rank** | Union-Find optimization: attach smaller tree under larger tree's root |

---

## 11.9 Chapter Summary — The "Cheat Sheet"

```
┌─────────────────────────────────────────────────────────────────┐
│  CHAPTER 11 SUMMARY CARD                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DYNAMIC PROGRAMMING:                                            │
│  • Requires: overlapping subproblems + optimal substructure      │
│  • Top-down = recursion + memoization (cache)                    │
│  • Bottom-up = iteration + tabulation (table)                    │
│  • Framework: define STATE → TRANSITION → BASE CASE → ANSWER    │
│  • Classic: Fibonacci, Coin Change, Knapsack, LCS, Edit Dist    │
│  • Signal: "How many ways…", "Minimum cost to…", "Is possible…" │
│                                                                  │
│  GREEDY:                                                         │
│  • Locally optimal → globally optimal (when property holds)      │
│  • Classic: Activity selection, Fractional knapsack, Huffman     │
│  • FAILS on: 0/1 knapsack, non-canonical coins, TSP             │
│  • Faster than DP but only correct for specific problems         │
│                                                                  │
│  GRAPH ALGORITHMS:                                               │
│  • Shortest path: BFS (unweighted), Dijkstra (non-neg, E log V),│
│    Bellman-Ford (neg weights, V×E, detects neg cycles)           │
│  • Topological sort: Kahn's (BFS) or DFS; DAGs only; O(V+E)     │
│  • MST: Prim's (grow tree, dense), Kruskal's (sort edges, sparse)│
│  • Union-Find: Find + Union in O(α(n)) ≈ O(1) amortized         │
│  • Cycle detection: DFS coloring, Union-Find, or Kahn's          │
│                                                                  │
│  BACKTRACKING:                                                   │
│  • Systematic DFS on decision tree + pruning                     │
│  • Template: choose → explore → un-choose                        │
│  • Classic: N-Queens, Sudoku, permutations, subset sum           │
│  • Use when: no greedy/DP pattern; need ALL solutions            │
│                                                                  │
│  PARADIGM SELECTION:                                             │
│  • Graph? → which graph algorithm                                │
│  • Optimization + greedy property? → Greedy                      │
│  • Optimization + overlapping subproblems? → DP                  │
│  • Enumeration/constraint satisfaction? → Backtracking           │
│  • Independent subproblems? → Divide & Conquer                   │
│                                                                  │
│  PALANTIR CONNECTION:                                            │
│  • Foundry pipeline = DAG → topological sort                     │
│  • Supply chain routing = shortest path → Dijkstra's             │
│  • Network design = MST → Prim's/Kruskal's                      │
│  • Budget optimization = knapsack → DP                           │
│  • Task scheduling = interval scheduling → Greedy                │
│  • Entity resolution = matching → Backtracking                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11.10 What's Next

With Chapters 10–11, you now have the **complete CS theory toolkit** for the Deployment Strategist interview. In **Chapter 12: Data Modeling I — Schema Design, Normalization, and ER Diagrams**, we pivot from algorithms to data architecture — learning how to model the real-world entities and relationships that form the foundation of every Palantir Ontology.

---

*"The best Deployment Strategists don't memorize algorithms — they recognize patterns. When a client says 'we need to optimize delivery routes,' the DS hears 'shortest path.' When they say 'schedule these tasks with dependencies,' the DS hears 'topological sort.' That pattern recognition is what this chapter builds."*
