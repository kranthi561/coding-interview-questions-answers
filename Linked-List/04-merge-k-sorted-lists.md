# Merge k Sorted Lists

---

1. What is meant by Merge k Sorted Lists?

## What is Merge k Sorted Lists?

**Merge k Sorted Lists** is a generalization of merging two sorted lists:

- You are given an array of `k` sorted linked list heads.

**Goal:** Merge all `k` lists into one **sorted** linked list and return its head.

**Constraints:**
- `k == lists.length`
- `0 <= k <= 10⁴`
- `0 <= lists[i].length <= 500`
- `-10⁴ <= lists[i][j] <= 10⁴`
- All lists are sorted in non-decreasing order.
- Total nodes ≤ 10⁴.

**Example:**

```
Input:  lists = [1→4→5, 1→3→4, 2→6]
Output: 1→1→2→3→4→4→5→6

Input:  lists = []
Output: (empty)

Input:  lists = [[]]
Output: (empty)
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Merge all k sorted lists into a single sorted list.
- Reuse existing nodes (no new allocations for values).
- Return null if all lists are empty.

### Non-Functional Requirements

| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | Up to 10⁴ lists, 500 nodes each — O(N log k) needed  |
| **Latency**     | O(N log k) — N total nodes, k lists                  |
| **Space**       | O(k) — min-heap holds at most k entries              |
| **Correctness** | Global sorted order across all lists                 |

### Clarifying Questions to Ask

- Can lists be empty? → Yes — skip them.
- All values non-decreasing per list? → Guaranteed.
- Can k be 0 or 1? → Yes — return null or the single list.
- Must nodes be reused? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

Let `N` = total nodes across all lists, `k` = number of lists.

### Approach 1: Brute Force (collect all, sort, rebuild)

| | Complexity |
| --- | --- |
| **Time** | O(N log N) — sorting all values |
| **Space** | O(N) — store all values |

### Approach 2: Compare All k Heads Each Step

| | Complexity |
| --- | --- |
| **Time** | O(N × k) — for each node, scan all k heads |
| **Space** | O(1) |

### Approach 3: Min-Heap (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(N log k) — each push/pop costs O(log k) |
| **Space** | O(k) — heap holds at most k nodes |

### Approach 4: Divide and Conquer (Merge Pairs)

| | Complexity |
| --- | --- |
| **Time** | O(N log k) — log k rounds, each O(N) |
| **Space** | O(log k) — recursion stack |

**Min-Heap and Divide & Conquer both achieve O(N log k)**, which is optimal.


---

4. Which Algorithm and Why?

## Algorithm: Min-Heap (Priority Queue)

### Why Min-Heap?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute force sort | O(N log N) | O(N) | No |
| Compare all k heads | O(Nk) | O(1) | No |
| **Min-Heap** | **O(N log k)** | **O(k)** | **Yes** |
| Divide & Conquer | O(N log k) | O(log k) | Alternative |

### Key Insight

Maintain a **min-heap of size at most k**, one entry per list (the current head of each non-exhausted list).

At each step:
1. Pop the **globally smallest node** from the heap in O(log k).
2. Append it to the result.
3. Push the **next node** from the same list (if it exists) in O(log k).

Total: N pop+push pairs × O(log k) each = **O(N log k)**.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Heap Initializer** — Push the head of each non-null list into the min-heap.
2. **Min-Heap** — Priority queue keyed on node value; stores `(val, list_index, node)`.
3. **Result Builder** — Dummy head + tail pointer; nodes appended in order.
4. **Heap Refiller** — After popping a node, push its `next` (if non-null).
5. **Output** — `dummy.next` — real head of merged list.

### Data Flow Diagram

```
lists = [1→4→5,  1→3→4,  2→6]

