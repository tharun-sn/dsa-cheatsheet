# 5. Linked Lists

> Linked-list problems are short on logic but harsh on pointer hygiene. Three subroutines do most of the work: **reverse**, **find middle (fast/slow)**, and **merge two sorted lists**. Memorize these and most "hard" linked-list problems become compositions. Sentinel (dummy) nodes drastically reduce edge-case branching.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Reverse Linked List | Iterative three-pointer reversal | Easy |
| 2 | Merge Two Sorted Lists | Dummy head + two pointers | Easy |
| 3 | Reorder List | Find middle + reverse + merge | Medium |
| 4 | Remove Nth Node From End | Fast/slow with gap n | Medium |
| 5 | Copy List with Random Pointer | Map old → new, two passes | Medium |
| 6 | Add Two Numbers | Digit iteration with carry | Medium |
| 7 | Linked List Cycle II | Floyd's tortoise/hare + reset | Medium |
| 8 | LRU Cache | HashMap + doubly linked list | Medium |
| 9 | Merge K Sorted Lists | Min-heap of heads / divide & conquer | Hard |

### Reusable Subroutines

```java
// Standard node definition (LeetCode)
class ListNode { int val; ListNode next; ListNode(int v) { val = v; } }

// Reverse — returns new head
ListNode reverse(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;
        cur.next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}

// Find middle (for even length, returns lower middle)
ListNode middle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast.next != null && fast.next.next != null) {
        slow = slow.next; fast = fast.next.next;
    }
    return slow;
}
```

---

## 1. Reverse Linked List

**LeetCode #206 | Difficulty: Easy**

### Problem
Reverse a singly linked list. Return the new head.

### Pattern Flashcard
> **Trigger phrase:** "Reverse the list / reverse a range"
> **Recognize when:**
> - You need to flip `next` pointers in place.
> - Foundational subroutine appearing inside §5.3, §5.7, palindrome checks, etc.
>
> **Core idea in one line:** Walk forward with three pointers — `prev`, `cur`, `next` — and re-link `cur.next = prev` each step.

### Approaches & Trade-offs

#### Approach 1 — Stack
- Push all nodes; pop and relink. O(n) time / O(n) space.

#### Approach 2 — Iterative (optimal)
- O(n) / O(1).

#### Approach 3 — Recursive
- O(n) / O(n) stack. Elegant but stack overflow risk on huge inputs.

### Java Solution (optimal — iterative)
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null, cur = head;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        return prev;
    }
}
```

### Java Solution (recursive)
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) return head;
        ListNode rest = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return rest;
    }
}
```

### Dry Run / Key Insight
`1 → 2 → 3 → null`. prev=null, cur=1. Save next=2; 1.next=null; prev=1, cur=2. … → `3 → 2 → 1 → null`.

### Edge Cases
- Empty or single-node → unchanged.
- Don't forget `head.next = null` in recursion (or you create a cycle).

