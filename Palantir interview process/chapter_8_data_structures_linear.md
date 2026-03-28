# Chapter 8: Data Structures I — Linear Structures and Hash Tables

---

## 8.1 Why This Matters for a Deployment Strategist

You're not interviewing for a software engineering role — so why study data structures? Three reasons:

1. **Decomposition thinking.** The ability to break a problem into "what kind of container do I need?" is the same skill you use to decompose a client's data problem into "what kind of pipeline, schema, or transformation do I need?"
2. **Technical credibility.** When a client's engineer says "we're hitting hash collisions in our dedup step" or "the queue is backing up," you need to understand what they mean and respond intelligently
3. **Interview patterns.** Palantir's DS interview tests your ability to think systematically about trade-offs — the exact muscle that data structure knowledge builds

This chapter covers the five foundational structures you'll encounter everywhere: arrays, linked lists, stacks, queues, and hash tables/maps/sets.

> **Interview tip:** You likely won't be asked to implement a linked list from scratch. But being able to say "an LRU cache is a doubly-linked list plus a hash map, giving O(1) for both access and eviction" demonstrates real technical depth.

---

## 8.2 Arrays and Dynamic Arrays

### 8.2.1 The Mental Model

An array is the simplest data structure: a contiguous block of memory where each element occupies a fixed-size slot. Because the elements are physically adjacent, the computer can calculate the address of any element with simple arithmetic:

```
address(element[i]) = base_address + (i × element_size)
```

This is why array access is **O(1)** — it's just math, no searching.

### 8.2.2 Memory Layout

```
┌───────────────────────────────────────────────────────────┐
│  ARRAY: Contiguous Memory                                  │
│                                                             │
│  Memory:  ┌────┬────┬────┬────┬────┬────┬────┬────┐        │
│  Address: │1000│1004│1008│1012│1016│1020│1024│1028│        │
│  Index:   │ 0  │ 1  │ 2  │ 3  │ 4  │ 5  │ 6  │ 7  │        │
│  Value:   │ 42 │ 17 │ 93 │ 28 │ 65 │ 11 │ 84 │ 56 │        │
│           └────┴────┴────┴────┴────┴────┴────┴────┘        │
│                                                             │
│  To access index 5: address = 1000 + (5 × 4) = 1020       │
│  One calculation, one memory access → O(1)                  │
└───────────────────────────────────────────────────────────┘
```

### 8.2.3 Operations and Complexity

| Operation | Time Complexity | Why |
|---|---|---|
| Access by index | **O(1)** | Direct address calculation |
| Search (unsorted) | **O(n)** | Must check each element |
| Search (sorted) | **O(log n)** | Binary search |
| Insert at end | **O(1)** amortized | Append to back (may trigger resize) |
| Insert at beginning/middle | **O(n)** | Must shift all subsequent elements right |
| Delete at end | **O(1)** | Remove last element |
| Delete at beginning/middle | **O(n)** | Must shift all subsequent elements left |

### 8.2.4 Dynamic Arrays — The Doubling Strategy

Static arrays have a fixed size set at creation. Dynamic arrays (Python `list`, Java `ArrayList`, C++ `std::vector`) solve this by automatically resizing when full.

**How it works:**

1. Start with an internal array of some initial capacity (e.g., 4)
2. When the array is full and a new element needs to be added:
   - Allocate a new array of **double the size** (4 → 8 → 16 → 32 → ...)
   - Copy all existing elements to the new array — **O(n)** operation
   - Add the new element
   - Deallocate the old array
3. Most appends just fill an existing slot — **O(1)**

```
Append sequence for a dynamic array starting at capacity 4:

Append 1: [1, _, _, _]           → O(1) — just fill slot
Append 2: [1, 2, _, _]           → O(1)
Append 3: [1, 2, 3, _]           → O(1)
Append 4: [1, 2, 3, 4]           → O(1)
Append 5: [1, 2, 3, 4, 5, _, _, _]  → O(n)! Resize: copy 4 elements, capacity now 8
Append 6: [1, 2, 3, 4, 5, 6, _, _]  → O(1)
...
```

