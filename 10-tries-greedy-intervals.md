# 10. Tries, Greedy & Intervals

> A short but high-yield section. **Tries** are the canonical data structure for prefix problems and turn many "search many words in many places" problems into linear-time sweeps. **Greedy** algorithms commit to a locally optimal choice with a proof of global optimality. **Interval scheduling** problems lean heavily on sorting + sweep — sometimes greedy, sometimes heap-based.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Implement Trie | Tree of 26-child nodes with terminal flag | Medium |
| 2 | Word Search II | Grid DFS + Trie pruning | Hard |
| 3 | Gas Station | Single-pass greedy with running tank | Medium |
| 4 | Jump Game / II | Farthest-reach greedy | Medium |
| 5 | Non-overlapping Intervals | Sort by end, count keepers | Medium |
| 6 | Meeting Rooms II | Min-heap of end times OR sweep-line | Medium |

### Trie Node Template

```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean end;
}
```

---

## 1. Implement Trie (Prefix Tree)

**LeetCode #208 | Difficulty: Medium**

### Problem
Implement a trie supporting `insert(word)`, `search(word)` (exact match), and `startsWith(prefix)`.

### Pattern Flashcard
> **Trigger phrase:** "Prefix lookup / autocomplete / many-word search"
> **Recognize when:**
> - You'll perform many prefix or exact-word queries.
> - HashMap of words is fine for exact match; prefixes need a trie.
>
> **Core idea in one line:** Tree where each edge represents one character; mark nodes that are *ends* of inserted words.

### Approaches & Trade-offs

#### Approach 1 — HashSet of Words
- O(1) `search`, but `startsWith` is O(words · len). Too slow.

#### Approach 2 — Array of Children (optimal for fixed alphabet)
- O(L) per op; O(N · L · 26) memory worst case.

#### Approach 3 — HashMap of Children (general alphabet)
- O(L) per op; slightly more space.

### Java Solution (optimal)
```java
class Trie {
    private static class Node { Node[] c = new Node[26]; boolean end; }
    private final Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        for (char ch : word.toCharArray()) {
            int k = ch - 'a';
            if (cur.c[k] == null) cur.c[k] = new Node();
            cur = cur.c[k];
        }
        cur.end = true;
    }

    public boolean search(String word) {
        Node n = find(word);
        return n != null && n.end;
    }

    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private Node find(String s) {
        Node cur = root;
        for (char ch : s.toCharArray()) {
            int k = ch - 'a';
            if (cur.c[k] == null) return null;
            cur = cur.c[k];
        }
        return cur;
    }
}
```

### Dry Run / Key Insight
`insert("app")`, `insert("apple")`: at node after `"app"`, both `.end = true` (for "app") and child `'l'` (for "apple") coexist.

### Edge Cases
- Empty string: setting `root.end = true` makes `search("")` valid.
- Searching for a prefix that exists only as a substring of a longer word → `search` false, `startsWith` true.

