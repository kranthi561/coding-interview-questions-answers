# Invert Binary Tree

---

1. What is meant by Invert Binary Tree?

## What is Invert Binary Tree?

**Invert Binary Tree** is a structural transformation problem where:

- You are given the root of a binary tree.

**Goal:** Invert (mirror) the tree — swap the left and right children at **every** node — and return its root.

**Constraints:**
- `0 <= number of nodes <= 100`
- `-100 <= Node.val <= 100`

**Example:**

```
Input:
        4
       / \
      2   7
     / \ / \
    1  3 6  9

Output:
        4
       / \
      7   2
     / \ / \
    9  6 3  1
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Swap left and right children at every node in the tree.
- The transformation must be in-place (modifies existing nodes).
- Return the root of the inverted tree.
- An empty tree should return null.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | Up to 100 nodes — O(n) trivially fast               |
| **Latency**     | O(n) — every node must be visited                   |
| **Space**       | O(h) recursive / O(w) iterative BFS                 |
| **Correctness** | Every node's children must be swapped, not just root |

### Clarifying Questions to Ask

- Must the swap be in-place? → Yes, return the modified root.
- Does an empty tree return null? → Yes.
- Does node value change? → No, only the left/right pointers swap.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Recursive DFS

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every node once |
| **Space** | O(h) — recursion stack |

### Approach 2: Iterative BFS

| | Complexity |
| --- | --- |
| **Time** | O(n) |
| **Space** | O(w) — queue holds one level at a time |

Both are O(n) time. The recursive solution is the most concise.


---

4. Which Algorithm and Why?

## Algorithm: Recursive Post-order DFS

### Why Recursive?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Recursive DFS** | **O(n)** | **O(h)** | **Yes — simplest** |
| Iterative BFS | O(n) | O(w) | Alternative |

### Key Insight

At each node: swap its left and right children. It doesn't matter whether you swap before or after recursing (pre-order or post-order both work), because swapping at the node level is independent of what happens below.

```
invert(node):
  if node is null → return null
  swap(node.left, node.right)
  invert(node.left)
  invert(node.right)
  return node
```

Or equivalently (pre-order style):
```
invert(node):
  if node is null → return null
  left  = invert(node.left)
  right = invert(node.right)
  node.left  = right
  node.right = left
  return node
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Base Case** — Null node returns null.
2. **Swap** — Exchange `node.left` and `node.right`.
3. **Recurse** — Apply inversion to both subtrees.
4. **Output** — Return the root (same object, children swapped).

### Data Flow Diagram

```
Input:        4
             / \
            2   7
           / \ / \
          1  3 6  9

invert(4):
  invert(2):
    invert(1): returns 1 (leaf)
    invert(3): returns 3 (leaf)
    swap(1,3) → 2.left=3, 2.right=1   →  2 (right=1,left=3)
  invert(7):
    invert(6): returns 6 (leaf)
    invert(9): returns 9 (leaf)
    swap(6,9) → 7.left=9, 7.right=6
  swap(2,7) at root → 4.left=7, 4.right=2

Output:       4
             / \
            7   2
           / \ / \
          9  6 3  1  ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class InvertBinaryTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Invert (mirror) a binary tree recursively.
     *
     * At each node: swap left and right children,
     * then recurse into the (now-swapped) subtrees.
     *
     * Time:  O(n) — visit every node
     * Space: O(h) — call stack depth
     */
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;

        // Swap children
        TreeNode temp  = root.left;
        root.left  = root.right;
        root.right = temp;

        // Recurse into both subtrees
        invertTree(root.left);
        invertTree(root.right);

        return root;
    }

    /**
     * Iterative BFS version.
     * Process each node level-by-level, swapping children at every node.
     */
    public TreeNode invertTreeBFS(TreeNode root) {
        if (root == null) return null;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();

            // Swap children
            TreeNode temp  = node.left;
            node.left  = node.right;
            node.right = temp;

            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        return root;
    }

    private static String levelOrder(TreeNode root) {
        if (root == null) return "[]";
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        List<String> result = new ArrayList<>();
        while (!q.isEmpty()) {
            TreeNode n = q.poll();
            if (n == null) { result.add("null"); continue; }
            result.add(String.valueOf(n.val));
            q.offer(n.left); q.offer(n.right);
        }
        // trim trailing nulls
        int last = result.size() - 1;
        while (last >= 0 && result.get(last).equals("null")) last--;
        return result.subList(0, last + 1).toString();
    }

    public static void main(String[] args) {
        InvertBinaryTree sol = new InvertBinaryTree();

        // Example 1: [4,2,7,1,3,6,9]
        TreeNode root = new TreeNode(4);
        root.left = new TreeNode(2); root.right = new TreeNode(7);
        root.left.left = new TreeNode(1); root.left.right = new TreeNode(3);
        root.right.left = new TreeNode(6); root.right.right = new TreeNode(9);
        System.out.println(levelOrder(sol.invertTree(root))); // [4, 7, 2, 9, 6, 3, 1]

        // Example 2: [2,1,3]
        TreeNode root2 = new TreeNode(2);
        root2.left = new TreeNode(1); root2.right = new TreeNode(3);
        System.out.println(levelOrder(sol.invertTree(root2))); // [2, 3, 1]

        // Example 3: empty
        System.out.println(levelOrder(sol.invertTree(null))); // []
    }
}
```

