# Reverse Linked List

---

1. What is meant by Reverse Linked List?

## What is Reverse Linked List?

**Reverse Linked List** is a foundational linked list problem where:

- You are given the head of a **singly linked list**.
- Each node has a `val` and a `next` pointer.

**Goal:** Reverse the list **in place** and return the **new head** (the old tail).

**Constraints:**
- `0 <= Number of nodes <= 5000`
- `-5000 <= Node.val <= 5000`

**Example:**

```
Input:  1 → 2 → 3 → 4 → 5 → null
Output: 5 → 4 → 3 → 2 → 1 → null

Input:  1 → 2 → null
Output: 2 → 1 → null

Input:  null
Output: null
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Reverse the direction of all `next` pointers.
- The original tail becomes the new head.
- The original head's `next` becomes `null`.
- Return the new head.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | Up to 5000 nodes — O(n) trivially fast              |
| **Latency**     | O(n) — single pass                                  |
| **Space**       | O(1) for iterative; O(n) for recursive (call stack) |
| **Correctness** | Must not lose any nodes; no cycles in result        |

### Clarifying Questions to Ask

- Singly or doubly linked list? → Singly (one direction).
- Reverse in place or return a new list? → In place, return new head.
- Can the list be empty? → Yes — return null.
- Can there be cycles? → No, assume standard linked list.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Iterative (Three Pointers)

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit each node once |
| **Space** | O(1) — only `prev`, `curr`, `next` variables |

### Approach 2: Recursive

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit each node once |
| **Space** | O(n) — recursion call stack depth = list length |

**Iterative is preferred** in production for large lists (avoids stack overflow). Recursive is elegant and useful in interviews to demonstrate understanding.


---

4. Which Algorithm and Why?

## Algorithm: Iterative Three-Pointer Reversal

### Why Iterative?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Iterative (3 pointers)** | **O(n)** | **O(1)** | **Yes** |
| Recursive | O(n) | O(n) | No — stack overhead |
| Stack-based | O(n) | O(n) | No — extra space |

### Key Insight

At each step, redirect `curr.next` from the next node to the previous node.  
Use three pointers to avoid losing the reference to the rest of the list:

```
prev = null
curr = head

while curr != null:
    next = curr.next   // save the next node before we overwrite it
    curr.next = prev   // reverse the pointer
    prev = curr        // move prev forward
    curr = next        // move curr forward

return prev            // prev is the new head when curr reaches null
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Three-Pointer State** — `prev` (new reversed part), `curr` (current node), `next` (saved future node).
2. **Pointer Reverser** — Redirects `curr.next = prev` at each step.
3. **Advancement Step** — Slides `prev` and `curr` forward by one each iteration.
4. **Output** — Returns `prev` (the new head when `curr` becomes null).

### Data Flow Diagram

```
Initial:  null ← (prev)   1 → 2 → 3 → 4 → 5 → null
                           ↑
                         (curr)

Step 1: save next=2, 1.next=null(prev), prev=1, curr=2
         null ← 1   2 → 3 → 4 → 5 → null
                ↑   ↑
              (prev)(curr)

Step 2: save next=3, 2.next=1, prev=2, curr=3
         null ← 1 ← 2   3 → 4 → 5 → null
                      ↑  ↑
                   (prev)(curr)

Step 3: → null ← 1 ← 2 ← 3   4 → 5
Step 4: → null ← 1 ← 2 ← 3 ← 4   5
Step 5: → null ← 1 ← 2 ← 3 ← 4 ← 5   curr=null

return prev = 5 (new head)
Result: 5 → 4 → 3 → 2 → 1 → null ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class ReverseLinkedList {

    static class ListNode {
        int val;
        ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Reverse a singly linked list in place.
     *
     * Iterative three-pointer approach:
     *   prev  = tracks the already-reversed portion (new head grows here)
     *   curr  = node currently being processed
     *   next  = saved reference to the next node before we overwrite curr.next
     *
     * @param head head of the input list
     * @return new head (old tail) after reversal
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;

        while (curr != null) {
            ListNode next = curr.next; // save next before overwriting
            curr.next = prev;          // reverse the pointer
            prev = curr;               // advance prev
            curr = next;               // advance curr
        }

        return prev; // prev is the new head when curr == null
    }

    // Recursive version (elegant, but O(n) stack space)
    public ListNode reverseList_Recursive(ListNode head) {
        if (head == null || head.next == null) return head; // base case

        ListNode newHead = reverseList_Recursive(head.next); // reverse the rest
        head.next.next = head; // make the next node point back to current
        head.next = null;      // remove the original forward pointer

        return newHead;
    }

    // Helper: build list from array
    private static ListNode build(int... vals) {
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        for (int v : vals) { cur.next = new ListNode(v); cur = cur.next; }
        return dummy.next;
    }

    // Helper: list to string
    private static String str(ListNode head) {
        StringBuilder sb = new StringBuilder();
        while (head != null) {
            sb.append(head.val);
            if (head.next != null) sb.append(" → ");
            head = head.next;
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        ReverseLinkedList sol = new ReverseLinkedList();

        // Example 1: standard 5-node list
        System.out.println("Input:  " + str(build(1,2,3,4,5)));
        System.out.println("Output: " + str(sol.reverseList(build(1,2,3,4,5)))); // 5→4→3→2→1

        // Example 2: two nodes
        System.out.println("\nInput:  " + str(build(1,2)));
        System.out.println("Output: " + str(sol.reverseList(build(1,2)))); // 2→1

        // Example 3: single node
        System.out.println("\nInput:  " + str(build(1)));
        System.out.println("Output: " + str(sol.reverseList(build(1)))); // 1

        // Example 4: empty
        System.out.println("\nInput:  (empty)");
        System.out.println("Output: " + str(sol.reverseList(null))); // (empty)

        // Recursive version
        System.out.println("\nRecursive [1,2,3,4,5]: " +
            str(sol.reverseList_Recursive(build(1,2,3,4,5)))); // 5→4→3→2→1
    }
}
```

