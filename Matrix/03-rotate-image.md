# Rotate Image

---

1. What is Rotate Image?

## What is Rotate Image?

**Rotate Image** is an in-place matrix transformation problem where you are given:

- An `n × n` 2D matrix representing an image

**Goal:** Rotate the matrix **90 degrees clockwise**, **in place**.

**Constraints:**

- You must rotate the matrix in place — do not allocate another matrix.
- The matrix is always square (`n × n`).

**Example:**

```
Input:
[[1, 2, 3],
 [4, 5, 6],
 [7, 8, 9]]

Output:
[[7, 4, 1],
 [8, 5, 2],
 [9, 6, 3]]
```

```
Input:
[[ 5,  1,  9, 11],
 [ 2,  4,  8, 10],
 [13,  3,  6,  7],
 [15, 14, 12, 16]]

Output:
[[15, 13,  2,  5],
 [14,  3,  4,  1],
 [12,  6,  8,  9],
 [16,  7, 10, 11]]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Rotate an `n × n` integer matrix 90 degrees clockwise.
- Modification must be done in place — no extra matrix allowed.

### Non-Functional Requirements

| Concern         | Expectation                                     |
| --------------- | ----------------------------------------------- |
| **Scale**       | Matrix up to 20 × 20 (LeetCode constraint)      |
| **Latency**     | O(n²) is optimal — every cell must be touched   |
| **Space**       | O(1) extra space (in-place requirement)         |
| **Correctness** | Must handle n=1 (single cell, no-op)            |

### Clarifying Questions to Ask

- Is the matrix always square? → Yes (`n × n`).
- Are negative values possible? → Yes, rotation is value-agnostic.
- Is it always clockwise? → Yes (counter-clockwise would be: transpose + reverse each column).
- Can n be 1? → Yes, result is the same single cell.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Extra Matrix

|           | Complexity                        |
| --------- | --------------------------------- |
| **Time**  | O(n²) — copy every cell           |
| **Space** | O(n²) — allocates a full copy     |

### Approach 2: Transpose + Reverse Rows (Optimal)

|           | Complexity                                  |
| --------- | ------------------------------------------- |
| **Time**  | O(n²) — two full passes over the matrix     |
| **Space** | O(1) — only a single temp variable used     |

**Mathematical proof:**  
A 90° clockwise rotation maps cell `(i, j)` to `(j, n-1-i)`.  
Equivalently: **transpose** maps `(i, j) → (j, i)`, then **reversing each row** maps `(j, i) → (j, n-1-i)`.  
These two O(n²) passes compose to a 90° clockwise rotation.


---

4. Algorithm Choice and Why

## Algorithm: Transpose Then Reverse Rows

### Why This Algorithm?

| Option                     | Time  | Space  | Chosen?                              |
| -------------------------- | ----- | ------ | ------------------------------------ |
| Copy to new matrix         | O(n²) | O(n²)  | No — violates in-place requirement   |
| Four-cell ring rotation    | O(n²) | O(1)   | Valid but harder to implement        |
| **Transpose + reverse row**| O(n²) | **O(1)**| **Yes — cleanest two-step approach** |

### Step-by-Step Logic

**Step 1 — Transpose:** Swap `matrix[i][j]` with `matrix[j][i]` for all `i < j`.

```
[[1, 2, 3],      [[1, 4, 7],
 [4, 5, 6],  →   [2, 5, 8],
 [7, 8, 9]]       [3, 6, 9]]
```

**Step 2 — Reverse each row:**

```
[[1, 4, 7],      [[7, 4, 1],
 [2, 5, 8],  →   [8, 5, 2],
 [3, 6, 9]]       [9, 6, 3]]
```

Result is a 90° clockwise rotation.

### Why NOT counter-clockwise?

Counter-clockwise = **transpose + reverse each column** (or: reverse each row + transpose). Just swap the order/direction.


---

5. High-Level Design

## High-Level Design

### Components

1. **Transpose Engine** — Swaps `matrix[i][j]` and `matrix[j][i]` for the upper triangle.
2. **Row Reverser** — Reverses each row in place using two pointers.
3. **In-Place Coordinator** — Sequences transpose then reverse; no output structure needed.

### Data Flow Diagram

```
Original:
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]

After Transpose (swap [i][j] with [j][i] for i < j):
  swap(matrix[0][1], matrix[1][0]) → swap(2, 4)
  swap(matrix[0][2], matrix[2][0]) → swap(3, 7)
  swap(matrix[1][2], matrix[2][1]) → swap(6, 8)

Transposed:
[1, 4, 7]
[2, 5, 8]
[3, 6, 9]

After Reverse each row:
  [1,4,7] → [7,4,1]
  [2,5,8] → [8,5,2]
  [3,6,9] → [9,6,3]

Rotated 90° clockwise:
[7, 4, 1]
[8, 5, 2]
[9, 6, 3]
```


---

6. Java Solution

## Java Implementation

```java
public class RotateImage {

