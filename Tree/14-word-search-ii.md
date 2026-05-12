# Word Search II

---

1. What is meant by Word Search II?

## What is Word Search II?

**Word Search II** is a grid + trie problem where:

- You are given an `m × n` board of characters and a list of words.

**Goal:** Return all words from the list that exist as paths in the board. A word path uses cells adjacent horizontally or vertically; each cell may be used at most once per word.

**Constraints:**
- `1 <= m, n <= 12`
- Board contains only lowercase English letters.
- `1 <= words.length <= 3 × 10⁴`
- `1 <= words[i].length <= 10`
- All words are distinct.

**Example:**

```
Board:
  o a a n
  e t a e
  i h k r
  i f l v

words = ["oath", "pea", "eat", "rain"]
Output: ["eat", "oath"]
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Each found word added to the result exactly once (no duplicates).
- Path uses adjacent (up/down/left/right) cells, no diagonal.
- Each cell used at most once per path.
- Order of result does not matter.

### Non-Functional Requirements

| Concern         | Expectation                                                                      |
| --------------- | -------------------------------------------------------------------------------- |
| **Scale**       | Board up to 12×12=144 cells; up to 3×10⁴ words — O(M×N×4^L×W) naive is too slow |
| **Latency**     | Trie prunes branches not matching any word prefix — drastically reduces DFS work  |
| **Space**       | O(sum of word lengths) for trie + O(M×N) DFS stack                               |
| **Correctness** | Must not revisit cells in a single path; must find ALL matching words            |

### Clarifying Questions to Ask

- Can the same word appear multiple times? → No — words list has distinct entries.
- Output order matters? → No.
- Can a word start from any cell? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Naive: DFS for each word separately

| | Complexity |
| --- | --- |
| **Time** | O(W × M × N × 4^L) — W words × all cells × DFS depth L |
| **Space** | O(L) DFS stack per word |

### Trie + DFS (Optimal for this problem)

| | Complexity |
| --- | --- |
| **Time** | O(M × N × 4^maxL) — single DFS per cell, trie prunes non-word branches |
| **Space** | O(sum word lengths) trie + O(M × N) DFS stack |

The Trie approach avoids redundant DFS — if no word starts with `'z'`, any DFS path starting with `z` stops immediately.


---

4. Which Algorithm and Why?

## Algorithm: Build Trie of Words + DFS Grid with Trie Navigation

### Why Trie + DFS?

| Option | Time | Chosen? |
| --- | --- | --- |
| DFS per word | O(W × M×N × 4^L) | No — too slow for 3×10⁴ words |
| **Trie + single DFS** | **O(M×N × 4^maxL)** | **Yes** |

### Key Insight

Insert all words into a Trie. Then run **one DFS from each board cell**, navigating the Trie simultaneously:

- At each DFS step, check if the current character exists as a child in the current Trie node.
- If not → prune immediately (no word in our list continues this way).
- If the current Trie node marks a word end → add to result and **remove from Trie** (prevents duplicates).

**Pruning optimization:** After finding a word, remove its terminal node from the Trie. Also remove nodes with no remaining children (backtracking cleanup). This prevents re-finding the same word and further prunes the search.

```
Build Trie from all words
for each cell (r,c):
  DFS(r, c, root of trie)

DFS(r, c, trieNode):
  if r,c out of bounds OR cell visited OR board[r][c] not in trieNode.children → return
  ch = board[r][c]
  nextNode = trieNode.children[ch]
  if nextNode.word != null → add to result; nextNode.word = null (de-duplicate)
  board[r][c] = '#'  (mark visited)
  DFS all 4 neighbors with nextNode
  board[r][c] = ch   (restore)
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Trie Builder** — Insert all words; store the complete word string at the terminal node.
2. **Grid DFS** — Starts from every cell; navigates Trie simultaneously with board.
3. **Visited Marker** — Temporarily replace `board[r][c]` with `'#'` to prevent re-use.
4. **Word Collector** — When a terminal node is reached, add its word to the result.
5. **De-duplicator** — Null out `node.word` after collecting to avoid duplicates.

### Data Flow Diagram

