# Find Median from Data Stream

---

1. What is meant by Find Median from Data Stream?

## What is Find Median from Data Stream?

**Find Median from Data Stream** is a heap design problem where you must build a data structure that:

- Accepts integers one at a time via `addNum(int num)`
- Returns the current **median** of all numbers seen so far via `findMedian()`

**Median Definition:**

- If the count of numbers is **odd**, the median is the middle element.
- If the count is **even**, the median is the average of the two middle elements.

**Example:**

```
addNum(1)     → stream: [1]          → median = 1.0
addNum(2)     → stream: [1, 2]       → median = 1.5   (avg of 1 and 2)
addNum(3)     → stream: [1, 2, 3]    → median = 2.0
addNum(4)     → stream: [1, 2, 3, 4] → median = 2.5   (avg of 2 and 3)
```

**Why it's hard:** The data arrives one element at a time, and you must answer the median query in O(1) without re-sorting after every insertion.


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Support `addNum(num)` — insert a number into the data structure.
- Support `findMedian()` — return the current median of all inserted numbers as a double.
- Median is the middle value (odd count) or average of two middle values (even count).

### Non-Functional Requirements

| Concern | Expectation |
| --- | --- |
| **Scale** | Up to 5×10⁴ calls total |
| **Latency** | `addNum` in O(log n); `findMedian` in O(1) |
| **Correctness** | Median must be exact (handle even/odd count) |
| **Memory** | O(n) — must store all seen elements |

### Clarifying Questions to Ask

- Can numbers be negative? → Yes
- Are numbers integers only or floats? → Integers (but median can be a float)
- Is `findMedian` called after every `addNum`? → Not necessarily, but it can be
- Is the stream bounded? → Up to 5×10⁴ elements
- Can we assume `findMedian` is never called on an empty stream? → Yes


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Sorted List / Re-sort on Every Insert

Keep a sorted list; compute median by index after each insert.

| | Complexity |
| --- | --- |
| **addNum** | O(n) — insertion into sorted list (shift elements) |
| **findMedian** | O(1) — index directly |
| **Space** | O(n) |

### Approach 2: Two Heaps (Optimal)

Maintain a max-heap for the lower half and a min-heap for the upper half.

| | Complexity |
| --- | --- |
| **addNum** | O(log n) — heap push/pop |
| **findMedian** | O(1) — peek at heap tops |
| **Space** | O(n) |

### Approach 3: Order Statistics Tree (BST-based)

Use a balanced BST (e.g., sortedcontainers.SortedList in Python) to find median by index.

| | Complexity |
| --- | --- |
| **addNum** | O(log n) |
| **findMedian** | O(log n) |
| **Space** | O(n) |

**Best choice:** Two Heaps — O(log n) insert, O(1) median, simple to implement.


---

4. Algorithm Choice and Why

## Algorithm: Two Heaps (Max-Heap + Min-Heap)

### Why Two Heaps?

| Option | addNum | findMedian | Chosen? |
| --- | --- | --- | --- |
| Re-sort array | O(n) | O(1) | No — too slow |
| Binary Insertion (sorted list) | O(n) | O(1) | No — shifting is O(n) |
| Order Statistics Tree | O(log n) | O(log n) | No — more complex |
| **Two Heaps** | **O(log n)** | **O(1)** | **Yes** |

### The Core Insight

Split the stream into two halves at all times:

- **`lo` (max-heap):** lower half — the top is the largest of the small numbers
- **`hi` (min-heap):** upper half — the top is the smallest of the large numbers

**Invariant:** `len(lo) == len(hi)` or `len(lo) == len(hi) + 1`

```
Stream after [1, 2, 3, 4, 5]:

    lo (max-heap)    hi (min-heap)
    [ 3, 2, 1 ]      [ 4, 5 ]
         ↑                ↑
       lo.top           hi.top
    (largest small)  (smallest large)

Median = lo.top = 3  (odd count — middle element)
```

