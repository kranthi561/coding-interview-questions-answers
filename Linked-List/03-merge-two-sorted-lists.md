# Merge Two Sorted Lists

---

1. What is meant by Merge Two Sorted Lists?

## What is Merge Two Sorted Lists?

**Merge Two Sorted Lists** is a classic linked list problem where:

- You are given the heads of two **sorted** singly linked lists `list1` and `list2`.

**Goal:** Merge them into one **sorted** linked list and return its head. The merged list should be made by **splicing together the nodes** of the two input lists (no new nodes).

**Constraints:**
- `0 <= number of nodes in each list <= 50`
- `-100 <= Node.val <= 100`
- Both lists are sorted in non-decreasing order.

**Example:**

```
Input:  list1 = 1 → 2 → 4, list2 = 1 → 3 → 4
Output: 1 → 1 → 2 → 3 → 4 → 4

Input:  list1 = (empty), list2 = (empty)
Output: (empty)

Input:  list1 = (empty), list2 = 0
Output: 0
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Merge two sorted lists in-place (reuse existing nodes, no new allocations).
- Maintain non-decreasing order in the output.
- Handle empty lists gracefully.

### Non-Functional Requirements

| Concern         | Expectation                                        |
| --------------- | -------------------------------------------------- |
| **Scale**       | Up to 50 nodes each — O(m+n) trivially fast       |
| **Latency**     | O(m + n) — single pass through both lists         |
| **Space**       | O(1) iterative / O(m+n) recursive (call stack)    |
| **Correctness** | All nodes preserved; correct relative ordering    |

### Clarifying Questions to Ask

- Can we create new nodes or must we reuse? → Reuse existing nodes.
- Are lists sorted ascending? → Yes, non-decreasing.
- Can both lists be empty? → Yes — return null.
- Are there duplicate values? → Yes, maintain all of them.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Iterative with Dummy Head

| | Complexity |
| --- | --- |
| **Time** | O(m + n) — traverse both lists once |
| **Space** | O(1) — only a dummy node and a few pointers |

### Approach 2: Recursive

| | Complexity |
| --- | --- |
| **Time** | O(m + n) |
| **Space** | O(m + n) — recursion depth = total nodes |

**Iterative is preferred** for large inputs (no stack overflow risk). Recursive is concise and elegant for interviews.


---

4. Which Algorithm and Why?

## Algorithm: Iterative with Dummy Head Node

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| **Iterative + dummy** | **O(m+n)** | **O(1)** | **Yes** |
| Recursive | O(m+n) | O(m+n) | No — stack overhead |

### Key Insight

A **dummy head** node eliminates the need for special-casing the first node. The tail pointer `curr` always points to the last node added.

```
dummy → (result builds here)
curr = dummy

while list1 and list2:
    if list1.val <= list2.val:
        curr.next = list1
        list1 = list1.next
    else:
        curr.next = list2
        list2 = list2.next
    curr = curr.next

curr.next = list1 OR list2  // attach remaining non-empty list

return dummy.next
```

The remaining list (whichever is non-empty) is already sorted, so we simply attach it at the end.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Dummy Head** — Sentinel node that simplifies result-list construction.
2. **Tail Pointer** (`curr`) — Always points to the last node in the result.
3. **Comparator** — Picks the smaller of the two current heads.
4. **Tail Attacher** — Appends the remaining non-empty list in O(1).
5. **Output** — Returns `dummy.next` (the real head of the merged list).

### Data Flow Diagram

```
list1: 1 → 2 → 4
list2: 1 → 3 → 4

dummy → (result)

