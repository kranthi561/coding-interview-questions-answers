# Spiral Matrix

---

1. What is Spiral Matrix?

## What is Spiral Matrix?

**Spiral Matrix** is a matrix traversal problem where you are given:

- An `m × n` matrix of integers

**Goal:** Return all elements of the matrix in **spiral (clockwise) order**, starting from the top-left corner.

**Constraints:**

- Traverse: right → down → left → up, repeating inward.
- Output must list every element exactly once.

**Example:**

```
Input:
[[1,  2,  3],
 [4,  5,  6],
 [7,  8,  9]]

Output: [1, 2, 3, 6, 9, 8, 7, 4, 5]
```

```
Input:
[[1,  2,  3,  4],
 [5,  6,  7,  8],
 [9, 10, 11, 12]]

Output: [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Traverse the matrix in clockwise spiral order starting from the top-left.
- Return a flat list of all elements in traversal order.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Matrix up to 10 × 10 (LeetCode), up to 200 × 200    |
| **Latency**     | O(m × n) is required — every element visited once   |
| **Correctness** | Handle non-square matrices and 1-row/1-col edge cases |

### Clarifying Questions to Ask

- Is the matrix always non-empty? → Yes.
- Can the matrix be a single row or single column? → Yes, handle those.
- Is the matrix square? → Not necessarily.
- Do we need to handle negative values? → Yes, just traverse them.


---

3. Time and Space Complexity

## Complexity Analysis

### Boundary-Shrinking Approach (Optimal)

|           | Complexity                                       |
| --------- | ------------------------------------------------ |
| **Time**  | O(m × n) — every element is visited exactly once |
| **Space** | O(m × n) — output list stores all elements       |

**Note:** The O(m × n) space is for the output list itself. No additional data structures are used beyond four integer boundary variables — the algorithm's working space is O(1).


---

4. Algorithm Choice and Why

## Algorithm: Layer-by-Layer Boundary Shrinking

### Why This Algorithm?

| Option                         | Time     | Space    | Chosen?                          |
| ------------------------------ | -------- | -------- | -------------------------------- |
| Visited array + direction array| O(m × n) | O(m × n) | No — wastes space on visited set |
| **Boundary shrinking**         | O(m × n) | **O(1)** | **Yes**                          |

### Logic

Maintain four pointers:
- `top`, `bottom` — current row bounds
- `left`, `right` — current column bounds

Each iteration:
1. Traverse left → right along `top`, then increment `top`.
2. Traverse top → bottom along `right`, then decrement `right`.
3. Traverse right → left along `bottom` (if `top <= bottom`), then decrement `bottom`.
4. Traverse bottom → top along `left` (if `left <= right`), then increment `left`.

Stop when `top > bottom` or `left > right`.

The boundary checks on steps 3 and 4 prevent re-traversing a row or column when the spiral has collapsed to a single row or column.


---

5. High-Level Design

## High-Level Design

### Components

1. **Boundary Manager** — Tracks `top`, `bottom`, `left`, `right`.
2. **Direction Traverser** — Executes each of the four directional sweeps.
3. **Boundary Shrinker** — Advances each boundary pointer after completing that side.
4. **Output Collector** — Appends each visited value to the result list.

### Data Flow Diagram

```
Matrix:
[[1, 2, 3],
 [4, 5, 6],
 [7, 8, 9]]

Initial: top=0, bottom=2, left=0, right=2

Iteration 1:
  → Right along row 0:    [1, 2, 3]   top=1
  ↓ Down along col 2:     [6, 9]      right=1
  ← Left along row 2:     [8, 7]      bottom=1
  ↑ Up along col 0:       [4]         left=1

Iteration 2:
  → Right along row 1:    [5]         top=2

top (2) > bottom (1) → stop

Result: [1, 2, 3, 6, 9, 8, 7, 4, 5]
```


---

6. Java Solution

## Java Implementation

```java
import java.util.ArrayList;
import java.util.List;

public class SpiralMatrix {

    /**
     * Returns all elements of the matrix in clockwise spiral order.
     * Uses four boundary pointers that shrink inward after each side is traversed.
     *
     * @param matrix m x n integer matrix
     * @return elements in spiral order
     */
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();

        if (matrix == null || matrix.length == 0) return result;

        int top = 0, bottom = matrix.length - 1;
        int left = 0, right = matrix[0].length - 1;

