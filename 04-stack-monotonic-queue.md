# 4. Stack & Monotonic Queue

> Stacks shine in two scenarios: (1) **LIFO structural problems** — matching brackets, parsing, simulating call stacks. (2) **Monotonic stack / deque** — when you ask "next greater," "next smaller," or "max in window," the right structure preserves a monotone sequence and provides answers in amortized O(1).

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Valid Parentheses | Stack matches opening with expected closer | Easy |
| 2 | Min Stack | Auxiliary stack OR encoded value tracking min | Medium |
| 3 | Evaluate Reverse Polish Notation | Operand stack | Medium |
| 4 | Generate Parentheses | Backtracking with open/close counts | Medium |
| 5 | Daily Temperatures | Monotonic decreasing stack of indices | Medium |
| 6 | Car Fleet | Sort + stack of arrival times | Medium |
| 7 | Largest Rectangle in Histogram | Monotonic increasing stack, compute widths | Hard |
| 8 | Asteroid Collision | Stack with collision resolution loop | Medium |

### Monotonic Stack — Template

Use a stack that stores **indices** (so you can compute distances) and is **monotonically increasing or decreasing in the values they reference**.

```java
Deque<Integer> stack = new ArrayDeque<>();
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && shouldPop(arr[stack.peek()], arr[i])) {
        int popped = stack.pop();
        // popped just lost — answer for popped index involves i
    }
    stack.push(i);
}
```

---

## 1. Valid Parentheses

**LeetCode #20 | Difficulty: Easy**

### Problem
Given a string containing `()`, `[]`, `{}`, determine if every opening bracket has a properly nested and ordered matching closer.

### Pattern Flashcard
> **Trigger phrase:** "Balanced brackets / matching pairs / well-formed"
> **Recognize when:**
> - Strict LIFO matching required.
> - Innermost unmatched element is the one to be examined.
>
> **Core idea in one line:** Push openers; on a closer, peek the stack and verify it matches.

### Approaches & Trade-offs

#### Approach 1 — Repeated Find-and-Replace
- Repeatedly remove "()", "[]", "{}" until no change. O(n²).

#### Approach 2 — Stack (optimal)
- O(n) time, O(n) space.

### Java Solution (optimal)
```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') stack.push(c);
            else {
                if (stack.isEmpty()) return false;
                char top = stack.pop();
                if (c == ')' && top != '(') return false;
                if (c == ']' && top != '[') return false;
                if (c == '}' && top != '{') return false;
            }
        }
        return stack.isEmpty();
    }
}
```

### Dry Run / Key Insight
`"([{}])"` → push `(`, `[`, `{`. Pop on `}` (match). Pop on `]` (match). Pop on `)` (match). Stack empty → valid.

### Edge Cases
- Empty string → valid (no rule violated).
- Odd length → must be invalid (can short-circuit if you want).
- Closer with empty stack → invalid.

