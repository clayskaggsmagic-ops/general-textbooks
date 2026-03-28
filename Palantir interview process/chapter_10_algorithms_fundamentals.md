# Chapter 10: Algorithms I — Complexity, Sorting, Searching, and Recursion

---

## 10.1 Why This Matters for a Deployment Strategist

Chapters 8–9 covered the *containers* — data structures that hold information. This chapter is about the *operations* — the algorithms that transform, search, and sort that information.

This matters for Palantir because:

1. **Decomposition is the DS superpower.** The interview tests whether you can break a vague problem into structured, solvable pieces. Algorithm design is literally the discipline of structured decomposition
2. **Pipeline efficiency is your responsibility.** When a Foundry pipeline takes 14 hours and the client wants it in 2, you need to understand *why* something is slow — and complexity analysis is the vocabulary for that conversation
3. **SQL optimization is algorithm optimization.** Every `ORDER BY` is a sort, every `WHERE` with an index is a binary search, every `GROUP BY` is a hash aggregation. The query optimizer chooses algorithms; you need to understand what it chose and why
4. **Sorting and searching are everywhere.** Entity resolution sorts and matches records. Gotham searches relationship graphs. AIP retrieves and ranks relevant objects. The algorithms in this chapter are the building blocks

> **Key insight:** You won't be asked to implement Quick Sort on a whiteboard. You *will* be asked to reason about why a client's pipeline is slow, what complexity means in practice, and how to decompose a messy problem into tractable pieces.

---

## 10.2 Algorithmic Complexity — The Language of Efficiency

### 10.2.1 What is Big O Notation?

Big O describes how an algorithm's resource usage (time or space) **grows** as the input size increases. It captures the **worst-case upper bound** — the maximum resources an algorithm will ever need.

Big O drops constant factors and lower-order terms because we care about *scaling behavior*, not exact execution time:

```
  Actual operations: 3n² + 7n + 42
                           ↓
  Big O:             O(n²)

  Why? As n → ∞, the n² term dominates everything else.
  Whether it's 3n² or 100n², it's still O(n²).
```

### 10.2.2 The Complexity Classes — From Fast to Catastrophic

```
  TIME
   ↑
   │                                              O(n!)
   │                                          ╱
   │                                      O(2ⁿ)
   │                                   ╱
   │                              O(n²)
   │                          ╱╱
   │                    O(n log n)
   │                 ╱╱
   │            O(n)
   │         ╱╱
   │    O(log n)
   │  ╱
   │ O(1)
   └──────────────────────────────────────→ INPUT SIZE (n)
```

| Class | Name | Growth | Example | n=1,000 |
|---|---|---|---|---|
| **O(1)** | Constant | Flat — same regardless of input | Array access by index, hash table lookup | 1 |
| **O(log n)** | Logarithmic | Cuts problem in half each step | Binary search, B-tree lookup | ~10 |
| **O(n)** | Linear | Proportional to input | Linear search, single-pass scan | 1,000 |
| **O(n log n)** | Linearithmic | Slightly above linear | Merge Sort, Quick Sort (avg) | ~10,000 |
| **O(n²)** | Quadratic | Nested loops over input | Bubble Sort, comparing all pairs | 1,000,000 |
| **O(2ⁿ)** | Exponential | Doubles with each additional element | Naive Fibonacci, subset enumeration | ~10³⁰⁰ |
| **O(n!)** | Factorial | All permutations | Brute-force Traveling Salesman | ~10²⁵⁶⁷ |

**The practical dividing line:** O(n log n) is generally the threshold for "fast enough at scale." Anything O(n²) or worse becomes problematic for large datasets. Anything exponential or factorial is intractable beyond tiny inputs.

### 10.2.3 Big O, Big Theta, Big Omega — The Full Picture

Big O is the most commonly used notation, but it's not the only one:

| Notation | Symbol | Meaning | Analogy |
|---|---|---|---|
| **Big O** | O(f(n)) | Upper bound — at most this fast | "This road has a speed limit of 65 mph" |
| **Big Omega** | Ω(f(n)) | Lower bound — at least this fast | "You must drive at least 25 mph" |
| **Big Theta** | Θ(f(n)) | Tight bound — exactly this growth rate | "Traffic moves at exactly 45 mph on this road" |

