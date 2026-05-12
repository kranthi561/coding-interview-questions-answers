# Subtree of Another Tree

---

1. What is meant by Subtree of Another Tree?

## What is Subtree of Another Tree?

**Subtree of Another Tree** is a structural containment problem where:

- You are given the roots of two binary trees `root` and `subRoot`.

**Goal:** Return `true` if `subRoot` is a **subtree** of `root`. A subtree means there exists a node in `root` such that the subtree rooted at that node is **identical** (same structure and values) to `subRoot`.

**Constraints:**
- `1 <= root nodes <= 2000`
- `1 <= subRoot nodes <= 1000`
- `-10⁴ <= Node.val <= 10⁴`

**Example:**

```
Input:  root = [3,4,5,1,2],  subRoot = [4,1,2]
Output: true

root:       3           subRoot:  4
           / \                   / \
          4   5                 1   2
         / \
        1   2

Input:  root = [3,4,5,1,2,null,null,null,null,0],  subRoot = [4,1,2]
Output: false  (extra node 0 under 2 makes node 4's subtree different)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- `subRoot` must exactly match a complete subtree of `root` (not just a partial match).
- The matching subtree must be identical in both structure and values.
- `root` itself counts as a subtree.

### Non-Functional Requirements

| Concern         | Expectation                                                   |
| --------------- | ------------------------------------------------------------- |
| **Scale**       | root up to 2000 nodes, subRoot up to 1000 — O(m×n) acceptable |
| **Latency**     | O(m×n) — at each root node run isSameTree against subRoot    |
| **Space**       | O(m+n) — recursion stack depth for both trees                |
| **Correctness** | Partial matches (subtree has extra children) must return false |

### Clarifying Questions to Ask

- Does the root itself count as a valid subtree? → Yes.
- Must the match be exact (same structure + values)? → Yes.
- Can there be duplicate values in the trees? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

Let `m` = nodes in `root`, `n` = nodes in `subRoot`.

### Approach 1: DFS + isSameTree at Each Node

| | Complexity |
| --- | --- |
| **Time** | O(m × n) — for each of m nodes, compare up to n nodes |
| **Space** | O(m + n) — recursion stack |

### Approach 2: Serialization + String Matching (KMP)

| | Complexity |
| --- | --- |
| **Time** | O(m + n) — serialize both, KMP substring search |
| **Space** | O(m + n) — stored serializations |

For interview purposes, Approach 1 (DFS + isSameTree) is the standard expected answer. Approach 2 is a bonus.


---

4. Which Algorithm and Why?

## Algorithm: DFS on Root + isSameTree at Each Node

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **DFS + isSameTree** | O(m×n) | O(m+n) | **Yes — standard** |
| Serialization + KMP | O(m+n) | O(m+n) | Optimal but complex |

### Key Insight

At every node in `root`, check if the subtree rooted there is identical to `subRoot`. This is exactly the **Same Tree** problem applied at each node.

```
isSubtree(root, subRoot):
  if root is null → false    (exhausted root without finding match)
  if isSameTree(root, subRoot) → true
  return isSubtree(root.left, subRoot) OR isSubtree(root.right, subRoot)
```

The short-circuit `OR` stops as soon as a match is found.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Root Traverser** — DFS visits every node of `root` as a candidate match point.
2. **isSameTree** — Structurally compares the candidate subtree with `subRoot`.
3. **Short-circuit OR** — Once a match is found, propagates `true` upward immediately.

### Data Flow Diagram

```
root:         subRoot:
    3              4
   / \            / \
  4   5          1   2
 / \
1   2

isSubtree(3, 4): isSameTree(3,4)? → 3≠4 → false
  isSubtree(4, 4): isSameTree(4,4)?
    4==4 ✓, isSameTree(1,1) ✓, isSameTree(2,2) ✓ → true!
  → propagates true all the way up → return true ✓

---

Modified root (extra 0 under 2):
    3
   / \
  4   5
 / \
1   2
     \
      0

isSubtree(4, [4,1,2]): isSameTree(4, [4,1,2])?
  4==4 ✓, isSameTree(1,1) ✓
  isSameTree(2,2): 2==2 ✓, isSameTree(2.left=null, null) ✓
    isSameTree(2.right=0, null) → one null one not → false
  → false