Initial heap (val, list_id):  [(1,0,node1), (1,1,node1'), (2,2,node2)]

Step 1: pop (1,0,1)  → result=1,      push (4,0,4)     heap=[(1,1),(2,2),(4,0)]
Step 2: pop (1,1,1)  → result=1→1,    push (3,1,3)     heap=[(2,2),(3,1),(4,0)]
Step 3: pop (2,2,2)  → result=1→1→2,  push (6,2,6)     heap=[(3,1),(4,0),(6,2)]
Step 4: pop (3,1,3)  → result=…→3,    push (4,1,4)     heap=[(4,0),(4,1),(6,2)]
Step 5: pop (4,0,4)  → result=…→4,    push (5,0,5)     heap=[(4,1),(5,0),(6,2)]
Step 6: pop (4,1,4)  → result=…→4,    list1 exhausted  heap=[(5,0),(6,2)]
Step 7: pop (5,0,5)  → result=…→5,    list0 exhausted  heap=[(6,2)]
Step 8: pop (6,2,6)  → result=…→6,    list2 exhausted  heap=[]

Output: 1→1→2→3→4→4→5→6 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class MergeKSortedLists {

    static class ListNode {
        int val; ListNode next;
        ListNode(int val) { this.val = val; }
    }

    /**
     * Merge k sorted linked lists using a min-heap.
     *
     * Insert the head of each list into a min-heap (keyed by node value).
     * Repeatedly extract the minimum node, attach it to the result,
     * and push the next node from the same list into the heap.
     *
     * @param lists array of k sorted linked list heads
     * @return head of the merged sorted list
     *
     * Time:  O(N log k) — N total nodes, k lists
     * Space: O(k) — heap size
     */
    public ListNode mergeKLists(ListNode[] lists) {
        // Min-heap ordered by node value; use list index as tie-breaker
        PriorityQueue<ListNode> heap = new PriorityQueue<>(
            (a, b) -> a.val - b.val
        );

        // Seed heap with first node of each non-empty list
        for (ListNode head : lists) {
            if (head != null) heap.offer(head);
        }

        ListNode dummy = new ListNode(0);
        ListNode curr  = dummy;

        while (!heap.isEmpty()) {
            ListNode node = heap.poll();    // globally smallest node
            curr.next = node;               // append to result
            curr = curr.next;

            if (node.next != null) {        // push next node from the same list
                heap.offer(node.next);
            }
        }

        return dummy.next;
    }

    // Divide & Conquer alternative
    public ListNode mergeKLists_DC(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        return mergeRange(lists, 0, lists.length - 1);
    }

    private ListNode mergeRange(ListNode[] lists, int lo, int hi) {
        if (lo == hi) return lists[lo];
        int mid = (lo + hi) / 2;
        ListNode left  = mergeRange(lists, lo, mid);
        ListNode right = mergeRange(lists, mid + 1, hi);
        return mergeTwoLists(left, right);
    }

    private ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0); ListNode cur = dummy;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) { cur.next = l1; l1 = l1.next; }
            else                  { cur.next = l2; l2 = l2.next; }
            cur = cur.next;
        }
        cur.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }

    private static ListNode build(int... v) {
        ListNode d = new ListNode(0); ListNode c = d;
        for (int x : v) { c.next = new ListNode(x); c = c.next; }
        return d.next;
    }

    private static String str(ListNode h) {
        StringBuilder sb = new StringBuilder();
        while (h != null) { sb.append(h.val); if (h.next != null) sb.append("→"); h = h.next; }
        return sb.length() == 0 ? "(empty)" : sb.toString();
    }

    public static void main(String[] args) {
        MergeKSortedLists sol = new MergeKSortedLists();

        // Example 1: three lists
        ListNode[] lists1 = { build(1,4,5), build(1,3,4), build(2,6) };
        System.out.println(str(sol.mergeKLists(lists1))); // 1→1→2→3→4→4→5→6

        // Example 2: empty input
        System.out.println(str(sol.mergeKLists(new ListNode[]{}))); // (empty)

        // Example 3: one empty list
        System.out.println(str(sol.mergeKLists(new ListNode[]{ null }))); // (empty)

        // Example 4: single list
        System.out.println(str(sol.mergeKLists(new ListNode[]{ build(1,2,3) }))); // 1→2→3

        // Divide & Conquer version
        ListNode[] lists2 = { build(1,4,5), build(1,3,4), build(2,6) };
        System.out.println("D&C: " + str(sol.mergeKLists_DC(lists2))); // 1→1→2→3→4→4→5→6
    }
}
```

### Output

```
1→1→2→3→4→4→5→6
(empty)
(empty)
1→2→3
D&C: 1→1→2→3→4→4→5→6
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from __future__ import annotations
import heapq

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val; self.next = next


