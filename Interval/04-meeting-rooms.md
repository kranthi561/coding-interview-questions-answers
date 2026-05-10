# Meeting Rooms

---

<details>
<summary><strong>1. What is meant by Meeting Rooms?</strong></summary>

## What is Meeting Rooms?

**Meeting Rooms** is an interval overlap detection problem where:

- You are given an array of meeting time intervals `intervals[i] = [starti, endi]`.

**Goal:** Determine if a person can **attend all meetings** — i.e., no two meetings overlap.

**Constraints:**
- `0 <= intervals.length <= 10⁴`
- `intervals[i].length == 2`
- `0 <= starti < endi <= 10⁶`

**Example:**

```
Input:  intervals = [[0,30],[5,10],[15,20]]
Output: false
Explanation: [0,30] and [5,10] overlap.

Input:  intervals = [[7,10],[2,4]]
Output: true
Explanation: After sorting: [2,4] and [7,10] — no overlap.
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Return `true` only if **all** meetings can be attended without any time conflict.
- Two meetings conflict if one starts before the other ends.
- Touching meetings (e.g., `[1,5]` and `[5,10]`) are NOT a conflict.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | Up to 10⁴ meetings — O(n log n) easily handles     |
| **Latency**     | O(n log n) — dominated by sort                      |
| **Correctness** | Must detect any overlap, including partial ones     |
| **Edge Cases**  | Empty list → true, single meeting → true            |

### Clarifying Questions to Ask

- Does the person attend all meetings or just some? → All (return false if any overlap).
- Is `[1,5]` and `[5,10]` a conflict? → No — the first ends exactly when the second starts.
- Is the input sorted? → Not necessarily.
- Can meetings have zero duration? → Constraints say `start < end`, so no.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force (Compare All Pairs)

| | Complexity |
| --- | --- |
| **Time** | O(n²) — check every pair for overlap |
| **Space** | O(1) |

### Approach 2: Sort + Linear Scan (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sorting dominates |
| **Space** | O(1) — or O(log n) for the sort call stack |

**Why O(n log n)?** After sorting by start time, overlapping meetings must be adjacent. A single linear scan suffices to detect any overlap.

**Can we do O(n)?** Only if the input is already sorted. Sorting is required in the general case.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Sort by Start Time + Linear Overlap Check

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(n²) | O(1) | No |
| **Sort + Linear Scan** | **O(n log n)** | **O(1)** | **Yes** |

### Key Insight

After sorting by start time, if any overlap exists, it **must** appear between two consecutive meetings in the sorted order:

```
Sort by start time.

For i from 1 to n-1:
  if intervals[i].start < intervals[i-1].end:
    return false  // overlap detected
return true
```

**Why check only consecutive pairs?**  
In a sorted list, if `A` and `C` overlap but are not adjacent (B is between them), then either `B` overlaps with `A` or `B` overlaps with `C` — the consecutive check catches the conflict anyway.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Edge Case Guard** — Return `true` for empty or single-element list.
2. **Sorter** — Sort meetings by start time.
3. **Consecutive Overlap Checker** — Compare each meeting's start with the previous one's end.
4. **Output** — Return `false` on first conflict, `true` if none found.

### Data Flow Diagram

```
intervals = [[0,30],[5,10],[15,20]]

After sort by start: [[0,30],[5,10],[15,20]]

i=1: intervals[1].start=5 < intervals[0].end=30 → OVERLAP → return false ✗

---

intervals = [[7,10],[2,4]]

After sort by start: [[2,4],[7,10]]

i=1: intervals[1].start=7 >= intervals[0].end=4 → OK

No overlap → return true ✓

---

intervals = [[1,5],[5,10],[10,15]]

After sort: [[1,5],[5,10],[10,15]]

i=1: 5 >= 5 → OK (touching, not overlapping)
i=2: 10 >= 10 → OK

