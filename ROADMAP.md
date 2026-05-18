# DSA Patterns Roadmap

> "You don't learn algorithms by memorizing solutions. You learn them by recognizing which tool fits the shape of the problem."

## Philosophy

**Don't memorize solutions. Recognize patterns.**

For each pattern we will:
1. **Solve it brute-force first** — write the O(n²) or O(2ⁿ) solution that works
2. **Feel the pain** — see the TLE, the redundant work, the wasted space
3. **Discover the pattern** — introduce it as an optimization to a problem you now understand
4. **Implement it in Java** — clean, hands-on code you write yourself
5. **Analyze complexity** — time & space, best/worst/average cases

---

## The Decision Tree

Before picking a pattern, ask: **"What is the shape of this problem?"**

```
What are you working with?
│
├── ARRAY / MATRIX — linear or grid data, order matters
│   │
│   ├── Need to find a pair/triplet satisfying a condition?
│   │   └── → Two Pointers (sorted) / Hash Map (unsorted)
│   │
│   ├── Need a contiguous subarray/substring of some property?
│   │   └── → Sliding Window
│   │
│   ├── Need cumulative sums or range queries?
│   │   └── → Prefix Sum / Kadane's
│   │
│   ├── Need to find next greater/smaller element?
│   │   └── → Monotonic Stack
│   │
│   ├── Need to merge/schedule overlapping ranges?
│   │   └── → Merge Intervals
│   │
│   ├── Array contains numbers 1..n with duplicates/missing?
│   │   └── → Cyclic Sort
│   │
│   └── In-place transformation (rotate, zero-out, spiral)?
│       └── → Array & Matrix Manipulation
│
├── SORTED DATA — binary decisions possible
│   │
│   ├── Search for a target or boundary?
│   │   └── → Binary Search (on sorted array)
│   │
│   ├── Find first/last occurrence?
│   │   └── → Binary Search (boundary variant)
│   │
│   ├── Rotated sorted array?
│   │   └── → Modified Binary Search
│   │
│   └── Answer lies in a range of possible values?
│       └── → Binary Search on Answer
│
├── LINKED LIST
│   │
│   ├── Detect cycle or find middle?
│   │   └── → Fast & Slow Pointers
│   │
│   ├── Reverse a portion of the list?
│   │   └── → In-place Reversal
│   │
│   ├── Merge sorted lists?
│   │   └── → Merge Pattern
│   │
│   └── Reorder or partition nodes?
│       └── → Reordering & Partitioning
│
├── STACK problems
│   │
│   ├── Matching brackets / valid nesting?
│   │   └── → Parentheses Matching
│   │
│   ├── Next greater/smaller element?
│   │   └── → Monotonic Stack
│   │
│   ├── Largest rectangle / trapping water?
│   │   └── → Histogram Pattern (Monotonic Stack)
│   │
│   └── Evaluate or parse expressions?
│       └── → Expression Evaluation (Stack)
│
├── TREE
│   │
│   ├── Process level by level?
│   │   └── → Level-Order BFS
│   │
│   ├── Path sums, subtree problems?
│   │   └── → DFS (Preorder / Inorder / Postorder)
│   │
│   ├── Serialize / deserialize?
│   │   └── → Tree Serialization
│   │
│   └── Find ancestor relationships?
│       └── → Lowest Common Ancestor
│
├── GRAPH
│   │
│   ├── Connected components / flood fill?
│   │   └── → DFS or BFS Components
│   │
│   ├── Shortest path (unweighted)?
│   │   └── → BFS
│   │
│   ├── Shortest path (weighted)?
│   │   └── → Dijkstra's / Bellman-Ford
│   │
│   ├── Tasks with prerequisites / ordering?
│   │   └── → Topological Sort
│   │
│   ├── Cycle detection in directed graph?
│   │   └── → DFS with coloring / Topological Sort
│   │
│   ├── Dynamic connectivity / grouping?
│   │   └── → Union Find (DSU)
│   │
│   ├── Minimum cost to connect all nodes?
│   │   └── → Minimum Spanning Tree (Kruskal/Prim)
│   │
│   └── Find bridges / critical connections?
│       └── → Bridges & Articulation Points (Tarjan's)
│
├── NEED OPTIMAL SUBSTRUCTURE
│   │
│   ├── Locally optimal → globally optimal?
│   │   └── → Greedy
│   │
│   ├── Overlapping subproblems?
│   │   └── → Dynamic Programming
│   │
│   └── Need all combinations / permutations / subsets?
│       └── → Backtracking
│
├── PRIORITY / RANKING
│   │
│   ├── Top K / Kth largest?
│   │   └── → Heap (Top K Elements)
│   │
│   ├── Running median from stream?
│   │   └── → Two Heaps
│   │
│   ├── Merge K sorted sources?
│   │   └── → K-way Merge
│   │
│   └── Scheduling / minimum cost selection?
│       └── → Heap + Greedy
│
├── STRING-SPECIFIC
│   │
│   ├── Prefix matching / autocomplete?
│   │   └── → Trie
│   │
│   ├── Pattern matching in text?
│   │   └── → KMP / Rabin-Karp
│   │
│   ├── Anagram / frequency problems?
│   │   └── → Hash Map + Sliding Window
│   │
│   └── Palindrome problems?
│       └── → Two Pointers (expand from center)
│
└── BIT-LEVEL operations?
    └── → Bit Manipulation (XOR, masks, counting)
```

