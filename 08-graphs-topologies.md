# 8. Graphs & Topologies

> Graph problems split into a handful of templates: **BFS/DFS for connectivity**, **topological sort for dependencies**, **Dijkstra for shortest paths with weights**, and **DSU (union-find) for dynamic connectivity / cycle detection in undirected graphs**. Grid problems are graphs in disguise — neighbors are the 4 (sometimes 8) adjacent cells.

## Quick Pattern Index

| # | Problem | Core Pattern | Difficulty |
|---|---------|--------------|------------|
| 1 | Number of Islands | Grid DFS/BFS flood-fill | Medium |
| 2 | Clone Graph | DFS/BFS with old→new map | Medium |
| 3 | Rotting Oranges | Multi-source BFS, layer count | Medium |
| 4 | Course Schedule / II | Topological sort (Kahn / DFS) | Medium |
| 5 | Pacific Atlantic Water Flow | Reverse BFS from boundaries | Medium |
| 6 | Surrounded Regions | Mark border-connected, flip rest | Medium |
| 7 | Number of Connected Components | DSU or DFS/BFS | Medium |
| 8 | Graph Valid Tree | DSU with cycle detection | Medium |
| 9 | Word Ladder | BFS over implicit string graph | Hard |
| 10 | Network Delay Time | Dijkstra (PQ) | Medium |
| 11 | Operations to Make Network Connected | DSU + redundant edges | Medium |
| 12 | Kruskal's / Prim's MST | DSU sort-by-weight / PQ frontier | Medium |
| 13 | Accounts Merge | DSU on emails → owner | Medium |

### Direction Vector (4-neighborhood)

```java
int[][] DIRS = {{1,0},{-1,0},{0,1},{0,-1}};
```

### DSU (Union-Find) Template

```java
class DSU {
    int[] parent, rank;
    DSU(int n) {
        parent = new int[n]; rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) { return parent[x] == x ? x : (parent[x] = find(parent[x])); }
    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) { parent[ra] = rb; }
        else if (rank[rb] < rank[ra]) { parent[rb] = ra; }
        else { parent[rb] = ra; rank[ra]++; }
        return true;
    }
}
```

---

## 1. Number of Islands

**LeetCode #200 | Difficulty: Medium**

### Problem
Given a 2D grid of `'1'` (land) and `'0'` (water), count connected islands (4-directional).

### Pattern Flashcard
> **Trigger phrase:** "Count connected regions in a grid"
> **Core idea in one line:** Scan every cell; on an unvisited `'1'`, flood-fill (DFS or BFS) and increment count.

### Approaches & Trade-offs

#### Approach 1 — DFS Flood Fill (optimal-clear)
- O(mn) · O(mn) recursion worst case.

#### Approach 2 — BFS Flood Fill
- O(mn) · O(min(m,n)) queue.

#### Approach 3 — DSU
- Union each `'1'` with its right and down neighbors; final count = roots of `'1'` cells. O(mn α(mn)).

### Java Solution (optimal — DFS)
```java
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int r = 0; r < grid.length; r++)
            for (int c = 0; c < grid[0].length; c++)
                if (grid[r][c] == '1') { dfs(grid, r, c); count++; }
        return count;
    }

    private void dfs(char[][] g, int r, int c) {
        if (r < 0 || c < 0 || r >= g.length || c >= g[0].length || g[r][c] != '1') return;
        g[r][c] = '0';
        dfs(g, r + 1, c); dfs(g, r - 1, c); dfs(g, r, c + 1); dfs(g, r, c - 1);
    }
}
```

### Edge Cases
- Empty grid → 0.
- All water → 0.
- All land → 1.

