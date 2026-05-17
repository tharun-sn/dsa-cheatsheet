# 9. Dynamic Programming

> DP is recursion that *remembers*. Every DP problem boils down to two questions:
> 1. **What's the state?** (the minimal set of variables that uniquely identifies a subproblem)
> 2. **What's the transition?** (how does this state's answer depend on smaller states?)
>
> If you can write the recursion as a clean function `solve(state) = combine(solve(smaller_states))`, you have a DP. Memoization turns top-down into O(states · work-per-state); tabulation makes it iterative and often saves space.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Climbing Stairs / House Robber | 1D DP, O(1) space rolling | Easy |
| 2 | House Robber II | Run 1D DP on two ranges | Medium |
| 3 | Longest Palindromic Substring | Expand around center | Medium |
| 4 | Decode Ways | 1D DP on parsing positions | Medium |
| 5 | Coin Change | Unbounded knapsack (min) | Medium |
| 6 | 0/1 Knapsack | Pick-or-skip 2D, can be 1D | Medium |
| 7 | Maximum Product Subarray | Track running max AND min | Medium |
| 8 | Word Break | 1D DP over string positions | Medium |
| 9 | Longest Increasing Subsequence | Patience sort + binary search | Medium |
| 10 | Partition Equal Subset Sum | Subset-sum DP | Medium |
| 11 | Unique Paths | 2D grid DP | Medium |
| 12 | Longest Common Subsequence | 2D DP across two strings | Medium |
| 13 | Edit Distance | 2D DP with 3 transitions | Hard |
| 14 | Matrix Chain Multiplication | Interval DP | Hard |
| 15 | Coin Change II | Unbounded knapsack (count) | Medium |
| 16 | Target Sum | Sign assignment → subset sum | Medium |

---

## 1. Climbing Stairs / House Robber

**LeetCode #70 / #198 | Difficulty: Easy / Medium**

### Problem (#70)
Each step climbs 1 or 2 stairs. How many distinct ways to reach step n?

### Problem (#198)
Houses on a street, each with `nums[i]` money. Can't rob two adjacent. Max money?

### Pattern Flashcard
> **Trigger phrase:** "Each step depends on the last one or two"
> **Recognize when:**
> - Linear state with O(1) prior dependencies.
>
> **Core idea in one line (Stairs):** `f(n) = f(n−1) + f(n−2)`.
> **Core idea in one line (Robber):** `dp[i] = max(dp[i−1], dp[i−2] + nums[i])`.

### Approaches & Trade-offs

#### Approach 1 — Recursion
- Exponential without memoization.

#### Approach 2 — Tabulation (full array)
- O(n) time / O(n) space.

#### Approach 3 — Rolling Two Variables (optimal)
- O(n) / O(1).

### Java Solution (Climbing Stairs — optimal)
```java
class Solution {
    public int climbStairs(int n) {
        int a = 1, b = 1;
        for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
        return b;
    }
}
```

### Java Solution (House Robber — optimal)
```java
class Solution {
    public int rob(int[] nums) {
        int prev2 = 0, prev1 = 0;
        for (int x : nums) { int cur = Math.max(prev1, prev2 + x); prev2 = prev1; prev1 = cur; }
        return prev1;
    }
}
```

### Edge Cases
- n = 0 or empty array → result 0 (or 1 for stairs depending on spec).
- n = 1 → that single value.

