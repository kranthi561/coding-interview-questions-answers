# Set Matrix Zeroes

---

1. What is Set Matrix Zeroes?

## What is Set Matrix Zeroes?

**Set Matrix Zeroes** is an in-place matrix manipulation problem where you are given:

- An `m × n` integer matrix

**Goal:** If any element is `0`, set its **entire row** and **entire column** to `0`, **in place**.

**Constraints:**

- Must be done in-place (do not use a copy of the matrix).
- O(1) extra space is the optimal target.

**Example:**

```
Input:
[[1, 1, 1],
 [1, 0, 1],
 [1, 1, 1]]

Output:
[[1, 0, 1],
 [0, 0, 0],
 [1, 0, 1]]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Given an `m × n` matrix, locate every cell that contains `0`.
- Set the entire row and column of each such cell to `0`.
- Modification must happen on the same matrix (in-place).

### Non-Functional Requirements

| Concern         | Expectation                                              |
| --------------- | -------------------------------------------------------- |
| **Scale**       | Matrix up to 200 × 200 (40,000 elements)                 |
| **Latency**     | Sub-millisecond for typical sizes                        |
| **Space**       | Ideally O(1) extra space; O(m+n) is acceptable           |
| **Correctness** | Must not use a zero propagated in this pass as a trigger |

### Clarifying Questions to Ask

- Can we modify the matrix in place? → Yes, required.
- Are there multiple zeros? → Yes, handle all of them.
- Does the order of zeroing matter? → No, final state is what matters.
- Is it guaranteed the matrix is non-empty? → Yes.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Extra Space (Set of rows/columns)

|           | Complexity                               |
| --------- | ---------------------------------------- |
| **Time**  | O(m × n) — scan entire matrix twice      |
| **Space** | O(m + n) — store which rows/cols are zero |

### Approach 2: In-Place Using First Row/Column as Markers (Optimal)

|           | Complexity                                              |
| --------- | ------------------------------------------------------- |
| **Time**  | O(m × n) — three passes over the matrix                 |
| **Space** | O(1) — only two boolean flags for first row/col state   |

**Why O(1) space?** We repurpose the first row and first column of the matrix itself as a marker array, avoiding any extra data structure.


---

4. Algorithm Choice and Why

## Algorithm: In-Place First Row/Column Markers

### Why This Algorithm?

| Option                         | Time     | Space    | Chosen?                        |
| ------------------------------ | -------- | -------- | ------------------------------ |
| Brute Force (copy matrix)      | O(m × n) | O(m × n) | No — uses too much space        |
| Extra set for rows/cols        | O(m × n) | O(m + n) | No — violates O(1) space goal  |
| **First row/col as markers**   | O(m × n) | **O(1)** | **Yes**                        |

### Logic

1. **Pass 1:** Check whether the first row or first column contains any zero. Record this in two boolean flags.
2. **Pass 2:** For every cell `(i, j)` where `i > 0` and `j > 0`, if `matrix[i][j] == 0`, mark `matrix[i][0] = 0` and `matrix[0][j] = 0`.
3. **Pass 3:** For each cell `(i, j)` where `i > 0` and `j > 0`, if `matrix[i][0] == 0` or `matrix[0][j] == 0`, set `matrix[i][j] = 0`.
4. **Pass 4:** Use the flags from step 1 to zero out the first row and/or first column if needed.

This avoids overwriting markers before they are read.


---

5. High-Level Design

## High-Level Design

### Components

1. **First Row/Column Scanner** — Checks if row 0 or column 0 originally has a zero.
2. **Marker Writer** — Uses row 0 and column 0 to record which rows/columns need zeroing.
3. **Zero Applier** — Uses the markers to zero interior cells.
4. **Boundary Zeroing** — Zeros row 0 and column 0 based on the saved flags.

### Data Flow Diagram

```
Original Matrix:
[[1, 1, 1],
 [1, 0, 1],
 [1, 1, 1]]

Step 1 — Scan first row/col for zeros:
  firstRowHasZero = false
  firstColHasZero = false

Step 2 — Mark using row 0 / col 0:
  matrix[1][1] = 0 → mark matrix[1][0] = 0, matrix[0][1] = 0

After marking:
[[1, 0, 1],
 [0, 0, 1],
 [1, 1, 1]]

Step 3 — Apply zeros to interior:
  matrix[1][0] = 0 → zero row 1
  matrix[0][1] = 0 → zero col 1

Final:
[[1, 0, 1],
 [0, 0, 0],
 [1, 0, 1]]
```


---

6. Java Solution

## Java Implementation

```java
public class SetMatrixZeroes {

    /**
     * Sets entire rows and columns to zero wherever a zero is found.
     * Uses the first row and column as in-place markers — O(1) extra space.
     *
     * @param matrix m x n integer matrix, modified in place
     */
    public void setZeroes(int[][] matrix) {
        int rows = matrix.length;
        int cols = matrix[0].length;

        // Step 1: Record whether the first row or first column originally has a zero
        boolean firstRowHasZero = false;
        boolean firstColHasZero = false;

        for (int j = 0; j < cols; j++) {
            if (matrix[0][j] == 0) {
                firstRowHasZero = true;
                break;
            }
        }
        for (int i = 0; i < rows; i++) {
            if (matrix[i][0] == 0) {
                firstColHasZero = true;
                break;
            }
        }

        // Step 2: Use first row/col as markers for interior cells
        for (int i = 1; i < rows; i++) {
            for (int j = 1; j < cols; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = 0; // mark row
                    matrix[0][j] = 0; // mark column
                }
            }
        }

