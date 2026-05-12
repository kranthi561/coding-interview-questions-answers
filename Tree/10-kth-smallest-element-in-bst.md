# Kth Smallest Element in a BST

---

1. What is meant by Kth Smallest Element in a BST?

## What is Kth Smallest Element in a BST?

**Kth Smallest Element in a BST** is a BST property exploitation problem where:

- You are given the root of a Binary Search Tree and an integer `k`.

**Goal:** Return the **kth smallest** value (1-indexed) among all node values in the BST.

**Constraints:**
- `1 <= k <= n <= 10⁴`
- `0 <= Node.val <= 10⁴`
- k is always valid (1 ≤ k ≤ total nodes).

**Example:**

```
Input:
    3
   / \
  1   4
   \
    2
k = 1 → Output: 1

Input:
       5
      / \
     3   6
    / \
   2   4
  /
 1
k = 3 → Output: 3
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the kth smallest value (not node).
- k is 1-indexed (k=1 means the smallest).
- k is always valid per constraints.

### Non-Functional Requirements

| Concern         | Expectation                                              |
| --------------- | -------------------------------------------------------- |
| **Scale**       | Up to 10⁴ nodes — O(n) in worst case                    |
| **Latency**     | O(h + k) — stop early at the kth node found             |
| **Space**       | O(h) — recursion stack or iterative stack                |
| **Correctness** | BST inorder = ascending order — kth element is at position k |

### Clarifying Questions to Ask

- Is k guaranteed valid? → Yes, 1 ≤ k ≤ n.
- Can we modify the tree? → No.
- Are duplicates possible? → No (BST property requires distinct values here).


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach: Inorder Traversal (Early Stop)

| | Complexity |
| --- | --- |
| **Time** | O(h + k) — traverse h levels to reach leftmost, then k steps; worst case O(n) |
| **Space** | O(h) — recursion stack or explicit stack |

### If the BST is frequently queried:

Augment each node with the **count of nodes in its left subtree** to find kth in O(h) time, O(n) space preprocessing. This is the follow-up optimization.


---

4. Which Algorithm and Why?

## Algorithm: Inorder Traversal with Early Stop

### Key Insight

**Inorder traversal of a BST produces values in strictly ascending (sorted) order.** The kth node visited during inorder traversal is the kth smallest.

We decrement a counter at each visited node and stop the moment the counter hits 0.

**Recursive version:**
```
counter = k
result  = -1

inorder(node):
  if node is null OR result != -1 → return
  inorder(node.left)
  counter -= 1
  if counter == 0 → result = node.val; return
  inorder(node.right)
```

**Iterative version** (more control, stops immediately):
```
stack = [], curr = root
while curr OR stack not empty:
  while curr: push curr, curr = curr.left
  curr = stack.pop()
  k -= 1
  if k == 0: return curr.val
  curr = curr.right
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Inorder Traverser** — Visits nodes in ascending BST order (left → node → right).
2. **Counter** — Decrements on each visited node; stops at 0.
3. **Result Holder** — Captures the kth node's value.

### Data Flow Diagram

```
BST:      5
         / \
        3   6
       / \
      2   4
     /
    1

k = 3

Inorder sequence: 1 → 2 → 3 → 4 → 5 → 6

Iterative stack walk:
  push 5, 3, 2, 1 (go left as far as possible)
  pop 1 → k=3→2 (1st smallest: 1)
  pop 2 → k=2→1 (2nd smallest: 2)
  pop 3 → k=1→0 → RETURN 3 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class KthSmallestElementBST {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Find kth smallest element in BST using iterative inorder traversal.
     *
     * Inorder BST traversal visits nodes in ascending order.
     * Decrement k at each visited node; return value when k reaches 0.
     *
     * Time:  O(h + k) — h to reach leftmost, k steps inorder
     * Space: O(h)
     */
    public int kthSmallest(TreeNode root, int k) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode curr = root;

        while (curr != null || !stack.isEmpty()) {
            // Go as far left as possible
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            // Visit the node
            curr = stack.pop();
            k--;
            if (k == 0) return curr.val;   // kth smallest found

            // Move to right subtree
            curr = curr.right;
        }
        return -1; // unreachable given valid k
    }

    /**
     * Recursive version with early stop via array counter.
     */
    private int count;
    private int result;

    public int kthSmallestRecursive(TreeNode root, int k) {
        count = k; result = -1;
        inorder(root);
        return result;
    }

    private void inorder(TreeNode node) {
        if (node == null || count == 0) return;
        inorder(node.left);
        if (--count == 0) { result = node.val; return; }
        inorder(node.right);
    }

    public static void main(String[] args) {
        KthSmallestElementBST sol = new KthSmallestElementBST();

        // Example 1: k=1 → 1
        TreeNode r1 = new TreeNode(3);
        r1.left = new TreeNode(1); r1.right = new TreeNode(4);
        r1.left.right = new TreeNode(2);
        System.out.println(sol.kthSmallest(r1, 1)); // 1

        // Example 2: k=3 → 3
        TreeNode r2 = new TreeNode(5);
        r2.left = new TreeNode(3); r2.right = new TreeNode(6);
        r2.left.left = new TreeNode(2); r2.left.right = new TreeNode(4);
        r2.left.left.left = new TreeNode(1);
        System.out.println(sol.kthSmallest(r2, 3)); // 3
    }
}
```

