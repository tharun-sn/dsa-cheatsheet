# 3. Binary Search

> Binary search shows up in two flavors: (1) **Classical** — sorted array, find a value or boundary. (2) **Binary search on the answer** — the answer space is monotone (a property is "true past some threshold"), so binary-search over the *value* itself and use a feasibility check. Mastering the second flavor is what makes "hard" problems feel routine.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Search in Rotated Sorted Array | Identify sorted half via mid comparison | Medium |
| 2 | Find Minimum in Rotated Sorted Array | Binary search for inflection point | Medium |
| 3 | Search a 2D Matrix | Treat as flat sorted array | Medium |
| 4 | Koko Eating Bananas | Binary search on the answer | Medium |
| 5 | Time Based Key-Value Store | Map + binary search by timestamp | Medium |
| 6 | Median of Two Sorted Arrays | Partition both arrays via binary search | Hard |
| 7 | First & Last Position of Element | Lower bound + upper bound | Medium |
| 8 | Allocate Minimum Pages / Aggressive Cows | Binary search on the answer with feasibility check | Hard |

### Two Canonical Templates

**Lower-bound (first index with `arr[i] ≥ target`):**
```java
int l = 0, r = n;             // r exclusive
while (l < r) {
    int m = (l + r) >>> 1;
    if (arr[m] < target) l = m + 1;
    else r = m;
}
return l;                      // 0..n
```

**Binary search on the answer (smallest `x` for which `feasible(x)` is true):**
```java
int l = lowBound, r = highBound;
while (l < r) {
    int m = l + (r - l) / 2;
    if (feasible(m)) r = m;
    else l = m + 1;
}
return l;
```

---

## 1. Search in Rotated Sorted Array

**LeetCode #33 | Difficulty: Medium**

### Problem
A sorted array with distinct values was rotated around an unknown pivot. Given `target`, return its index (or −1) in **O(log n)**.

### Pattern Flashcard
> **Trigger phrase:** "Sorted but rotated array"
> **Recognize when:**
> - Array is sorted *except for one rotation*.
> - You must achieve O(log n).
>
> **Core idea in one line:** At each step, exactly one half (`[l..m]` or `[m..r]`) is sorted. Test which, then check if target lies inside that sorted half.

### Approaches & Trade-offs

#### Approach 1 — Linear Scan
- O(n). Ignores sortedness.

#### Approach 2 — Find Pivot, then Binary Search (two passes)
- O(log n). Cleaner mental model but two phases.

#### Approach 3 — Single-Pass Modified Binary Search (optimal)
- O(log n) with one loop.

### Java Solution (optimal)
```java
class Solution {
    public int search(int[] nums, int target) {
        int l = 0, r = nums.length - 1;
        while (l <= r) {
            int m = (l + r) >>> 1;
            if (nums[m] == target) return m;
            if (nums[l] <= nums[m]) {              // left half sorted
                if (target >= nums[l] && target < nums[m]) r = m - 1;
                else l = m + 1;
            } else {                                // right half sorted
                if (target > nums[m] && target <= nums[r]) l = m + 1;
                else r = m - 1;
            }
        }
        return -1;
    }
}
```

### Dry Run / Key Insight
`[4,5,6,7,0,1,2], target=0` → m=3 (7); left `[4..7]` sorted, target 0 not in `[4,7)`, go right. l=4, m=5 (1); left `[0,1]` sorted, target 0 in `[0,1)`, go left. l=4,r=4, found.
**Mental model:** Rotated array = two sorted halves stuck together; the half containing `mid` and the leftmost element is the sorted one.