### Rebalancing Rule (Two-Step Insert)

Every `addNum` must preserve the heap invariant:

1. Always push `num` to `lo` (max-heap) first — ensures `lo.top` is a valid median candidate.
2. If `lo.top > hi.top` (order violated) → move `lo.top` to `hi`.
3. If `len(lo) > len(hi) + 1` → move `lo.top` to `hi`.
4. If `len(hi) > len(lo)` → move `hi.top` to `lo`.

### Finding Median

- If sizes equal → median = `(lo.top + hi.top) / 2.0`
- If `lo` is larger → median = `lo.top`


---

5. High-Level Design

## High-Level Design

### Components

1. **Input Layer** — Receives integers one at a time via `addNum`
2. **Max-Heap (`lo`)** — Stores the lower half; Java: negate values to simulate max-heap with PriorityQueue
3. **Min-Heap (`hi`)** — Stores the upper half; standard min-heap
4. **Rebalancer** — Maintains the size invariant after each insertion
5. **Median Reader** — Peeks at heap tops to compute median in O(1)

### Data Flow Diagram

```
addNum(1): push 1→lo.  lo=[1],  hi=[]
           sizes ok.
           findMedian → lo.top = 1.0

addNum(2): push 2→lo.  lo=[2,1], hi=[]
           lo.top(2) > hi.top(∞)? no issue but |lo|>|hi|+1
           → move lo.top(2) → hi. lo=[1], hi=[2]
           findMedian → (1+2)/2 = 1.5

addNum(3): push 3→lo.  lo=[3,1], hi=[2]
           lo.top(3) > hi.top(2) → move 3→hi. lo=[1], hi=[2,3]
           |hi|>|lo| → move hi.top(2)→lo. lo=[2,1], hi=[3]
           findMedian → lo.top = 2.0

addNum(4): push 4→lo.  lo=[4,2,1], hi=[3]
           lo.top(4) > hi.top(3) → move 4→hi. lo=[2,1], hi=[3,4]
           sizes equal.
           findMedian → (2+3)/2 = 2.5
```

### Visual

```
           lo (max-heap)        hi (min-heap)
           ┌────────────┐       ┌────────────┐
           │  2  1      │       │  3  4      │
           │  top=2     │       │  top=3     │
           └────────────┘       └────────────┘
                 ↑                    ↑
          largest in              smallest in
          lower half              upper half

Median = (2 + 3) / 2 = 2.5
```


---

6. Java Solution

## Java Implementation

