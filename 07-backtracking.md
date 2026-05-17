# 7. Backtracking

> Backtracking is depth-first exploration of a decision tree with **explicit undo** ("make a choice → recurse → undo it"). The skeleton is identical across problems; only the *choice set* and *validity check* change. Once the template is muscle memory, "generate all X under constraint Y" is a fill-in-the-blank exercise.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Subsets / Subsets II | Include/exclude at each index | Medium |
| 2 | Combination Sum / II | Unbounded vs bounded with dedup | Medium |
| 3 | Permutations | Swap-in-place or used[] mask | Medium |
| 4 | Word Search | DFS on grid with visited flag | Medium |
| 5 | Palindrome Partitioning | Try every prefix, recurse on suffix | Medium |
| 6 | N-Queens | Column + diagonal sets | Hard |
| 7 | Sudoku Solver | 27-constraint backtrack with bitmasks | Hard |

### Backtracking Template

```java
void backtrack(state, choices) {
    if (isGoal(state)) { record(state); return; }
    for (choice : choices(state)) {
        if (!isValid(choice, state)) continue;
        apply(choice, state);
        backtrack(state, choices);
        undo(choice, state);
    }
}
```

Three knobs:
1. **State representation** — partial list, indices, bitmask, board.
2. **Choice generation** — index range, candidate letters, valid moves.
3. **Pruning** — bound checks, dedup of equal choices at same level.

---

## 1. Subsets / Subsets II

**LeetCode #78 / #90 | Difficulty: Medium / Medium**

### Problem
Given a set (with optional duplicates), return all possible subsets. The result must not contain duplicate subsets.

### Pattern Flashcard
> **Trigger phrase:** "All subsets / power set"
> **Recognize when:**
> - You need every subset, not just one.
> - Duplicates in input → must dedup at each recursion level.
>
> **Core idea in one line:** Sort (for duplicates). At index `i`, decide for each candidate `j ≥ i`: include `nums[j]` and recurse with `i = j + 1`. Skip `j` if `j > i && nums[j] == nums[j-1]`.

### Approaches & Trade-offs

#### Approach 1 — Iterative Doubling
- For each element, append it to every existing subset. O(n · 2ⁿ).

#### Approach 2 — Backtracking (canonical, optimal)
- O(n · 2ⁿ).

#### Approach 3 — Bitmask Enumeration (no duplicates only)
- For i in `0..2ⁿ−1`, take elements where bit set. Clean but doesn't dedup naturally.

### Java Solution (Subsets II — handles duplicates)
```java
class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> out = new ArrayList<>();
        backtrack(0, nums, new ArrayList<>(), out);
        return out;
    }

    private void backtrack(int start, int[] nums, List<Integer> cur, List<List<Integer>> out) {
        out.add(new ArrayList<>(cur));
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i - 1]) continue;  // dedup at this level
            cur.add(nums[i]);
            backtrack(i + 1, nums, cur, out);
            cur.remove(cur.size() - 1);
        }
    }
}
```

### Dry Run / Key Insight
`[1,2,2]` → `[]`, `[1]`, `[1,2]`, `[1,2,2]`, `[2]`, `[2,2]`. The `i > start && nums[i] == nums[i-1]` skip prevents `[1, 2_b]` after we've already produced `[1, 2_a]`.

### Edge Cases
- Empty input → `[[]]`.
- All duplicates `[2,2,2]` → `[], [2], [2,2], [2,2,2]`.

