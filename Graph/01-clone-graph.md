# Clone Graph

---

1. What is meant by Clone Graph?

## What is Clone Graph?

**Clone Graph** is a graph traversal problem where:

- You are given a reference to a node in a **connected undirected graph**.
- Each node has an integer `val` and a list of `neighbors`.

**Goal:** Return a **deep copy** (clone) of the graph — every node and every edge must be duplicated into a completely new set of nodes.

**Constraints:**
- The number of nodes is in the range `[0, 100]`.
- `1 <= Node.val <= 100`
- Each node value is unique.
- No self-loops or repeated edges.

**Example:**

```
Input:  adjList = [[2,4],[1,3],[2,4],[1,3]]
         Node 1 — neighbors: [2, 4]
         Node 2 — neighbors: [1, 3]
         Node 3 — neighbors: [2, 4]
         Node 4 — neighbors: [1, 3]

Output: A deep copy with the same adjacency structure.
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Produce a deep copy: new node objects with the same values and edges.
- The original graph must remain unchanged.
- The clone must be a separate object graph (no shared node references).

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 100 nodes, 100 edges — trivially small         |
| **Latency**     | O(V + E) — linear in graph size                      |
| **Correctness** | Every node cloned exactly once; all edges preserved  |
| **Memory**      | O(V) for the visited hashmap                         |

### Clarifying Questions to Ask

- Is the graph connected? → Yes (guaranteed by problem).
- Can there be cycles? → Yes — must handle to avoid infinite loops.
- Can the input be null? → Yes — return null.
- Are there self-loops? → No per constraints.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### BFS / DFS with HashMap

| | Complexity |
| --- | --- |
| **Time** | O(V + E) — visit every node once, process every edge once |
| **Space** | O(V) — HashMap storing original → clone mappings + queue/stack |

**Why O(V + E)?** BFS/DFS traverses each vertex once and each edge once in an undirected graph (each edge seen twice, once from each endpoint).

### Without Memoization (Wrong — Infinite Loop)

Cycles would cause infinite recursion/traversal without the visited map.


---

4. Which Algorithm and Why?

## Algorithm: BFS with HashMap

### Why BFS?

| Option | Correct? | Notes | Chosen? |
| --- | --- | --- | --- |
| BFS + HashMap | Yes | Iterative, safe for large graphs | Yes |
| DFS + HashMap | Yes | Recursive — stack overflow risk for very deep graphs | Equivalent |
| No HashMap | No | Cycles cause infinite loops | No |

### Key Insight

Use a `HashMap<Node, Node>` (original → clone) to:
1. **Check if already cloned** — avoid duplicating nodes in cyclic graphs.
2. **Connect neighbors** — look up the clone of each neighbor from the map.

### BFS Algorithm

```
1. Create a clone of the starting node.
2. Put it in visited map: original → clone.
3. Add original node to queue.
4. While queue is not empty:
   a. Dequeue node.
   b. For each neighbor of node:
      - If not in visited: create clone, add to map, enqueue original.
      - Add clone of neighbor to clone node's neighbor list.
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Input Guard** — Return null if input is null.
2. **Clone Map** — `HashMap<Node, Node>` mapping originals to clones.
3. **BFS Queue** — Processes each original node exactly once.
4. **Neighbor Linker** — Connects each clone node to the clones of its neighbors.
5. **Output** — Return the clone of the input node.

### Data Flow Diagram

