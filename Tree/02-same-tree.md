# Same Tree

---

1. What is meant by Same Tree?

## What is Same Tree?

**Same Tree** is a structural comparison problem where:

- You are given the roots of two binary trees `p` and `q`.

**Goal:** Return `true` if the two trees are **structurally identical** with the **same node values** at every position.

**Constraints:**
- `0 <= number of nodes in each tree <= 100`
- `-10⁴ <= Node.val <= 10⁴`

**Example:**

```
Input:  p = [1,2,3],  q = [1,2,3]
Output: true

      1           1
     / \         / \
    2   3       2   3

Input:  p = [1,2],  q = [1,null,2]
Output: false

      1           1
     /             \
    2               2

Input:  p = [1,2,1],  q = [1,1,2]
Output: false
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Both trees must have identical structure (same shape).
- Corresponding nodes must have equal values.
- Two empty trees are the same (both null → true).
- One null and one non-null → false.

### Non-Functional Requirements

| Concern         | Expectation                                            |
| --------------- | ------------------------------------------------------ |
| **Scale**       | Up to 100 nodes each — any traversal works             |
| **Latency**     | O(n) — visit every node pair once                      |
| **Space**       | O(h) — recursion stack depth = min(h1, h2)             |
| **Correctness** | Must catch both structural and value mismatches        |

### Clarifying Questions to Ask

- Are two empty trees the same? → Yes.
- Does node ordering matter (left vs. right)? → Yes — structural position matters.
- Can node values be negative? → Yes, -10⁴ to 10⁴.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Recursive DFS

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit every pair of corresponding nodes |
| **Space** | O(h) — recursion stack, h = min height of the two trees |

### Approach 2: Iterative (stack or queue)

| | Complexity |
| --- | --- |
| **Time** | O(n) |
| **Space** | O(n) — stack/queue holds node pairs |

Recursive DFS is the standard solution — clean and concise.


---

4. Which Algorithm and Why?

## Algorithm: Recursive Pre-order DFS

### Why Recursive?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Recursive DFS** | **O(n)** | **O(h)** | **Yes** |
| Iterative (stack pairs) | O(n) | O(n) | No — more verbose |

### Key Insight

Compare corresponding nodes top-down:

1. If both are `null` → `true` (structural match, no values to check).
2. If one is `null` and the other is not → `false` (structural mismatch).
3. If values differ → `false`.
4. Recursively check left subtrees AND right subtrees.

Short-circuit evaluation stops as soon as any mismatch is found.

```
isSame(p, q):
  if p == null AND q == null → true
  if p == null OR  q == null → false
  if p.val != q.val          → false
  return isSame(p.left, q.left) AND isSame(p.right, q.right)
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Both-Null Check** — Two empty subtrees are identical.
2. **One-Null Check** — Structural mismatch; return false immediately.
3. **Value Check** — Values at corresponding positions must match.
4. **Recursive Descent** — Both left subtrees AND both right subtrees must match.

### Data Flow Diagram

```
p:    1          q:    1
     / \               / \
    2   3             2   3

isSame(1,1): vals match → isSame(2,2) AND isSame(3,3)
  isSame(2,2): vals match → isSame(null,null) AND isSame(null,null) → true AND true = true
  isSame(3,3): vals match → true AND true = true
→ true AND true = true ✓

---

p: 1      q: 1
   /            \
  2              2

isSame(1,1): vals match → isSame(p.left=2, q.left=null)
  → one null, one not → false
→ false ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class SameTree {

    static class TreeNode {
        int val; TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    /**
     * Check if two binary trees are structurally identical with equal values.
     *
     * Pre-order recursive comparison:
     *   both null   → true
     *   one null    → false
     *   values differ → false
     *   else → recurse on left pair AND right pair
     *
     * Time:  O(n)  — n = min(nodes in p, nodes in q) until mismatch
     * Space: O(h)  — h = min height of p, q
     */
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;   // both empty → same
        if (p == null || q == null) return false;  // one empty → different
        if (p.val != q.val)         return false;  // value mismatch

        return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
    }

    // Iterative version using a queue of node pairs
    public boolean isSameTreeIterative(TreeNode p, TreeNode q) {
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(p); queue.offer(q);

        while (!queue.isEmpty()) {
            TreeNode a = queue.poll();
            TreeNode b = queue.poll();

            if (a == null && b == null) continue;
            if (a == null || b == null) return false;
            if (a.val != b.val)         return false;

            queue.offer(a.left);  queue.offer(b.left);
            queue.offer(a.right); queue.offer(b.right);
        }
        return true;
    }

    public static void main(String[] args) {
        SameTree sol = new SameTree();

        // Example 1: same structure and values
        TreeNode p1 = new TreeNode(1);
        p1.left = new TreeNode(2); p1.right = new TreeNode(3);
        TreeNode q1 = new TreeNode(1);
        q1.left = new TreeNode(2); q1.right = new TreeNode(3);
        System.out.println(sol.isSameTree(p1, q1)); // true

        // Example 2: different structure
        TreeNode p2 = new TreeNode(1); p2.left = new TreeNode(2);
        TreeNode q2 = new TreeNode(1); q2.right = new TreeNode(2);
        System.out.println(sol.isSameTree(p2, q2)); // false

        // Example 3: same structure, different values
        TreeNode p3 = new TreeNode(1);
        p3.left = new TreeNode(2); p3.right = new TreeNode(1);
        TreeNode q3 = new TreeNode(1);
        q3.left = new TreeNode(1); q3.right = new TreeNode(2);
        System.out.println(sol.isSameTree(p3, q3)); // false

        // Example 4: both empty
        System.out.println(sol.isSameTree(null, null)); // true
    }
}
```

