# Graph Valid Tree

---

1. What is meant by Graph Valid Tree?

## What is Graph Valid Tree?

**Graph Valid Tree** asks you to verify if a given set of nodes and edges forms a valid tree.

- A valid tree is a **connected, acyclic undirected graph**.
- Given `n` nodes (labeled `0` to `n-1`) and a list of undirected edges.

**Goal:** Return `true` if the graph is a valid tree, `false` otherwise.

**Constraints:**
- `1 <= n <= 2000`
- `0 <= edges.length <= 5000`
- `edges[i] = [a, b]` — undirected edge between a and b.

**Example:**

```
Input:  n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]
Output: true

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false  (cycle: 1-2-3-1)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- A valid tree must be:
  1. **Acyclic** — no cycles.
  2. **Connected** — all nodes reachable from any node.
- Equivalently: exactly **n-1 edges** AND connected.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 2000 nodes, 5000 edges — O(V+E) is fast       |
| **Latency**     | O(V + E) — linear                                    |
| **Correctness** | Must check both acyclicity AND full connectivity     |
| **Edge Cases**  | n=1 (single node, no edges → tree), duplicate edges |

### Clarifying Questions to Ask

- Is the graph directed or undirected? → Undirected.
- Can there be duplicate edges? → Assume no.
- Can there be self-loops? → No (by convention).
- Must all nodes be connected? → Yes, a tree is connected by definition.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: DFS Cycle Detection + Connectivity Check

| | Complexity |
| --- | --- |
| **Time** | O(V + E) — DFS traverses all edges |
| **Space** | O(V + E) — adjacency list + visited array + stack |

### Approach 2: Union-Find

| | Complexity |
| --- | --- |
| **Time** | O(E × α(V)) ≈ O(E) — nearly linear |
| **Space** | O(V) — parent and rank arrays |

### Quick Check Shortcut

A tree has **exactly `n-1` edges**. If the edge count differs, immediately return false:
- `edges.length != n-1` → not a tree (too many = cycle; too few = disconnected).

This check alone doesn't guarantee a tree (must also be connected), but it eliminates many invalid inputs instantly.


---

4. Which Algorithm and Why?

## Algorithm: Union-Find

### Why Union-Find?

| Option | Time | Space | Notes | Chosen? |
| --- | --- | --- | --- | --- |
| DFS + visited | O(V+E) | O(V+E) | Good, easy to understand | Alternative |
| **Union-Find** | **O(E·α(V))** | **O(V)** | **Elegant, handles both checks in one pass** | **Yes** |

### Union-Find Logic

A valid tree satisfies:
1. **n-1 edges exactly** (quick pre-check).
2. **No cycle** — when processing an edge `(u, v)`, if `find(u) == find(v)`, they're already connected → adding this edge creates a cycle.
3. **Connected** — if the above two conditions hold, the graph is a tree.

```
if edges.length != n - 1: return false

for each edge (u, v):
    if find(u) == find(v): return false  // cycle
    union(u, v)

return true  // n-1 edges + no cycle → tree
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Edge Count Guard** — Pre-check: `edges.length != n-1` → false.
2. **Union-Find Structure** — `parent[]` and `rank[]` arrays.
3. **Find with Path Compression** — `find(x)` collapses paths for efficiency.
4. **Union by Rank** — `union(x, y)` merges smaller tree under larger.
5. **Cycle Detector** — Returns false if `find(u) == find(v)` before union.
6. **Output** — If all edges processed without cycle → valid tree.

### Data Flow Diagram

```
n=5, edges=[[0,1],[0,2],[0,3],[1,4]]

parent = [0,1,2,3,4]  (each node is its own root initially)

Process [0,1]: find(0)=0, find(1)=1 → different → union(0,1) → parent[1]=0
Process [0,2]: find(0)=0, find(2)=2 → different → union(0,2) → parent[2]=0
Process [0,3]: find(0)=0, find(3)=3 → different → union(0,3) → parent[3]=0
Process [1,4]: find(1)=0, find(4)=4 → different → union(0,4) → parent[4]=0

4 edges, 5 nodes, no cycle → return true ✓

---

Cycle example: n=4, edges=[[0,1],[1,2],[2,0]]
Process [0,1]: union → parent[1]=0
Process [1,2]: union → parent[2]=0
Process [2,0]: find(2)=0, find(0)=0 → SAME → cycle → return false ✗
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class GraphValidTree {

    int[] parent, rank;

    // Find root with path compression
    private int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    // Union by rank; returns false if already in same component (cycle)
    private boolean union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false; // same component → cycle

        if (rank[px] < rank[py]) { int tmp = px; px = py; py = tmp; }
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
        return true;
    }

    /**
     * Determine if n nodes and given edges form a valid undirected tree.
     *
     * A valid tree has:
     *   (1) Exactly n-1 edges
     *   (2) No cycles (all n nodes connected in one component)
     *
     * Union-Find processes each edge once: if two nodes already share
     * a root, the edge would create a cycle → not a tree.
     *
     * @param n     number of nodes (0 to n-1)
     * @param edges list of undirected edges
     * @return true if the graph is a valid tree
     *
     * Time:  O(E · α(V)) ≈ O(E)
     * Space: O(V)
     */
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n - 1) return false; // a tree has exactly n-1 edges

        parent = new int[n];
        rank   = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i; // each node is its own root

        for (int[] edge : edges) {
            if (!union(edge[0], edge[1])) return false; // cycle detected
        }

        return true; // n-1 edges + no cycle → connected tree
    }

    // DFS alternative
    public boolean validTree_DFS(int n, int[][] edges) {
        if (edges.length != n - 1) return false;

        java.util.List<java.util.List<Integer>> adj = new java.util.ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new java.util.ArrayList<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
        }

        boolean[] visited = new boolean[n];
        dfs(adj, visited, 0);

        for (boolean v : visited) if (!v) return false; // disconnected component
        return true;
    }

    private void dfs(java.util.List<java.util.List<Integer>> adj, boolean[] visited, int node) {
        visited[node] = true;
        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) dfs(adj, visited, neighbor);
        }
    }

    public static void main(String[] args) {
        GraphValidTree sol = new GraphValidTree();

        // Example 1: Valid tree
        System.out.println("n=5, edges=[[0,1],[0,2],[0,3],[1,4]]");
        System.out.println("Output: " + sol.validTree(5, new int[][]{{0,1},{0,2},{0,3},{1,4}})); // true

        // Example 2: Cycle exists
        System.out.println("\nn=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]]");
        System.out.println("Output: " + sol.validTree(5, new int[][]{{0,1},{1,2},{2,3},{1,3},{1,4}})); // false

        // Example 3: Single node, no edges
        System.out.println("\nn=1, edges=[]");
        System.out.println("Output: " + sol.validTree(1, new int[][]{})); // true

        // Example 4: Disconnected (too few edges)
        System.out.println("\nn=4, edges=[[0,1],[2,3]]");
        System.out.println("Output: " + sol.validTree(4, new int[][]{{0,1},{2,3}})); // false
    }
}
```

