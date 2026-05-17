# 1. Arrays & Hashing

> The foundational pattern family. Almost every problem here is solved by trading **O(n²) scanning** for **O(n) hashing** — the moment you see "pair / triple / equal / category / count," your hand should reach for `HashMap` or `HashSet`. The other recurring trick is **prefix accumulation**, which converts range questions into single subtractions.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Two Sum | Hash map complement lookup | Easy |
| 2 | Group Anagrams | Sorted-string key OR character-count key | Medium |
| 3 | Top K Frequent Elements | Bucket sort by frequency / min-heap of size K | Medium |
| 4 | Product of Array Except Self | Prefix product + suffix product | Medium |
| 5 | Valid Sudoku | Coordinate-keyed HashSet for row/col/box | Medium |
| 6 | Longest Consecutive Sequence | HashSet + start-of-streak check | Medium |
| 7 | Merge Intervals | Sort by start, merge if overlap | Medium |
| 8 | Insert Interval | Three-phase scan (before / overlap / after) | Medium |
| 9 | Subarray Sum Equals K | Prefix sum + hash map of seen sums | Medium |
| 10 | Next Permutation | Find pivot, swap successor, reverse suffix | Medium |

---

## 1. Two Sum

**LeetCode #1 | Difficulty: Easy**

### Problem
Given an integer array `nums` and an integer `target`, return indices of the two numbers that add up to `target`. Each input has exactly one solution; you may not use the same element twice.

### Pattern Flashcard
> **Trigger phrase:** "Find two numbers that sum to X" / "find pair with complement"
> **Recognize when:**
> - You're asked for a *pair* satisfying an additive condition.
> - The array is **unsorted** and order matters (need original indices).
> - You can afford O(n) extra space.
>
> **Core idea in one line:** As you walk the array, ask the map "have I already seen `target − nums[i]`?"

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- Two nested loops checking every pair.
- Time: `O(n²)` · Space: `O(1)`
- Why it fails: TLE for n > ~10⁴.

#### Approach 2 — Hash Map (optimal)
- Single pass; map value → index. For each `nums[i]`, check if `target − nums[i]` exists in the map.
- Time: `O(n)` · Space: `O(n)`
- Why this wins: Trades one constant-factor lookup for the inner loop.

#### Approach 3 — Sort + Two Pointers
- Sort, then converge pointers. **Loses original indices** unless you store `(value, index)` pairs first.
- Time: `O(n log n)` · Space: `O(n)`
- Trade-off vs Approach 2: Useful only if you also need the values *sorted* downstream, or if memory is tight (in place after pair-array sort).

### Java Solution (optimal)
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (seen.containsKey(complement)) {
                return new int[]{ seen.get(complement), i };
            }
            seen.put(nums[i], i);
        }
        return new int[]{-1, -1};
    }
}
```

### Dry Run / Key Insight
`nums = [2,7,11,15], target = 9` → at `i=0` map is empty, store `{2:0}`. At `i=1`, complement = 9−7 = 2, found → return `[0,1]`.
**Mental model:** the map is your memory of "what I'd need to see later" — equivalently, "what I've already seen that could pair with someone in the future."

### Edge Cases
- Duplicates: `[3,3], target=6` — the *check before put* order ensures the same index isn't used twice.
- Negative numbers / zero / overflow on `target − nums[i]` for extreme `Integer.MIN_VALUE` inputs.

### Follow-ups & Variants
- **Return all pairs** (no "exactly one solution" guarantee) → still hash map, but collect.
- **Sorted input** → two pointers in O(1) extra space.
- **Stream version** (numbers arrive one at a time, queries arrive interleaved) → "Two Sum III" — design class with `add()` and `find()`.
- Related: LC #167 Two Sum II, LC #15 3Sum, LC #454 4Sum II.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute force | O(n²) | O(1) |
| Hash map (optimal) | O(n) | O(n) |
| Sort + two pointers | O(n log n) | O(n) |

---

## 2. Group Anagrams

**LeetCode #49 | Difficulty: Medium**

### Problem
Given an array of strings, group the anagrams together. Return a list of groups, in any order.

### Pattern Flashcard
> **Trigger phrase:** "Group / bucket strings that are rearrangements of each other"
> **Recognize when:**
> - You need to **categorize** strings whose internal order doesn't matter.
> - You need a **canonical key** for each equivalence class.
>
> **Core idea in one line:** Compute a *signature* that is identical for all anagrams (sorted chars, or fixed-length char-count vector), then group by signature in a hash map.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For every pair, check if anagrams using sorted comparison.
- Time: `O(n² · k log k)` (n strings, length k) · Space: `O(nk)`
- Why it fails: Quadratic in n.

#### Approach 2 — Sorted-string Key (optimal, simple)
- Key = sorted string. Group in `Map<String, List<String>>`.
- Time: `O(n · k log k)` · Space: `O(nk)`

#### Approach 3 — Character-Count Key (optimal, faster for long strings)
- Key = `"a3b1c2..."` from 26-length count array (or the array itself wrapped via `Arrays.toString`).
- Time: `O(n · k)` · Space: `O(nk)`
- Trade-off: Faster when `k log k > 26`, more code.

### Java Solution (optimal — char count key)
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> groups = new HashMap<>();
        for (String s : strs) {
            int[] count = new int[26];
            for (char c : s.toCharArray()) count[c - 'a']++;
            String key = Arrays.toString(count);
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        }
        return new ArrayList<>(groups.values());
    }
}
```