def merge_k_lists(lists: list[ListNode | None]) -> ListNode | None:
    """
    Merge k sorted linked lists into one sorted list using a min-heap.

    Push the head of each non-empty list into a min-heap.
    Repeatedly extract the smallest node, append to result,
    push its successor into the heap.

    Args:
        lists: List of k sorted linked list heads

    Returns:
        Head of the merged sorted list

    Time:  O(N log k) — N total nodes, k lists
    Space: O(k) — heap size
    """
    dummy = ListNode()
    curr  = dummy
    heap  = []  # (val, tie_breaker_idx, node)

    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))

    idx = len(lists)  # unique tie-breaker for same-value nodes

    while heap:
        val, _, node = heapq.heappop(heap)
        curr.next = node
        curr = curr.next
        if node.next:
            heapq.heappush(heap, (node.next.val, idx, node.next))
            idx += 1

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
    print(to_str(merge_k_lists([build(1,4,5), build(1,3,4), build(2,6)])))
    # 1→1→2→3→4→4→5→6

    print(to_str(merge_k_lists([])))          # (empty)
    print(to_str(merge_k_lists([None])))      # (empty)
    print(to_str(merge_k_lists([build(1,2,3)])))  # 1→2→3
```

### Output

```
1→1→2→3→4→4→5→6
(empty)
(empty)
1→2→3
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why is the min-heap size bounded by k, not N?**
A: We only ever push one node per list at a time — the current "frontier" of that list. So the heap contains at most one node from each of the k lists, keeping its size ≤ k regardless of N.

**Q: Why not just merge lists pairwise sequentially (list1+list2, then result+list3…)?**
A: Sequential merging processes the first list O(k) times instead of O(log k) times, giving O(Nk) total. The heap and divide-and-conquer approaches reduce this to O(N log k).

**Q: How does Divide & Conquer achieve O(N log k)?**
A: It merges lists in a tournament-bracket style: merge pairs → halve the number of lists → repeat. After log₂k rounds, one list remains. Each round processes N nodes total, giving O(N log k).

**Q: What is the tie-breaking index in the heap for?**
A: Python's `heapq` compares tuples element by element. If two nodes have the same `val`, it would try to compare `ListNode` objects — which raises a TypeError. The unique index as the second element breaks ties without comparing nodes.

**Q: How does this compare to the brute force sort approach?**
A: Brute force collects all N values into a list, sorts in O(N log N), and rebuilds. Min-heap is O(N log k) which is strictly better when k << N (few lists, many nodes each). For k = N (each list has one node), both reduce to O(N log N).

**Q: Can this problem be solved in O(N) time?**
A: Not with comparison-based methods — any comparison sort requires Ω(N log N). However, if values are bounded integers, counting sort could do O(N + range). In practice, O(N log k) is the standard optimal.


---

9. Summary

## Summary

| Attribute             | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Problem**           | Merge k sorted linked lists into one sorted list                  |
| **Optimal Algorithm** | Min-Heap (Priority Queue) — one entry per list                   |
| **Time Complexity**   | O(N log k) — N total nodes, k lists                             |
| **Space Complexity**  | O(k) — heap size                                                 |
| **Key Insight**       | Heap always yields the globally smallest unprocessed node in O(log k) |

### Core Pattern

> Seed a min-heap with the head of every list. Repeatedly: pop the min, append to result, push its successor. Heap size stays ≤ k throughout — O(log k) per node = O(N log k) total.

### When to Use This Pattern

- Merging multiple sorted streams/lists efficiently.
- Related: Merge Two Sorted Lists, Sort List, Find K Pairs with Smallest Sums, External Sort.