### Output

```
Input:  1 → 2 → 3 → 4 → 5
Output: 5 → 4 → 3 → 2 → 1

Input:  1 → 2
Output: 2 → 1

Input:  1
Output: 1

Input:  (empty)
Output:

Recursive [1,2,3,4,5]: 5 → 4 → 3 → 2 → 1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class ListNode:
    def __init__(self, val: int = 0, next: ListNode | None = None):
        self.val = val
        self.next = next


def reverse_list(head: ListNode | None) -> ListNode | None:
    """
    Reverse a singly linked list in place.

    Iterative three-pointer approach:
      prev  = reversed portion (grows left-to-right into the result)
      curr  = node being processed
      next  = saved next node before we redirect curr.next

    Args:
        head: Head of the input linked list

    Returns:
        New head of the reversed list

    Time:  O(n)
    Space: O(1)
    """
    prev, curr = None, head

    while curr:
        nxt = curr.next   # save next
        curr.next = prev  # reverse pointer
        prev = curr       # advance prev
        curr = nxt        # advance curr

    return prev  # new head


def reverse_list_recursive(head: ListNode | None) -> ListNode | None:
    """Recursive version — O(n) time, O(n) space (call stack)."""
    if not head or not head.next:
        return head

    new_head = reverse_list_recursive(head.next)
    head.next.next = head  # make next node point back
    head.next = None       # cut original forward link

    return new_head


# ── Helpers ──────────────────────────────────────────────────────────────
def build(*vals: int) -> ListNode | None:
    dummy = ListNode()
    cur = dummy
    for v in vals:
        cur.next = ListNode(v)
        cur = cur.next
    return dummy.next


def to_str(head: ListNode | None) -> str:
    parts = []
    while head:
        parts.append(str(head.val))
        head = head.next
    return " → ".join(parts) if parts else "(empty)"


if __name__ == "__main__":
    cases = [
        build(1, 2, 3, 4, 5),
        build(1, 2),
        build(1),
        None,
    ]

    for lst in cases:
        print(f"Input:  {to_str(lst)}")
        print(f"Output: {to_str(reverse_list(lst))}\n")
```

### Output

```
Input:  1 → 2 → 3 → 4 → 5
Output: 5 → 4 → 3 → 2 → 1

Input:  1 → 2
Output: 2 → 1

Input:  1
Output: 1

Input:  (empty)
Output: (empty)
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why do we save `next` before redirecting `curr.next`?**
A: Once we set `curr.next = prev`, the reference to the rest of the list is lost. Saving `next = curr.next` beforehand preserves that reference so we can advance `curr` to the next node.

**Q: What is the base case for recursion?**
A: `head == null` (empty list) or `head.next == null` (single node). In both cases, the node is already "reversed" — return it as the new head.

**Q: How does recursive reversal work?**
A: It reverses the tail first (recursing to the end), then on the way back makes each node's successor point back to it: `head.next.next = head`, and cuts the original forward link: `head.next = null`.

**Q: Can the iterative approach be done with only two pointers?**
A: No — you always need three: one to track the previous node (for redirecting), one for the current node, and one to save the next node before overwriting. Two variables would lose track of either the reversed portion or the remaining list.

**Q: What if the list has only one node?**
A: The while loop doesn't execute (curr is the only node, next is null). After one iteration: prev=head, curr=null → returns head immediately. Correct.

**Q: What is the difference between reversing a singly vs. doubly linked list?**
A: In a doubly linked list, each node has both `next` and `prev` pointers. Reversal requires swapping both on every node, plus updating the head. Singly linked lists only need the `next` pointer redirected.


---

9. Summary

## Summary

| Attribute             | Value                                                       |
| --------------------- | ----------------------------------------------------------- |
| **Problem**           | Reverse a singly linked list in place                       |
| **Optimal Algorithm** | Iterative three-pointer reversal                            |
| **Time Complexity**   | O(n)                                                        |
| **Space Complexity**  | O(1) iterative / O(n) recursive                            |
| **Key Insight**       | Save `next`, redirect `curr.next = prev`, advance both     |

### Core Pattern

> Three pointers: `prev` (reversed side), `curr` (current), `next` (saved). At each step, flip the arrow and slide forward. When `curr` is null, `prev` is the new head.

### When to Use This Pattern

- Any in-place linked list reversal or partial reversal.
- Related: Reverse Nodes in k-Group, Palindrome Linked List, Reorder List.