### Output

```
[4, 7, 2, 9, 6, 3, 1]
[2, 3, 1]
[]
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


def invert_tree(root: TreeNode | None) -> TreeNode | None:
    """
    Invert (mirror) a binary tree in place.

    Recursively swap left and right at every node.

    Time:  O(n)
    Space: O(h)
    """
    if root is None:
        return None
    root.left, root.right = root.right, root.left  # swap
    invert_tree(root.left)
    invert_tree(root.right)
    return root


def invert_tree_bfs(root: TreeNode | None) -> TreeNode | None:
    """Iterative BFS — swap children at each dequeued node."""
    if not root:
        return None
    queue = deque([root])
    while queue:
        node = queue.popleft()
        node.left, node.right = node.right, node.left
        if node.left:  queue.append(node.left)
        if node.right: queue.append(node.right)
    return root


def level_order(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:  queue.append(node.left)
        if node.right: queue.append(node.right)
    return result


if __name__ == "__main__":
    root = TreeNode(4, TreeNode(2, TreeNode(1), TreeNode(3)),
                       TreeNode(7, TreeNode(6), TreeNode(9)))
    invert_tree(root)
    print(level_order(root))  # [4, 7, 2, 9, 6, 3, 1]

    root2 = TreeNode(2, TreeNode(1), TreeNode(3))
    invert_tree(root2)
    print(level_order(root2)) # [2, 3, 1]
```

### Output

```
[4, 7, 2, 9, 6, 3, 1]
[2, 3, 1]
```


---

8. Interview Questions and Answers

## Q&A

**Q: Does it matter whether we use pre-order or post-order for the swap?**
A: No. Swapping at a node is independent of what happens in its subtrees. Both pre-order (swap then recurse) and post-order (recurse then swap) produce the same final result.

**Q: What is the famous story about this problem?**
A: Max Howell (creator of Homebrew) reportedly was rejected by Google because he couldn't solve this problem on a whiteboard. It became a meme about the disconnect between algorithmic interviews and real-world engineering.

**Q: How does this differ from rotating a tree?**
A: Inverting (mirroring) produces a reflection of the original tree — every left/right pair is swapped at every level. Rotation typically refers to AVL/Red-Black tree rotations used for rebalancing, which only move subtrees locally.

**Q: What if we only swap the root's children but not the subtrees?**
A: Only the top level would be mirrored. Subtrees would remain in their original orientation, producing an incorrect partial mirror.

**Q: Can you invert using only one line of Python?**
A: `return TreeNode(root.val, invert_tree(root.right), invert_tree(root.left)) if root else None` — though this creates new nodes rather than inverting in place.


---

9. Summary

## Summary

| Attribute             | Value                                                    |
| --------------------- | -------------------------------------------------------- |
| **Problem**           | Mirror a binary tree by swapping all left/right children |
| **Optimal Algorithm** | Recursive DFS — swap at each node, recurse on children   |
| **Time Complexity**   | O(n)                                                     |
| **Space Complexity**  | O(h)                                                     |
| **Key Insight**       | Swap at every node; order (pre vs post) doesn't matter   |

### Core Pattern

> At each node: swap left and right. Recurse into both children. Base case: null → null. Pre-order or post-order both work.

### When to Use This Pattern

- Any tree structural transformation applied at every node.
- Related: Symmetric Tree (check mirror equality), Same Tree, Flip Equivalent Binary Trees.