→ isSubtree(1, [4,1,2]) → false, isSubtree(2, [4,1,2]) → false
→ false ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class SubtreeOfAnotherTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Check if subRoot is a subtree of root.
     *
     * DFS each node in root; at each node run isSameTree.
     * Short-circuit: return true as soon as a match is found.
     *
     * Time:  O(m × n) — m nodes in root, n in subRoot
     * Space: O(m + n) — recursion stacks
     */
    public boolean isSubtree(TreeNode root, TreeNode subRoot) {
        if (root == null) return false;         // exhausted root, no match
        if (isSameTree(root, subRoot)) return true;  // match found here

        // Try left and right subtrees
        return isSubtree(root.left, subRoot) || isSubtree(root.right, subRoot);
    }

    private boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        if (p.val != q.val)         return false;
        return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
    }

    public static void main(String[] args) {
        SubtreeOfAnotherTree sol = new SubtreeOfAnotherTree();

        // Example 1: subRoot [4,1,2] is a subtree of root [3,4,5,1,2]
        TreeNode root1 = new TreeNode(3);
        root1.left = new TreeNode(4); root1.right = new TreeNode(5);
        root1.left.left = new TreeNode(1); root1.left.right = new TreeNode(2);

        TreeNode sub1 = new TreeNode(4);
        sub1.left = new TreeNode(1); sub1.right = new TreeNode(2);

        System.out.println(sol.isSubtree(root1, sub1)); // true

        // Example 2: extra node 0 makes it NOT a subtree
        root1.left.right.right = new TreeNode(0);
        System.out.println(sol.isSubtree(root1, sub1)); // false

        // Example 3: single node match
        TreeNode root3 = new TreeNode(1);
        System.out.println(sol.isSubtree(root3, new TreeNode(1))); // true
        System.out.println(sol.isSubtree(root3, new TreeNode(2))); // false
    }
}
```

### Output

```
true
false
true
false
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def is_same_tree(p: TreeNode | None, q: TreeNode | None) -> bool:
    if p is None and q is None: return True
    if p is None or  q is None: return False
    if p.val != q.val:          return False
    return is_same_tree(p.left, q.left) and is_same_tree(p.right, q.right)


def is_subtree(root: TreeNode | None, sub_root: TreeNode | None) -> bool:
    """
    Check if sub_root is an exact subtree of root.

    DFS every node in root; at each candidate apply isSameTree.

    Time:  O(m × n) — m = nodes in root, n = nodes in sub_root
    Space: O(m + n) — recursion stack
    """
    if root is None:
        return False
    if is_same_tree(root, sub_root):
        return True
    return is_subtree(root.left, sub_root) or is_subtree(root.right, sub_root)


if __name__ == "__main__":
    root = TreeNode(3, TreeNode(4, TreeNode(1), TreeNode(2)), TreeNode(5))
    sub  = TreeNode(4, TreeNode(1), TreeNode(2))
    print(is_subtree(root, sub))   # True

    # Add extra node to make it not a subtree
    root.left.right.right = TreeNode(0)
    print(is_subtree(root, sub))   # False

    r = TreeNode(1)
    print(is_subtree(r, TreeNode(1)))  # True
    print(is_subtree(r, TreeNode(2)))  # False
```

### Output

```
True
False
True
False
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why check `isSameTree(root, subRoot)` before recursing into children?**
A: Pre-order approach — check the current candidate first. If it matches, no need to look further. If not, recurse left and right to find another candidate.

**Q: What is the worst-case scenario for O(m×n)?**
A: A completely skewed tree (like a linked list) of m nodes, and a subRoot that almost-matches at every node but fails at the last comparison. Each of the m candidate positions triggers a full n-node comparison.

**Q: How does the serialization + KMP approach improve this?**
A: Serialize both trees to strings (with null markers to make serialization unique). Then use KMP (Knuth-Morris-Pratt) string search to find subRoot's serialization within root's serialization in O(m+n) time.

**Q: Does `root` being null mean the answer is always false?**
A: Yes — we've exhausted all nodes of `root` without finding a match. Note: `subRoot` is guaranteed non-null per constraints; if it were null, two-null comparison in `isSameTree` would handle it.

**Q: Can there be multiple occurrences of subRoot in root?**
A: Yes — the algorithm returns true as soon as the first match is found (short-circuit OR). If we needed to count all occurrences, we'd remove the short-circuit and accumulate counts.


---

9. Summary

## Summary

| Attribute             | Value                                                                  |
| --------------------- | ---------------------------------------------------------------------- |
| **Problem**           | Check if subRoot appears as an exact subtree within root               |
| **Optimal Algorithm** | DFS on root + isSameTree at each candidate node                        |
| **Time Complexity**   | O(m × n)                                                               |
| **Space Complexity**  | O(m + n)                                                               |
| **Key Insight**       | Reuse Same Tree check at every node of root; short-circuit on first match |

### Core Pattern

> DFS root. At each node: if `isSameTree(node, subRoot)` → return true. Else recurse left OR right. Base case: null root → false.

### When to Use This Pattern

- Tree containment checks.
- Related: Same Tree, Count Univalue Subtrees, Find Duplicate Subtrees, Maximum Average Subtree.