---

## Learning Path

Organized into 13 categories with 99 sub-patterns. Within each category, sub-patterns are ordered
from foundational to advanced. Each sub-pattern has curated practice problems.

---

### Phase 1: Array & Matrix Manipulation

| #  | Sub-Pattern               | Core Idea                                              | Status |
|----|---------------------------|--------------------------------------------------------|--------|
| 1  | Cyclic Sort               | Numbers 1..n — put each in its correct index           | [ ]    |
| 2  | In-Place Rotation         | Rotate array/matrix without extra space                | [ ]    |
| 3  | Merge Sorted Array In-Place | Two sorted arrays, merge without extra allocation     | [ ]    |
| 4  | Spiral Traversal          | Walk a matrix in spiral order                          | [ ]    |
| 5  | Set Matrix Zeroes         | Mark rows/cols zero in-place                           | [ ]    |
| 6  | Product Except Self       | Build result array without division                    | [ ]    |
| 7  | Plus One                  | Carry propagation on array-represented numbers         | [ ]    |

**Practice problems:** First Missing Positive, Find All Duplicates, Find All Disappeared Numbers, Game of Life, Diagonal Traverse

---

### Phase 2: Linked List Manipulation

| #  | Sub-Pattern               | Core Idea                                              | Status |
|----|---------------------------|--------------------------------------------------------|--------|
| 8  | In-Place Reversal         | Reverse entire list or sublist using pointer swaps     | [ ]    |
| 9  | Merging Sorted Lists      | Merge two+ sorted linked lists                         | [ ]    |
| 10 | Reordering & Partitioning | Rearrange nodes by condition without extra list        | [ ]    |
| 11 | Addition of Numbers       | Add numbers represented as linked lists (carry logic)  | [ ]    |
| 12 | Intersection Detection    | Find where two lists meet                              | [ ]    |

**Practice problems:** Reverse Linked List, Add Two Numbers, Intersection of Two Linked Lists, Copy List with Random Pointer

---

### Phase 3: Two Pointers

