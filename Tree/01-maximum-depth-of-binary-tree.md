# Maximum Depth of Binary Tree

---

1. What is meant by Maximum Depth of Binary Tree?

## What is Maximum Depth of Binary Tree?

**Maximum Depth of Binary Tree** is a fundamental tree traversal problem where:

- You are given the root of a binary tree.

**Goal:** Return its **maximum depth** — the number of nodes along the longest path from the root node down to the farthest leaf node.

**Constraints:**
- `0 <= number of nodes <= 10⁴`
- `-100 <= Node.val <= 100`

**Example:**

```
Input:
        3
       / \
      9  20
         / \
        15   7

Output: 3

Input: root = [1, null, 2]
Output: 2

Input: root = []
Output: 0
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the number of nodes along the longest root-to-leaf path.
- An empty tree has depth 0.
- A tree with just a root has depth 1.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 10⁴ nodes — O(n) trivially fast               |
| **Latency**     | O(n) — visit every node exactly once                 |
| **Space**       | O(h) recursive stack / O(w) iterative BFS queue      |
| **Correctness** | Must count nodes, not edges (depth of single node = 1) |

### Clarifying Questions to Ask

- Is an empty tree depth 0 or undefined? → 0.
- Does depth count nodes or edges? → Nodes (root alone = depth 1).
- Can nodes have only one child? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Recursive DFS (Post-order)

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every node once |
| **Space** | O(h) — call stack depth = tree height; O(n) worst case (skewed), O(log n) balanced |

### Approach 2: Iterative BFS (Level Order)

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every node once |
| **Space** | O(w) — queue holds one level at a time; O(n) worst case (complete tree's last level) |

Both are O(n) time. DFS is simpler code; BFS avoids recursion and stack overflow risk on deep trees.


---

4. Which Algorithm and Why?

## Algorithm: Recursive DFS (Post-order)

### Why Recursive DFS?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Recursive DFS** | **O(n)** | **O(h)** | **Yes — simplest** |
| Iterative BFS | O(n) | O(w) | Alternative |
| Iterative DFS (stack) | O(n) | O(h) | Alternative |

### Key Insight

The depth of any node = `1 + max(depth(left), depth(right))`.

This is a natural post-order recursion: solve for both children first, then combine their results. The base case is a `null` node returning 0.

```
depth(3) = 1 + max(depth(9), depth(20))
         = 1 + max(1, 1 + max(depth(15), depth(7)))
         = 1 + max(1, 1 + max(1, 1))
         = 1 + max(1, 2)
         = 3
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Base Case** — `null` node returns depth 0.
2. **Recursive Solver** — Returns `1 + max(solve(left), solve(right))`.
3. **Output** — Depth of the root.

### Data Flow Diagram

```
        3
       / \
      9  20
         / \
        15   7

maxDepth(3):
  maxDepth(9)  → maxDepth(null)=0, maxDepth(null)=0 → 1+max(0,0) = 1
  maxDepth(20):
    maxDepth(15) → 1
    maxDepth(7)  → 1
    → 1 + max(1, 1) = 2
  → 1 + max(1, 2) = 3  ✓

BFS alternative:
  Level 1: [3]          → depth = 1
  Level 2: [9, 20]      → depth = 2
  Level 3: [15, 7]      → depth = 3
  Queue empty → return 3 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class MaximumDepthBinaryTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Recursive DFS — post-order.
     * depth of a node = 1 + max(depth of left, depth of right)
     *
     * Time:  O(n)
     * Space: O(h) — h = height of tree
     */
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;                          // base case
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }

    /**
     * Iterative BFS — count levels as they're dequeued.
     *
     * Time:  O(n)
     * Space: O(w) — w = max width of tree
     */
    public int maxDepthBFS(TreeNode root) {
        if (root == null) return 0;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        int depth = 0;

        while (!queue.isEmpty()) {
            int size = queue.size();           // number of nodes at current level
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                if (node.left  != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            depth++;                           // finished one level
        }
        return depth;
    }

    public static void main(String[] args) {
        MaximumDepthBinaryTree sol = new MaximumDepthBinaryTree();

        // Example 1: [3,9,20,null,null,15,7]  → depth 3
        TreeNode root1 = new TreeNode(3);
        root1.left = new TreeNode(9);
        root1.right = new TreeNode(20);
        root1.right.left = new TreeNode(15);
        root1.right.right = new TreeNode(7);
        System.out.println(sol.maxDepth(root1));    // 3
        System.out.println(sol.maxDepthBFS(root1)); // 3

        // Example 2: [1,null,2]  → depth 2
        TreeNode root2 = new TreeNode(1);
        root2.right = new TreeNode(2);
        System.out.println(sol.maxDepth(root2));    // 2

        // Example 3: empty
        System.out.println(sol.maxDepth(null));     // 0
    }
}
```

