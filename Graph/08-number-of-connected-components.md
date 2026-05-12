# Number of Connected Components in an Undirected Graph

---

1. What is meant by Number of Connected Components?

## What is Number of Connected Components?

**Number of Connected Components** is a graph problem where:

- You have `n` nodes labeled `0` to `n-1`.
- You are given a list of undirected `edges`.

**Goal:** Return the **number of connected components** in the graph.

**Constraints:**
- `1 <= n <= 2000`
- `0 <= edges.length <= 5000`
- `edges[i] = [a, b]` — undirected edge between a and b.
- No duplicate edges or self-loops.

**Example:**

```
Input:  n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2
Explanation: Component 1: {0,1,2}, Component 2: {3,4}

Input:  n = 5, edges = [[0,1],[1,2],[2,3],[3,4]]
Output: 1
Explanation: All nodes connected in one component.
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Count the number of distinct groups of nodes that are mutually reachable.
- A node with no edges is its own connected component.
- Edges are undirected.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 2000 nodes, 5000 edges — O(V+E) is fast       |
| **Latency**     | O(V + E) — linear                                    |
| **Correctness** | Isolated nodes count as their own component          |
| **Edge Cases**  | All isolated nodes → n components; all connected → 1 |

### Clarifying Questions to Ask

- Is the graph directed or undirected? → Undirected.
- Can nodes be isolated (no edges)? → Yes — each is its own component.
- Are edges guaranteed unique? → Yes.
- Do we need to return the component groups or just the count? → Just the count.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### DFS / BFS

| | Complexity |
| --- | --- |
| **Time** | O(V + E) — visit every node and edge once |
| **Space** | O(V + E) — adjacency list + visited array + stack/queue |

### Union-Find (Optimal for this problem)

| | Complexity |
| --- | --- |
| **Time** | O(E · α(V)) ≈ O(E) — nearly linear |
| **Space** | O(V) — parent and rank arrays |

**Union-Find is preferred** here because:
- Minimal space (no adjacency list needed).
- Component count is maintained automatically.
- Handles dynamic edge additions efficiently.


---

4. Which Algorithm and Why?

## Algorithm: Union-Find

### Why Union-Find?

| Option | Time | Space | Notes | Chosen? |
| --- | --- | --- | --- | --- |
| DFS/BFS | O(V+E) | O(V+E) | Requires adjacency list | Alternative |
| **Union-Find** | **O(E·α(V))** | **O(V)** | **No adjacency list, direct component count** | **Yes** |

### Key Insight

Start with `n` components (each node is its own component).

For each edge `(u, v)`:
- If `find(u) != find(v)`: merge components → decrement component count.
- If `find(u) == find(v)`: already same component → do nothing.

At the end, the component count is the answer.

```
components = n

for each edge (u, v):
    if union(u, v):   // returns true if two different components were merged
        components -= 1

return components
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Union-Find Structure** — `parent[]` (path compression) and `rank[]` (union by rank).
2. **Component Counter** — Starts at `n`, decrements on each successful union.
3. **Edge Processor** — For each edge, attempt union and update counter.
4. **Output** — Return the final component count.

### Data Flow Diagram

```
n=5, edges=[[0,1],[1,2],[3,4]]

Initial: parent=[0,1,2,3,4], components=5

Edge [0,1]: find(0)=0, find(1)=1 → merge → parent[1]=0, components=4
Edge [1,2]: find(1)=0, find(2)=2 → merge → parent[2]=0, components=3
Edge [3,4]: find(3)=3, find(4)=4 → merge → parent[4]=3, components=2

Output: 2 ✓

---

n=5, edges=[[0,1],[1,2],[2,3],[3,4]]

After all merges: parent=[0,0,0,0,0], components=1

Output: 1 ✓

---

n=3, edges=[] (all isolated)

No edges processed, components stays at 3.

Output: 3 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class NumberOfConnectedComponents {

    int[] parent, rank;

    private int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    // Returns true if union succeeded (two different components merged)
    private boolean union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;

        if (rank[px] < rank[py]) { int tmp = px; px = py; py = tmp; }
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
        return true;
    }

    /**
     * Count the number of connected components in an undirected graph.
     *
     * Strategy: Union-Find.
     * Start with n components. Each successful union reduces the count by 1.
     *
     * @param n     number of nodes
     * @param edges list of undirected edges
     * @return number of connected components
     *
     * Time:  O(E · α(V)) ≈ O(E)
     * Space: O(V)
     */
    public int countComponents(int n, int[][] edges) {
        parent = new int[n];
        rank   = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i; // each node is its own root

        int components = n; // start with n disconnected components

        for (int[] edge : edges) {
            if (union(edge[0], edge[1])) {
                components--; // two components merged into one
            }
        }

        return components;
    }

    // DFS alternative
    public int countComponents_DFS(int n, int[][] edges) {
        java.util.List<java.util.List<Integer>> adj = new java.util.ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new java.util.ArrayList<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
        }

        boolean[] visited = new boolean[n];
        int count = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                count++;
                dfs(adj, visited, i);
            }
        }
        return count;
    }

    private void dfs(java.util.List<java.util.List<Integer>> adj, boolean[] visited, int node) {
        visited[node] = true;
        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) dfs(adj, visited, neighbor);
        }
    }

    public static void main(String[] args) {
        NumberOfConnectedComponents sol = new NumberOfConnectedComponents();

        // Example 1: 2 components
        System.out.println("n=5, edges=[[0,1],[1,2],[3,4]]");
        System.out.println("Output: " + sol.countComponents(5, new int[][]{{0,1},{1,2},{3,4}})); // 2

        // Example 2: 1 component
        System.out.println("\nn=5, edges=[[0,1],[1,2],[2,3],[3,4]]");
        System.out.println("Output: " + sol.countComponents(5, new int[][]{{0,1},{1,2},{2,3},{3,4}})); // 1

        // Example 3: All isolated
        System.out.println("\nn=4, edges=[]");
        System.out.println("Output: " + sol.countComponents(4, new int[][]{})); // 4

        // Example 4: Single node
        System.out.println("\nn=1, edges=[]");
        System.out.println("Output: " + sol.countComponents(1, new int[][]{})); // 1
    }
}
```