| #  | Sub-Pattern                          | Core Idea                                         | Status |
|----|--------------------------------------|---------------------------------------------------|--------|
| 13 | Converging — Sorted Array Target Sum | Left + right pointers walk inward                 | [x]    |
| 14 | Fast & Slow — Cycle Detection        | Tortoise & hare for cycles, middle finding        | [x]    |
| 15 | Expanding from Center — Palindromes  | Expand outward to find palindromic substrings     | [x]    |
| 16 | In-Place Array Modification          | Remove/move elements with read+write pointers     | [x]    |
| 17 | String Reversal                      | Reverse in-place with two pointers                | [x]    |
| 18 | String Comparison — Backspaces       | Compare strings with backspace characters         | [x]    |
| 19 | Fixed Separation — Nth Node          | Two pointers N apart for nth-from-end problems    | [ ]    |

**Practice problems:** Three Sum, Container With Most Water, Two Sum II, Squares of a Sorted Array, Happy Number, Linked List Cycle, Find the Duplicate Number, Remove Nth Node, Middle of Linked List, Remove Duplicates, Move Zeroes, Sort Colors, Longest Palindromic Substring, Backspace String Compare

---

### Phase 4: Sliding Window

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 20 | Fixed-Size Subarray            | Window of constant size K slides across              | [ ]    |
| 21 | Variable-Size Condition        | Expand right, shrink left until condition met        | [ ]    |
| 22 | Character Frequency Matching   | Anagram/permutation detection with frequency maps    | [ ]    |
| 23 | Monotonic Queue (Max/Min)      | Deque maintaining window max/min in O(1)             | [ ]    |

**Practice problems:** Find All Anagrams, Fruit Into Baskets, Frequency of Most Frequent Element, Continuous Subarrays, Contains Duplicate II, Jump Game VI, Find XSum of K-Long Subarrays

---

### Phase 5: Binary Search

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 24 | On Sorted Array/List           | Classic halving search for target                    | [ ]    |
| 25 | Find First/Last Occurrence     | Boundary binary search (lower/upper bound)           | [ ]    |
| 26 | Rotated Sorted Array           | Modified BS handling the rotation pivot              | [ ]    |
| 27 | Median/Kth in Sorted Arrays    | Binary search on partitions across arrays            | [ ]    |
| 28 | On Answer (Condition Function) | Binary search the answer space (min/max feasibility) | [ ]    |

**Practice problems:** Binary Search, First Bad Version, Guess Number, Find Peak Element, Find Minimum in Rotated Sorted Array, Koko Eating Bananas, Capacity to Ship Packages, Find First and Last Position, Find K Closest Elements, Kth Missing Positive Number, Find in Mountain Array, Find K-th Smallest Pair Distance

---

### Phase 6: Stack Patterns

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 29 | Valid Parentheses Matching     | Stack tracks opening brackets for matching           | [ ]    |
| 30 | Monotonic Stack                | Maintain increasing/decreasing stack for NGE/NSE     | [ ]    |
| 31 | Largest Rectangle (Histogram)  | Stack-based area calculation                         | [ ]    |
| 32 | Min Stack Design               | Stack with O(1) getMin                               | [ ]    |
| 33 | Simulation / Backtracking Helper | Stack simulates recursion or undo                  | [ ]    |
| 34 | Expression Evaluation          | Operator precedence with stack(s)                    | [ ]    |

**Practice problems:** Daily Temperatures, Asteroid Collision, Decode String, Evaluate Reverse Polish Notation, Basic Calculator I/II/III, Largest Rectangle in Histogram, Removing Stars from String, Find Most Competitive Subsequence

---

### Phase 7: String Manipulation

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 35 | Anagram Check                  | Frequency counting to compare character sets         | [ ]    |
| 36 | Palindrome Check               | Two pointers or expand from center                   | [ ]    |
| 37 | String to Integer (atoi)       | Parsing with edge cases (overflow, whitespace, sign) | [ ]    |
| 38 | Integer/Roman Conversion       | Mapping rules + greedy subtraction                   | [ ]    |
| 39 | Multiply Strings               | Grade-school multiplication digit by digit           | [ ]    |
| 40 | Repeated Substring Pattern     | Detect if string = repeated copies of a substring    | [ ]    |
| 41 | Pattern Matching (KMP/Rabin-Karp) | Efficient substring search algorithms             | [ ]    |