### Follow-ups & Variants
- **Minimum number of insertions to balance** (LC #921).
- **Longest valid parentheses** (LC #32) — DP / stack tracking indices.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Replace loop | O(n²) | O(n) |
| Stack (optimal) | O(n) | O(n) |

---

## 2. Min Stack

**LeetCode #155 | Difficulty: Medium**

### Problem
Design a stack supporting `push`, `pop`, `top`, and `getMin` — all in **O(1)**.

### Pattern Flashcard
> **Trigger phrase:** "O(1) min / max alongside stack ops"
> **Recognize when:**
> - You need extreme-value queries layered over a normal stack.
>
> **Core idea in one line:** Keep an auxiliary stack of "running minimums" — push the new min (or old min if smaller) every time; pop in lockstep.

### Approaches & Trade-offs

#### Approach 1 — Recompute on getMin
- O(n) per getMin.

#### Approach 2 — Auxiliary Min Stack (optimal)
- O(1) all operations · O(n) extra space.

#### Approach 3 — Encode Difference from Min (one stack)
- Store `value − currentMin`. Update `currentMin` on push if value < min; pop using stored diff. O(1)/O(n). Trickier.

### Java Solution (optimal — two stacks)
```java
class MinStack {
    private final Deque<Integer> stack = new ArrayDeque<>();
    private final Deque<Integer> mins  = new ArrayDeque<>();

    public void push(int val) {
        stack.push(val);
        mins.push(mins.isEmpty() ? val : Math.min(val, mins.peek()));
    }
    public void pop()        { stack.pop(); mins.pop(); }
    public int top()         { return stack.peek(); }
    public int getMin()      { return mins.peek(); }
}
```

### Dry Run / Key Insight
push 5 → mins:[5]. push 3 → mins:[3,5]. push 7 → mins:[3,3,5] (carry 3 forward). pop → mins:[3,5]. getMin → 3.

### Edge Cases
- Pop on empty → undefined behavior; problem typically guarantees valid ops.
- Duplicates of min: keep them all on the min stack so pops stay in sync.

### Follow-ups & Variants
- **Max Stack** (LC #716) — same with reversed comparisons; can also support `popMax` with two stacks or a TreeMap.
- **Min Queue** → two-stack queue + min tracking, or monotonic deque (§2.9).

### Complexity Summary
| Operation | Time |
|---|---|
| push / pop / top / getMin | O(1) |

---

## 3. Evaluate Reverse Polish Notation

**LeetCode #150 | Difficulty: Medium**

### Problem
Evaluate an arithmetic expression in postfix notation. Tokens are integers or one of `+ - * /`.

### Pattern Flashcard
> **Trigger phrase:** "Postfix / RPN" / "stack-machine expression"
> **Recognize when:**
> - Tokens already in operand-then-operator order.
> - Operators act on the most recent operands (LIFO).
>
> **Core idea in one line:** Walk tokens; push numbers; on operator, pop two, apply, push result. **Order matters**: `a = pop()` is the right operand, `b = pop()` the left.

### Approaches & Trade-offs

#### Approach 1 — Recursive Tree Build
- More code; same complexity.

#### Approach 2 — Stack Evaluation (optimal)
- O(n) · O(n).

### Java Solution (optimal)
```java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> st = new ArrayDeque<>();
        for (String t : tokens) {
            switch (t) {
                case "+": case "-": case "*": case "/":
                    int b = st.pop(), a = st.pop();
                    st.push(switch (t) {
                        case "+" -> a + b;
                        case "-" -> a - b;
                        case "*" -> a * b;
                        default  -> a / b;
                    });
                    break;
                default:
                    st.push(Integer.parseInt(t));
            }
        }
        return st.pop();
    }
}
```

### Dry Run / Key Insight
`["2","1","+","3","*"]` → push 2, 1. `+` → push 3. Push 3. `*` → push 9. Result 9.

### Edge Cases
- Division by zero → problem-dependent.
- Negative integers in tokens like `"-3"`.
- Single-number expression → just return that.
- Java integer truncation toward zero matches problem spec.

### Follow-ups & Variants
- **Infix to postfix** → Shunting-yard algorithm.
- **Evaluate infix with parentheses** (LC #224, #227).
- **Basic Calculator** family.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Stack (optimal) | O(n) | O(n) |

---

## 4. Generate Parentheses

**LeetCode #22 | Difficulty: Medium**

### Problem
Given `n`, generate all well-formed combinations of `n` pairs of parentheses.

### Pattern Flashcard
> **Trigger phrase:** "Generate all valid balanced bracket sequences"
> **Recognize when:**
> - You're enumerating strings under structural constraints.
> - At each step you have a small fixed set of choices.
>
> **Core idea in one line:** Backtracking — add `(` if `open < n`; add `)` if `close < open`. Stop when length = 2n.

### Approaches & Trade-offs

#### Approach 1 — Generate All `2^(2n)` and Filter
- O(2^(2n) · n) checking each.

#### Approach 2 — Constrained Backtracking (optimal)
- O(C_n · n) where C_n is the n-th Catalan number.

### Java Solution (optimal)
```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> out = new ArrayList<>();
        backtrack(out, new StringBuilder(), 0, 0, n);
        return out;
    }

    private void backtrack(List<String> out, StringBuilder cur, int open, int close, int n) {
        if (cur.length() == 2 * n) { out.add(cur.toString()); return; }
        if (open < n) {
            cur.append('(');
            backtrack(out, cur, open + 1, close, n);
            cur.deleteCharAt(cur.length() - 1);
        }
        if (close < open) {
            cur.append(')');
            backtrack(out, cur, open, close + 1, n);
            cur.deleteCharAt(cur.length() - 1);
        }
    }
}
```

### Dry Run / Key Insight
n=3 → 5 Catalan-many: `((()))`, `(()())`, `(())()`, `()(())`, `()()()`.
**Mental model:** the invariant `close ≤ open ≤ n` is exactly the "no unmatched closer yet" rule; everything that maintains the invariant is valid.

### Edge Cases
- n = 0 → `[""]` or `[]` depending on spec.
- Large n → output grows as Catalan number.

### Follow-ups & Variants
- **Remove Invalid Parentheses** (LC #301) — much harder; BFS on string mutations.
- **Different Ways to Add Parentheses** (LC #241) — DP on operator splits.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Filter all | O(2^(2n) · n) | O(2n) |
| Backtrack (optimal) | O(C_n · n) | O(n) |

---

## 5. Daily Temperatures

**LeetCode #739 | Difficulty: Medium**

### Problem
Given an array `temperatures`, return `answer[i]` = number of days you have to wait after day `i` until a warmer temperature, or 0 if none.

### Pattern Flashcard
> **Trigger phrase:** "Next greater / next warmer / next larger element"
> **Recognize when:**
> - Output per index depends on the *next* element satisfying a comparison.
>
> **Core idea in one line:** Walk left-to-right with a **monotonic decreasing stack of indices**; when `arr[i]` exceeds `arr[stack.top()]`, the popped index's answer is `i − popped`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For each i, scan right. O(n²).

#### Approach 2 — Monotonic Stack (optimal)
- O(n) total — each index pushed/popped at most once.

### Java Solution (optimal)
```java
class Solution {
    public int[] dailyTemperatures(int[] t) {
        int n = t.length;
        int[] answer = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && t[i] > t[stack.peek()]) {
                int j = stack.pop();
                answer[j] = i - j;
            }
            stack.push(i);
        }
        return answer;
    }
}
```

### Dry Run / Key Insight
`[73,74,75,71,69,72,76,73]` → stack of "waiting-for-warmer" indices. When a hotter day arrives, all colder days in the stack get their answer.

### Edge Cases
- Strictly decreasing → all zeros.
- Strictly increasing → all ones.

### Follow-ups & Variants
- **Next Greater Element I/II** (LC #496, #503) — circular array variant.
- **Stock Span** — same idea but counts to the *left*.
- **Largest Rectangle in Histogram** — §4.7 (related stack pattern).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Mono stack (optimal) | O(n) | O(n) |

---

## 6. Car Fleet

**LeetCode #853 | Difficulty: Medium**

### Problem
Cars at given `position[i]` go at `speed[i]` toward a target. A slower car ahead is caught up to and forms a fleet. How many fleets reach the target?

### Pattern Flashcard
> **Trigger phrase:** "Catch-up / convoy / merge while going same direction"
> **Recognize when:**
> - Items move at different speeds; a faster one stuck behind a slower one merges.
>
> **Core idea in one line:** Sort cars by position **descending** (closest to target first). For each, compute arrival time `(target − pos)/speed`. Push if its arrival time is strictly greater than the current fleet's leader; otherwise it merges.

### Approaches & Trade-offs

#### Approach 1 — Simulate Movement
- Way too slow.

#### Approach 2 — Sort + Stack of Arrival Times (optimal)
- O(n log n) sort dominates.

### Java Solution (optimal)
```java
class Solution {
    public int carFleet(int target, int[] position, int[] speed) {
        int n = position.length;
        double[][] cars = new double[n][2];
        for (int i = 0; i < n; i++) {
            cars[i][0] = position[i];
            cars[i][1] = (double)(target - position[i]) / speed[i];
        }
        Arrays.sort(cars, (a, b) -> Double.compare(b[0], a[0]));  // by position desc

        int fleets = 0;
        double leadTime = 0;
        for (double[] c : cars) {
            if (c[1] > leadTime) {           // strictly slower than current leader
                fleets++;
                leadTime = c[1];
            }
            // else: merges into the leader's fleet (no change)
        }
        return fleets;
    }
}
```

### Dry Run / Key Insight
`target=12, pos=[10,8,0,5,3], speed=[2,4,1,1,3]` → arrival times `[1, 1, 12, 7, 3]`. Sorted by position desc: (10,1),(8,1),(5,7),(3,3),(0,12). Fleet leaders: 1, then 7, then 12 → 3 fleets.
**Mental model:** A car can't pass the one ahead. If you arrive *later than* the slower car ahead, you stay slower; if you'd arrive sooner, the car ahead is your bottleneck.

### Edge Cases
- All cars at same position → distinct speeds form distinct fleets? Per spec usually count as one fleet at the target; double-check problem.
- Single car → one fleet.

### Follow-ups & Variants
- **Car Fleet II** (LC #1776) — return times each car catches up; more complex.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Sort + scan (optimal) | O(n log n) | O(n) |

---

## 7. Largest Rectangle in Histogram

**LeetCode #84 | Difficulty: Hard**

### Problem
Given heights of bars (width 1), find the area of the largest rectangle that fits inside.

### Pattern Flashcard
> **Trigger phrase:** "Largest rectangle / maximum area histogram"
> **Recognize when:**
> - Each bar can be a rectangle's *height*; you want its widest valid extent.
> - Need: for each bar, the nearest shorter bar to the left and right.
>
> **Core idea in one line:** Monotonic increasing stack of indices. When `h[i] < h[top]`, pop: the popped bar's height defines a rectangle whose width = `i − newTop − 1`.

### Approaches & Trade-offs

#### Approach 1 — Brute Force
- For each bar, expand both ways. O(n²).

#### Approach 2 — Divide & Conquer
- Recurse on the lowest bar. O(n²) worst case, O(n log n) with segment tree.

#### Approach 3 — Monotonic Stack (optimal)
- O(n) · O(n).

### Java Solution (optimal)
```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length, best = 0;
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i <= n; i++) {
            int cur = (i == n) ? 0 : heights[i];
            while (!stack.isEmpty() && heights[stack.peek()] > cur) {
                int h = heights[stack.pop()];
                int left = stack.isEmpty() ? -1 : stack.peek();
                best = Math.max(best, h * (i - left - 1));
            }
            stack.push(i);
        }
        return best;
    }
}
```

### Dry Run / Key Insight
Sentinel `cur=0` at `i=n` forces the stack to drain. Each pop gives a rectangle of height `h[popped]` and width "from the bar just left of popped (i.e., new top after pop) to the current `i`, exclusive both ends."

### Edge Cases
- All equal heights → `n × h`.
- Strictly increasing → drains all at the end via the sentinel.
- Length 1 → just that bar.

### Follow-ups & Variants
- **Maximal Rectangle in Binary Matrix** (LC #85) — apply histogram per row.
- **Sum of Subarray Minimums** (LC #907) — contribution technique with monotonic stack.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute | O(n²) | O(1) |
| Mono stack (optimal) | O(n) | O(n) |

---

## 8. Asteroid Collision

**LeetCode #735 | Difficulty: Medium**

### Problem
Each asteroid is an integer: positive = moving right, negative = moving left. Same magnitude → both explode; otherwise the smaller explodes. Return the state after all collisions.

### Pattern Flashcard
> **Trigger phrase:** "Process items that may annihilate the previous"
> **Recognize when:**
> - One direction can be cleanly stacked; arrivals in the other direction trigger pop loops.
>
> **Core idea in one line:** Stack of surviving asteroids. When a left-moving asteroid arrives, repeatedly compare with the stack top until either both annihilate, the top survives, or the incoming survives.

### Approaches & Trade-offs

#### Approach 1 — Simulation in Array
- Possible but messy.

#### Approach 2 — Stack (optimal)
- O(n) amortized · O(n).

### Java Solution (optimal)
```java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> st = new ArrayDeque<>();
        for (int a : asteroids) {
            boolean alive = true;
            while (alive && a < 0 && !st.isEmpty() && st.peek() > 0) {
                int top = st.peek();
                if (top < -a) { st.pop(); }
                else if (top == -a) { st.pop(); alive = false; }
                else { alive = false; }
            }
            if (alive) st.push(a);
        }
        int[] out = new int[st.size()];
        for (int i = st.size() - 1; i >= 0; i--) out[i] = st.pop();
        return out;
    }
}
```

### Dry Run / Key Insight
`[5,10,-5]` → push 5, push 10, -5 vs 10 → -5 dies. Result `[5,10]`.
`[10,2,-5]` → push 10, push 2, -5 vs 2 → 2 dies. -5 vs 10 → -5 dies. Result `[10]`.
**Mental model:** Only a right-moving asteroid followed by a left-moving asteroid is dangerous; everything else stacks peacefully.

### Edge Cases
- All same direction → no collisions.
- Many simultaneous collisions resolved left-to-right via the while loop.
- Empty input → empty output.

### Follow-ups & Variants
- **Validate stack sequences** (LC #946) — similar simulation.
- **Robot Collisions / Stronger Robot** (LC #2751).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Stack (optimal) | O(n) amortized | O(n) |
