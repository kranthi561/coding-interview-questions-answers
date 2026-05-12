# Binary Tree Level Order Traversal

---

1. What is meant by Binary Tree Level Order Traversal?

## What is Binary Tree Level Order Traversal?

**Binary Tree Level Order Traversal** (BFS) is a problem where:

- You are given the root of a binary tree.

**Goal:** Return the node values **level by level** from left to right, as a list of lists.

**Constraints:**
- `0 <= number of nodes <= 2000`
- `-1000 <= Node.val <= 1000`

**Example:**

```
Input:
        3
       / \
      9  20
         / \
        15   7

Output: [[3], [9, 20], [15, 7]]

Input:  root = [1]
Output: [[1]]

Input:  root = []
Output: []
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return each level's nodes as a separate sub-list.
- Left-to-right ordering within each level.
- Empty tree → empty list.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 2000 nodes — O(n) required                     |
| **Latency**     | O(n) — visit every node once                         |
| **Space**       | O(w) — queue holds the widest level (up to n/2 nodes) |
| **Correctness** | Must group nodes by exact level depth                |

### Clarifying Questions to Ask

- Left-to-right within levels? → Yes.
- Return as a list of lists? → Yes.
- What if root is null? → Return empty list.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach: BFS with Level Boundary Tracking

| | Complexity |
| --- | --- |
| **Time** | O(n) — every node enqueued and dequeued once |
| **Space** | O(w) — queue size = widest level; O(n/2) for a full binary tree's last level |

The output itself is O(n) since it contains all node values.


---

4. Which Algorithm and Why?

## Algorithm: BFS (Level-by-Level Queue Processing)

### Why BFS?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **BFS with level snapshot** | **O(n)** | **O(w)** | **Yes** |
| DFS with depth tracking | O(n) | O(h) | Alternative — less natural |

### Key Insight

Use a queue. Before processing each level, **snapshot the current queue size** — that is the number of nodes at this level. Dequeue exactly that many nodes, collect their values, and enqueue their children. Each outer loop iteration corresponds to one tree level.

```
queue = [root]
result = []

while queue not empty:
    levelSize = queue.size()
    levelValues = []
    for i in 0..levelSize-1:
        node = queue.dequeue()
        levelValues.append(node.val)
        if node.left:  queue.enqueue(node.left)
        if node.right: queue.enqueue(node.right)
    result.append(levelValues)
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Queue** — Holds nodes to process; initialized with root.
2. **Level Sizer** — Reads `queue.size()` at the start of each outer iteration.
3. **Level Collector** — Dequeues `levelSize` nodes, records values, enqueues children.
4. **Result Builder** — Appends each level's list to the final result.

### Data Flow Diagram

```
Tree:    3
        / \
       9  20
          / \
         15   7

Init: queue=[3], result=[]

Outer 1: levelSize=1
  dequeue 3  → level=[3],  enqueue 9, 20
  result=[[3]], queue=[9,20]

Outer 2: levelSize=2
  dequeue 9  → level=[9],  no children
  dequeue 20 → level=[9,20], enqueue 15, 7
  result=[[3],[9,20]], queue=[15,7]

Outer 3: levelSize=2
  dequeue 15 → level=[15], no children
  dequeue 7  → level=[15,7], no children
  result=[[3],[9,20],[15,7]], queue=[]

Queue empty → return [[3],[9,20],[15,7]] ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class BinaryTreeLevelOrderTraversal {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Level order traversal — collect node values grouped by level.
     *
     * BFS: snapshot queue.size() at the start of each level loop.
     * Process exactly that many nodes, enqueue their children.
     * Each outer iteration = one tree level.
     *
     * Time:  O(n)
     * Space: O(w) — w = max width (queue)
     */
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int levelSize = queue.size();          // nodes at this level
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);               // collect value
                if (node.left  != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }

    public static void main(String[] args) {
        BinaryTreeLevelOrderTraversal sol = new BinaryTreeLevelOrderTraversal();

        // Example 1: [3,9,20,null,null,15,7]
        TreeNode root = new TreeNode(3);
        root.left = new TreeNode(9);
        root.right = new TreeNode(20);
        root.right.left = new TreeNode(15);
        root.right.right = new TreeNode(7);
        System.out.println(sol.levelOrder(root)); // [[3], [9, 20], [15, 7]]

        // Example 2: single node
        System.out.println(sol.levelOrder(new TreeNode(1))); // [[1]]

        // Example 3: empty
        System.out.println(sol.levelOrder(null)); // []
    }
}
```

### Output

```
[[3], [9, 20], [15, 7]]
[[1]]
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


def level_order(root: TreeNode | None) -> list[list[int]]:
    """
    Level order traversal — return node values grouped by level.

    BFS: snapshot queue length at each level start to know how many
    nodes belong to the current level.

    Time:  O(n)
    Space: O(w) — w = max width of tree
    """
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)         # nodes at current level
        level = []

        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)

        result.append(level)

    return result


if __name__ == "__main__":
    root = TreeNode(3, TreeNode(9), TreeNode(20, TreeNode(15), TreeNode(7)))
    print(level_order(root))            # [[3], [9, 20], [15, 7]]
    print(level_order(TreeNode(1)))     # [[1]]
    print(level_order(None))            # []
```

### Output

```
[[3], [9, 20], [15, 7]]
[[1]]
[]
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why snapshot `queue.size()` at the start of each level?**
A: As we process nodes in the current level, we enqueue children for the next level. Without snapshotting the size upfront, we wouldn't know where one level ends and the next begins — we'd process all nodes in one flat list.

**Q: How would you do reverse level order (bottom to top)?**
A: Same BFS, but prepend each level to the result (or append and reverse at the end). Alternatively, use a stack: push each level, then pop all.

**Q: How would you do zigzag level order (alternate left-right, right-left)?**
A: Track a boolean `leftToRight`. On even levels add left-to-right; on odd levels add right-to-left (either reverse the level list or use a deque and add to front or back based on direction).

**Q: What is the space complexity of the output?**
A: O(n) — the result contains all n node values across all levels, regardless of the tree shape.

**Q: Can this be done with DFS?**
A: Yes — pass the current depth to a recursive DFS and append the node's value to `result[depth]`. Pre-order DFS with depth tracking achieves the same O(n) time but O(h) auxiliary space (recursion stack) instead of O(w).


---

9. Summary

## Summary

| Attribute             | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Problem**           | Collect binary tree node values grouped by level               |
| **Optimal Algorithm** | BFS with level-size snapshot                                   |
| **Time Complexity**   | O(n)                                                           |
| **Space Complexity**  | O(w) — w = max width of tree                                   |
| **Key Insight**       | Snapshot `queue.size()` at the start of each outer loop = nodes at this level |

### Core Pattern

> BFS queue. Before each level: record `levelSize = queue.size()`. Dequeue exactly `levelSize` nodes, collect values, enqueue their children. Append the collected level to result.

### When to Use This Pattern

- Any level-by-level tree processing (zigzag, right side view, average per level).
- Related: Binary Tree Right Side View, Average of Levels, Zigzag Traversal, Minimum Depth.

