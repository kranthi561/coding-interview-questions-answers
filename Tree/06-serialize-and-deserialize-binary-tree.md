# Serialize and Deserialize Binary Tree

---

1. What is meant by Serialize and Deserialize Binary Tree?

## What is Serialize and Deserialize Binary Tree?

**Serialize and Deserialize Binary Tree** is a design problem where:

- **Serialize:** Convert a binary tree to a string representation.
- **Deserialize:** Reconstruct the original tree from that string.

**Goal:** Design a codec that losslessly converts a binary tree to/from a string. Any valid encoding scheme is acceptable as long as deserialize(serialize(root)) produces the same tree.

**Constraints:**
- `0 <= number of nodes <= 10⁴`
- `-1000 <= Node.val <= 1000`

**Example:**

```
Input:
        1
       / \
      2   3
         / \
        4   5

serialize → "1,2,null,null,3,4,null,null,5,null,null"
deserialize("1,2,null,null,3,4,null,null,5,null,null") → same tree
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- `serialize(root)` → a string encoding the tree.
- `deserialize(data)` → the original TreeNode root.
- Round-trip must be lossless: `deserialize(serialize(t)) == t` structurally and by value.
- Handle empty tree (null root).

### Non-Functional Requirements

| Concern         | Expectation                                              |
| --------------- | -------------------------------------------------------- |
| **Scale**       | Up to 10⁴ nodes — O(n) both directions                  |
| **Latency**     | O(n) serialize and deserialize                           |
| **Space**       | O(n) — string length proportional to node count         |
| **Correctness** | Must preserve structure: null children encoded explicitly |

### Clarifying Questions to Ask

- Can we use any encoding format? → Yes, as long as it round-trips.
- Must null nodes be encoded? → Yes — required to reconstruct structure.
- Can node values contain commas? → No, they're integers.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach: Pre-order DFS with Null Markers

| | Complexity |
| --- | --- |
| **Serialize Time** | O(n) — visit every node once |
| **Serialize Space** | O(n) — string length proportional to nodes |
| **Deserialize Time** | O(n) — consume each token once |
| **Deserialize Space** | O(n) — split string + recursion stack |

Both directions are O(n) time and space.


---

4. Which Algorithm and Why?

## Algorithm: Pre-order DFS with Null Markers

### Why Pre-order?

| Option | Complexity | Notes |
| --- | --- | --- |
| **Pre-order DFS + null markers** | O(n) | Simple; uniquely determines tree |
| BFS level-order | O(n) | Also works; slightly more complex parsing |
| In-order only | O(n) | NOT unique without extra info (need pre/post too) |

### Key Insight

**Pre-order (root → left → right)** uniquely encodes the tree if null leaves are explicitly marked.

**Serialize:** DFS pre-order; append `"null"` for missing children, node value for existing nodes.

**Deserialize:** Use an index (or iterator) into the token array. Read the next token:
- If `"null"` → return null.
- Else → create a node with that value, recurse for left child, then right child.

The recursive structure of deserialization mirrors the recursive structure of serialization — each recursive call consumes exactly the tokens it needs.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Serializer (DFS)** — Pre-order traversal appending values and `"null"` markers.
2. **Delimiter** — Comma (`,`) separates tokens; allows splitting on deserialize.
3. **Token Queue/Index** — Deserializer consumes tokens sequentially.
4. **Deserializer (DFS)** — Reads one token per node; recurses for children.

### Data Flow Diagram

```
Tree:     1
         / \
        2   3
           / \
          4   5

Serialize (pre-order):
  visit 1   → "1"
  visit 2   → "1,2"
  visit null→ "1,2,null"
  visit null→ "1,2,null,null"
  visit 3   → "1,2,null,null,3"
  visit 4   → "1,2,null,null,3,4"
  visit null→ "1,2,null,null,3,4,null"
  visit null→ "1,2,null,null,3,4,null,null"
  visit 5   → "1,2,null,null,3,4,null,null,5"
  visit null→ "1,2,null,null,3,4,null,null,5,null"
  visit null→ "1,2,null,null,3,4,null,null,5,null,null"

