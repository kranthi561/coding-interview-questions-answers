# Number of Islands

---

<details>
<summary><strong>1. What is meant by Number of Islands?</strong></summary>

## What is Number of Islands?

**Number of Islands** is a classic graph flood-fill problem where:

- You are given an `m × n` 2D binary grid.
- `'1'` represents land, `'0'` represents water.
- An **island** is a maximal group of `'1'`s connected horizontally or vertically.

**Goal:** Return the **number of islands** in the grid.

**Constraints:**
- `1 <= m, n <= 300`
- `grid[i][j]` is either `'0'` or `'1'`.

**Example:**

```
Input:
grid = [["1","1","0","0","0"],
        ["1","1","0","0","0"],
        ["0","0","1","0","0"],
        ["0","0","0","1","1"]]

Output: 3
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Count distinct connected components of `'1'` cells.
- Connectivity is horizontal and vertical only (4-directional, no diagonals).
- An island must be surrounded by water or the grid boundary.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | Up to 300×300 = 90,000 cells — O(mn) is fast       |
| **Latency**     | O(m × n) — single pass with flood fills             |
| **Correctness** | Must count each connected group exactly once        |
| **Mutability**  | Modifying the grid is acceptable (mark visited)     |

### Clarifying Questions to Ask

- Is diagonal adjacency counted? → No, only 4 directions.
- Can we modify the input grid? → Yes (mark visited cells as '0').
- Can the grid be empty? → Constraints say m,n ≥ 1.
- Are there other values in the grid? → Only '0' and '1'.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### DFS / BFS Flood Fill

| | Complexity |
| --- | --- |
| **Time** | O(m × n) — each cell visited at most once |
| **Space** | O(m × n) — recursion stack (DFS) or queue (BFS) in worst case (all land) |

### Union-Find

| | Complexity |
| --- | --- |
| **Time** | O(m × n × α(mn)) ≈ O(mn) — nearly linear with path compression |
| **Space** | O(m × n) — parent and rank arrays |

**DFS is standard** for simplicity. Union-Find is useful when cells are added dynamically (stream of updates).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: DFS Flood Fill

### Why DFS?

| Option | Time | Space | Notes | Chosen? |
| --- | --- | --- | --- | --- |
| **DFS Flood Fill** | **O(mn)** | **O(mn)** | Simple, in-place | **Yes** |
| BFS Flood Fill | O(mn) | O(min(m,n)) | Iterative, smaller queue | Equivalent |
| Union-Find | O(mn·α) | O(mn) | Good for dynamic updates | No (overkill here) |

### Key Insight

For every unvisited `'1'` cell, increment the island count and flood-fill the entire island by recursively marking all connected `'1'` cells as visited (`'0'` or a separate boolean).

This ensures each island is counted exactly once and each cell is processed at most once.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Grid Scanner** — Iterate over all cells looking for unvisited `'1'`.
2. **Island Counter** — Increment count when a new island is found.
3. **DFS Flood Fill** — Mark all connected `'1'` cells as visited.
4. **Boundary Guard** — Skip cells outside the grid or already visited.
5. **Output** — Return the island count.

### Data Flow Diagram

```
Grid:
  1 1 0 0 0
  1 1 0 0 0
  0 0 1 0 0
  0 0 0 1 1

Scan row by row:

[0,0] = '1' → count=1, DFS floods: [0,0],[0,1],[1,0],[1,1] → mark as '0'
[0,2..4] = '0', skip
[1,0],[1,1] = '0' (already flooded), skip
[2,2] = '1' → count=2, DFS floods: [2,2] only → mark as '0'
[3,3] = '1' → count=3, DFS floods: [3,3],[3,4] → mark as '0'

Output: 3
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class NumberOfIslands {

    /**
     * Count the number of islands in a 2D binary grid.
     *
     * DFS flood fill: for every unvisited '1', increment island count
     * and recursively mark all connected '1' cells as '0' (visited).
     *
     * @param grid 2D char array ('0' = water, '1' = land)
     * @return number of distinct islands
     *
     * Time:  O(m × n) — each cell visited at most once
     * Space: O(m × n) — recursion stack in worst case (all land)
     */
    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) return 0;

        int count = 0;
        for (int r = 0; r < grid.length; r++) {
            for (int c = 0; c < grid[0].length; c++) {
                if (grid[r][c] == '1') {
                    count++;            // found a new island
                    dfs(grid, r, c);   // flood-fill to mark it entirely visited
                }
            }
        }
        return count;
    }

    // Recursively mark all connected '1' cells as '0'
    private void dfs(char[][] grid, int r, int c) {
        // Out of bounds or already water/visited → stop
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length || grid[r][c] != '1') {
            return;
        }

        grid[r][c] = '0'; // mark as visited

        dfs(grid, r + 1, c); // down
        dfs(grid, r - 1, c); // up
        dfs(grid, r, c + 1); // right
        dfs(grid, r, c - 1); // left
    }

    // BFS version — avoids deep recursion stack
    public int numIslands_BFS(char[][] grid) {
        if (grid == null || grid.length == 0) return 0;
        int m = grid.length, n = grid[0].length, count = 0;
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        java.util.Queue<int[]> queue = new java.util.LinkedList<>();

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == '1') {
                    count++;
                    grid[r][c] = '0';
                    queue.offer(new int[]{r, c});
                    while (!queue.isEmpty()) {
                        int[] cur = queue.poll();
                        for (int[] d : dirs) {
                            int nr = cur[0]+d[0], nc = cur[1]+d[1];
                            if (nr>=0 && nr<m && nc>=0 && nc<n && grid[nr][nc]=='1') {
                                grid[nr][nc] = '0';
                                queue.offer(new int[]{nr, nc});
                            }
                        }
                    }
                }
            }
        }
        return count;
    }

    public static void main(String[] args) {
        NumberOfIslands sol = new NumberOfIslands();

        // Example 1: 3 islands
        char[][] grid1 = {
            {'1','1','0','0','0'},
            {'1','1','0','0','0'},
            {'0','0','1','0','0'},
            {'0','0','0','1','1'}
        };
        System.out.println("Example 1: " + sol.numIslands(grid1)); // 3

        // Example 2: 1 large island
        char[][] grid2 = {
            {'1','1','1'},
            {'1','0','1'},
            {'1','1','1'}
        };
        System.out.println("Example 2: " + sol.numIslands(grid2)); // 1

        // Example 3: All water
        char[][] grid3 = {{'0','0'},{'0','0'}};
        System.out.println("Example 3: " + sol.numIslands(grid3)); // 0

        // Example 4: All land
        char[][] grid4 = {{'1','1'},{'1','1'}};
        System.out.println("Example 4: " + sol.numIslands(grid4)); // 1
    }
}
```

### Output

```
Example 1: 3
Example 2: 1
Example 3: 0
Example 4: 1
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
from collections import deque