**Amortized O(1):** Over n appends, the total cost of all copies sums to roughly 2n. Dividing by n gives an average (amortized) cost of O(1) per append.

### 8.2.5 Cache Friendliness

Arrays are **cache-friendly** because modern CPUs load memory in blocks called "cache lines" (typically 64 bytes). When you access `array[0]`, the CPU loads `array[0]` through roughly `array[15]` into the fast L1 cache. Subsequent accesses hit the cache instead of main memory, which is 50-200x faster.

This is why iterating over an array is dramatically faster than iterating over a linked list — even though both are theoretically O(n).

### 8.2.6 Real-World Applications

| Application | Why Arrays/Dynamic Arrays |
|---|---|
| Database rows | Tables store rows contiguously for scan performance |
| Time-series data | Sensor readings, stock prices — sequential, indexed by time |
| Buffers | Network I/O, file reading — contiguous memory blocks |
| Matrices | 2D arrays for image data, spreadsheets, ML feature matrices |
| Python lists | The default container — dynamic array under the hood |

---

## 8.3 Linked Lists

### 8.3.1 The Mental Model

Instead of contiguous memory, each element ("node") is a self-contained package: it holds data *and* a pointer to the next node. The list is a chain of these nodes scattered throughout memory.

### 8.3.2 Variants

**Singly Linked List:** Each node points to the next node only. Traversal is one-way.

```
HEAD → [A | →] → [B | →] → [C | →] → [D | ∅]
        data next  data next  data next  data next=NULL
```

**Doubly Linked List:** Each node points to both the next *and* previous node. Traversal goes both ways.

```
NULL ← [← A →] ⇄ [← B →] ⇄ [← C →] ⇄ [← D →] → NULL
        prev data next
```

**Circular Linked List:** The last node points back to the first, forming a loop. Useful for round-robin scheduling.

```
→ [A | →] → [B | →] → [C | →] → [D | →] ─┐
↑                                            │
└────────────────────────────────────────────┘
```

### 8.3.3 Operations and Complexity

| Operation | Singly Linked | Doubly Linked | Why |
|---|---|---|---|
| Access by index | **O(n)** | **O(n)** | Must traverse from head |
| Search | **O(n)** | **O(n)** | Must traverse to find |
| Insert at head | **O(1)** | **O(1)** | Update head pointer |
| Insert at tail | **O(1)** with tail ptr | **O(1)** | Update tail and links |
| Insert at known position | **O(1)** | **O(1)** | Just re-link pointers |
| Delete at head | **O(1)** | **O(1)** | Update head pointer |
| Delete at known position | **O(n)** singly* | **O(1)** | *Need to find previous node in singly |
| Delete at tail | **O(n)** | **O(1)** | Singly: must find second-to-last |

*Singly linked deletion at a known position is O(n) because you need to find the *previous* node to update its pointer. Doubly linked lists give you the previous node directly.

### 8.3.4 When Linked Lists Beat Arrays