**Practice problems:** Group Anagrams, Integer to Roman, Add Strings, Find Index of First Occurrence, Palindromic Substrings, Reverse Words in a String, String Compression, Add Binary

---

### Phase 8: Heap & Priority Queue

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 42 | Top K Elements                 | Min-heap of size K to track largest K                | [ ]    |
| 43 | Two Heaps — Median             | Max-heap (left) + min-heap (right) for running median| [ ]    |
| 44 | K-Way Merge                    | Min-heap to merge K sorted sources                   | [ ]    |
| 45 | Scheduling / Minimum Cost      | Greedy + heap for optimal task ordering              | [ ]    |

**Practice problems:** K Closest Points to Origin, Kth Largest Element in Array/Stream, Last Stone Weight, Find Median from Data Stream, Find K Pairs with Smallest Sums, Kth Smallest in Sorted Matrix, Furthest Building You Can Reach, MK Average

---

### Phase 9: Tree Traversal

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 46 | Recursive Preorder Traversal   | Root → Left → Right (top-down processing)            | [ ]    |
| 47 | Recursive Inorder Traversal    | Left → Root → Right (BST gives sorted order)         | [ ]    |
| 48 | Recursive Postorder Traversal  | Left → Right → Root (bottom-up aggregation)          | [ ]    |
| 49 | Level-Order Traversal (BFS)    | Queue-based layer-by-layer processing                | [ ]    |
| 50 | Serialization / Deserialization| Convert tree ↔ string representation                 | [ ]    |
| 51 | Lowest Common Ancestor         | Find shared ancestor of two nodes                    | [ ]    |

**Practice problems:** Binary Tree Level Order, Zigzag Level Order, Right Side View, Largest Value in Each Row, Invert Binary Tree, Flatten to Linked List, Construct from Preorder+Inorder, Kth Smallest in BST, BST Iterator, Balanced Binary Tree, Diameter, Maximum Path Sum, House Robber III, All Nodes Distance K, Delete Nodes and Return Forest, Find Duplicate Subtrees, Find Leaves

---

### Phase 10: Graph Traversal

| #  | Sub-Pattern                       | Core Idea                                          | Status |
|----|-----------------------------------|----------------------------------------------------|--------|
| 52 | DFS — Connected Components        | Explore depth-first, mark visited, count groups    | [ ]    |
| 53 | BFS — Connected Components        | Level-by-level exploration, shortest in unweighted | [ ]    |
| 54 | Topological Sort                  | Kahn's BFS or DFS for dependency ordering          | [ ]    |
| 55 | Cycle Detection (Directed)        | DFS coloring (white/gray/black) for directed cycles| [ ]    |
| 56 | Deep Copy / Cloning               | Clone graph/list with HashMap for visited nodes    | [ ]    |
| 57 | Dijkstra's Algorithm              | Priority queue for weighted shortest path          | [ ]    |
| 58 | Bellman-Ford / BFS with K stops   | Shortest path with edge count constraint           | [ ]    |
| 59 | Bidirectional BFS                 | Search from both ends to meet in the middle        | [ ]    |
| 60 | Union Find (DSU)                  | Disjoint set with union-by-rank + path compression | [ ]    |
| 61 | Minimum Spanning Tree             | Kruskal's (sort edges + UF) or Prim's (heap)      | [ ]    |
| 62 | Bridges & Articulation Points     | Tarjan's algorithm for critical connections        | [ ]    |
| 63 | Strongly Connected Components     | Kosaraju's or Tarjan's for directed graph SCCs     | [ ]    |