Deserialize tokens: [1, 2, null, null, 3, 4, null, null, 5, null, null]
  consume 1  → node(1), recurse left
    consume 2 → node(2), recurse left
      consume null → null
    recurse right
      consume null → null
    return node(2)
  recurse right
    consume 3 → node(3), recurse left
      consume 4 → node(4), recurse left
        consume null → null
      recurse right
        consume null → null
      return node(4)
    recurse right
      consume 5 → node(5), ...
    return node(3)
  return node(1)  ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class SerializeDeserializeBinaryTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    private static final String NULL = "null";
    private static final String SEP  = ",";

    /**
     * Serialize: pre-order DFS appending values and "null" markers.
     *
     * Time:  O(n)
     * Space: O(n)
     */
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeDFS(root, sb);
        return sb.toString();
    }

    private void serializeDFS(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append(NULL).append(SEP);
            return;
        }
        sb.append(node.val).append(SEP);  // root first (pre-order)
        serializeDFS(node.left,  sb);
        serializeDFS(node.right, sb);
    }

    /**
     * Deserialize: split on comma, use a queue to consume tokens
     * in the same pre-order sequence.
     *
     * Time:  O(n)
     * Space: O(n)
     */
    public TreeNode deserialize(String data) {
        Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(SEP)));
        return deserializeDFS(tokens);
    }

    private TreeNode deserializeDFS(Queue<String> tokens) {
        String token = tokens.poll();
        if (NULL.equals(token)) return null;       // null marker → no node

        TreeNode node = new TreeNode(Integer.parseInt(token));
        node.left  = deserializeDFS(tokens);       // consume left subtree
        node.right = deserializeDFS(tokens);       // consume right subtree
        return node;
    }

    private static boolean isSame(TreeNode a, TreeNode b) {
        if (a == null && b == null) return true;
        if (a == null || b == null) return false;
        return a.val == b.val && isSame(a.left, b.left) && isSame(a.right, b.right);
    }

    public static void main(String[] args) {
        SerializeDeserializeBinaryTree codec = new SerializeDeserializeBinaryTree();

        // Example 1
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.right.left = new TreeNode(4);
        root.right.right = new TreeNode(5);

        String serialized = codec.serialize(root);
        System.out.println("Serialized: " + serialized);
        // 1,2,null,null,3,4,null,null,5,null,null,

        TreeNode restored = codec.deserialize(serialized);
        System.out.println("Round-trip same: " + isSame(root, restored)); // true

        // Example 2: empty
        System.out.println("Empty: " + codec.serialize(null)); // null,
        System.out.println("Empty restored: " + (codec.deserialize("null,") == null)); // true
    }
}
```

### Output

```
Serialized: 1,2,null,null,3,4,null,null,5,null,null,
Round-trip same: true
Empty: null,
Empty restored: true
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def serialize(root: TreeNode | None) -> str:
    """
    Pre-order DFS serialization with 'null' markers.

    Time:  O(n)
    Space: O(n)
    """
    parts = []

    def dfs(node):
        if node is None:
            parts.append("null")
            return
        parts.append(str(node.val))
        dfs(node.left)
        dfs(node.right)

    dfs(root)
    return ",".join(parts)


def deserialize(data: str) -> TreeNode | None:
    """
    Pre-order DFS deserialization consuming tokens from a queue.

    Time:  O(n)
    Space: O(n)
    """
    tokens = deque(data.split(","))

    def dfs() -> TreeNode | None:
        token = tokens.popleft()
        if token == "null":
            return None
        node = TreeNode(int(token))
        node.left  = dfs()
        node.right = dfs()
        return node

    return dfs()


def is_same(a, b):
    if a is None and b is None: return True
    if a is None or  b is None: return False
    return a.val == b.val and is_same(a.left, b.left) and is_same(a.right, b.right)


if __name__ == "__main__":
    root = TreeNode(1, TreeNode(2),
                    TreeNode(3, TreeNode(4), TreeNode(5)))
    s = serialize(root)
    print("Serialized:", s)
    # 1,2,null,null,3,4,null,null,5,null,null

    restored = deserialize(s)
    print("Round-trip same:", is_same(root, restored))  # True

    print("Empty:", serialize(None))                    # null
    print("Empty null?", deserialize("null") is None)  # True
```

### Output

```
Serialized: 1,2,null,null,3,4,null,null,5,null,null
Round-trip same: True
Empty: null
Empty null? True
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why must null nodes be explicitly encoded?**
A: Without null markers, different trees can produce the same sequence of values. For example, `[1,2]` and `[1,null,2]` both have values `1,2` but different structures. Null markers disambiguate left vs. right positions.

**Q: Why does pre-order uniquely determine the tree (with nulls)?**
A: Pre-order visits root first. Knowing the root, the subsequent tokens for the left subtree are consumed recursively until exhausted (via null markers), then the right subtree tokens follow. Each recursive call consumes exactly what it needs, leaving the rest for the caller.

**Q: Can you use BFS for serialization instead?**
A: Yes — serialize level by level with null markers. Deserialization is slightly more complex: use two queues (parents, children), match children to parents in order. It produces a shorter string for wide trees but the same O(n) complexity.

**Q: What separators or formats could you use instead of comma?**
A: Any delimiter that doesn't appear in node values: space, pipe `|`, newline. You could also length-prefix each token (e.g., `"3:abc"`) for strings with potential delimiters.

**Q: What if the tree contains duplicate values?**
A: Doesn't matter — the serialization encodes structure (positions) via pre-order + null markers, not uniqueness of values.


---

9. Summary

## Summary

| Attribute             | Value                                                                        |
| --------------------- | ---------------------------------------------------------------------------- |
| **Problem**           | Losslessly encode and reconstruct a binary tree via a string                 |
| **Optimal Algorithm** | Pre-order DFS with explicit null markers                                     |
| **Time Complexity**   | O(n) serialize and deserialize                                               |
| **Space Complexity**  | O(n)                                                                         |
| **Key Insight**       | Pre-order + null markers uniquely identifies both structure and values; deserialize mirrors the same recursive structure |

### Core Pattern

> Serialize: DFS pre-order, append value or "null" with delimiter. Deserialize: split into token queue, DFS consuming one token per node — "null" returns null, otherwise create node and recurse left then right.

### When to Use This Pattern

- Any tree/graph encoding for storage or transmission.
- Related: Construct Binary Tree from Preorder+Inorder, Deserialize BST, Encode N-ary Tree.

