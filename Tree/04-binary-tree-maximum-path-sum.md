# Binary Tree Maximum Path Sum

---

1. What is meant by Binary Tree Maximum Path Sum?

## What is Binary Tree Maximum Path Sum?

**Binary Tree Maximum Path Sum** is a tree DP problem where:

- You are given the root of a binary tree where each node has an integer value (can be negative).

**Goal:** Return the **maximum sum** of any path in the tree. A path is any sequence of nodes where each pair of adjacent nodes shares an edge. A path does **not** need to pass through the root, and each node can appear at most once.

**Constraints:**
- `1 <= number of nodes <= 3 × 10⁴`
- `-1000 <= Node.val <= 1000`

**Example:**

```
Input:  [1, 2, 3]
         1
        / \
       2   3
Output: 6   (path: 2 → 1 → 3)

Input:  [-10, 9, 20, null, null, 15, 7]
        -10
        /  \
       9   20
           / \
          15   7
Output: 42   (path: 15 → 20 → 7)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- The path can start and end at any node (not necessarily root).
- Each node can be used at most once per path.
- Path must contain at least one node (single node is a valid path).
- Negative values can be excluded (don't extend a path if it decreases the sum).

### Non-Functional Requirements

| Concern         | Expectation                                               |
| --------------- | --------------------------------------------------------- |
| **Scale**       | Up to 3×10⁴ nodes — O(n) required                        |
| **Latency**     | O(n) — single DFS post-order pass                         |
| **Space**       | O(h) — recursion stack                                    |
| **Correctness** | Handles all-negative trees (answer = max single node val) |

### Clarifying Questions to Ask

- Can node values be negative? → Yes.
- Must the path go through the root? → No.
- Minimum path length? → 1 node.
- Can we skip a subtree if it has a negative contribution? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach: Post-order DFS with Global Max

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every node once |
| **Space** | O(h) — recursion stack; O(n) skewed, O(log n) balanced |

There is no better approach — we must visit every node to consider all paths.


---

4. Which Algorithm and Why?

## Algorithm: Post-order DFS with Global Maximum Tracker

### Key Insight

At each node, we have two separate questions:

1. **What is the best path that PASSES THROUGH this node as a peak (can go both left and right)?**
   `val + max(0, leftGain) + max(0, rightGain)` — update the global max with this.

2. **What is the best single-arm extension we can return to the parent?**
   `val + max(0, leftGain, rightGain)` — return only ONE arm (the better one) because a path through the parent can only extend in one direction from this node.

We take `max(gain, 0)` to ignore negative subtrees — extending into a negative subtree would only decrease the path sum.

```
globalMax = -∞

gain(node):
  if node is null → return 0
  leftGain  = max(0, gain(node.left))
  rightGain = max(0, gain(node.right))

  // path through this node as the "peak"
  globalMax = max(globalMax, node.val + leftGain + rightGain)

  // return best single-arm to parent
  return node.val + max(leftGain, rightGain)

return globalMax
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Post-order Visitor** — Visits children before the current node.
2. **Gain Clipper** — `max(0, childGain)` discards negative subtree contributions.
3. **Peak Calculator** — `node.val + leftGain + rightGain` — local path through this node.
4. **Global Max Updater** — Tracks the best peak seen across all nodes.
5. **Arm Returner** — Returns `node.val + max(leftGain, rightGain)` upward.

### Data Flow Diagram

```
Tree:
   -10
   /  \
  9   20
      / \
     15   7

gain(9)  = 9  (leaf),  globalMax = max(-∞, 9+0+0) = 9
gain(15) = 15 (leaf),  globalMax = max(9, 15) = 15
gain(7)  = 7  (leaf),  globalMax = max(15, 7) = 15

gain(20):
  leftGain=15, rightGain=7
  peak = 20 + 15 + 7 = 42 → globalMax = max(15, 42) = 42
  return 20 + max(15, 7) = 35

gain(-10):
  leftGain = max(0, 9) = 9
  rightGain = max(0, 35) = 35
  peak = -10 + 9 + 35 = 34 → globalMax = max(42, 34) = 42
  return -10 + max(9, 35) = 25

Answer: 42 ✓  (path: 15 → 20 → 7)
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class BinaryTreeMaximumPathSum {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    private int globalMax;

    /**
     * Find the maximum path sum in a binary tree.
     *
     * Post-order DFS: at each node compute:
     *   - best "peak" path through this node (updates globalMax)
     *   - best single-arm gain to return to parent
     *
     * Key: use max(0, childGain) to skip negative branches.
     *
     * Time:  O(n)
     * Space: O(h)
     */
    public int maxPathSum(TreeNode root) {
        globalMax = Integer.MIN_VALUE;
        gain(root);
        return globalMax;
    }

    private int gain(TreeNode node) {
        if (node == null) return 0;

        // Clip negative gains — never extend into a loss-making subtree
        int leftGain  = Math.max(0, gain(node.left));
        int rightGain = Math.max(0, gain(node.right));

        // Best path with this node as the topmost (peak) node
        int peakPath = node.val + leftGain + rightGain;
        globalMax = Math.max(globalMax, peakPath);

        // Return the best single arm to the parent
        return node.val + Math.max(leftGain, rightGain);
    }

    public static void main(String[] args) {
        BinaryTreeMaximumPathSum sol = new BinaryTreeMaximumPathSum();

        // Example 1: [1,2,3] → 6
        TreeNode r1 = new TreeNode(1);
        r1.left = new TreeNode(2); r1.right = new TreeNode(3);
        System.out.println(sol.maxPathSum(r1)); // 6

        // Example 2: [-10,9,20,null,null,15,7] → 42
        TreeNode r2 = new TreeNode(-10);
        r2.left = new TreeNode(9);
        r2.right = new TreeNode(20);
        r2.right.left = new TreeNode(15);
        r2.right.right = new TreeNode(7);
        System.out.println(sol.maxPathSum(r2)); // 42

        // Example 3: all negatives [-3] → -3
        System.out.println(sol.maxPathSum(new TreeNode(-3))); // -3

        // Example 4: [-1,-2,-3] → -1
        TreeNode r4 = new TreeNode(-1);
        r4.left = new TreeNode(-2); r4.right = new TreeNode(-3);
        System.out.println(sol.maxPathSum(r4)); // -1
    }
}
```

