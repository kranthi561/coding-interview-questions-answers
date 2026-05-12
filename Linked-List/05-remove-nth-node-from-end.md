# Remove Nth Node From End of List

---

1. What is meant by Remove Nth Node From End?

## What is Remove Nth Node From End?

**Remove Nth Node From End** is a linked list problem where:

- You are given the head of a singly linked list and an integer `n`.

**Goal:** Remove the **nth node from the end** of the list and return its head.

**Constraints:**
- `1 <= number of nodes <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= number of nodes`

**Example:**

```
Input:  1 → 2 → 3 → 4 → 5,  n = 2
Output: 1 → 2 → 3 → 5

Input:  1,  n = 1
Output: (empty)

Input:  1 → 2,  n = 1
Output: 1
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Remove exactly the nth node counting from the end (1-indexed).
- Return the head of the modified list.
- If the head itself is removed, return `head.next`.

### Non-Functional Requirements

| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | Up to 30 nodes — any O(n) approach is fine            |
| **Latency**     | O(n) — single-pass preferred                          |
| **Space**       | O(1) — no extra data structures                       |
| **Correctness** | Must handle n = length (remove head), n = 1 (remove tail) |

### Clarifying Questions to Ask

- Is n always valid (1 ≤ n ≤ length)? → Yes, guaranteed.
- Can the list have only one node? → Yes — removing it returns null.
- Must this be done in one pass? → Preferred (two-pointer trick achieves this).


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Two-Pass (count length, then delete)

| | Complexity |
| --- | --- |
| **Time** | O(n) — one pass to count, one pass to delete |
| **Space** | O(1) |

### Approach 2: Two-Pointer (one pass)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single traversal |
| **Space** | O(1) — two pointer variables |

**Two-pointer is preferred** — same complexity but achieves the result in a single pass, which is cleaner and often asked for in interviews.


---

4. Which Algorithm and Why?

## Algorithm: Two-Pointer (One Pass)

### Why Two-Pointer?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Two-pass (count + delete) | O(n) | O(1) | No — two traversals |
| **Two-pointer (single pass)** | **O(n)** | **O(1)** | **Yes** |

### Key Insight

Use a **gap of n nodes** between two pointers:

1. Advance `fast` pointer n steps ahead.
2. Move both `slow` and `fast` together until `fast` reaches the last node.
3. Now `slow` is one node **before** the node to delete.
4. `slow.next = slow.next.next` removes the target node.

Use a **dummy head** so that removing the first node needs no special case.

```
dummy → 1 → 2 → 3 → 4 → 5,  n = 2

Step 1: advance fast 2 steps:    fast = 3
Step 2: move both until fast.next == null:
         fast=4, slow=1
         fast=5, slow=2
Step 3: slow.next = slow.next.next   →  2.next = 4
Result: 1 → 2 → 4 → 5  ✓  (node 3 removed, which was 2nd from end)
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Dummy Head** — Sentinel before the list; lets slow start before the real head.
2. **Fast Pointer** — Advanced n steps ahead; marks the "end" reference.
3. **Slow Pointer** — Lags n steps behind fast; stops at the node before the target.
4. **Deletion Step** — `slow.next = slow.next.next`.
5. **Output** — `dummy.next` — the real head (handles head-removal case).

### Data Flow Diagram

```
n = 2,  list: 1 → 2 → 3 → 4 → 5

dummy → 1 → 2 → 3 → 4 → 5 → null
↑
slow=dummy, fast=dummy

--- Advance fast n=2 steps ---
fast moves to 1, then 2:
  dummy → 1 → 2 → 3 → 4 → 5
  slow=dummy, fast=2

--- Move both until fast.next == null ---
Step 1: slow=1, fast=3
Step 2: slow=2, fast=4
Step 3: slow=3, fast=5   (fast.next == null → stop)

--- Delete slow.next ---
slow (node 3) → skip node 4 → attach node 5
3.next = 5

Result: dummy → 1 → 2 → 3 → 5
Return: dummy.next = 1 → 2 → 3 → 5 ✓

--- Head-removal case: n = 5 ---
fast advances 5 steps to node 5.
Move both until fast.next==null → fast stays at 5, slow stays at dummy.
slow.next = slow.next.next → dummy.next = node 2
Result: 2 → 3 → 4 → 5 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class RemoveNthNodeFromEnd {

    static class ListNode {
        int val; ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Remove the nth node from the end of the list in one pass.
     *
     * Two-pointer trick with a dummy head:
     *  1. Advance fast n steps from dummy.
     *  2. Move slow and fast together until fast.next == null.
     *  3. slow is now the node just before the target — skip it.
     *
     * @param head head of the list
     * @param n    1-indexed position from the end to remove
     * @return head of the modified list
     *
     * Time:  O(L) — L = list length
     * Space: O(1)
     */
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;

        ListNode slow = dummy;
        ListNode fast = dummy;

        // Advance fast n steps ahead
        for (int i = 0; i < n; i++) {
            fast = fast.next;
        }

        // Move both until fast reaches the last node
        while (fast.next != null) {
            slow = slow.next;
            fast = fast.next;
        }

        // slow.next is the node to remove
        slow.next = slow.next.next;

        return dummy.next;
    }

    private static ListNode build(int... vals) {
        ListNode dummy = new ListNode(0); ListNode cur = dummy;
        for (int v : vals) { cur.next = new ListNode(v); cur = cur.next; }
        return dummy.next;
    }

    private static String str(ListNode h) {
        StringBuilder sb = new StringBuilder();
        while (h != null) { sb.append(h.val); if (h.next != null) sb.append("→"); h = h.next; }
        return sb.length() == 0 ? "(empty)" : sb.toString();
    }

    public static void main(String[] args) {
        RemoveNthNodeFromEnd sol = new RemoveNthNodeFromEnd();

        // Example 1: remove 2nd from end (node 4)
        System.out.println(str(sol.removeNthFromEnd(build(1,2,3,4,5), 2))); // 1→2→3→5

        // Example 2: remove only node
        System.out.println(str(sol.removeNthFromEnd(build(1), 1)));         // (empty)

        // Example 3: remove last node
        System.out.println(str(sol.removeNthFromEnd(build(1,2), 1)));       // 1

        // Example 4: remove head (n == length)
        System.out.println(str(sol.removeNthFromEnd(build(1,2,3), 3)));     // 2→3
    }
}
```