```
Board:     words: ["eat", "oath"]
  o a a n
  e t a e    Trie: root → e → a → t [word="eat"]
  i h k r         root → o → a → t → h [word="oath"]
  i f l v

DFS from (1,0)='e':
  trieNode = root.children['e']  (exists)
  board[1][0] = '#'
  DFS from (0,0)='o': root.children['e'].children['o'] = null → prune
  DFS from (2,0)='i': .children['i'] = null → prune
  DFS from (1,1)='t': .children['t'] = null → prune
  DFS from (1,-1): out of bounds → prune
  — backtrack: wrong direction
  Actually DFS correctly explores e→a→t:
  (1,0)→(0,0)? 'o' not in e-node, (1,0)→(1,1)? 't' not in e-node...
  Wait, explore (1,0)='e' → node=e-node
    explore neighbor (0,0)='a': node=e-node.children['a'] = a-node
      explore (0,1)='a': not in a-node children (need 't')... prune
      explore (1,0)='#': visited → prune
      explore (0,-1): OOB → prune
      But (1,0)→... we'd eventually find e→a→t path via (1,0)→(0,0)→(0,1)?
      No: (0,0)→(0,1)='a': a-node.children['a']? no... 
      We need path e(1,0)→a(0,0) [wait, (0,0)='o' not 'a']

  Let me retrace correctly:
  (1,1)='t', (0,1)='a', (0,0)='o'... 
  Path "eat": e(1,0)→a(0,1)? (0,1) is adjacent to (1,1) not (1,0)
  Adjacent to (1,0): up=(0,0)='o', down=(2,0)='i', left=OOB, right=(1,1)='t'
  Hmm... let me re-read the board:
  Row 0: o a a n
  Row 1: e t a e
  Adjacent to (1,0): up=(0,0)='o', right=(1,1)='t'
  Adjacent to (1,3): up=(0,3)='n', left=(1,2)='a', down=(2,3)='r'
  Path "eat": e(1,3)→a(1,2)→t(1,1): valid! (all adjacent)
  → result: ["eat"]

DFS from (1,0)='e' leads to "oath" exploration:
  Path "oath": o(0,0)→a(0,1)→t(1,1)→h(2,1): valid!
  → result: ["eat", "oath"]
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class WordSearchII {

    private static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word; // non-null at terminal nodes — stores the complete word
    }

    private final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    /**
     * Find all words from the list present as paths in the board.
     *
     * 1. Build a Trie from all words (store word string at terminal nodes).
     * 2. DFS from every cell, navigating the Trie simultaneously.
     * 3. When a terminal node is reached, collect the word and null it out.
     *
     * Time:  O(M×N×4^maxL)  — single DFS per cell, trie prunes bad branches
     * Space: O(sum word lengths) trie + O(M×N) DFS stack
     */
    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = buildTrie(words);
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                dfs(board, r, c, root, result);
            }
        }
        return result;
    }

    private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
        if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return;
        char ch = board[r][c];
        if (ch == '#' || node.children[ch - 'a'] == null) return; // visited or no match

        node = node.children[ch - 'a'];

        if (node.word != null) {
            result.add(node.word);
            node.word = null; // prevent duplicate collection
        }

        board[r][c] = '#'; // mark visited
        for (int[] d : DIRS) {
            dfs(board, r + d[0], c + d[1], node, result);
        }
        board[r][c] = ch;  // restore
    }

    private TrieNode buildTrie(String[] words) {
        TrieNode root = new TrieNode();
        for (String word : words) {
            TrieNode node = root;
            for (char c : word.toCharArray()) {
                int idx = c - 'a';
                if (node.children[idx] == null)
                    node.children[idx] = new TrieNode();
                node = node.children[idx];
            }
            node.word = word; // store word at terminal
        }
        return root;
    }

    public static void main(String[] args) {
        WordSearchII sol = new WordSearchII();

        char[][] board = {
            {'o','a','a','n'},
            {'e','t','a','e'},
            {'i','h','k','r'},
            {'i','f','l','v'}
        };
        String[] words = {"oath","pea","eat","rain"};
        List<String> result = sol.findWords(board, words);
        Collections.sort(result);
        System.out.println(result); // [eat, oath]

        // Example 2: single cell
        char[][] board2 = {{'a'}};
        System.out.println(sol.findWords(board2, new String[]{"a"})); // [a]
    }
}
```

