# Implement Trie (Prefix Tree)

---

1. What is meant by Implement Trie?

## What is Implement Trie (Prefix Tree)?

A **Trie** (also called a prefix tree or digital tree) is a tree-shaped data structure used for efficient string storage and retrieval.

**Goal:** Implement the `Trie` class supporting:
- `insert(word)` — Insert the string into the trie.
- `search(word)` — Return `true` if the exact word exists in the trie.
- `startsWith(prefix)` — Return `true` if any word in the trie starts with the given prefix.

**Constraints:**
- `1 <= word.length, prefix.length <= 2000`
- All characters are lowercase English letters (`a–z`).
- At most `3 × 10⁴` calls to the three methods.

**Example:**

```java
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");    // true
trie.search("app");      // false  (app not inserted)
trie.startsWith("app");  // true
trie.insert("app");
trie.search("app");      // true
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- `insert`: Add a word; all prefixes become valid in `startsWith`.
- `search`: Exact match — word must have been inserted.
- `startsWith`: Prefix match — at least one inserted word begins with this prefix.

### Non-Functional Requirements

| Concern         | Expectation                                                     |
| --------------- | --------------------------------------------------------------- |
| **Scale**       | Up to 3×10⁴ operations, words up to 2000 chars                 |
| **Latency**     | O(L) per operation — L = length of word/prefix                  |
| **Space**       | O(ALPHABET × N) — N total characters across all inserted words |
| **Correctness** | `search` must not return true for prefixes; `startsWith` must not require exact words |

### Clarifying Questions to Ask

- Only lowercase letters? → Yes, 26 possibilities.
- Case-sensitive? → Yes (lowercase only, per constraints).
- Can the same word be inserted multiple times? → Yes — has no effect logically.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

| Operation | Time | Space |
| --- | --- | --- |
| `insert(word)` | O(L) | O(L × 26) worst case new nodes |
| `search(word)` | O(L) | O(1) — read-only traversal |
| `startsWith(prefix)` | O(L) | O(1) |

**Total space:** O(ALPHABET_SIZE × N) where N = total characters across all words. In practice much less due to shared prefixes.


---

4. Which Algorithm and Why?

## Algorithm: Array-Based Trie Node (26 children)

### Why Trie?

| Option | insert | search | startsWith | Space |
| --- | --- | --- | --- | --- |
| HashSet of words | O(L) | O(L) | O(L × words) | O(total chars) |
| Sorted array + binary search | O(log n) | O(L log n) | O(L log n) | O(total chars) |
| **Trie** | **O(L)** | **O(L)** | **O(L)** | **O(26 × nodes)** |

### Key Insight

Each node has:
- An array of 26 child pointers (one per lowercase letter).
- A boolean `isEnd` flag marking complete words.

`insert`: Walk/create nodes for each character.  
`search`: Walk nodes; return `isEnd` at the last character.  
`startsWith`: Walk nodes; return `true` if path exists (ignore `isEnd`).

The path from root to any node represents a prefix; `isEnd` distinguishes inserted words from intermediate nodes.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **TrieNode** — 26-element array `children[]` + `isEnd` boolean.
2. **Root Node** — Empty root; all paths start here.
3. **Inserter** — Creates new nodes along the path character by character; sets `isEnd` at the last character.
4. **Searcher** — Traverses nodes; returns `isEnd` at end of word (search) or `true` if path exists (startsWith).

### Data Flow Diagram

```
Insert "apple":
  root → [a] → [p] → [p] → [l] → [e, isEnd=true]

Insert "app":
  root → [a] → [p] → [p, isEnd=true]
                      ↑
            (existing node, just set isEnd)