```java
import java.util.PriorityQueue;

/**
 * MedianFinder — maintains two heaps to find the running median in O(log n) per
 * insertion and O(1) per median query.
 *
 * lo: max-heap (lower half) — simulate with negated values in PriorityQueue
 * hi: min-heap (upper half) — standard PriorityQueue
 *
 * Invariant after each addNum:
 *   lo.size() == hi.size()      → even count, median = avg of tops
 *   lo.size() == hi.size() + 1  → odd count, median = lo.peek()
 */
public class MedianFinder {

    // Max-heap: negate values because Java's PriorityQueue is a min-heap
    private final PriorityQueue<Integer> lo = new PriorityQueue<>((a, b) -> b - a);
    // Min-heap: standard
    private final PriorityQueue<Integer> hi = new PriorityQueue<>();

    /**
     * Adds a number into the data structure.
     *
     * Two-step rebalance:
     *  1. Push to lo; if lo.top > hi.top, move lo.top to hi (fix order).
     *  2. If lo is more than 1 element larger than hi, move lo.top to hi.
     *     If hi is larger than lo, move hi.top to lo.
     *
     * Time: O(log n)
     */
    public void addNum(int num) {
        lo.offer(num);  // always insert into lower-half heap first

        // Maintain order: no element in lo should exceed any element in hi
        if (!hi.isEmpty() && lo.peek() > hi.peek()) {
            hi.offer(lo.poll());
        }

        // Maintain size invariant: lo can have at most one more element than hi
        if (lo.size() > hi.size() + 1) {
            hi.offer(lo.poll());
        } else if (hi.size() > lo.size()) {
            lo.offer(hi.poll());
        }
    }

    /**
     * Returns the median of all numbers added so far.
     *
     * Time: O(1)
     */
    public double findMedian() {
        if (lo.size() == hi.size()) {
            // Even count: average of the two middle elements
            return (lo.peek() + (double) hi.peek()) / 2.0;
        }
        // Odd count: lo has one extra element — it holds the true middle
        return lo.peek();
    }

    // ---- Main: Example usage ----
    public static void main(String[] args) {
        MedianFinder mf = new MedianFinder();

        int[] stream = {1, 2, 3, 4, 5};
        for (int num : stream) {
            mf.addNum(num);
            System.out.printf("addNum(%d) → median = %.1f%n", num, mf.findMedian());
        }

        System.out.println();

        // Example 2: interleaved
        MedianFinder mf2 = new MedianFinder();
        mf2.addNum(6);
        System.out.printf("addNum(6) → median = %.1f%n", mf2.findMedian()); // 6.0
        mf2.addNum(10);
        System.out.printf("addNum(10) → median = %.1f%n", mf2.findMedian()); // 8.0
        mf2.addNum(2);
        System.out.printf("addNum(2) → median = %.1f%n", mf2.findMedian()); // 6.0
        mf2.addNum(6);
        System.out.printf("addNum(6) → median = %.1f%n", mf2.findMedian()); // 6.0
    }
}
```

### Output

```
addNum(1) → median = 1.0
addNum(2) → median = 1.5
addNum(3) → median = 2.0
addNum(4) → median = 2.5
addNum(5) → median = 3.0

addNum(6)  → median = 6.0
addNum(10) → median = 8.0
addNum(2)  → median = 6.0
addNum(6)  → median = 6.0
```


---

7. Python Solution

## Python Implementation

```python
import heapq
from typing import Optional


class MedianFinder:
    """
    Maintains the running median of a number stream using two heaps.

    lo: max-heap (lower half) — Python only has min-heap, so negate values.
    hi: min-heap (upper half) — standard heapq.

    Invariant after every addNum:
      len(lo) == len(hi)      → even count, median = avg of tops
      len(lo) == len(hi) + 1  → odd count,  median = -lo[0]

    addNum:    O(log n)
    findMedian: O(1)
    Space:     O(n)
    """

    def __init__(self):
        self.lo: list = []  # max-heap via negated values
        self.hi: list = []  # min-heap (standard)

    def addNum(self, num: int) -> None:
        """
        Insert a number and rebalance heaps to maintain the invariant.

        Step 1: Push to lo (always enter through the lower half).
        Step 2: Fix ordering — if lo's max exceeds hi's min, move it to hi.
        Step 3: Fix sizes  — lo can be at most 1 larger than hi;
                             hi must never be larger than lo.
        """
        # Push to max-heap (negate to simulate max-heap with heapq)
        heapq.heappush(self.lo, -num)

        # Fix ordering: lo's maximum must not exceed hi's minimum
        if self.hi and (-self.lo[0]) > self.hi[0]:
            heapq.heappush(self.hi, -heapq.heappop(self.lo))

        # Fix sizes
        if len(self.lo) > len(self.hi) + 1:
            heapq.heappush(self.hi, -heapq.heappop(self.lo))
        elif len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def findMedian(self) -> float:
        """
        Return the median in O(1) by peeking at heap tops.
        """
        if len(self.lo) == len(self.hi):
            # Even total: average of the two middle values
            return (-self.lo[0] + self.hi[0]) / 2.0
        # Odd total: lo has the extra (middle) element
        return float(-self.lo[0])


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: sequential stream
    mf = MedianFinder()
    for num in [1, 2, 3, 4, 5]:
        mf.addNum(num)
        print(f"addNum({num}) → median = {mf.findMedian()}")

    print()

    # Example 2: mixed values
    mf2 = MedianFinder()
    for num in [6, 10, 2, 6]:
        mf2.addNum(num)
        print(f"addNum({num}) → median = {mf2.findMedian()}")

    print()

    # Example 3: negative numbers
    mf3 = MedianFinder()
    for num in [-5, -3, -1, 0, 2]:
        mf3.addNum(num)
        print(f"addNum({num}) → median = {mf3.findMedian()}")
```