| Choose Linked List When | Choose Array When |
|---|---|
| Frequent insertions/deletions at arbitrary positions | Random access by index is common |
| Size is unknown and varies wildly | Size is known or relatively stable |
| No need for random access | Cache performance matters |
| Memory is fragmented (can't get contiguous blocks) | Sequential iteration is the primary pattern |

### 8.3.5 The LRU Cache — The Classic Combined Structure

The **Least Recently Used (LRU) cache** is the textbook example of combining data structures for optimal performance. It needs:
- **O(1) lookup** by key → Hash map
- **O(1) insertion** of new items → Doubly linked list (add to head)
- **O(1) eviction** of oldest item → Doubly linked list (remove from tail)
- **O(1) promotion** of accessed item → Doubly linked list (move to head)

```
┌──────────────────────────────────────────────────────┐
│  LRU CACHE = Hash Map + Doubly Linked List            │
│                                                        │
│  Hash Map                    Doubly Linked List        │
│  ┌─────────┬──────┐                                    │
│  │  Key A  │ ──────────→  [A] ⇄ [B] ⇄ [C] ⇄ [D]     │
│  │  Key B  │ ──────────→   ↑                    ↑      │
│  │  Key C  │ ──────────→  MRU                  LRU     │
│  │  Key D  │ ──────────→  (head)              (tail)   │
│  └─────────┴──────┘                                    │
│                                                        │
│  GET(Key B):                                           │
│    1. Hash map → find node B in O(1)                   │
│    2. Remove node B from its position in O(1)          │
│    3. Move node B to head (most recently used) in O(1) │
│                                                        │
│  Cache full + INSERT(Key E):                           │
│    1. Remove node D from tail (LRU eviction) in O(1)   │
│    2. Remove Key D from hash map in O(1)               │
│    3. Create node E, add to head in O(1)               │
│    4. Add Key E → node E to hash map in O(1)           │
│                                                        │
│  Every operation: O(1)                                  │
└──────────────────────────────────────────────────────┘
```

### 8.3.6 Real-World Applications

| Application | Why Linked Lists |
|---|---|
| LRU caches | Doubly linked list + hash map (see above) |
| Undo/redo history | Doubly linked list of application states |
| Music playlists | Circular DLL — next song, previous song, loop |
| Memory allocators | Free lists track available memory blocks |
| Blockchain | Each block points to the hash of the previous block |

---

## 8.4 Stacks — LIFO (Last In, First Out)

### 8.4.1 The Mental Model

A stack is like a stack of plates: you add to the top, remove from the top. The last plate you put on is the first one you take off.

```
         ┌─────┐
  push → │  D  │ ← pop/peek (top)
         ├─────┤
         │  C  │
         ├─────┤
         │  B  │
         ├─────┤
         │  A  │
         └─────┘
```

### 8.4.2 Operations

| Operation | Description | Time Complexity |
|---|---|---|
| `push(item)` | Add item to the top | **O(1)** |
| `pop()` | Remove and return the top item | **O(1)** |
| `peek()` / `top()` | View the top item without removing | **O(1)** |
| `isEmpty()` | Check if stack is empty | **O(1)** |
| `size()` | Number of elements | **O(1)** |

### 8.4.3 Implementations

| Implementation | Pros | Cons |
|---|---|---|
| **Array-backed** | Cache-friendly, memory-efficient (no pointer overhead), simpler | Fixed capacity (unless dynamic array is used) |
| **Linked-list-backed** | Dynamic size, never overflows | More memory per element (pointer overhead), less cache-friendly |

In practice, array-backed stacks (using dynamic arrays) are almost always preferred due to cache performance.

### 8.4.4 Applications

**The Function Call Stack:**

Every program uses a call stack to track function execution:

```
main() calls → processData() calls → validateInput() calls → checkFormat()

Call Stack:                    
┌──────────────┐              
│ checkFormat()│  ← Currently executing
├──────────────┤              
│validateInput()│  ← Will resume after checkFormat returns
├──────────────┤              
│processData() │  ← Will resume after validateInput returns
├──────────────┤              
│   main()     │  ← Will resume after processData returns
└──────────────┘

When checkFormat() returns → popped off → validateInput() resumes
When too many nested calls → "Stack Overflow" error
```

**Expression Evaluation (Postfix):**

```
Infix:    3 + 4 × 2
Postfix:  3 4 2 × +

Processing postfix with a stack:
  Read 3  → push 3            Stack: [3]
  Read 4  → push 4            Stack: [3, 4]
  Read 2  → push 2            Stack: [3, 4, 2]
  Read ×  → pop 2 and 4,      Stack: [3]
             push (4×2=8)      Stack: [3, 8]
  Read +  → pop 8 and 3,      Stack: []
             push (3+8=11)     Stack: [11]
  Result: 11
```

**Balanced Parentheses:**

```
Input: "({[A + B] × (C - D)})"

Process:
  (  → push           Stack: [(]
  {  → push           Stack: [(, {]
  [  → push           Stack: [(, {, []
  ]  → pop [, match!  Stack: [(, {]
  (  → push           Stack: [(, {, (]
  )  → pop (, match!  Stack: [(, {]
  }  → pop {, match!  Stack: [(]
  )  → pop (, match!  Stack: []

Stack empty at end → BALANCED ✓
```

**DFS Traversal:** Iterative Depth-First Search uses an explicit stack. Push a node, pop it, push its unvisited neighbors, repeat.

---

## 8.5 Queues — FIFO (First In, First Out)

### 8.5.1 The Mental Model

A queue is like a line at a store: first person in line is the first person served.

```
  enqueue →  ┌─────┬─────┬─────┬─────┐  → dequeue
  (rear)     │  D  │  C  │  B  │  A  │    (front)
             └─────┴─────┴─────┴─────┘
```

### 8.5.2 Operations

| Operation | Description | Time Complexity |
|---|---|---|
| `enqueue(item)` | Add item to the rear | **O(1)** |
| `dequeue()` | Remove and return item from the front | **O(1)** |
| `peek()` / `front()` | View the front item without removing | **O(1)** |
| `isEmpty()` | Check if queue is empty | **O(1)** |
| `size()` | Number of elements | **O(1)** |

### 8.5.3 Queue Variants

**Circular Queue (Ring Buffer):**

In a linear array-based queue, dequeuing wastes space — the front moves forward but the vacated slots can't be reused. A circular queue wraps the array around so the rear pointer follows the front pointer in a loop.

```
Linear Queue Problem:               Circular Queue Solution:
[_, _, _, C, D, E]                   Front →  C → D → E
 ↑ wasted space!                               ↑         ↓
                                               ← new items fill here
```

Use cases: fixed-size buffers, streaming data, embedded systems.

**Deque (Double-Ended Queue):**

A deque allows insertion and removal at *both* ends:

```
push_front →  ┌─────┬─────┬─────┐  ← push_back
pop_front  →  │  A  │  B  │  C  │  ← pop_back
              └─────┴─────┴─────┘
```

A deque can act as both a stack (use one end) and a queue (use both ends). Python's `collections.deque` is a deque — it's the recommended structure for queues in Python.

**Priority Queue:**

Elements are served by **priority**, not arrival order. Internally implemented using a **binary heap** (a complete binary tree stored in an array).

| Operation | Time Complexity |
|---|---|
| Insert | **O(log n)** — must maintain heap property |
| Extract highest priority | **O(log n)** — must re-heapify |
| Peek highest priority | **O(1)** — always at the root |

Use cases: Dijkstra's algorithm, task scheduling by urgency, event-driven simulations.

### 8.5.4 Applications

| Application | Queue Type | Why |
|---|---|---|
| BFS graph traversal | FIFO queue | Explore nodes level by level |
| Task scheduling (OS) | Priority queue | Higher-priority processes run first |
| Print queue | FIFO queue | Jobs processed in submission order |
| Message buffering (Kafka, RabbitMQ) | FIFO queue | Async producers and consumers |
| Rate limiting | Circular queue | Fixed window of recent requests |
| "Undo + Redo" (both directions) | Deque | Push/pop from both ends |
| Sliding window algorithms | Deque | Efficiently track min/max in window |

---

## 8.6 Hash Tables / Hash Maps

### 8.6.1 The Core Idea

A hash table maps **keys** to **values** using a hash function. The hash function converts a key into an array index, enabling near-instant lookup.

```
┌─────────────────────────────────────────────────────────┐
│  HASH TABLE CONCEPT                                      │
│                                                           │
│  Key "alice"  ──→  hash("alice") = 739284                │
│                    739284 % table_size = 2                │
│                                                           │
│  Index:  0    1    2         3    4    5                  │
│        ┌────┬────┬─────────┬────┬────┬────┐              │
│        │    │    │"alice"→ │    │    │    │              │
│        │    │    │  42     │    │    │    │              │
│        └────┴────┴─────────┴────┴────┴────┘              │
│                                                           │
│  Lookup "alice": hash → index 2 → return 42  →  O(1)    │
└─────────────────────────────────────────────────────────┘
```

### 8.6.2 Hash Functions

A good hash function must be:

| Property | Meaning |
|---|---|
| **Deterministic** | Same key always produces same hash |
| **Fast** | O(1) computation |
| **Uniform distribution** | Keys spread evenly across buckets to minimize collisions |
| **Avalanche effect** | Small input change → large output change |

Common hash functions: division method (`h(k) = k mod m`), multiplication method, MurmurHash, SipHash (Python's default), SHA-256 (cryptographic).

### 8.6.3 Collision Resolution

Two keys can hash to the same index — this is a **collision**. With a finite number of buckets and a potentially infinite number of keys, collisions are inevitable.

**Strategy 1: Chaining (Separate Chaining)**

Each bucket holds a linked list (or other collection). Colliding keys are simply appended to the list at that bucket.

```
Index 0: → NULL
Index 1: → ["bob": 25] → NULL
Index 2: → ["alice": 42] → ["charlie": 38] → NULL  ← collision!
Index 3: → ["dave": 19] → NULL
Index 4: → NULL
```

| Pros | Cons |
|---|---|
| Simple to implement | Linked list traversal on collision — O(n) worst case |
| Load factor can exceed 1.0 | Extra memory for pointers |
| Graceful degradation | Poor cache locality (pointer chasing) |

**Strategy 2: Open Addressing**

All entries stored directly in the array. On collision, probe for the next open slot.

**Linear Probing:** Check index+1, index+2, index+3, ...

```
hash("alice") = 2   →  slot 2: empty → store here
hash("charlie") = 2 →  slot 2: occupied → try 3: empty → store here
hash("eve") = 2     →  slot 2: occupied → try 3: occupied → try 4: empty → store
```

- **Problem: Primary clustering.** Long runs of occupied slots form clusters, making future insertions and lookups slower.

**Quadratic Probing:** Check index+1², index+2², index+3², ...

- Reduces primary clustering but can cause "secondary clustering" (same initial hash → same probe sequence).

**Double Hashing:** Use a second hash function to determine the step size.

```
probe(i) = (h1(key) + i × h2(key)) % table_size
```

- Best distribution — different keys get different probe sequences even if they collide on the first hash.

**Chaining vs. Open Addressing — Decision Table:**

| Factor | Chaining | Open Addressing |
|---|---|---|
| Load factor | Can exceed 1.0 | Must stay below 1.0 (ideally < 0.7) |
| Memory | Extra per pointer | Compact — no pointers |
| Cache performance | Poor (pointer chasing) | Good (data in array) |
| Deletion | Easy (remove from list) | Tricky (must use tombstones) |
| Implementation | Simpler | More complex |
| Best when | High load factor, variable-size keys | Memory-constrained, cache matters |

### 8.6.4 Load Factor and Rehashing

The **load factor** (λ) is the ratio of stored elements to table capacity:

```
λ = n / capacity

Where n = number of stored key-value pairs
```

| Load Factor Range | Impact |
|---|---|
| λ < 0.5 | Wasteful memory, but very fast |
| 0.5 < λ < 0.75 | Sweet spot — good space/speed trade-off |
| λ > 0.75 | Collisions increase, performance degrades |
| λ → 1.0 (open addressing) | Severe degradation — probe chains become very long |

**Rehashing** occurs when the load factor exceeds the threshold (commonly 0.75):

1. Allocate a new array **2x** the current size
2. Re-hash every existing key-value pair into the new array
3. Deallocate the old array

This is an **O(n)** operation, but it happens infrequently enough that the **amortized** cost per insert remains **O(1)** — the same principle as dynamic array resizing.

### 8.6.5 Complexity Summary

| Operation | Average Case | Worst Case (all collisions) |
|---|---|---|
| Insert | **O(1)** | O(n) |
| Lookup | **O(1)** | O(n) |
| Delete | **O(1)** | O(n) |
| Space | O(n) | O(n) |

The worst case occurs when every key hashes to the same bucket — effectively becoming a linked list. This is why good hash functions and proper load factor management are essential.

### 8.6.6 Real-World Applications

| Application | Why Hash Tables |
|---|---|
| Database indexes (hash index) | O(1) equality lookup for exact-match queries |
| Caches (Redis, Memcached) | O(1) key-value lookup |
| Frequency counters | Count occurrences of words, events, errors |
| Deduplication | Track "seen" items in O(1) |
| Symbol tables (compilers) | Map variable names to memory addresses |
| HTTP session storage | Map session IDs to user state |
| Python `dict`, Java `HashMap` | The default associative container |

---

## 8.7 Sets

A set is an unordered collection of **unique elements**. It's a hash table that stores only keys (no values).

### 8.7.1 Set Operations

| Operation | Notation | Description | Example |
|---|---|---|---|
| **Union** | A ∪ B | All elements in either set | {1,2,3} ∪ {3,4,5} = {1,2,3,4,5} |
| **Intersection** | A ∩ B | Elements in both sets | {1,2,3} ∩ {3,4,5} = {3} |
| **Difference** | A − B | Elements in A but not B | {1,2,3} − {3,4,5} = {1,2} |
| **Symmetric Difference** | A △ B | Elements in one but not both | {1,2,3} △ {3,4,5} = {1,2,4,5} |
| **Membership** | x ∈ A | Is x in the set? | 3 ∈ {1,2,3} → True |
| **Subset** | A ⊆ B | All elements of A are in B | {1,2} ⊆ {1,2,3} → True |

### 8.7.2 Hash Set vs. Tree Set

| | Hash Set | Tree Set |
|---|---|---|
| **Implementation** | Hash table | Self-balancing BST (Red-Black Tree) |
| **Ordering** | ❌ Unordered | ✅ Sorted (natural or custom comparator) |
| **Membership test** | **O(1)** average | **O(log n)** guaranteed |
| **Insert** | **O(1)** average | **O(log n)** |
| **Delete** | **O(1)** average | **O(log n)** |
| **Range queries** | ❌ Not supported | ✅ "All elements between X and Y" |
| **Null elements** | ✅ One null allowed | ❌ Not allowed (can't compare null) |
| **Use when** | Speed is priority, order doesn't matter | You need sorted iteration or range queries |

### 8.7.3 Set Operation Complexity

| Operation | Hash Set | Tree Set |
|---|---|---|
| Union (A ∪ B) | O(n + m) | O((n+m) log(n+m)) |
| Intersection (A ∩ B) | O(min(n,m)) | O(min(n,m) × log(max(n,m))) |
| Difference (A − B) | O(n + m) | O(n × log m) |
| Membership (x ∈ A) | O(1) | O(log n) |

### 8.7.4 Real-World Applications

| Application | Why Sets |
|---|---|
| Deduplication | Add all items to a set — duplicates automatically ignored |
| "Have we seen this before?" | O(1) membership testing |
| Finding common elements | Intersection of two sets |
| Unique visitors / unique values | Count elements in a set |
| Permission checking | Is user in the "admins" set? |
| SQL DISTINCT | Conceptually a set operation |
| Tag/label systems | Each item has a set of tags |

---

## 8.8 The Grand Comparison Table

| Structure | Access | Search | Insert | Delete | Space | Best Use Case |
|---|---|---|---|---|---|---|
| **Array** | O(1) | O(n) | O(n)* | O(n)* | O(n) | Random access by index, cache-critical iteration |
| **Dynamic Array** | O(1) | O(n) | O(1)† | O(n)* | O(n) | General-purpose list, growing collections |
| **Singly Linked List** | O(n) | O(n) | O(1)‡ | O(n) | O(n) | Stack implementation, simple forward iteration |
| **Doubly Linked List** | O(n) | O(n) | O(1)‡ | O(1)‡ | O(n) | LRU caches, undo/redo, bidirectional traversal |
| **Stack** | O(n)§ | O(n) | O(1) | O(1) | O(n) | LIFO: call stack, DFS, expression eval, undo |
| **Queue** | O(n)§ | O(n) | O(1) | O(1) | O(n) | FIFO: BFS, task scheduling, buffering |
| **Priority Queue** | O(n)§ | O(n) | O(log n) | O(log n) | O(n) | Scheduling by priority, Dijkstra's, event sim |
| **Hash Table** | N/A | O(1)¶ | O(1)¶ | O(1)¶ | O(n) | Key-value lookup, caches, frequency counting |
| **Hash Set** | N/A | O(1)¶ | O(1)¶ | O(1)¶ | O(n) | Dedup, membership testing, set operations |
| **Tree Set** | N/A | O(log n) | O(log n) | O(log n) | O(n) | Sorted unique elements, range queries |

*\* O(n) due to shifting elements*
*† O(1) amortized at end; O(n) at beginning/middle*
*‡ O(1) given a reference to the position; O(n) to find the position*
*§ Not designed for arbitrary access — only top/front is O(1)*
*¶ Average case; worst case O(n) due to collisions*

---

## 8.9 Choosing the Right Structure — Decision Flowchart

```
┌──────────────────────────────────────────────────────┐
│  CHOOSING A DATA STRUCTURE                            │
│                                                        │
│  Do I need key-value pairs?                            │
│    YES → Hash Table / Hash Map                         │
│                                                        │
│  Do I need only unique membership testing?             │
│    YES → Hash Set (unordered) or Tree Set (sorted)     │
│                                                        │
│  Do I need LIFO access (last in, first out)?           │
│    YES → Stack                                         │
│                                                        │
│  Do I need FIFO access (first in, first out)?          │
│    YES → Queue (or Deque for both-end access)          │
│                                                        │
│  Do I need priority-based access?                      │
│    YES → Priority Queue (heap)                         │
│                                                        │
│  Do I need O(1) random access by index?                │
│    YES → Array / Dynamic Array                         │
│                                                        │
│  Do I need frequent insert/delete at known positions?  │
│    YES → Linked List (doubly for bidirectional)        │
│                                                        │
│  Default for most general-purpose needs:               │
│    → Dynamic Array (Python list, Java ArrayList)       │
└──────────────────────────────────────────────────────┘
```

---

## 8.10 Common Interview Connections

These data structures don't exist in isolation — interviewers test your ability to connect them to real-world scenarios:

| Scenario | Structure(s) | Why |
|---|---|---|
| "Design a system to detect duplicate events" | Hash Set | O(1) membership check |
| "Implement browser back/forward buttons" | Two Stacks | Push/pop between undo and redo stacks |
| "Build a task scheduler that prioritizes VIP users" | Priority Queue | Extract-max by priority level |
| "Find the most frequently accessed pages" | Hash Map + Priority Queue | Count frequencies (map), then extract top-K (heap) |
| "Design an LRU cache with O(1) everything" | Hash Map + Doubly Linked List | Map for lookup, DLL for ordering |
| "Check if two datasets share any common records" | Hash Set (intersection) | O(n+m) to find common elements |
| "Process events in order of arrival" | Queue | FIFO guarantees ordering |
| "Evaluate mathematical expressions" | Stack | Infix-to-postfix conversion, then evaluation |
| "Rate limit API requests (sliding window)" | Deque | Track timestamps, remove expired from front |

---

## 8.11 Glossary

| Term | Definition |
|---|---|
| **Amortized analysis** | Averaging the cost of expensive operations over a sequence of cheap ones. E.g., dynamic array append is amortized O(1) despite occasional O(n) resizes |
| **Binary heap** | A complete binary tree where each parent ≥ children (max-heap) or ≤ children (min-heap). Stored as an array. Used to implement priority queues |
| **Cache line** | Block of contiguous memory (typically 64 bytes) loaded into CPU cache at once. Arrays exploit this via spatial locality |
| **Chaining** | Hash collision resolution where each bucket holds a linked list of colliding entries |
| **Circular queue** | Array-based queue where the rear pointer wraps around to reuse vacated front slots |
| **Collision** | When two different keys produce the same hash table index |
| **Deque** | Double-ended queue supporting insertion and removal at both front and rear |
| **Double hashing** | Open addressing with a second hash function determining the probe step size |
| **Dynamic array** | Automatically-resizing array that doubles capacity when full |
| **FIFO** | First In, First Out — the principle governing queues |
| **Hash function** | Maps an input key to a fixed-size integer index |
| **Hash set** | A set implemented using a hash table — O(1) average for add/contains/remove |
| **Hash table** | Data structure mapping keys to values via a hash function for O(1) average access |
| **LIFO** | Last In, First Out — the principle governing stacks |
| **Linear probing** | Open addressing where you check the next sequential slot on collision. Prone to primary clustering |
| **Linked list** | Chain of nodes connected by pointers. Singly (one-way), doubly (two-way), or circular |
| **Load factor** | Ratio of stored elements to hash table capacity (λ = n / capacity). Triggers rehashing when exceeded |
| **LRU cache** | Least Recently Used cache — evicts the least recently accessed item. Implemented with a hash map + doubly linked list |
| **Open addressing** | Hash collision resolution where alternative slots in the array are probed |
| **Priority queue** | Queue where elements are served by priority, not arrival order. Implemented with a binary heap |
| **Quadratic probing** | Open addressing where the probe step grows quadratically (1, 4, 9, ...) to reduce clustering |
| **Rehashing** | Resizing a hash table and re-inserting all keys when the load factor threshold is exceeded |
| **Stack overflow** | Error when the call stack exceeds its memory limit, typically from excessive recursion |
| **Tree set** | Set implemented using a balanced BST (Red-Black Tree) — O(log n), maintains sorted order |

---

## 8.12 Chapter Summary — The "Cheat Sheet"

```
┌───────────────────────────────────────────────────────────────┐
│  CHAPTER 8 SUMMARY CARD                                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ARRAYS / DYNAMIC ARRAYS:                                     │
│  • Contiguous memory → O(1) access by index                  │
│  • Cache-friendly (spatial locality)                          │
│  • Dynamic: amortized O(1) append via doubling strategy       │
│  • Insert/delete middle: O(n) — must shift elements           │
│                                                               │
│  LINKED LISTS:                                                │
│  • Nodes scattered in memory, connected by pointers           │
│  • O(1) insert/delete at known positions                      │
│  • O(n) search — must traverse                                │
│  • Doubly linked: O(1) delete at arbitrary known node         │
│  • Classic combo: DLL + Hash Map = LRU Cache                  │
│                                                               │
│  STACKS (LIFO):                                               │
│  • Push/Pop/Peek all O(1)                                     │
│  • Call stack, DFS, expression eval, balanced parens           │
│  • Array-backed preferred for cache performance                │
│                                                               │
│  QUEUES (FIFO):                                               │
│  • Enqueue/Dequeue O(1)                                       │
│  • Circular: wraps to reuse space                             │
│  • Deque: both-end ops — Python collections.deque             │
│  • Priority Queue: binary heap, O(log n) insert/extract       │
│  • BFS, task scheduling, message buffering                    │
│                                                               │
│  HASH TABLES / MAPS:                                          │
│  • Hash function → array index → O(1) avg lookup              │
│  • Collision: Chaining (linked lists) vs Open Addressing      │
│    (linear/quadratic probing, double hashing)                 │
│  • Load factor: λ = n/capacity (threshold ~0.75)              │
│  • Rehash: double size + re-insert all → amortized O(1)       │
│  • Caches, dedup, frequency counting, indexes                 │
│                                                               │
│  SETS:                                                        │
│  • Unique elements — auto-deduplication                       │
│  • Union ∪, Intersection ∩, Difference −, Sym.Diff. △         │
│  • Hash Set: O(1) — fast, unordered                           │
│  • Tree Set: O(log n) — slower, sorted                        │
│                                                               │
│  CHOOSING:                                                    │
│  • Key-value pairs?         → Hash Map                        │
│  • Unique membership?       → Hash Set                        │
│  • LIFO?                    → Stack                           │
│  • FIFO?                    → Queue                           │
│  • Priority?                → Priority Queue (Heap)           │
│  • Random access by index?  → Array                           │
│  • Frequent insert/delete?  → Linked List                    │
│  • Default general purpose? → Dynamic Array                   │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 8.13 What's Next

In **Chapter 9: Data Structures II — Trees, Graphs, and Advanced Structures**, we move beyond linear structures into the hierarchical and networked data structures that power databases (B-trees), search engines (tries), and Palantir's own Ontology graph. These structures are foundational to understanding how complex relationships are modeled and queried.

---

*"The difference between a list, a stack, a queue, and a hash table isn't just academic — it's the difference between a solution that scales and one that crumbles."*
