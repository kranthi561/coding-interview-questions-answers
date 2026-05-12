# Non-overlapping Intervals

---

1. What is meant by Non-overlapping Intervals?

## What is Non-overlapping Intervals?

**Non-overlapping Intervals** is a greedy interval scheduling problem where:

- You are given an array of intervals `intervals[i] = [starti, endi]`.

**Goal:** Find the **minimum number of intervals to remove** so that the remaining intervals are non-overlapping.

**Constraints:**
- `1 <= intervals.length <= 10⁵`
- `-5 × 10⁴ <= starti < endi <= 5 × 10⁴`

**Example:**

```
Input:  intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1
Explanation: Remove [1,3] — remaining [[1,2],[2,3],[3,4]] are non-overlapping.

Input:  intervals = [[1,2],[1,2],[1,2]]
Output: 2
Explanation: Remove 2 of the 3 identical intervals.
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the **minimum count** of intervals to remove (not which ones).
- After removal, no two remaining intervals should overlap.
- Touching intervals (e.g., `[1,2]` and `[2,3]`) are NOT overlapping.

### Non-Functional Requirements

| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | Up to 10⁵ intervals — O(n log n) required            |
| **Latency**     | O(n log n) — dominated by sort                        |
| **Correctness** | Must find the global minimum, not a local greedy min |
| **Edge Cases**  | Single interval → 0 removals; all same → n-1 removals |

### Clarifying Questions to Ask

- Do touching endpoints count as overlapping? → No, `[1,2]` and `[2,3]` are fine.
- Return count or the actual intervals to remove? → Count only.
- Can intervals be negative? → Yes.
- Is the input sorted? → No.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: DP (Longest Non-overlapping Subsequence)

| | Complexity |
| --- | --- |
| **Time** | O(n²) — for each interval, check all prior non-overlapping ones |
| **Space** | O(n) — dp array |

Answer = `n - LIS_length` where LIS = longest non-overlapping subsequence.

### Approach 2: Greedy (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sort + single pass |
| **Space** | O(1) — only tracking last end time |

**Why O(n log n)?** Sorting is the dominant cost. The greedy scan is O(n).


---

4. Which Algorithm and Why?

## Algorithm: Greedy — Sort by End Time, Keep Earliest-Ending

### Why Greedy?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| DP (LIS variant) | O(n²) | O(n) | No |
| **Greedy (sort by end)** | **O(n log n)** | **O(1)** | **Yes** |

### Key Insight

This is the classic **Activity Selection Problem**. Greedy proof:

> Always keep the interval that **ends earliest** among conflicting intervals.  
> Why? An interval ending earlier leaves more room for future intervals, maximizing the number we can keep (minimizing removals).

**Sort by end time.** Track `lastEnd` (end of the last kept interval):
- If `current.start >= lastEnd` → no overlap → **keep** it, update `lastEnd`.
- If `current.start < lastEnd` → overlap → **remove** it (increment counter).

**Why sort by end (not start)?**  
Sorting by end ensures that among overlapping intervals, we always keep the one that ends soonest, which is the greedy optimal choice.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Sorter** — Sort intervals by end time.
2. **Last-End Tracker** — Track the end time of the last kept interval.
3. **Conflict Detector** — Compare current start vs. last end.
4. **Removal Counter** — Increment when a conflict (overlap) is found.
5. **Output** — Return the removal count.

### Data Flow Diagram

```
intervals = [[1,2],[2,3],[3,4],[1,3]]

After sort by end: [[1,2],[2,3],[1,3],[3,4]]

lastEnd = -∞, removals = 0

[1,2]: start=1 >= -∞ → keep   → lastEnd=2
[2,3]: start=2 >= 2  → keep   → lastEnd=3
[1,3]: start=1 <  3  → overlap → remove! removals=1
[3,4]: start=3 >= 3  → keep   → lastEnd=4

Output: 1

---