### Dry Run / Key Insight
`["eat","tea","tan","ate","nat","bat"]` → keys `[1,0,..,1,..,1..]` collapse `eat/tea/ate` together; `tan/nat` together; `bat` alone.
**Mental model:** anagrams form equivalence classes. Picking a canonical representative is the entire job.

### Edge Cases
- Empty strings (all empty strings form one group).
- Unicode / case sensitivity — if expected, normalize first.

### Follow-ups & Variants
- "Find all anagrams of a pattern in a string" → sliding-window frequency (LC #438).
- "Are two strings anagrams?" → just compare count arrays (LC #242).
- If strings contain *any* unicode characters, use `HashMap<Character,Integer>` or sorted string as key.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute pair check | O(n²·k log k) | O(nk) |
| Sorted-string key | O(n·k log k) | O(nk) |
| Char-count key (optimal) | O(n·k) | O(nk) |

---

## 3. Top K Frequent Elements

**LeetCode #347 | Difficulty: Medium**

### Problem
Given an integer array and integer `k`, return the `k` most frequent elements. Answer is guaranteed unique.

### Pattern Flashcard
> **Trigger phrase:** "Top K / K most frequent / K largest"
> **Recognize when:**
> - You need a *partial* ranking, not the whole order.
> - K is much smaller than n.
>
> **Core idea in one line:** Don't sort all n — maintain a heap of size K, or bucket items by frequency (since frequency ≤ n).

### Approaches & Trade-offs

#### Approach 1 — Sort by Frequency
- Count, then sort entries by count descending, take first K.
- Time: `O(n log n)` · Space: `O(n)`

#### Approach 2 — Min-Heap of size K
- Push entries; when size > K, poll smallest. Final K are the answer.
- Time: `O(n log k)` · Space: `O(n + k)`
- Wins when K ≪ n.

#### Approach 3 — Bucket Sort (optimal)
- Frequencies are bounded by `n`. Bucket `i` holds elements appearing `i` times. Walk buckets from high to low, collect K.
- Time: `O(n)` · Space: `O(n)`
- Best in theory; slightly more code.

### Java Solution (optimal — bucket sort)
```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int x : nums) freq.merge(x, 1, Integer::sum);

        List<Integer>[] buckets = new List[nums.length + 1];
        for (var e : freq.entrySet()) {
            int f = e.getValue();
            if (buckets[f] == null) buckets[f] = new ArrayList<>();
            buckets[f].add(e.getKey());
        }

        int[] result = new int[k];
        int idx = 0;
        for (int i = buckets.length - 1; i >= 0 && idx < k; i--) {
            if (buckets[i] == null) continue;
            for (int x : buckets[i]) {
                if (idx == k) break;
                result[idx++] = x;
            }
        }
        return result;
    }
}
```

### Dry Run / Key Insight
`nums=[1,1,1,2,2,3], k=2` → freq `{1:3, 2:2, 3:1}`. Buckets: `[3]→[1], [2]→[2], [1]→[3]`. Walk from end: pick 1, then 2.
**Mental model:** Frequency is naturally bounded by `n`, so we get a "free" radix that beats comparison-based sorting.

### Edge Cases
- All elements equal: single bucket at index n, contains one element.
- K = n: return everything (skip the heap/bucket logic if you like).

### Follow-ups & Variants
- **K most frequent strings** (LC #692) — tie-break by lexicographic order, heap comparator needs custom logic.
- **Streaming top-K** → min-heap of size K is the right answer; bucket sort doesn't fit a stream.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Sort by freq | O(n log n) | O(n) |
| Min-heap size K | O(n log k) | O(n+k) |
| Bucket sort (optimal) | O(n) | O(n) |

---

## 4. Product of Array Except Self

**LeetCode #238 | Difficulty: Medium**

### Problem
Given `nums`, return array `answer` where `answer[i]` is the product of all elements except `nums[i]`. Solve **without division** and in **O(n)** time.

### Pattern Flashcard
> **Trigger phrase:** "Aggregate of everything except element at i"
> **Recognize when:**
> - You'd reach for division reflexively but can't.
> - Result at index i depends on *prefix* and *suffix* aggregates.
>
> **Core idea in one line:** `answer[i] = prefixProduct(i) × suffixProduct(i)` — compute prefix in one pass left-to-right, then multiply by suffix in one pass right-to-left.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For each i, multiply all others.
- Time: `O(n²)` · Space: `O(1)`

#### Approach 2 — Division
- `total = product of all; answer[i] = total / nums[i]`. **Fails on zero elements** and forbidden by problem.

#### Approach 3 — Prefix + Suffix Arrays
- Two arrays, then combine.
- Time: `O(n)` · Space: `O(n)`

#### Approach 4 — Single Output Array, Suffix on the Fly (optimal)
- Use `answer[]` to store prefix products, then sweep right-to-left tracking a running suffix product.
- Time: `O(n)` · Space: `O(1)` (excluding output)

### Java Solution (optimal)
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];

        answer[0] = 1;
        for (int i = 1; i < n; i++) answer[i] = answer[i - 1] * nums[i - 1];

        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;
            suffix *= nums[i];
        }
        return answer;
    }
}
```

### Dry Run / Key Insight
`nums = [1,2,3,4]` → prefix pass: `[1, 1, 2, 6]`. Suffix pass (right→left, suffix starts at 1): i=3 → 6·1=6, suffix=4; i=2 → 2·4=8, suffix=12; i=1 → 1·12=12, suffix=24; i=0 → 1·24=24. Result `[24,12,8,6]`.

### Edge Cases
- Single zero in the array: only that index gets non-zero answer.
- Multiple zeros: every entry is 0.
- Integer overflow: problem typically guarantees fits in `int`; otherwise use `long`.

### Follow-ups & Variants
- **With division allowed** → one-pass.
- **Max product subarray** (LC #152) — completely different (DP tracking min and max).
- **Range product queries** → segment tree or prefix product array.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Prefix + suffix arrays | O(n) | O(n) |
| In-place (optimal) | O(n) | O(1)* |

\*Excluding the output array.

---

## 5. Valid Sudoku

**LeetCode #36 | Difficulty: Medium**

### Problem
Given a 9×9 board (cells `.` or `'1'..'9'`), determine whether the current state is valid: each row, column, and 3×3 sub-box contains digits without repetition. Empty cells are allowed.

### Pattern Flashcard
> **Trigger phrase:** "No duplicates in row / column / box / region"
> **Recognize when:**
> - You need to *partition* a 2D structure and check uniqueness within each partition.
> - The partition index is derivable from coordinates.
>
> **Core idea in one line:** One pass, three hash sets per cell — row, column, box; box index = `(r/3)*3 + c/3`.

### Approaches & Trade-offs

#### Approach 1 — Three 2D arrays
- `rowSeen[9][9]`, `colSeen[9][9]`, `boxSeen[9][9]` as booleans.
- Time: `O(81)` = O(1) for fixed 9×9. Space: O(1).

#### Approach 2 — HashSet with Encoded Strings (optimal, single set)
- Encode each observation as `"5 in row 3"`, `"5 in col 7"`, `"5 in box 1"`. Add to a single set; collision = invalid.
- Time / Space: O(1) for fixed board, O(n²) in general n×n.
- Trade-off: One unified data structure; clean code.

### Java Solution (optimal — encoded set)
```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        Set<String> seen = new HashSet<>();
        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                char v = board[r][c];
                if (v == '.') continue;
                int box = (r / 3) * 3 + c / 3;
                if (!seen.add(v + "/r" + r)
                 || !seen.add(v + "/c" + c)
                 || !seen.add(v + "/b" + box)) return false;
            }
        }
        return true;
    }
}
```

### Dry Run / Key Insight
Box index = `(r/3)*3 + c/3` maps any cell to one of 9 boxes (0..8). `Set.add` returns false on duplicate, which is exactly the test we want.

### Edge Cases
- Completely empty board → valid (no rule violated).
- Same digit in same cell counted once (we read the cell once).

### Follow-ups & Variants
- **Solve the Sudoku** (LC #37) — full backtracking, see §7.7.
- **N×N variant** — same approach generalizes; box side = √N when N is a perfect square.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 2D arrays | O(81) = O(1) | O(1) |
| Encoded set | O(81) = O(1) | O(1) |

---

## 6. Longest Consecutive Sequence

**LeetCode #128 | Difficulty: Medium**

### Problem
Given an unsorted array of integers, return the length of the longest sequence of consecutive integers. Must run in **O(n)**.

### Pattern Flashcard
> **Trigger phrase:** "Longest run of consecutive numbers in unsorted data"
> **Recognize when:**
> - Sorting would be the obvious O(n log n) solution and you must beat it.
> - You can look up "is x in the set?" in O(1).
>
> **Core idea in one line:** Put everything in a `HashSet`. Only start counting a streak from `x` if `x - 1` is **not** in the set — guarantees each element is the seed of at most one streak.

### Approaches & Trade-offs

#### Approach 1 — Sort
- Sort, then linear scan counting consecutive runs.
- Time: `O(n log n)` · Space: `O(1)` extra (or O(n) for non-primitive).

#### Approach 2 — HashSet with start-of-streak check (optimal)
- Time: `O(n)` amortized (each element entered & exited a streak loop at most once).
- Space: `O(n)`

### Java Solution (optimal)
```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int x : nums) set.add(x);

        int best = 0;
        for (int x : set) {
            if (set.contains(x - 1)) continue;  // not a streak start
            int len = 1;
            while (set.contains(x + len)) len++;
            best = Math.max(best, len);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
`[100,4,200,1,3,2]` → set `{100,4,200,1,3,2}`. Streak starts (no `x-1` present): 100 (len 1), 200 (len 1), 1 (extends to 2,3,4 → len 4). Answer: 4.
**Mental model:** The `x-1` guard is what makes the inner while-loop's total work O(n), not O(n²).

### Edge Cases
- Empty array → 0.
- Duplicates — `HashSet` deduplicates automatically.
- Very negative / `Integer.MIN_VALUE` — `x - 1` underflow is irrelevant because the wraparound value won't be in the set.

### Follow-ups & Variants
- **Return the actual sequence**, not just the length → track start and length together.
- **Allow gap of K** → much harder; sort or sweep-line with TreeSet.
- **Streaming version** → "Data Stream as Disjoint Intervals" (LC #352) — TreeMap of intervals.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Sort | O(n log n) | O(1) / O(n) |
| HashSet (optimal) | O(n) | O(n) |

---

## 7. Merge Intervals

**LeetCode #56 | Difficulty: Medium**

### Problem
Given a collection of `[start, end]` intervals, merge all overlapping intervals and return non-overlapping intervals covering all input.

### Pattern Flashcard
> **Trigger phrase:** "Merge / combine / collapse overlapping ranges"
> **Recognize when:**
> - Intervals are unsorted.
> - You need to output a *reduced* set of intervals.
>
> **Core idea in one line:** Sort by start. Walk the list; if current overlaps the tail of result (`current.start ≤ tail.end`), extend the tail's end; otherwise append.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- Repeatedly find any overlapping pair and merge. O(n²) or worse.

#### Approach 2 — Sort + Sweep (optimal)
- Time: `O(n log n)` (sort dominates) · Space: `O(n)` for output.

#### Approach 3 — Sweep-line (events)
- Treat starts as `+1`, ends as `-1`; merge while running count > 0. Useful generalization but overkill here.

### Java Solution (optimal)
```java
class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        List<int[]> out = new ArrayList<>();
        int[] cur = intervals[0];
        out.add(cur);
        for (int i = 1; i < intervals.length; i++) {
            int[] iv = intervals[i];
            if (iv[0] <= cur[1]) {
                cur[1] = Math.max(cur[1], iv[1]);
            } else {
                cur = iv;
                out.add(cur);
            }
        }
        return out.toArray(new int[0][]);
    }
}
```

### Dry Run / Key Insight
`[[1,3],[2,6],[8,10],[15,18]]` → after sort: same. cur=[1,3]; 2≤3 → cur=[1,6]; 8>6 → push [8,10]; 15>10 → push [15,18]. Result: `[[1,6],[8,10],[15,18]]`.
**Mental model:** sorting by start removes a degree of freedom; you only ever need to compare the new interval to the last merged one.

### Edge Cases
- "Touching" intervals `[1,4],[4,5]` — usually counted as overlapping (`<=`); read the problem.
- Single interval — return as is.

### Follow-ups & Variants
- **Insert one new interval** into a sorted list → see §1.8.
- **Erase / remove overlapping intervals** → see §10.5.
- **Meeting rooms II** (max concurrent intervals) → see §10.6.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(n) |
| Sort + sweep (optimal) | O(n log n) | O(n) |

---

## 8. Insert Interval

**LeetCode #57 | Difficulty: Medium**

### Problem
Given a list of non-overlapping intervals sorted by start, and a new interval, insert it (merging if needed) and return the resulting list, still sorted and non-overlapping.

### Pattern Flashcard
> **Trigger phrase:** "Insert and merge into already-sorted intervals"
> **Recognize when:**
> - The list is **already sorted** — don't re-sort.
> - The answer is naturally in **three phases**: before, overlap, after.
>
> **Core idea in one line:** Walk linearly. Phase 1: copy intervals strictly before. Phase 2: while current overlaps new, expand new. Phase 3: copy remaining.

### Approaches & Trade-offs

#### Approach 1 — Concat + Merge Intervals (§1.7)
- Append new, re-run merge.
- Time: `O(n log n)` · Space: `O(n)`
- Loses the "already sorted" advantage.

#### Approach 2 — Three-Phase Linear Scan (optimal)
- Time: `O(n)` · Space: `O(n)` for output.

### Java Solution (optimal)
```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInt) {
        List<int[]> out = new ArrayList<>();
        int i = 0, n = intervals.length;

        while (i < n && intervals[i][1] < newInt[0]) out.add(intervals[i++]);

        while (i < n && intervals[i][0] <= newInt[1]) {
            newInt[0] = Math.min(newInt[0], intervals[i][0]);
            newInt[1] = Math.max(newInt[1], intervals[i][1]);
            i++;
        }
        out.add(newInt);

        while (i < n) out.add(intervals[i++]);
        return out.toArray(new int[0][]);
    }
}
```

### Dry Run / Key Insight
`intervals=[[1,3],[6,9]], new=[2,5]` → phase 1: nothing (1,3 overlaps 2,5). phase 2: merge → new=[1,5]. phase 3: push [6,9]. Result `[[1,5],[6,9]]`.

### Edge Cases
- New interval before all existing: only phase 3 runs.
- New interval after all existing: only phase 1 runs, then push new.
- Empty intervals list: result is just `[new]`.

### Follow-ups & Variants
- **Calendar problem** (LC #729/731/732) — same template but inserts into a TreeMap for many queries.
- **Stream of intervals** → TreeMap of intervals keyed by start.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Concat + merge | O(n log n) | O(n) |
| Three-phase scan (optimal) | O(n) | O(n) |

---

## 9. Subarray Sum Equals K

**LeetCode #560 | Difficulty: Medium**

### Problem
Given an integer array and integer `k`, return the **number of contiguous subarrays** whose sum equals `k`. Numbers can be negative.

### Pattern Flashcard
> **Trigger phrase:** "Number of subarrays with sum = K" / "with property = X"
> **Recognize when:**
> - Sliding window **won't work** because negatives break monotonicity.
> - Question asks for *count* of subarrays, not just existence.
>
> **Core idea in one line:** Running prefix sum `P[i]`. A subarray `[j+1..i]` sums to `k` iff `P[i] − P[j] = k` iff `P[j] = P[i] − k`. Count how many times `P[i] − k` has been seen.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- Try every (i, j). O(n²) or O(n³).

#### Approach 2 — Prefix-Sum HashMap (optimal)
- Time: `O(n)` · Space: `O(n)`

### Java Solution (optimal)
```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> count = new HashMap<>();
        count.put(0, 1);  // empty prefix
        int prefix = 0, answer = 0;
        for (int x : nums) {
            prefix += x;
            answer += count.getOrDefault(prefix - k, 0);
            count.merge(prefix, 1, Integer::sum);
        }
        return answer;
    }
}
```

### Dry Run / Key Insight
`nums=[1,1,1], k=2` → prefix=1, look for −1 (not seen), store {0:1,1:1}. prefix=2, look for 0 → found 1, ans=1. prefix=3, look for 1 → found 1, ans=2.
**Mental model:** The map answers "how many prefixes can I subtract from the current prefix to land on k?" — that's exactly the count of qualifying subarrays ending at i.

### Edge Cases
- `k = 0` and the array contains zeros — count of `P[i] = P[j]` events.
- Negative numbers — the whole reason a sliding window doesn't work.
- Initial `count.put(0, 1)` is essential: handles subarrays that start at index 0.

### Follow-ups & Variants
- **Longest subarray with sum K** → same map, store **first index** of each prefix sum (LC #325).
- **Continuous Subarray Sum divisible by K** (LC #523) → key = `prefix % k`.
- **Subarray sum equals K, all positive** → sliding window in O(n) without a map.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Prefix + map (optimal) | O(n) | O(n) |

---

## 10. Next Permutation

**LeetCode #31 | Difficulty: Medium**

### Problem
Rearrange numbers into the lexicographically next greater permutation. If no such permutation exists, rearrange into the smallest (sorted ascending). **In place**, O(1) extra space.

### Pattern Flashcard
> **Trigger phrase:** "Next lexicographic permutation" / "next bigger number with same digits"
> **Recognize when:**
> - You need *the immediate successor* in lex order, not all permutations.
> - In-place transformation requested.
>
> **Core idea in one line:** Walk right-to-left, find the first index `i` where `nums[i] < nums[i+1]` (the "pivot"). Swap pivot with smallest element greater than it in the suffix. Reverse the suffix.

### Approaches & Trade-offs

#### Approach 1 — Generate All Permutations and Sort
- Find current's position, return next.
- Time: `O(n!)`. Wildly impractical.

#### Approach 2 — Three-Step In-Place (optimal)
- Time: `O(n)` · Space: `O(1)`

### Java Solution (optimal)
```java
class Solution {
    public void nextPermutation(int[] nums) {
        int n = nums.length, i = n - 2;
        while (i >= 0 && nums[i] >= nums[i + 1]) i--;

        if (i >= 0) {
            int j = n - 1;
            while (nums[j] <= nums[i]) j--;
            swap(nums, i, j);
        }
        reverse(nums, i + 1, n - 1);
    }

    private void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    private void reverse(int[] a, int l, int r) {
        while (l < r) swap(a, l++, r--);
    }
}
```

### Dry Run / Key Insight
`[1,2,3]` → pivot i=1 (nums[1]=2 < nums[2]=3). Swap with smallest greater (3). → `[1,3,2]`. Reverse suffix (just `[2]`). Result `[1,3,2]`.
`[3,2,1]` → no pivot (whole array decreasing). i = −1. Skip swap, reverse all → `[1,2,3]`.
**Mental model:** A strictly decreasing suffix is "already at the end." The pivot is the rightmost place where we can still *increase* something; we increase it minimally, then reset the rest to ascending.

### Edge Cases
- Already largest permutation → returns smallest (sorted) by the reverse step.
- Duplicates: `[1,5,1]` → pivot at i=0, swap with last index > 1 → `[5,1,1]`, reverse suffix → `[5,1,1]`.

### Follow-ups & Variants
- **Previous permutation** — mirror the algorithm with reversed comparators.
- **K-th permutation sequence** (LC #60) — factorial number system.
- **All permutations** → backtracking (§7.3).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute (all perms) | O(n!) | O(n!) |
| In-place (optimal) | O(n) | O(1) |
