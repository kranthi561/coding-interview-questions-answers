# Linked List Cycle

---

1. What is meant by Linked List Cycle?

## What is Linked List Cycle?

**Linked List Cycle** is a detection problem where:

- You are given the head of a linked list.
- Some node's `next` pointer may point back to an earlier node, creating a cycle.

**Goal:** Return `true` if the linked list has a cycle, `false` otherwise.

**Constraints:**
- `0 <= number of nodes <= 10⁴`
- `-10⁵ <= Node.val <= 10⁵`
- `pos` is -1 (no cycle) or a valid index where the tail connects.

**Example:**

```
Input:  3 → 2 → 0 → -4 → (back to node at index 1)
Output: true

Input:  1 → 2 → (back to node at index 0)
Output: true

Input:  1 → null
Output: false
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Detect whether any cycle exists in the list.
- Return a boolean — do not need to find cycle entry point (that is Linked List Cycle II).
- Must not modify the list.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | Up to 10⁴ nodes — O(n) is fine                     |
| **Latency**     | O(n) — single traversal                              |
| **Space**       | O(1) — Floyd's algorithm needs no extra storage     |
| **Correctness** | Must detect all cycles including one-node self-loops |

### Clarifying Questions to Ask

- Do we need the entry point? → No, just detect presence.
- Can we modify the list? → Ideally no (but marking visited is an alternative approach).
- Can there be a self-loop (node.next == node)? → Yes, handle it.
- Is the list singly linked? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: HashSet of Visited Nodes

| | Complexity |
| --- | --- |
| **Time** | O(n) — visit each node once |
| **Space** | O(n) — store all visited node references |

### Approach 2: Floyd's Cycle Detection (Tortoise & Hare)

| | Complexity |
| --- | --- |
| **Time** | O(n) — slow and fast meet within one loop cycle |
| **Space** | O(1) — only two pointer variables |

**Floyd's algorithm is preferred** — same time complexity but O(1) space, no heap allocation.


---

4. Which Algorithm and Why?

## Algorithm: Floyd's Cycle Detection (Tortoise and Hare)

### Why Floyd's?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| HashSet | O(n) | O(n) | No — extra space |
| Modify node values | O(n) | O(1) | No — destructive |
| **Floyd's (slow/fast)** | **O(n)** | **O(1)** | **Yes** |

### Key Insight

Use two pointers:
- **Slow** moves 1 step at a time.
- **Fast** moves 2 steps at a time.

If there is **no cycle**: fast reaches `null` before meeting slow.  
If there is **a cycle**: fast laps slow inside the cycle — they must eventually meet at the same node.

**Why they always meet in a cycle?**  
Each step, the gap between slow and fast decreases by 1 (relative to cycle length). Starting from any gap, they converge in at most `cycle_length` steps.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Null Guard** — Return false if list is empty or has one non-cyclic node.
2. **Slow Pointer** — Advances 1 step per iteration.
3. **Fast Pointer** — Advances 2 steps per iteration.
4. **Collision Detector** — Returns true when `slow == fast`.
5. **Terminator** — Returns false when `fast` or `fast.next` becomes null.

### Data Flow Diagram

```
List: 3 → 2 → 0 → -4 → (back to 2, index 1)

Start: slow=3, fast=3

Step 1: slow=2,   fast=0    (slow+1, fast+2)
Step 2: slow=0,   fast=2    (slow+1, fast+2, fast wraps via cycle)
Step 3: slow=-4,  fast=-4   ← MEET! → cycle detected → return true ✓

---

No-cycle list: 1 → 2 → 3 → null

Step 1: slow=2, fast=3
Step 2: slow=3, fast=null  → fast.next == null → return false ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class LinkedListCycle {

    static class ListNode {
        int val;
        ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Detect if a linked list has a cycle using Floyd's algorithm.
     *
     * Slow moves 1 step; fast moves 2 steps.
     * If no cycle: fast reaches null.
     * If cycle: fast laps slow — they meet at the same node.
     *
     * @param head head of the linked list
     * @return true if a cycle exists, false otherwise
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;       // 1 step
            fast = fast.next.next;  // 2 steps
            if (slow == fast) return true; // pointers met → cycle!
        }

        return false; // fast reached end → no cycle
    }

    // HashSet alternative (O(n) space)
    public boolean hasCycle_HashSet(ListNode head) {
        java.util.Set<ListNode> visited = new java.util.HashSet<>();
        ListNode curr = head;
        while (curr != null) {
            if (!visited.add(curr)) return true; // already seen this node
            curr = curr.next;
        }
        return false;
    }

    public static void main(String[] args) {
        LinkedListCycle sol = new LinkedListCycle();

        // Example 1: cycle at pos 1  (3→2→0→-4→back to 2)
        ListNode n1 = new ListNode(3);
        ListNode n2 = new ListNode(2);
        ListNode n3 = new ListNode(0);
        ListNode n4 = new ListNode(-4);
        n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n2; // cycle
        System.out.println("Cycle at pos 1: " + sol.hasCycle(n1)); // true

        // Example 2: cycle at pos 0  (1→2→back to 1)
        ListNode a = new ListNode(1);
        ListNode b = new ListNode(2);
        a.next = b; b.next = a; // cycle
        System.out.println("Cycle at pos 0: " + sol.hasCycle(a)); // true

        // Example 3: no cycle  (1→null)
        ListNode single = new ListNode(1);
        System.out.println("No cycle (1 node): " + sol.hasCycle(single)); // false

        // Example 4: empty
        System.out.println("Empty list: " + sol.hasCycle(null)); // false

        // Example 5: self-loop
        ListNode self = new ListNode(5);
        self.next = self;
        System.out.println("Self-loop: " + sol.hasCycle(self)); // true
    }
}
```

### Output

```
Cycle at pos 1: true
Cycle at pos 0: true
No cycle (1 node): false
Empty list: false
Self-loop: true
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


