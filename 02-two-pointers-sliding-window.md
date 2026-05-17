# 2. Two Pointers & Sliding Window

> Two pointers convert nested-loop ideas into a single linear pass. The two big variants: **converging pointers** (sorted arrays, palindromes, container problems) and **sliding window** (longest/shortest substring satisfying X). The window pattern in particular has a near-universal template — memorize it once and stamp it out.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | 3Sum | Sort + fixed anchor + two pointers | Medium |
| 2 | Container With Most Water | Greedy inward two pointers | Medium |
| 3 | Trapping Rain Water | Two pointers tracking left/right max | Hard |
| 4 | Best Time to Buy and Sell Stock | Track minimum so far, one pass | Easy |
| 5 | Longest Substring Without Repeating Chars | Expanding window + last-seen map | Medium |
| 6 | Longest Repeating Character Replacement | Window with maxFreq bookkeeping | Medium |
| 7 | Permutation in String | Fixed-size window frequency match | Medium |
| 8 | Minimum Window Substring | Variable window, contract on full match | Hard |
| 9 | Sliding Window Maximum | Monotonic decreasing deque | Hard |
| 10 | Max Consecutive Ones III | Window with "at most K zeros" | Medium |
| 11 | Minimum Size Subarray Sum | Shrinking window, all positive | Medium |

### Sliding Window — Universal Template (memorize this)

```java
int l = 0;
// state for window: counters, maps, sums...
for (int r = 0; r < n; r++) {
    addToWindow(arr[r]);
    while (windowIsInvalid()) {
        removeFromWindow(arr[l]);
        l++;
    }
    // now [l..r] is the largest valid window ending at r
    answer = Math.max(answer, r - l + 1);
}
```

For **minimum-window** problems, flip the inner condition: shrink **while window IS valid** and record the minimum each time.

---

## 1. 3Sum

**LeetCode #15 | Difficulty: Medium**

### Problem
Given an integer array `nums`, return all unique triplets `[a,b,c]` such that `a + b + c = 0`. The solution set must not contain duplicate triplets.

### Pattern Flashcard
> **Trigger phrase:** "Find triplet summing to target" / "k-sum"
> **Recognize when:**
> - Triplet (or k-tuple) with additive condition.
> - Original indices don't matter; only values.
>
> **Core idea in one line:** Sort. For each `i`, run two pointers on the right of `i` looking for `−nums[i]`. Skip duplicates at each level.

### Approaches & Trade-offs

#### Approach 1 — Three Nested Loops
- O(n³). TLE for n > ~500.

#### Approach 2 — Hash Set per Anchor
- For each pair (i,j), look up `−(nums[i]+nums[j])` in a set.
- Time: `O(n²)` · Space: `O(n)`. Dedup is messy.

#### Approach 3 — Sort + Two Pointers (optimal)
- Time: `O(n²)` · Space: `O(1)` extra (besides output / sort).
- Clean dedup via "skip equal neighbors."