```
  Big Theta = Big O ∩ Big Omega

  If an algorithm is both O(n log n) and Ω(n log n),
  then it is Θ(n log n) — a tight bound.

  Example: Merge Sort is Θ(n log n) because its best, average,
  and worst cases are ALL n log n.

  Example: Quick Sort is O(n²) but Θ(n log n) on average.
  Its worst case (O(n²)) differs from its average case.
```

**When to use each:**

| Notation | Use when... | Example |
|---|---|---|
| **Big O** | Discussing worst-case guarantees (most common in practice) | "Binary search is O(log n)" |
| **Big Omega** | Proving theoretical lower bounds | "No comparison sort can be faster than Ω(n log n)" |
| **Big Theta** | The best and worst case are the same — precise characterization | "Merge Sort is Θ(n log n)" |

### 10.2.4 Time vs. Space Complexity

Every algorithm makes a trade-off between **speed** (time) and **memory** (space):

| Strategy | Time | Space | Example |
|---|---|---|---|
| **Compute on the fly** | Slower | Less memory | Recalculate values each time they're needed |
| **Cache/memoize results** | Faster | More memory | Store computed results in a hash table |
| **In-place algorithms** | Same | O(1) extra | Heap Sort — sorts within the array |
| **Out-of-place algorithms** | Same | O(n) extra | Merge Sort — needs temporary arrays |

**The classic trade-off in practice:**

```
  Naive Fibonacci (recursive):   Time: O(2ⁿ)    Space: O(n)     ← Very slow
  Memoized Fibonacci:            Time: O(n)      Space: O(n)     ← Fast, uses memory
  Iterative Fibonacci:           Time: O(n)      Space: O(1)     ← Fast AND lean

  You traded space (memoization table) for time (avoiding redundant work).
```

### 10.2.5 Amortized Analysis — When Worst-Case Lies

Sometimes, the worst-case for a *single* operation is misleading because expensive operations happen rarely and are "paid for" by many cheap operations.

**The Dynamic Array Example:**

```
  Appending to a dynamic array (Python list, Java ArrayList):

  Most appends are O(1) — just write to the next slot.
  Occasionally, the array is full → allocate 2x space and copy everything → O(n).

  But this expensive copy happens so rarely that, spread out over n appends,
  the AMORTIZED cost per append is O(1).

  Append sequence (capacity doubles at powers of 2):
  ┌─────────────────────────────────────────────────────────┐
  │ Operation:  1  2  3  4  5  6  7  8  9  ...             │
  │ Cost:       1  1  1  [3] 1  1  1  [5] 1  ...           │
  │                      ↑ copy 2        ↑ copy 4           │
  │                        + insert         + insert         │
  │                                                         │
  │ Total cost for n appends ≈ 3n → Amortized O(1) each    │
  └─────────────────────────────────────────────────────────┘
```

**Three Methods of Amortized Analysis:**