```
Original Graph:       Clone Graph (output):
  1 -- 2                1' -- 2'
  |    |                |     |
  4 -- 3                4' -- 3'

BFS Traversal:
Queue: [1]
  Process 1 → clone 1' → neighbors [2, 4] → clone 2', 4' → Queue: [2, 4]
  Process 2 → clone 2' → neighbors [1, 3] → 1' in map, clone 3' → Queue: [4, 3]
    Link: 2'.neighbors = [1', 3']
  Process 4 → clone 4' → neighbors [1, 3] → both in map
    Link: 4'.neighbors = [1', 3']
  Process 3 → clone 3' → neighbors [2, 4] → both in map
    Link: 3'.neighbors = [2', 4']

Return: clone of node 1 = 1'
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class CloneGraph {

    // Definition for a Node
    static class Node {
        int val;
        List<Node> neighbors;

        Node(int val) {
            this.val = val;
            this.neighbors = new ArrayList<>();
        }
    }

    /**
     * Deep copy a connected undirected graph using BFS + HashMap.
     *
     * HashMap maps each original node to its clone.
     * BFS ensures each node is visited and cloned exactly once.
     * Neighbor lists are built by looking up clones from the map.
     *
     * @param node starting node of the graph (may be null)
     * @return clone of the starting node, or null
     *
     * Time:  O(V + E)
     * Space: O(V) — for the visited map and BFS queue
     */
    public Node cloneGraph(Node node) {
        if (node == null) return null;

        Map<Node, Node> visited = new HashMap<>(); // original → clone
        Queue<Node> queue = new LinkedList<>();

        // Clone and enqueue the starting node
        visited.put(node, new Node(node.val));
        queue.offer(node);

        while (!queue.isEmpty()) {
            Node curr = queue.poll();

            for (Node neighbor : curr.neighbors) {
                // Clone neighbor if not yet seen
                if (!visited.containsKey(neighbor)) {
                    visited.put(neighbor, new Node(neighbor.val));
                    queue.offer(neighbor);
                }
                // Connect the clone of curr to the clone of this neighbor
                visited.get(curr).neighbors.add(visited.get(neighbor));
            }
        }

        return visited.get(node); // return the clone of the input node
    }

    // DFS recursive version (alternative)
    public Node cloneGraph_DFS(Node node) {
        if (node == null) return null;
        return dfs(node, new HashMap<>());
    }

    private Node dfs(Node node, Map<Node, Node> visited) {
        if (visited.containsKey(node)) return visited.get(node);

        Node clone = new Node(node.val);
        visited.put(node, clone); // put BEFORE recursing to handle cycles

        for (Node neighbor : node.neighbors) {
            clone.neighbors.add(dfs(neighbor, visited));
        }
        return clone;
    }

    public static void main(String[] args) {
        CloneGraph sol = new CloneGraph();

        // Build graph: 1 -- 2 -- 3 -- 4 -- 1 (cycle)
        Node n1 = new Node(1);
        Node n2 = new Node(2);
        Node n3 = new Node(3);
        Node n4 = new Node(4);
        n1.neighbors.addAll(Arrays.asList(n2, n4));
        n2.neighbors.addAll(Arrays.asList(n1, n3));
        n3.neighbors.addAll(Arrays.asList(n2, n4));
        n4.neighbors.addAll(Arrays.asList(n1, n3));

        Node cloned = sol.cloneGraph(n1);

        // Verify: same structure, different objects
        System.out.println("Original node 1: " + System.identityHashCode(n1));
        System.out.println("Cloned   node 1: " + System.identityHashCode(cloned));
        System.out.println("Same object? " + (n1 == cloned));                     // false
        System.out.println("Same value?  " + (n1.val == cloned.val));             // true
        System.out.println("Cloned node 1 neighbors count: " + cloned.neighbors.size()); // 2

        // Null input
        System.out.println("\nNull input: " + sol.cloneGraph(null)); // null

        // Single node, no neighbors
        Node single = new Node(7);
        Node clonedSingle = sol.cloneGraph(single);
        System.out.println("\nSingle node val: " + clonedSingle.val);             // 7
        System.out.println("Single node neighbors: " + clonedSingle.neighbors);   // []
    }
}
```

### Output