### Output

```
1→2→3→5
(empty)
1
2→3
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val; self.next = next


def remove_nth_from_end(head: ListNode | None, n: int) -> ListNode | None:
    """
    Remove the nth node from the end of the list in a single pass.

    Two-pointer trick with dummy head:
      - fast advances n steps ahead of slow.
      - Both move together until fast.next is None.
      - slow.next is the node to remove; skip it.

    Args:
        head: Head of the linked list
        n:    1-indexed position from end to remove

    Returns:
        Head of the modified list

    Time:  O(L) — L = list length
    Space: O(1)
    """
    dummy = ListNode(0, head)
    slow = fast = dummy

    for _ in range(n):       # advance fast n steps
        fast = fast.next

    while fast.next:         # move both until fast is last
        slow = slow.next
        fast = fast.next

    slow.next = slow.next.next  # skip the target node

    return dummy.next


def build(*vals):
    dummy = ListNode(); cur = dummy
    for v in vals: cur.next = ListNode(v); cur = cur.next
    return dummy.next

def to_str(h):
    parts = []
    while h: parts.append(str(h.val)); h = h.next
    return "→".join(parts) if parts else "(empty)"


if __name__ == "__main__":
    print(to_str(remove_nth_from_end(build(1,2,3,4,5), 2)))  # 1→2→3→5
    print(to_str(remove_nth_from_end(build(1), 1)))           # (empty)
    print(to_str(remove_nth_from_end(build(1,2), 1)))         # 1
    print(to_str(remove_nth_from_end(build(1,2,3), 3)))       # 2→3
```

### Output

```
1→2→3→5
(empty)
1
2→3
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why use a dummy head node?**
A: When the node to remove is the head itself (n == list length), `slow` would need to update `head` directly — requiring a special case. With a dummy head, `slow` starts before the real head, so `slow.next = slow.next.next` handles head removal the same way as any other removal.

**Q: How does the gap of n guarantee slow stops at the right place?**
A: After fast is n steps ahead, there are exactly n nodes between slow and fast. When fast reaches the last node (`fast.next == null`), slow is exactly n nodes from the end — meaning `slow.next` is the nth node from the end.

**Q: What if n equals the list length (remove the head)?**
A: Fast advances n steps from dummy to the last node. The while loop doesn't execute because `fast.next == null` immediately. Slow stays at dummy. `dummy.next = dummy.next.next` skips the head. `dummy.next` is the new head.

**Q: Can you solve this in two passes instead?**
A: Yes — first pass counts the length L, second pass stops at node (L - n) and skips the next. Two-pointer achieves the same result in one pass without needing to know L up front.

**Q: What if n is 0 or greater than the list length?**
A: Both are guaranteed invalid by the problem constraints (`1 ≤ n ≤ length`). In a defensive implementation, you could validate and return head unchanged for out-of-range n.

**Q: Why stop when `fast.next == null` rather than `fast == null`?**
A: We need slow positioned one node **before** the target so we can do `slow.next = slow.next.next`. If we moved until `fast == null`, slow would be on the target itself — and we'd have no way to skip it without a back-pointer.


---

9. Summary

## Summary

| Attribute             | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| **Problem**           | Remove the nth node from the end of a linked list                     |
| **Optimal Algorithm** | Two-pointer with dummy head — gap of n between slow and fast          |
| **Time Complexity**   | O(L) — single pass                                                    |
| **Space Complexity**  | O(1)                                                                  |
| **Key Insight**       | Advance fast n steps; then move both — when fast hits end, slow is at the node before the target |

### Core Pattern

> Dummy head + two pointers. Advance fast n steps ahead of slow. Move both until fast is at the last node. slow.next is the target — skip it with `slow.next = slow.next.next`. Return `dummy.next`.

### When to Use This Pattern

- Any "find position from the end" problem on a linked list.
- Related: Middle of Linked List (fast 2× slow), Remove Duplicates, Reorder List.

