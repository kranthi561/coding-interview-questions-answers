# Pacific Atlantic Water Flow

---

<details>
<summary><strong>1. What is meant by Pacific Atlantic Water Flow?</strong></summary>

## What is Pacific Atlantic Water Flow?

**Pacific Atlantic Water Flow** is a graph reachability problem where:

- You have an `m × n` integer matrix `heights` representing a continent's elevation.
- The **Pacific Ocean** borders the top and left edges.
- The **Atlantic Ocean** borders the bottom and right edges.
- Water flows from a cell to an adjacent cell (up/down/left/right) only if the neighbor's height is **≤** the current cell's height.

**Goal:** Return all cells `[r, c]` from which water can flow to **both** the Pacific and Atlantic oceans.

**Constraints:**
- `1 <= m, n <= 200`
- `0 <= heights[r][c] <= 10⁵`

**Example:**

```
Input:
heights = [[1,2,2,3,5],
           [3,2,3,4,4],
           [2,4,5,3,1],
           [6,7,1,4,5],
           [5,1,1,2,4]]

Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Find all cells from which water can reach both oceans.
- Water flows to adjacent cells with equal or lower elevation.
- Diagonal movement is not allowed.

### Non-Functional Requirements

| Concern         | Expectation                                             |
| --------------- | ------------------------------------------------------- |
| **Scale**       | Up to 200×200 grid — O(m×n) is 40,000 cells, fast      |
| **Latency**     | O(m × n) — two BFS/DFS passes                          |
| **Correctness** | Must find every cell reachable by both oceans           |
| **Space**       | O(m × n) — two boolean matrices                        |

### Clarifying Questions to Ask

- Does water flow equally or only downhill? → Equal height OR downhill (≤).
- Are diagonal moves allowed? → No, only 4 directions.
- Can a border cell be in the result? → Yes, border cells are adjacent to an ocean.
- What if the matrix is 1×1? → Always returns `[[0,0]]`.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force (BFS from Every Cell)

| | Complexity |
| --- | --- |
| **Time** | O(m²×n²) — BFS from each of m×n cells costs O(m×n) |
| **Space** | O(m × n) — BFS visited array per cell |

### Approach 2: Reverse BFS from Borders (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(m × n) — each cell visited at most twice (once per ocean) |
| **Space** | O(m × n) — two boolean reachability matrices |

**Key observation:** Instead of asking "can water from cell (r,c) reach the ocean?", reverse the question: "can water flow **from** the ocean **inward** to (r,c)?" Flow inward by climbing uphill (≥ instead of ≤).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Reverse BFS from Both Ocean Borders

### Why Reverse BFS?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| BFS from every cell | O(m²n²) | O(mn) | No — too slow |
| **Reverse BFS from borders** | **O(mn)** | **O(mn)** | **Yes** |

### Reverse Direction Insight

Forward (cell → ocean): water flows down (≤). Checking from each cell is costly.

**Reverse (ocean → cell): "flood" inward going uphill (≥)**. Any cell reachable from the Pacific border in this reverse traversal can flow to the Pacific. Same for Atlantic.

The answer is the **intersection**: cells reachable from both Pacific borders AND Atlantic borders.

### Algorithm Steps

```
1. Initialize Pacific BFS with all top-row and left-column cells.
2. Initialize Atlantic BFS with all bottom-row and right-column cells.
3. Run BFS for Pacific: expand to neighbors with height >= current height.
4. Run BFS for Atlantic: same logic.
5. Collect cells where both pacific[r][c] and atlantic[r][c] are true.
```

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Border Seeders** — Prime the queues with all border cells for each ocean.
2. **BFS Engine** — Expands cells where `neighbor.height >= current.height` (reverse flow).
3. **Pacific Reachability Matrix** — Boolean `m×n` array.
4. **Atlantic Reachability Matrix** — Boolean `m×n` array.
5. **Intersection Collector** — Finds cells marked true in both matrices.

### Data Flow Diagram

```
heights:
  1  2  2  3  5
  3  2  3  4  4
  2  4  5  3  1
  6  7  1  4  5
  5  1  1  2  4