def has_cycle(head: ListNode | None) -> bool:
    """
    Detect if a linked list has a cycle using Floyd's Cycle Detection.

    Slow pointer moves 1 step; fast pointer moves 2 steps.
    If no cycle: fast reaches None (end of list).
    If cycle: fast laps slow inside the cycle — they meet at the same node.

    Args:
        head: Head of the linked list

    Returns:
        True if cycle exists, False otherwise

    Time:  O(n)
    Space: O(1)
    """
    slow = fast = head

    while fast and fast.next:
        slow = slow.next         # 1 step
        fast = fast.next.next    # 2 steps
        if slow is fast:
            return True          # met inside cycle

    return False  # fast reached end → no cycle


def has_cycle_hashset(head: ListNode | None) -> bool:
    """HashSet alternative — O(n) space."""
    visited = set()
    curr = head
    while curr:
        if id(curr) in visited:
            return True
        visited.add(id(curr))
        curr = curr.next
    return False


if __name__ == "__main__":
    # Example 1: cycle  3→2→0→-4→(back to 2)
    n1, n2, n3, n4 = ListNode(3), ListNode(2), ListNode(0), ListNode(-4)
    n1.next = n2; n2.next = n3; n3.next = n4; n4.next = n2
    print(f"Cycle at pos 1:   {has_cycle(n1)}")   # True

    # Example 2: cycle  1→2→(back to 1)
    a, b = ListNode(1), ListNode(2)
    a.next = b; b.next = a
    print(f"Cycle at pos 0:   {has_cycle(a)}")    # True

    # Example 3: no cycle
    single = ListNode(1)
    print(f"Single node:      {has_cycle(single)}")  # False

    # Example 4: empty
    print(f"Empty list:       {has_cycle(None)}")    # False

    # Example 5: self-loop
    s = ListNode(5); s.next = s
    print(f"Self-loop:        {has_cycle(s)}")       # True
```

### Output

```
Cycle at pos 1:   True
Cycle at pos 0:   True
Single node:      False
Empty list:       False
Self-loop:        True
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why does Floyd's algorithm guarantee the pointers will meet if a cycle exists?**
A: Once both pointers are inside the cycle, fast gains on slow by 1 step per iteration (relative speed = 1). Starting from any distance, after at most `cycle_length` steps the gap closes to 0 — they must meet.

**Q: Why check `fast != null && fast.next != null`?**
A: Fast moves 2 steps, so we must ensure both `fast` and `fast.next` exist before accessing `fast.next.next`. Checking only `fast != null` would cause a null pointer error on `fast.next.next` when `fast.next` is null.

**Q: Can Floyd's algorithm find WHERE the cycle starts?**
A: Yes (Linked List Cycle II). After slow and fast meet, reset one pointer to head. Move both one step at a time — they meet again at the cycle entry point. This works because of a mathematical property of the meeting point within the cycle.

**Q: Why not just use a HashSet?**
A: HashSet uses O(n) extra space. Floyd's algorithm detects cycles in O(1) space, which is a stronger result. For very large lists, the space difference matters.

**Q: What if the list has only one node with no self-loop?**
A: `fast = head`, `fast.next = null` → the while condition `fast.next != null` fails immediately → return false. Correct.

**Q: What if fast and slow start at the same node — can they "meet" before completing even one step?**
A: No — the check `if slow == fast` is placed after both pointers advance. They start at head but the check only fires after at least one move.


---

9. Summary

## Summary

| Attribute             | Value                                                           |
| --------------------- | --------------------------------------------------------------- |
| **Problem**           | Detect whether a linked list contains a cycle                   |
| **Optimal Algorithm** | Floyd's Cycle Detection (Tortoise and Hare)                     |
| **Time Complexity**   | O(n)                                                            |
| **Space Complexity**  | O(1)                                                            |
| **Key Insight**       | Slow (×1) and fast (×2) pointers must meet in a cycle          |

### Core Pattern

> Two pointers at different speeds. In a cycle, fast laps slow — they must meet. Without a cycle, fast falls off the end. No extra memory needed.

### When to Use This Pattern

- Cycle detection in any linked structure.
- Related: Linked List Cycle II (find entry), Find Duplicate Number (Floyd on array), Happy Number.