Step 1: l1.val=1 == l2.val=1 → pick l1=1  → dummy→1,         l1=2, l2=1
Step 2: l2.val=1 < l1.val=2  → pick l2=1  → dummy→1→1,       l1=2, l2=3
Step 3: l1.val=2 < l2.val=3  → pick l1=2  → dummy→1→1→2,     l1=4, l2=3
Step 4: l2.val=3 < l1.val=4  → pick l2=3  → dummy→1→1→2→3,   l1=4, l2=4
Step 5: l1.val=4 == l2.val=4 → pick l1=4  → dummy→1→1→2→3→4, l1=null, l2=4
Step 6: l1 is null → attach l2=4
Result: 1→1→2→3→4→4 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class MergeTwoSortedLists {

    static class ListNode {
        int val; ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Merge two sorted singly linked lists into one sorted list.
     *
     * Iterative with dummy head:
     *  - Compare heads of both lists; attach the smaller node.
     *  - Advance the pointer of the list we just took from.
     *  - Attach the non-empty remainder directly at the end.
     *
     * @param list1 head of the first sorted list
     * @param list2 head of the second sorted list
     * @return head of the merged sorted list
     *
     * Time:  O(m + n)
     * Space: O(1)
     */
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(0); // sentinel — real result starts at dummy.next
        ListNode curr  = dummy;           // tail of the result list so far

        while (list1 != null && list2 != null) {
            if (list1.val <= list2.val) {
                curr.next = list1;   // take from list1
                list1 = list1.next;
            } else {
                curr.next = list2;   // take from list2
                list2 = list2.next;
            }
            curr = curr.next;        // advance tail
        }

        // Attach whichever list still has nodes (already sorted)
        curr.next = (list1 != null) ? list1 : list2;

        return dummy.next;
    }

    // Recursive version (elegant, O(m+n) stack space)
    public ListNode mergeTwoLists_Recursive(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        if (l1.val <= l2.val) {
            l1.next = mergeTwoLists_Recursive(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists_Recursive(l1, l2.next);
            return l2;
        }
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
        MergeTwoSortedLists sol = new MergeTwoSortedLists();

        System.out.println(str(sol.mergeTwoLists(build(1,2,4), build(1,3,4)))); // 1→1→2→3→4→4
        System.out.println(str(sol.mergeTwoLists(null, null)));                 // (empty)
        System.out.println(str(sol.mergeTwoLists(null, build(0))));             // 0
        System.out.println(str(sol.mergeTwoLists(build(5), build(1,4,7))));    // 1→4→5→7
    }
}
```

### Output

```
1→1→2→3→4→4
(empty)
0
1→4→5→7
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val; self.next = next


def merge_two_lists(list1: ListNode | None, list2: ListNode | None) -> ListNode | None:
    """
    Merge two sorted linked lists into one sorted list (in-place node reuse).

    Iterative with a dummy sentinel head:
      Compare the current heads of both lists, attach the smaller,
      advance that list's pointer. Attach the remaining list at the end.

    Args:
        list1: Head of first sorted list
        list2: Head of second sorted list

    Returns:
        Head of the merged sorted list

    Time:  O(m + n)
    Space: O(1)
    """
    dummy = ListNode()   # sentinel node
    curr  = dummy

    while list1 and list2:
        if list1.val <= list2.val:
            curr.next = list1
            list1 = list1.next
        else:
            curr.next = list2
            list2 = list2.next
        curr = curr.next

    curr.next = list1 or list2  # attach whichever list remains
    return dummy.next


def merge_two_lists_recursive(l1: ListNode | None, l2: ListNode | None) -> ListNode | None:
    """Recursive — O(m+n) time, O(m+n) stack space."""
    if not l1: return l2
    if not l2: return l1
    if l1.val <= l2.val:
        l1.next = merge_two_lists_recursive(l1.next, l2)
        return l1
    else:
        l2.next = merge_two_lists_recursive(l1, l2.next)
        return l2


def build(*vals):
    dummy = ListNode(); cur = dummy
    for v in vals: cur.next = ListNode(v); cur = cur.next
    return dummy.next

def to_str(h):
    parts = []
    while h: parts.append(str(h.val)); h = h.next
    return "→".join(parts) if parts else "(empty)"


if __name__ == "__main__":
    print(to_str(merge_two_lists(build(1,2,4), build(1,3,4)))) # 1→1→2→3→4→4
    print(to_str(merge_two_lists(None, None)))                  # (empty)
    print(to_str(merge_two_lists(None, build(0))))              # 0
    print(to_str(merge_two_lists(build(5), build(1,4,7))))     # 1→4→5→7
```

### Output

```
1→1→2→3→4→4
(empty)
0
1→4→5→7
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why use a dummy head node?**
A: It eliminates special-casing for the first node. Without it, you'd need an if-block before the loop to set the result head. The dummy node lets the loop run uniformly from the start — `dummy.next` is always the real head.

**Q: What happens if one list is empty?**
A: The while loop doesn't execute. `curr.next = list1 or list2` attaches the non-empty list (or null if both are empty). Returns the non-empty list as-is.

**Q: Why don't we need to copy nodes?**
A: Both input lists are already in memory. We simply redirect `next` pointers to weave them together. No new allocations needed — O(1) space.

**Q: How does the recursive version work?**
A: At each call, we pick the smaller head, set its `next` to the result of merging the rest, and return the smaller head. The base cases handle empty lists. Elegant but uses O(m+n) call stack depth.

**Q: How would you extend this to merge k sorted lists?**
A: Use a min-heap (priority queue) that stores `(val, list_id, node)` for the current head of each list. Repeatedly extract the minimum, append it to the result, and push the next node from that list. This is the Merge k Sorted Lists problem — O(N log k) where N = total nodes.

**Q: Can this be adapted for sorted arrays?**
A: Yes — the same two-pointer merge is the core of merge sort's merge step. Compare array elements instead of node values, write to a temporary array.


---

9. Summary

## Summary

| Attribute             | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Problem**           | Merge two sorted linked lists into one sorted list             |
| **Optimal Algorithm** | Iterative with dummy sentinel head                             |
| **Time Complexity**   | O(m + n)                                                       |
| **Space Complexity**  | O(1)                                                           |
| **Key Insight**       | Compare heads, attach smaller, advance that pointer; attach remainder |

### Core Pattern

> Dummy head + tail pointer. Always compare current heads; attach the smaller. When one list is exhausted, append the rest. The dummy node removes the need for a special case on the first node.

### When to Use This Pattern

- Any merge of two sorted sequences.
- Related: Merge k Sorted Lists, Sort List (merge sort), Merge Sorted Array.