```
Original node 1: 1829164700
Cloned   node 1: 2018699554
Same object? false
Same value?  true
Cloned node 1 neighbors count: 2

Null input: null

Single node val: 7
Single node neighbors: []
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from collections import deque

class Node:
    def __init__(self, val: int, neighbors: list = None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []


def clone_graph(node: 'Node') -> 'Node':
    """
    Deep copy a connected undirected graph using BFS + HashMap.

    For each original node, create a clone and store in a dict.
    BFS processes each node once; neighbor lists are built from the clone dict.

    Args:
        node: Starting node (may be None)

    Returns:
        Clone of the starting node, or None

    Time:  O(V + E)
    Space: O(V)
    """
    if not node:
        return None

    visited = {}  # original node → clone node
    visited[node] = Node(node.val)
    queue = deque([node])

    while queue:
        curr = queue.popleft()

        for neighbor in curr.neighbors:
            if neighbor not in visited:
                visited[neighbor] = Node(neighbor.val)
                queue.append(neighbor)
            # Link clone of curr to clone of neighbor
            visited[curr].neighbors.append(visited[neighbor])

    return visited[node]


def clone_graph_dfs(node: 'Node', visited: dict = None) -> 'Node':
    """DFS recursive version."""
    if not node:
        return None
    if visited is None:
        visited = {}
    if node in visited:
        return visited[node]

    clone = Node(node.val)
    visited[node] = clone  # store BEFORE recursing (cycle safety)

    for neighbor in node.neighbors:
        clone.neighbors.append(clone_graph_dfs(neighbor, visited))

    return clone


if __name__ == "__main__":
    # Build: 1 -- 2 -- 3 -- 4 -- 1
    n1, n2, n3, n4 = Node(1), Node(2), Node(3), Node(4)
    n1.neighbors = [n2, n4]
    n2.neighbors = [n1, n3]
    n3.neighbors = [n2, n4]
    n4.neighbors = [n1, n3]

    cloned = clone_graph(n1)

    print(f"Original id: {id(n1)}, Clone id: {id(cloned)}")
    print(f"Same object? {n1 is cloned}")        # False
    print(f"Same value?  {n1.val == cloned.val}") # True
    print(f"Clone neighbors: {[nb.val for nb in cloned.neighbors]}")  # [2, 4]

    # Null input
    print(f"\nNull input: {clone_graph(None)}")  # None

    # Single node
    single = Node(5)
    c = clone_graph(single)
    print(f"\nSingle: val={c.val}, neighbors={c.neighbors}")  # val=5, neighbors=[]
```

### Output

```
Original id: 4371234512, Clone id: 4371234672
Same object? False
Same value?  True
Clone neighbors: [2, 4]

Null input: None

Single: val=5, neighbors=[]
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why do we need a HashMap (visited map)?**
A: Without it, cycles in the graph cause infinite recursion (DFS) or infinite looping (BFS). The map serves two purposes: it prevents revisiting nodes and it lets us look up the clone for any original node in O(1).

**Q: In DFS, why must we put the clone in the map BEFORE recursing into neighbors?**
A: If we recurse first, a cycle will bring us back to the same node before it's in the map — we'd create a second clone, resulting in duplicates. Inserting the clone first breaks the cycle immediately.

**Q: BFS vs DFS — which is better here?**
A: Both are O(V+E). BFS is iterative (safe for very large graphs — no stack overflow risk). DFS is more concise. For interviews, DFS with memoization is usually shown as it's fewer lines.

**Q: What if the graph is disconnected?**
A: The problem guarantees a connected graph. For disconnected graphs, you'd need to iterate over all nodes (e.g., from an adjacency list) and clone each connected component separately.

**Q: What's the difference between a shallow copy and a deep copy here?**
A: A shallow copy would create new node objects but share the same neighbor list references — modifying the clone's neighbors would affect the original. A deep copy creates new nodes AND new neighbor lists with references to the cloned nodes.

**Q: Can we solve this without a HashMap?**
A: Not cleanly for cyclic graphs. For acyclic graphs (trees), we could use recursion without a map. For general graphs, the map is essential.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Deep copy a connected undirected graph                             |
| **Optimal Algorithm** | BFS or DFS with HashMap (original → clone)                        |
| **Time Complexity**   | O(V + E)                                                           |
| **Space Complexity**  | O(V)                                                               |
| **Key Insight**       | HashMap prevents infinite loops on cycles AND enables neighbor lookup |

### Core Pattern

> To clone a graph: create the clone first, store the mapping, then fill in its neighbors by looking up (or creating) their clones. Never recurse into a node that's already in the map.

### When to Use This Pattern

- Any graph deep copy / serialization task.
- Related: Serialize and Deserialize Binary Tree, Graph traversal with memoization.

