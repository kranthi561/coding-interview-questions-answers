# Unique Paths

---

<details>
<summary><strong>1. What is meant by Unique Paths?</strong></summary>

## What is Unique Paths?

**Unique Paths** is a classic DP/combinatorics problem where:

- You have an `m × n` grid.
- A robot starts at the top-left corner `(0, 0)`.
- The robot can only move **right** or **down** at each step.

**Goal:** Return the number of **unique paths** from the top-left to the bottom-right corner `(m-1, n-1)`.

**Constraints:**
- `1 <= m, n <= 100`

**Example:**

```
Input:  m = 3, n = 7
Output: 28

Input:  m = 3, n = 2
Output: 3
Explanation:
  → ↓     (right then down twice)
  ↓ → ↓
  ↓ ↓ →
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Count all distinct paths from top-left to bottom-right.
- Movement restricted to right or down only.
- Every cell in the path is visited exactly once (no backtracking).

### Non-Functional Requirements

| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | m, n up to 100 — O(m×n) trivially fast                |
| **Latency**     | O(m×n) — runs in microseconds                         |
| **Correctness** | Must count all unique paths precisely                 |
| **Space**       | Reducible to O(n) with single-row DP                  |

### Clarifying Questions to Ask

- Can the robot move left or up? → No, only right or down.
- Are there obstacles? → No (that's Unique Paths II).
- What if m=1 or n=1? → Only one path (a straight line).
- Return count or actual paths? → Count only.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force Recursion

| | Complexity |
| --- | --- |
| **Time** | O(2^(m+n)) — exponential |
| **Space** | O(m+n) — call stack depth |

### Approach 2: Memoization (Top-Down DP)

| | Complexity |
| --- | --- |
| **Time** | O(m × n) — each cell computed once |
| **Space** | O(m × n) — memo table + call stack |

### Approach 3: Bottom-Up DP

| | Complexity |
| --- | --- |
| **Time** | O(m × n) |
| **Space** | O(m × n) — full 2D table |

### Approach 4: Space-Optimized DP (Single Row)

| | Complexity |
| --- | --- |
| **Time** | O(m × n) |
| **Space** | O(n) — single row |

### Approach 5: Math (Combinatorics)

| | Complexity |
| --- | --- |
| **Time** | O(min(m,n)) — compute combination |
| **Space** | O(1) |

**Math insight:** To reach bottom-right from top-left, you must make exactly `(m-1)` down moves and `(n-1)` right moves. Total moves = `m+n-2`. Choose which moves are "down": `C(m+n-2, m-1)`.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Bottom-Up DP (or Combinatorics)

### Comparison

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2^(m+n)) | O(m+n) | No |
| Top-Down Memo | O(m×n) | O(m×n) | Acceptable |
| **Bottom-Up DP** | **O(m×n)** | **O(n)** | **Yes — standard** |
| Math (nCr) | O(min(m,n)) | O(1) | Yes for interviews |

### Key Recurrence

`dp[i][j]` = paths from `(0,0)` to `(i,j)`:

```
dp[i][j] = dp[i-1][j] + dp[i][j-1]    (from above + from left)

Base cases:
  dp[0][j] = 1  (top row: only one way — go all right)
  dp[i][0] = 1  (left column: only one way — go all down)
```

### Math Shortcut (Interview Favorite)

```
Unique paths = C(m+n-2, m-1) = (m+n-2)! / ((m-1)! × (n-1)!)
```

For m=3, n=7: `C(8, 2) = 28`.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **DP Table** — 2D array `dp[m][n]`, all initialized to 1 (base case)
2. **Row/Column Base Cases** — First row and column all set to 1
3. **Inner Cell Filler** — `dp[i][j] = dp[i-1][j] + dp[i][j-1]`
4. **Output Layer** — Returns `dp[m-1][n-1]`

### Data Flow Diagram (m=3, n=3)

```
Initial (base cases):
  1  1  1
  1  0  0
  1  0  0

After filling:
  1  1  1
  1  2  3
  1  3  6

Output: dp[2][2] = 6
```

### Visual Paths (m=3, n=2)

```
Grid (3 rows, 2 cols):
  S → ·
  ↓   ↓
  · → ·
  ↓   ↓
  · → E

Path 1: → ↓ ↓
Path 2: ↓ → ↓
Path 3: ↓ ↓ →
Total: 3
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class UniquePaths {

    /**
     * Count unique paths from top-left to bottom-right of an m×n grid.
     * Movement: right or down only.
     *
     * DP: dp[i][j] = paths from above + paths from left.
     * First row and column are all 1 (only one way to reach them).
     *
     * @param m number of rows
     * @param n number of columns
     * @return number of unique paths
     *
     * Time:  O(m × n)
     * Space: O(m × n) — reducible to O(n)
     */
    public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];

        // Base case: only one way to reach any cell in the first row or column
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;

        // Fill remaining cells
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]; // from above + from left
            }
        }

        return dp[m - 1][n - 1];
    }

    /**
     * Space-optimized version using a single row.
     * O(n) space — overwrite in-place from left to right.
     */
    public int uniquePathsOptimized(int m, int n) {
        int[] dp = new int[n];
        java.util.Arrays.fill(dp, 1); // first row: all 1s

        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[j] += dp[j - 1]; // dp[j] (from above) + dp[j-1] (from left)
            }
        }

        return dp[n - 1];
    }

    /**
     * Math solution using combinatorics: C(m+n-2, m-1).
     * Total moves = m+n-2; choose m-1 of them to be downward.
     */
    public int uniquePathsMath(int m, int n) {
        long result = 1;
        for (int i = 0; i < Math.min(m - 1, n - 1); i++) {
            result = result * (m + n - 2 - i) / (i + 1);
        }
        return (int) result;
    }

    public static void main(String[] args) {
        UniquePaths sol = new UniquePaths();

        // Example 1
        System.out.println("m=3, n=7 → " + sol.uniquePaths(3, 7));         // 28
        System.out.println("m=3, n=7 → " + sol.uniquePathsOptimized(3, 7)); // 28
        System.out.println("m=3, n=7 → " + sol.uniquePathsMath(3, 7));      // 28

        // Example 2
        System.out.println("\nm=3, n=2 → " + sol.uniquePaths(3, 2));        // 3

        // Example 3: Single row
        System.out.println("\nm=1, n=10 → " + sol.uniquePaths(1, 10));      // 1

        // Example 4: Single column
        System.out.println("\nm=10, n=1 → " + sol.uniquePaths(10, 1));      // 1

        // Example 5: Square
        System.out.println("\nm=4, n=4 → " + sol.uniquePaths(4, 4));        // 20
    }
}
```

### Output

```
m=3, n=7 → 28
m=3, n=7 → 28
m=3, n=7 → 28