### Follow-ups & Variants
- **House Robber II** (§9.2) — circular street.
- **Min Cost Climbing Stairs** (LC #746).
- **Decode Ways** (§9.4) — similar 1-or-2 transition but with validity check.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Recursion | O(2ⁿ) | O(n) |
| Tabulation | O(n) | O(n) |
| Rolling (optimal) | O(n) | O(1) |

---

## 2. House Robber II

**LeetCode #213 | Difficulty: Medium**

### Problem
Same as House Robber but houses are arranged in a **circle** — house 0 and house n−1 are adjacent.

### Pattern Flashcard
> **Trigger phrase:** "Circular array, can't take first and last together"
> **Core idea in one line:** Run linear House Robber on `nums[0..n-2]` and `nums[1..n-1]`; answer is the max.

### Java Solution (optimal)
```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];
        return Math.max(robLinear(nums, 0, nums.length - 2),
                        robLinear(nums, 1, nums.length - 1));
    }
    private int robLinear(int[] nums, int l, int r) {
        int prev2 = 0, prev1 = 0;
        for (int i = l; i <= r; i++) { int cur = Math.max(prev1, prev2 + nums[i]); prev2 = prev1; prev1 = cur; }
        return prev1;
    }
}
```

### Edge Cases
- Single house → that value.
- Two houses → max of the two.

### Follow-ups & Variants
- **House Robber III** (LC #337) — trees instead of arrays; bottom-up DP returning `(robbed, skipped)` pairs.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Two passes (optimal) | O(n) | O(1) |

---

## 3. Longest Palindromic Substring

**LeetCode #5 | Difficulty: Medium**

### Problem
Return the longest palindromic substring of `s`.

### Pattern Flashcard
> **Trigger phrase:** "Longest palindromic substring"
> **Core idea in one line:** Expand around each potential center (2n − 1 centers including between-characters); track the longest.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- O(n³).

#### Approach 2 — 2D DP `pal[i][j]`
- O(n²) time / O(n²) space.

#### Approach 3 — Expand Around Center (optimal-readable)
- O(n²) time / O(1) space.

#### Approach 4 — Manacher's Algorithm
- O(n). Specialized; rarely required.

### Java Solution (optimal — expand around center)
```java
class Solution {
    private int start = 0, maxLen = 0;

    public String longestPalindrome(String s) {
        for (int i = 0; i < s.length(); i++) {
            expand(s, i, i);
            expand(s, i, i + 1);
        }
        return s.substring(start, start + maxLen);
    }

    private void expand(String s, int l, int r) {
        while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
        int len = r - l - 1;
        if (len > maxLen) { maxLen = len; start = l + 1; }
    }
}
```

### Edge Cases
- Empty string → "".
- Single char → that char.
- All same chars → the whole string.

### Follow-ups & Variants
- **Palindromic Substrings** (LC #647) — count instead of return longest.
- **Longest Palindromic Subsequence** (LC #516) — different problem (not contiguous), 2D DP.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n³) | O(1) |
| 2D DP | O(n²) | O(n²) |
| Expand center (optimal) | O(n²) | O(1) |
| Manacher | O(n) | O(n) |

---

## 4. Decode Ways

**LeetCode #91 | Difficulty: Medium**

### Problem
A message of digits is decoded by mapping `1→A, …, 26→Z`. Given the digit string, return the number of distinct decodings.

### Pattern Flashcard
> **Trigger phrase:** "Number of parses / decodings"
> **Core idea in one line:** `dp[i]` = ways for prefix `s[0..i]`. Add `dp[i-1]` if `s[i] != '0'`; add `dp[i-2]` if `s[i-1..i]` is in `10..26`.

### Java Solution (optimal — rolling two)
```java
class Solution {
    public int numDecodings(String s) {
        if (s.charAt(0) == '0') return 0;
        int prev2 = 1, prev1 = 1;
        for (int i = 1; i < s.length(); i++) {
            int cur = 0;
            if (s.charAt(i) != '0') cur += prev1;
            int two = Integer.parseInt(s.substring(i - 1, i + 1));
            if (two >= 10 && two <= 26) cur += prev2;
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
}
```

### Edge Cases
- Leading zero → 0.
- "10" → 1; "27" → 1 (only "2" + "7").
- "00" → 0; "06" → 0.

### Follow-ups & Variants
- **Decode Ways II** (LC #639) — '*' wildcard; tricky case analysis.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 1D DP (optimal) | O(n) | O(1) |

---

## 5. Coin Change

**LeetCode #322 | Difficulty: Medium**

### Problem
Given coin denominations and a target `amount`, return the **fewest coins** needed (or −1 if impossible). Unlimited supply per denomination.

### Pattern Flashcard
> **Trigger phrase:** "Min coins / min steps to reach value"
> **Recognize when:**
> - Unbounded knapsack: each item reusable, count or sum target.
>
> **Core idea in one line:** `dp[v]` = min coins for amount v. `dp[v] = 1 + min(dp[v − c]) for c in coins`.

### Approaches & Trade-offs

#### Approach 1 — Greedy (largest first)
- **Wrong in general** (e.g., coins=[1,3,4], amount=6 → greedy 4+1+1=3, optimal 3+3=2).

#### Approach 2 — DP (optimal)
- O(amount · #coins).

#### Approach 3 — BFS for "min steps"
- O(amount · #coins). Equivalent.

### Java Solution (optimal — DP)
```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;
        for (int v = 1; v <= amount; v++) {
            for (int c : coins) if (c <= v) dp[v] = Math.min(dp[v], dp[v - c] + 1);
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

### Edge Cases
- amount = 0 → 0.
- All coins > amount → -1.
- Coin == 1 present → always solvable.

### Follow-ups & Variants
- **Coin Change II** (§9.15) — count combinations, not min coins.
- **Combination Sum IV** (LC #377) — permutations counted (order matters).
- **Perfect Squares** (LC #279) — same template with squares as denominations.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DP (optimal) | O(amount · k) | O(amount) |

---

## 6. 0/1 Knapsack Problem

**Classic | Difficulty: Medium**

### Problem
Given weights and values of `n` items and a capacity `W`, choose a subset (each item at most once) maximizing total value subject to total weight ≤ W.

### Pattern Flashcard
> **Trigger phrase:** "Pick or skip each item once, capacity constraint"
> **Recognize when:**
> - Items distinct (no reuse).
> - Two dimensions: items and capacity.
>
> **Core idea in one line:** `dp[i][w]` = max value using first `i` items at capacity `w`. Choice: skip (`dp[i−1][w]`) or take if `wt[i] ≤ w` (`dp[i−1][w − wt[i]] + val[i]`).

### Approaches & Trade-offs

#### Approach 1 — 2D DP (clearest)
- O(nW) time / O(nW) space.

#### Approach 2 — 1D DP (rolling) (optimal space)
- Iterate weight **descending** to avoid reusing the same item.
- O(nW) time / O(W) space.

### Java Solution (optimal — 1D)
```java
class Knapsack {
    public int maxValue(int W, int[] wt, int[] val) {
        int n = wt.length;
        int[] dp = new int[W + 1];
        for (int i = 0; i < n; i++) {
            for (int w = W; w >= wt[i]; w--) {       // DESC: each item used at most once
                dp[w] = Math.max(dp[w], dp[w - wt[i]] + val[i]);
            }
        }
        return dp[W];
    }
}
```

### Dry Run / Key Insight
For **unbounded** knapsack (items reusable), iterate `w` ascending — that intentionally lets the same item be picked again.

### Edge Cases
- W = 0 or no items → 0.
- All items heavier than W → 0.

### Follow-ups & Variants
- **Subset Sum** / **Partition Equal Subset Sum** (§9.10) — value = weight; ask boolean reachability.
- **Target Sum** (§9.16) — transformed into subset sum.
- **Unbounded Knapsack** (Rod Cutting, Coin Change) — different inner-loop direction.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 2D | O(nW) | O(nW) |
| 1D (optimal) | O(nW) | O(W) |

---

## 7. Maximum Product Subarray

**LeetCode #152 | Difficulty: Medium**

### Problem
Find the contiguous subarray with the largest **product** and return that product. Numbers may be negative.

### Pattern Flashcard
> **Trigger phrase:** "Max product / running product with negatives"
> **Recognize when:**
> - Negatives can flip min ↔ max.
>
> **Core idea in one line:** Track both `maxSoFar` and `minSoFar` ending at i; a new max can come from `min × negative`.

### Java Solution (optimal)
```java
class Solution {
    public int maxProduct(int[] nums) {
        int maxEnd = nums[0], minEnd = nums[0], best = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int x = nums[i];
            int newMax = Math.max(x, Math.max(maxEnd * x, minEnd * x));
            int newMin = Math.min(x, Math.min(maxEnd * x, minEnd * x));
            maxEnd = newMax; minEnd = newMin;
            best = Math.max(best, maxEnd);
        }
        return best;
    }
}
```

### Edge Cases
- Single element → that element.
- Contains zero → the running window resets at zero.
- All negatives → answer might be a single number or an even-count product.

### Follow-ups & Variants
- **Maximum Subarray (sum)** (LC #53) — single state (running sum) — Kadane's algorithm.
- **Max Sum of Rectangle No Larger Than K** (LC #363) — much harder.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Two-state DP (optimal) | O(n) | O(1) |

---

## 8. Word Break

**LeetCode #139 | Difficulty: Medium**

### Problem
Given a string `s` and dictionary `wordDict`, return whether `s` can be segmented into space-separated dictionary words.

### Pattern Flashcard
> **Trigger phrase:** "Can the string be split into known pieces?"
> **Core idea in one line:** `dp[i]` = `s[0..i]` is segmentable. `dp[i] = OR over j: dp[j] && s[j..i] in dict`.

### Approaches & Trade-offs

#### Approach 1 — Brute-force Recursion
- Exponential.

#### Approach 2 — DP (optimal)
- O(n² · L) where L = avg word length (substring + set lookup).

#### Approach 3 — Trie-based DFS with Memoization
- Better when dictionary is enormous and shares prefixes.

### Java Solution (optimal — DP)
```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> dict = new HashSet<>(wordDict);
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && dict.contains(s.substring(j, i))) { dp[i] = true; break; }
            }
        }
        return dp[n];
    }
}
```

### Edge Cases
- Empty string → true (vacuously).
- Dictionary empty → false unless s empty.

### Follow-ups & Variants
- **Word Break II** (LC #140) — return all valid sentences; backtracking + memo.
- **Concatenated Words** (LC #472) — Word Break per candidate word.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | exponential | O(n) |
| DP (optimal) | O(n² · L) | O(n) |

---

## 9. Longest Increasing Subsequence

**LeetCode #300 | Difficulty: Medium**

### Problem
Length of the longest strictly increasing subsequence in `nums`.

### Pattern Flashcard
> **Trigger phrase:** "Longest increasing / strictly monotone subsequence"
> **Recognize when:**
> - Need length, not the actual sequence (though reconstructable).
>
> **Core idea in one line (DP):** `dp[i] = 1 + max(dp[j]) for j < i, nums[j] < nums[i]`. O(n²).
> **Core idea in one line (Patience):** Maintain a sorted "tails" array; for each x, binary-search the first tail ≥ x and replace it. Final length = tails.length.

### Approaches & Trade-offs

#### Approach 1 — DP O(n²)
- Easy to write; sometimes sufficient.

#### Approach 2 — Patience Sort + Binary Search (optimal)
- O(n log n) · O(n).

### Java Solution (optimal — patience)
```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] tails = new int[nums.length];
        int size = 0;
        for (int x : nums) {
            int l = 0, r = size;
            while (l < r) {
                int m = (l + r) >>> 1;
                if (tails[m] < x) l = m + 1;
                else r = m;
            }
            tails[l] = x;
            if (l == size) size++;
        }
        return size;
    }
}
```

### Dry Run / Key Insight
`tails[k]` = smallest tail value among all increasing subsequences of length k+1. The array is sorted by construction; `tails` is **not** the LIS itself, but its length equals the LIS length.

### Edge Cases
- Empty → 0.
- All equal → 1.
- Strictly decreasing → 1.

### Follow-ups & Variants
- **Russian Doll Envelopes** (LC #354) — sort by width asc, height desc → LIS on heights.
- **Longest Increasing Path in Matrix** (LC #329) — DFS + memo.
- **Reconstruct the actual LIS** — store predecessor indices during binary-search update.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DP | O(n²) | O(n) |
| Patience (optimal) | O(n log n) | O(n) |

---

## 10. Partition Equal Subset Sum

**LeetCode #416 | Difficulty: Medium**

### Problem
Given a non-empty array of positive integers, determine whether it can be partitioned into two subsets of equal sum.

### Pattern Flashcard
> **Trigger phrase:** "Can split into equal-sum halves?"
> **Recognize when:**
> - Total sum even is a necessary condition.
> - Reduces to: can we pick a subset summing to `total / 2`?
>
> **Core idea in one line:** Subset-sum DP. `dp[s] = true` iff some subset reaches `s`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force / Backtracking
- 2ⁿ subsets.

#### Approach 2 — 2D DP (boolean grid)
- O(nT) time / O(nT) space (T = target).

#### Approach 3 — 1D Bitset DP (optimal)
- O(nT / word_size). Java: use `BitSet`.

### Java Solution (optimal — boolean[] rolling)
```java
class Solution {
    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int x : nums) total += x;
        if (total % 2 != 0) return false;
        int target = total / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;
        for (int x : nums) {
            for (int s = target; s >= x; s--) dp[s] = dp[s] || dp[s - x];
            if (dp[target]) return true;
        }
        return dp[target];
    }
}
```

### Java Solution (BitSet version — fastest in practice)
```java
class Solution {
    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int x : nums) total += x;
        if (total % 2 != 0) return false;
        int target = total / 2;
        BitSet bs = new BitSet(target + 1);
        bs.set(0);
        for (int x : nums) {
            // shift bs left by x and OR into itself: each existing reachable sum s spawns s + x
            BitSet shifted = new BitSet(target + 1);
            for (int s = bs.nextSetBit(0); s >= 0 && s + x <= target; s = bs.nextSetBit(s + 1)) {
                shifted.set(s + x);
            }
            bs.or(shifted);
            if (bs.get(target)) return true;
        }
        return bs.get(target);
    }
}
```

### Edge Cases
- Odd total sum → false immediately.
- Single element → false (can't split nonempty single).
- All zeros → true.

### Follow-ups & Variants
- **Target Sum** (§9.16) — same DP after sign-substitution math.
- **Partition to K Equal Sum Subsets** (LC #698) — backtracking; DP infeasible.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(2ⁿ) | O(n) |
| 1D DP (optimal) | O(nT) | O(T) |
| BitSet | O(nT / 64) | O(T) |

---

## 11. Unique Paths

**LeetCode #62 | Difficulty: Medium**

### Problem
A robot starts at top-left of an `m × n` grid and wants to reach bottom-right, moving only right or down. How many unique paths?

### Pattern Flashcard
> **Trigger phrase:** "Lattice paths / count paths in grid"
> **Core idea in one line:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. Closed-form: `C(m + n - 2, m - 1)`.

### Java Solution (optimal — 1D DP)
```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 1; i < m; i++)
            for (int j = 1; j < n; j++) dp[j] += dp[j - 1];
        return dp[n - 1];
    }
}
```

### Java Solution (combinatorial, closed-form)
```java
class Solution {
    public int uniquePaths(int m, int n) {
        long ans = 1;
        for (int i = 1; i <= m - 1; i++) {
            ans = ans * (n - 1 + i) / i;
        }
        return (int) ans;
    }
}
```

### Edge Cases
- 1×n or m×1 → 1 path.
- m = n = 1 → 1.

### Follow-ups & Variants
- **Unique Paths II** (LC #63) — with obstacles; treat as 0 paths through obstacle cells.
- **Minimum Path Sum** (LC #64) — same grid, sum metric.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 1D DP (optimal) | O(mn) | O(n) |
| Combinatorial | O(min(m, n)) | O(1) |

---

## 12. Longest Common Subsequence

**LeetCode #1143 | Difficulty: Medium**

### Problem
Given two strings `text1`, `text2`, return the length of their longest common subsequence.

### Pattern Flashcard
> **Trigger phrase:** "Longest common (subseq / supersequence / edit relation) of two strings"
> **Core idea in one line:** `dp[i][j]` = LCS length of `text1[0..i-1]`, `text2[0..j-1]`. If chars equal → `dp[i-1][j-1] + 1`, else `max(dp[i-1][j], dp[i][j-1])`.

### Java Solution (optimal — 1D rolling)
```java
class Solution {
    public int longestCommonSubsequence(String a, String b) {
        int m = a.length(), n = b.length();
        int[] dp = new int[n + 1];
        for (int i = 1; i <= m; i++) {
            int prev = 0;          // dp[i-1][j-1] for next iter
            for (int j = 1; j <= n; j++) {
                int temp = dp[j];
                if (a.charAt(i - 1) == b.charAt(j - 1)) dp[j] = prev + 1;
                else dp[j] = Math.max(dp[j], dp[j - 1]);
                prev = temp;
            }
        }
        return dp[n];
    }
}
```

### Edge Cases
- One string empty → 0.
- Identical strings → length.

### Follow-ups & Variants
- **Edit Distance** (§9.13) — close cousin.
- **Shortest Common Supersequence** (LC #1092) — LCS + remainder interleaved.
- **Longest Palindromic Subsequence** (LC #516) — LCS of `s` and `reverse(s)`.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 2D | O(mn) | O(mn) |
| 1D (optimal) | O(mn) | O(n) |

---

## 13. Edit Distance

**LeetCode #72 | Difficulty: Hard**

### Problem
Compute the minimum number of single-character edits (insert / delete / replace) to convert `word1` into `word2`.

### Pattern Flashcard
> **Trigger phrase:** "Levenshtein / minimum edits to transform"
> **Core idea in one line:** `dp[i][j]` = edits between `w1[0..i]` and `w2[0..j]`. Match → carry `dp[i-1][j-1]`. Otherwise: 1 + min(insert `dp[i][j-1]`, delete `dp[i-1][j]`, replace `dp[i-1][j-1]`).

### Java Solution (optimal — 2D, easy to read)
```java
class Solution {
    public int minDistance(String a, String b) {
        int m = a.length(), n = b.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (a.charAt(i - 1) == b.charAt(j - 1)) dp[i][j] = dp[i - 1][j - 1];
                else dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
            }
        }
        return dp[m][n];
    }
}
```

### Dry Run / Key Insight
Base cases: converting from "" to `b[0..j]` = j inserts; `a[0..i]` to "" = i deletes.

### Edge Cases
- Either string empty → length of the other.
- Identical strings → 0.

### Follow-ups & Variants
- **One Edit Distance** (LC #161) — boolean check; O(n) one-pass.
- **Delete Operation for Two Strings** (LC #583) — only deletes; LCS-based.
- **Minimum ASCII Delete Sum** (LC #712).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| 2D (optimal) | O(mn) | O(mn) |
| 1D rolling | O(mn) | O(n) |

---

## 14. Matrix Chain Multiplication

**Classic | Difficulty: Hard**

### Problem
Given an array `p` where matrix `i` has dimensions `p[i-1] × p[i]`, find the minimum number of scalar multiplications to compute the full product.

### Pattern Flashcard
> **Trigger phrase:** "Optimally split / parenthesize a sequence"
> **Recognize when:**
> - Interval DP: `dp[i][j]` = optimal for sub-range `i..j`.
>
> **Core idea in one line:** `dp[i][j] = min over k: dp[i][k] + dp[k+1][j] + p[i-1]*p[k]*p[j]`. Iterate by length, then by start.

### Java Solution (optimal)
```java
class MCM {
    public int matrixChainOrder(int[] p) {
        int n = p.length - 1;
        int[][] dp = new int[n + 1][n + 1];
        for (int len = 2; len <= n; len++) {
            for (int i = 1; i + len - 1 <= n; i++) {
                int j = i + len - 1;
                dp[i][j] = Integer.MAX_VALUE;
                for (int k = i; k < j; k++) {
                    int cost = dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j];
                    if (cost < dp[i][j]) dp[i][j] = cost;
                }
            }
        }
        return dp[1][n];
    }
}
```

### Dry Run / Key Insight
**Interval DP iteration order matters:** when computing `dp[i][j]`, all smaller intervals must already be solved → outer loop by length, inner loop by starting index.

### Edge Cases
- One matrix → 0 multiplications.
- All same-size square matrices → cost depends on dimension.

### Follow-ups & Variants
- **Burst Balloons** (LC #312) — interval DP with reversed thinking (last to burst).
- **Stone Game variants** — interval DP for two-player games.
- **Optimal BST** (Knuth's optimization variant).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Interval DP (optimal) | O(n³) | O(n²) |

---

## 15. Coin Change II

**LeetCode #518 | Difficulty: Medium**

### Problem
Return the number of **combinations** of coins that sum to `amount`. Unlimited supply.

### Pattern Flashcard
> **Trigger phrase:** "How many ways to make change / count combinations"
> **Recognize when:**
> - Combinations, not permutations (order doesn't matter).
> - Unbounded use.
>
> **Core idea in one line:** Outer loop over coins, inner loop over amount ascending — this enforces a canonical coin ordering, avoiding double-counting permutations.

### Java Solution (optimal)
```java
class Solution {
    public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;
        for (int c : coins)
            for (int v = c; v <= amount; v++) dp[v] += dp[v - c];
        return dp[amount];
    }
}
```

### Dry Run / Key Insight
**Direction matters:**
- Coins outer → counts **combinations** (`{1,2}` and `{2,1}` are the same).
- Amount outer → counts **permutations** (`{1,2}` and `{2,1}` distinct) → that's Combination Sum IV (LC #377).

### Edge Cases
- amount = 0 → 1 (empty combination).
- No coins → 0 unless amount = 0.

### Follow-ups & Variants
- **Combination Sum IV** (LC #377) — order matters, just swap loops.
- **Knight Dialer** (LC #935) — sequence counts on a graph.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DP (optimal) | O(amount · k) | O(amount) |

---

## 16. Target Sum

**LeetCode #494 | Difficulty: Medium**

### Problem
Assign `+` or `−` to each number in `nums`. Return the number of ways to make the sum equal `target`.

### Pattern Flashcard
> **Trigger phrase:** "Assign +/− signs to reach target"
> **Recognize when:**
> - Each element gets a binary sign decision.
> - Reduces to subset-sum: pick positives `P`; negatives `N` = total − P; target = P − N = 2P − total → `P = (target + total) / 2`.
>
> **Core idea in one line:** Count subsets summing to `(total + target) / 2` using subset-sum DP.

### Java Solution (optimal)
```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int total = 0;
        for (int x : nums) total += x;
        if (Math.abs(target) > total || (total + target) % 2 != 0) return 0;
        int sub = (total + target) / 2;
        int[] dp = new int[sub + 1];
        dp[0] = 1;
        for (int x : nums)
            for (int s = sub; s >= x; s--) dp[s] += dp[s - x];
        return dp[sub];
    }
}
```

### Dry Run / Key Insight
The math conversion is the trick. Once you reduce to subset-count, this is the same template as §9.10 / §9.15.

### Edge Cases
- `target` greater than total → 0.
- `(total + target)` odd → 0 (no integer subset sum).
- All zeros → `2^n` ways for target = 0.

### Follow-ups & Variants
- **Partition Equal Subset Sum** (§9.10).
- **Last Stone Weight II** (LC #1049) — subset-sum minimizing |2P − total|.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DP (optimal) | O(n · sub) | O(sub) |