### Edge Cases
- No rotation → behaves like normal binary search.
- Single element → trivial.
- **Duplicates** version (LC #81): when `nums[l] == nums[m] == nums[r]`, can't decide — shrink both ends; worst-case O(n).

### Follow-ups & Variants
- **Find minimum** in rotated → §3.2.
- **Rotated array with duplicates** (LC #81).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Linear | O(n) | O(1) |
| Modified binary search (optimal) | O(log n) | O(1) |

---

## 2. Find Minimum in Rotated Sorted Array

**LeetCode #153 | Difficulty: Medium**

### Problem
Find the minimum of a rotated sorted array with distinct values in O(log n).

### Pattern Flashcard
> **Trigger phrase:** "Find inflection / smallest in rotated array"
> **Recognize when:**
> - Min sits at the *rotation point*.
> - Use comparison to `nums[r]` (not `nums[l]`) for cleanest invariant.
>
> **Core idea in one line:** Compare `nums[m]` to `nums[r]`. If `nums[m] > nums[r]`, the min is in `(m, r]`. Else it's in `[l, m]`.

### Approaches & Trade-offs

#### Approach 1 — Linear
- O(n).

#### Approach 2 — Binary Search (optimal)
- O(log n).

### Java Solution (optimal)
```java
class Solution {
    public int findMin(int[] nums) {
        int l = 0, r = nums.length - 1;
        while (l < r) {
            int m = (l + r) >>> 1;
            if (nums[m] > nums[r]) l = m + 1;
            else r = m;
        }
        return nums[l];
    }
}
```

### Dry Run / Key Insight
`[3,4,5,1,2]` → m=2 (5), 5 > 2, l=3. m=3, 1 < 2, r=3. Return nums[3]=1.
**Mental model:** `nums[r]` is the only fixed reference for "are we past the rotation?".

### Edge Cases
- Not rotated → returns `nums[0]`.
- Two elements → standard binary collapse.
- **With duplicates** (LC #154): when `nums[m] == nums[r]`, decrement `r`. Worst-case O(n).

### Follow-ups & Variants
- **Maximum in rotated array** → similar with flipped comparison.
- **Search target in rotated** → §3.1.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Linear | O(n) | O(1) |
| Binary search (optimal) | O(log n) | O(1) |

---

## 3. Search a 2D Matrix

**LeetCode #74 | Difficulty: Medium**

### Problem
Given an `m × n` matrix where each row is sorted and the first integer of each row is greater than the last integer of the previous row, determine if `target` exists.

### Pattern Flashcard
> **Trigger phrase:** "Matrix sorted row-by-row, rows chained sorted"
> **Recognize when:**
> - Globally sorted matrix → treat as 1D.
> - Globally sorted **per row only** → different problem (staircase search).
>
> **Core idea in one line:** Linearize: index `i` ↔ cell `(i / n, i % n)`. Binary search 0..m*n.

### Approaches & Trade-offs

#### Approach 1 — Linear Scan
- O(mn).

#### Approach 2 — Row Binary Search × Row Locate
- Find correct row, then binary search within. O(log m + log n) = O(log mn). Same asymptotic.

#### Approach 3 — Virtual 1D Binary Search (optimal, cleanest)
- O(log(mn)) · O(1) space.

### Java Solution (optimal)
```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;
        int l = 0, r = m * n - 1;
        while (l <= r) {
            int mid = (l + r) >>> 1;
            int val = matrix[mid / n][mid % n];
            if (val == target) return true;
            if (val < target) l = mid + 1;
            else r = mid - 1;
        }
        return false;
    }
}
```

### Dry Run / Key Insight
Matrix as a flat sorted sequence: row `i` represents indices `[i·n, (i+1)·n − 1]`.

### Edge Cases
- Empty matrix or empty rows → return false.
- 1×n or n×1 → reduces to standard binary search.

### Follow-ups & Variants
- **Search a 2D Matrix II** (LC #240) — only row- and column-sorted (not globally). Use staircase search starting from top-right corner: move left if too large, down if too small. O(m + n).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Linear | O(mn) | O(1) |
| 1D binary search (optimal) | O(log mn) | O(1) |
| Staircase (II) | O(m + n) | O(1) |

---

## 4. Koko Eating Bananas

**LeetCode #875 | Difficulty: Medium**

### Problem
Koko has `piles[i]` bananas in pile `i` and `h` hours. Each hour she eats up to `k` bananas from any one pile. Find the minimum `k` such that she finishes within `h` hours.

### Pattern Flashcard
> **Trigger phrase:** "Minimize / maximize a value subject to a feasibility condition"
> **Recognize when:**
> - There's a *threshold value*: feasible for all `x ≥ T`, infeasible below.
> - You can write `feasible(x)` in O(n).
>
> **Core idea in one line:** Binary-search `k` in `[1, maxPile]`; for each candidate compute total hours = `Σ ceil(pile/k)` and shrink toward smaller `k` while still ≤ h.

### Approaches & Trade-offs

#### Approach 1 — Try every k from 1 upward
- O(maxPile · n). TLE.

#### Approach 2 — Binary Search on Answer (optimal)
- O(n · log(maxPile)).

### Java Solution (optimal)
```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int lo = 1, hi = 0;
        for (int p : piles) hi = Math.max(hi, p);
        while (lo < hi) {
            int k = (lo + hi) >>> 1;
            if (hoursNeeded(piles, k) <= h) hi = k;
            else lo = k + 1;
        }
        return lo;
    }

    private long hoursNeeded(int[] piles, int k) {
        long hours = 0;
        for (int p : piles) hours += (p + k - 1) / k;   // ceil division
        return hours;
    }
}
```

### Dry Run / Key Insight
`piles=[3,6,7,11], h=8` → search [1,11]. Feasibility monotone: any k ≥ answer is feasible. Answer collapses to 4.
**Mental model:** Map "smallest k that is fast enough" to the lower-bound binary-search template with predicate = "finishes in time."

### Edge Cases
- `h = piles.length` → answer = `max(piles)` (one pile per hour).
- Single pile → `ceil(p / h)`.
- Watch for **overflow** in `hoursNeeded` → use `long`.

### Follow-ups & Variants
- **Capacity to Ship Packages in D Days** (LC #1011) — same template; feasibility = "can pack in D days with capacity C?"
- **Split Array Largest Sum** (LC #410) — minimize maximum subarray sum with K splits.
- **Magnetic Force / Aggressive Cows** — §3.8.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Try all k | O(maxPile · n) | O(1) |
| Binary search on answer (optimal) | O(n log maxPile) | O(1) |

---

## 5. Time Based Key-Value Store

**LeetCode #981 | Difficulty: Medium**

### Problem
Implement `set(key, value, timestamp)` and `get(key, timestamp)` where `get` returns the value with **the largest timestamp ≤ given** for that key (or `""`).

### Pattern Flashcard
> **Trigger phrase:** "Latest value at or before time T" / "floor of timestamp"
> **Recognize when:**
> - Inserts are in increasing timestamp order (or you can keep sorted lists per key).
> - Lookups want a *floor*.
>
> **Core idea in one line:** Per-key list of `(timestamp, value)`. Lookups = lower-bound binary search for the largest entry with `ts ≤ target`.

### Approaches & Trade-offs

#### Approach 1 — Linear Scan per get
- O(n) per get.

#### Approach 2 — TreeMap per Key
- `floorEntry(timestamp)` is O(log n) but TreeMap has high constants.

#### Approach 3 — ArrayList per Key + Binary Search (optimal)
- O(log n) per get; O(1) per set (timestamps monotone increasing per spec).

### Java Solution (optimal)
```java
class TimeMap {
    private final Map<String, List<int[]>> store = new HashMap<>();  // (ts, valueIdx)
    private final List<String> values = new ArrayList<>();

    public void set(String key, String value, int timestamp) {
        values.add(value);
        store.computeIfAbsent(key, k -> new ArrayList<>())
             .add(new int[]{timestamp, values.size() - 1});
    }

    public String get(String key, int timestamp) {
        List<int[]> list = store.get(key);
        if (list == null) return "";
        int l = 0, r = list.size();
        while (l < r) {
            int m = (l + r) >>> 1;
            if (list.get(m)[0] <= timestamp) l = m + 1;
            else r = m;
        }
        return l == 0 ? "" : values.get(list.get(l - 1)[1]);
    }
}
```

### Dry Run / Key Insight
`set("foo","bar",1); set("foo","bar2",4); get("foo",4) → "bar2"; get("foo",5) → "bar2"; get("foo",0) → ""`.
**Mental model:** lower_bound for "first ts > target" then step back by 1.

### Edge Cases
- Key never set → "".
- Query timestamp before any set → "".

### Follow-ups & Variants
- Out-of-order inserts → switch to `TreeMap<Integer, String>` and `floorEntry`.
- Need history range → store contiguous arrays per key for prefix-style queries.

### Complexity Summary
| Operation | Time | Space |
|---|---|---|
| set | O(1) amortized | O(n) |
| get | O(log n) | — |

---

## 6. Median of Two Sorted Arrays

**LeetCode #4 | Difficulty: Hard**

### Problem
Given two sorted arrays `nums1` and `nums2` of sizes m and n, return the median of the merged sorted array in **O(log(min(m, n)))**.

### Pattern Flashcard
> **Trigger phrase:** "Median of two sorted arrays" / "partition K elements from two sorted lists"
> **Recognize when:**
> - You must beat O(m+n) merge.
> - You need not the full merge — just one or two middle elements.
>
> **Core idea in one line:** Binary-search the partition point of the smaller array; the other array's partition is fixed by the requirement that left halves sum to `(m+n+1)/2`. Adjust until `maxLeft ≤ minRight` for both sides.

### Approaches & Trade-offs

#### Approach 1 — Merge and Pick
- O(m + n).

#### Approach 2 — Find K-th using Binary Search Across Both
- O(log(m + n)). Conceptually similar.

#### Approach 3 — Partition Both Arrays (optimal)
- O(log(min(m, n))).

### Java Solution (optimal)
```java
class Solution {
    public double findMedianSortedArrays(int[] a, int[] b) {
        if (a.length > b.length) return findMedianSortedArrays(b, a);
        int m = a.length, n = b.length;
        int total = m + n, half = (total + 1) / 2;

        int lo = 0, hi = m;
        while (lo <= hi) {
            int i = (lo + hi) >>> 1;
            int j = half - i;

            int aLeft  = i == 0 ? Integer.MIN_VALUE : a[i - 1];
            int aRight = i == m ? Integer.MAX_VALUE : a[i];
            int bLeft  = j == 0 ? Integer.MIN_VALUE : b[j - 1];
            int bRight = j == n ? Integer.MAX_VALUE : b[j];

            if (aLeft <= bRight && bLeft <= aRight) {
                if (total % 2 == 1) return Math.max(aLeft, bLeft);
                return (Math.max(aLeft, bLeft) + Math.min(aRight, bRight)) / 2.0;
            } else if (aLeft > bRight) hi = i - 1;
            else lo = i + 1;
        }
        return 0.0;  // unreachable for valid input
    }
}
```

### Dry Run / Key Insight
`a=[1,3], b=[2]` → total=3, half=2. lo=0, hi=2. Try i=1, j=1: aLeft=1, aRight=3, bLeft=2, bRight=∞ → 1≤∞ ✓, 2≤3 ✓. Odd total → max(1,2)=2.
**Mental model:** Picking `i` elements from `a`'s left and `j = half − i` from `b`'s left covers exactly the "left half" of the merged array. The median sits on the partition boundary.

### Edge Cases
- One array empty → median of the other.
- All elements of one array smaller than the other → partition lands at boundary; sentinels handle it.

### Follow-ups & Variants
- **Find K-th smallest in two sorted arrays** → same technique with offset.
- **Median of M sorted arrays** → much harder; multi-way merge or different approach.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Merge | O(m + n) | O(m + n) |
| Partition (optimal) | O(log(min(m, n))) | O(1) |

---

## 7. Find First and Last Position of Element

**LeetCode #34 | Difficulty: Medium**

### Problem
Given a sorted array and a target value, return `[firstIndex, lastIndex]` of the target, or `[-1, -1]`.

### Pattern Flashcard
> **Trigger phrase:** "Range of equal elements in sorted array" / "leftmost / rightmost occurrence"
> **Recognize when:**
> - Duplicates exist; you need the *boundary*, not just *any* occurrence.
>
> **Core idea in one line:** Two binary searches — one for the **lower bound** (first index ≥ target), one for the **upper bound** (first index > target). Last position = upper − 1.

### Approaches & Trade-offs

#### Approach 1 — Linear scan
- O(n).

#### Approach 2 — Find Once, Walk Both Directions
- O(n) worst case if many duplicates.

#### Approach 3 — Two Binary Searches (optimal)
- O(log n).

### Java Solution (optimal)
```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int lo = lowerBound(nums, target);
        if (lo == nums.length || nums[lo] != target) return new int[]{-1, -1};
        int hi = lowerBound(nums, target + 1) - 1;
        return new int[]{lo, hi};
    }

    private int lowerBound(int[] nums, int target) {
        int l = 0, r = nums.length;
        while (l < r) {
            int m = (l + r) >>> 1;
            if (nums[m] < target) l = m + 1;
            else r = m;
        }
        return l;
    }
}
```

### Dry Run / Key Insight
`[5,7,7,8,8,10], target=8` → lowerBound(8)=3, lowerBound(9)=5 → range [3, 4].

### Edge Cases
- Target not present → first lowerBound check fails.
- Target larger than all elements → lo = n.
- Empty array → returns [-1, -1].

### Follow-ups & Variants
- **Count occurrences of target** → `hi − lo + 1`.
- **First true in monotone predicate** → same lowerBound template generalized.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Linear | O(n) | O(1) |
| Two binary searches (optimal) | O(log n) | O(1) |

---

## 8. Allocate Minimum Number of Pages / Aggressive Cows

**Classic Hard | Difficulty: Hard**

### Problem (Allocate Pages variant)
Given `books[]` (pages) and `m` students, assign each student a contiguous block of books such that the maximum pages assigned to any student is minimized.

### Problem (Aggressive Cows variant)
Place `k` cows in `n` stalls (positions on a number line) to maximize the minimum distance between any two cows.

### Pattern Flashcard
> **Trigger phrase:** "Minimize the maximum" / "maximize the minimum"
> **Recognize when:**
> - Direct optimization is hard, but checking *"is value X feasible?"* is easy and monotone.
>
> **Core idea in one line:** Binary search the answer; feasibility check is a greedy O(n) sweep.

### Approaches & Trade-offs

#### Approach 1 — Try every candidate value
- O((upper − lower) · n).

#### Approach 2 — Binary Search on Answer (optimal)
- O(n log range).

### Java Solution (Allocate Pages — minimize max pages)
```java
class Solution {
    public int allocateBooks(int[] books, int m) {
        if (m > books.length) return -1;
        int lo = 0, hi = 0;
        for (int b : books) { lo = Math.max(lo, b); hi += b; }
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;
            if (canSplit(books, m, mid)) hi = mid;
            else lo = mid + 1;
        }
        return lo;
    }

    private boolean canSplit(int[] books, int m, int maxPages) {
        int students = 1, current = 0;
        for (int b : books) {
            if (current + b > maxPages) { students++; current = b; }
            else current += b;
        }
        return students <= m;
    }
}
```

### Java Solution (Aggressive Cows — maximize min distance)
```java
class Solution {
    public int aggressiveCows(int[] stalls, int k) {
        Arrays.sort(stalls);
        int lo = 1, hi = stalls[stalls.length - 1] - stalls[0];
        while (lo < hi) {
            int mid = (lo + hi + 1) >>> 1;  // upper-bound flavor
            if (canPlace(stalls, k, mid)) lo = mid;
            else hi = mid - 1;
        }
        return lo;
    }

    private boolean canPlace(int[] stalls, int k, int dist) {
        int placed = 1, last = stalls[0];
        for (int i = 1; i < stalls.length; i++) {
            if (stalls[i] - last >= dist) { placed++; last = stalls[i]; }
            if (placed >= k) return true;
        }
        return false;
    }
}
```

### Dry Run / Key Insight
**Allocate Pages**: `books=[12,34,67,90], m=2`. Answer search range `[90, 203]`. Mid=146 → feasible. Mid=118 → feasible. Mid=104 → infeasible. Converges to 113.
**Mental model:** "Smallest maximum that still fits in ≤ m students" maps directly onto the lower-bound template; "largest minimum that's still placeable" maps onto upper-bound (note the `+1` in mid).

### Edge Cases
- More students than books → -1 (or assign empty).
- All books in one student → answer = total sum.
- Single book → answer = that book's pages.

### Follow-ups & Variants
- **Split Array Largest Sum** (LC #410) — identical.
- **Capacity to Ship Packages in D Days** (LC #1011) — same template.
- **Maximum Distance Between Houses with K** — Aggressive Cows.
- **Painter's Partition Problem** — identical to allocate pages.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Try all | O(range · n) | O(1) |
| Binary search on answer (optimal) | O(n log range) | O(1) |
