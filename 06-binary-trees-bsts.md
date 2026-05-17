# 6. Binary Trees & BSTs

> Tree problems split cleanly into two recursion styles: **top-down** (carry state into children, e.g. depth, valid range) and **bottom-up** (children compute summaries, parent combines them). When in doubt, sketch the recursive return type first — it usually reveals the algorithm. BFS via a queue handles every "level-by-level" problem.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Invert Binary Tree | Recursive swap of children | Easy |
| 2 | Maximum Depth / Diameter | Bottom-up height returning value | Easy/Medium |
| 3 | Level Order Traversal | BFS with queue | Medium |
| 4 | Lowest Common Ancestor | Bottom-up "matched count" | Medium |
| 5 | Validate BST | Top-down range bounds | Medium |
| 6 | Build Tree from Preorder + Inorder | Split inorder around preorder root | Medium |
| 7 | Binary Tree Max Path Sum | Bottom-up "gain" with global max | Hard |
| 8 | Serialize / Deserialize Tree | Preorder with null sentinels | Hard |
| 9 | Kth Smallest in BST | In-order traversal with counter | Medium |
| 10 | Right Side View | BFS taking last node per level | Medium |
| 11 | Path Sum III | Prefix sum on root-to-node paths | Medium |
| 12 | Flatten BT to Linked List | Reverse-preorder pointer rewiring | Medium |

```java
class TreeNode { int val; TreeNode left, right; TreeNode(int v){val=v;} }
```

---

## 1. Invert Binary Tree

**LeetCode #226 | Difficulty: Easy**

### Problem
Mirror the binary tree: swap left and right children at every node.

### Pattern Flashcard
> **Trigger phrase:** "Mirror / flip / reflect the tree"
> **Core idea in one line:** Recurse, swap children at each node.

### Approaches & Trade-offs

#### Approach 1 — Recursion
- O(n) / O(h) stack.

#### Approach 2 — Iterative BFS / DFS with Stack or Queue
- O(n) / O(w) for breadth or O(h) for depth.

### Java Solution (optimal — recursive)
```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        TreeNode tmp = invertTree(root.left);
        root.left = invertTree(root.right);
        root.right = tmp;
        return root;
    }
}
```

### Edge Cases
- Empty tree → null.
- Single node → unchanged.

