# Word Search

---

1. What is Word Search?

## What is Word Search?

**Word Search** is a grid path-finding problem where you are given:

- An `m × n` character grid (`board`)
- A target string (`word`)

**Goal:** Return `true` if the word exists as a connected path in the grid using **adjacent cells** (up, down, left, right). The same cell may not be used more than once in a single path.

**Constraints:**

- Adjacency is horizontal/vertical only — no diagonals.
- The same cell cannot be reused within a single path.
- The word must be found as a contiguous sequence of adjacent cells.

**Example:**

```
Input:
board = [["A","B","C","E"],
         ["S","F","C","S"],
         ["A","D","E","E"]]
word = "ABCCED"

Output: true

Path: A(0,0) → B(0,1) → C(0,2) → C(1,2) → E(2,2) → D(2,1)
```

```
word = "SEE"   → true   (S(1,3) → E(2,3) → E(2,2))
word = "ABCB"  → false  (B cannot be reused)
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Search for a word as a connected path through adjacent cells in the grid.
- Each cell can be used at most once per path.
- Return `true` if any valid path exists; `false` otherwise.

### Non-Functional Requirements

| Concern         | Expectation                                                     |
| --------------- | --------------------------------------------------------------- |
| **Scale**       | Board up to 6 × 6 (LeetCode constraint), word up to length 15  |
| **Latency**     | Exponential worst case but pruned heavily in practice            |
| **Correctness** | Must handle words longer than board dimensions → return false   |

### Clarifying Questions to Ask

- Are characters case-sensitive? → Yes, uppercase only in LeetCode constraints.
- Can the starting cell be any cell? → Yes, try every cell as a potential start.
- Is wrapping allowed (edges connect)? → No.
- Can the word repeat characters? → Yes, just not reuse cells.


---

3. Time and Space Complexity

## Complexity Analysis

### DFS with Backtracking

|           | Complexity                                                              |
| --------- | ----------------------------------------------------------------------- |
| **Time**  | O(m × n × 4^L) — at each starting cell, DFS explores 4 directions for L steps |
| **Space** | O(L) — recursion stack depth equals the word length L                  |

**Practical note:** The actual performance is much better than the theoretical worst case because:
- Mismatching characters prune the search immediately.
- Marking visited cells prevents cycles.
- Early termination on the first successful path.


---

4. Algorithm Choice and Why

## Algorithm: DFS with Backtracking

### Why This Algorithm?

| Option                      | Time           | Space  | Chosen?                                          |
| --------------------------- | -------------- | ------ | ------------------------------------------------ |
| BFS from each start cell    | O(m×n × 4^L)  | O(L)   | Possible but BFS is awkward for path tracking    |
| **DFS with backtracking**   | **O(m×n×4^L)**| **O(L)**| **Yes — natural fit for path exploration**      |
| Trie + DFS (Word Search II) | O(m×n × 4^L)  | O(W×L) | Overkill for single word; needed for many words  |

### Logic

1. For every cell `(i, j)` in the board, call `dfs(i, j, 0)` where `0` is the current index in `word`.
2. In DFS:
   - If `index == word.length`, all characters matched → return `true`.
   - If out of bounds, already visited, or `board[i][j] != word[index]` → return `false`.
   - Mark `board[i][j]` as visited (temporarily overwrite with `#`).
   - Recurse in all 4 directions with `index + 1`.
   - **Backtrack:** Restore `board[i][j]` to its original character.
3. Return `true` if any DFS call returns `true`.

**Why overwrite with `#` instead of a visited array?**  
It avoids allocating an O(m × n) boolean array. The original value is restored after backtracking, so the board remains correct after each path attempt.


---

5. High-Level Design

## High-Level Design

### Components

1. **Starting Cell Scanner** — Iterates every `(i, j)` cell to try as a word start.
2. **DFS Pathfinder** — Recursively explores 4-directional neighbors matching the next character.
3. **Visited Marker** — Temporarily marks cells with `#` to prevent reuse within the current path.
4. **Backtracker** — Restores the cell value after returning from a DFS branch.

### Data Flow Diagram

```
Board:                  word = "ABCCED"
[A, B, C, E]
[S, F, C, S]
[A, D, E, E]

Try start = (0,0): board[0][0]='A' matches word[0]='A' ✓
  Mark (0,0) = '#'
  Try (0,1): 'B' matches word[1]='B' ✓
    Mark (0,1) = '#'
    Try (0,2): 'C' matches word[2]='C' ✓
      Mark (0,2) = '#'
      Try (1,2): 'C' matches word[3]='C' ✓
        Mark (1,2) = '#'
        Try (2,2): 'E' matches word[4]='E' ✓
          Mark (2,2) = '#'
          Try (2,1): 'D' matches word[5]='D' ✓
            index == word.length → return true ✓
```


---

6. Java Solution

## Java Implementation

