# Course Schedule

---

<details>
<summary><strong>1. What is meant by Course Schedule?</strong></summary>

## What is Course Schedule?

**Course Schedule** is a directed graph cycle-detection problem where:

- You have `numCourses` courses labeled `0` to `numCourses - 1`.
- `prerequisites[i] = [a, b]` means you must take course `b` before course `a`.

**Goal:** Determine if it is **possible to finish all courses** — i.e., the prerequisite graph has **no cycle**.

**Constraints:**
- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= 5000`
- Each pair `[a, b]` is unique.

**Example:**

```
Input:  numCourses = 2, prerequisites = [[1, 0]]
Output: true
Explanation: Take course 0 first, then course 1.

Input:  numCourses = 2, prerequisites = [[1, 0], [0, 1]]
Output: false
Explanation: Cycle: 0 → 1 → 0
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Determine if a valid course ordering exists (no cycle in the directed prerequisite graph).
- Return `true` if all courses can be completed; `false` if a circular dependency exists.

### Non-Functional Requirements

| Concern         | Expectation                                            |
| --------------- | ------------------------------------------------------ |
| **Scale**       | Up to 2000 courses, 5000 prerequisites — O(V+E) fast  |
| **Latency**     | O(V + E) — linear                                      |
| **Correctness** | Must detect all directed cycles, including indirect    |
| **Edge Cases**  | No prerequisites (always true), self-loop (false)      |

### Clarifying Questions to Ask

- Can there be duplicate prerequisite pairs? → No (guaranteed unique).
- Can a course be a prerequisite of itself? → No self-loops per constraints.
- Is the graph directed? → Yes (prerequisite direction matters).
- Do we need the actual order? → No (that's Course Schedule II).

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### DFS with 3-State Coloring (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(V + E) — each node and edge visited once |
| **Space** | O(V + E) — adjacency list + state array + call stack |

### Kahn's Algorithm (BFS Topological Sort)

| | Complexity |
| --- | --- |
| **Time** | O(V + E) — process each node and edge once |
| **Space** | O(V + E) — adjacency list + in-degree array + queue |

**Both are O(V+E)**. The choice is style: DFS detects cycles directly; Kahn's algorithm detects them indirectly (if not all nodes are processed, there's a cycle).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: DFS with 3-State Node Coloring

### Why?

| Option | Cycle Detection | Time | Chosen? |
| --- | --- | --- | --- |
| DFS (2-state: visited/unvisited) | Incorrect for directed graphs | O(V+E) | No |
| **DFS (3-state: unvisited/visiting/visited)** | **Correct** | **O(V+E)** | **Yes** |
| Kahn's BFS (in-degree) | Correct | O(V+E) | Alternative |

### Why 3 States?

In directed graphs, a node can be reachable via multiple paths. 2-state (visited/unvisited) would incorrectly flag legitimate diamond-shaped paths as cycles.

- **0 = Unvisited** — not yet explored
- **1 = Visiting** — currently in the DFS call stack (part of the current path)
- **2 = Visited** — fully processed, no cycle found from here

A cycle is detected when DFS reaches a node in state **1** (currently on the stack) — this means we found a back edge.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Graph Builder** — Convert prerequisites list into adjacency list.
2. **State Array** — `int[numCourses]`: 0=unvisited, 1=visiting, 2=visited.
3. **DFS Engine** — Recursively explores each unvisited node.
4. **Cycle Detector** — Returns false when a "visiting" node is encountered.
5. **Output** — Return true if DFS completes without detecting a cycle.

### Data Flow Diagram

