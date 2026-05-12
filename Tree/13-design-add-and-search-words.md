# Design Add and Search Words Data Structure

---

1. What is meant by Add and Search Words?

## What is Design Add and Search Words Data Structure?

**Add and Search Words** extends the Trie to support wildcard matching:

- **addWord(word)** — Add a word to the data structure.
- **search(word)** — Return `true` if the word exists. The word may contain dots (`.`) where each `.` matches **any single character**.

**Constraints:**
- `1 <= word.length <= 25`
- Words in `addWord` consist of lowercase letters only.
- Words in `search` may contain `.` (any char) or lowercase letters.
- At most `10⁴` calls to `addWord`, at most `10⁴` calls to `search`.

**Example:**

```java
WordDictionary wd = new WordDictionary();
wd.addWord("bad");
wd.addWord("dad");
wd.addWord("mad");
wd.search("pad");  // false
wd.search("bad");  // true
wd.search(".ad");  // true  (b/d/m)ad
wd.search("b..");  // true  (bad)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- `addWord`: Insert a word (no wildcards).
- `search`: Exact match for letters; `.` matches any single character.
- `.` can appear at any position in the search word.

### Non-Functional Requirements

| Concern         | Expectation                                                   |
| --------------- | ------------------------------------------------------------- |
| **Scale**       | Up to 10⁴ words, word length up to 25                        |
| **Latency**     | O(L) addWord; O(26^dots × L) search worst case with many `.` |
| **Space**       | O(26 × N) trie nodes, N = total inserted characters           |
| **Correctness** | `.` must match any character, not be treated as a literal     |

### Clarifying Questions to Ask

- Does `.` match any character including end-of-word? → `.` matches any single character (not end-of-word).
- Can wildcards be in `addWord`? → No, only in `search`.
- Can the search word be all dots? → Yes — matches any word of that length.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

| Operation | Time | Space |
| --- | --- | --- |
| `addWord(word)` | O(L) | O(L × 26) worst case |
| `search(word)` no dots | O(L) | O(L) recursion stack |
| `search(word)` with d dots | O(26^d × L) worst case | O(L) |

In practice, words are short (≤ 25) and dots sparse, so this is fast.


---

4. Which Algorithm and Why?

## Algorithm: Trie + DFS with Backtracking for Wildcards

### Why DFS for `.`?

| Option | Correctness | Efficiency |
| --- | --- | --- |
| Flat string match + regex | Correct | O(n × L) per query |
| **Trie + DFS for wildcard** | **Correct** | **O(26^dots × L)** — prunes non-matching branches |

### Key Insight

Build a standard Trie for `addWord`. For `search`:
- If the current character is a **letter** → standard Trie traversal (deterministic).
- If the current character is `.` → try **all 26 children** recursively. If any branch succeeds, return true.

This is DFS with backtracking: explore all possibilities, short-circuit on the first match.

```
search(word):
  dfs(node, index):
    if index == word.length → return node.isEnd
    c = word[index]
    if c == '.':
      for each child in node.children:
        if child != null AND dfs(child, index+1) → return true
      return false
    else:
      child = node.children[c - 'a']
      if child == null → return false
      return dfs(child, index+1)
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Trie** — Standard trie for storing inserted words.
2. **DFS Traverser** — Navigates the trie character by character.
3. **Wildcard Handler** — On `.`, fans out to all non-null children and recurses.
4. **Early Exit** — Returns `true` as soon as any branch succeeds.

### Data Flow Diagram

```
addWord("bad"), addWord("dad"), addWord("mad")

Trie structure (simplified):
  root
  ├── b → a → d [isEnd]
  ├── d → a → d [isEnd]
  └── m → a → d [isEnd]

search(".ad"):
  dfs(root, 0):  char='.' → try all children: b, d, m
    dfs(b-node, 1): char='a' → follow 'a'
      dfs(a-node, 2): char='d' → follow 'd'
        dfs(d-node, 3): index==len → isEnd=true → return true ✓

search("b.."):
  dfs(root, 0): char='b' → follow b-node
    dfs(b-node, 1): char='.' → try all children of b: only 'a'
      dfs(a-node, 2): char='.' → try all children of a: only 'd'
        dfs(d-node, 3): index==len → isEnd=true → return true ✓

search("pad"):
  dfs(root, 0): char='p' → root.children['p']=null → false ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class WordDictionary {

    private static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root = new TrieNode();

    /**
     * Add a word to the trie.
     *
     * Time:  O(L)
     */
    public void addWord(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null)
                node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    /**
     * Search for a word with optional '.' wildcards.
     *
     * '.' matches any single character → DFS all children at that position.
     * Letters → standard trie traversal.
     *
     * Time:  O(26^dots × L) worst case
     * Space: O(L) — recursion depth
     */
    public boolean search(String word) {
        return dfs(root, word, 0);
    }

    private boolean dfs(TrieNode node, String word, int i) {
        if (i == word.length()) return node.isEnd;   // consumed all chars

        char c = word.charAt(i);

        if (c == '.') {
            // Try every possible child at this position
            for (TrieNode child : node.children) {
                if (child != null && dfs(child, word, i + 1))
                    return true;                      // short-circuit on first match
            }
            return false;
        } else {
            // Deterministic: follow exact character
            TrieNode child = node.children[c - 'a'];
            return child != null && dfs(child, word, i + 1);
        }
    }

    public static void main(String[] args) {
        WordDictionary wd = new WordDictionary();
        wd.addWord("bad");
        wd.addWord("dad");
        wd.addWord("mad");

        System.out.println(wd.search("pad"));  // false
        System.out.println(wd.search("bad"));  // true
        System.out.println(wd.search(".ad"));  // true
        System.out.println(wd.search("b.."));  // true
        System.out.println(wd.search("..."));  // true
        System.out.println(wd.search("b.d."));  // false (length 4, words length 3)
    }
}
```