### Follow-ups & Variants
- **Letter Combinations of a Phone Number** (LC #17) — fixed choices per index.
- **Combinations** (LC #77) — choose k of n.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Backtracking (optimal) | O(n · 2ⁿ) | O(n) recursion + output |

---

## 2. Combination Sum / Combination Sum II

**LeetCode #39 / #40 | Difficulty: Medium / Medium**

### Problem
Given candidates and a target, find all unique combinations summing to target.
- **#39:** unlimited uses per candidate; candidates are distinct.
- **#40:** each candidate used at most once; duplicates in input; combos must be unique.

### Pattern Flashcard
> **Trigger phrase:** "All combinations summing to target"
> **Recognize when:**
> - Find *all* solutions, not just count or one.
> - Order doesn't matter (else it'd be permutation flavor).
>
> **Core idea in one line:** Sort. Recurse with `start` index. For unbounded (#39): recurse with same `i`. For bounded (#40): recurse with `i + 1` and skip duplicates at same level.

### Java Solution (#39 — unbounded)
```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> out = new ArrayList<>();
        backtrack(0, target, candidates, new ArrayList<>(), out);
        return out;
    }
    private void backtrack(int start, int rem, int[] c, List<Integer> cur, List<List<Integer>> out) {
        if (rem == 0) { out.add(new ArrayList<>(cur)); return; }
        for (int i = start; i < c.length && c[i] <= rem; i++) {
            cur.add(c[i]);
            backtrack(i, rem - c[i], c, cur, out);   // i (reuse allowed)
            cur.remove(cur.size() - 1);
        }
    }
}
```

### Java Solution (#40 — bounded with dups)
```java
class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> out = new ArrayList<>();
        backtrack(0, target, candidates, new ArrayList<>(), out);
        return out;
    }
    private void backtrack(int start, int rem, int[] c, List<Integer> cur, List<List<Integer>> out) {
        if (rem == 0) { out.add(new ArrayList<>(cur)); return; }
        for (int i = start; i < c.length && c[i] <= rem; i++) {
            if (i > start && c[i] == c[i - 1]) continue;
            cur.add(c[i]);
            backtrack(i + 1, rem - c[i], c, cur, out);  // i + 1 (single use)
            cur.remove(cur.size() - 1);
        }
    }
}
```

### Dry Run / Key Insight
The pruning `c[i] <= rem` (because sorted) cuts off branches that can't possibly hit zero.

### Edge Cases
- target = 0 → one empty combo (or zero, depending on spec).
- Candidates greater than target → none contribute (pruned by `<= rem`).
- Negative candidates would invalidate the pruning — different problem.

### Follow-ups & Variants
- **Combination Sum III** (LC #216) — fixed-size k, candidates 1..9.
- **Combination Sum IV** (LC #377) — order *matters*; switch to DP (it's actually counting, not enumerating).
- **Coin Change** (§9.5) — count or min coins; permutation-vs-combo direction matters.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Backtracking | O(2ᴺ) worst case | O(target) |

---

## 3. Permutations

**LeetCode #46 | Difficulty: Medium**

### Problem
Given a distinct-value array, return all permutations.

### Pattern Flashcard
> **Trigger phrase:** "All orderings / arrangements"
> **Core idea in one line:** Two clean styles — (a) `used[]` mask tracking placed elements, or (b) in-place swap to fix prefixes.

### Approaches & Trade-offs

#### Approach 1 — `used[]` Mask (optimal-clear)
- O(n · n!) · O(n).

#### Approach 2 — In-place Swap
- Slightly less extra space.
- For **Permutations II** (LC #47, with dups), the `used[]` style is easier to dedup.

### Java Solution (used-mask)
```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> out = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), out);
        return out;
    }
    private void backtrack(int[] nums, boolean[] used, List<Integer> cur, List<List<Integer>> out) {
        if (cur.size() == nums.length) { out.add(new ArrayList<>(cur)); return; }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            cur.add(nums[i]);
            backtrack(nums, used, cur, out);
            cur.remove(cur.size() - 1);
            used[i] = false;
        }
    }
}
```

### Java Solution (Permutations II — with dups)
```java
class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> out = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), out);
        return out;
    }
    private void backtrack(int[] nums, boolean[] used, List<Integer> cur, List<List<Integer>> out) {
        if (cur.size() == nums.length) { out.add(new ArrayList<>(cur)); return; }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;  // dedup
            used[i] = true;
            cur.add(nums[i]);
            backtrack(nums, used, cur, out);
            cur.remove(cur.size() - 1);
            used[i] = false;
        }
    }
}
```

### Dry Run / Key Insight
**Dedup invariant (II):** among equal values, force them to be used in the original order; otherwise we'd produce the same permutation more than once.

### Edge Cases
- Empty array → `[[]]`.
- One element → `[[x]]`.

### Follow-ups & Variants
- **Next Permutation** (§1.10) — one step instead of all.
- **K-th Permutation Sequence** (LC #60) — factorial number system.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Backtracking | O(n · n!) | O(n) |

---

## 4. Word Search

**LeetCode #79 | Difficulty: Medium**

### Problem
Given an `m × n` board of letters and a word, determine if the word can be constructed by moving to 4-adjacent cells (no cell reused per traversal).

### Pattern Flashcard
> **Trigger phrase:** "Search a word in a grid / DFS path constraint"
> **Recognize when:**
> - Grid traversal that disallows revisiting cells.
> - Path-shape matters; visited must be undone on backtrack.
>
> **Core idea in one line:** Start DFS from each cell matching `word[0]`. Temporarily mark visited (e.g., overwrite with `#`); restore on return.

### Approaches & Trade-offs

#### Approach 1 — DFS with `visited[][]` Boolean (optimal-clear)
- O(m·n·4^L) · O(L) recursion (L = word length).

#### Approach 2 — In-place Mark (saves space)
- Replace cell with sentinel during recursion; restore on return.

### Java Solution (optimal — in-place)
```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int m = board.length, n = board[0].length;
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                if (dfs(board, r, c, word, 0)) return true;
        return false;
    }

    private boolean dfs(char[][] b, int r, int c, String w, int i) {
        if (i == w.length()) return true;
        if (r < 0 || c < 0 || r >= b.length || c >= b[0].length) return false;
        if (b[r][c] != w.charAt(i)) return false;

        char saved = b[r][c];
        b[r][c] = '#';                                    // mark
        boolean found =
            dfs(b, r + 1, c, w, i + 1) ||
            dfs(b, r - 1, c, w, i + 1) ||
            dfs(b, r, c + 1, w, i + 1) ||
            dfs(b, r, c - 1, w, i + 1);
        b[r][c] = saved;                                  // unmark
        return found;
    }
}
```

### Dry Run / Key Insight
The temporary overwrite acts as a *visited* flag with no extra memory. Backtrack restores the board exactly.

### Edge Cases
- Empty word — typically not allowed; return true if it is.
- Word longer than total cells → false.

### Follow-ups & Variants
- **Word Search II** (LC #212) — many words simultaneously; switch to Trie (§10.2).
- **Number of Distinct Paths in Grid** — DP if no obstacles, else DFS.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DFS (optimal) | O(m·n·4^L) | O(L) |

---

## 5. Palindrome Partitioning

**LeetCode #131 | Difficulty: Medium**

### Problem
Partition string `s` into substrings such that every part is a palindrome. Return all possible partitions.

### Pattern Flashcard
> **Trigger phrase:** "Partition into / split into palindromes"
> **Core idea in one line:** Backtrack with a `start` pointer; for each end ≥ start, if `s[start..end]` is a palindrome, recurse with `start = end + 1`.

### Approaches & Trade-offs

#### Approach 1 — Backtracking + Manual Palindrome Check
- O(n · 2ⁿ).

#### Approach 2 — Precompute `isPal[i][j]` DP (optimal)
- O(2ⁿ · n) with O(1) palindrome lookups.

### Java Solution (optimal — DP precompute)
```java
class Solution {
    public List<List<String>> partition(String s) {
        int n = s.length();
        boolean[][] pal = new boolean[n][n];
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                pal[i][j] = s.charAt(i) == s.charAt(j) && (j - i < 2 || pal[i + 1][j - 1]);
            }
        }
        List<List<String>> out = new ArrayList<>();
        backtrack(0, s, pal, new ArrayList<>(), out);
        return out;
    }

    private void backtrack(int start, String s, boolean[][] pal,
                           List<String> cur, List<List<String>> out) {
        if (start == s.length()) { out.add(new ArrayList<>(cur)); return; }
        for (int end = start; end < s.length(); end++) {
            if (!pal[start][end]) continue;
            cur.add(s.substring(start, end + 1));
            backtrack(end + 1, s, pal, cur, out);
            cur.remove(cur.size() - 1);
        }
    }
}
```

### Dry Run / Key Insight
DP table `pal[i][j]` = `s[i..j]` palindromic; bottom-up by length. Lookup turns each in-recursion check from O(n) to O(1).

### Edge Cases
- Single character → one partition `[[c]]`.
- All same chars → exponential output; no shortcut.

### Follow-ups & Variants
- **Palindrome Partitioning II** (LC #132) — *minimum* cuts; pure DP.
- **Palindrome Partitioning IV** (LC #1745) — exactly 3 palindromes; precompute + double loop.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Naïve | O(n · 2ⁿ) | O(n²) |
| DP precompute (optimal) | O(2ⁿ · n) | O(n²) |

---

## 6. N-Queens

**LeetCode #51 | Difficulty: Hard**

### Problem
Place `n` queens on an `n × n` board so no two attack each other. Return all distinct solutions.

### Pattern Flashcard
> **Trigger phrase:** "Place K non-attacking pieces / no-conflict layout"
> **Core idea in one line:** Place row-by-row; track which **columns**, **`r + c` diagonals**, and **`r − c` anti-diagonals** are occupied. Add a queen if all three sets free.

### Approaches & Trade-offs

#### Approach 1 — Check Every Pair on Each Placement
- O(n^n).

#### Approach 2 — Row-by-Row with Three Sets (optimal)
- O(n!).

#### Approach 3 — Bitmask State (very fast for n ≤ 30)
- O(n!) but tiny constants.

### Java Solution (optimal — three sets)
```java
class Solution {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> out = new ArrayList<>();
        backtrack(0, n, new int[n], new boolean[n], new boolean[2 * n], new boolean[2 * n], out);
        return out;
    }

    private void backtrack(int row, int n, int[] queens,
                           boolean[] cols, boolean[] diag, boolean[] anti,
                           List<List<String>> out) {
        if (row == n) { out.add(build(queens, n)); return; }
        for (int c = 0; c < n; c++) {
            int d = row + c, a = row - c + n;
            if (cols[c] || diag[d] || anti[a]) continue;
            queens[row] = c;
            cols[c] = diag[d] = anti[a] = true;
            backtrack(row + 1, n, queens, cols, diag, anti, out);
            cols[c] = diag[d] = anti[a] = false;
        }
    }

    private List<String> build(int[] queens, int n) {
        List<String> board = new ArrayList<>();
        for (int r = 0; r < n; r++) {
            char[] row = new char[n];
            Arrays.fill(row, '.');
            row[queens[r]] = 'Q';
            board.add(new String(row));
        }
        return board;
    }
}
```

### Dry Run / Key Insight
`row + col` is constant on a `↘` diagonal; `row − col` (offset by `+n` to avoid negatives) on a `↙` anti-diagonal. Three booleans per attempt → O(1) conflict check.

### Edge Cases
- n = 1 → `[["Q"]]`.
- n = 2 or 3 → no solution.
- For just *counting* solutions, use **N-Queens II** (LC #52) — same but increment a counter.

### Follow-ups & Variants
- **Sudoku Solver** (§7.7) — similar constraint structure.
- **K-Knights / K-Bishops** — different attack patterns; same template.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Naïve | O(n^n) | — |
| Three sets (optimal) | O(n!) | O(n) |

---

## 7. Sudoku Solver

**LeetCode #37 | Difficulty: Hard**

### Problem
Solve a partially filled 9×9 Sudoku, modifying the board in place.

### Pattern Flashcard
> **Trigger phrase:** "Solve constraint-satisfaction puzzle"
> **Core idea in one line:** Backtrack cell-by-cell; maintain 3 boolean arrays (rows, cols, boxes) so "is digit d legal here?" is O(1).

### Approaches & Trade-offs

#### Approach 1 — Naïve Backtracking with O(N) Conflict Checks
- Works but slower.

#### Approach 2 — Three-Track Membership Sets (optimal)
- O(1) legality test per try.

#### Approach 3 — Bitmask Membership + Minimum-Remaining-Values Heuristic
- Picks the most constrained empty cell first; significantly faster on hard boards.

### Java Solution (optimal — three boolean arrays)
```java
class Solution {
    private boolean[][] rows = new boolean[9][10];
    private boolean[][] cols = new boolean[9][10];
    private boolean[][] boxes = new boolean[9][10];

    public void solveSudoku(char[][] board) {
        for (int r = 0; r < 9; r++)
            for (int c = 0; c < 9; c++)
                if (board[r][c] != '.') {
                    int d = board[r][c] - '0';
                    rows[r][d] = cols[c][d] = boxes[box(r, c)][d] = true;
                }
        solve(board, 0, 0);
    }

    private boolean solve(char[][] b, int r, int c) {
        if (r == 9) return true;
        int nr = c == 8 ? r + 1 : r;
        int nc = c == 8 ? 0     : c + 1;
        if (b[r][c] != '.') return solve(b, nr, nc);
        for (int d = 1; d <= 9; d++) {
            int boxId = box(r, c);
            if (rows[r][d] || cols[c][d] || boxes[boxId][d]) continue;
            b[r][c] = (char)('0' + d);
            rows[r][d] = cols[c][d] = boxes[boxId][d] = true;
            if (solve(b, nr, nc)) return true;
            b[r][c] = '.';
            rows[r][d] = cols[c][d] = boxes[boxId][d] = false;
        }
        return false;
    }

    private int box(int r, int c) { return (r / 3) * 3 + c / 3; }
}
```

### Dry Run / Key Insight
Constraint membership arrays make legality O(1). Without them, you'd rescan the row, column, and box on every guess.

### Edge Cases
- Empty board → produces *some* valid sudoku (problem usually guarantees a unique solution given).
- Inconsistent input → returns false; board untouched only if no recursion changed anything.

### Follow-ups & Variants
- **Valid Sudoku** (§1.5) — just verification.
- **Word Search II** with Trie (§10.2) — similar "DFS with shared constraint structure."
- **General CSP solvers** (MRV + LCV + AC-3) — deeper rabbit hole.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| With membership sets (optimal) | O(9^empty) worst case | O(1) (fixed 9×9) |