### Output

```
6
42
-3
-1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations
import math

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def max_path_sum(root: TreeNode) -> int:
    """
    Maximum path sum in a binary tree.

    Post-order DFS:
      - At each node compute the best path where this node is the peak.
      - Update global max with that peak.
      - Return best single-arm gain to parent (clip negative arms with max(0,...)).

    Time:  O(n)
    Space: O(h)
    """
    global_max = [-math.inf]   # list so nested function can mutate it

    def gain(node: TreeNode | None) -> int:
        if node is None:
            return 0
        left_gain  = max(0, gain(node.left))
        right_gain = max(0, gain(node.right))

        # path through this node as the topmost point
        global_max[0] = max(global_max[0], node.val + left_gain + right_gain)

        # best single arm returned to parent
        return node.val + max(left_gain, right_gain)

    gain(root)
    return global_max[0]


if __name__ == "__main__":
    r1 = TreeNode(1, TreeNode(2), TreeNode(3))
    print(max_path_sum(r1))   # 6

    r2 = TreeNode(-10, TreeNode(9),
                  TreeNode(20, TreeNode(15), TreeNode(7)))
    print(max_path_sum(r2))   # 42

    print(max_path_sum(TreeNode(-3)))  # -3

    r4 = TreeNode(-1, TreeNode(-2), TreeNode(-3))
    print(max_path_sum(r4))   # -1
```

### Output

```
6
42
-3
-1
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why do we use `max(0, childGain)` instead of just `childGain`?**
A: If a subtree has a negative total gain, including it only hurts the path sum. Capping at 0 means we simply don't extend the path into that subtree — effectively treating it as not taken.

**Q: Why can the arm returned to the parent only go in ONE direction (left OR right)?**
A: A path in a tree is a sequence of connected nodes with no branching. If this node contributes to a longer path through its parent, it can only extend in one direction from itself — otherwise the "path" would branch and no longer be a valid path.

**Q: What initializes `globalMax` and why not 0?**
A: We initialize to `Integer.MIN_VALUE` (or `-∞`) because all nodes could be negative, and the minimum valid answer is the value of the worst node. Initializing to 0 would incorrectly return 0 for an all-negative tree.

**Q: How does this handle a single-node tree?**
A: `gain(node)`: leftGain=0, rightGain=0. Peak = node.val + 0 + 0 = node.val. globalMax = node.val. Returns node.val. Correct.

**Q: Can the path go from a leaf up to a node and back down another branch?**
A: Yes — that is exactly what the `node.val + leftGain + rightGain` peak calculation captures. The path goes up through the left subtree, through this node, and down through the right subtree (or any subset of these).


---

9. Summary

## Summary

| Attribute             | Value                                                                        |
| --------------------- | ---------------------------------------------------------------------------- |
| **Problem**           | Find the maximum sum of any path in a binary tree                            |
| **Optimal Algorithm** | Post-order DFS with a global max tracker                                     |
| **Time Complexity**   | O(n)                                                                         |
| **Space Complexity**  | O(h)                                                                         |
| **Key Insight**       | Separate "peak path through node" (both arms) from "arm returned to parent" (one arm); clip negatives |

### Core Pattern

> Post-order: compute left/right gains clipped to 0. Update global max with `val + leftGain + rightGain`. Return `val + max(leftGain, rightGain)` to parent. Initialize global max to -∞ for all-negative trees.

### When to Use This Pattern

- Any path optimization in a tree where paths can start/end anywhere.
- Related: Diameter of Binary Tree (same pattern, counting edges), Path Sum III, Maximum Sum BST in Binary Tree.

