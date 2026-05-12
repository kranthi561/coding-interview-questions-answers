# Validate Binary Search Tree

---

1. What is meant by Validate Binary Search Tree?

## What is Validate Binary Search Tree?

**Validate Binary Search Tree** is a correctness-check problem where:

- You are given the root of a binary tree.

**Goal:** Determine if it is a valid **Binary Search Tree (BST)**. A valid BST satisfies:
- The left subtree of every node contains only nodes with values **strictly less than** the node's value.
- The right subtree contains only nodes with values **strictly greater than** the node's value.
- Both the left and right subtrees must also be valid BSTs.

**Constraints:**
- `1 <= number of nodes <= 10⁴`
- `-2³¹ <= Node.val <= 2³¹ - 1`

**Example:**

```
Input:
    2
   / \
  1   3
Output: true  ✓ (BST)

Input:
    5
   / \
  1   4
     / \
    3   6
Output: false  ✗ (4 is in right subtree of 5, but 4 < 5)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- The BST property must hold for **every** node, not just parent-child pairs.
- Strictly less/greater (no equal values — BSTs don't have duplicates).
- Return true/false.

### Non-Functional Requirements

| Concern         | Expectation                                               |
| --------------- | --------------------------------------------------------- |
| **Scale**       | Up to 10⁴ nodes — O(n)                                   |
| **Latency**     | O(n) — single traversal                                   |
| **Space**       | O(h) — recursion stack                                    |
| **Correctness** | Must check the GLOBAL constraint, not just local parent-child |

### Clarifying Questions to Ask

- Are duplicates allowed? → No — BST requires strict inequality.
- Can values be `Integer.MIN_VALUE` or `Integer.MAX_VALUE`? → Yes — use Long bounds.
- Must every subtree also be a BST? → Yes — BST property is recursive.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Min/Max Bound Propagation (DFS)

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every node once |
| **Space** | O(h) — recursion stack |

### Approach 2: Inorder Traversal (check ascending order)

| | Complexity |
| --- | --- |
| **Time** | O(n) |
| **Space** | O(h) — recursion stack |

Both are O(n). The **bound propagation** approach is more direct and generalizable.


---

4. Which Algorithm and Why?

## Algorithm: DFS with Min/Max Bound Propagation

### Why Bounds?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Bound propagation (DFS)** | O(n) | O(h) | **Yes — direct** |
| Inorder check ascending | O(n) | O(h) | Alternative |
| Check only parent-child | O(n) | O(h) | **Wrong** — misses global violations |

### Key Insight

The common mistake: checking only that `left.val < node.val` and `right.val > node.val` is **insufficient**. Example: `[5, 1, 4, null, null, 3, 6]` — node 4 is in the right subtree of 5, but 4 < 5 violates BST.

**Correct approach**: Each node must satisfy the **accumulated bounds** inherited from all its ancestors:
- Going left: update **upper bound** to `node.val`.
- Going right: update **lower bound** to `node.val`.

```
isValid(node, min, max):
  if node is null → true
  if node.val <= min OR node.val >= max → false
  return isValid(node.left, min, node.val)
      AND isValid(node.right, node.val, max)
```

Use `Long.MIN_VALUE` / `Long.MAX_VALUE` as initial open bounds (to handle `Integer.MIN/MAX_VALUE` nodes).


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Bound Tracker** — Passes `(min, max)` bounds down the recursion.
2. **Validator** — Checks `min < node.val < max` at each node.
3. **Left Descent** — Tightens upper bound: `max = node.val`.
4. **Right Descent** — Tightens lower bound: `min = node.val`.

### Data Flow Diagram

```
Tree:      5
          / \
         1   4
            / \
           3   6

isValid(5, -∞, +∞):  -∞ < 5 < +∞ ✓
  isValid(1, -∞, 5):  -∞ < 1 < 5 ✓
    isValid(null,...) → true
    isValid(null,...) → true
  isValid(4, 5, +∞):  5 < 4? NO → false! ✗
→ false ✓

---

Tree:    2
        / \
       1   3

isValid(2, -∞, +∞): -∞ < 2 < +∞ ✓
  isValid(1, -∞, 2): -∞ < 1 < 2 ✓ → true
  isValid(3, 2, +∞): 2 < 3 < +∞ ✓ → true