### Output

```
n=5, edges=[[0,1],[0,2],[0,3],[1,4]]
Output: true

n=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false

n=1, edges=[]
Output: true

n=4, edges=[[0,1],[2,3]]
Output: false
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def valid_tree(n: int, edges: list[list[int]]) -> bool:
    """
    Determine if n nodes and given edges form a valid undirected tree.

    Strategy: Union-Find.
    A tree has exactly n-1 edges and no cycles.
    Process each edge: if both endpoints already share a root → cycle.

    Args:
        n: Number of nodes
        edges: List of undirected edges

    Returns:
        True if the graph is a valid tree, False otherwise

    Time:  O(E · α(V)) ≈ O(E)
    Space: O(V)
    """
    if len(edges) != n - 1:
        return False  # wrong edge count

    parent = list(range(n))
    rank   = [0] * n

    def find(x: int) -> int:
        if parent[x] != x:
            parent[x] = find(parent[x])  # path compression
        return parent[x]

    def union(x: int, y: int) -> bool:
        px, py = find(x), find(y)
        if px == py:
            return False  # same component → cycle
        if rank[px] < rank[py]:
            px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]:
            rank[px] += 1
        return True

    return all(union(u, v) for u, v in edges)


if __name__ == "__main__":
    test_cases = [
        (5, [[0,1],[0,2],[0,3],[1,4]],        True),
        (5, [[0,1],[1,2],[2,3],[1,3],[1,4]],  False),
        (1, [],                               True),
        (4, [[0,1],[2,3]],                    False),
        (3, [[0,1],[1,2]],                    True),
    ]

    for n, edges, expected in test_cases:
        result = valid_tree(n, edges)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] n={n}, edges={edges} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] n=5, edges=[[0,1],[0,2],[0,3],[1,4]] → Output: True, Expected: True
[PASS] n=5, edges=[[0,1],[1,2],[2,3],[1,3],[1,4]] → Output: False, Expected: False
[PASS] n=1, edges=[] → Output: True, Expected: True
[PASS] n=4, edges=[[0,1],[2,3]] → Output: False, Expected: False
[PASS] n=3, edges=[[0,1],[1,2]] → Output: True, Expected: True
```


---

8. Interview Questions and Answers

## Q&A

**Q: What are the two necessary and sufficient conditions for a valid tree?**
A: (1) Exactly `n-1` edges, and (2) the graph is connected (equivalently: no cycles). If both hold, it is a tree. If either fails, it is not.

**Q: Why does exactly n-1 edges + no cycle imply connectivity?**
A: A forest (collection of trees) with `n` nodes has at most `n-1` edges. If there are exactly `n-1` edges and no cycle, there can be only one connected component — making it a single spanning tree.

**Q: What is path compression in Union-Find?**
A: When calling `find(x)`, instead of traversing up to the root every time, we set each visited node's parent directly to the root. This flattens the tree and amortizes future lookups to nearly O(1).

**Q: When would you choose DFS over Union-Find here?**
A: DFS is intuitive and builds an adjacency list (useful for other queries). Union-Find is more efficient for just the validity check, especially when edges arrive dynamically.

**Q: What if n=1 and edges=[]?**
A: A single node with no edges is a valid tree. `edges.length = 0 = n-1 = 0`. Union-Find trivially returns true.

**Q: How would you handle a directed graph version?**
A: For a directed tree (rooted tree), each node except the root has exactly one parent. Check: each non-root has exactly one incoming edge, and there are no cycles (DFS with 3-state coloring).


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Determine if n nodes and edges form a valid undirected tree        |
| **Optimal Algorithm** | Union-Find (or DFS + connectivity check)                           |
| **Time Complexity**   | O(E · α(V)) ≈ O(E)                                               |
| **Space Complexity**  | O(V)                                                               |
| **Key Insight**       | Tree ↔ exactly n-1 edges + no cycles (connectivity follows)       |

### Core Pattern

> Pre-check edge count (`n-1`). Then use Union-Find: for each edge, if both nodes already share a root, it's a cycle. If all edges are processed without a cycle, it's a valid tree.

### When to Use This Pattern

- Tree/forest validation.
- Cycle detection in undirected graphs.
- Related: Number of Connected Components, Redundant Connection, Minimum Spanning Tree (Kruskal's).