### Output

```
[eat, oath]
[a]
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TrieNode:
    def __init__(self):
        self.children: dict[str, TrieNode] = {}
        self.word: str | None = None  # non-None at terminal nodes


def find_words(board: list[list[str]], words: list[str]) -> list[str]:
    """
    Find all words present as paths in the board using Trie + DFS.

    Build a Trie from all target words (store word string at terminal nodes).
    DFS from every cell, navigating the Trie simultaneously.
    Collect words when terminal node is reached; null out to prevent duplicates.

    Time:  O(M×N × 4^maxL)
    Space: O(sum word lengths) + O(M×N)
    """
    # Build Trie
    root = TrieNode()
    for word in words:
        node = root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.word = word

    rows, cols = len(board), len(board[0])
    result: list[str] = []

    def dfs(r: int, c: int, node: TrieNode) -> None:
        if r < 0 or r >= rows or c < 0 or c >= cols:
            return
        ch = board[r][c]
        if ch == '#' or ch not in node.children:
            return                       # visited or no matching trie child

        node = node.children[ch]
        if node.word:
            result.append(node.word)
            node.word = None             # de-duplicate

        board[r][c] = '#'               # mark visited
        for dr, dc in ((0,1),(0,-1),(1,0),(-1,0)):
            dfs(r + dr, c + dc, node)
        board[r][c] = ch                # restore

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)

    return result


if __name__ == "__main__":
    board = [
        ['o','a','a','n'],
        ['e','t','a','e'],
        ['i','h','k','r'],
        ['i','f','l','v']
    ]
    print(sorted(find_words(board, ["oath","pea","eat","rain"])))
    # ['eat', 'oath']

    print(find_words([['a']], ["a"]))  # ['a']
```

### Output

```
['eat', 'oath']
['a']
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why use a Trie instead of a HashSet of words?**
A: A HashSet checks exact word matches — you'd need to run a full DFS for every word separately. The Trie lets a single DFS from a cell simultaneously check all words sharing the same prefix. If the current path doesn't match any word prefix, the DFS stops immediately — avoiding redundant exploration.

**Q: Why store the word string in the terminal node instead of a boolean?**
A: Storing the word string avoids reconstructing it from the DFS path. When we reach a terminal node, we have the word string ready to add to the result without needing to track the current path explicitly.

**Q: Why set `node.word = null` after collecting a word?**
A: The same word might be reachable via different paths in the board (if the board has repeated characters). Setting word to null prevents adding the same word multiple times to the result.

**Q: What is the further optimization of pruning empty Trie nodes?**
A: After setting `node.word = null`, if that node has no remaining children, remove it from its parent's children map. This shrinks the trie as words are found, further pruning the DFS. Implement via returning a boolean from DFS and removing the child pointer if true.

**Q: How does this scale compared to Word Search I (single word)?**
A: Word Search I is O(M×N×4^L) for one word. Word Search II without Trie would be O(W × M×N × 4^L). With the Trie, it's O(M×N × 4^maxL) — constant factor independent of W, amortized across all words via prefix sharing.


---

9. Summary

## Summary

| Attribute             | Value                                                                          |
| --------------------- | ------------------------------------------------------------------------------ |
| **Problem**           | Find all words from a list that exist as paths in a 2D grid                    |
| **Optimal Algorithm** | Build Trie from word list + DFS from each cell navigating trie simultaneously  |
| **Time Complexity**   | O(M×N × 4^maxL) — trie prunes branches early                                  |
| **Space Complexity**  | O(sum word lengths) trie + O(M×N) DFS stack                                   |
| **Key Insight**       | Trie prefix pruning eliminates redundant DFS paths; store word at terminal; null out after finding to deduplicate |

### Core Pattern

> Build Trie (word strings at terminals). For each cell: DFS navigating trie — if char not in trie children, prune. If terminal, collect word and null it. Mark visited with '#', restore on backtrack.

### When to Use This Pattern

- Multi-word grid searches.
- Related: Word Search I (DFS only), Implement Trie, Boggle, Stream of Characters (Trie + DFS on stream).

