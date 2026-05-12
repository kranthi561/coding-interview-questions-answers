# Construct Binary Tree from Preorder and Inorder Traversal

---

1. What is meant by Construct Binary Tree from Preorder and Inorder?

## What is Construct Binary Tree from Preorder and Inorder Traversal?

**Construct Binary Tree** is a tree reconstruction problem where:

- You are given two integer arrays `preorder` and `inorder` of length `n`.
- `preorder` is the preorder traversal (root → left → right) of a binary tree.
- `inorder` is the inorder traversal (left → root → right) of the same tree.

**Goal:** Reconstruct and return the binary tree.

**Constraints:**
- `1 <= n <= 3000`
- `-3000 <= preorder[i], inorder[i] <= 3000`
- All values are unique.
- `preorder` and `inorder` are guaranteed to be valid traversals of the same tree.

**Example:**

```
Input:  preorder = [3,9,20,15,7],  inorder = [9,3,15,20,7]
Output:
        3
       / \
      9  20
         / \
        15   7

Input:  preorder = [-1],  inorder = [-1]
Output: -1  (single node)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the root of the reconstructed binary tree.
- The tree must match both traversal sequences exactly.
- All node values are unique (required for unambiguous reconstruction).

### Non-Functional Requirements

| Concern         | Expectation                                              |
| --------------- | -------------------------------------------------------- |
| **Scale**       | Up to 3000 nodes — O(n) or O(n²) acceptable             |
| **Latency**     | O(n) with HashMap index lookup; O(n²) naive scan        |
| **Space**       | O(n) — recursion stack + HashMap                         |
| **Correctness** | All values unique ensures each inorder position is unambiguous |

### Clarifying Questions to Ask

- Are all values unique? → Yes — required for unique reconstruction.
- Both arrays guaranteed valid? → Yes.
- What to return for a single-node tree? → Root with no children.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Naive Scan (find root in inorder each time)

| | Complexity |
| --- | --- |
| **Time** | O(n²) — linear scan for root position at each recursive call |
| **Space** | O(n) — recursion stack |

### Approach 2: HashMap Index Lookup (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — O(1) root lookup in HashMap |
| **Space** | O(n) — HashMap + recursion stack |

**HashMap approach is optimal** — building a value→index map converts each lookup from O(n) to O(1).


---

4. Which Algorithm and Why?

## Algorithm: Recursive DFS with HashMap Index

### Key Insight

**Preorder**: first element is always the **root** of the current subtree.

**Inorder**: once we find the root's position `idx` in the inorder array:
- Elements to the **left** of `idx` form the left subtree.
- Elements to the **right** of `idx` form the right subtree.
- `leftSize = idx - inStart`

Use `leftSize` to split the preorder array:
- Left subtree preorder: `preStart+1` to `preStart+leftSize`
- Right subtree preorder: `preStart+leftSize+1` to `preEnd`

```
build(preStart, preEnd, inStart, inEnd):
  if preStart > preEnd → null

  rootVal = preorder[preStart]
  idx = inorderIndex[rootVal]         // O(1) with HashMap
  leftSize = idx - inStart

  node = TreeNode(rootVal)
  node.left  = build(preStart+1, preStart+leftSize, inStart, idx-1)
  node.right = build(preStart+leftSize+1, preEnd, idx+1, inEnd)
  return node
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **HashMap** — Prebuilt map from value → inorder index for O(1) lookup.
2. **Root Extractor** — `preorder[preStart]` is always the current subtree root.
3. **Subtree Sizer** — `leftSize = inorderIndex[root] - inStart`.
4. **Recursive Splitter** — Passes adjusted `[preStart, preEnd]` and `[inStart, inEnd]` to each child.

### Data Flow Diagram

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]
inMap: {9:0, 3:1, 15:2, 20:3, 7:4}

