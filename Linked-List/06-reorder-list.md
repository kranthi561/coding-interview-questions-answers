# Reorder List

---

1. What is meant by Reorder List?

## What is Reorder List?

**Reorder List** is a linked list problem where:

- You are given the head of a singly linked list: `L0 → L1 → … → Ln-1 → Ln`.

**Goal:** Reorder it **in place** to: `L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → …`

You must **not** create new nodes — only redirect existing `next` pointers.

**Constraints:**
- `1 <= number of nodes <= 5 × 10⁴`
- `1 <= Node.val <= 1000`

**Example:**

```
Input:  1 → 2 → 3 → 4
Output: 1 → 4 → 2 → 3

Input:  1 → 2 → 3 → 4 → 5
Output: 1 → 5 → 2 → 4 → 3
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Reorder the list in place (no new nodes).
- The pattern interleaves from the front and back, alternating.
- Modify the list; no return value required (void in Java, None in Python).

### Non-Functional Requirements

| Concern         | Expectation                                            |
| --------------- | ------------------------------------------------------ |
| **Scale**       | Up to 5×10⁴ nodes — O(n) required                     |
| **Latency**     | O(n) — three linear passes                             |
| **Space**       | O(1) — in-place pointer manipulation                   |
| **Correctness** | No nodes lost; alternating front-back order maintained |

### Clarifying Questions to Ask

- Must this be in-place? → Yes, no extra list or array.
- Even and odd lengths both valid? → Yes — middle node goes to the center.
- Must we return something? → No, modify the list in place.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Store in Array / Deque

| | Complexity |
| --- | --- |
| **Time** | O(n) — collect all, then reassemble |
| **Space** | O(n) — extra array or deque |

### Approach 2: In-Place Three-Phase (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — three linear passes |
| **Space** | O(1) — only pointer variables |

**In-place three-phase is optimal** — same time, no extra allocation.


---

4. Which Algorithm and Why?

## Algorithm: Find Midpoint → Reverse Second Half → Interleave

### Why Three-Phase?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Array/Deque | O(n) | O(n) | No — extra space |
| **Three-phase in-place** | **O(n)** | **O(1)** | **Yes** |

### Key Insight

The reorder pattern is: take from the front, take from the back, alternate.  
This is equivalent to **merging** the first half with the **reversed** second half.

**Phase 1 — Find midpoint** (slow/fast pointers):
- Slow moves 1 step, fast moves 2 steps → slow stops at midpoint.

**Phase 2 — Reverse the second half** (standard 3-pointer reversal):
- Reverse from `mid.next` onward; cut the connection between halves.

**Phase 3 — Interleave** the two halves:
- Alternate nodes: take one from first half, then one from second half, repeat.

```
List: 1 → 2 → 3 → 4 → 5

Phase 1: mid = 3  (slow/fast)
         first half:  1 → 2 → 3
         second half: 4 → 5

Phase 2: reverse second half:  5 → 4

Phase 3: interleave:
         1 → 5 → 2 → 4 → 3  ✓
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Midpoint Finder** — Slow/fast pointers; slow stops at the middle node.
2. **Second-Half Reverser** — 3-pointer iterative reversal starting from `mid.next`.
3. **Halves Separator** — `mid.next = null` cuts the list into two independent halves.
4. **Interleaver** — Alternates nodes from `first` and `second` halves.
5. **Output** — Modifies the list in place; no return value.

### Data Flow Diagram

```
Input: 1 → 2 → 3 → 4 → 5

─── Phase 1: Find midpoint ───
slow/fast both start at 1.
fast moves 2×, slow moves 1×:
  fast=3,  slow=2
  fast=5,  slow=3   → fast.next==null → stop
mid = slow = 3

─── Phase 2: Reverse second half (starting at mid.next = 4) ───
mid.next = null  (cut: first half = 1→2→3→null)
Reverse 4→5: prev=null, curr=4
  next=5, 4.next=null, prev=4, curr=5
  next=null, 5.next=4, prev=5, curr=null
second = 5→4→null

─── Phase 3: Interleave ───
first = 1→2→3,  second = 5→4

Iteration 1:
  f1=first.next=2,  s1=second.next=4
  first.next = second  →  1→5
  second.next = f1     →  5→2
  first=2, second=4

Iteration 2:
  f1=first.next=3,  s1=second.next=null
  first.next = second  →  2→4
  second.next = f1     →  4→3
  first=3, second=null  → stop

Result: 1→5→2→4→3 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class ReorderList {

    static class ListNode {
        int val; ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Reorder a linked list in place: L0→Ln→L1→Ln-1→...
     *
     * Three phases:
     *   1. Find the midpoint using slow/fast pointers.
     *   2. Reverse the second half in place.
     *   3. Interleave the first and reversed second halves.
     *
     * @param head head of the list (modified in place)
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // Phase 1: find midpoint
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Phase 2: reverse second half
        ListNode second = reverse(slow.next);
        slow.next = null; // cut: first half ends at slow

        // Phase 3: interleave
        ListNode first = head;
        while (second != null) {
            ListNode f1 = first.next;
            ListNode s1 = second.next;

            first.next  = second;  // insert second node after first
            second.next = f1;      // connect it to the rest of first half

            first  = f1;
            second = s1;
        }
    }

    private ListNode reverse(ListNode head) {
        ListNode prev = null, curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
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
        ReorderList sol = new ReorderList();

        // Example 1: even length
        ListNode l1 = build(1,2,3,4);
        sol.reorderList(l1);
        System.out.println(str(l1)); // 1→4→2→3

        // Example 2: odd length
        ListNode l2 = build(1,2,3,4,5);
        sol.reorderList(l2);
        System.out.println(str(l2)); // 1→5→2→4→3

        // Example 3: single node
        ListNode l3 = build(1);
        sol.reorderList(l3);
        System.out.println(str(l3)); // 1

        // Example 4: two nodes
        ListNode l4 = build(1,2);
        sol.reorderList(l4);
        System.out.println(str(l4)); // 1→2
    }
}
```

