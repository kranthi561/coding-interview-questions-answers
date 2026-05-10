# Merge Intervals

---

<details>
<summary><strong>1. What is meant by Merge Intervals?</strong></summary>

## What is Merge Intervals?

**Merge Intervals** is a classic interval problem where:

- You are given an array of intervals `intervals[i] = [starti, endi]`.
- The intervals may be **unsorted** and may **overlap**.

**Goal:** Merge all overlapping intervals and return an array of the resulting **non-overlapping** intervals that cover all the intervals in the input.

**Constraints:**
- `1 <= intervals.length <= 10⁴`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 10⁴`

**Example:**

```
Input:  intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: [1,3] and [2,6] overlap → merge to [1,6]

Input:  intervals = [[1,4],[4,5]]
Output: [[1,5]]
Explanation: [1,4] and [4,5] touch at 4 → merge (touching = overlapping)
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Merge all overlapping or touching intervals.
- Return the result sorted by start time.
- Input intervals may be in any order.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | Up to 10⁴ intervals — O(n log n) is acceptable     |
| **Latency**     | O(n log n) — dominated by sorting                  |
| **Correctness** | Must handle touching intervals (e.g., [1,4],[4,5]) |
| **Edge Cases**  | Single interval, all overlapping, none overlapping  |

### Clarifying Questions to Ask

- Do touching intervals count as overlapping? → Yes (`[1,4]` and `[4,5]` → `[1,5]`).
- Is the output required to be sorted? → Yes.
- Can start and end be equal (single-point interval)? → Yes.
- Can we modify the input? → Yes.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force (Compare All Pairs)

| | Complexity |
| --- | --- |
| **Time** | O(n²) — compare every pair |
| **Space** | O(n) — output list |

### Approach 2: Sort + Linear Merge (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sorting dominates |
| **Space** | O(n) — output list (O(log n) for in-place sort stack) |

**Why O(n log n)?** Sorting costs O(n log n). The single-pass merge after sorting is O(n). Total = O(n log n).

**Can we do O(n)?** Only if the intervals are already sorted. In the general unsorted case, O(n log n) is optimal because we need to sort.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Sort by Start Time + Linear Merge

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(n²) | O(n) | No |
| **Sort + Linear Merge** | **O(n log n)** | **O(n)** | **Yes** |

### Key Insight

After sorting by start time, overlapping intervals must be **adjacent**. So we only need to compare each interval to the last interval in the result:

```
Sort intervals by start time.
Add first interval to result.

For each remaining interval:
  if current.start <= last.end:
    merge: last.end = max(last.end, current.end)
  else:
    add current to result (no overlap)
```

**Why `max(last.end, current.end)`?**
One interval could be completely inside another: `[1,10]` and `[2,5]` → merged end is 10, not 5.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Sorter** — Sort intervals by start time.
2. **Result Builder** — Maintains the running list of merged intervals.
3. **Overlap Checker** — For each interval, check if it overlaps with the last in result.
4. **Merger** — Extend the last interval's end if overlap exists.
5. **Output** — Return result list.

### Data Flow Diagram

```
intervals = [[1,3],[2,6],[8,10],[15,18]]

After sort by start: [[1,3],[2,6],[8,10],[15,18]]

result = [[1,3]]

[2,6]:  start=2 <= last.end=3 → merge: last=[1, max(3,6)]=[1,6]
        result = [[1,6]]

[8,10]: start=8 > last.end=6 → no overlap → add
        result = [[1,6],[8,10]]

[15,18]:start=15 > last.end=10 → no overlap → add
        result = [[1,6],[8,10],[15,18]]

Output: [[1,6],[8,10],[15,18]]

---

Touching: [[1,4],[4,5]]
After sort: [[1,4],[4,5]]
result = [[1,4]]
[4,5]: start=4 <= last.end=4 → merge: last=[1,max(4,5)]=[1,5]
Output: [[1,5]]
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class MergeIntervals {

    /**
     * Merge all overlapping intervals in the given list.
     *
     * Strategy:
     *  1. Sort by start time so overlapping intervals are adjacent.
     *  2. Iterate: if current start <= last merged end, extend the end.
     *     Otherwise, start a new interval in the result.
     *
     * @param intervals array of [start, end] intervals (unsorted, may overlap)
     * @return array of merged non-overlapping intervals
     *
     * Time:  O(n log n) — sort dominates
     * Space: O(n) — result list
     */
    public int[][] merge(int[][] intervals) {
        // Sort by start time
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        List<int[]> result = new ArrayList<>();
        result.add(intervals[0]); // seed with first interval

        for (int i = 1; i < intervals.length; i++) {
            int[] last = result.get(result.size() - 1);
            int[] curr = intervals[i];

            if (curr[0] <= last[1]) {
                // Overlap or touching: extend the end of the last merged interval
                last[1] = Math.max(last[1], curr[1]);
            } else {
                // No overlap: start a new interval
                result.add(curr);
            }
        }

        return result.toArray(new int[0][]);
    }

    public static void main(String[] args) {
        MergeIntervals sol = new MergeIntervals();

        // Example 1: Standard overlap
        System.out.println("Example 1: " + Arrays.deepToString(
            sol.merge(new int[][]{{1,3},{2,6},{8,10},{15,18}}))); // [[1,6],[8,10],[15,18]]

        // Example 2: Touching at boundary
        System.out.println("Example 2: " + Arrays.deepToString(
            sol.merge(new int[][]{{1,4},{4,5}}))); // [[1,5]]

        // Example 3: Single interval
        System.out.println("Example 3: " + Arrays.deepToString(
            sol.merge(new int[][]{{5,10}}))); // [[5,10]]

        // Example 4: No overlaps
        System.out.println("Example 4: " + Arrays.deepToString(
            sol.merge(new int[][]{{1,2},{3,4},{5,6}}))); // [[1,2],[3,4],[5,6]]

        // Example 5: One interval contains another
        System.out.println("Example 5: " + Arrays.deepToString(
            sol.merge(new int[][]{{1,10},{2,5},{3,7}}))); // [[1,10]]

        // Example 6: Unsorted input
        System.out.println("Example 6: " + Arrays.deepToString(
            sol.merge(new int[][]{{15,18},{1,3},{2,6},{8,10}}))); // [[1,6],[8,10],[15,18]]
    }
}
```

