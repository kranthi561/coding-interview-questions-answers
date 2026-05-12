# Lowest Common Ancestor of a BST

---

1. What is meant by Lowest Common Ancestor of a BST?

## What is Lowest Common Ancestor of a BST?

**Lowest Common Ancestor (LCA) of a BST** is a problem where:

- You are given the root of a **Binary Search Tree** and two nodes `p` and `q`.

**Goal:** Return the **lowest common ancestor** of `p` and `q` — the deepest node that is an ancestor of both `p` and `q`. A node is considered an ancestor of itself.

**Constraints:**
- `2 <= number of nodes <= 10⁵`
- `-10⁹ <= Node.val <= 10⁹`
- All node values are unique.
- `p` and `q` are different nodes and both exist in the BST.

**Example:**

```
BST:
        6
       / \
      2   8
     / \ / \
    0  4 7  9
      / \
     3   5

p=2, q=8 → LCA = 6
p=2, q=4 → LCA = 2  (node is its own ancestor)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the LCA node (deepest common ancestor).
- A node can be its own ancestor (p=2, q=4 → LCA=2).
- Both p and q are guaranteed to exist in the tree.

### Non-Functional Requirements

| Concern         | Expectation                                                |
| --------------- | ---------------------------------------------------------- |
| **Scale**       | Up to 10⁵ nodes — O(h) required                           |
| **Latency**     | O(h) — h = height; O(log n) balanced, O(n) skewed         |
| **Space**       | O(1) iterative / O(h) recursive                            |
| **Correctness** | BST ordering determines whether to go left, right, or stop |

### Clarifying Questions to Ask

- Is this a BST (with ordering property)? → Yes (different from general binary tree LCA).
- Can p or q be the root? → Yes — the root is its own ancestor.
- Are p and q guaranteed present? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Iterative (BST property)

| | Complexity |
| --- | --- |
| **Time** | O(h) — traverse from root toward LCA |
| **Space** | O(1) — no extra structures |

### Approach 2: Recursive (BST property)

| | Complexity |
| --- | --- |
| **Time** | O(h) |
| **Space** | O(h) — recursion stack |

**Iterative is optimal** — O(h) time, O(1) space.


---

4. Which Algorithm and Why?

## Algorithm: Iterative BST Navigation

### Key Insight

The BST ordering property eliminates the need to explore both subtrees. At each node:

- If **both p and q are less than** the current node → LCA is in the left subtree → go left.
- If **both p and q are greater than** the current node → LCA is in the right subtree → go right.
- Otherwise → the current node is where p and q "diverge" → **it is the LCA**.

The "diverge" condition covers three cases:
1. p and q are on opposite sides.
2. Current node equals p (or q) → it's an ancestor of q (or p).
3. Current node equals p or q.

```
node = root
while node:
  if p.val < node.val AND q.val < node.val:
    node = node.left
  elif p.val > node.val AND q.val > node.val:
    node = node.right
  else:
    return node          // diverge point = LCA
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Direction Decider** — Compares `p.val` and `q.val` with `node.val`.
2. **Left Descender** — Both targets less than current → go left.
3. **Right Descender** — Both targets greater than current → go right.
4. **LCA Detector** — Diverge point (or exact match) → return current node.

### Data Flow Diagram

```
BST:         6
            / \
           2   8
          / \ / \
         0  4 7  9
           / \
          3   5

p=2, q=8:
  node=6: 2<6 but 8>6 → diverge → return 6 ✓

p=2, q=4:
  node=6: 2<6 AND 4<6 → go left
  node=2: 2==2 (p is current) → diverge → return 2 ✓

p=3, q=5:
  node=6: 3<6 AND 5<6 → go left
  node=2: 3>2 AND 5>2 → go right
  node=4: 3<4 AND 5>4 → diverge → return 4 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class LowestCommonAncestorBST {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Find LCA of p and q in a BST using BST ordering property.
     *
     * Iterative — O(1) space:
     *   Both < current → LCA is in left subtree
     *   Both > current → LCA is in right subtree
     *   Otherwise      → current node is the LCA (diverge point)
     *
     * Time:  O(h)
     * Space: O(1)
     */
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode node = root;
        while (node != null) {
            if (p.val < node.val && q.val < node.val) {
                node = node.left;          // both smaller → go left
            } else if (p.val > node.val && q.val > node.val) {
                node = node.right;         // both larger  → go right
            } else {
                return node;               // diverge or exact match → LCA
            }
        }
        return null; // unreachable given p, q exist in tree
    }

    /**
     * Recursive version — same logic.
     *
     * Time:  O(h)
     * Space: O(h)
     */
    public TreeNode lowestCommonAncestorRecursive(TreeNode root, TreeNode p, TreeNode q) {
        if (p.val < root.val && q.val < root.val)
            return lowestCommonAncestorRecursive(root.left, p, q);
        if (p.val > root.val && q.val > root.val)
            return lowestCommonAncestorRecursive(root.right, p, q);
        return root;  // diverge point
    }

    public static void main(String[] args) {
        LowestCommonAncestorBST sol = new LowestCommonAncestorBST();

        TreeNode root = new TreeNode(6);
        root.left = new TreeNode(2); root.right = new TreeNode(8);
        root.left.left = new TreeNode(0); root.left.right = new TreeNode(4);
        root.right.left = new TreeNode(7); root.right.right = new TreeNode(9);
        root.left.right.left = new TreeNode(3);
        root.left.right.right = new TreeNode(5);

        // p=2, q=8 → LCA=6
        System.out.println(sol.lowestCommonAncestor(root,
            root.left, root.right).val); // 6

        // p=2, q=4 → LCA=2
        System.out.println(sol.lowestCommonAncestor(root,
            root.left, root.left.right).val); // 2

        // p=3, q=5 → LCA=4
        System.out.println(sol.lowestCommonAncestor(root,
            root.left.right.left, root.left.right.right).val); // 4
    }
}
```

