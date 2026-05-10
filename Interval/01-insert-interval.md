# Insert Interval

---

<details>
<summary><strong>1. What is meant by Insert Interval?</strong></summary>

## What is Insert Interval?

**Insert Interval** is an interval manipulation problem where:

- You are given a list of **non-overlapping** intervals sorted by start time.
- You are given a **new interval** to insert into the list.

**Goal:** Insert the new interval into the list, merging any overlapping intervals, and return the result (still sorted and non-overlapping).

**Constraints:**
- `0 <= intervals.length <= 10⁴`
- `intervals[i] = [starti, endi]` where `starti <= endi`
- Intervals are sorted by `starti` in ascending order.
- `newInterval = [start, end]` where `start <= end`

**Example:**

```
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
Explanation: [1,3] and [2,5] overlap → merge to [1,5]

Input:  intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]
Explanation: [3,5],[6,7],[8,10] all overlap with [4,8] → merge to [3,10]
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Insert the new interval in the correct sorted position.
- Merge all intervals that overlap with the new interval.
- Return a list of non-overlapping intervals in sorted order.
- The original list is already sorted and non-overlapping.

### Non-Functional Requirements

| Concern         | Expectation                                             |
| --------------- | ------------------------------------------------------- |
| **Scale**       | Up to 10⁴ intervals — O(n) single-pass is expected     |
| **Latency**     | O(n) — linear scan, no sorting needed                  |
| **Correctness** | All overlapping intervals must be merged, none skipped |
| **Edge Cases**  | Empty list, new interval before all, after all, inside one |

### Clarifying Questions to Ask

- Is the input already sorted? → Yes, guaranteed.
- Are intervals already non-overlapping? → Yes, in the input.
- Can the new interval be a duplicate? → Yes, must still merge correctly.
- What defines overlap? → `[a,b]` and `[c,d]` overlap if `a <= d && c <= b`.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Insert + Sort + Merge

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sort dominates |
| **Space** | O(n) — result list |

### Approach 2: Single-Pass Linear Scan (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — one pass through intervals |
| **Space** | O(n) — result list |

**Why O(n)?** Since intervals are already sorted, we can place intervals in three buckets in a single pass:
1. Intervals that come **before** the new interval (no overlap).
2. Intervals that **overlap** with the new interval (merge into one).
3. Intervals that come **after** the new interval (no overlap).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Single-Pass Linear Scan

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Insert + Sort + Merge | O(n log n) | O(n) | No — wastes the sorted property |
| **Single-Pass Scan** | **O(n)** | **O(n)** | **Yes** |

### Key Insight

Scan the sorted list in three phases:

**Phase 1 — No overlap, before new interval:**
```
interval.end < newInterval.start  → add directly to result
```

**Phase 2 — Overlap:**
```
interval.start <= newInterval.end  → merge by expanding newInterval:
  newInterval.start = min(newInterval.start, interval.start)
  newInterval.end   = max(newInterval.end,   interval.end)
```

**Phase 3 — No overlap, after new interval:**
```
Remaining intervals → add directly to result
```

At the end of Phase 1 / start of Phase 3, insert the (possibly expanded) `newInterval`.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Pre-overlap Collector** — Add intervals ending before the new interval starts.
2. **Merger** — Expand new interval to cover all overlapping intervals.
3. **New Interval Inserter** — Add the merged new interval to result.
4. **Post-overlap Collector** — Add remaining intervals after the new interval.
5. **Output** — Return the result list.

### Data Flow Diagram

```
intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]

Phase 1 — ends before 4:
  [1,2]: end=2 < 4 → add [1,2]
  [3,5]: end=5 >= 4 → overlap starts

Phase 2 — overlaps with [4,8]:
  [3,5]:  start=3 <=8 → expand: new=[min(4,3), max(8,5)] = [3,8]
  [6,7]:  start=6 <=8 → expand: new=[min(3,6), max(8,7)] = [3,8]
  [8,10]: start=8 <=8 → expand: new=[min(3,8), max(8,10)]= [3,10]
  [12,16]:start=12 > 8 → overlap ends