### Output

```
3
3
2
0
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


def max_depth(root: TreeNode | None) -> int:
    """
    Recursive DFS (post-order).
    depth(node) = 1 + max(depth(left), depth(right))

    Time:  O(n)
    Space: O(h)
    """
    if root is None:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))


def max_depth_bfs(root: TreeNode | None) -> int:
    """
    Iterative BFS — count levels.

    Time:  O(n)
    Space: O(w)
    """
    if not root:
        return 0
    queue = deque([root])
    depth = 0
    while queue:
        for _ in range(len(queue)):   # process one full level
            node = queue.popleft()
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)
        depth += 1
    return depth


if __name__ == "__main__":
    # Tree: 3 → left=9, right=20(left=15, right=7)
    root = TreeNode(3, TreeNode(9), TreeNode(20, TreeNode(15), TreeNode(7)))
    print(max_depth(root))       # 3
    print(max_depth_bfs(root))   # 3

    root2 = TreeNode(1, None, TreeNode(2))
    print(max_depth(root2))      # 2

    print(max_depth(None))       # 0
```

### Output

```
3
3
2
0
```


---

8. Interview Questions and Answers

## Q&A

**Q: What is the difference between depth and height of a tree?**
A: They're often used interchangeably but technically: depth of a node is its distance from the root; height of a node is the longest path from that node to a leaf. The maximum depth of the tree equals the height of the root.

**Q: Why does the recursive solution use O(h) space?**
A: Each recursive call sits on the call stack until it returns. In the worst case (a completely skewed tree), the stack depth equals n. For a balanced tree the stack depth is O(log n).

**Q: How does the BFS approach count depth?**
A: It processes the tree level by level. Before dequeuing each level, it records the level's size (`queue.size()`), processes exactly that many nodes, then increments depth. When the queue is empty, depth holds the number of levels processed.

**Q: When would you prefer BFS over recursive DFS here?**
A: BFS avoids stack overflow for very deep skewed trees (n = 10⁴ nodes, recursion depth = 10⁴). Iterative BFS uses heap memory (queue), which is less constrained than the call stack.

**Q: Can you solve this with a single line in Python?**
A: Yes: `return 0 if not root else 1 + max(max_depth(root.left), max_depth(root.right))`. The recursive formulation is that clean.


---

9. Summary

## Summary

| Attribute             | Value                                                         |
| --------------------- | ------------------------------------------------------------- |
| **Problem**           | Find the maximum depth (root-to-leaf node count)              |
| **Optimal Algorithm** | Recursive DFS: `1 + max(depth(left), depth(right))`          |
| **Time Complexity**   | O(n)                                                          |
| **Space Complexity**  | O(h) recursive / O(w) BFS                                    |
| **Key Insight**       | Post-order recursion: solve children first, combine with +1   |

### Core Pattern

> Base case: null → 0. Recursive case: `1 + max(left depth, right depth)`. BFS alternative: count levels as they're dequeued.

### When to Use This Pattern

- Any tree height/depth computation.
- Related: Minimum Depth of Binary Tree, Balanced Binary Tree, Diameter of Binary Tree.