### Follow-ups & Variants
- **Reverse Linked List II** (LC #92) — reverse only `[left..right]`.
- **Reverse Nodes in k-Group** (LC #25) — reverse in chunks of k.
- **Palindrome Linked List** (LC #234) — reverse second half, compare.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Iterative (optimal) | O(n) | O(1) |
| Recursive | O(n) | O(n) |

---

## 2. Merge Two Sorted Lists

**LeetCode #21 | Difficulty: Easy**

### Problem
Merge two sorted lists into a single sorted list, splicing existing nodes (no new allocations needed).

### Pattern Flashcard
> **Trigger phrase:** "Merge two sorted linked lists"
> **Recognize when:**
> - Two sorted inputs, one sorted output.
> - Foundational subroutine for merge sort / §5.9.
>
> **Core idea in one line:** Dummy head + tail pointer; pick smaller of the two heads each iteration, advance.

### Approaches & Trade-offs

#### Approach 1 — Collect + Sort
- O((m+n) log(m+n)). Wastes the sortedness.

#### Approach 2 — Iterative Merge (optimal)
- O(m + n) · O(1).

#### Approach 3 — Recursive
- O(m + n) time / O(m + n) stack.

### Java Solution (optimal)
```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0), tail = dummy;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) { tail.next = l1; l1 = l1.next; }
            else                  { tail.next = l2; l2 = l2.next; }
            tail = tail.next;
        }
        tail.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }
}
```

### Dry Run / Key Insight
`l1=1→3→5, l2=2→4` → 1,2,3,4,5. The dummy avoids special-casing "no head yet."

### Edge Cases
- Either list empty → return the other.
- Both empty → null.

### Follow-ups & Variants
- **Merge K Sorted Lists** → §5.9.
- **Sort List** (LC #148) — merge sort using this primitive + §5.3 midpoint.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Collect+sort | O((m+n) log…) | O(m+n) |
| Iterative (optimal) | O(m + n) | O(1) |

---

## 3. Reorder List

**LeetCode #143 | Difficulty: Medium**

### Problem
Given `L₀ → L₁ → … → Lₙ`, reorder it to `L₀ → Lₙ → L₁ → Lₙ₋₁ → L₂ → …`. In place.

### Pattern Flashcard
> **Trigger phrase:** "Interleave first half with reversed second half"
> **Recognize when:**
> - You're producing pattern `a, z, b, y, c, x, …`.
> - In-place required.
>
> **Core idea in one line:** Three steps: find middle, reverse second half, weave the two halves together.

### Approaches & Trade-offs

#### Approach 1 — Use Array
- Copy to array, two-pointer reorder. O(n) time / O(n) space.

#### Approach 2 — In-place 3-Step (optimal)
- O(n) / O(1).

### Java Solution (optimal)
```java
class Solution {
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // 1. Find middle (slow ends at lower middle)
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next; fast = fast.next.next;
        }

        // 2. Reverse second half
        ListNode prev = null, cur = slow.next;
        slow.next = null;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }

        // 3. Weave
        ListNode a = head, b = prev;
        while (b != null) {
            ListNode aNext = a.next, bNext = b.next;
            a.next = b;
            b.next = aNext;
            a = aNext;
            b = bNext;
        }
    }
}
```

### Dry Run / Key Insight
`1→2→3→4→5`. Middle: 3. Reverse `4→5` → `5→4`. Weave: 1→5→2→4→3.

### Edge Cases
- Length ≤ 2 → no change needed.
- Odd vs even length — picking lower middle keeps the second half ≤ first half (so weaving terminates correctly when `b == null`).

### Follow-ups & Variants
- **Palindrome Linked List** — same midpoint+reverse decomposition.
- **Sort List** (LC #148) — midpoint + merge sort.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Array | O(n) | O(n) |
| In-place (optimal) | O(n) | O(1) |

---

## 4. Remove Nth Node From End of List

**LeetCode #19 | Difficulty: Medium**

### Problem
Remove the n-th node from the end of the list and return its head. One pass preferred.

### Pattern Flashcard
> **Trigger phrase:** "K-th from end / from tail"
> **Recognize when:**
> - You want a fixed offset from a tail you haven't reached yet.
>
> **Core idea in one line:** Two pointers separated by `n` edges; advance both until the leader hits the end. Trailer is at node-to-remove (or its predecessor with dummy head).

### Approaches & Trade-offs

#### Approach 1 — Two Passes
- Length count first, then walk `length − n`. O(n) time, two passes.

#### Approach 2 — One Pass with Gap Pointers (optimal)
- O(n) one pass.

### Java Solution (optimal)
```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode fast = dummy, slow = dummy;
        for (int i = 0; i < n; i++) fast = fast.next;
        while (fast.next != null) { fast = fast.next; slow = slow.next; }
        slow.next = slow.next.next;
        return dummy.next;
    }
}
```

### Dry Run / Key Insight
`1→2→3→4→5, n=2`. fast advances 2 → at node 2. Walk both till fast.next=null → fast=5, slow=3. Remove slow.next (=4). Result `1→2→3→5`.
**Mental model:** the dummy lets you uniformly remove the head when n = length.

### Edge Cases
- Remove head (n = length): handled by dummy.
- Single-node list with n = 1 → returns null.

### Follow-ups & Variants
- **K-th node from beginning** → trivial loop.
- **Find middle of list** → fast moves 2x slow.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Two passes | O(n) | O(1) |
| One pass (optimal) | O(n) | O(1) |

---

## 5. Copy List with Random Pointer

**LeetCode #138 | Difficulty: Medium**

### Problem
Each node has `next` and a `random` pointer (anywhere in the list, or null). Return a deep copy.

### Pattern Flashcard
> **Trigger phrase:** "Deep copy a graph-like linked list"
> **Recognize when:**
> - Pointers can target any node, including future ones.
> - You need a way to translate old node references to new ones.
>
> **Core idea in one line:** Build a `Map<Node, Node>` (old → new) in pass 1; assign `next` and `random` pointers in pass 2.

### Approaches & Trade-offs

#### Approach 1 — HashMap (two passes, optimal-readable)
- O(n) time / O(n) space.

#### Approach 2 — Interleaving (O(1) extra space)
- Create copies inline: `A → A' → B → B' → C → C'`. Set `randoms` by reference (`A'.random = A.random.next`), then split. O(n) / O(1).

### Java Solution (optimal — map)
```java
class Solution {
    public Node copyRandomList(Node head) {
        Map<Node, Node> map = new HashMap<>();
        for (Node cur = head; cur != null; cur = cur.next) {
            map.put(cur, new Node(cur.val));
        }
        for (Node cur = head; cur != null; cur = cur.next) {
            map.get(cur).next = map.get(cur.next);
            map.get(cur).random = map.get(cur.random);
        }
        return map.get(head);
    }
}
```

### Dry Run / Key Insight
Map lookups translate "old node" → "new node," handling future references cleanly.

### Edge Cases
- Null head → null result.
- `random` pointing to self.
- `random == null` for some nodes — `map.get(null)` returns null naturally.

### Follow-ups & Variants
- **Clone Graph** (LC #133) — see §8.2.
- **Deep clone with multilevel pointers** (LC #430).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Map (optimal) | O(n) | O(n) |
| Interleaving | O(n) | O(1) |

---

## 6. Add Two Numbers

**LeetCode #2 | Difficulty: Medium**

### Problem
Two non-empty lists representing non-negative integers in **reverse** digit order. Return their sum as a list, same format.

### Pattern Flashcard
> **Trigger phrase:** "Big-integer arithmetic on linked digits"
> **Recognize when:**
> - Each node is a digit; you need carry propagation.
>
> **Core idea in one line:** Walk both lists in lockstep with a running `carry`. Continue while either list has nodes or carry is nonzero.

### Approaches & Trade-offs

#### Approach 1 — Convert to BigInteger
- Trivial but assumes you can rebuild list. Not idiomatic.

#### Approach 2 — Digit Iteration with Carry (optimal)
- O(max(m, n)) · O(max(m, n)) for output.

### Java Solution (optimal)
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0), tail = dummy;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry
                    + (l1 != null ? l1.val : 0)
                    + (l2 != null ? l2.val : 0);
            tail.next = new ListNode(sum % 10);
            tail = tail.next;
            carry = sum / 10;
            if (l1 != null) l1 = l1.next;
            if (l2 != null) l2 = l2.next;
        }
        return dummy.next;
    }
}
```

### Dry Run / Key Insight
`2→4→3 (=342)` + `5→6→4 (=465)` = 807 → `7→0→8`.
The `carry != 0` clause is what handles "999 + 1 = 1000" growing the output.

### Edge Cases
- Different lengths → the missing-side contributes 0.
- Both null but carry = 1 → append `1`.
- Long-list overflow → not an issue since each digit is a node.

### Follow-ups & Variants
- **Add Two Numbers II** (LC #445) — digits stored most-significant-first → use stacks or reverse first.
- **Multiply Strings** (LC #43) — multi-digit multiplication; similar carry mechanics.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Digit iteration (optimal) | O(max(m, n)) | O(max(m, n)) |

---

## 7. Linked List Cycle II

**LeetCode #142 | Difficulty: Medium**

### Problem
Given a linked list, return the node where the cycle begins, or null if no cycle.

### Pattern Flashcard
> **Trigger phrase:** "Find the entry point of a cycle"
> **Recognize when:**
> - Possibly cyclic list.
> - Must be O(1) extra space (otherwise just hash visited nodes).
>
> **Core idea in one line:** Floyd's algorithm. Phase 1: fast/slow meet inside cycle. Phase 2: reset one pointer to head; advance both one step until they meet — that's the cycle entry.

### Approaches & Trade-offs

#### Approach 1 — HashSet
- O(n) / O(n).

#### Approach 2 — Floyd's (optimal)
- O(n) / O(1).

### Java Solution (optimal)
```java
class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                ListNode p = head;
                while (p != slow) { p = p.next; slow = slow.next; }
                return p;
            }
        }
        return null;
    }
}
```

### Dry Run / Key Insight
Let `L` = distance from head to cycle entry, `C` = cycle length, `k` = distance from entry to first meeting. When fast and slow meet, slow has walked `L + k`, fast `2(L + k)` = `L + k + nC` → `L + k = nC` → `L = nC − k`. Walking `L` from the head and `L` from the meeting point arrives at the entry simultaneously.

### Edge Cases
- No cycle → fast hits null.
- Cycle starts at head → both meet at head.
- Single-node cycle to itself.

### Follow-ups & Variants
- **Cycle existence** only (LC #141) — phase 1 only.
- **Find duplicate number** (LC #287) — treat indices as pointers and apply Floyd's.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| HashSet | O(n) | O(n) |
| Floyd (optimal) | O(n) | O(1) |

---

## 8. LRU Cache

**LeetCode #146 | Difficulty: Medium**

### Problem
Design a cache supporting `get(key)` and `put(key, value)` in **O(1)**. Evict the least-recently-used entry when capacity is exceeded.

### Pattern Flashcard
> **Trigger phrase:** "O(1) cache / LRU / MRU"
> **Recognize when:**
> - Need O(1) lookup AND O(1) updates of "recency order."
>
> **Core idea in one line:** Doubly linked list keyed by recency (head = MRU, tail = LRU); HashMap from key to node for O(1) splice.

### Approaches & Trade-offs

#### Approach 1 — LinkedHashMap (idiomatic Java)
- Override `removeEldestEntry`. O(1) all ops. Less impressive in interview.

#### Approach 2 — HashMap + Doubly Linked List (the textbook answer)
- O(1) get / put.

### Java Solution (optimal — DIY)
```java
class LRUCache {
    private static class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    private final int cap;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0);

    public LRUCache(int capacity) {
        cap = capacity;
        head.next = tail; tail.prev = head;
    }

    public int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        moveToFront(n);
        return n.val;
    }

    public void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) { n.val = value; moveToFront(n); return; }
        if (map.size() == cap) {
            Node lru = tail.prev;
            unlink(lru);
            map.remove(lru.key);
        }
        Node fresh = new Node(key, value);
        addAfter(head, fresh);
        map.put(key, fresh);
    }

    private void unlink(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addAfter(Node where, Node n) {
        n.prev = where; n.next = where.next;
        where.next.prev = n; where.next = n;
    }
    private void moveToFront(Node n) { unlink(n); addAfter(head, n); }
}
```

### Dry Run / Key Insight
A DLL allows O(1) removal of any node *given the node* — that's why the map points to *nodes*, not values.

### Edge Cases
- `get` on missing key → -1.
- `put` updating existing key — must move to front, not insert new.
- Capacity 1 → every put after the first evicts.

### Follow-ups & Variants
- **LFU Cache** (LC #460) — frequency + recency; nested DLL by frequency bucket.
- **All O(1) data structure** (LC #432).

### Complexity Summary
| Operation | Time |
|---|---|
| get / put | O(1) |

---

## 9. Merge K Sorted Lists

**LeetCode #23 | Difficulty: Hard**

### Problem
Merge `k` sorted linked lists into one sorted list.

### Pattern Flashcard
> **Trigger phrase:** "Merge K sorted streams"
> **Recognize when:**
> - Multiple sorted inputs, one sorted output.
> - K > 2 makes pairwise merge inefficient.
>
> **Core idea in one line:** Either (a) min-heap holding current heads, or (b) divide & conquer: pairwise merge in `log k` rounds.

### Approaches & Trade-offs

#### Approach 1 — Concatenate and Sort
- O(N log N) where N = total nodes.

#### Approach 2 — Pairwise Merge in Order
- O(kN) — k merges, each O(N).

#### Approach 3 — Min-Heap of Heads (optimal)
- O(N log k) · O(k).

#### Approach 4 — Divide & Conquer Merge
- O(N log k) · O(log k) stack.

### Java Solution (optimal — min-heap)
```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
        for (ListNode h : lists) if (h != null) pq.offer(h);

        ListNode dummy = new ListNode(0), tail = dummy;
        while (!pq.isEmpty()) {
            ListNode min = pq.poll();
            tail.next = min;
            tail = min;
            if (min.next != null) pq.offer(min.next);
        }
        return dummy.next;
    }
}
```

### Java Solution (divide & conquer)
```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) return null;
        return merge(lists, 0, lists.length - 1);
    }
    private ListNode merge(ListNode[] a, int l, int r) {
        if (l == r) return a[l];
        int m = (l + r) >>> 1;
        return mergeTwo(merge(a, l, m), merge(a, m + 1, r));
    }
    private ListNode mergeTwo(ListNode a, ListNode b) { /* see §5.2 */
        ListNode dummy = new ListNode(0), tail = dummy;
        while (a != null && b != null) {
            if (a.val <= b.val) { tail.next = a; a = a.next; }
            else                { tail.next = b; b = b.next; }
            tail = tail.next;
        }
        tail.next = (a != null) ? a : b;
        return dummy.next;
    }
}
```

### Dry Run / Key Insight
**Heap mental model:** at every moment the heap contains exactly one node from each list still in play, so polling gives the global minimum.
**D&C mental model:** identical to bottom-up merge sort across lists.

### Edge Cases
- All lists empty (`null` heads) → null result.
- `lists` array empty → null.
- Mixed null and non-null heads — heap skips nulls (see initial `if`).

### Follow-ups & Variants
- **Smallest range covering K lists** (LC #632) — heap + sliding range.
- **K-way merge for external sort** — same template.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Concat + sort | O(N log N) | O(N) |
| Pairwise | O(kN) | O(1) |
| Heap (optimal) | O(N log k) | O(k) |
| Divide & conquer | O(N log k) | O(log k) |