Pacific seeds: top row [0,0..4] + left col [0..4,0]
Atlantic seeds: bottom row [4,0..4] + right col [0..4,4]

After Pacific BFS (P=reachable):
  P  P  P  P  P
  P  .  P  P  P
  P  P  P  P  .
  P  P  .  .  P
  P  .  .  .  P

After Atlantic BFS (A=reachable):
  .  .  .  .  A
  .  .  .  A  A
  .  .  A  .  A
  .  A  .  A  A
  A  A  A  A  A

Intersection (both P and A):
  [0,4], [1,3], [1,4], [2,2], [3,0], [3,1], [4,0]
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class PacificAtlanticWaterFlow {

    private static final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    /**
     * Find all cells from which water can flow to both oceans.
     *
     * Strategy: Reverse BFS from each ocean's border.
     * Instead of flowing downhill from cells, flow uphill from the ocean borders.
     * Any cell reachable from both Pacific and Atlantic borders is an answer.
     *
     * @param heights elevation grid
     * @return list of [r, c] cells reachable by both oceans
     *
     * Time:  O(m × n)
     * Space: O(m × n)
     */
    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        int m = heights.length, n = heights[0].length;
        boolean[][] pacific  = new boolean[m][n];
        boolean[][] atlantic = new boolean[m][n];

        // Seed Pacific: top row + left column
        Queue<int[]> pQueue = new LinkedList<>();
        // Seed Atlantic: bottom row + right column
        Queue<int[]> aQueue = new LinkedList<>();

        for (int r = 0; r < m; r++) {
            pQueue.offer(new int[]{r, 0});    pacific[r][0]   = true;
            aQueue.offer(new int[]{r, n-1});  atlantic[r][n-1] = true;
        }
        for (int c = 0; c < n; c++) {
            pQueue.offer(new int[]{0, c});    pacific[0][c]   = true;
            aQueue.offer(new int[]{m-1, c});  atlantic[m-1][c] = true;
        }

        bfs(heights, pQueue, pacific);
        bfs(heights, aQueue, atlantic);

        // Intersection: cells reachable from both oceans
        List<List<Integer>> result = new ArrayList<>();
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (pacific[r][c] && atlantic[r][c]) {
                    result.add(Arrays.asList(r, c));
                }
            }
        }
        return result;
    }

    // BFS expanding to neighbors with height >= current (reverse flow = uphill)
    private void bfs(int[][] heights, Queue<int[]> queue, boolean[][] visited) {
        int m = heights.length, n = heights[0].length;
        while (!queue.isEmpty()) {
            int[] cell = queue.poll();
            int r = cell[0], c = cell[1];
            for (int[] d : DIRS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n
                        && !visited[nr][nc]
                        && heights[nr][nc] >= heights[r][c]) { // uphill or equal
                    visited[nr][nc] = true;
                    queue.offer(new int[]{nr, nc});
                }
            }
        }
    }

    public static void main(String[] args) {
        PacificAtlanticWaterFlow sol = new PacificAtlanticWaterFlow();

        int[][] heights = {
            {1,2,2,3,5},
            {3,2,3,4,4},
            {2,4,5,3,1},
            {6,7,1,4,5},
            {5,1,1,2,4}
        };
        System.out.println("Output: " + sol.pacificAtlantic(heights));
        // [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]

        // Edge case: 1x1 grid
        System.out.println("\n1x1 grid [[1]]: " + sol.pacificAtlantic(new int[][]{{1}}));
        // [[0,0]]
    }
}
```

### Output

```
Output: [[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]]

1x1 grid [[1]]: [[0, 0]]
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
from collections import deque