```java
public class WordSearch {

    private char[][] board;
    private String word;

    /**
     * Returns true if word exists as a connected path in the board.
     * DFS with backtracking: overwrite cells with '#' during traversal,
     * restore on backtrack — avoids a separate visited array.
     *
     * @param board m x n character grid
     * @param word  target string to find
     * @return true if word exists as a path
     */
    public boolean exist(char[][] board, String word) {
        this.board = board;
        this.word = word;

        int rows = board.length;
        int cols = board[0].length;

        // Try every cell as the starting point
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (dfs(i, j, 0)) return true;
            }
        }
        return false;
    }

    /**
     * DFS from cell (row, col) attempting to match word[index..].
     *
     * @param row   current row
     * @param col   current column
     * @param index current character index in word
     * @return true if remaining word is found from this cell
     */
    private boolean dfs(int row, int col, int index) {
        // All characters matched
        if (index == word.length()) return true;

        // Out of bounds
        if (row < 0 || row >= board.length || col < 0 || col >= board[0].length) return false;

        // Cell already visited or character mismatch
        if (board[row][col] != word.charAt(index)) return false;

        // Mark visited by overwriting with sentinel
        char original = board[row][col];
        board[row][col] = '#';

        // Explore all four neighbors
        boolean found = dfs(row + 1, col, index + 1)
                     || dfs(row - 1, col, index + 1)
                     || dfs(row, col + 1, index + 1)
                     || dfs(row, col - 1, index + 1);

        // Backtrack: restore original character
        board[row][col] = original;

        return found;
    }

    public static void main(String[] args) {
        WordSearch solution = new WordSearch();

        // Example 1: word exists
        char[][] board1 = {
            {'A','B','C','E'},
            {'S','F','C','S'},
            {'A','D','E','E'}
        };
        System.out.println("word='ABCCED': " + solution.exist(board1, "ABCCED")); // true

        // Example 2: word exists
        char[][] board2 = {
            {'A','B','C','E'},
            {'S','F','C','S'},
            {'A','D','E','E'}
        };
        System.out.println("word='SEE':    " + solution.exist(board2, "SEE")); // true

        // Example 3: word does not exist (reuses cell)
        char[][] board3 = {
            {'A','B','C','E'},
            {'S','F','C','S'},
            {'A','D','E','E'}
        };
        System.out.println("word='ABCB':   " + solution.exist(board3, "ABCB")); // false
    }
}
```

### Output

```
word='ABCCED': true
word='SEE':    true
word='ABCB':   false
```


---

7. Python Solution

## Python Implementation

```python
def exist(board: list[list[str]], word: str) -> bool:
    """
    Return True if word exists as a connected path in the board.
    DFS with backtracking: temporarily mark visited cells with '#',
    restore on backtrack.

    Args:
        board: m x n grid of characters
        word:  target string

    Returns:
        True if word exists as a path in the board

    Time:  O(m × n × 4^L) where L = len(word)
    Space: O(L) recursion stack
    """
    rows, cols = len(board), len(board[0])

    def dfs(r: int, c: int, idx: int) -> bool:
        # All characters matched
        if idx == len(word):
            return True
        # Out of bounds, visited, or mismatch
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != word[idx]:
            return False

        # Mark visited
        original, board[r][c] = board[r][c], '#'

        # Explore four directions
        found = (dfs(r + 1, c, idx + 1) or
                 dfs(r - 1, c, idx + 1) or
                 dfs(r, c + 1, idx + 1) or
                 dfs(r, c - 1, idx + 1))

        # Backtrack
        board[r][c] = original

        return found

    # Try every cell as the starting point
    for i in range(rows):
        for j in range(cols):
            if dfs(i, j, 0):
                return True
    return False


# ---- Example usage ----
if __name__ == "__main__":
    import copy

    board = [["A","B","C","E"],
             ["S","F","C","S"],
             ["A","D","E","E"]]

    # Example 1: exists
    print("word='ABCCED':", exist(copy.deepcopy(board), "ABCCED"))  # True

    # Example 2: exists
    print("word='SEE':   ", exist(copy.deepcopy(board), "SEE"))     # True

    # Example 3: does not exist (reuses cell)
    print("word='ABCB':  ", exist(copy.deepcopy(board), "ABCB"))    # False
```

### Output

```
word='ABCCED': True
word='SEE':    True
word='ABCB':   False
```


---

8. Interview Q&A

## Q&A

**Q: Why overwrite with `#` instead of a separate `visited` boolean array?**  
A: It achieves the same result (prevent reuse within the current path) while using O(1) extra space. Because we restore the character on backtrack, the board is correct for all subsequent starting cells.

**Q: What prevents infinite loops in the DFS?**  
A: Marking the current cell as `#` before recursing means none of the four neighbors will match `word[index]` and try to return to this cell.

**Q: Is BFS a valid alternative?**  
A: BFS can find a path but tracking visited cells *per path* (not globally) is expensive — you'd need to copy the visited state at each BFS node. DFS with backtracking naturally undoes state on return, making it far more efficient.

**Q: How would you extend this to Word Search II (find all words from a list)?**  
A: Build a Trie from all target words. Run the same DFS but navigate the Trie in parallel with the grid. When a Trie node marks end-of-word, record the word. This amortizes the search cost across all words instead of searching each word independently.

**Q: What is the worst-case input?**  
A: A board filled with the same character (e.g., all `'A'`) and a word of all `'A'`s of maximum length. Every cell triggers the full DFS with no early pruning.

**Q: Can this be optimized?**  
A: Yes — reverse the word if its last character is rarer than its first. Starting from the rarer end prunes more branches early. Also check `len(word) > m*n` upfront and return false immediately.


---

9. Summary

## Summary

| Attribute             | Value                                                                |
| --------------------- | -------------------------------------------------------------------- |
| **Problem**           | Find if a word exists as a connected path in a character grid        |
| **Optimal Algorithm** | DFS with backtracking (in-place visited marking)                     |
| **Time Complexity**   | O(m × n × 4^L) where L = word length                                |
| **Space Complexity**  | O(L) recursion stack                                                 |
| **Key Insight**       | Overwrite cells with `#` during DFS; restore on backtrack            |

### Core Pattern

> DFS + backtracking: commit to a choice (mark visited), explore all continuations, undo the choice on return (restore cell). This pattern applies to any grid path or combination problem.

### When to Use This Pattern

- Grid path problems where each cell can be visited once per path.
- Combination/permutation problems with a "used" state that needs undoing.
- Any exhaustive search where pruning the search tree early is possible (character mismatch stops the branch).