build(pre[0..4], in[0..4]):
  root = preorder[0] = 3,  idx=1,  leftSize=1

  left  = build(pre[1..1], in[0..0])  ← leftSize=1
    root = 9, idx=0, leftSize=0
    left  = build(pre[2..1], ...) → null (empty range)
    right = build(pre[2..1], ...) → null
    → node(9)

  right = build(pre[2..4], in[2..4])  ← right subtree
    root = preorder[2] = 20, idx=3, leftSize=1
    left  = build(pre[3..3], in[2..2])
      root=15 → node(15)
    right = build(pre[4..4], in[4..4])
      root=7  → node(7)
    → node(20, left=15, right=7)

  → node(3, left=9, right=20(15,7)) ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class ConstructBinaryTreeFromPreorderInorder {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    private int[] preorder;
    private Map<Integer, Integer> inorderIndex;

    /**
     * Reconstruct binary tree from preorder and inorder traversals.
     *
     * Key: preorder[preStart] = root of current subtree.
     *      inorderIndex[root] splits inorder into left/right subtrees.
     *      leftSize determines how to split preorder.
     *
     * Time:  O(n) — each node created exactly once, O(1) index lookup
     * Space: O(n) — HashMap + recursion stack
     */
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        this.inorderIndex = new HashMap<>();
        for (int i = 0; i < inorder.length; i++) {
            inorderIndex.put(inorder[i], i);   // value → index
        }
        return build(0, preorder.length - 1, 0, inorder.length - 1);
    }

    private TreeNode build(int preStart, int preEnd, int inStart, int inEnd) {
        if (preStart > preEnd) return null;    // empty range

        int rootVal  = preorder[preStart];
        int idx      = inorderIndex.get(rootVal);  // position in inorder
        int leftSize = idx - inStart;              // nodes in left subtree

        TreeNode node  = new TreeNode(rootVal);
        node.left  = build(preStart + 1, preStart + leftSize, inStart, idx - 1);
        node.right = build(preStart + leftSize + 1, preEnd, idx + 1, inEnd);
        return node;
    }

    private static String inorder(TreeNode root) {
        if (root == null) return "";
        return inorder(root.left) + root.val + " " + inorder(root.right);
    }

    public static void main(String[] args) {
        ConstructBinaryTreeFromPreorderInorder sol =
            new ConstructBinaryTreeFromPreorderInorder();

        // Example 1
        int[] pre1 = {3, 9, 20, 15, 7};
        int[] in1  = {9, 3, 15, 20, 7};
        TreeNode root1 = sol.buildTree(pre1, in1);
        System.out.println("Inorder: " + inorder(root1).trim()); // 9 3 15 20 7

        // Example 2: single node
        int[] pre2 = {-1};
        int[] in2  = {-1};
        TreeNode root2 = sol.buildTree(pre2, in2);
        System.out.println("Single: " + inorder(root2).trim());  // -1
    }
}
```

### Output

```
Inorder: 9 3 15 20 7
Single: -1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val; self.left = left; self.right = right


def build_tree(preorder: list[int], inorder: list[int]) -> TreeNode | None:
    """
    Reconstruct binary tree from preorder and inorder traversals.

    preorder[preStart] = root of current subtree.
    inorder_idx[root]  = split point: elements left = left subtree.
    leftSize determines how to advance preStart for right subtree.

    Time:  O(n)
    Space: O(n)
    """
    inorder_idx = {val: i for i, val in enumerate(inorder)}  # O(1) lookup

    def build(pre_start: int, pre_end: int, in_start: int, in_end: int) -> TreeNode | None:
        if pre_start > pre_end:
            return None

        root_val  = preorder[pre_start]
        idx       = inorder_idx[root_val]
        left_size = idx - in_start

        node = TreeNode(root_val)
        node.left  = build(pre_start + 1, pre_start + left_size, in_start, idx - 1)
        node.right = build(pre_start + left_size + 1, pre_end, idx + 1, in_end)
        return node

    n = len(preorder)
    return build(0, n - 1, 0, n - 1)


def inorder_str(root):
    if not root: return ""
    return inorder_str(root.left) + str(root.val) + " " + inorder_str(root.right)


if __name__ == "__main__":
    root1 = build_tree([3,9,20,15,7], [9,3,15,20,7])
    print(inorder_str(root1).strip())  # 9 3 15 20 7

    root2 = build_tree([-1], [-1])
    print(inorder_str(root2).strip())  # -1
```

### Output

```
9 3 15 20 7
-1
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why is preorder the first choice for the root?**
A: Preorder visits root before children (root → left → right), so the first element of any preorder sub-array is always the root of that subtree.

**Q: Why must all values be unique?**
A: If values could repeat, finding the root in the inorder array would be ambiguous — there could be multiple positions, leading to multiple valid trees. Unique values guarantee a single reconstruction.

**Q: What does `leftSize` represent and how is it used?**
A: `leftSize = inorderIndex[root] - inStart` counts the number of nodes in the left subtree. We use it to advance `preStart` by exactly `leftSize` positions to find where the right subtree begins in the preorder array.

**Q: Can you reconstruct a tree from only preorder + postorder?**
A: Only if the tree is a full binary tree (every node has 0 or 2 children). With inorder, any binary tree is uniquely recoverable.

**Q: What if preorder and inorder both have only one element?**
A: Single element → root is that element, `leftSize = 0`, both recursive calls get empty ranges → null children. Returns a single leaf node.


---

9. Summary

## Summary

| Attribute             | Value                                                                        |
| --------------------- | ---------------------------------------------------------------------------- |
| **Problem**           | Reconstruct a binary tree from preorder + inorder traversals                 |
| **Optimal Algorithm** | Recursive split with HashMap index for O(1) root lookup                      |
| **Time Complexity**   | O(n)                                                                         |
| **Space Complexity**  | O(n)                                                                         |
| **Key Insight**       | `preorder[preStart]` = root; its inorder position splits left/right subtrees; `leftSize` splits the preorder array |

### Core Pattern

> HashMap: value → inorder index. Recurse: root = preorder[preStart]; idx = inorderMap[root]; leftSize = idx - inStart. Left child gets preorder[preStart+1..preStart+leftSize], right gets the rest.

### When to Use This Pattern

- Any tree reconstruction from traversal pairs.
- Related: Construct from Postorder+Inorder, Construct from Preorder only (BST), Serialize/Deserialize Binary Tree.