### Java Solution (optimal)
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();
        int n = nums.length;

        for (int i = 0; i < n - 2; i++) {
            if (nums[i] > 0) break;
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            int l = i + 1, r = n - 1;
            while (l < r) {
                int sum = nums[i] + nums[l] + nums[r];
                if (sum == 0) {
                    result.add(List.of(nums[i], nums[l], nums[r]));
                    while (l < r && nums[l] == nums[l + 1]) l++;
                    while (l < r && nums[r] == nums[r - 1]) r--;
                    l++; r--;
                } else if (sum < 0) {
                    l++;
                } else {
                    r--;
                }
            }
        }
        return result;
    }
}
```

### Dry Run / Key Insight
`[-1,0,1,2,-1,-4]` → sorted `[-4,-1,-1,0,1,2]`. i=0(-4): no pair. i=1(-1): l=2,r=5 → -1+(-1)+2=0 ✓, dedup, advance. l=3,r=4 → -1+0+1=0 ✓. i=2: duplicate of -1, skip. i=3(0): l=4,r=5 → 0+1+2=3 > 0, r--. End.
**Mental model:** Fixing one element reduces 3Sum to 2Sum on a sorted array → two pointers in O(n).

### Edge Cases
- Length < 3 → empty.
- All zeros → one triplet `[0,0,0]`.
- All positive → no solution (early break helps).

### Follow-ups & Variants
- **3Sum Closest** (LC #16) — track min |diff|.
- **4Sum** (LC #18) — extra outer loop; O(n³).
- **k-Sum** — recursive reduction to 2-sum base.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 3 loops | O(n³) | O(1) |
| Hash set | O(n²) | O(n) |
| Sort + two pointers (optimal) | O(n²) | O(1) |

---

## 2. Container With Most Water

**LeetCode #11 | Difficulty: Medium**

### Problem
Given an integer array `height` representing vertical lines on the x-axis, find two lines forming a container that traps the most water. Return the max area.

### Pattern Flashcard
> **Trigger phrase:** "Max area between two indices" / "best pair from both ends"
> **Recognize when:**
> - Area / quantity depends on `min(left, right) × width`.
> - You can always *prove* moving the smaller side inward is safe.
>
> **Core idea in one line:** Two pointers from both ends; always move the **shorter** wall inward (the other choice can only decrease both height and width).

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- All pairs. O(n²).

#### Approach 2 — Two Pointers (optimal)
- Time: `O(n)` · Space: `O(1)`

### Java Solution (optimal)
```java
class Solution {
    public int maxArea(int[] height) {
        int l = 0, r = height.length - 1, best = 0;
        while (l < r) {
            int h = Math.min(height[l], height[r]);
            best = Math.max(best, h * (r - l));
            if (height[l] < height[r]) l++;
            else r--;
        }
        return best;
    }
}
```

### Dry Run / Key Insight
**Why move the shorter side?** Area = `min(h_l, h_r) × (r − l)`. If you move the taller side, width drops *and* `min(...)` cannot increase. Moving the shorter side at least gives the chance of a taller minimum.

### Edge Cases
- All equal heights → answer is `h × (n − 1)`.
- Two elements → just that pair.

### Follow-ups & Variants
- **Trapping Rain Water** (§2.3) — superficially similar, but per-index volume not single pair.
- **Largest Rectangle in Histogram** (§4.7) — looks alike, solved differently (monotonic stack).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Two pointers (optimal) | O(n) | O(1) |

---

## 3. Trapping Rain Water

**LeetCode #42 | Difficulty: Hard**

### Problem
Given an array `height` representing an elevation map (each bar width 1), compute how much rainwater is trapped.

### Pattern Flashcard
> **Trigger phrase:** "Water trapped between bars" / "volume above an elevation map"
> **Recognize when:**
> - Per-index quantity depends on `min(max-to-left, max-to-right) − current`.
>
> **Core idea in one line:** Maintain running `leftMax` and `rightMax` from both ends; at each step, the side with smaller max is locked, and water above that index = `sideMax − height[i]`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For each i, scan left and right for max. O(n²).

#### Approach 2 — Precomputed Max Arrays
- Two arrays `leftMax[i]`, `rightMax[i]`.
- Time: `O(n)` · Space: `O(n)`

#### Approach 3 — Two Pointers (optimal)
- Time: `O(n)` · Space: `O(1)`

#### Approach 4 — Monotonic Stack
- Time: `O(n)` · Space: `O(n)`. Useful if "compute per layer" is asked.

### Java Solution (optimal — two pointers)
```java
class Solution {
    public int trap(int[] height) {
        int l = 0, r = height.length - 1;
        int leftMax = 0, rightMax = 0, water = 0;
        while (l < r) {
            if (height[l] < height[r]) {
                if (height[l] >= leftMax) leftMax = height[l];
                else water += leftMax - height[l];
                l++;
            } else {
                if (height[r] >= rightMax) rightMax = height[r];
                else water += rightMax - height[r];
                r--;
            }
        }
        return water;
    }
}
```

### Dry Run / Key Insight
`[0,1,0,2,1,0,1,3,2,1,2,1]` → expected 6. At each step, whichever side is shorter is "boxed in" by its own running max — so we can commit to that side's contribution and advance.

### Edge Cases
- Strictly increasing or decreasing → 0 water.
- All equal → 0.
- Length < 3 → 0.

### Follow-ups & Variants
- **Trapping Rain Water II** (LC #407) — 2D grid, needs min-heap BFS from boundary.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Two arrays | O(n) | O(n) |
| Two pointers (optimal) | O(n) | O(1) |
| Mono stack | O(n) | O(n) |

---

## 4. Best Time to Buy and Sell Stock

**LeetCode #121 | Difficulty: Easy**

### Problem
Given daily prices, find the maximum profit from a **single** buy/sell. Sell must be after buy.

### Pattern Flashcard
> **Trigger phrase:** "Maximum increase from a low to a later high"
> **Recognize when:**
> - You want a `max(a_j − a_i)` for `i < j`.
>
> **Core idea in one line:** Track the minimum price seen so far; at each step, candidate profit = `price − minSoFar`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- All (i,j). O(n²).

#### Approach 2 — One-Pass Min Tracking (optimal)
- Time: `O(n)` · Space: `O(1)`

### Java Solution (optimal)
```java
class Solution {
    public int maxProfit(int[] prices) {
        int minSeen = Integer.MAX_VALUE, best = 0;
        for (int p : prices) {
            if (p < minSeen) minSeen = p;
            else best = Math.max(best, p - minSeen);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
`[7,1,5,3,6,4]` → minSeen 7→1; profit at 5 is 4, at 6 is 5, at 4 is 3. Answer 5.

### Edge Cases
- Strictly decreasing → 0 (no transaction).
- Length 0 or 1 → 0.

### Follow-ups & Variants
- **II** (LC #122): Multiple transactions → sum every up-step.
- **III** (LC #123): At most 2 transactions → DP with 4 states.
- **IV** (LC #188): At most K transactions → general DP.
- **With cooldown** (LC #309) / **with fee** (LC #714) → state-machine DP.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| One pass (optimal) | O(n) | O(1) |

---

## 5. Longest Substring Without Repeating Characters

**LeetCode #3 | Difficulty: Medium**

### Problem
Given a string `s`, find the length of the longest substring containing all distinct characters.

### Pattern Flashcard
> **Trigger phrase:** "Longest substring/subarray with no duplicates"
> **Recognize when:**
> - Maximize window length under "all distinct" or similar uniqueness constraint.
>
> **Core idea in one line:** Sliding window over `[l..r]`; on a repeated char, jump `l` to *one past the previous occurrence* of that char (use a map storing last index).

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- Every substring, check uniqueness. O(n³).

#### Approach 2 — Sliding Window with HashSet
- Move `l` forward one at a time until duplicate is gone.
- Time: O(n) · Space: O(min(n, alphabet)).

#### Approach 3 — Window with Last-Seen Map (optimal)
- Jump `l` directly. Each char processed in O(1).
- Time: O(n) · Space: O(min(n, alphabet)).

### Java Solution (optimal)
```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> last = new HashMap<>();
        int l = 0, best = 0;
        for (int r = 0; r < s.length(); r++) {
            char c = s.charAt(r);
            if (last.containsKey(c) && last.get(c) >= l) {
                l = last.get(c) + 1;
            }
            last.put(c, r);
            best = Math.max(best, r - l + 1);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
`"abcabcbb"` → at r=3 ('a'), `last['a']=0 >= l(0)` → l = 1. Continue. Best = 3.
**Mental model:** `last` is "ghosts of windows past"; we only care about ghosts inside the current window (`last[c] >= l`).

### Edge Cases
- Empty string → 0.
- All identical → 1.
- ASCII only → use `int[128]` instead of HashMap for ~3× speedup.

### Follow-ups & Variants
- **At most K distinct chars** (LC #340) → window + count map.
- **At most 2 distinct chars** (LC #159).
- **Smallest window with all distinct** → variant of §2.8.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n³) | O(n) |
| Window + set | O(n) | O(σ) |
| Window + map (optimal) | O(n) | O(σ) |

σ = alphabet size.

---

## 6. Longest Repeating Character Replacement

**LeetCode #424 | Difficulty: Medium**

### Problem
Given a string `s` and integer `k`, you may replace any `k` characters with any letter. Return the length of the longest substring containing the same letter after up to `k` replacements.

### Pattern Flashcard
> **Trigger phrase:** "Longest substring after K modifications"
> **Recognize when:**
> - You can change at most K characters in a window.
> - "Same letter" can be relaxed to "almost same letter."
>
> **Core idea in one line:** A window `[l..r]` is valid iff `(r − l + 1) − maxFreqInWindow ≤ k`. Track `maxFreq` and shrink only when invalid.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- All windows × all target letters. O(n²·26).

#### Approach 2 — Sliding Window with Frequency Map (optimal)
- Track count[26] and a running `maxFreq`.
- Time: O(n) · Space: O(1) (26 letters).
- Subtlety: We never *decrease* `maxFreq` even though it may be stale — the answer only grows, so stale values don't shrink the answer.

### Java Solution (optimal)
```java
class Solution {
    public int characterReplacement(String s, int k) {
        int[] count = new int[26];
        int l = 0, maxFreq = 0, best = 0;
        for (int r = 0; r < s.length(); r++) {
            maxFreq = Math.max(maxFreq, ++count[s.charAt(r) - 'A']);
            if (r - l + 1 - maxFreq > k) {
                count[s.charAt(l) - 'A']--;
                l++;
            }
            best = Math.max(best, r - l + 1);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
`s="AABABBA", k=1` → window grows to "AABA" (maxFreq=3, allowed since 4−3=1≤1). Adds 'B' → "AABAB" (5−3=2 > 1) → slide. Continues; best length 4.
**Mental model:** The "winning letter" in a window is whichever has max frequency. Everything else must be replaceable within K. The check `windowLen − maxFreq ≤ k` is exactly that.

### Edge Cases
- k ≥ n → answer is n.
- All same letter → answer is n.

### Follow-ups & Variants
- **Max Consecutive Ones III** (§2.10) — special case (2-letter alphabet).
- **Longest subarray with at most K distinct** → different formulation.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²·σ) | O(σ) |
| Window + count (optimal) | O(n) | O(σ) |

---

## 7. Permutation in String

**LeetCode #567 | Difficulty: Medium**

### Problem
Given strings `s1` and `s2`, return `true` if `s2` contains a permutation of `s1` as a substring.

### Pattern Flashcard
> **Trigger phrase:** "Anagram / permutation appears as substring"
> **Recognize when:**
> - The "needle" has a **fixed length**.
> - You must match multiset, not order.
>
> **Core idea in one line:** Slide a fixed-size window of length `|s1|` over `s2`; check if the window's char-count matches `s1`'s.

### Approaches & Trade-offs

#### Approach 1 — Sort Every Window
- O(n·k log k).

#### Approach 2 — Fixed Window with Count Arrays + Match Counter (optimal)
- Track how many of the 26 letters currently have matching counts.
- Time: O(n) · Space: O(1).

### Java Solution (optimal)
```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        if (s1.length() > s2.length()) return false;
        int[] need = new int[26], have = new int[26];
        for (char c : s1.toCharArray()) need[c - 'a']++;
        int k = s1.length(), matches = 0;
        for (int i = 0; i < 26; i++) if (need[i] == 0) matches++;

        for (int r = 0; r < s2.length(); r++) {
            int in = s2.charAt(r) - 'a';
            have[in]++;
            if (have[in] == need[in]) matches++;
            else if (have[in] == need[in] + 1) matches--;

            if (r >= k) {
                int out = s2.charAt(r - k) - 'a';
                have[out]--;
                if (have[out] == need[out]) matches++;
                else if (have[out] == need[out] - 1) matches--;
            }
            if (matches == 26) return true;
        }
        return false;
    }
}
```

### Dry Run / Key Insight
`s1="ab", s2="eidbaooo"` → window "ei", "id", "db", "ba" — "ba" matches.
**Mental model:** `matches` ranges 0..26 and changes by ±1 per character touched, so each step is O(1).

### Edge Cases
- `s1.length() > s2.length()` → return false.
- Characters outside `a..z` → adjust alphabet.

### Follow-ups & Variants
- **Find All Anagrams in a String** (LC #438) — same template, collect indices.
- **Substring with concat of all words** (LC #30) — fixed-window over word-multiset.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Sort per window | O(n·k log k) | O(k) |
| Count + matches (optimal) | O(n) | O(σ) |

---

## 8. Minimum Window Substring

**LeetCode #76 | Difficulty: Hard**

### Problem
Given strings `s` and `t`, return the minimum window in `s` that contains every character of `t` (with multiplicity). If none, return `""`.

### Pattern Flashcard
> **Trigger phrase:** "Smallest window containing all characters / all required items"
> **Recognize when:**
> - You need the **shortest** substring satisfying some "contains all of X" condition.
> - Variable window size.
>
> **Core idea in one line:** Expand `r` until the window has all required chars (`have == need`), then **shrink** `l` while still valid, recording the minimum.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- All substrings, check coverage. O(n²·σ).

#### Approach 2 — Sliding Window with Counter (optimal)
- Track `have` (number of distinct chars currently satisfying their need count) vs `need` (number of distinct chars required).
- Time: O(n + m) · Space: O(σ).

### Java Solution (optimal)
```java
class Solution {
    public String minWindow(String s, String t) {
        if (t.isEmpty() || s.length() < t.length()) return "";
        int[] need = new int[128];
        for (char c : t.toCharArray()) need[c]++;
        int required = t.length();          // total chars (with multiplicity) needed
        int l = 0, bestL = 0, bestLen = Integer.MAX_VALUE;
        for (int r = 0; r < s.length(); r++) {
            if (need[s.charAt(r)]-- > 0) required--;
            while (required == 0) {
                if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
                if (++need[s.charAt(l)] > 0) required++;
                l++;
            }
        }
        return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestL, bestL + bestLen);
    }
}
```

### Dry Run / Key Insight
`s="ADOBECODEBANC", t="ABC"` → first valid window "ADOBEC" (len 6), shrink → invalid. Continue, eventually "BANC" (len 4) becomes the answer.
**Mental model:** Use `need[]` as both a *requirement* (positive values) and a *surplus* (zero/negative). Decrement on add; increment on remove. `required == 0` means every char satisfied.

### Edge Cases
- `t` empty → "" by spec.
- `t` longer than `s` → "".
- Duplicates in `t`: handled by `need[c]--` tracking multiplicity.

### Follow-ups & Variants
- **Min Window Subsequence** (LC #727) → different (DP).
- **Smallest range covering K lists** (LC #632) → multi-pointer on K sorted lists with min-heap.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²·σ) | O(σ) |
| Sliding window (optimal) | O(n + m) | O(σ) |

---

## 9. Sliding Window Maximum

**LeetCode #239 | Difficulty: Hard**

### Problem
Given an array `nums` and window size `k`, return an array of max values for every contiguous window of size `k`.

### Pattern Flashcard
> **Trigger phrase:** "Max / min of every window of size K"
> **Recognize when:**
> - You need a rolling extreme, not just one global.
> - Brute is O(n·k); want O(n).
>
> **Core idea in one line:** Maintain a **monotonic decreasing deque of indices**. Front is the current window's max. Evict indices that fall outside the window from the front; pop smaller elements from the back before adding new.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- O(n·k).

#### Approach 2 — Max-Heap
- O(n log n); lazy deletion of stale entries.

#### Approach 3 — Monotonic Deque (optimal)
- Time: O(n) · Space: O(k).
- Each index pushed/popped at most once.

### Java Solution (optimal)
```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] result = new int[n - k + 1];
        Deque<Integer> dq = new ArrayDeque<>();  // indices, values decreasing
        for (int i = 0; i < n; i++) {
            while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();
            while (!dq.isEmpty() && nums[dq.peekLast()] < nums[i]) dq.pollLast();
            dq.offerLast(i);
            if (i >= k - 1) result[i - k + 1] = nums[dq.peekFirst()];
        }
        return result;
    }
}
```

### Dry Run / Key Insight
`[1,3,-1,-3,5,3,6,7], k=3` → windows: [1,3,-1]→3, [3,-1,-3]→3, [-1,-3,5]→5, [-3,5,3]→5, [5,3,6]→6, [3,6,7]→7.
**Mental model:** The deque is "candidates for being the window max, in decreasing order." Smaller values get evicted because a larger value to their right makes them irrelevant forever.

### Edge Cases
- k = 1 → answer is `nums` itself.
- k = n → single value, the global max.

### Follow-ups & Variants
- **Sliding window min** → mirror with increasing deque.
- **Median in a sliding window** (LC #480) → two heaps or TreeMap.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(nk) | O(1) |
| Heap | O(n log n) | O(n) |
| Mono deque (optimal) | O(n) | O(k) |

---

## 10. Max Consecutive Ones III

**LeetCode #1004 | Difficulty: Medium**

### Problem
Given a binary array `nums` and integer `k`, return the longest subarray of 1s after flipping at most `k` zeros to 1s.

### Pattern Flashcard
> **Trigger phrase:** "Longest run of 1s allowing K flips" / "longest valid after up to K modifications"
> **Recognize when:**
> - Binary array, fixed budget of changes.
> - You want longest **valid** window.
>
> **Core idea in one line:** Window can contain at most K zeros. Slide window; when zero-count exceeds K, advance `l`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- All windows. O(n²).

#### Approach 2 — Sliding Window (optimal)
- Time: O(n) · Space: O(1).

### Java Solution (optimal)
```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int l = 0, zeros = 0, best = 0;
        for (int r = 0; r < nums.length; r++) {
            if (nums[r] == 0) zeros++;
            while (zeros > k) {
                if (nums[l] == 0) zeros--;
                l++;
            }
            best = Math.max(best, r - l + 1);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
`[1,1,1,0,0,0,1,1,1,1,0], k=2` → window grows to contain at most 2 zeros; the longest valid run reaches length 6.

### Edge Cases
- k = 0 → longest run of pure 1s.
- All 1s → n.
- All 0s, k ≥ n → n.

### Follow-ups & Variants
- **Longest subarray with at most K of something** — same template generalizes.
- **Max Consecutive Ones II** (one flip) → k=1 special case.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Sliding window (optimal) | O(n) | O(1) |

---

## 11. Minimum Size Subarray Sum

**LeetCode #209 | Difficulty: Medium**

### Problem
Given array of **positive** integers `nums` and integer `target`, return the minimal length of a contiguous subarray with sum ≥ `target`. Return 0 if none exists.

### Pattern Flashcard
> **Trigger phrase:** "Smallest window with sum ≥ target" / "shortest qualifying subarray"
> **Recognize when:**
> - All values positive (monotone window sums) — sliding window legal.
> - You want the **minimum** length, not maximum.
>
> **Core idea in one line:** Grow `r` to add to sum. Whenever sum ≥ target, shrink `l` and record min length.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- O(n²).

#### Approach 2 — Sliding Window (optimal, positives only)
- Time: O(n) · Space: O(1).

#### Approach 3 — Prefix Sum + Binary Search
- For each prefix `P[i]`, binary-search smallest j with `P[j] − P[i] ≥ target`.
- Time: O(n log n) · Space: O(n). Useful when negatives exist (then sliding window fails).

### Java Solution (optimal)
```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int l = 0, sum = 0, best = Integer.MAX_VALUE;
        for (int r = 0; r < nums.length; r++) {
            sum += nums[r];
            while (sum >= target) {
                best = Math.min(best, r - l + 1);
                sum -= nums[l++];
            }
        }
        return best == Integer.MAX_VALUE ? 0 : best;
    }
}
```

### Dry Run / Key Insight
`target=7, nums=[2,3,1,2,4,3]` → sum reaches 8 at r=4, shrink to `[4,3]` (len 2). Answer 2.
**Mental model:** Positive numbers → adding to the window monotonically increases sum; removing monotonically decreases. The window's validity is monotone in `r` for fixed `l`, which is exactly what a sliding window requires.

### Edge Cases
- No valid subarray → return 0.
- A single element ≥ target → length 1.
- All elements ≥ target → length 1.

### Follow-ups & Variants
- **With negatives** (LC #862 — Shortest Subarray with Sum at Least K) → monotonic deque on prefix sums (Hard).
- **Maximum-length subarray with sum ≤ target** → flip direction.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Sliding window (optimal) | O(n) | O(1) |
| Prefix sum + binary search | O(n log n) | O(n) |