        while (top <= bottom && left <= right) {

            // Traverse right along the top row
            for (int j = left; j <= right; j++) {
                result.add(matrix[top][j]);
            }
            top++; // top row is fully consumed

            // Traverse down along the right column
            for (int i = top; i <= bottom; i++) {
                result.add(matrix[i][right]);
            }
            right--; // right column is fully consumed

            // Traverse left along the bottom row (guard: top <= bottom)
            if (top <= bottom) {
                for (int j = right; j >= left; j--) {
                    result.add(matrix[bottom][j]);
                }
                bottom--; // bottom row is fully consumed
            }

            // Traverse up along the left column (guard: left <= right)
            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    result.add(matrix[i][left]);
                }
                left++; // left column is fully consumed
            }
        }

        return result;
    }

    public static void main(String[] args) {
        SpiralMatrix solution = new SpiralMatrix();

        // Example 1: 3x3
        int[][] m1 = {{1,2,3},{4,5,6},{7,8,9}};
        System.out.println("Input: 3x3 matrix");
        System.out.println("Output: " + solution.spiralOrder(m1));
        // [1, 2, 3, 6, 9, 8, 7, 4, 5]

        // Example 2: 3x4
        int[][] m2 = {{1,2,3,4},{5,6,7,8},{9,10,11,12}};
        System.out.println("\nInput: 3x4 matrix");
        System.out.println("Output: " + solution.spiralOrder(m2));
        // [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]

        // Example 3: single row
        int[][] m3 = {{1,2,3,4}};
        System.out.println("\nInput: single row [1,2,3,4]");
        System.out.println("Output: " + solution.spiralOrder(m3));
        // [1, 2, 3, 4]
    }
}
```

### Output

```
Input: 3x3 matrix
Output: [1, 2, 3, 6, 9, 8, 7, 4, 5]

Input: 3x4 matrix
Output: [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]

Input: single row [1,2,3,4]
Output: [1, 2, 3, 4]
```


---

7. Python Solution

## Python Implementation

```python
def spiral_order(matrix: list[list[int]]) -> list[int]:
    """
    Return all elements of the matrix in clockwise spiral order.
    Four boundary pointers shrink inward after each side is traversed.

    Args:
        matrix: m x n list of lists

    Returns:
        Elements in spiral order

    Time:  O(m × n)
    Space: O(m × n) for output list; O(1) working space
    """
    result = []
    if not matrix:
        return result

    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1

    while top <= bottom and left <= right:

        # Traverse right along the top row
        for j in range(left, right + 1):
            result.append(matrix[top][j])
        top += 1

        # Traverse down along the right column
        for i in range(top, bottom + 1):
            result.append(matrix[i][right])
        right -= 1

        # Traverse left along the bottom row (guard needed for single-row collapse)
        if top <= bottom:
            for j in range(right, left - 1, -1):
                result.append(matrix[bottom][j])
            bottom -= 1

        # Traverse up along the left column (guard needed for single-col collapse)
        if left <= right:
            for i in range(bottom, top - 1, -1):
                result.append(matrix[i][left])
            left += 1

    return result


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: 3x3
    m1 = [[1,2,3],[4,5,6],[7,8,9]]
    print("Input: 3x3 matrix")
    print("Output:", spiral_order(m1))   # [1, 2, 3, 6, 9, 8, 7, 4, 5]

    # Example 2: 3x4
    m2 = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
    print("\nInput: 3x4 matrix")
    print("Output:", spiral_order(m2))   # [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]

    # Example 3: single row
    m3 = [[1, 2, 3, 4]]
    print("\nInput: single row")
    print("Output:", spiral_order(m3))   # [1, 2, 3, 4]
```

### Output

```
Input: 3x3 matrix
Output: [1, 2, 3, 6, 9, 8, 7, 4, 5]

Input: 3x4 matrix
Output: [1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]

Input: single row
Output: [1, 2, 3, 4]
```


---

8. Interview Q&A

## Q&A

**Q: Why check `top <= bottom` before the left-traversal step?**  
A: After advancing `top` and `right`, the spiral may have collapsed to a single row. Without the check, we'd re-traverse that row in reverse, adding duplicate elements.

**Q: What is the difference between `top <= bottom` and `left <= right` at the loop entry vs inside the loop?**  
A: The loop-entry checks confirm there is at least one row and one column remaining. The inner checks after right and bottom traversals handle single-row or single-column residuals that arise mid-iteration.

**Q: Can this be solved recursively?**  
A: Yes — traverse the outermost ring, recurse on the submatrix. The iterative boundary approach avoids call-stack overhead and is cleaner for interviews.

**Q: How would you handle an all-zero or all-same-value matrix?**  
A: The traversal logic is value-agnostic. It visits every cell regardless of value, so no special handling is needed.

**Q: What is the time complexity if the matrix is very "thin" (e.g., 1 × n)?**  
A: Still O(m × n) = O(n). The boundary guards ensure a single sweep across the row with no double-counting.


---

9. Summary

## Summary

| Attribute             | Value                                                            |
| --------------------- | ---------------------------------------------------------------- |
| **Problem**           | Return matrix elements in clockwise spiral order                 |
| **Optimal Algorithm** | Four-boundary shrinking (top/bottom/left/right)                  |
| **Time Complexity**   | O(m × n)                                                         |
| **Space Complexity**  | O(1) working space; O(m × n) for output                          |
| **Key Insight**       | Shrink the boundary after each directional sweep; guard mid-loop |

### Core Pattern

> Maintain four boundary pointers. Sweep each side, shrink that boundary, and guard against re-traversal when the spiral collapses to a single row or column.

### When to Use This Pattern

- Any matrix problem requiring layer-by-layer traversal.
- Generating spiral arrays, onion-peeling problems, or rotating rings.