```
numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]

Adjacency list (a → [b means a requires b]):
0 → []
1 → [0]
2 → [0]
3 → [1, 2]

DFS from 0: state[0]=1 → no neighbors → state[0]=2
DFS from 1: state[1]=1 → visit 0 → state[0]=2 (done) → state[1]=2
DFS from 2: state[2]=1 → visit 0 → state[0]=2 (done) → state[2]=2
DFS from 3: state[3]=1 → visit 1 (state=2 ✓) → visit 2 (state=2 ✓) → state[3]=2

No cycle found → return true ✓

---

Cycle example: [[0,1],[1,0]]
DFS from 0: state[0]=1 → visit 1: state[1]=1 → visit 0: state[0]=1 → CYCLE! false
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class CourseSchedule {

    /**
     * Determine if all courses can be finished (no cycle in prerequisite graph).
     *
     * DFS with 3-state coloring:
     *   0 = unvisited, 1 = visiting (on current DFS path), 2 = completed (safe)
     * A back edge (visiting → visiting) means a cycle exists.
     *
     * @param numCourses    total number of courses
     * @param prerequisites pairs [a, b] meaning b must be taken before a
     * @return true if all courses can be completed (no cycle)
     *
     * Time:  O(V + E)
     * Space: O(V + E)
     */
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        // Build adjacency list: course → list of its prerequisites
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        for (int[] pre : prerequisites) {
            adj.get(pre[0]).add(pre[1]);
        }

        int[] state = new int[numCourses]; // 0=unvisited, 1=visiting, 2=done

        for (int i = 0; i < numCourses; i++) {
            if (state[i] == 0 && hasCycle(adj, state, i)) return false;
        }
        return true;
    }

    // Returns true if a cycle is found starting from 'node'
    private boolean hasCycle(List<List<Integer>> adj, int[] state, int node) {
        state[node] = 1; // mark as visiting (on current path)

        for (int neighbor : adj.get(node)) {
            if (state[neighbor] == 1) return true;  // back edge → cycle
            if (state[neighbor] == 0 && hasCycle(adj, state, neighbor)) return true;
        }

        state[node] = 2; // mark as completed — no cycle from here
        return false;
    }

    /**
     * Kahn's Algorithm (BFS / in-degree) — alternative approach.
     * If not all nodes are processed, there's a cycle.
     */
    public boolean canFinish_Kahn(int numCourses, int[][] prerequisites) {
        int[] inDegree = new int[numCourses];
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());

        for (int[] pre : prerequisites) {
            adj.get(pre[1]).add(pre[0]); // b → a (b is a prerequisite of a)
            inDegree[pre[0]]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) queue.offer(i); // start with courses that have no prerequisites
        }

        int processed = 0;
        while (!queue.isEmpty()) {
            int course = queue.poll();
            processed++;
            for (int next : adj.get(course)) {
                if (--inDegree[next] == 0) queue.offer(next);
            }
        }

        return processed == numCourses; // all courses processed → no cycle
    }

    public static void main(String[] args) {
        CourseSchedule sol = new CourseSchedule();

        // Example 1: Valid order exists
        System.out.println("numCourses=2, prereqs=[[1,0]]");
        System.out.println("Output: " + sol.canFinish(2, new int[][]{{1, 0}})); // true

        // Example 2: Cycle exists
        System.out.println("\nnumCourses=2, prereqs=[[1,0],[0,1]]");
        System.out.println("Output: " + sol.canFinish(2, new int[][]{{1, 0}, {0, 1}})); // false

        // Example 3: No prerequisites
        System.out.println("\nnumCourses=4, prereqs=[]");
        System.out.println("Output: " + sol.canFinish(4, new int[][]{})); // true

        // Example 4: Longer cycle
        System.out.println("\nnumCourses=3, prereqs=[[0,1],[1,2],[2,0]]");
        System.out.println("Output: " + sol.canFinish(3, new int[][]{{0, 1}, {1, 2}, {2, 0}})); // false
    }
}
```

### Output