### Output

```
Example 1: [[1, 6], [8, 10], [15, 18]]
Example 2: [[1, 5]]
Example 3: [[5, 10]]
Example 4: [[1, 2], [3, 4], [5, 6]]
Example 5: [[1, 10]]
Example 6: [[1, 6], [8, 10], [15, 18]]
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    """
    Merge all overlapping intervals.

    Strategy:
      1. Sort by start time — overlapping intervals become adjacent.
      2. Maintain a result list. For each interval:
           - If it overlaps with the last in result (start <= last end),
             extend the last interval's end.
           - Otherwise, append it as a new interval.

    Args:
        intervals: List of [start, end] intervals (unsorted, may overlap)

    Returns:
        List of merged non-overlapping intervals

    Time:  O(n log n)
    Space: O(n)
    """
    intervals.sort(key=lambda x: x[0])  # sort by start time

    result = [intervals[0]]  # seed with first interval

    for start, end in intervals[1:]:
        if start <= result[-1][1]:      # overlap or touching
            result[-1][1] = max(result[-1][1], end)  # extend end
        else:
            result.append([start, end]) # new non-overlapping interval

    return result


if __name__ == "__main__":
    import copy

    test_cases = [
        ([[1,3],[2,6],[8,10],[15,18]], [[1,6],[8,10],[15,18]]),
        ([[1,4],[4,5]],               [[1,5]]),
        ([[5,10]],                    [[5,10]]),
        ([[1,2],[3,4],[5,6]],         [[1,2],[3,4],[5,6]]),
        ([[1,10],[2,5],[3,7]],        [[1,10]]),
        ([[15,18],[1,3],[2,6],[8,10]],[[1,6],[8,10],[15,18]]),
    ]

    for intervals, expected in test_cases:
        result = merge(copy.deepcopy(intervals))
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] Input: {intervals} → Output: {result}")
```

### Output

```
[PASS] Input: [[1, 3], [2, 6], [8, 10], [15, 18]] → Output: [[1, 6], [8, 10], [15, 18]]
[PASS] Input: [[1, 4], [4, 5]] → Output: [[1, 5]]
[PASS] Input: [[5, 10]] → Output: [[5, 10]]
[PASS] Input: [[1, 2], [3, 4], [5, 6]] → Output: [[1, 2], [3, 4], [5, 6]]
[PASS] Input: [[1, 10], [2, 5], [3, 7]] → Output: [[1, 10]]
[PASS] Input: [[15, 18], [1, 3], [2, 6], [8, 10]] → Output: [[1, 6], [8, 10], [15, 18]]
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why sort by start time?**
A: Sorting ensures overlapping intervals are adjacent. Without sorting, an interval could overlap with one far away, requiring O(n²) comparisons. With sorting, a single left-to-right pass is sufficient.

**Q: Why use `max(last.end, current.end)` instead of just `current.end`?**
A: One interval can be completely contained within another (e.g., `[1,10]` and `[2,5]`). Using only `current.end` would shrink the merged interval incorrectly. `max` handles this case.

**Q: What does "touching" mean and why does it count as overlapping?**
A: Two intervals `[1,4]` and `[4,5]` share the endpoint 4. They can be merged into `[1,5]` because there's no gap between them. The condition `current.start <= last.end` (not strictly less than) handles touching.

**Q: Can this be done in O(n) time?**
A: Only if the input is already sorted. For unsorted input, any comparison-based sort is Ω(n log n). The merge pass itself is O(n).

**Q: How does this compare to Insert Interval?**
A: Merge Intervals handles an unsorted list with multiple overlaps. Insert Interval handles a pre-sorted non-overlapping list with one new interval. Insert Interval is O(n); Merge Intervals is O(n log n) due to sorting.

**Q: What if all intervals overlap?**
A: The result list ends up with a single interval: `[min_start, max_end]`. The merge step keeps extending the last interval's end for every subsequent interval.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Problem**           | Merge all overlapping intervals in an unsorted list            |
| **Optimal Algorithm** | Sort by start time + single-pass linear merge                  |
| **Time Complexity**   | O(n log n)                                                     |
| **Space Complexity**  | O(n)                                                           |
| **Key Insight**       | Sort makes overlaps adjacent; one pass with `max(end)` merges them |

### Core Pattern

> Sort → iterate → compare current start with last merged end. If overlap, extend; otherwise, append. The sort step collapses a 2D problem into a 1D problem.

### When to Use This Pattern

- Any interval merging or deduplication task.
- Related: Insert Interval, Non-overlapping Intervals, Interval List Intersections, Employee Free Time.

</details>