Insert merged [3,10]

Phase 3 — starts after 10:
  [12,16]: add [12,16]

Result: [[1,2],[3,10],[12,16]]
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class InsertInterval {

    /**
     * Insert a new interval into a sorted non-overlapping interval list and merge overlaps.
     *
     * Single-pass O(n):
     *  Phase 1: add intervals ending before newInterval starts.
     *  Phase 2: merge all intervals overlapping with newInterval.
     *  Phase 3: add remaining intervals after newInterval.
     *
     * @param intervals   sorted list of non-overlapping intervals
     * @param newInterval the interval to insert
     * @return merged interval list with newInterval inserted
     *
     * Time:  O(n)
     * Space: O(n) — output list
     */
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0, n = intervals.length;

        // Phase 1: intervals completely before newInterval (end < new start)
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i++]);
        }

        // Phase 2: merge all overlapping intervals into newInterval
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval); // add the merged interval

        // Phase 3: intervals completely after newInterval (start > new end)
        while (i < n) {
            result.add(intervals[i++]);
        }

        return result.toArray(new int[0][]);
    }

    public static void main(String[] args) {
        InsertInterval sol = new InsertInterval();

        // Example 1: overlap in middle
        int[][] r1 = sol.insert(new int[][]{{1,3},{6,9}}, new int[]{2,5});
        System.out.println("Example 1: " + Arrays.deepToString(r1)); // [[1,5],[6,9]]

        // Example 2: multiple merges
        int[][] r2 = sol.insert(new int[][]{{1,2},{3,5},{6,7},{8,10},{12,16}}, new int[]{4,8});
        System.out.println("Example 2: " + Arrays.deepToString(r2)); // [[1,2],[3,10],[12,16]]

        // Example 3: empty list
        int[][] r3 = sol.insert(new int[][]{}, new int[]{5,7});
        System.out.println("Example 3: " + Arrays.deepToString(r3)); // [[5,7]]

        // Example 4: insert before all
        int[][] r4 = sol.insert(new int[][]{{3,5},{6,9}}, new int[]{1,2});
        System.out.println("Example 4: " + Arrays.deepToString(r4)); // [[1,2],[3,5],[6,9]]

        // Example 5: insert after all
        int[][] r5 = sol.insert(new int[][]{{1,2},{3,5}}, new int[]{7,9});
        System.out.println("Example 5: " + Arrays.deepToString(r5)); // [[1,2],[3,5],[7,9]]

        // Example 6: new interval covers all
        int[][] r6 = sol.insert(new int[][]{{1,3},{4,6}}, new int[]{0,10});
        System.out.println("Example 6: " + Arrays.deepToString(r6)); // [[0,10]]
    }
}
```

### Output

```
Example 1: [[1, 5], [6, 9]]
Example 2: [[1, 2], [3, 10], [12, 16]]
Example 3: [[5, 7]]
Example 4: [[1, 2], [3, 5], [6, 9]]
Example 5: [[1, 2], [3, 5], [7, 9]]
Example 6: [[0, 10]]
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def insert(intervals: list[list[int]], new_interval: list[int]) -> list[list[int]]:
    """
    Insert a new interval into a sorted non-overlapping list and merge overlaps.

    Single-pass O(n) — three phases:
      Phase 1: collect intervals ending before new_interval starts (no overlap).
      Phase 2: merge all overlapping intervals into new_interval.
      Phase 3: collect remaining intervals after new_interval (no overlap).

    Args:
        intervals:    Sorted list of non-overlapping intervals [[s,e], ...]
        new_interval: Interval to insert [start, end]

    Returns:
        Merged list of non-overlapping intervals

    Time:  O(n)
    Space: O(n)
    """
    result = []
    i, n = 0, len(intervals)

    # Phase 1: no overlap, interval ends before new_interval starts
    while i < n and intervals[i][1] < new_interval[0]:
        result.append(intervals[i])
        i += 1

    # Phase 2: merge overlapping intervals
    while i < n and intervals[i][0] <= new_interval[1]:
        new_interval[0] = min(new_interval[0], intervals[i][0])
        new_interval[1] = max(new_interval[1], intervals[i][1])
        i += 1
    result.append(new_interval)  # insert merged interval

    # Phase 3: no overlap, interval starts after new_interval ends
    while i < n:
        result.append(intervals[i])
        i += 1

    return result