### Output

```
true
false
false
true
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


def is_same_tree(p: TreeNode | None, q: TreeNode | None) -> bool:
    """
    Check if two binary trees are structurally identical with equal values.

    Pre-order recursive comparison:
      both None         → True
      one None          → False (structural mismatch)
      values differ     → False
      otherwise         → recurse on both subtrees

    Time:  O(n)
    Space: O(h)
    """
    if p is None and q is None: return True
    if p is None or  q is None: return False
    if p.val != q.val:          return False
    return is_same_tree(p.left, q.left) and is_same_tree(p.right, q.right)


def is_same_tree_iterative(p: TreeNode | None, q: TreeNode | None) -> bool:
    """Iterative BFS version using a queue of pairs."""
    queue = deque([(p, q)])
    while queue:
        a, b = queue.popleft()
        if a is None and b is None: continue
        if a is None or  b is None: return False
        if a.val != b.val:          return False
        queue.append((a.left,  b.left))
        queue.append((a.right, b.right))
    return True


if __name__ == "__main__":
    p1 = TreeNode(1, TreeNode(2), TreeNode(3))
    q1 = TreeNode(1, TreeNode(2), TreeNode(3))
    print(is_same_tree(p1, q1))   # True

    p2 = TreeNode(1, TreeNode(2))
    q2 = TreeNode(1, None, TreeNode(2))
    print(is_same_tree(p2, q2))   # False

    p3 = TreeNode(1, TreeNode(2), TreeNode(1))
    q3 = TreeNode(1, TreeNode(1), TreeNode(2))
    print(is_same_tree(p3, q3))   # False

    print(is_same_tree(None, None))  # True
```

### Output

```
True
False
False
True
```


---

8. Interview Questions and Answers

## Q&A

**Q: What are the three base cases you must check?**
A: (1) Both null → true (both subtrees empty). (2) Exactly one null → false (structural mismatch). (3) Values differ → false. Only after all three pass do we recurse.

**Q: Why does the order of null checks matter?**
A: Check "both null" before "one null". If we checked `p == null || q == null` first and both are null, we'd incorrectly return false. The both-null check must come first.

**Q: How would you adapt this to check structural equality only (ignoring values)?**
A: Remove the value check and just ensure both null simultaneously or both non-null simultaneously, recursing on both children.

**Q: Can this be solved without recursion?**
A: Yes — use an explicit stack or queue storing pairs of corresponding nodes. Dequeue a pair, run the same three checks, then enqueue child pairs. Avoids recursion overhead.

**Q: What is the worst-case space usage?**
A: O(n) for a completely skewed tree (effectively a linked list). O(log n) for a perfectly balanced tree.


---

9. Summary

## Summary

| Attribute             | Value                                                         |
| --------------------- | ------------------------------------------------------------- |
| **Problem**           | Determine if two binary trees are structurally + value equal  |
| **Optimal Algorithm** | Recursive pre-order DFS with three short-circuit checks       |
| **Time Complexity**   | O(n)                                                          |
| **Space Complexity**  | O(h)                                                          |
| **Key Insight**       | Both null = true; one null = false; values differ = false; then recurse both sides |

### Core Pattern

> Three guards before recursion: both-null → true, one-null → false, value mismatch → false. Then `isSame(left, left) AND isSame(right, right)`.

### When to Use This Pattern

- Any pair-wise tree comparison.
- Related: Subtree of Another Tree (calls isSameTree at each node), Symmetric Tree, Leaf-Similar Trees.