```
numCourses=2, prereqs=[[1,0]]
Output: true

numCourses=2, prereqs=[[1,0],[0,1]]
Output: false

numCourses=4, prereqs=[]
Output: true

numCourses=3, prereqs=[[0,1],[1,2],[2,0]]
Output: false
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
from collections import defaultdict, deque

def can_finish(num_courses: int, prerequisites: list[list[int]]) -> bool:
    """
    Determine if all courses can be finished (no cycle in prerequisite graph).

    Strategy: DFS with 3-state coloring.
    0 = unvisited, 1 = visiting (on current path), 2 = fully processed (safe)
    If DFS reaches a node with state 1, a back edge (cycle) is detected.

    Args:
        num_courses: Total number of courses
        prerequisites: List of [a, b] pairs (b must come before a)

    Returns:
        True if all courses can be completed (no cycle), False otherwise

    Time:  O(V + E)
    Space: O(V + E)
    """
    adj = defaultdict(list)
    for course, prereq in prerequisites:
        adj[course].append(prereq)

    state = [0] * num_courses  # 0=unvisited, 1=visiting, 2=done

    def has_cycle(node: int) -> bool:
        state[node] = 1  # mark visiting

        for neighbor in adj[node]:
            if state[neighbor] == 1:  # back edge → cycle
                return True
            if state[neighbor] == 0 and has_cycle(neighbor):
                return True

        state[node] = 2  # mark fully processed
        return False

    return all(state[i] != 0 or not has_cycle(i) for i in range(num_courses))


def can_finish_kahn(num_courses: int, prerequisites: list[list[int]]) -> bool:
    """Kahn's BFS topological sort — cycle detected if not all nodes processed."""
    in_degree = [0] * num_courses
    adj = defaultdict(list)

    for course, prereq in prerequisites:
        adj[prereq].append(course)
        in_degree[course] += 1

    queue = deque(i for i in range(num_courses) if in_degree[i] == 0)
    processed = 0

    while queue:
        node = queue.popleft()
        processed += 1
        for neighbor in adj[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return processed == num_courses


if __name__ == "__main__":
    test_cases = [
        (2, [[1, 0]],              True),
        (2, [[1, 0], [0, 1]],      False),
        (4, [],                    True),
        (3, [[0, 1], [1, 2], [2, 0]], False),
    ]

    for n, prereqs, expected in test_cases:
        result = can_finish(n, prereqs)
        kahn   = can_finish_kahn(n, prereqs)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] n={n}, prereqs={prereqs}")
        print(f"  DFS: {result}, Kahn: {kahn}, Expected: {expected}\n")
```

### Output

```
[PASS] n=2, prereqs=[[1, 0]]
  DFS: True, Kahn: True, Expected: True

[PASS] n=2, prereqs=[[1, 0], [0, 1]]
  DFS: False, Kahn: False, Expected: False

[PASS] n=4, prereqs=[]
  DFS: True, Kahn: True, Expected: True

[PASS] n=3, prereqs=[[0, 1], [1, 2], [2, 0]]
  DFS: False, Kahn: False, Expected: False
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why do we need 3 states instead of 2 (visited/unvisited)?**
A: In a directed graph, a node can be reachable via multiple paths. With 2 states, visiting a node already marked "visited" (but via a different path) would be incorrectly flagged as a cycle. State "1" (visiting) specifically means "currently on the active DFS path," which is the actual cycle indicator.

**Q: What does Kahn's algorithm detect and how?**
A: Kahn's processes nodes with in-degree 0 first, reduces in-degrees of neighbors. If the graph has a cycle, those cycle nodes will never reach in-degree 0 and won't be processed. If `processed < numCourses`, a cycle exists.

**Q: How does Course Schedule II differ?**
A: Course Schedule II returns the actual ordering (topological sort order). Use Kahn's algorithm and collect nodes as they're processed — the collection order is a valid ordering.

**Q: Can this approach handle disconnected graphs?**
A: Yes. The outer loop `for i in range(num_courses)` ensures we start DFS from every unvisited node, covering all connected components.

**Q: What is the real-world analogy?**
A: Dependency resolution — package managers (npm, Maven) use topological sort to determine build/install order. A cycle means a dependency deadlock (A requires B requires A).

**Q: When would in-degree (Kahn's) be preferred over DFS?**
A: Kahn's is naturally iterative (no recursion stack overflow), and it produces the topological order as a byproduct. DFS is more compact for just detecting cycles.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                            |
| --------------------- | ---------------------------------------------------------------- |
| **Problem**           | Detect if a directed course-prerequisite graph has a cycle       |
| **Optimal Algorithm** | DFS with 3-state coloring OR Kahn's BFS (topological sort)      |
| **Time Complexity**   | O(V + E)                                                         |
| **Space Complexity**  | O(V + E)                                                         |
| **Key Insight**       | Back edge (reaching a "visiting" node) = cycle in directed graph |

### Core Pattern

> Color nodes: unvisited → visiting → done. If DFS steps onto a "visiting" node, you've found a cycle. If all nodes reach "done," the graph is a DAG (no cycle).

### When to Use This Pattern

- Cycle detection in directed graphs.
- Task scheduling, build systems, dependency resolution.
- Related: Course Schedule II, Alien Dictionary, Task Scheduler.

</details>