    /**
     * Rotates an n x n matrix 90 degrees clockwise in place.
     *
     * Step 1: Transpose — swap matrix[i][j] with matrix[j][i] for i < j.
     * Step 2: Reverse each row — two-pointer swap from ends toward center.
     *
     * @param matrix n x n integer matrix, modified in place
     */
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        // Step 1: Transpose — only upper triangle needs to be swapped
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }

        // Step 2: Reverse each row
        for (int i = 0; i < n; i++) {
            int left = 0, right = n - 1;
            while (left < right) {
                int temp = matrix[i][left];
                matrix[i][left] = matrix[i][right];
                matrix[i][right] = temp;
                left++;
                right--;
            }
        }
    }

    // ---- Helper: Print matrix ----
    private static void printMatrix(int[][] matrix) {
        for (int[] row : matrix) {
            System.out.print("[");
            for (int j = 0; j < row.length; j++) {
                System.out.printf("%3d%s", row[j], j < row.length - 1 ? "," : "");
            }
            System.out.println("]");
        }
    }

    public static void main(String[] args) {
        RotateImage solution = new RotateImage();

        // Example 1: 3x3
        int[][] m1 = {{1,2,3},{4,5,6},{7,8,9}};
        System.out.println("Input:");
        printMatrix(m1);
        solution.rotate(m1);
        System.out.println("Output (90° clockwise):");
        printMatrix(m1);

        System.out.println();

        // Example 2: 4x4
        int[][] m2 = {{5,1,9,11},{2,4,8,10},{13,3,6,7},{15,14,12,16}};
        System.out.println("Input:");
        printMatrix(m2);
        solution.rotate(m2);
        System.out.println("Output (90° clockwise):");
        printMatrix(m2);
    }
}
```

### Output

```
Input:
[  1,  2,  3]
[  4,  5,  6]
[  7,  8,  9]
Output (90° clockwise):
[  7,  4,  1]
[  8,  5,  2]
[  9,  6,  3]

Input:
[  5,  1,  9, 11]
[  2,  4,  8, 10]
[ 13,  3,  6,  7]
[ 15, 14, 12, 16]
Output (90° clockwise):
[ 15, 13,  2,  5]
[ 14,  3,  4,  1]
[ 12,  6,  8,  9]
[ 16,  7, 10, 11]
```


---

7. Python Solution

## Python Implementation

```python
def rotate(matrix: list[list[int]]) -> None:
    """
    Rotate an n x n matrix 90 degrees clockwise in place.

    Step 1: Transpose — swap matrix[i][j] and matrix[j][i] for i < j.
    Step 2: Reverse each row.

    Args:
        matrix: n x n list of lists, modified in place

    Time:  O(n²)
    Space: O(1)
    """
    n = len(matrix)

    # Step 1: Transpose (upper triangle only)
    for i in range(n):
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]

    # Step 2: Reverse each row
    for row in matrix:
        row.reverse()


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: 3x3
    m1 = [[1,2,3],[4,5,6],[7,8,9]]
    print("Input:")
    for row in m1: print(row)
    rotate(m1)
    print("Output (90° clockwise):")
    for row in m1: print(row)

    print()

    # Example 2: 4x4
    m2 = [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]]
    print("Input:")
    for row in m2: print(row)
    rotate(m2)
    print("Output (90° clockwise):")
    for row in m2: print(row)
```

### Output

```
Input:
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
Output (90° clockwise):
[7, 4, 1]
[8, 5, 2]
[9, 6, 3]

Input:
[5, 1, 9, 11]
[2, 4, 8, 10]
[13, 3, 6, 7]
[15, 14, 12, 16]
Output (90° clockwise):
[15, 13, 2, 5]
[14, 3, 4, 1]
[12, 6, 8, 9]
[16, 7, 10, 11]
```


---

8. Interview Q&A

## Q&A

**Q: Why does transpose followed by row reversal equal a 90° clockwise rotation?**  
A: The 90° clockwise mapping is `(i, j) → (j, n-1-i)`. Transpose gives `(i, j) → (j, i)`. Reversing each row maps column index `i → n-1-i`. Composing them produces `(i, j) → (j, n-1-i)`, which is exactly 90° clockwise.

**Q: How would you rotate 90° counter-clockwise instead?**  
A: Two options: (1) Transpose then reverse each **column**, or (2) reverse each row first, then transpose.

**Q: How would you rotate 180°?**  
A: Reverse each row, then reverse the row order of the matrix (i.e., flip vertically). Or equivalently apply 90° rotation twice.

**Q: What happens if n = 1?**  
A: The transpose loop doesn't execute (`j` starts at `i+1 = 1` which exceeds `n-1 = 0`). Reversing a single-element row is a no-op. Correct result.

**Q: Why not use the four-cell ring rotation approach?**  
A: The ring rotation approach (rotate four cells at a time: top→right→bottom→left→top) achieves O(1) space and O(n²) time, but the index arithmetic is error-prone in interviews. Transpose + reverse is equally efficient and far easier to derive and verify.


---

9. Summary

## Summary

| Attribute             | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Problem**           | Rotate an n × n matrix 90° clockwise in place                  |
| **Optimal Algorithm** | Transpose the matrix, then reverse each row                    |
| **Time Complexity**   | O(n²)                                                          |
| **Space Complexity**  | O(1)                                                           |
| **Key Insight**       | 90° CW rotation = transpose + row reversal (proven by mapping) |

### Core Pattern

> Decompose the rotation into two simpler operations (transpose + reverse) each of which is trivially correct and O(n²) time, O(1) space.

### When to Use This Pattern

- Any in-place matrix rotation problem.
- Image processing where 90°/180°/270° rotations are needed without extra memory.