→ true ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class ValidateBinarySearchTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Validate BST by propagating (min, max) bounds top-down.
     *
     * Each node must satisfy: min < node.val < max.
     * Going left tightens the upper bound to node.val.
     * Going right tightens the lower bound to node.val.
     *
     * Use Long bounds to handle Integer.MIN/MAX_VALUE nodes.
     *
     * Time:  O(n)
     * Space: O(h)
     */
    public boolean isValidBST(TreeNode root) {
        return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private boolean validate(TreeNode node, long min, long max) {
        if (node == null) return true;                    // empty subtree is valid
        if (node.val <= min || node.val >= max) return false;  // violates bound

        return validate(node.left,  min, node.val)        // left: tighten upper
            && validate(node.right, node.val, max);       // right: tighten lower
    }

    /**
     * Inorder traversal alternative: valid BST ↔ strictly ascending inorder.
     */
    private long prev = Long.MIN_VALUE;
    public boolean isValidBST_Inorder(TreeNode root) {
        if (root == null) return true;
        if (!isValidBST_Inorder(root.left)) return false;
        if (root.val <= prev)  return false;    // not strictly ascending
        prev = root.val;
        return isValidBST_Inorder(root.right);
    }

    public static void main(String[] args) {
        ValidateBinarySearchTree sol = new ValidateBinarySearchTree();

        // Example 1: valid BST [2,1,3]
        TreeNode r1 = new TreeNode(2);
        r1.left = new TreeNode(1); r1.right = new TreeNode(3);
        System.out.println(sol.isValidBST(r1)); // true

        // Example 2: invalid [5,1,4,null,null,3,6]
        TreeNode r2 = new TreeNode(5);
        r2.left = new TreeNode(1);
        r2.right = new TreeNode(4);
        r2.right.left = new TreeNode(3); r2.right.right = new TreeNode(6);
        System.out.println(sol.isValidBST(r2)); // false

        // Example 3: tricky — [10,5,15,null,null,6,20] — 6 < 10 in right subtree
        TreeNode r3 = new TreeNode(10);
        r3.left = new TreeNode(5);
        r3.right = new TreeNode(15);
        r3.right.left = new TreeNode(6); r3.right.right = new TreeNode(20);
        System.out.println(sol.isValidBST(r3)); // false (6 < 10 in right subtree)
    }
}
```

### Output

```
true
false
false
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


def is_valid_bst(root: TreeNode | None) -> bool:
    """
    Validate BST with min/max bound propagation.

    Each node must satisfy: min < node.val < max.
    Going left: upper bound = node.val
    Going right: lower bound = node.val

    Time:  O(n)
    Space: O(h)
    """
    def validate(node: TreeNode | None, min_val: float, max_val: float) -> bool:
        if node is None:
            return True
        if not (min_val < node.val < max_val):
            return False
        return (validate(node.left,  min_val,   node.val) and
                validate(node.right, node.val, max_val))

    return validate(root, -math.inf, math.inf)


def is_valid_bst_inorder(root: TreeNode | None) -> bool:
    """Inorder traversal alternative — valid BST ↔ strictly ascending."""
    prev = [-math.inf]

    def inorder(node):
        if not node: return True
        if not inorder(node.left): return False
        if node.val <= prev[0]:    return False
        prev[0] = node.val
        return inorder(node.right)

    return inorder(root)


if __name__ == "__main__":
    r1 = TreeNode(2, TreeNode(1), TreeNode(3))
    print(is_valid_bst(r1))   # True

    r2 = TreeNode(5, TreeNode(1), TreeNode(4, TreeNode(3), TreeNode(6)))
    print(is_valid_bst(r2))   # False

    r3 = TreeNode(10, TreeNode(5), TreeNode(15, TreeNode(6), TreeNode(20)))
    print(is_valid_bst(r3))   # False (6 < 10 in right subtree)
```

### Output

```
True
False
False
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why is checking only parent-child pairs insufficient?**
A: Consider `[10, 5, 15, null, null, 6, 20]`. Node 6 is the left child of 15 (6 < 15 ✓) and 15 is the right child of 10 (15 > 10 ✓). But 6 < 10, so it violates the global BST constraint. Local checks miss this; bound propagation catches it.

**Q: Why use `Long.MIN_VALUE` / `Long.MAX_VALUE` instead of `Integer`?**
A: If a node has value `Integer.MIN_VALUE` and we initialized `min = Integer.MIN_VALUE`, then `node.val <= min` would fire falsely. Using Long bounds gives headroom beyond the valid int range.

**Q: How does the inorder approach work?**
A: Inorder traversal of a BST visits nodes in strictly ascending order. So we traverse inorder and check that each node's value is strictly greater than the previous. If any violation is found, return false.

**Q: Can the inorder approach handle duplicates?**
A: The strict `<=` check (`node.val <= prev`) correctly rejects duplicates (equal counts as a violation in BST).

**Q: What is the difference between a BST and a balanced BST?**
A: A BST enforces the ordering property. A balanced BST (AVL, Red-Black) additionally maintains height balance — O(log n) operations. Validation only checks the ordering property.


---

9. Summary

## Summary

| Attribute             | Value                                                                       |
| --------------------- | --------------------------------------------------------------------------- |
| **Problem**           | Verify that a binary tree satisfies the BST ordering property at all nodes  |
| **Optimal Algorithm** | DFS with propagated (min, max) bounds; each node must fall strictly within  |
| **Time Complexity**   | O(n)                                                                        |
| **Space Complexity**  | O(h)                                                                        |
| **Key Insight**       | Use inherited bounds (not just parent-child comparison) to catch global violations |

### Core Pattern

> `validate(node, min, max)`: null → true; value out of (min,max) → false; recurse left with max=node.val, right with min=node.val. Initialize with `(-∞, +∞)`.

### When to Use This Pattern

- Any BST validity or search operation.
- Related: Kth Smallest in BST, LCA of BST, Convert Sorted Array to BST, BST Iterator.