**Practice problems:** Flood Fill, Count Sub Islands, Keys and Rooms, Course Schedule I/II, Find Eventual Safe States, Clone Graph, Cheapest Flights Within K Stops, Graph Valid Tree, Accounts Merge, Critical Connections, Alien Dictionary, Find Safest Path, Bus Routes, Build Matrix With Conditions, Largest Color Value

---

### Phase 11: Greedy Patterns

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 64 | Jump Game                      | Can you reach the end? Greedy furthest reach        | [ ]    |
| 65 | Interval Merging / Scheduling  | Sort by start/end, merge or pick non-overlapping    | [ ]    |
| 66 | Buy & Sell Stock               | Track min price, maximize profit                    | [ ]    |
| 67 | Sorting-Based Greedy           | Sort + greedy assignment (cookies, boats)           | [ ]    |
| 68 | Task Scheduling                | Frequency-based scheduling with cooldown            | [ ]    |
| 69 | Gas Station / Circuit          | Circular route feasibility with prefix sums         | [ ]    |

**Practice problems:** Jump Game I/II, Insert Interval, Interval List Intersections, Employee Free Time, Divide Intervals into Minimum Groups, Best Time to Buy/Sell Stock I/II, Gas Station, Assign Cookies, Candy, Distant Barcodes

---

### Phase 12: Backtracking Patterns

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 70 | Subsets                        | Include/exclude each element — power set generation  | [ ]    |
| 71 | Permutations                   | Arrange all elements in every possible order         | [ ]    |
| 72 | Combination Sum                | Find combinations that sum to target (with/without reuse) | [ ]    |
| 73 | Parentheses Generation         | Generate valid bracket sequences                     | [ ]    |
| 74 | N-Queens                       | Place queens with row/col/diagonal constraints       | [ ]    |
| 75 | Word Search / Path Finding     | DFS on grid with visited tracking                    | [ ]    |
| 76 | Palindrome Partitioning        | Split string into palindromic substrings             | [ ]    |

**Practice problems:** Combinations, Letter Combinations of Phone Number, Combination Sum I/II, Generate Parentheses, Check Word in Crossword

---

### Phase 13: Dynamic Programming

| #  | Sub-Pattern                        | Core Idea                                          | Status |
|----|------------------------------------|----------------------------------------------------|--------|
| 77 | Fibonacci-Style                    | f(n) = f(n-1) + f(n-2) — overlapping 1D subproblems | [ ]    |
| 78 | Kadane's Algorithm                 | Max subarray sum with running local/global max      | [ ]    |
| 79 | Unique Paths (Grid DP)             | 2D grid — paths from top-left to bottom-right       | [ ]    |
| 80 | Coin Change                        | Unbounded knapsack — fewest coins to make amount    | [ ]    |
| 81 | Knapsack / Subset Sum              | 0/1 knapsack and its subset sum variant             | [ ]    |
| 82 | Longest Common Subsequence         | Two-string 2D DP table                              | [ ]    |
| 83 | Edit Distance                      | Insert/delete/replace operations between strings    | [ ]    |
| 84 | Longest Increasing Subsequence     | O(n²) DP or O(n log n) with patience sorting        | [ ]    |
| 85 | Word Break                         | Can string be segmented into dictionary words?      | [ ]    |
| 86 | Stock Problems (DP variant)        | Buy/sell with cooldown, K transactions, etc.        | [ ]    |
| 87 | Interval DP                        | Burst balloons, matrix chain — split subproblems    | [ ]    |
| 88 | Catalan Numbers                    | Unique BSTs, valid parentheses count                | [ ]    |

**Practice problems:** Climbing Stairs, Fibonacci, House Robber I/II/III, Decode Ways, Delete and Earn, Coin Change I/II, Combination Sum IV, Edit Distance, Delete Operation for Two Strings, Count Square Submatrices, Burst Balloons, Best Time to Buy/Sell Stock III/IV/Cooldown, Different Ways to Add Parentheses

---

