# DSA Cheat Sheet — Java

A comprehensive, exam-ready reference covering **100 curated LeetCode-style problems** across 10 algorithmic categories. Each problem entry includes:

- One-paragraph plain-English restatement
- A **Pattern Flashcard** with trigger phrases and recognition cues
- Brute-force, optimal, and (where applicable) alternative-optimal approaches with trade-offs
- A clean, compilable **Java solution**
- Dry run / key insight, edge cases, follow-ups
- A **complexity summary table**

## How to use

1. **First pass:** Read a whole category file end-to-end. Internalize the *Pattern Flashcards* — they are how you map an unseen problem to a known technique under interview pressure.
2. **Drill pass:** Read just the **Quick Pattern Index** at the top of each file. Try to recall the solution before opening the entry.
3. **Mock-interview pass:** Solve from scratch in an editor; only consult the entry when stuck.

## Table of Contents

| # | File | Category | Problems |
|---|------|----------|----------|
| 1 | [01-arrays-hashing.md](01-arrays-hashing.md) | Arrays & Hashing | 10 |
| 2 | [02-two-pointers-sliding-window.md](02-two-pointers-sliding-window.md) | Two Pointers & Sliding Window | 11 |
| 3 | [03-binary-search.md](03-binary-search.md) | Binary Search | 8 |
| 4 | [04-stack-monotonic-queue.md](04-stack-monotonic-queue.md) | Stack & Monotonic Queue | 8 |
| 5 | [05-linked-lists.md](05-linked-lists.md) | Linked Lists | 9 |
| 6 | [06-binary-trees-bsts.md](06-binary-trees-bsts.md) | Binary Trees & BSTs | 12 |
| 7 | [07-backtracking.md](07-backtracking.md) | Backtracking | 7 |
| 8 | [08-graphs-topologies.md](08-graphs-topologies.md) | Graphs & Topologies | 13 |
| 9 | [09-dynamic-programming.md](09-dynamic-programming.md) | Dynamic Programming | 16 |
| 10 | [10-tries-greedy-intervals.md](10-tries-greedy-intervals.md) | Tries, Greedy & Intervals | 6 |

## Suggested Study Order

1. **Arrays & Hashing** — foundation; teaches the hash-map reflex.
2. **Two Pointers & Sliding Window** — the second most common pattern family.
3. **Stack & Monotonic Queue** — short, distinctive cues; quick wins.
4. **Binary Search** — once "search-on-answer" clicks, many "hard" problems collapse.
5. **Linked Lists** — pointer hygiene; reusable subroutines (reverse, find middle).
6. **Binary Trees & BSTs** — recursion templates that generalize to graphs.
7. **Graphs & Topologies** — BFS/DFS/topo-sort/DSU. Heavy but very pattern-driven.
8. **Backtracking** — once recursion is comfortable, backtracking is just constrained DFS.
9. **Dynamic Programming** — the hardest to "see"; do this once everything else feels routine.
10. **Tries, Greedy & Intervals** — small but high-yield grab-bag for finishing strong.

## Reverse Pattern Index — "I see X, I should reach for Y"

| Cue you spot in the problem | Reach for |
|---|---|
| "Pair with sum / complement / target" | Hash map — §1.1 Two Sum, §2.1 3Sum |
| "Anagram / characters of the same multiset" | Frequency map or sorted-key map — §1.2 |
| "Top K / K most frequent" | Min-heap of size K, or bucket sort — §1.3 |
| "Product / sum except self" | Prefix + suffix arrays — §1.4 |
| "Subarray sum equals K" | Prefix sum + hash map — §1.9 |
| "Longest consecutive / range in unsorted array" | HashSet + boundary check — §1.6 |
| "Merge / insert / count overlap intervals" | Sort by start, sweep — §1.7, §1.8, §10.5 |
| "Find pair / triplet in sorted array" | Two pointers from ends — §2.1, §2.2 |
| "Longest substring / window with constraint" | Sliding window — §2.5, §2.6, §2.8 |
| "Max / min in every window of size K" | Monotonic deque — §2.9 |
| "Trap water / span / next greater" | Monotonic stack — §2.3, §4.5, §4.7 |
| "Sorted array, find target/boundary" | Binary search — §3 |
| "Minimize the maximum / maximize the minimum" | Binary search on the answer — §3.4, §3.8 |
| "Median of two sorted arrays" | Partition-based binary search — §3.6 |
| "Balanced brackets / LIFO order" | Stack — §4.1, §4.3, §4.8 |
| "Cycle in linked list / find midpoint" | Fast & slow pointers — §5.3, §5.4, §5.7 |
| "LRU / O(1) cache" | HashMap + Doubly Linked List — §5.8 |
| "Merge K sorted streams" | Min-heap — §5.9 |
| "Tree path / depth / diameter" | Post-order DFS with return values — §6.2, §6.7 |
| "Level-by-level tree traversal" | BFS with queue — §6.3, §6.10 |
| "BST validation / search" | In-order is sorted — §6.5, §6.9 |
| "Number of islands / regions in grid" | DFS or BFS flood-fill — §8.1, §8.6, §8.7 |
| "Course prerequisites / build order" | Topological sort (Kahn's) — §8.4 |
| "Shortest path in unweighted graph / fewest moves" | BFS — §8.3, §8.9 |
| "Shortest path with positive weights" | Dijkstra (priority queue) — §8.10 |
| "Group / merge connected components / union accounts" | Disjoint Set Union — §8.11, §8.13 |
| "Generate all subsets / permutations / combinations" | Backtracking — §7 |
| "Optimal value with overlapping subproblems" | DP — §9 |
| "Knapsack / pick or skip items" | 0/1 or unbounded knapsack DP — §9.5, §9.6, §9.10 |
| "Two strings compared character by character" | 2D DP grid — §9.12, §9.13 |
| "Prefix matching / autocomplete / word search" | Trie — §10.1, §10.2 |
| "Minimum number of jumps / can reach end" | Greedy farthest-reach — §10.4 |
| "Minimum rooms / max concurrent" | Sort + min-heap — §10.6 |

## Complexity Cheat Card (Java specifics)

| Structure | Lookup | Insert | Delete | Notes |
|---|---|---|---|---|
| `HashMap<K,V>` | O(1) avg | O(1) avg | O(1) avg | Worst-case O(n) on hash collisions |
| `TreeMap<K,V>` | O(log n) | O(log n) | O(log n) | Sorted by key; `floorKey`, `ceilingKey` |
| `ArrayDeque` | — | O(1) push/pop | O(1) | Use as stack AND queue; faster than `Stack`/`LinkedList` |
| `PriorityQueue` | O(1) peek | O(log n) | O(log n) poll | Min-heap by default; `(a,b)->b-a` for max |
| `LinkedHashMap` | O(1) | O(1) | O(1) | Maintains insertion (or access) order — LRU base |
| `int[]` array as map | O(1) | O(1) | — | Use for ASCII / bounded-integer keys; ~5x faster than HashMap |

## Style Conventions Used Throughout

- Java code follows the LeetCode `class Solution { public ... }` shape so it can be pasted directly.
- `Math.floorDiv` / `(low + high) >>> 1` are used in binary search to avoid overflow.
- Inline comments are sparse — only where the WHY of a line is not obvious from the name.
- Time/space complexities use n, m, k consistently per problem.