intervals = [[1,2],[1,2],[1,2]]
After sort by end: [[1,2],[1,2],[1,2]]
lastEnd=-∞
[1,2]: keep → lastEnd=2
[1,2]: start=1 < 2 → remove! removals=1
[1,2]: start=1 < 2 → remove! removals=2
Output: 2
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class NonOverlappingIntervals {

    /**
     * Find the minimum number of intervals to remove to make the rest non-overlapping.
     *
     * Greedy (Activity Selection):
     *  Sort by end time → always keep the interval ending earliest.
     *  If an interval overlaps the last kept one, remove it (count++).
     *  Otherwise, keep it and update lastEnd.
     *
     * @param intervals array of [start, end] intervals
     * @return minimum number of intervals to remove
     *
     * Time:  O(n log n) — sort + O(n) scan
     * Space: O(1) — no extra storage beyond sort
     */
    public int eraseOverlapIntervals(int[][] intervals) {
        // Sort by end time — greedy choice: keep the earliest-ending interval
        Arrays.sort(intervals, (a, b) -> a[1] - b[1]);

        int removals = 0;
        int lastEnd = Integer.MIN_VALUE; // end of the last kept interval

        for (int[] interval : intervals) {
            if (interval[0] >= lastEnd) {
                // No overlap — keep this interval
                lastEnd = interval[1];
            } else {
                // Overlap — remove this interval (keep the one ending earlier)
                removals++;
            }
        }

        return removals;
    }

    public static void main(String[] args) {
        NonOverlappingIntervals sol = new NonOverlappingIntervals();

        // Example 1: Remove 1
        System.out.println("[[1,2],[2,3],[3,4],[1,3]] → " +
            sol.eraseOverlapIntervals(new int[][]{{1,2},{2,3},{3,4},{1,3}})); // 1

        // Example 2: Remove 2 (all same)
        System.out.println("[[1,2],[1,2],[1,2]] → " +
            sol.eraseOverlapIntervals(new int[][]{{1,2},{1,2},{1,2}})); // 2

        // Example 3: No overlap needed
        System.out.println("[[1,2],[2,3]] → " +
            sol.eraseOverlapIntervals(new int[][]{{1,2},{2,3}})); // 0

        // Example 4: Remove all but one
        System.out.println("[[1,100],[11,22],[1,11],[2,12]] → " +
            sol.eraseOverlapIntervals(new int[][]{{1,100},{11,22},{1,11},{2,12}})); // 2

        // Example 5: Single interval
        System.out.println("[[1,5]] → " +
            sol.eraseOverlapIntervals(new int[][]{{1,5}})); // 0
    }
}
```

### Output

```
[[1,2],[2,3],[3,4],[1,3]] → 1
[[1,2],[1,2],[1,2]] → 2
[[1,2],[2,3]] → 0
[[1,100],[11,22],[1,11],[2,12]] → 2
[[1,5]] → 0
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def erase_overlap_intervals(intervals: list[list[int]]) -> int:
    """
    Find the minimum number of intervals to remove to eliminate all overlaps.

    Strategy: Greedy — Activity Selection Problem.
    Sort by end time. Greedily keep the interval ending earliest.
    If the next interval starts before the last kept interval ends → overlap → remove.

    Args:
        intervals: List of [start, end] intervals

    Returns:
        Minimum number of intervals to remove

    Time:  O(n log n)
    Space: O(1)
    """
    intervals.sort(key=lambda x: x[1])  # sort by end time

    removals = 0
    last_end = float('-inf')

    for start, end in intervals:
        if start >= last_end:
            last_end = end   # keep this interval
        else:
            removals += 1    # overlap → remove (the current one has a later end, so discard it)

    return removals


if __name__ == "__main__":
    import copy

    test_cases = [
        ([[1,2],[2,3],[3,4],[1,3]], 1),
        ([[1,2],[1,2],[1,2]],       2),
        ([[1,2],[2,3]],             0),
        ([[1,100],[11,22],[1,11],[2,12]], 2),
        ([[1,5]],                   0),
        ([[-52,31],[-73,-26],[82,97],[-65,-11],[-62,-49],[95,99],[58,95],[-31,49],[66,98],[-63,2],[30,47],[-40,-26]], 7),
    ]

    for intervals, expected in test_cases:
        result = erase_overlap_intervals(copy.deepcopy(intervals))
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] intervals={intervals[:3]}{'...' if len(intervals)>3 else ''} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] intervals=[[1, 2], [2, 3], [3, 4]] → Output: 1, Expected: 1
[PASS] intervals=[[1, 2], [1, 2], [1, 2]] → Output: 2, Expected: 2
[PASS] intervals=[[1, 2], [2, 3]] → Output: 0, Expected: 0
[PASS] intervals=[[1, 100], [11, 22], [1, 11]] → Output: 2, Expected: 2
[PASS] intervals=[[1, 5]] → Output: 0, Expected: 0
[PASS] intervals=[[-52, 31], [-73, -26], [82, 97]]... → Output: 7, Expected: 7
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why sort by end time instead of start time?**
A: Sorting by end time is the standard greedy strategy for activity selection — it maximizes the number of non-overlapping intervals we can keep, minimizing removals. Sorting by start time does not give the optimal result in all cases.

**Q: Prove why keeping the earliest-ending interval is always optimal.**
A: Suppose we keep interval A (earliest ending) instead of B (later ending). Any interval that fits after B also fits after A (since A ends no later than B). So keeping A never leads to fewer future choices than keeping B. This is the exchange argument — swapping B for A never makes things worse.

**Q: How does touching (`[1,2]` and `[2,3]`) affect the solution?**
A: The condition is `start >= lastEnd` (≥, not >). So `start=2 >= lastEnd=2` means no overlap — they merely touch. This correctly keeps both intervals without counting either as a removal.

**Q: What's the relationship between this and the "Longest Non-overlapping Subsequence"?**
A: `min_removals = n - max_non_overlapping`. The greedy solution for max non-overlapping subsequence (Activity Selection) gives `max_non_overlapping`, so subtracting from n gives `min_removals`. Both perspectives lead to the same answer.

**Q: Why does this greedy fail if sorted by start time?**
A: Consider `[[1,100],[2,3],[3,4]]`. Sorted by start: `[1,100],[2,3],[3,4]`. Greedy by start would keep `[1,100]` first and remove both others (2 removals). But the optimal is to remove `[1,100]` and keep `[2,3],[3,4]` (1 removal). Sorting by end prevents this mistake.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Minimum intervals to remove so no two overlap                      |
| **Optimal Algorithm** | Greedy — sort by end time, keep earliest-ending non-overlapping   |
| **Time Complexity**   | O(n log n)                                                         |
| **Space Complexity**  | O(1)                                                               |
| **Key Insight**       | Activity Selection: always keep the interval ending soonest        |

### Core Pattern

> Sort by end time. Scan left to right. If the current interval doesn't overlap the last kept one, keep it. If it does, discard it (it ends later and blocks more future intervals). Count discards.

### When to Use This Pattern

- Interval scheduling / activity selection problems.
- Related: Meeting Rooms II, Minimum Number of Arrows to Burst Balloons, Task Scheduler.