if __name__ == "__main__":
    test_cases = [
        ([[1,3],[6,9]],                    [2,5],  [[1,5],[6,9]]),
        ([[1,2],[3,5],[6,7],[8,10],[12,16]], [4,8], [[1,2],[3,10],[12,16]]),
        ([],                               [5,7],  [[5,7]]),
        ([[3,5],[6,9]],                    [1,2],  [[1,2],[3,5],[6,9]]),
        ([[1,2],[3,5]],                    [7,9],  [[1,2],[3,5],[7,9]]),
        ([[1,3],[4,6]],                    [0,10], [[0,10]]),
    ]

    for intervals, new_interval, expected in test_cases:
        result = insert([iv[:] for iv in intervals], new_interval[:])
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] new={new_interval} → {result}")
```

### Output

```
[PASS] new=[2, 5] → [[1, 5], [6, 9]]
[PASS] new=[4, 8] → [[1, 2], [3, 10], [12, 16]]
[PASS] new=[5, 7] → [[5, 7]]
[PASS] new=[1, 2] → [[1, 2], [3, 5], [6, 9]]
[PASS] new=[7, 9] → [[1, 2], [3, 5], [7, 9]]
[PASS] new=[0, 10] → [[0, 10]]
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: What is the overlap condition between two intervals `[a,b]` and `[c,d]`?**
A: They overlap if `a <= d && c <= b`. Equivalently, they do NOT overlap if `b < c` (first ends before second starts) or `d < a` (second ends before first starts).

**Q: Why don't we need to sort the input?**
A: The problem guarantees the input is already sorted by start time. We exploit this to do a single O(n) pass. If the input were unsorted, we'd need O(n log n) sorting first.

**Q: What happens when the new interval doesn't overlap with any existing interval?**
A: Phase 2 runs zero iterations. The new interval is inserted exactly at its sorted position between Phase 1 and Phase 3.

**Q: How do you define the merged interval during Phase 2?**
A: `merged.start = min(newInterval.start, current.start)` and `merged.end = max(newInterval.end, current.end)`. This expands the new interval to cover all overlapping intervals.

**Q: What if the new interval is a subset of an existing interval?**
A: Phase 2 catches it — the existing interval's start ≤ newInterval's end, so they overlap. After merging, the result interval is the existing interval (since its bounds are larger). The new interval is "absorbed."

**Q: What if intervals is empty?**
A: Phase 1, 2, and 3 all skip. Only `result.add(newInterval)` runs. Returns `[[newInterval]]`.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                                  |
| --------------------- | ---------------------------------------------------------------------- |
| **Problem**           | Insert a new interval into a sorted non-overlapping list, merging overlaps |
| **Optimal Algorithm** | Single-pass linear scan (three phases)                                 |
| **Time Complexity**   | O(n)                                                                   |
| **Space Complexity**  | O(n)                                                                   |
| **Key Insight**       | Three phases: before/overlap/after the new interval — exploit sorted order |

### Core Pattern

> Exploit the sorted order. Skip non-overlapping intervals before, greedily merge all overlapping ones into the new interval, then append the rest. No sorting needed — O(n) is achievable.

### When to Use This Pattern

- Inserting into a sorted interval structure with merge.
- Related: Merge Intervals, Interval List Intersections, Calendar scheduling.

</details>