### Follow-ups & Variants
- **Symmetric Tree** (LC #101) — check if tree is its own mirror.
- **Mirror without recursion** for very deep trees → BFS queue.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Recursion | O(n) | O(h) |
| BFS | O(n) | O(w) |

---

## 2. Maximum Depth / Diameter of Binary Tree

**LeetCode #104, #543 | Difficulty: Easy / Easy**

### Problem (Max Depth)
Length of the longest root-to-leaf path (number of nodes).

### Problem (Diameter)
Length of the longest path between any two nodes (number of *edges*).

### Pattern Flashcard
> **Trigger phrase:** "Height / depth / diameter / longest path"
> **Core idea in one line:** Bottom-up: each node returns its height; for diameter, also update a global max with `leftH + rightH` at every node.

### Approaches & Trade-offs

#### Approach 1 — Brute Force for Diameter
- Compute height at every node. O(n²).

#### Approach 2 — Single DFS, height returned (optimal)
- O(n) / O(h).

### Java Solution (Max Depth)
```java
class Solution {
    public int maxDepth(TreeNode root) {
        return root == null ? 0 : 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }
}
```

### Java Solution (Diameter — optimal)
```java
class Solution {
    private int best = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        height(root);
        return best;
    }

    private int height(TreeNode node) {
        if (node == null) return 0;
        int l = height(node.left), r = height(node.right);
        best = Math.max(best, l + r);  // edges through this node
        return 1 + Math.max(l, r);
    }
}
```

### Dry Run / Key Insight
At each node, the longest path *passing through it* equals leftHeight + rightHeight (in edges). The answer is the max over all nodes.

### Edge Cases
- Empty tree → depth 0, diameter 0.
- Single node → depth 1, diameter 0.
- Skewed tree → depth = n, diameter = n − 1.

### Follow-ups & Variants
- **Balanced Binary Tree** (LC #110) — same DFS, return −1 sentinel if unbalanced.
- **Longest Univalue Path** (LC #687).
- **Longest Path in N-ary Tree** — generalize to top-2 children.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Brute (diameter) | O(n²) | O(h) |
| Single DFS (optimal) | O(n) | O(h) |

---

## 3. Binary Tree Level Order Traversal

**LeetCode #102 | Difficulty: Medium**

### Problem
Return a list of lists, one per level, top-down.

### Pattern Flashcard
> **Trigger phrase:** "Level-by-level / BFS / per-row tree output"
> **Core idea in one line:** Queue; at each round, snapshot its size and drain exactly that many nodes into a level list.

### Approaches & Trade-offs

#### Approach 1 — BFS with size-stamped rounds (optimal-readable)
- O(n) / O(w).

#### Approach 2 — DFS with level-index appends
- O(n) / O(h). Less natural.

### Java Solution (optimal — BFS)
```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> out = new ArrayList<>();
        if (root == null) return out;
        Queue<TreeNode> q = new ArrayDeque<>();
        q.offer(root);
        while (!q.isEmpty()) {
            int size = q.size();
            List<Integer> level = new ArrayList<>(size);
            for (int i = 0; i < size; i++) {
                TreeNode n = q.poll();
                level.add(n.val);
                if (n.left  != null) q.offer(n.left);
                if (n.right != null) q.offer(n.right);
            }
            out.add(level);
        }
        return out;
    }
}
```

### Edge Cases
- Empty tree → empty list.
- Skewed tree → list of singleton lists.

### Follow-ups & Variants
- **Bottom-up** (LC #107) — reverse the result, or collect with prepend.
- **Zigzag** (LC #103) — toggle reverse per level.
- **Average of Levels** (LC #637), **Right Side View** (§6.10).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| BFS (optimal) | O(n) | O(w) |

---

## 4. Lowest Common Ancestor of a Binary Tree

**LeetCode #236 | Difficulty: Medium**

### Problem
Given two nodes `p` and `q` in a binary tree (both guaranteed present), return their LCA.

### Pattern Flashcard
> **Trigger phrase:** "First common ancestor / merge point of two nodes"
> **Recognize when:**
> - General binary tree (no BST property).
> - Both nodes guaranteed to exist.
>
> **Core idea in one line:** Post-order recursion. Return the node itself if `p` or `q` matches. If both subtrees return non-null, the current node is the LCA.

### Approaches & Trade-offs

#### Approach 1 — Parent Pointers + Path
- Build parent map, walk both up, find meeting node. O(n) but two passes.

#### Approach 2 — Recursive Post-order (optimal)
- One pass. O(n) / O(h).

### Java Solution (optimal)
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode left  = lowestCommonAncestor(root.left,  p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left != null && right != null) return root;
        return left != null ? left : right;
    }
}
```

### Dry Run / Key Insight
The function "returns p or q if found in this subtree, the LCA if both, else null." Once a subtree finds the LCA, that node bubbles up unchanged.

### Edge Cases
- p or q is the root → return root.
- One node is an ancestor of the other → return the ancestor.
- Both subtrees null → null.

### Follow-ups & Variants
- **LCA in BST** (LC #235) — simpler: descend left/right based on values.
- **LCA when nodes may be absent** — first verify presence, or track found-counts.
- **LCA of K nodes** — same template, generalize.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Parent map | O(n) | O(n) |
| Recursive post-order (optimal) | O(n) | O(h) |

---

## 5. Validate Binary Search Tree

**LeetCode #98 | Difficulty: Medium**

### Problem
Determine whether a binary tree is a valid BST: for every node, all left descendants are strictly less and all right descendants are strictly greater.

### Pattern Flashcard
> **Trigger phrase:** "Verify BST" / "is this BST valid?"
> **Recognize when:**
> - Common gotcha: checking only direct children is insufficient.
>
> **Core idea in one line:** Pass down `(lo, hi)` bounds — every node's value must lie strictly within; recurse with tightened bounds.

### Approaches & Trade-offs

#### Approach 1 — Compare to Direct Children Only
- **Wrong**. A right grandchild can be smaller than the root.

#### Approach 2 — In-order Traversal Must Be Strictly Increasing
- Compare consecutive values. O(n) / O(h).

#### Approach 3 — Top-Down Range Bounds (optimal-clear)
- O(n) / O(h).

### Java Solution (optimal — range bounds)
```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return check(root, null, null);
    }
    private boolean check(TreeNode node, Integer lo, Integer hi) {
        if (node == null) return true;
        if ((lo != null && node.val <= lo) || (hi != null && node.val >= hi)) return false;
        return check(node.left, lo, node.val) && check(node.right, node.val, hi);
    }
}
```

### Dry Run / Key Insight
Using `Integer` (boxed) and null sentinels avoids the `Integer.MIN_VALUE` corner where a real node holds the sentinel value.

### Edge Cases
- Empty tree → valid by convention.
- Duplicate values → invalid (strict inequality).
- Skewed tree → still O(h) stack.

### Follow-ups & Variants
- **Recover BST** (LC #99) — exactly two nodes swapped, find them via in-order.
- **Largest BST Subtree** (LC #333) — bottom-up triple `(min, max, size)`.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| In-order check | O(n) | O(h) |
| Range bounds (optimal) | O(n) | O(h) |

---

## 6. Construct Binary Tree from Preorder and Inorder Traversal

**LeetCode #105 | Difficulty: Medium**

### Problem
Given two arrays representing preorder and inorder traversals of a tree with **unique values**, reconstruct it.

### Pattern Flashcard
> **Trigger phrase:** "Rebuild tree from traversals"
> **Recognize when:**
> - You have two complementary traversals.
> - Preorder gives roots; inorder splits subtrees.
>
> **Core idea in one line:** First element of preorder is the root. Find it in inorder — everything to its left belongs to the left subtree, everything to the right belongs to the right.

### Approaches & Trade-offs

#### Approach 1 — Naïve Recursion with Slicing
- Each recursion copies arrays — O(n²) time / O(n²) space (Java slicing).

#### Approach 2 — Indexed Recursion + Hash Lookup (optimal)
- Map `inorderValue → inorderIndex`. O(n) time / O(n) space.

### Java Solution (optimal)
```java
class Solution {
    private int preIdx = 0;
    private Map<Integer, Integer> inIdx;
    private int[] preorder;

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        inIdx = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) inIdx.put(inorder[i], i);
        return build(0, inorder.length - 1);
    }

    private TreeNode build(int inL, int inR) {
        if (inL > inR) return null;
        int rootVal = preorder[preIdx++];
        TreeNode root = new TreeNode(rootVal);
        int mid = inIdx.get(rootVal);
        root.left  = build(inL, mid - 1);
        root.right = build(mid + 1, inR);
        return root;
    }
}
```

### Dry Run / Key Insight
**Important:** the order is `build(left)` then `build(right)` — that matches the way `preIdx` is consumed.

### Edge Cases
- Empty arrays → null.
- Single node → trivial.
- Duplicate values would break the inorder lookup; problem guarantees uniqueness.

### Follow-ups & Variants
- **From Inorder + Postorder** (LC #106) — consume postorder from the *end*, build right then left.
- **From Preorder + Postorder** (LC #889) — only valid for full binary trees, not unique in general.
- **Serialize/Deserialize** — see §6.8.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Slicing | O(n²) | O(n²) |
| Indexed + map (optimal) | O(n) | O(n) |

---

## 7. Binary Tree Maximum Path Sum

**LeetCode #124 | Difficulty: Hard**

### Problem
Path = sequence of nodes connected by edges (no node twice). Find the max sum of any path in the tree.

### Pattern Flashcard
> **Trigger phrase:** "Best path through tree without root constraint"
> **Recognize when:**
> - Path can start and end anywhere.
> - Path can bend at exactly one node (an "upside-down V" through it).
>
> **Core idea in one line:** At each node, "gain" returned to the parent = max(0, max(leftGain, rightGain)) + node.val. Separately update a global max with `leftGain + rightGain + node.val` (the V case).

### Approaches & Trade-offs

#### Approach 1 — Try Every Pair (LCA-based)
- O(n²).

#### Approach 2 — Single DFS with Global (optimal)
- O(n) / O(h).

### Java Solution (optimal)
```java
class Solution {
    private int best = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        gain(root);
        return best;
    }

    private int gain(TreeNode node) {
        if (node == null) return 0;
        int l = Math.max(0, gain(node.left));
        int r = Math.max(0, gain(node.right));
        best = Math.max(best, node.val + l + r);  // path that bends here
        return node.val + Math.max(l, r);          // path that continues up
    }
}
```

### Dry Run / Key Insight
Clamping `l` and `r` at 0 means "I can choose to skip a negative subtree." The returned value is the best *straight* path; the global captures the best *bending* path.

### Edge Cases
- Single node, possibly negative → that single value.
- All negative → max single node.

### Follow-ups & Variants
- **Diameter** (§6.2) — same template but in edge counts, no node values.
- **Longest path with constraints** — similar bottom-up state.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| LCA pairs | O(n²) | O(h) |
| Single DFS (optimal) | O(n) | O(h) |

---

## 8. Serialize and Deserialize Binary Tree

**LeetCode #297 | Difficulty: Hard**

### Problem
Encode a binary tree to a string and decode it back. Must roundtrip exactly.

### Pattern Flashcard
> **Trigger phrase:** "Persist / transmit / reconstruct a tree"
> **Core idea in one line:** Preorder with explicit `null` sentinels uniquely encodes any binary tree.

### Approaches & Trade-offs

#### Approach 1 — Level-order with `null` markers
- Works; needs queue-based parsing.

#### Approach 2 — Preorder + null markers (optimal, simplest)
- O(n) both ways.

### Java Solution (optimal)
```java
public class Codec {
    private static final String NULL = "#", SEP = ",";

    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        write(root, sb);
        return sb.toString();
    }
    private void write(TreeNode n, StringBuilder sb) {
        if (n == null) { sb.append(NULL).append(SEP); return; }
        sb.append(n.val).append(SEP);
        write(n.left, sb);
        write(n.right, sb);
    }

    public TreeNode deserialize(String data) {
        return read(new ArrayDeque<>(Arrays.asList(data.split(SEP))));
    }
    private TreeNode read(Deque<String> tokens) {
        String t = tokens.poll();
        if (NULL.equals(t)) return null;
        TreeNode n = new TreeNode(Integer.parseInt(t));
        n.left  = read(tokens);
        n.right = read(tokens);
        return n;
    }
}
```

### Dry Run / Key Insight
Preorder + null sentinels is unique because every position knows whether to recurse based on the sentinel.

### Edge Cases
- Empty tree → `"#,"`.
- Single node → `"5,#,#,"`.
- Very deep tree → recursion may stack-overflow; switch to iterative if needed.

### Follow-ups & Variants
- **Serialize/Deserialize BST** (LC #449) — can omit nulls (BST property reconstructs structure).
- **Serialize/Deserialize N-ary Tree** (LC #428).

### Complexity Summary
| Direction | Time | Space |
|---|---|---|
| Serialize | O(n) | O(n) |
| Deserialize | O(n) | O(n) |

---

## 9. Kth Smallest Element in a BST

**LeetCode #230 | Difficulty: Medium**

### Problem
Find the k-th smallest value in a BST.

### Pattern Flashcard
> **Trigger phrase:** "K-th smallest / largest in BST"
> **Recognize when:**
> - BST property is the whole point.
>
> **Core idea in one line:** In-order traversal yields sorted values; stop at the k-th.

### Approaches & Trade-offs

#### Approach 1 — Full In-order to List
- O(n) / O(n).

#### Approach 2 — Iterative In-order with Early Stop (optimal)
- O(h + k) / O(h).

#### Approach 3 — Augment Tree with Subtree Counts
- O(h) per query after O(n) preprocessing — best for repeated queries (follow-up).

### Java Solution (optimal — iterative in-order)
```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            while (cur != null) { stack.push(cur); cur = cur.left; }
            cur = stack.pop();
            if (--k == 0) return cur.val;
            cur = cur.right;
        }
        return -1;
    }
}
```

### Edge Cases
- k = 1 → leftmost node.
- k = n → rightmost node.
- k > n → undefined; problem usually guarantees valid k.

### Follow-ups & Variants
- **K-th Largest** → reverse in-order.
- **Frequent queries with inserts/deletes** → augment subtree sizes (LC #230 follow-up).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Full in-order | O(n) | O(n) |
| Iterative w/ early stop (optimal) | O(h + k) | O(h) |

---

## 10. Binary Tree Right Side View

**LeetCode #199 | Difficulty: Medium**

### Problem
Return values of nodes visible from the right side, top to bottom.

### Pattern Flashcard
> **Trigger phrase:** "Visible from one side / silhouette"
> **Core idea in one line:** BFS per level; the last node added to each level list is the rightmost.

### Approaches & Trade-offs

#### Approach 1 — BFS (optimal-clear)
- O(n) / O(w).

#### Approach 2 — DFS Right-First with Level Tracking
- O(n) / O(h). Take a node only if it's the first seen at its level.

### Java Solution (optimal — BFS)
```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> out = new ArrayList<>();
        if (root == null) return out;
        Queue<TreeNode> q = new ArrayDeque<>();
        q.offer(root);
        while (!q.isEmpty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                TreeNode n = q.poll();
                if (i == size - 1) out.add(n.val);
                if (n.left  != null) q.offer(n.left);
                if (n.right != null) q.offer(n.right);
            }
        }
        return out;
    }
}
```

### Edge Cases
- Empty tree → empty list.
- Skewed left tree → still produces a value per level.

### Follow-ups & Variants
- **Left Side View** → take first node per level.
- **Top View / Bottom View** — vertical-order traversal (LC #314).
- **Vertical Order Traversal** (LC #987).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| BFS (optimal) | O(n) | O(w) |
| DFS right-first | O(n) | O(h) |

---

## 11. Path Sum III

**LeetCode #437 | Difficulty: Medium**

### Problem
Count the number of paths in the tree (going downward, parent → child only) whose values sum to `targetSum`. Paths can start and end anywhere.

### Pattern Flashcard
> **Trigger phrase:** "Number of downward paths with sum K"
> **Recognize when:**
> - Path *starts and ends* anywhere on a root-to-leaf chain.
> - Reminds you of "Subarray sum equals K" (§1.9) — same trick applies on each root-to-node path.
>
> **Core idea in one line:** DFS carrying running prefix sum from root; at each node, look up how many earlier prefixes equal `current − target`.

### Approaches & Trade-offs

#### Approach 1 — Try Every Start Node
- O(n²).

#### Approach 2 — Prefix Sum + HashMap (optimal)
- O(n) / O(h).

### Java Solution (optimal)
```java
class Solution {
    public int pathSum(TreeNode root, int targetSum) {
        Map<Long, Integer> count = new HashMap<>();
        count.put(0L, 1);   // empty prefix
        return dfs(root, 0L, targetSum, count);
    }
    private int dfs(TreeNode node, long sum, int target, Map<Long, Integer> count) {
        if (node == null) return 0;
        sum += node.val;
        int paths = count.getOrDefault(sum - target, 0);
        count.merge(sum, 1, Integer::sum);
        paths += dfs(node.left, sum, target, count);
        paths += dfs(node.right, sum, target, count);
        count.merge(sum, -1, Integer::sum);  // backtrack
        return paths;
    }
}
```

### Dry Run / Key Insight
Same trick as Subarray Sum K: count[`s − target`] gives qualifying *root-to-here* subpaths ending at the current node. Backtrack on the way up.

### Edge Cases
- Negative node values → why `long` (avoid overflow on big sums).
- Empty tree → 0.
- targetSum = 0 with zero-valued path → counted by the `(0,1)` initial entry.

### Follow-ups & Variants
- **Path Sum II** (LC #113) — list all root-to-leaf paths summing to target.
- **Longest path with given property** — same prefix-sum logic, store first index.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| All starts | O(n²) | O(h) |
| Prefix-sum map (optimal) | O(n) | O(n) |

---

## 12. Flatten Binary Tree to Linked List

**LeetCode #114 | Difficulty: Medium**

### Problem
Flatten the tree in place to a "right-skewed linked list" using preorder (left null, right = next node).

### Pattern Flashcard
> **Trigger phrase:** "Flatten tree in place"
> **Core idea in one line:** Reverse-preorder traversal (right → left → node) and at each node set `node.right = prev; node.left = null`.

### Approaches & Trade-offs

#### Approach 1 — Preorder to List, Re-link
- O(n) / O(n).

#### Approach 2 — Reverse-Preorder DFS with `prev` pointer (optimal)
- O(n) / O(h).

#### Approach 3 — Morris-style In-place (no recursion)
- O(n) / O(1).

### Java Solution (optimal — reverse-preorder)
```java
class Solution {
    private TreeNode prev = null;

    public void flatten(TreeNode root) {
        if (root == null) return;
        flatten(root.right);
        flatten(root.left);
        root.right = prev;
        root.left = null;
        prev = root;
    }
}
```

### Java Solution (Morris-style, O(1) space)
```java
class Solution {
    public void flatten(TreeNode root) {
        TreeNode cur = root;
        while (cur != null) {
            if (cur.left != null) {
                TreeNode pred = cur.left;
                while (pred.right != null) pred = pred.right;
                pred.right = cur.right;
                cur.right = cur.left;
                cur.left = null;
            }
            cur = cur.right;
        }
    }
}
```

### Dry Run / Key Insight
**Reverse-preorder mental model:** `prev` is "the node that should come right after the current node in the flattened list." Visiting right first means `prev` is built tail-first.

### Edge Cases
- Empty tree → no-op.
- Already right-skewed → effectively a no-op.
- Single node → unchanged.

### Follow-ups & Variants
- **Convert BST to Sorted DLL** (LC #426).
- **Flatten N-ary Tree** (LC #430).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| List + relink | O(n) | O(n) |
| Reverse-preorder (optimal) | O(n) | O(h) |
| Morris-style | O(n) | O(1) |