### Output

```
false
true
true
true
true
false
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TrieNode:
    def __init__(self):
        self.children: dict[str, TrieNode] = {}
        self.is_end: bool = False


class WordDictionary:
    """
    Trie with DFS wildcard search.
    '.' in search matches any single character.

    addWord:  O(L)
    search:   O(26^dots × L) worst case
    """

    def __init__(self):
        self.root = TrieNode()

    def add_word(self, word: str) -> None:
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:
        return self._dfs(self.root, word, 0)

    def _dfs(self, node: TrieNode, word: str, i: int) -> bool:
        if i == len(word):
            return node.is_end          # consumed all characters

        ch = word[i]
        if ch == '.':
            # Try all children at this wildcard position
            for child in node.children.values():
                if self._dfs(child, word, i + 1):
                    return True         # short-circuit on first match
            return False
        else:
            if ch not in node.children:
                return False
            return self._dfs(node.children[ch], word, i + 1)


if __name__ == "__main__":
    wd = WordDictionary()
    wd.add_word("bad")
    wd.add_word("dad")
    wd.add_word("mad")

    print(wd.search("pad"))   # False
    print(wd.search("bad"))   # True
    print(wd.search(".ad"))   # True
    print(wd.search("b.."))   # True
    print(wd.search("..."))   # True
    print(wd.search("b.d."))  # False
```

### Output

```
False
True
True
True
True
False
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why does the wildcard require DFS instead of a direct lookup?**
A: A `.` at position i could match any of 26 characters. There's no single path to follow — we must explore all existing children at that level and continue downward. DFS with backtracking explores all possibilities.

**Q: What is the worst-case search time?**
A: If every character is `.` (all wildcards) in a word of length L, we must explore up to `26^L` paths. In practice this is bounded by the actual number of nodes in the trie — if the trie has N nodes, the worst case is O(N).

**Q: How can you improve search performance for heavy wildcard usage?**
A: For very dense wildcard queries, you could precompute all words and use a KMP/Aho-Corasick automaton. Alternatively, limit prefix length and use caching. In practice the standard DFS trie is efficient enough for typical word lengths (≤ 25).

**Q: How does this differ from `Implement Trie`?**
A: `Implement Trie` only supports exact character traversal. `WordDictionary` adds `'.'` wildcard support by fanning out DFS at each `.` position, while the rest of the logic is identical.

**Q: Can you handle `*` (zero or more characters) like in glob matching?**
A: Yes — at a `*`, try both staying at the current node (match zero more chars) and consuming the child (match one char). This is similar to the regular expression matching problem with dynamic programming or recursive backtracking.


---

9. Summary

## Summary

| Attribute             | Value                                                                            |
| --------------------- | -------------------------------------------------------------------------------- |
| **Problem**           | Trie with wildcard `.` search (matches any single character)                     |
| **Optimal Algorithm** | Trie + DFS — letters follow deterministic path; `.` fans out to all children    |
| **Time Complexity**   | O(L) addWord; O(26^dots × L) search                                             |
| **Space Complexity**  | O(26 × N)                                                                        |
| **Key Insight**       | At `.`: iterate all non-null children, DFS each; short-circuit on first true     |

### Core Pattern

> Standard Trie insert. DFS search: letter → follow exact child; `.` → for each non-null child, DFS. Short-circuit return true. Base case: end of word → return isEnd.

### When to Use This Pattern

- Pattern matching with wildcards in string datasets.
- Related: Implement Trie, Word Search II (Trie + grid DFS), Regular Expression Matching.