return true ✓
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class MeetingRooms {

    /**
     * Determine if a person can attend all meetings (no overlaps).
     *
     * Strategy: Sort by start time, then check each consecutive pair.
     * If meeting[i].start < meeting[i-1].end → overlap → return false.
     * Touching (start == end of previous) is allowed.
     *
     * @param intervals array of [start, end] meeting times
     * @return true if all meetings can be attended without conflict
     *
     * Time:  O(n log n)
     * Space: O(1)
     */
    public boolean canAttendMeetings(int[][] intervals) {
        if (intervals.length <= 1) return true;

        // Sort by start time
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        // Check each consecutive pair for overlap
        for (int i = 1; i < intervals.length; i++) {
            // If the next meeting starts before the current one ends → conflict
            if (intervals[i][0] < intervals[i - 1][1]) {
                return false;
            }
        }

        return true; // no overlap found
    }

    public static void main(String[] args) {
        MeetingRooms sol = new MeetingRooms();

        // Example 1: Overlap exists
        System.out.println("[[0,30],[5,10],[15,20]] → " +
            sol.canAttendMeetings(new int[][]{{0,30},{5,10},{15,20}})); // false

        // Example 2: No overlap (unsorted input)
        System.out.println("[[7,10],[2,4]] → " +
            sol.canAttendMeetings(new int[][]{{7,10},{2,4}})); // true

        // Example 3: Touching meetings (not a conflict)
        System.out.println("[[1,5],[5,10],[10,15]] → " +
            sol.canAttendMeetings(new int[][]{{1,5},{5,10},{10,15}})); // true

        // Example 4: Empty list
        System.out.println("[] → " +
            sol.canAttendMeetings(new int[][]{})); // true

        // Example 5: Single meeting
        System.out.println("[[3,7]] → " +
            sol.canAttendMeetings(new int[][]{{3,7}})); // true

        // Example 6: All same time
        System.out.println("[[1,10],[1,10]] → " +
            sol.canAttendMeetings(new int[][]{{1,10},{1,10}})); // false
    }
}
```

### Output

```
[[0,30],[5,10],[15,20]] → false
[[7,10],[2,4]] → true
[[1,5],[5,10],[10,15]] → true
[] → true
[[3,7]] → true
[[1,10],[1,10]] → false
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def can_attend_meetings(intervals: list[list[int]]) -> bool:
    """
    Determine if a person can attend all meetings without any time conflict.

    Strategy: Sort by start time, then check each consecutive pair.
    If meeting[i].start < meeting[i-1].end → overlap → return False.
    Touching meetings (start == previous end) are allowed.

    Args:
        intervals: List of [start, end] meeting time intervals

    Returns:
        True if all meetings can be attended, False if any conflict exists

    Time:  O(n log n)
    Space: O(1)
    """
    intervals.sort(key=lambda x: x[0])  # sort by start time

    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i - 1][1]:  # start before previous ends
            return False

    return True


if __name__ == "__main__":
    import copy

    test_cases = [
        ([[0,30],[5,10],[15,20]], False),
        ([[7,10],[2,4]],          True),
        ([[1,5],[5,10],[10,15]],  True),
        ([],                      True),
        ([[3,7]],                 True),
        ([[1,10],[1,10]],         False),
        ([[1,4],[2,5],[7,9]],     False),
    ]

    for intervals, expected in test_cases:
        result = can_attend_meetings(copy.deepcopy(intervals))
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] intervals={intervals} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] intervals=[[0, 30], [5, 10], [15, 20]] → Output: False, Expected: False
[PASS] intervals=[[7, 10], [2, 4]] → Output: True, Expected: True
[PASS] intervals=[[1, 5], [5, 10], [10, 15]] → Output: True, Expected: True
[PASS] intervals=[] → Output: True, Expected: True
[PASS] intervals=[[3, 7]] → Output: True, Expected: True
[PASS] intervals=[[1, 10], [1, 10]] → Output: False, Expected: False
[PASS] intervals=[[1, 4], [2, 5], [7, 9]] → Output: False, Expected: False
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why do we sort by start time and not end time?**
A: We want to detect if any meeting starts before the previous one ends. Sorting by start ensures that consecutive meetings in the sorted list are the only ones that can potentially conflict. If they don't conflict, no unsorted pair would either.

**Q: Is `[1,5]` and `[5,10]` a conflict?**
A: No. The condition is `start < previous_end` (strictly less than). `5 < 5` is false, so they don't conflict. The first meeting ends exactly when the second begins — the person is free.

**Q: Why is it sufficient to check only consecutive pairs?**
A: In a sorted list, if meeting `i` and meeting `k` (k > i+1) conflict, meeting `i+1` must have started before `i` ended (since `i+1.start <= k.start < i.end`). The consecutive check catches the conflict between `i` and `i+1`.

**Q: What if two meetings have the same start time?**
A: After sorting, they will be adjacent. The check `intervals[i][0] < intervals[i-1][1]` will detect `start == start < end` — flagged as a conflict. Both meetings can't be attended simultaneously.

**Q: How does this differ from Meeting Rooms II?**
A: Meeting Rooms (I) asks "can one person attend all?" — a binary answer. Meeting Rooms II asks "how many rooms (people) are needed to cover all meetings?" — a count. Meeting Rooms I is O(n log n) with O(1) extra space; Meeting Rooms II uses a min-heap and O(n) space.

**Q: Can this be solved in O(n)?**
A: Only if the input is already sorted. For unsorted input, any algorithm must sort or use a hash/bucket approach (which has high constants for large ranges).

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                                  |
| --------------------- | ---------------------------------------------------------------------- |
| **Problem**           | Determine if one person can attend all meetings (no overlaps)           |
| **Optimal Algorithm** | Sort by start time + check consecutive pairs                           |
| **Time Complexity**   | O(n log n)                                                             |
| **Space Complexity**  | O(1)                                                                   |
| **Key Insight**       | In sorted order, overlaps only appear between consecutive meetings     |

### Core Pattern

> Sort by start time. Scan pairs. If `meeting[i].start < meeting[i-1].end`, there's a conflict — return false. Otherwise return true. Simplest of all interval problems.

### When to Use This Pattern

- Detecting any overlap in an interval set.
- Related: Meeting Rooms II (room count), Non-overlapping Intervals, Merge Intervals.

</details>