### Output

```
addNum(1) → median = 1.0
addNum(2) → median = 1.5
addNum(3) → median = 2.0
addNum(4) → median = 2.5
addNum(5) → median = 3.0

addNum(6)  → median = 6.0
addNum(10) → median = 8.0
addNum(2)  → median = 6.0
addNum(6)  → median = 6.0

addNum(-5) → median = -5.0
addNum(-3) → median = -4.0
addNum(-1) → median = -3.0
addNum(0)  → median = -2.0
addNum(2)  → median = -1.0
```


---

8. Interview Q&A

## Q&A

**Q: Why use two heaps instead of one sorted list?**  
A: A sorted list (array) requires O(n) time to insert in sorted order due to element shifting. Two heaps give O(log n) insertion while still providing O(1) median access via the heap tops.

**Q: Why is a max-heap used for the lower half?**  
A: The lower half's most important value is its maximum — the largest of the small numbers. A max-heap exposes that at the top in O(1). Similarly, the upper half's most important value is its minimum, so a min-heap is correct for `hi`.

**Q: Why must Python negate values to simulate a max-heap?**  
A: Python's `heapq` module only supports min-heaps. Negating values flips the ordering: the true maximum becomes the smallest negated value, so it naturally rises to the top of the min-heap.

**Q: What is the rebalancing invariant and why does it hold?**  
A: `len(lo) == len(hi)` (even count) or `len(lo) == len(hi) + 1` (odd count). This ensures all elements in `lo` are ≤ all elements in `hi`, and the middle element is always `lo.top`. We enforce this after every insertion with at most two heap operations.

**Q: What if numbers are inserted in sorted order (worst case)?**  
A: The two-heap approach remains O(log n) per insertion regardless of input order — it does not depend on sortedness.

**Q: Can we use a balanced BST instead?**  
A: Yes — a balanced BST (like Java's `TreeMap` or Python's `sortedcontainers.SortedList`) also gives O(log n) insert and O(log n) median. It's more flexible (supports deletion), but slightly more complex and slower in practice than two heaps for this specific problem.

**Q: How would you handle a `deleteNum` operation?**  
A: This is a harder follow-up. One approach is lazy deletion: mark deleted elements and skip them when peeking. A cleaner solution uses an order statistics tree that supports O(log n) deletion.

**Q: What is the space complexity?**  
A: O(n) — both heaps together store all n elements.


---

9. Summary

## Summary

| Attribute | Value |
| --- | --- |
| **Problem** | Return median of a growing number stream |
| **Optimal Algorithm** | Two Heaps (max-heap + min-heap) |
| **addNum Complexity** | O(log n) |
| **findMedian Complexity** | O(1) |
| **Space Complexity** | O(n) |
| **Key Insight** | Split data at the median; each heap exposes its boundary in O(1) |

### Core Pattern

> **Partition and peek**: split elements into two halves with a max-heap on the left and a min-heap on the right. The median lives at the boundary — always accessible in O(1) by peeking at the two tops.

### Heap Invariant

```
lo (max-heap)    hi (min-heap)
[ ... ≤ M ]      [ M ≤ ... ]
     ↑                ↑
  lo.top           hi.top
  (left middle)  (right middle)

len(lo) == len(hi)     → median = (lo.top + hi.top) / 2
len(lo) == len(hi) + 1 → median = lo.top
```

### When to Use This Pattern

- Running median, sliding window median
- Any problem requiring O(1) access to the "boundary between two halves"
- Stream-based problems where you can't store all elements sorted