### Output

```
n=5, edges=[[0,1],[1,2],[3,4]]
Output: 2

n=5, edges=[[0,1],[1,2],[2,3],[3,4]]
Output: 1

n=4, edges=[]
Output: 4

n=1, edges=[]
Output: 1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def count_components(n: int, edges: list[list[int]]) -> int:
    """
    Count connected components in an undirected graph using Union-Find.

    Start with n components (each node isolated).
    For each edge, if the two endpoints belong to different components,
    merge them and decrement the count.

    Args:
        n: Number of nodes
        edges: List of undirected edges

    Returns:
        Number of connected components

    Time:  O(E · α(V)) ≈ O(E)
    Space: O(V)
    """
    parent = list(range(n))
    rank   = [0] * n

    def find(x: int) -> int:
        if parent[x] != x:
            parent[x] = find(parent[x])  # path compression
        return parent[x]

    def union(x: int, y: int) -> bool:
        px, py = find(x), find(y)
        if px == py:
            return False  # already in same component

        if rank[px] < rank[py]:
            px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]:
            rank[px] += 1
        return True

    components = n  # each node starts as its own component

    for u, v in edges:
        if union(u, v):
            components -= 1  # merged two components

    return components


def count_components_dfs(n: int, edges: list[list[int]]) -> int:
    """DFS alternative — builds adjacency list first."""
    from collections import defaultdict
    adj = defaultdict(list)
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = set()

    def dfs(node: int) -> None:
        visited.add(node)
        for neighbor in adj[node]:
            if neighbor not in visited:
                dfs(neighbor)

    count = 0
    for i in range(n):
        if i not in visited:
            count += 1
            dfs(i)
    return count


if __name__ == "__main__":
    test_cases = [
        (5, [[0,1],[1,2],[3,4]],        2),
        (5, [[0,1],[1,2],[2,3],[3,4]],  1),
        (4, [],                         4),
        (1, [],                         1),
        (3, [[0,1]],                    2),
    ]

    for n, edges, expected in test_cases:
        uf  = count_components(n, [e[:] for e in edges])
        dfs = count_components_dfs(n, edges)
        status = "PASS" if uf == expected else "FAIL"
        print(f"[{status}] n={n}, edges={edges}")
        print(f"  UF: {uf}, DFS: {dfs}, Expected: {expected}\n")
```

### Output

```
[PASS] n=5, edges=[[0, 1], [1, 2], [3, 4]]
  UF: 2, DFS: 2, Expected: 2

[PASS] n=5, edges=[[0, 1], [1, 2], [2, 3], [3, 4]]
  UF: 1, DFS: 1, Expected: 1

[PASS] n=4, edges=[]
  UF: 4, DFS: 4, Expected: 4

[PASS] n=1, edges=[]
  UF: 1, DFS: 1, Expected: 1

[PASS] n=3, edges=[[0, 1]]
  UF: 2, DFS: 2, Expected: 2
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why start `components = n` instead of 0?**
A: Each node begins as its own component (isolated). Every successful union reduces the count by 1, so starting at `n` and counting down gives the correct final answer without needing a separate counting step.

**Q: What is path compression in `find()`?**
A: When traversing up to the root, we point each node directly to the root instead of its immediate parent. This flattens the tree, making future `find()` calls nearly O(1) (amortized inverse Ackermann).

**Q: What is union by rank?**
A: Always attach the shorter tree under the taller tree. This keeps the overall tree flat and prevents degenerating into a linked list (O(n) lookup). Rank is an upper bound on height.

**Q: How does Union-Find differ from DFS/BFS here?**
A: Union-Find needs no adjacency list — it works directly on the edge list. DFS/BFS requires building an adjacency list first. Union-Find handles dynamic edge additions efficiently; DFS/BFS would need to re-traverse.

**Q: How would you count components in a dynamically changing graph?**
A: Union-Find naturally supports dynamic edge additions — each `union` call takes nearly O(1). For edge deletions, Union-Find doesn't help directly; you'd use a different structure (e.g., offline LCT or rebuild with DFS).

**Q: What is the inverse Ackermann function α(n)?**
A: It's an extremely slowly growing function — for all practical purposes, α(n) ≤ 4 for any realistic input. So Union-Find operations are effectively O(1) each.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Count distinct connected groups of nodes in an undirected graph    |
| **Optimal Algorithm** | Union-Find with path compression and union by rank                 |
| **Time Complexity**   | O(E · α(V)) ≈ O(E)                                               |
| **Space Complexity**  | O(V)                                                               |
| **Key Insight**       | Start with n components; each successful union reduces count by 1  |

### Core Pattern

> Initialize every node as its own component. Process edges: if an edge connects two different components, merge them and decrement the count. Remaining count = number of components.

### When to Use This Pattern

- Counting connected components in static or dynamic graphs.
- Related: Graph Valid Tree, Redundant Connection, Number of Islands, Accounts Merge.
- Union-Find is the go-to data structure whenever "which group does X belong to?" is asked repeatedly.