### Follow-ups & Variants
- **Design Add and Search Word** (LC #211) — supports `.` wildcard via recursive DFS at each node.
- **Replace Words** (LC #648) — preprocess roots into trie, replace each word by shortest matching root.
- **Maximum XOR of Two Numbers** (LC #421) — binary trie of bits.

### Complexity Summary
| Operation | Time | Space |
|---|---|---|
| insert / search / startsWith | O(L) | O(total chars × 26) |

---

## 2. Word Search II

**LeetCode #212 | Difficulty: Hard**

### Problem
Given an `m × n` board of letters and a list of words, return all words that can be formed by 4-adjacent paths (no cell reused per path).

### Pattern Flashcard
> **Trigger phrase:** "Find many words in a grid"
> **Recognize when:**
> - Running Word Search (§7.4) per word would be O(words · m·n·4^L).
> - You can share work across words via a trie.
>
> **Core idea in one line:** Insert all words into a trie. DFS the grid; at each step, descend the trie. If you hit a terminal node, record the word. Prune branches where no trie child exists.

### Approaches & Trade-offs

#### Approach 1 — Run Word Search per Word
- TLE for many words.

#### Approach 2 — Trie + DFS (optimal)
- O(m·n·4^L). The trie shares prefix work; pruning kills dead paths.

### Java Solution (optimal)
```java
class Solution {
    private static class Node { Node[] c = new Node[26]; String word; }

    public List<String> findWords(char[][] board, String[] words) {
        Node root = new Node();
        for (String w : words) {
            Node cur = root;
            for (char ch : w.toCharArray()) {
                int k = ch - 'a';
                if (cur.c[k] == null) cur.c[k] = new Node();
                cur = cur.c[k];
            }
            cur.word = w;
        }

        List<String> out = new ArrayList<>();
        for (int r = 0; r < board.length; r++)
            for (int c = 0; c < board[0].length; c++)
                dfs(board, r, c, root, out);
        return out;
    }

    private void dfs(char[][] b, int r, int c, Node node, List<String> out) {
        if (r < 0 || c < 0 || r >= b.length || c >= b[0].length) return;
        char ch = b[r][c];
        if (ch == '#' || node.c[ch - 'a'] == null) return;
        Node next = node.c[ch - 'a'];
        if (next.word != null) { out.add(next.word); next.word = null; }  // dedupe

        b[r][c] = '#';
        dfs(b, r + 1, c, next, out);
        dfs(b, r - 1, c, next, out);
        dfs(b, r, c + 1, next, out);
        dfs(b, r, c - 1, next, out);
        b[r][c] = ch;
    }
}
```

### Dry Run / Key Insight
Setting `next.word = null` after recording prevents duplicates in output and lets us **prune the trie** as words are consumed.

### Edge Cases
- Empty words list → empty output.
- A word equal to a prefix of another → both get found if both present in the trie.
- Repeated letters in board — handled by `#` marker.

### Follow-ups & Variants
- **Stream of Characters** (LC #1032) — reversed trie + suffix matching.
- **Maximum XOR** (LC #421) — binary trie.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Trie + DFS (optimal) | O(m·n·4^L) | O(total chars × 26) |

L = longest word length.

---

## 3. Gas Station

**LeetCode #134 | Difficulty: Medium**

### Problem
A circular route of `n` stations; `gas[i]` units available and `cost[i]` to reach the next. Return the starting index from which you can complete the circuit, or −1 if impossible. Unique answer guaranteed if it exists.

### Pattern Flashcard
> **Trigger phrase:** "Circular route / can we complete the loop"
> **Recognize when:**
> - Total feasibility check first: sum(gas) ≥ sum(cost) is necessary AND sufficient.
> - Need to find the *right starting point*.
>
> **Core idea in one line:** If at some point running tank goes negative, no station up to that point can be the start. Reset start to next station, continue. One pass.

### Approaches & Trade-offs

#### Approach 1 — Try Every Start
- O(n²).

#### Approach 2 — Single-pass Greedy (optimal)
- O(n) · O(1).

### Java Solution (optimal)
```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int total = 0, tank = 0, start = 0;
        for (int i = 0; i < gas.length; i++) {
            int diff = gas[i] - cost[i];
            total += diff;
            tank  += diff;
            if (tank < 0) { start = i + 1; tank = 0; }
        }
        return total < 0 ? -1 : start;
    }
}
```

### Dry Run / Key Insight
**Proof of correctness:** if you run out of gas at station `j` starting from `i`, no station in `[i..j]` works as a start (each prefix sum is non-negative up to that point, so removing earlier non-positive contributions can't help). Skip to `j + 1`.

### Edge Cases
- Sum(gas) < Sum(cost) → −1.
- Single station that satisfies → return 0 if it works.

### Follow-ups & Variants
- **Candy** (LC #135) — two-pass greedy.
- **Jump Game** (§10.4) — similar reach-tracking ideas.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Greedy (optimal) | O(n) | O(1) |

---

## 4. Jump Game / Jump Game II

**LeetCode #55 / #45 | Difficulty: Medium / Medium**

### Problem (#55)
Each element is your max jump from that index. Can you reach the last index?

### Problem (#45)
Same setup; return the **minimum number of jumps** to reach the end. Assume always reachable.

### Pattern Flashcard
> **Trigger phrase:** "Reach the end / minimum jumps"
> **Core idea in one line:** Track `farthest` reachable. At each position, update `farthest = max(farthest, i + nums[i])`. For minimum jumps, also track the **end of the current jump's range** and increment jumps when crossed.

### Approaches & Trade-offs

#### Approach 1 — BFS
- O(n²) worst case.

#### Approach 2 — DP `reachable[i]`
- O(n²).

#### Approach 3 — Greedy (optimal)
- O(n) · O(1).

### Java Solution (#55 — Can Jump)
```java
class Solution {
    public boolean canJump(int[] nums) {
        int farthest = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > farthest) return false;
            farthest = Math.max(farthest, i + nums[i]);
        }
        return true;
    }
}
```

### Java Solution (#45 — Jump Game II)
```java
class Solution {
    public int jump(int[] nums) {
        int jumps = 0, farthest = 0, end = 0;
        for (int i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);
            if (i == end) {           // we've consumed current jump's range
                jumps++;
                end = farthest;       // commit to the new range
            }
        }
        return jumps;
    }
}
```

### Dry Run / Key Insight
`#45` is BFS in disguise: layer k is "indices reachable in k jumps." `end` marks the boundary of the current layer; when we cross it, we've started a new layer.

### Edge Cases
- Single element → reachable; 0 jumps.
- nums[0] = 0 with n > 1 → unreachable.

### Follow-ups & Variants
- **Frog Jump** (LC #403) — variable step sizes; DP.
- **Minimum Jumps to Reach Home** (LC #1654) — BFS with forward/backward rules.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| BFS / DP | O(n²) | O(n) |
| Greedy (optimal) | O(n) | O(1) |

---

## 5. Non-overlapping Intervals

**LeetCode #435 | Difficulty: Medium**

### Problem
Given a collection of intervals, find the minimum number to remove so that the rest are non-overlapping.

### Pattern Flashcard
> **Trigger phrase:** "Remove fewest intervals to make compatible"
> **Recognize when:**
> - Equivalent: maximize the count of mutually non-overlapping intervals (classic activity selection).
>
> **Core idea in one line:** Sort by **end time**. Greedily keep an interval if its start ≥ last kept end. The number to remove = total − kept.

### Approaches & Trade-offs

#### Approach 1 — DP
- O(n²).

#### Approach 2 — Greedy by End (optimal)
- O(n log n).

### Java Solution (optimal)
```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));
        int kept = 0, end = Integer.MIN_VALUE;
        for (int[] iv : intervals) {
            if (iv[0] >= end) { kept++; end = iv[1]; }
        }
        return intervals.length - kept;
    }
}
```

### Dry Run / Key Insight
**Why sort by end?** Earlier finishes leave more room for future picks. Sorting by start can mislead you when a long early-starting interval should be skipped for a shorter later one.

### Edge Cases
- All non-overlapping → 0.
- All identical → keep one, remove rest.
- Empty → 0.

### Follow-ups & Variants
- **Minimum Number of Arrows to Burst Balloons** (LC #452) — same template (kept = arrows).
- **Maximum Length of Pair Chain** (LC #646) — same pattern.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DP | O(n²) | O(n) |
| Greedy (optimal) | O(n log n) | O(1) |

---

## 6. Meeting Rooms II

**LeetCode #253 | Difficulty: Medium**

### Problem
Given an array of meeting intervals `[start, end]`, return the **minimum number of conference rooms** needed.

### Pattern Flashcard
> **Trigger phrase:** "Max concurrent intervals / min rooms / max overlap"
> **Recognize when:**
> - Question is "what's the *peak* number of overlapping intervals?"
>
> **Core idea in one line:** Sort by start. Min-heap of meeting end times — when a new meeting begins, if the earliest-ending meeting has finished (top of heap ≤ new start), reuse that room (pop); otherwise allocate a new one. Heap size = rooms in use.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For each meeting, count overlapping. O(n²).

#### Approach 2 — Min-Heap of End Times (optimal)
- O(n log n).

#### Approach 3 — Sweep-Line / Two-Pointer
- Sort starts and ends separately; walk both. O(n log n). No heap needed.

### Java Solution (optimal — min-heap)
```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        if (intervals.length == 0) return 0;
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        PriorityQueue<Integer> ends = new PriorityQueue<>();
        for (int[] iv : intervals) {
            if (!ends.isEmpty() && ends.peek() <= iv[0]) ends.poll();
            ends.offer(iv[1]);
        }
        return ends.size();
    }
}
```

### Java Solution (sweep-line)
```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        int n = intervals.length;
        int[] starts = new int[n], ends = new int[n];
        for (int i = 0; i < n; i++) { starts[i] = intervals[i][0]; ends[i] = intervals[i][1]; }
        Arrays.sort(starts); Arrays.sort(ends);
        int rooms = 0, peak = 0, j = 0;
        for (int i = 0; i < n; i++) {
            if (starts[i] < ends[j]) rooms++;
            else { j++; }
            peak = Math.max(peak, rooms);
        }
        return peak;
    }
}
```

### Dry Run / Key Insight
**Heap version intuition:** at any moment the heap holds the rooms currently in use, keyed by when they free up. We pop whenever the soonest-freeing room is already free.
**Sweep-line intuition:** treat starts as `+1` events and ends as `−1` events; the running count reaches the answer at its peak.

### Edge Cases
- No meetings → 0.
- All at once → equal to count of meetings.
- Touching intervals `[1,5],[5,10]` — typically counted as **not** overlapping (`<=` test); confirm problem spec.

### Follow-ups & Variants
- **Meeting Rooms** (LC #252) — can a single person attend? Sort by start, check overlaps.
- **My Calendar I / II / III** (LC #729, #731, #732) — incremental booking; TreeMap-based.
- **Employee Free Time** (LC #759) — merge intervals across employees.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Min-heap (optimal) | O(n log n) | O(n) |
| Sweep-line | O(n log n) | O(n) |