### Follow-ups & Variants
- **Max Area of Island** (LC #695) — count and return max area.
- **Number of Distinct Islands** (LC #694) — record relative shape strings.
- **Surrounded Regions** (§8.6).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DFS/BFS (optimal) | O(mn) | O(mn) |
| DSU | O(mn α(mn)) | O(mn) |

---

## 2. Clone Graph

**LeetCode #133 | Difficulty: Medium**

### Problem
Given a reference to a node in a connected undirected graph, return a deep copy.

### Pattern Flashcard
> **Trigger phrase:** "Deep copy of arbitrary graph"
> **Core idea in one line:** Traverse (DFS or BFS), maintain `Map<Node, Node>` from original to clone, and copy neighbors lazily.

### Approaches & Trade-offs

#### Approach 1 — DFS (optimal-clear)
- O(V + E) · O(V).

#### Approach 2 — BFS
- Same complexity, iterative.

### Java Solution (optimal — DFS)
```java
class Solution {
    private final Map<Node, Node> map = new HashMap<>();

    public Node cloneGraph(Node node) {
        if (node == null) return null;
        if (map.containsKey(node)) return map.get(node);
        Node copy = new Node(node.val);
        map.put(node, copy);
        for (Node nb : node.neighbors) copy.neighbors.add(cloneGraph(nb));
        return copy;
    }
}
```

### Edge Cases
- Null input → null.
- Self-loop / cycle → handled by the map lookup short-circuit.

### Follow-ups & Variants
- **Copy List with Random Pointer** (§5.5) — same template, different shape.
- **Clone N-ary Tree** (LC #1490).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DFS (optimal) | O(V + E) | O(V) |

---

## 3. Rotting Oranges

**LeetCode #994 | Difficulty: Medium**

### Problem
Grid: `0` empty, `1` fresh orange, `2` rotten. Each minute, rotten oranges rot 4-adjacent fresh ones. Return minutes until none fresh remain (or −1 if impossible).

### Pattern Flashcard
> **Trigger phrase:** "Multi-source BFS / spreads simultaneously"
> **Core idea in one line:** Seed the BFS queue with **all** initially rotten cells; process round-by-round, counting minutes.

### Approaches & Trade-offs

#### Approach 1 — Single-source BFS per rotten orange
- Wrong — they spread simultaneously.

#### Approach 2 — Multi-source BFS (optimal)
- O(mn) · O(mn).

### Java Solution (optimal)
```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int m = grid.length, n = grid[0].length, fresh = 0;
        Queue<int[]> q = new ArrayDeque<>();
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 2) q.offer(new int[]{r, c});
                else if (grid[r][c] == 1) fresh++;
            }

        int minutes = 0;
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        while (!q.isEmpty() && fresh > 0) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                int[] p = q.poll();
                for (int[] d : dirs) {
                    int nr = p[0] + d[0], nc = p[1] + d[1];
                    if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
                    if (grid[nr][nc] != 1) continue;
                    grid[nr][nc] = 2;
                    fresh--;
                    q.offer(new int[]{nr, nc});
                }
            }
            minutes++;
        }
        return fresh == 0 ? minutes : -1;
    }
}
```

### Edge Cases
- No fresh oranges → 0 minutes.
- Isolated fresh oranges → -1.

### Follow-ups & Variants
- **Walls and Gates** (LC #286) — multi-source BFS computing nearest-gate distances.
- **01 Matrix** (LC #542) — multi-source BFS from all zeros.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Multi-source BFS (optimal) | O(mn) | O(mn) |

---

## 4. Course Schedule / Course Schedule II

**LeetCode #207 / #210 | Difficulty: Medium / Medium**

### Problem (#207)
Given prerequisites as `[a, b]` meaning "to take a, you must take b first," can you finish all courses?

### Problem (#210)
Same setup but return a valid course order, or empty if impossible.

### Pattern Flashcard
> **Trigger phrase:** "Build order / dependency / can finish all tasks"
> **Recognize when:**
> - Directed graph; question is "does it have a topological order?"
>
> **Core idea in one line:** Topological sort. Either (a) BFS Kahn's algorithm using in-degree counts, or (b) DFS with 3-color cycle detection.

### Approaches & Trade-offs

#### Approach 1 — Kahn's BFS (optimal-clear)
- O(V + E) · O(V + E).

#### Approach 2 — DFS with Colors (white/gray/black)
- Same complexity; produces order in reverse on finishing stack.

### Java Solution (optimal — Kahn's, returns order)
```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prereqs) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        int[] indeg = new int[numCourses];
        for (int[] p : prereqs) {
            adj.get(p[1]).add(p[0]);
            indeg[p[0]]++;
        }
        Queue<Integer> q = new ArrayDeque<>();
        for (int i = 0; i < numCourses; i++) if (indeg[i] == 0) q.offer(i);

        int[] order = new int[numCourses];
        int idx = 0;
        while (!q.isEmpty()) {
            int u = q.poll();
            order[idx++] = u;
            for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
        }
        return idx == numCourses ? order : new int[0];
    }
}
```

### Dry Run / Key Insight
Kahn's invariant: a node enters the queue only when all its prerequisites have been emitted. If processing ends with fewer than V emitted → cycle exists.

### Edge Cases
- No prerequisites → any order works (e.g., 0..n−1).
- Cycle of length 1 (self-prereq) → impossible.

### Follow-ups & Variants
- **Alien Dictionary** (LC #269) — build graph from string ordering, then topo sort.
- **Parallel Courses** (LC #1136) — count minimum semesters via BFS layers.
- **Course Schedule III** (LC #630) — completely different (greedy + heap).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Kahn's BFS / DFS (optimal) | O(V + E) | O(V + E) |

---

## 5. Pacific Atlantic Water Flow

**LeetCode #417 | Difficulty: Medium**

### Problem
Given a 2D heightmap, find all cells from which water can flow to both the Pacific (top/left edges) and Atlantic (bottom/right edges) oceans, flowing only to neighbors of equal or lower height.

### Pattern Flashcard
> **Trigger phrase:** "Reaches both boundaries / flows out both edges"
> **Recognize when:**
> - Forward flow from every cell is expensive (O((mn)²)).
> - Reverse the question: which cells can the *ocean* reach by flowing uphill?
>
> **Core idea in one line:** BFS/DFS inward from each ocean's border, allowing moves to ≥ current height. The intersection of reachable sets is the answer.

### Approaches & Trade-offs

#### Approach 1 — From each cell, DFS to ocean
- O((mn)²).

#### Approach 2 — Reverse BFS/DFS from oceans (optimal)
- O(mn).

### Java Solution (optimal)
```java
class Solution {
    public List<List<Integer>> pacificAtlantic(int[][] h) {
        int m = h.length, n = h[0].length;
        boolean[][] pac = new boolean[m][n], atl = new boolean[m][n];
        for (int r = 0; r < m; r++) { dfs(h, r, 0, pac); dfs(h, r, n - 1, atl); }
        for (int c = 0; c < n; c++) { dfs(h, 0, c, pac); dfs(h, m - 1, c, atl); }
        List<List<Integer>> out = new ArrayList<>();
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                if (pac[r][c] && atl[r][c]) out.add(List.of(r, c));
        return out;
    }
    private void dfs(int[][] h, int r, int c, boolean[][] seen) {
        seen[r][c] = true;
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nc < 0 || nr >= h.length || nc >= h[0].length) continue;
            if (seen[nr][nc] || h[nr][nc] < h[r][c]) continue;
            dfs(h, nr, nc, seen);
        }
    }
}
```

### Edge Cases
- 1×n or m×1 grids — every cell touches both oceans.
- Empty grid → empty result.

### Follow-ups & Variants
- **Surrounded Regions** (§8.6) — same reverse-from-border trick.
- **Walls and Gates** (LC #286) — multi-source BFS.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Forward DFS | O((mn)²) | O(mn) |
| Reverse BFS/DFS (optimal) | O(mn) | O(mn) |

---

## 6. Surrounded Regions

**LeetCode #130 | Difficulty: Medium**

### Problem
Flip every `'O'` to `'X'` **unless** the `'O'` is connected (4-directionally) to a border `'O'`.

### Pattern Flashcard
> **Trigger phrase:** "Flip / capture inner cells, preserve border-connected"
> **Core idea in one line:** Mark all border-connected `'O'`s with a sentinel, then flip remaining `'O'`s to `'X'` and sentinels back to `'O'`.

### Java Solution (optimal — DFS)
```java
class Solution {
    public void solve(char[][] board) {
        int m = board.length, n = board[0].length;
        for (int r = 0; r < m; r++) { dfs(board, r, 0); dfs(board, r, n - 1); }
        for (int c = 0; c < n; c++) { dfs(board, 0, c); dfs(board, m - 1, c); }
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                board[r][c] = board[r][c] == 'S' ? 'O' : 'X';
    }
    private void dfs(char[][] b, int r, int c) {
        if (r < 0 || c < 0 || r >= b.length || c >= b[0].length || b[r][c] != 'O') return;
        b[r][c] = 'S';
        dfs(b, r+1, c); dfs(b, r-1, c); dfs(b, r, c+1); dfs(b, r, c-1);
    }
}
```

### Edge Cases
- Grid smaller than 3×3 → all O's are on a border; nothing flips.
- All O's → all stay.

### Follow-ups & Variants
- **Number of Enclaves** (LC #1020) — count cells *not* connected to border.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Border DFS (optimal) | O(mn) | O(mn) |

---

## 7. Number of Connected Components in an Undirected Graph

**LeetCode #323 | Difficulty: Medium**

### Problem
Given `n` and a list of undirected edges, count the connected components.

### Pattern Flashcard
> **Trigger phrase:** "How many components / clusters / groups"
> **Core idea in one line:** DSU: each successful union reduces components by 1. Initial count = n.

### Approaches & Trade-offs

#### Approach 1 — DFS / BFS
- O(V + E) · O(V).

#### Approach 2 — DSU (optimal-clear)
- O((V + E) α(V)).

### Java Solution (optimal — DSU)
```java
class Solution {
    public int countComponents(int n, int[][] edges) {
        DSU dsu = new DSU(n);
        int comps = n;
        for (int[] e : edges) if (dsu.union(e[0], e[1])) comps--;
        return comps;
    }
}
```
(Use the DSU template at the top.)

### Edge Cases
- No edges → n components.
- All nodes connected → 1.

### Follow-ups & Variants
- **Graph Valid Tree** (§8.8).
- **Number of Provinces** (LC #547) — adjacency matrix variant.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DFS/BFS | O(V + E) | O(V + E) |
| DSU (optimal) | O((V+E) α(V)) | O(V) |

---

## 8. Graph Valid Tree

**LeetCode #261 | Difficulty: Medium**

### Problem
Given `n` nodes and a list of edges, determine if it forms a valid tree (connected + acyclic).

### Pattern Flashcard
> **Trigger phrase:** "Is the graph a tree?"
> **Recognize when:**
> - Need both connectivity AND acyclicity.
>
> **Core idea in one line:** A tree on n nodes has exactly `n − 1` edges and is connected. With DSU, any duplicate-root union → cycle.

### Java Solution (optimal — DSU)
```java
class Solution {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n - 1) return false;
        DSU dsu = new DSU(n);
        for (int[] e : edges) if (!dsu.union(e[0], e[1])) return false;
        return true;
    }
}
```

### Edge Cases
- n = 1, no edges → valid tree.
- Disconnected even with `n − 1` edges → caught because some union returns false earlier? No — only the duplicate-root case triggers. The `edges.length == n − 1` check together with the all-unions-succeed condition implies connectivity.

### Follow-ups & Variants
- **Redundant Connection** (LC #684) — find the extra edge causing the cycle.
- **Redundant Connection II** (LC #685) — directed version.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DSU (optimal) | O(E α(V)) | O(V) |

---

## 9. Word Ladder

**LeetCode #127 | Difficulty: Hard**

### Problem
Transform `beginWord` to `endWord` by changing one letter at a time; every intermediate must be in `wordList`. Return shortest transformation length, or 0 if impossible.

### Pattern Flashcard
> **Trigger phrase:** "Shortest sequence of single-letter mutations"
> **Recognize when:**
> - Implicit graph: nodes = words, edges = one-letter swaps.
> - Shortest path in an unweighted graph → BFS.
>
> **Core idea in one line:** BFS from `beginWord`; neighbors = words differing by one letter (generate via pattern keys like `"h*t"` for O(L) per word).

### Approaches & Trade-offs

#### Approach 1 — Naïve BFS with O(W·L) neighbor scan
- O(W² · L). Too slow for large dictionaries.

#### Approach 2 — Pattern-Keyed Adjacency (optimal)
- Build map `pattern → list of matching words`. O(W · L²) preprocessing; BFS visits each word once.

#### Approach 3 — Bidirectional BFS
- Tighter constant factors; from both ends.

### Java Solution (optimal — pattern BFS)
```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Set<String> dict = new HashSet<>(wordList);
        if (!dict.contains(endWord)) return 0;

        Queue<String> q = new ArrayDeque<>();
        q.offer(beginWord);
        Set<String> visited = new HashSet<>();
        visited.add(beginWord);

        int steps = 1;
        while (!q.isEmpty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                String w = q.poll();
                if (w.equals(endWord)) return steps;
                char[] arr = w.toCharArray();
                for (int j = 0; j < arr.length; j++) {
                    char orig = arr[j];
                    for (char c = 'a'; c <= 'z'; c++) {
                        if (c == orig) continue;
                        arr[j] = c;
                        String next = new String(arr);
                        if (dict.contains(next) && visited.add(next)) q.offer(next);
                    }
                    arr[j] = orig;
                }
            }
            steps++;
        }
        return 0;
    }
}
```

### Edge Cases
- `endWord` not in dictionary → 0.
- `beginWord == endWord` (problem-specific; usually returns 0 or 1).

### Follow-ups & Variants
- **Word Ladder II** (LC #126) — return *all* shortest paths; BFS to build parent DAG, then DFS to enumerate.
- **Minimum Genetic Mutation** (LC #433) — same pattern, alphabet `ACGT`.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Naïve | O(W² · L) | O(W·L) |
| Pattern BFS (optimal) | O(W · L · 26) | O(W·L) |

---

## 10. Dijkstra's Algorithm / Network Delay Time

**LeetCode #743 | Difficulty: Medium**

### Problem
Given a directed weighted graph and a source `k`, compute the time for the signal to reach all nodes (i.e., max shortest-path distance). Return −1 if unreachable.

### Pattern Flashcard
> **Trigger phrase:** "Shortest path with non-negative weights"
> **Recognize when:**
> - Edge weights are non-negative.
> - Need shortest paths from a single source.
>
> **Core idea in one line:** Min-priority queue of `(dist, node)`; pop the closest unvisited, relax outgoing edges. First pop of each node is its final shortest distance.

### Approaches & Trade-offs

#### Approach 1 — Bellman-Ford
- O(VE). Use when negative edges exist.

#### Approach 2 — Dijkstra with PQ (optimal for non-negative)
- O((V + E) log V).

#### Approach 3 — Floyd-Warshall
- O(V³). Useful for all-pairs.

### Java Solution (optimal — Dijkstra)
```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<List<int[]>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
        for (int[] t : times) adj.get(t[0]).add(new int[]{t[1], t[2]});

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, k});

        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int d = cur[0], u = cur[1];
            if (d > dist[u]) continue;                  // stale entry
            for (int[] nb : adj.get(u)) {
                int v = nb[0], w = nb[1];
                if (d + w < dist[v]) {
                    dist[v] = d + w;
                    pq.offer(new int[]{dist[v], v});
                }
            }
        }

        int max = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;
            max = Math.max(max, dist[i]);
        }
        return max;
    }
}
```

### Dry Run / Key Insight
The "stale entry" check is critical — we may push the same node multiple times with improving distances; skip the outdated pulls.

### Edge Cases
- Disconnected node → return -1.
- Self-loops with positive weight → harmless.
- **Negative edges → don't use Dijkstra.**

### Follow-ups & Variants
- **Cheapest Flights Within K Stops** (LC #787) — Bellman-Ford / modified Dijkstra with state.
- **Path with Minimum Effort** (LC #1631) — Dijkstra on a "min of max edge weight" metric.
- **Swim in Rising Water** (LC #778).

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Bellman-Ford | O(VE) | O(V) |
| Dijkstra PQ (optimal) | O((V + E) log V) | O(V + E) |
| Floyd-Warshall | O(V³) | O(V²) |

---

## 11. Number of Operations to Make Network Connected

**LeetCode #1319 | Difficulty: Medium**

### Problem
n computers and a list of `connections`. Each operation moves one cable between any two computers. Return minimum operations to connect all, or −1 if impossible.

### Pattern Flashcard
> **Trigger phrase:** "Minimum edges to connect everything / extras vs missing"
> **Recognize when:**
> - You're counting both **redundant edges** and **missing connections**.
>
> **Core idea in one line:** Need `n − 1` total edges. With DSU, redundant edges = unions that fail; components = roots. If `edges ≥ n − 1`, answer = `components − 1`.

### Java Solution (optimal)
```java
class Solution {
    public int makeConnected(int n, int[][] connections) {
        if (connections.length < n - 1) return -1;
        DSU dsu = new DSU(n);
        int comps = n;
        for (int[] c : connections) if (dsu.union(c[0], c[1])) comps--;
        return comps - 1;
    }
}
```

### Dry Run / Key Insight
The redundant-edge count doesn't actually appear in the formula — we just need *at least* `n − 1` edges total; any spare cables suffice to bridge components.

### Edge Cases
- Already fully connected → 0.
- `connections.length < n − 1` → impossible.

### Follow-ups & Variants
- **Min Cost to Connect All Points** (LC #1584) — MST (Prim or Kruskal).
- **Smallest String With Swaps** (LC #1202) — DSU of swap pairs.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DSU (optimal) | O(E α(V)) | O(V) |

---

## 12. Kruskal's / Prim's Algorithm (Minimum Spanning Tree)

**Classic | Difficulty: Medium**

### Problem
Given a weighted connected undirected graph, find a subset of edges that connects all vertices with minimum total weight.

### Pattern Flashcard
> **Trigger phrase:** "Minimum-cost network / connect all at minimum cost"
> **Recognize when:**
> - All vertices must end up connected.
> - Total edge weight is the optimization target.
>
> **Core idea in one line (Kruskal):** Sort edges by weight; greedily union with DSU, skipping cycle-forming edges.
> **Core idea in one line (Prim):** Min-PQ frontier; repeatedly pick the cheapest edge crossing into the unvisited set.

### Java Solution (Kruskal's — optimal for sparse graphs)
```java
class MST {
    public int kruskal(int n, int[][] edges) {
        Arrays.sort(edges, (a, b) -> a[2] - b[2]);
        DSU dsu = new DSU(n);
        int total = 0, used = 0;
        for (int[] e : edges) {
            if (dsu.union(e[0], e[1])) {
                total += e[2];
                if (++used == n - 1) return total;
            }
        }
        return -1;   // graph disconnected
    }
}
```

### Java Solution (Prim's — optimal for dense graphs)
```java
class MST {
    public int prim(int n, List<int[]>[] adj) {
        boolean[] inMST = new boolean[n];
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
        pq.offer(new int[]{0, 0});
        int total = 0, taken = 0;
        while (!pq.isEmpty() && taken < n) {
            int[] cur = pq.poll();
            int u = cur[0], w = cur[1];
            if (inMST[u]) continue;
            inMST[u] = true;
            total += w;
            taken++;
            for (int[] nb : adj[u]) if (!inMST[nb[0]]) pq.offer(nb);
        }
        return taken == n ? total : -1;
    }
}
```

### Dry Run / Key Insight
**Kruskal** thinks "edges first." **Prim** thinks "grow a tree from a seed." Both are O(E log V); Kruskal is preferred when edges are easy to sort, Prim when the graph is dense or adjacency-friendly.

### Edge Cases
- Disconnected graph → no spanning tree possible.
- Single vertex → cost 0.

### Follow-ups & Variants
- **Min Cost to Connect All Points** (LC #1584) — coordinate input; build complete graph or use Prim with O(V²) lazy approach.
- **Optimize Water Distribution** (LC #1168) — virtual node trick.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| Kruskal | O(E log E) | O(V) |
| Prim (PQ) | O(E log V) | O(V + E) |

---

## 13. Accounts Merge

**LeetCode #721 | Difficulty: Medium**

### Problem
A list of accounts; each is `[name, email1, email2, ...]`. Merge accounts that share any email. Return merged accounts with emails sorted; first element is the name.

### Pattern Flashcard
> **Trigger phrase:** "Merge entities sharing any attribute"
> **Recognize when:**
> - Equivalence relation defined by overlap on a key.
>
> **Core idea in one line:** DSU keyed by email → account index. For each account, union all its emails with the first one; finally group emails by root.

### Java Solution (optimal — DSU)
```java
class Solution {
    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        int n = accounts.size();
        DSU dsu = new DSU(n);
        Map<String, Integer> ownerOf = new HashMap<>();   // email -> account idx (root candidate)

        for (int i = 0; i < n; i++) {
            for (int j = 1; j < accounts.get(i).size(); j++) {
                String email = accounts.get(i).get(j);
                if (ownerOf.containsKey(email)) dsu.union(i, ownerOf.get(email));
                else ownerOf.put(email, i);
            }
        }

        Map<Integer, TreeSet<String>> byRoot = new HashMap<>();
        for (var entry : ownerOf.entrySet()) {
            int root = dsu.find(entry.getValue());
            byRoot.computeIfAbsent(root, k -> new TreeSet<>()).add(entry.getKey());
        }

        List<List<String>> out = new ArrayList<>();
        for (var entry : byRoot.entrySet()) {
            List<String> merged = new ArrayList<>();
            merged.add(accounts.get(entry.getKey()).get(0));
            merged.addAll(entry.getValue());
            out.add(merged);
        }
        return out;
    }
}
```

### Dry Run / Key Insight
Two accounts share an email → their indices end up in the same DSU component. Final grouping reads roots.

### Edge Cases
- Single-email accounts → still get their own component.
- Two accounts with same name but no shared emails → stay separate.

### Follow-ups & Variants
- **Most Stones Removed with Same Row or Column** (LC #947) — DSU keyed by row/col coordinate.
- **Number of Provinces** (LC #547).
- **Synonymous Sentences** (LC #1258) — DSU on synonym pairs.

### Complexity Summary
| Approach | Time | Space |
|---|---|---|
| DSU (optimal) | O(NK log NK) (sort emails) | O(NK) |

N = accounts, K = average emails/account.