m=3, n=2 → 3

m=1, n=10 → 1

m=10, n=1 → 1

m=4, n=4 → 20
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
import math

def unique_paths(m: int, n: int) -> int:
    """
    Count unique paths from top-left to bottom-right of an m×n grid.
    Only right and down moves allowed.

    Strategy: Bottom-up DP.
    dp[i][j] = dp[i-1][j] + dp[i][j-1]
    First row and column are all 1 (base case — only one path to reach them).

    Args:
        m: Number of rows
        n: Number of columns

    Returns:
        Number of unique paths

    Time:  O(m × n)
    Space: O(m × n)
    """
    dp = [[1] * n for _ in range(m)]  # Initialize all to 1 (handles base cases)

    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1]

    return dp[m - 1][n - 1]


def unique_paths_optimized(m: int, n: int) -> int:
    """Space-optimized: O(n) using a single row."""
    dp = [1] * n

    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]  # accumulate from left (this row) and above (prev value)

    return dp[n - 1]


def unique_paths_math(m: int, n: int) -> int:
    """Math solution: C(m+n-2, m-1) — O(min(m,n)) time, O(1) space."""
    return math.comb(m + n - 2, m - 1)


if __name__ == "__main__":
    test_cases = [
        (3, 7,  28),
        (3, 2,  3),
        (1, 10, 1),
        (10, 1, 1),
        (4, 4,  20),
    ]

    for m, n, expected in test_cases:
        dp_result   = unique_paths(m, n)
        opt_result  = unique_paths_optimized(m, n)
        math_result = unique_paths_math(m, n)
        print(f"m={m}, n={n} → DP: {dp_result}, Opt: {opt_result}, Math: {math_result}  (Expected: {expected})")
```

### Output

```
m=3, n=7 → DP: 28, Opt: 28, Math: 28  (Expected: 28)
m=3, n=2 → DP: 3, Opt: 3, Math: 3  (Expected: 3)
m=1, n=10 → DP: 1, Opt: 1, Math: 1  (Expected: 1)
m=10, n=1 → DP: 1, Opt: 1, Math: 1  (Expected: 1)
m=4, n=4 → DP: 20, Opt: 20, Math: 20  (Expected: 20)
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why is `dp[0][j] = 1` and `dp[i][0] = 1`?**
A: The top row can only be reached by moving right repeatedly — one way. The left column can only be reached by moving down — one way. These are the base cases.

**Q: What's the math formula and why does it work?**
A: To go from top-left to bottom-right in an m×n grid, you need exactly `(m-1)` down moves and `(n-1)` right moves, for a total of `m+n-2` moves. Choose which `m-1` of those moves are "down": `C(m+n-2, m-1)`.

**Q: How does Unique Paths II (with obstacles) differ?**
A: Cells with obstacles set `dp[i][j] = 0`. The recurrence is the same for non-obstacle cells, but you skip the addition when a cell is blocked.

**Q: Why does the space-optimized O(n) version work?**
A: We process row by row. `dp[j]` (before update) holds the value from the row above, and `dp[j-1]` (just updated) holds the value from the left in the current row. Updating left-to-right naturally preserves this.

**Q: Can you solve this with just O(1) space?**
A: Only with the math formula: `C(m+n-2, m-1)`. The DP approach inherently needs at least one row.

**Q: What's the maximum possible answer for m=n=100?**
A: `C(198, 99)` — an astronomically large number that fits in a 64-bit integer (Java `long` or Python `int`).

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Count unique paths in an m×n grid (only right/down moves)         |
| **Optimal Algorithm** | Bottom-Up DP or Combinatorics (C(m+n-2, m-1))                     |
| **Time Complexity**   | O(m×n) for DP; O(min(m,n)) for math                               |
| **Space Complexity**  | O(n) for optimized DP; O(1) for math                              |
| **Key Insight**       | `dp[i][j] = dp[i-1][j] + dp[i][j-1]` — paths from above + left  |

### Core Pattern

> Grid DP where each cell's answer is the sum of answers from reachable neighbors. Initialize the edges to 1 (boundary paths), then fill inward. The bottom-right corner holds the answer.

### When to Use This Pattern

- Any "count paths in a grid" problem with restricted movement.
- Related: Unique Paths II (obstacles), Minimum Path Sum, Triangle, Dungeon Game.

</details>