### Output

```
1→4→2→3
1→5→2→4→3
1
1→2
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val; self.next = next


def reorder_list(head: ListNode | None) -> None:
    """
    Reorder a linked list in place: L0→Ln→L1→Ln-1→...

    Three phases:
      1. Find midpoint via slow/fast pointers.
      2. Reverse the second half in place.
      3. Interleave nodes from first and reversed second halves.

    Args:
        head: Head of the list (modified in place, returns None)

    Time:  O(n)
    Space: O(1)
    """
    if not head or not head.next:
        return

    # Phase 1: find midpoint
    slow, fast = head, head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next

    # Phase 2: reverse second half
    second = _reverse(slow.next)
    slow.next = None  # cut first half

    # Phase 3: interleave
    first = head
    while second:
        f1, s1 = first.next, second.next
        first.next  = second
        second.next = f1
        first, second = f1, s1


def _reverse(node: ListNode | None) -> ListNode | None:
    prev = None
    while node:
        nxt = node.next
        node.next = prev
        prev = node
        node = nxt
    return prev


def build(*vals):
    dummy = ListNode(); cur = dummy
    for v in vals: cur.next = ListNode(v); cur = cur.next
    return dummy.next

def to_str(h):
    parts = []
    while h: parts.append(str(h.val)); h = h.next
    return "→".join(parts) if parts else "(empty)"


if __name__ == "__main__":
    l1 = build(1,2,3,4);   reorder_list(l1); print(to_str(l1))  # 1→4→2→3
    l2 = build(1,2,3,4,5); reorder_list(l2); print(to_str(l2))  # 1→5→2→4→3
    l3 = build(1);          reorder_list(l3); print(to_str(l3))  # 1
    l4 = build(1,2);        reorder_list(l4); print(to_str(l4))  # 1→2
```

### Output

```
1→4→2→3
1→5→2→4→3
1
1→2
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why split at the midpoint rather than using an array to track positions?**
A: An array approach costs O(n) space. Splitting at the midpoint lets us work purely with pointers — after reversing the second half, we know exactly which node to draw from next without any index lookup.

**Q: How does the slow/fast pointer find the correct midpoint?**
A: Fast moves 2× the speed of slow. When fast reaches the last node (for odd-length lists) or the second-to-last (for even-length lists), slow is at the midpoint. For a list of length n, slow ends up at node ⌊n/2⌋, which is where we cut.

**Q: Why cut the list with `slow.next = null` before reversing?**
A: The reverse function traverses until `curr == null`. If we don't cut, it would reverse the full remaining list rather than just the second half. Cutting ensures the two halves are independent before interleaving.

**Q: Why does the interleave loop only check `second != null`?**
A: The first half is always ≥ the second half in length (for even n they're equal; for odd n the first half has the extra middle node). So second will exhaust first or at the same time — we never need to append remaining `first` nodes, they're already connected.

**Q: What happens with a two-node list?**
A: Slow stays at node 1 (midpoint). `slow.next = null` cuts to just node 1. Second half = reversed [node 2] = node 2. Interleave: `1.next = 2`, `2.next = null`. Result: 1→2 — unchanged (already in correct form).

**Q: Can this be done recursively?**
A: Technically yes, but the in-place recursive approach is awkward because you need the end of the list and the current position simultaneously. The three-phase iterative approach is far cleaner and doesn't risk stack overflow for large lists.


---

9. Summary

## Summary

| Attribute             | Value                                                                      |
| --------------------- | -------------------------------------------------------------------------- |
| **Problem**           | Reorder list: L0→Ln→L1→Ln-1→…  in place                                  |
| **Optimal Algorithm** | Find midpoint → reverse second half → interleave both halves               |
| **Time Complexity**   | O(n) — three linear passes                                                 |
| **Space Complexity**  | O(1) — only pointer variables                                              |
| **Key Insight**       | Merging front + reversed-back halves produces the reordered interleaving   |

### Core Pattern

> Three phases: (1) find midpoint with slow/fast pointers, (2) reverse second half with 3-pointer reversal, (3) interleave by alternating one node from each half. All O(n) time, O(1) space.

### When to Use This Pattern

- Any problem requiring simultaneous access to both ends of a linked list.
- Related: Palindrome Linked List (same split+reverse idea), Reverse Nodes in k-Group, Merge Two Sorted Lists.