### Output

```
1
3
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def kth_smallest(root: TreeNode | None, k: int) -> int:
    """
    Kth smallest element in BST via iterative inorder traversal.

    Inorder traversal of BST yields sorted ascending values.
    Return the value at position k.

    Time:  O(h + k)
    Space: O(h)
    """
    stack = []
    curr = root

    while curr or stack:
        while curr:               # go left as far as possible
            stack.append(curr)
            curr = curr.left

        curr = stack.pop()        # visit node
        k -= 1
        if k == 0:
            return curr.val       # found kth smallest

        curr = curr.right         # move to right subtree

    return -1  # unreachable


def kth_smallest_recursive(root: TreeNode | None, k: int) -> int:
    """Recursive inorder with early stop."""
    state = [k, -1]  # [remaining, result]

    def inorder(node):
        if not node or state[0] == 0:
            return
        inorder(node.left)
        state[0] -= 1
        if state[0] == 0:
            state[1] = node.val
            return
        inorder(node.right)

    inorder(root)
    return state[1]


if __name__ == "__main__":
    r1 = TreeNode(3, TreeNode(1, None, TreeNode(2)), TreeNode(4))
    print(kth_smallest(r1, 1))   # 1

    r2 = TreeNode(5,
         TreeNode(3, TreeNode(2, TreeNode(1)), TreeNode(4)),
         TreeNode(6))
    print(kth_smallest(r2, 3))   # 3
```

### Output

```
1
3
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why does inorder traversal yield sorted values in a BST?**
A: By the BST property, the left subtree contains values smaller than the root, and the right subtree contains values larger. Visiting left → root → right at every node recursively produces elements in ascending order.

**Q: What is the complexity advantage of the iterative version over recursive?**
A: Both are O(h+k) time and O(h) space. The iterative version stops immediately when k reaches 0 without needing to propagate a "stop" signal through recursive calls — slightly cleaner early exit.

**Q: How would you optimize for frequent kth-smallest queries?**
A: Augment each node with `leftCount` (number of nodes in its left subtree). Then binary-search: if `k <= leftCount+1` go left (or return current if `k == leftCount+1`), else `k -= leftCount+1` and go right. This achieves O(h) per query.

**Q: What is the kth largest element?**
A: Reverse-inorder: right → root → left. Decrement k at each visit; return when k=0. Equivalently, kth largest = (n-k+1)th smallest.

**Q: What if k equals the number of nodes?**
A: Inorder visits all n nodes; the last one visited (the rightmost node) is returned. Works correctly.


---

9. Summary

## Summary

| Attribute             | Value                                                                  |
| --------------------- | ---------------------------------------------------------------------- |
| **Problem**           | Find the kth smallest value in a BST                                   |
| **Optimal Algorithm** | Iterative inorder traversal with decrement counter, stop at k=0        |
| **Time Complexity**   | O(h + k)                                                               |
| **Space Complexity**  | O(h)                                                                   |
| **Key Insight**       | BST inorder = ascending order; the kth node visited is the kth smallest |

### Core Pattern

> Iterative inorder: push left spine onto stack, pop and visit (decrement k), move right. Stop when k hits 0.

### When to Use This Pattern

- Any BST order statistics.
- Related: Validate BST, LCA of BST, BST Iterator, Convert BST to Greater Sum Tree.