### Phase 14: Bit Manipulation

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 89 | XOR — Single / Missing Number  | XOR cancels pairs, leaving the odd one out          | [ ]    |
| 90 | Power of Two / Four            | Bit tricks: n & (n-1) == 0                          | [ ]    |
| 91 | Counting Bits (DP)             | Count 1-bits for 0..n using DP relation             | [ ]    |
| 92 | Hamming Weight                 | Count set bits in an integer                        | [ ]    |

**Practice problems:** Find the Difference, Counting Bits, Single Number

---

### Phase 15: Design Patterns (Data Structure Design)

| #  | Sub-Pattern                    | Core Idea                                           | Status |
|----|--------------------------------|-----------------------------------------------------|--------|
| 93 | Trie (Prefix Tree)             | Tree of characters for prefix search / autocomplete | [ ]    |
| 94 | LRU / LFU Cache                | HashMap + Doubly Linked List for O(1) cache ops     | [ ]    |
| 95 | HashMap Design                 | Implement hash map from scratch                     | [ ]    |
| 96 | Stack / Queue from Primitives  | Implement stack using queues and vice versa          | [ ]    |
| 97 | Circular Queue / Deque         | Ring buffer for fixed-size queue                     | [ ]    |
| 98 | Iterator Design                | Flatten nested structures, custom iteration          | [ ]    |
| 99 | Special Data Structures        | All O(1), Insert/Delete/GetRandom, Hit Counter       | [ ]    |

**Practice problems:** Implement Trie, Design Add and Search Words, LFU Cache, Design HashMap, Design Stack with Increment, All O(1) Data Structure, Detect Squares, Design Text Editor, Design Search Autocomplete, Flatten Nested List Iterator, Insert Delete GetRandom O(1), Design Circular Queue/Deque, Encode and Decode Strings, Design Hit Counter, Logger Rate Limiter

---

## How Each Session Works

```
1. THE PROBLEM     → I describe a scenario / LeetCode-style problem
2. BRUTE FORCE     → You write the naive solution that works
3. THE PAIN        → We analyze: what's the time/space complexity? Why does it TLE?
4. THE PATTERN     → I introduce the pattern as the optimization
5. YOU IMPLEMENT   → You refactor using the pattern
6. THE ANALYSIS    → Time/space complexity, edge cases, when does this pattern apply?
```

---

## Progress Tracker

| Phase | Category                    | Sub-Patterns | Done | Remaining |
|-------|-----------------------------|:------------:|:----:|:---------:|
| 1     | Array & Matrix Manipulation | 7            | 0    | 7         |
| 2     | Linked List Manipulation    | 5            | 0    | 5         |
| 3     | Two Pointers                | 7            | 0    | 7         |
| 4     | Sliding Window              | 4            | 0    | 4         |
| 5     | Binary Search               | 5            | 0    | 5         |
| 6     | Stack Patterns              | 6            | 0    | 6         |
| 7     | String Manipulation         | 7            | 0    | 7         |
| 8     | Heap & Priority Queue       | 4            | 0    | 4         |
| 9     | Tree Traversal              | 6            | 0    | 6         |
| 10    | Graph Traversal             | 12           | 0    | 12        |
| 11    | Greedy Patterns             | 6            | 0    | 6         |
| 12    | Backtracking Patterns       | 7            | 0    | 7         |
| 13    | Dynamic Programming         | 12           | 0    | 12        |
| 14    | Bit Manipulation            | 4            | 0    | 4         |
| 15    | Design (Data Structures)    | 7            | 0    | 7         |
|       | **TOTAL**                   | **99**       | **0**| **99**    |

---

## Reference

- Pattern structure sourced from: [thita.ai DSA Patterns Sheet](https://thita.ai/dsa-patterns-sheet) (99 patterns, 412 problems)
- Additional references: Grokking the Coding Interview, NeetCode roadmap, LeetCode pattern lists
- All implementations: Java 21