| Method | Idea | Analogy |
|---|---|---|
| **Aggregate** | Sum total cost of n operations, divide by n | "Total bill ÷ number of meals = average cost per meal" |
| **Accounting (Banker's)** | Overcharge cheap operations, save credits for expensive ones | "Pay $5 per meal at the cafeteria; credits cover the occasional $50 banquet" |
| **Potential** | Define a "stored energy" function that absorbs expensive operations | "Stretch a rubber band slowly (cheap); release (expensive) but energy was stored" |

**Hash table insertions** follow the same pattern: most inserts are O(1), but occasionally the table must resize (rehash all elements). Amortized cost: O(1).

---

## 10.3 Sorting Algorithms

### 10.3.1 Properties of Sorting Algorithms

Before comparing algorithms, understand four key properties:

| Property | Definition | Why It Matters |
|---|---|---|
| **Stability** | Preserves relative order of elements with equal keys | Essential when sorting by multiple criteria (sort by name, then by age — names should stay ordered within same age) |
| **In-place** | Uses O(1) extra memory (sorts within the array) | Matters when memory is constrained |
| **Adaptive** | Runs faster on partially sorted input | Matters when data is often nearly sorted (e.g., streaming data with minor reordering) |
| **Comparison-based** | Compares elements using ≤, ≥, <, > | All comparison sorts have a theoretical lower bound of Ω(n log n) |

### 10.3.2 Simple Sorts — O(n²)

These are educational algorithms — rarely used in production for large datasets, but understanding them builds intuition.

**Bubble Sort** — repeatedly swap adjacent elements that are out of order:

```
  Pass 1: [5, 3, 8, 1, 2] → [3, 5, 1, 2, 8]   (8 "bubbles" to the end)
  Pass 2: [3, 5, 1, 2, 8] → [3, 1, 2, 5, 8]
  Pass 3: [3, 1, 2, 5, 8] → [1, 2, 3, 5, 8]   ✓ sorted

  Best: O(n) if already sorted (with early-stop optimization)
  Average/Worst: O(n²)
  Stable: Yes | In-place: Yes | Adaptive: Yes
```

**Selection Sort** — find the minimum, place it, repeat:

```
  [5, 3, 8, 1, 2] → find min (1), swap with pos 0 → [1, 3, 8, 5, 2]
  [1, |3, 8, 5, 2] → find min (2), swap with pos 1 → [1, 2, 8, 5, 3]
  [1, 2, |8, 5, 3] → find min (3), swap with pos 2 → [1, 2, 3, 5, 8] ✓

  Best/Average/Worst: O(n²) — always scans the remaining array
  Stable: No (swaps can change relative order of equal elements)
  In-place: Yes | Adaptive: No
```

**Insertion Sort** — build a sorted portion by inserting each element in its correct position:

```
  [5, |3, 8, 1, 2] → insert 3 → [3, 5, |8, 1, 2]
  [3, 5, |8, 1, 2] → insert 8 → [3, 5, 8, |1, 2]
  [3, 5, 8, |1, 2] → insert 1 → [1, 3, 5, 8, |2]
  [1, 3, 5, 8, |2] → insert 2 → [1, 2, 3, 5, 8] ✓

  Best: O(n) — if already sorted, just one comparison per element
  Average/Worst: O(n²)
  Stable: Yes | In-place: Yes | Adaptive: Yes

  KEY INSIGHT: Insertion Sort is the fastest simple sort for NEARLY SORTED data.
  It's actually used as a subroutine inside more complex sorts (e.g., Timsort
  uses Insertion Sort for small runs).
```

### 10.3.3 Efficient Sorts — O(n log n)

These are the workhorses of real-world sorting.

**Merge Sort** — divide in half, sort each half, merge the sorted halves:

```
  [38, 27, 43, 3, 9, 82, 10]
           ↓ split
  [38, 27, 43, 3]    [9, 82, 10]
       ↓ split            ↓ split
  [38, 27] [43, 3]   [9, 82] [10]
     ↓       ↓         ↓       ↓
  [27,38] [3,43]    [9,82]   [10]
       ↓ merge          ↓ merge
  [3, 27, 38, 43]   [9, 10, 82]
           ↓ merge
  [3, 9, 10, 27, 38, 43, 82] ✓

  ALL CASES: O(n log n) — guaranteed
  Space: O(n) — needs temporary arrays for merging
  Stable: Yes | In-place: No | Adaptive: No
```

| Strength | Weakness |
|---|---|
| **Guaranteed O(n log n)** — no worst case degradation | O(n) extra space |
| **Stable** — preserves equal-element order | Not in-place |
| Works well on linked lists (no random access needed) | Extra allocation overhead |

**Quick Sort** — pick a pivot, partition into smaller/larger, recurse:

```
  [38, 27, 43, 3, 9, 82, 10]   pivot = 27
                ↓ partition
  [3, 9, 10] [27] [43, 82, 38]
       ↓              ↓
  [3, 9, 10]    [38, 43, 82]    recurse on each side
       ↓              ↓
  [3, 9, 10]    [38, 43, 82]    ✓

  Average: O(n log n)
  Worst: O(n²) — when pivot is always min or max (sorted input + bad pivot)
  Space: O(log n) average (recursion stack), O(n) worst
  Stable: No | In-place: Yes | Adaptive: Partially
```

| Strength | Weakness |
|---|---|
| **Fastest in practice** — excellent cache locality | O(n²) worst case (mitigated by random pivots) |
| **In-place** — O(1) extra memory (ignoring stack) | Not stable |
| Small constant factors | Pivot choice is critical |

**Why Quick Sort is faster in practice:** Although Merge Sort has better worst-case guarantees, Quick Sort's sequential access pattern makes it extremely cache-friendly. On modern hardware, cache performance often matters more than raw comparison count.

**Heap Sort** — build a max-heap, repeatedly extract the maximum:

```
  [38, 27, 43, 3, 9, 82, 10]
           ↓ build max-heap
  [82, 27, 43, 3, 9, 38, 10]
           ↓ extract max repeatedly
  [3, 9, 10, 27, 38, 43, 82] ✓

  ALL CASES: O(n log n) — guaranteed
  Space: O(1) — in-place
  Stable: No | In-place: Yes | Adaptive: No
```

| Strength | Weakness |
|---|---|
| **O(n log n) guaranteed** — no degradation | Slower constant factors than Quick Sort |
| **O(1) space** — fully in-place | Not stable |
| No worst-case pathological inputs | Poor cache locality (heap jumps around memory) |

### 10.3.4 The Efficient Sort Decision Table

| Need | Choose | Why |
|---|---|---|
| **Fastest general-purpose sort** | Quick Sort (with random pivot) | Best cache performance, small constants |
| **Guaranteed worst-case O(n log n)** | Merge Sort or Heap Sort | No pathological inputs |
| **Stability required** | Merge Sort | Only stable O(n log n) comparison sort |
| **Minimal memory** | Heap Sort | O(1) space, O(n log n) guaranteed |
| **Nearly sorted data** | Insertion Sort or Timsort | Adaptive — O(n) on sorted input |
| **What your language actually uses** | Timsort (Python, Java) or Introsort (C++) | Hybrid algorithms combining the best properties |

### 10.3.5 Linear Sorts — Breaking the O(n log n) Barrier

Comparison-based sorts cannot do better than O(n log n) — this is a proven lower bound. But if you know something about your data's structure, you can sort in O(n) by not comparing elements at all.

**Counting Sort** — count occurrences, place directly:

```
  Input: [4, 2, 2, 8, 3, 3, 1]     Range: 1-8

  Count array:   [0, 1, 2, 2, 1, 0, 0, 0, 1]
  Index:          0  1  2  3  4  5  6  7  8

  Output: [1, 2, 2, 3, 3, 4, 8]

  Time: O(n + k) where k = range of values
  Space: O(n + k)
  Stable: Yes
  Constraint: Only works for non-negative integers with bounded range
  "O(n)" when: k = O(n) — i.e., the range is proportional to the count
```

**Radix Sort** — sort digit by digit (least significant to most significant):

```
  Input: [170, 45, 75, 90, 802, 24, 2, 66]

  Sort by 1s place:  [170, 90, 802, 2, 24, 45, 75, 66]
  Sort by 10s place: [802, 2, 24, 45, 66, 170, 75, 90]
  Sort by 100s place:[2, 24, 45, 66, 75, 90, 170, 802]  ✓

  Time: O(d × (n + b)) where d = digits, b = base
  Space: O(n + b)
  Stable: Yes (requires stable sub-sort, typically Counting Sort)
  Constraint: Integer keys (or fixed-length strings)
  "O(n)" when: d is constant and b = O(n)
```

**Bucket Sort** — distribute elements into buckets, sort each bucket:

```
  Input: [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]

  Bucket 0 (0.0–0.1): []
  Bucket 1 (0.1–0.2): [0.17, 0.12]
  Bucket 2 (0.2–0.3): [0.26, 0.21, 0.23]
  Bucket 3 (0.3–0.4): [0.39]
  ...
  Bucket 9 (0.9–1.0): [0.94]

  Sort each bucket (e.g., with Insertion Sort), concatenate.

  Average: O(n + k) — when data is uniformly distributed
  Worst: O(n²) — when all elements land in one bucket
  Space: O(n + k)
  Constraint: Uniformly distributed data over a known range
```

### 10.3.6 The Grand Sorting Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable | In-place | Adaptive |
|---|---|---|---|---|---|---|---|
| **Bubble Sort** | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ | ✅ |
| **Selection Sort** | O(n²) | O(n²) | O(n²) | O(1) | ❌ | ✅ | ❌ |
| **Insertion Sort** | O(n) | O(n²) | O(n²) | O(1) | ✅ | ✅ | ✅ |
| **Merge Sort** | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | ❌ | ❌ |
| **Quick Sort** | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | ✅ | Partial |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | ✅ | ❌ |
| **Counting Sort** | O(n+k) | O(n+k) | O(n+k) | O(n+k) | ✅ | ❌ | ❌ |
| **Radix Sort** | O(d(n+b)) | O(d(n+b)) | O(d(n+b)) | O(n+b) | ✅ | ❌ | ❌ |
| **Bucket Sort** | O(n+k) | O(n+k) | O(n²) | O(n+k) | ✅* | ❌ | ❌ |

*\* If using a stable sub-sort*

---

## 10.4 Searching Algorithms

### 10.4.1 Linear Search — The Brute Force Baseline

Scan every element until you find the target or exhaust the collection.

```
  Search for 7 in [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 7]:

  Check 3? No. Check 1? No. Check 4? No. ... Check 7? YES → index 10

  Time: O(n) — must potentially check every element
  Space: O(1)
  Requires sorted data? No — works on anything
  When to use: Small datasets, unsorted data, or when you need to check
  every element anyway (e.g., finding all occurrences)
```

### 10.4.2 Binary Search — Divide and Conquer

Repeatedly halve the search space. **Requires sorted data.**

```
  Search for 23 in [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]:

  Step 1: low=0, high=9, mid=4 → arr[4]=16 < 23 → search right half
  Step 2: low=5, high=9, mid=7 → arr[7]=56 > 23 → search left half
  Step 3: low=5, high=6, mid=5 → arr[5]=23 = 23 → FOUND at index 5

  Only 3 comparisons for 10 elements!
  For 1,000,000 elements: ~20 comparisons
  For 1,000,000,000 elements: ~30 comparisons

  Time: O(log n) | Space: O(1) iterative, O(log n) recursive
```

**Iterative vs. Recursive Implementation:**

```
  ITERATIVE (preferred in practice):    RECURSIVE (more elegant):

  function binarySearch(arr, target):    function binarySearch(arr, target, lo, hi):
    lo = 0, hi = len(arr) - 1              if lo > hi: return -1
    while lo <= hi:                        mid = (lo + hi) / 2
      mid = (lo + hi) / 2                 if arr[mid] == target: return mid
      if arr[mid] == target:               if arr[mid] < target:
        return mid                           return binarySearch(arr, target, mid+1, hi)
      if arr[mid] < target:                return binarySearch(arr, target, lo, mid-1)
        lo = mid + 1
      else:
        hi = mid - 1
    return -1                            Time: O(log n) both
                                         Space: O(1) vs O(log n) ← key difference
```

| | Iterative | Recursive |
|---|---|---|
| Time | O(log n) | O(log n) |
| Space | **O(1)** | O(log n) — call stack |
| Stack overflow risk | ❌ None | ⚠️ Very deep recursion |
| Readability | Slightly longer | More concise |
| **Preferred for** | Production code | Teaching/clarity |

### 10.4.3 Binary Search on Answer — The Power Technique

Instead of searching *for an element* in an array, you search *for the answer* in a range of possible values. This converts optimization problems into binary search.

```
  Problem: "What is the minimum speed to finish all tasks in D days?"

  Answer space: [1, max_task_size]    ← possible speeds

  For each candidate speed (mid):
    • Can all tasks be completed in D days at this speed? (feasibility check)
    • If yes → try slower (search left)
    • If no  → must go faster (search right)

  The key insight: the answer has MONOTONICITY
  ┌──────────────────────────────────────────────┐
  │ Speed:    1   2   3   4   5   6   7   8   9  │
  │ Feasible? ✗   ✗   ✗   ✗   ✓   ✓   ✓   ✓   ✓ │
  │                         ↑                     │
  │                    Answer = 5                  │
  │ Binary search finds this boundary in O(log R) │
  │ where R = range of possible answers            │
  └──────────────────────────────────────────────┘
```

**When to use binary search on answer:**
- The answer lies in a numeric range
- There's a monotonic feasibility function (if X works, X+1 also works)
- Checking feasibility is efficient
- The problem asks for "minimum X such that..." or "maximum Y such that..."

### 10.4.4 Interpolation Search — Smarter Guessing

Instead of always checking the middle, interpolation search *estimates* where the target likely is, based on its value:

```
  Binary Search:        Always checks the middle
  Interpolation Search: Estimates position proportionally

  Searching for 8 in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:

  Binary:        mid = (0 + 9) / 2 = 4 → check index 4
  Interpolation: mid = 0 + (8-1)/(10-1) × (9-0) = 7 → check index 7 → FOUND!

  Like how you search a phone book: for "Smith" you open near the back,
  not the middle.
```

| | Binary Search | Interpolation Search |
|---|---|---|
| **Average** | O(log n) | **O(log log n)** for uniform data |
| **Worst** | O(log n) | O(n) for skewed data |
| **Requires** | Sorted data | Sorted + uniformly distributed |
| **Use when** | General sorted data | Large, uniformly distributed numeric keys |

### 10.4.5 Search Algorithm Summary

| Algorithm | Time (Average) | Time (Worst) | Requires Sorted? | Space |
|---|---|---|---|---|
| **Linear Search** | O(n) | O(n) | ❌ | O(1) |
| **Binary Search** | O(log n) | O(log n) | ✅ | O(1) iterative |
| **Interpolation Search** | O(log log n) | O(n) | ✅ + uniform distribution | O(1) |
| **Hash Lookup** | O(1) | O(n) | ❌ (pre-built hash table) | O(n) |

---

## 10.5 Recursion — Thinking in Self-Similarity

### 10.5.1 The Core Concept

A recursive function is one that **calls itself** with a smaller version of the problem, until it reaches a **base case** that can be solved directly.

```
  Every recursive function has two parts:

  1. BASE CASE    → The simplest version of the problem (stops recursion)
  2. RECURSIVE CASE → Breaks the problem into smaller instances of itself

  Example — Factorial:

  factorial(5) = 5 × factorial(4)
               = 5 × 4 × factorial(3)
               = 5 × 4 × 3 × factorial(2)
               = 5 × 4 × 3 × 2 × factorial(1)
               = 5 × 4 × 3 × 2 × 1          ← base case: factorial(1) = 1
               = 120
```

### 10.5.2 The Call Stack — What Actually Happens

Each recursive call pushes a new **stack frame** onto the call stack, containing the function's local variables and return address:

```
  factorial(4):

  ┌─────────────────┐
  │ factorial(1) = 1 │  ← base case reached, start returning
  ├─────────────────┤
  │ factorial(2)     │  → 2 × 1 = 2
  ├─────────────────┤
  │ factorial(3)     │  → 3 × 2 = 6
  ├─────────────────┤
  │ factorial(4)     │  → 4 × 6 = 24
  └─────────────────┘

  Stack grows DOWN as calls are made
  Results flow UP as calls return

  Each frame uses memory → too many frames = STACK OVERFLOW
  Default stack sizes: ~1MB (C/C++), ~512KB–1MB (Java/Python)
  Python default recursion limit: 1,000 calls
```

### 10.5.3 Recursive vs. Iterative — The Trade-Off

| | Recursive | Iterative |
|---|---|---|
| **Readability** | Often more natural for tree/divide-and-conquer problems | Can be harder to follow for naturally recursive problems |
| **Space** | O(depth) call stack space | O(1) if using a loop |
| **Stack overflow** | ⚠️ Risk for deep recursion | ❌ No risk |
| **Performance** | Function call overhead per frame | Generally faster (no call overhead) |
| **Debug-ability** | Stack traces show the recursion path | Loop state is simpler to inspect |
| **Best for** | Trees, divide-and-conquer, graph traversal | Flat iteration, performance-critical code |

**Rule of thumb:** Use recursion when the problem is naturally recursive (trees, graphs, divide-and-conquer). Convert to iteration when depth might be very large or performance is critical.

### 10.5.4 Classic Recursive Problems

**Fibonacci — The Cautionary Tale:**

```
  Naive recursion:
  fib(5)
  ├── fib(4)
  │   ├── fib(3)
  │   │   ├── fib(2)         Calls EXPLODE!
  │   │   │   ├── fib(1) = 1   fib(5) → 15 calls
  │   │   │   └── fib(0) = 0   fib(10) → 177 calls
  │   │   └── fib(1) = 1       fib(30) → 2,692,537 calls
  │   └── fib(2)               fib(50) → 25+ BILLION calls
  │       ├── fib(1) = 1
  │       └── fib(0) = 0   Time: O(2ⁿ) — exponential!
  └── fib(3)
      ├── fib(2)            The problem: fib(3) is computed TWICE,
      │   ├── fib(1) = 1    fib(2) THREE times, etc.
      │   └── fib(0) = 0    Massive redundant computation.
      └── fib(1) = 1
```

**Tree Traversals — Recursion's Natural Habitat:**

```
  In-order traversal of a BST (left → root → right):

  function inorder(node):
    if node is null: return     ← base case
    inorder(node.left)          ← recursive case (left subtree)
    print(node.value)           ← process current node
    inorder(node.right)         ← recursive case (right subtree)

  This naturally expresses the self-similar structure:
  "A tree is a root with two smaller trees."
```

**Merge Sort — Divide and Conquer:**

```
  function mergeSort(arr):
    if len(arr) <= 1: return arr           ← base case
    mid = len(arr) / 2
    left = mergeSort(arr[:mid])            ← sort left half
    right = mergeSort(arr[mid:])           ← sort right half
    return merge(left, right)              ← combine sorted halves
```

**Binary Search — Recursive Version:**

```
  function binarySearch(arr, target, lo, hi):
    if lo > hi: return -1                  ← base case: not found
    mid = (lo + hi) / 2
    if arr[mid] == target: return mid      ← base case: found
    if arr[mid] < target:
      return binarySearch(arr, target, mid+1, hi)   ← search right
    return binarySearch(arr, target, lo, mid-1)     ← search left
```

### 10.5.5 Memoization — The Bridge to Dynamic Programming

Memoization fixes the Fibonacci explosion by **caching results**:

```
  Memoized Fibonacci:

  cache = {}

  function fib(n):
    if n in cache: return cache[n]     ← check cache first
    if n <= 1: return n                ← base case
    result = fib(n-1) + fib(n-2)      ← compute
    cache[n] = result                  ← store in cache
    return result

  fib(5) with memoization:
  fib(5) → fib(4) → fib(3) → fib(2) → fib(1) ✓ → fib(0) ✓ → cache
         ↘ fib(3)? CACHED!

  Without memo:  O(2ⁿ) time, O(n) space
  With memo:     O(n) time,  O(n) space    ← dramatic improvement!

  fib(50):
    Without memo: 25 billion+ calls (minutes to hours)
    With memo:    50 calls (microseconds)
```

**The connection to Dynamic Programming:**

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  RECURSION          →   MEMOIZATION       →   DYNAMIC       │
  │  (top-down,              (top-down +           PROGRAMMING   │
  │   may repeat work)        caching)             (bottom-up,    │
  │                                                iterative)     │
  │                                                              │
  │  fib(n) = fib(n-1)      Same recursion,       fib = [0, 1]  │
  │         + fib(n-2)      but cache results      for i in 2..n:│
  │                                                  fib[i] =    │
  │  O(2ⁿ)                 O(n)                    fib[i-1]     │
  │                                                + fib[i-2]    │
  │                                                              │
  │                                                O(n) time     │
  │                                                O(1) space*   │
  │                                                              │
  │  * With space optimization: only keep last 2 values         │
  │                                                              │
  │  Memoization is "top-down DP" — same results, same speed.   │
  │  Chapter 11 covers the full Dynamic Programming framework.   │
  └──────────────────────────────────────────────────────────────┘
```

---

## 10.6 Connecting It All — Algorithms in SQL and Palantir

Every SQL query involves the algorithms from this chapter:

| SQL Operation | Underlying Algorithm | Complexity |
|---|---|---|
| `SELECT * FROM users WHERE id = 42` | B+ tree search (binary search variant) | O(log n) |
| `SELECT * FROM users WHERE name = 'Alice'` (no index) | Sequential scan (linear search) | O(n) |
| `ORDER BY created_at` | Sort (typically Quick Sort or Merge Sort variant) | O(n log n) |
| `GROUP BY department` | Hash aggregation (hash table build) | O(n) |
| `JOIN ... ON a.id = b.id` | Hash join, merge join, or nested loop join | O(n), O(n log n), or O(n²) |
| `DISTINCT` | Sort + deduplicate, or hash set | O(n log n) or O(n) |
| `EXPLAIN ANALYZE` output: "Sort Method: quicksort" | That's the Quick Sort from §10.3.3 | — |

When your `EXPLAIN ANALYZE` shows a "Seq Scan" on a 100M row table, you now understand: that's a linear search (O(n)) where a B+ tree search (O(log n)) should be — and you know the fix is to add an index.

---

## 10.7 Glossary

| Term | Definition |
|---|---|
| **Adaptive sort** | A sort whose performance improves when input is already partially sorted |
| **Amortized analysis** | Averaging the cost of operations over a sequence to account for rare expensive operations |
| **Base case** | The condition in a recursive function that terminates the recursion |
| **Big O** | Asymptotic upper bound on an algorithm's growth rate — worst-case behavior |
| **Big Omega (Ω)** | Asymptotic lower bound — best-case behavior |
| **Big Theta (Θ)** | Asymptotic tight bound — both upper and lower (exact growth rate) |
| **Binary search** | O(log n) search that halves the search space each step; requires sorted data |
| **Binary search on answer** | Applying binary search to a range of possible answers using a monotonic feasibility check |
| **Bubble Sort** | O(n²) sort that repeatedly swaps adjacent out-of-order elements |
| **Bucket Sort** | O(n) average sort that distributes elements into buckets; requires uniform distribution |
| **Call stack** | Memory region storing stack frames for each function call; drives recursion |
| **Comparison sort** | Any sort that determines order by comparing elements. Lower bound: Ω(n log n) |
| **Counting Sort** | O(n+k) sort for bounded non-negative integers using element frequency counts |
| **Heap Sort** | O(n log n) in-place sort using a max-heap; not stable but guaranteed performance |
| **In-place** | Algorithm that uses O(1) extra memory |
| **Insertion Sort** | O(n²) sort that builds sorted portion left-to-right; O(n) for nearly sorted data |
| **Interpolation search** | Search that estimates position by value proportion; O(log log n) for uniform data |
| **Linear search** | O(n) search that checks every element sequentially |
| **Memoization** | Caching function results to avoid redundant computation; "top-down DP" |
| **Merge Sort** | O(n log n) stable sort using divide-and-conquer; requires O(n) extra space |
| **Quick Sort** | O(n log n) average in-place sort using pivot partitioning; O(n²) worst case |
| **Radix Sort** | O(d(n+b)) sort that processes digits from least to most significant |
| **Recursion** | A function calling itself with a smaller problem until reaching a base case |
| **Recursive case** | The part of a recursive function that calls itself with a reduced problem |
| **Selection Sort** | O(n²) sort that finds the minimum and places it; not stable, not adaptive |
| **Stability** | Property of a sort that preserves relative order of elements with equal keys |
| **Stack overflow** | Error when the call stack exceeds its allocated memory (too many recursive calls) |
| **Timsort** | Hybrid sort (Merge Sort + Insertion Sort) used by Python and Java; stable, adaptive, O(n log n) |

---

## 10.8 Chapter Summary — The "Cheat Sheet"

```
┌───────────────────────────────────────────────────────────────┐
│  CHAPTER 10 SUMMARY CARD                                     │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  COMPLEXITY:                                                  │
│  • O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ)     │
│  • Big O = upper bound, Ω = lower bound, Θ = tight bound     │
│  • Time vs Space: caching trades memory for speed             │
│  • Amortized: spread rare expensive ops over many cheap ones  │
│    (dynamic array append = amortized O(1))                    │
│                                                               │
│  SORTING:                                                     │
│  • O(n²): Bubble (stable, adaptive), Selection (none),        │
│    Insertion (stable, adaptive, FAST on nearly-sorted data)   │
│  • O(n log n): Merge (stable, O(n) space), Quick (fastest     │
│    in practice, O(n²) worst), Heap (O(1) space, guaranteed)   │
│  • O(n): Counting (bounded ints), Radix (digit-by-digit),     │
│    Bucket (uniform distribution) — NOT comparison-based       │
│  • Comparison sort lower bound: Ω(n log n)                    │
│  • Real-world: Timsort (Merge+Insertion) or Introsort          │
│                                                               │
│  SEARCHING:                                                   │
│  • Linear: O(n), unsorted data                                │
│  • Binary: O(log n), sorted data, halves search space         │
│  • Binary search on answer: search answer range, not array    │
│  • Interpolation: O(log log n) for uniform data               │
│                                                               │
│  RECURSION:                                                   │
│  • Base case + recursive case                                  │
│  • Call stack → stack overflow risk                             │
│  • Recursive vs iterative: clarity vs performance              │
│  • Memoization: cache results → avoids redundant work          │
│  • Recursion + memoization = "top-down dynamic programming"    │
│  • Chapter 11 covers the full DP framework                     │
│                                                               │
│  PALANTIR CONNECTION:                                          │
│  • SQL ORDER BY = sorting algorithm                            │
│  • Index lookup = binary search on B+ tree                     │
│  • Sequential scan = linear search                             │
│  • Pipeline optimization = complexity analysis                 │
│  • Problem decomposition = recursive thinking                  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 10.9 What's Next

In **Chapter 11: Algorithms II — Dynamic Programming, Greedy Algorithms, and System Design Thinking**, we build on the recursion and memoization foundation from this chapter to tackle optimization problems systematically. We'll cover the DP framework (optimal substructure + overlapping subproblems), greedy algorithms, and the algorithmic thinking patterns that separate strong candidates in the Deployment Strategist interview.

---

*"Algorithm design is the discipline of structured decomposition — breaking a big, vague problem into small, solvable pieces. That's not just a computer science skill; it's the core of what a Deployment Strategist does every day at a client site."*