def pacific_atlantic(heights: list[list[int]]) -> list[list[int]]:
    """
    Find all cells from which water can flow to both Pacific and Atlantic oceans.

    Strategy: Reverse BFS from each ocean's border cells.
    Flood inward going uphill (neighbor height >= current height).
    Intersection of Pacific-reachable and Atlantic-reachable cells = answer.

    Args:
        heights: m×n elevation grid

    Returns:
        List of [r, c] cells reachable by both oceans

    Time:  O(m × n)
    Space: O(m × n)
    """
    if not heights:
        return []

    m, n = len(heights), len(heights[0])
    DIRS = [(0,1),(0,-1),(1,0),(-1,0)]

    def bfs(seeds: list) -> list[list[bool]]:
        visited = [[False]*n for _ in range(m)]
        queue = deque(seeds)
        for r, c in seeds:
            visited[r][c] = True

        while queue:
            r, c = queue.popleft()
            for dr, dc in DIRS:
                nr, nc = r + dr, c + dc
                if 0 <= nr < m and 0 <= nc < n and not visited[nr][nc] \
                        and heights[nr][nc] >= heights[r][c]:
                    visited[nr][nc] = True
                    queue.append((nr, nc))

        return visited

    # Pacific: top row + left column
    pacific_seeds  = [(r, 0) for r in range(m)] + [(0, c) for c in range(n)]
    # Atlantic: bottom row + right column
    atlantic_seeds = [(r, n-1) for r in range(m)] + [(m-1, c) for c in range(n)]

    pacific  = bfs(pacific_seeds)
    atlantic = bfs(atlantic_seeds)

    return [[r, c] for r in range(m) for c in range(n)
            if pacific[r][c] and atlantic[r][c]]


if __name__ == "__main__":
    heights = [
        [1,2,2,3,5],
        [3,2,3,4,4],
        [2,4,5,3,1],
        [6,7,1,4,5],
        [5,1,1,2,4]
    ]
    print("Output:", pacific_atlantic(heights))
    # [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]

    print("\n1x1:", pacific_atlantic([[1]]))  # [[0, 0]]
    print("All same:", pacific_atlantic([[3,3],[3,3]]))  # all cells
```

### Output

```
Output: [[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]]

1x1: [[0, 0]]
All same: [[0, 0], [0, 1], [1, 0], [1, 1]]
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why reverse the direction (BFS from ocean inward)?**
A: Forward BFS from every cell costs O(m²n²). Reverse BFS from the border costs O(mn) — each cell is visited at most twice. The key insight is that "can water flow from (r,c) to Pacific" is equivalent to "can water flow from Pacific border to (r,c) going uphill."

**Q: Why do we check `heights[nr][nc] >= heights[r][c]` in the reverse BFS?**
A: In forward flow, water goes from high to low (≤). Reversed: water "flows" from low (ocean border) to high (inland). So we expand to neighbors with height ≥ current — meaning they could flow down to us.

**Q: What if there are no cells that reach both oceans?**
A: The result list remains empty. This is possible if, e.g., the grid is a perfectly isolated basin with no path to any border.

**Q: Can you use DFS instead of BFS?**
A: Yes, DFS works identically. Mark cells visited during DFS and collect the intersection afterward. BFS is often preferred for readability and avoids recursion depth issues.

**Q: What is the role of the border seed cells?**
A: They represent cells directly touching an ocean — water can trivially flow from them to that ocean. BFS expands "upstream," finding all cells from which water could flow down to these border cells.

**Q: How would you extend this to flow through obstacles?**
A: Add a condition to skip obstacle cells (e.g., `heights[nr][nc] == -1`) during BFS expansion.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                                    |
| --------------------- | ------------------------------------------------------------------------ |
| **Problem**           | Find cells from which water flows to both Pacific and Atlantic oceans     |
| **Optimal Algorithm** | Reverse BFS from both ocean borders simultaneously                        |
| **Time Complexity**   | O(m × n)                                                                 |
| **Space Complexity**  | O(m × n)                                                                 |
| **Key Insight**       | Reverse the flow: BFS uphill from ocean borders; intersection = answer   |

### Core Pattern

> Instead of asking "can this cell reach the ocean?", ask "can the ocean reach this cell going uphill?" Two reverse traversals + intersection = O(mn) solution instead of O(m²n²).

### When to Use This Pattern

- Any "can X reach both A and B?" graph problem where starting from both targets is cheaper.
- Related: Number of Islands, Surrounded Regions, Shortest Path in a Grid.

</details>