        // Step 3: Zero out interior cells based on markers
        for (int i = 1; i < rows; i++) {
            for (int j = 1; j < cols; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }

        // Step 4: Zero out first row if it originally had a zero
        if (firstRowHasZero) {
            for (int j = 0; j < cols; j++) {
                matrix[0][j] = 0;
            }
        }

        // Step 5: Zero out first column if it originally had a zero
        if (firstColHasZero) {
            for (int i = 0; i < rows; i++) {
                matrix[i][0] = 0;
            }
        }
    }

    // ---- Helper: Print matrix ----
    private static void printMatrix(int[][] matrix) {
        for (int[] row : matrix) {
            System.out.print("[");
            for (int j = 0; j < row.length; j++) {
                System.out.print(row[j] + (j < row.length - 1 ? ", " : ""));
            }
            System.out.println("]");
        }
    }

    public static void main(String[] args) {
        SetMatrixZeroes solution = new SetMatrixZeroes();

        // Example 1
        int[][] m1 = {{1,1,1},{1,0,1},{1,1,1}};
        System.out.println("Input:");
        printMatrix(m1);
        solution.setZeroes(m1);
        System.out.println("Output:");
        printMatrix(m1);

        System.out.println();

        // Example 2
        int[][] m2 = {{0,1,2,0},{3,4,5,2},{1,3,1,5}};
        System.out.println("Input:");
        printMatrix(m2);
        solution.setZeroes(m2);
        System.out.println("Output:");
        printMatrix(m2);
    }
}
```

### Output

```
Input:
[1, 1, 1]
[1, 0, 1]
[1, 1, 1]
Output:
[1, 0, 1]
[0, 0, 0]
[1, 0, 1]

Input:
[0, 1, 2, 0]
[3, 4, 5, 2]
[1, 3, 1, 5]
Output:
[0, 0, 0, 0]
[0, 4, 5, 0]
[0, 3, 1, 0]
```


---

7. Python Solution

## Python Implementation

```python
def set_zeroes(matrix: list[list[int]]) -> None:
    """
    Set entire row and column to zero wherever a zero is found.
    In-place using first row/column as markers — O(1) extra space.

    Args:
        matrix: m x n list of lists, modified in place

    Time:  O(m × n)
    Space: O(1)
    """
    rows, cols = len(matrix), len(matrix[0])

    # Step 1: Check if first row or first column originally contains a zero
    first_row_has_zero = any(matrix[0][j] == 0 for j in range(cols))
    first_col_has_zero = any(matrix[i][0] == 0 for i in range(rows))

    # Step 2: Use row 0 and col 0 as markers for interior cells
    for i in range(1, rows):
        for j in range(1, cols):
            if matrix[i][j] == 0:
                matrix[i][0] = 0  # mark the row
                matrix[0][j] = 0  # mark the column

    # Step 3: Zero out interior cells using the markers
    for i in range(1, rows):
        for j in range(1, cols):
            if matrix[i][0] == 0 or matrix[0][j] == 0:
                matrix[i][j] = 0

    # Step 4: Zero out first row if it originally had a zero
    if first_row_has_zero:
        for j in range(cols):
            matrix[0][j] = 0

    # Step 5: Zero out first column if it originally had a zero
    if first_col_has_zero:
        for i in range(rows):
            matrix[i][0] = 0


# ---- Example usage ----
if __name__ == "__main__":
    import copy

    # Example 1
    m1 = [[1,1,1],[1,0,1],[1,1,1]]
    print("Input:", m1)
    set_zeroes(m1)
    print("Output:", m1)

    # Example 2
    m2 = [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
    print("\nInput:", m2)
    set_zeroes(m2)
    print("Output:", m2)
```

### Output

```
Input: [[1, 1, 1], [1, 0, 1], [1, 1, 1]]
Output: [[1, 0, 1], [0, 0, 0], [1, 0, 1]]

Input: [[0, 1, 2, 0], [3, 4, 5, 2], [1, 3, 1, 5]]
Output: [[0, 0, 0, 0], [0, 4, 5, 0], [0, 3, 1, 0]]
```


---

8. Interview Q&A

## Q&A

**Q: Why can't we just zero cells as we find them in the first pass?**  
A: Zeroing during the scan would corrupt the markers. A cell set to zero in the current pass would incorrectly trigger more rows/columns to be zeroed during the same pass.

**Q: Why use the first row and first column as markers?**  
A: They act as a free O(m+n) marker array embedded inside the matrix itself. We just need two extra booleans to preserve whether those boundary structures originally had zeros.

**Q: What if multiple cells in the same row are zero?**  
A: The marker `matrix[i][0] = 0` is set regardless of how many zeros are in row `i`. The final pass reads it once and zeros the entire row — duplicates don't matter.

**Q: What is the space complexity of the O(m+n) approach?**  
A: It uses two sets — one for zero-rows, one for zero-columns — occupying O(m + n) space. Acceptable if the interviewer doesn't require O(1).

**Q: Can we solve this with a single extra boolean instead of two?**  
A: Only if we handle one boundary (e.g., the first column) with its boolean and use the first row for the other dimension's markers. The two-boolean approach shown here is the cleanest O(1) solution.


---

9. Summary

## Summary

| Attribute             | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Problem**           | Zero entire row and column for each zero cell, in place           |
| **Optimal Algorithm** | First row/column as in-place markers                              |
| **Time Complexity**   | O(m × n)                                                          |
| **Space Complexity**  | O(1)                                                              |
| **Key Insight**       | Repurpose matrix boundaries as markers; protect them with flags   |

### Core Pattern

> Use the matrix's own first row and column as a zero-indicator array — scan first, mark second, apply third, fix boundaries last.

### When to Use This Pattern

- In-place matrix problems where extra space is restricted.
- Any "mark then apply" problem where the marker set is bounded by matrix dimensions.