### Output

```
6
2
4
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def lowest_common_ancestor(root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
    """
    LCA of p and q in a BST using BST ordering property.

    Iterative — O(h) time, O(1) space:
      both < current → go left
      both > current → go right
      otherwise → current is the LCA (diverge point)
    """
    node = root
    while node:
        if p.val < node.val and q.val < node.val:
            node = node.left
        elif p.val > node.val and q.val > node.val:
            node = node.right
        else:
            return node  # diverge point or exact match
    return None  # unreachable


def lca_recursive(root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
    """Recursive version — O(h) time, O(h) space."""
    if p.val < root.val and q.val < root.val:
        return lca_recursive(root.left, p, q)
    if p.val > root.val and q.val > root.val:
        return lca_recursive(root.right, p, q)
    return root


if __name__ == "__main__":
    root = TreeNode(6,
        TreeNode(2, TreeNode(0),
                 TreeNode(4, TreeNode(3), TreeNode(5))),
        TreeNode(8, TreeNode(7), TreeNode(9)))

    p, q = root.left, root.right
    print(lowest_common_ancestor(root, p, q).val)  # 6

    p, q = root.left, root.left.right
    print(lowest_common_ancestor(root, p, q).val)  # 2

    p, q = root.left.right.left, root.left.right.right
    print(lowest_common_ancestor(root, p, q).val)  # 4
```

### Output

```
6
2
4
```


---

8. Interview Questions and Answers

## Q&A

**Q: How does this differ from LCA of a general binary tree?**
A: In a general binary tree (not BST), we can't use ordering to navigate. Instead we use DFS: if `root == p or root == q`, return root; recursively get LCA from both sides; if both sides return non-null, current node is the LCA. That's O(n) and requires visiting every node. In a BST we navigate directly in O(h).

**Q: Why does the diverge condition correctly identify the LCA?**
A: The LCA is the deepest node where p and q are no longer in the same subtree. If p < node and q > node (or vice versa), node is exactly where they split. If node equals p (or q), then node is an ancestor of q (or p) and is the deepest such ancestor reachable from above.

**Q: What if p and q are the same node?**
A: The constraints say p ≠ q, but if they were equal, at some ancestor both would equal node.val — the diverge condition fires and returns that node (which would be p=q itself if we reached it).

**Q: Can this problem be solved with O(1) space recursively?**
A: In Python/Java, recursion uses O(h) stack space. True O(1) space requires the iterative approach shown (no recursion, only a loop variable).

**Q: What if the BST is heavily unbalanced (skewed)?**
A: Worst case is a linked-list shape, h = n, so O(n). A balanced BST gives O(log n). This is inherent to tree navigation, not an algorithm weakness.


---

9. Summary

## Summary

| Attribute             | Value                                                                       |
| --------------------- | --------------------------------------------------------------------------- |
| **Problem**           | Find the deepest common ancestor of two nodes in a BST                      |
| **Optimal Algorithm** | Iterative BST navigation — follow ordering to find diverge point            |
| **Time Complexity**   | O(h) — h = tree height                                                      |
| **Space Complexity**  | O(1) iterative / O(h) recursive                                             |
| **Key Insight**       | Both in left → go left; both in right → go right; otherwise current = LCA  |

### Core Pattern

> Navigate from root. At each node: both targets smaller → go left; both larger → go right; otherwise (diverge or match) → return current node as LCA.

### When to Use This Pattern

- LCA queries on BSTs.
- Related: LCA of General Binary Tree (DFS), Kth Smallest in BST, Validate BST, Insert into BST.