def num_islands(grid: list[list[str]]) -> int:
    """
    Count the number of islands in a 2D binary grid.

    Strategy: DFS flood fill.
    For each unvisited '1', increment count and flood the entire island
    by recursively/iteratively marking connected '1' cells as '0'.

    Args:
        grid: 2D list of '0' and '1' characters (modified in-place)

    Returns:
        Number of distinct islands

    Time:  O(m × n)
    Space: O(m × n)
    """
    if not grid:
        return 0

    m, n = len(grid), len(grid[0])

    def dfs(r: int, c: int) -> None:
        if r < 0 or r >= m or c < 0 or c >= n or grid[r][c] != '1':
            return
        grid[r][c] = '0'  # mark visited
        dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1)

    count = 0
    for r in range(m):
        for c in range(n):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c)

    return count


def num_islands_bfs(grid: list[list[str]]) -> int:
    """BFS version — avoids recursion stack limit issues."""
    if not grid:
        return 0

    m, n = len(grid), len(grid[0])
    DIRS = [(1,0),(-1,0),(0,1),(0,-1)]
    count = 0

    for r in range(m):
        for c in range(n):
            if grid[r][c] == '1':
                count += 1
                grid[r][c] = '0'
                queue = deque([(r, c)])
                while queue:
                    cr, cc = queue.popleft()
                    for dr, dc in DIRS:
                        nr, nc = cr+dr, cc+dc
                        if 0<=nr<m and 0<=nc<n and grid[nr][nc]=='1':
                            grid[nr][nc] = '0'
                            queue.append((nr, nc))
    return count


if __name__ == "__main__":
    import copy

    test_cases = [
        ([["1","1","0","0","0"],
          ["1","1","0","0","0"],
          ["0","0","1","0","0"],
          ["0","0","0","1","1"]], 3),
        ([["1","1","1"],
          ["1","0","1"],
          ["1","1","1"]], 1),
        ([["0","0"],["0","0"]], 0),
        ([["1","1"],["1","1"]], 1),
    ]

    for grid, expected in test_cases:
        result = num_islands(copy.deepcopy(grid))
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] Output: 3, Expected: 3
[PASS] Output: 1, Expected: 1
[PASS] Output: 0, Expected: 0
[PASS] Output: 1, Expected: 1
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why do we mark cells as '0' during DFS instead of using a separate visited array?**
A: Modifying the grid avoids the extra O(mn) space of a visited array. It's acceptable when the problem allows mutation (most interview problems do). If mutation is not allowed, use a separate `boolean[][]`.

**Q: Does DFS or BFS perform better here?**
A: Both are O(mn). BFS uses a queue and avoids deep recursion (better for very large all-land grids where DFS stack depth could exceed limits). DFS is more concise.

**Q: What if islands are also diagonally connected?**
A: Add 4 more directions: `(-1,-1), (-1,+1), (+1,-1), (+1,+1)`. The algorithm is identical — just expand to 8 neighbors.

**Q: How would you count islands in a stream of grid updates?**
A: Use Union-Find (Disjoint Set Union). For each new '1', create a node; union it with adjacent '1' neighbors. Track the number of components after each update. DFS is not efficient for dynamic updates.

**Q: What's the maximum recursion depth for DFS?**
A: In the worst case (all cells are '1'), DFS visits all m×n cells. In Python, this can hit the default recursion limit (~1000). Use BFS or increase `sys.setrecursionlimit` for large grids.

**Q: How is this related to connected components in a graph?**
A: Each island is a connected component in the implicit graph where land cells are nodes and adjacency defines edges. Counting islands = counting connected components.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Problem**           | Count distinct connected groups of '1' cells in a binary grid  |
| **Optimal Algorithm** | DFS or BFS Flood Fill                                          |
| **Time Complexity**   | O(m × n)                                                       |
| **Space Complexity**  | O(m × n) — recursion/queue in worst case                      |
| **Key Insight**       | Each unvisited '1' starts a new island; flood-fill marks it all visited |

### Core Pattern

> Scan the grid. When you find an unvisited land cell, increment the count and use DFS/BFS to mark the entire island as visited. Repeat until all cells are processed.

### When to Use This Pattern

- Counting connected components in a grid.
- Related: Max Area of Island, Surrounded Regions, Pacific Atlantic Water Flow, Word Search.

</details>