search("apple")  → traverse a→p→p→l→e → isEnd=true ✓
search("app")    → traverse a→p→p → isEnd=true ✓
search("ap")     → traverse a→p → isEnd=false → false ✓
startsWith("ap") → traverse a→p → node exists → true ✓
startsWith("ax") → traverse a, children['x']=null → false ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class Trie {

    private static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    /**
     * Insert a word into the trie.
     * Walk/create child nodes for each character; mark isEnd at last character.
     *
     * Time:  O(L)
     * Space: O(L) new nodes in worst case
     */
    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();  // create node if absent
            }
            node = node.children[idx];
        }
        node.isEnd = true;  // mark end of word
    }

    /**
     * Return true if the exact word exists in the trie.
     *
     * Time:  O(L)
     */
    public boolean search(String word) {
        TrieNode node = traverse(word);
        return node != null && node.isEnd;  // path must exist AND be a word end
    }

    /**
     * Return true if any word in the trie starts with this prefix.
     *
     * Time:  O(L)
     */
    public boolean startsWith(String prefix) {
        return traverse(prefix) != null;  // only need the path to exist
    }

    // Shared traversal: follow path for each character; return null if path breaks
    private TrieNode traverse(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }

    public static void main(String[] args) {
        Trie trie = new Trie();
        trie.insert("apple");
        System.out.println(trie.search("apple"));    // true
        System.out.println(trie.search("app"));      // false
        System.out.println(trie.startsWith("app"));  // true
        trie.insert("app");
        System.out.println(trie.search("app"));      // true
        System.out.println(trie.startsWith("b"));    // false
    }
}
```

### Output

```
true
false
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


class Trie:
    """
    Prefix tree supporting insert, search, and startsWith in O(L) per operation.

    Uses a dict for children (sparse; good for large alphabets).
    For a fixed 26-letter alphabet an array of 26 is equally fine.
    """

    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        """Insert word; create nodes as needed; mark isEnd at last char."""
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:
        """True if the exact word was inserted."""
        node = self._traverse(word)
        return node is not None and node.is_end

    def starts_with(self, prefix: str) -> bool:
        """True if any inserted word starts with prefix."""
        return self._traverse(prefix) is not None

    def _traverse(self, s: str) -> TrieNode | None:
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node


if __name__ == "__main__":
    trie = Trie()
    trie.insert("apple")
    print(trie.search("apple"))       # True
    print(trie.search("app"))         # False
    print(trie.starts_with("app"))    # True
    trie.insert("app")
    print(trie.search("app"))         # True
    print(trie.starts_with("b"))      # False
```

### Output

```
True
False
True
True
False
```


---

8. Interview Questions and Answers

## Q&A

**Q: What is the difference between `search` and `startsWith`?**
A: `search` requires the exact word to have been inserted — the path must end at a node with `isEnd = true`. `startsWith` only requires the path to exist — `isEnd` is irrelevant, any traversal that completes returns true.

**Q: Why use an array of 26 vs. a HashMap for children?**
A: Array: O(1) index by `c - 'a'`; fixed 26×pointer space per node even if mostly null. HashMap: sparse (less wasted space for large alphabets like Unicode); slightly slower due to hashing. For lowercase English only, array is preferred for speed.

**Q: What is the space complexity vs. a hash set of words?**
A: A HashSet stores all words as full strings — O(total characters). A Trie shares prefixes — two words with a common 5-character prefix need only 5 shared nodes instead of 10 characters. For highly prefix-sharing datasets, the Trie uses less space.

**Q: How would you count the number of words with a given prefix?**
A: Augment each node with a `count` field incremented on every insert that passes through that node. `startsWith` returns the count at the endpoint node instead of a boolean.

**Q: How would you implement `delete(word)`?**
A: Traverse to the end of the word, set `isEnd = false`. Then backtrack: if a node has no children and `isEnd = false`, remove it from its parent's children. Continue upward until a non-removable node is found.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Design a Trie with insert, search, and startsWith                  |
| **Optimal Algorithm** | Array-based TrieNode (26 children) + isEnd flag                    |
| **Time Complexity**   | O(L) per operation — L = word/prefix length                        |
| **Space Complexity**  | O(ALPHABET × N) — N total inserted characters                      |
| **Key Insight**       | `search` checks path + isEnd; `startsWith` checks path only; shared prefix path = shared nodes |

### Core Pattern

> TrieNode: children[26] + isEnd. Insert: walk/create nodes, set isEnd. Search: traverse, return isEnd. StartsWith: traverse, return path-exists. Extract common traversal into a helper.

### When to Use This Pattern

- Autocomplete, spell checking, IP routing, word games.
- Related: Design Add and Search Words (wildcard), Word Search II (Trie + DFS grid), Replace Words, Stream of Characters.

