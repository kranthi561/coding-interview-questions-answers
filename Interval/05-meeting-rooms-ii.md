# Meeting Rooms II

---

1. What is meant by Meeting Rooms II?

## What is Meeting Rooms II?

**Meeting Rooms II** is an interval scheduling problem where:

- You are given an array of meeting time intervals `intervals[i] = [starti, endi]`.

**Goal:** Find the **minimum number of conference rooms** required so that all meetings can be held simultaneously without any room conflict.

**Constraints:**
- `1 <= intervals.length <= 10⁴`
- `0 <= starti < endi <= 10⁶`

**Example:**

```
Input:  intervals = [[0,30],[5,10],[15,25]]
Output: 2
Explanation:
  Room 1: [0,30]
  Room 2: [5,10], then [15,25]  ← [10,15] gap allows reuse

Input:  intervals = [[7,10],[2,4]]
Output: 1
Explanation: [2,4] ends before [7,10] starts → one room suffices.
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Return the minimum number of simultaneous rooms needed.
- A room can be reused when a meeting ends before or exactly when the next one starts.
- All meetings must be scheduled — none can be dropped.

### Non-Functional Requirements

| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | Up to 10⁴ meetings — O(n log n) required             |
| **Latency**     | O(n log n) — sorting + heap operations               |
| **Correctness** | Must count the maximum concurrent meetings at any point |
| **Edge Cases**  | Single meeting → 1, all overlapping → n rooms         |

### Clarifying Questions to Ask

- Can a room be reused when the previous meeting ends exactly when the new one starts? → Yes.
- Are we minimizing rooms or assigning meetings to rooms? → Minimizing rooms (return count).
- Can meetings overlap partially? → Yes — that's the whole challenge.
- Is input sorted? → No.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Brute Force

| | Complexity |
| --- | --- |
| **Time** | O(n²) — try all room assignments |
| **Space** | O(n) |

### Approach 2: Min-Heap (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sort + n heap operations (each O(log n)) |
| **Space** | O(n) — heap stores end times |

### Approach 3: Two-Pointer / Chronological Ordering

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sort starts and ends separately |
| **Space** | O(n) |

**The answer = maximum number of meetings happening simultaneously** at any point in time.


---

4. Which Algorithm and Why?

## Algorithm: Min-Heap of Room End Times

### Why Min-Heap?

| Option | Time | Space | Notes | Chosen? |
| --- | --- | --- | --- | --- |
| Brute Force | O(n²) | O(n) | Check all rooms | No |
| Two-Pointer | O(n log n) | O(n) | Sort starts+ends separately | Equivalent |
| **Min-Heap** | **O(n log n)** | **O(n)** | **Intuitive room-reuse model** | **Yes** |

### Key Insight

Sort meetings by start time. Use a min-heap to track the **end time of the room that frees up earliest**.

For each new meeting:
- **If the earliest-ending room frees up in time** (`heap.min <= current.start`): pop it and push the new end time (room reused).
- **Otherwise, all rooms are busy**: push a new end time (new room allocated).

The heap size at any point = current number of rooms in use. The maximum heap size ever reached = answer.

```
Sort by start time.
For each meeting [start, end]:
  if heap is non-empty AND heap.min <= start:
    heap.pop()     // free the earliest-ending room
  heap.push(end)  // assign meeting to (reused or new) room
return heap.size()
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Sorter** — Sort meetings by start time.
2. **Min-Heap** — Stores end times of all currently occupied rooms.
3. **Room Recycler** — Check heap's minimum; if freed → pop and reuse.
4. **Room Allocator** — Push current meeting's end time onto heap.
5. **Output** — Return final heap size = minimum rooms needed.

### Data Flow Diagram

```
intervals = [[0,30],[5,10],[15,25]]
After sort by start: [[0,30],[5,10],[15,25]]

heap = []

[0,30]:  heap empty → allocate new room    → heap=[30]        rooms=1
[5,10]:  heap.min=30 > 5 → busy → new room → heap=[10,30]     rooms=2
[15,25]: heap.min=10 <= 15 → reuse!        → pop 10, push 25
                                              heap=[25,30]     rooms=2

Output: 2 ✓

---

intervals = [[7,10],[2,4]]
After sort: [[2,4],[7,10]]

heap=[]
[2,4]:  new room → heap=[4]           rooms=1
[7,10]: heap.min=4 <= 7 → reuse room → pop 4, push 10
                                        heap=[10]            rooms=1

Output: 1 ✓
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class MeetingRoomsII {

    /**
     * Find the minimum number of conference rooms required for all meetings.
     *
     * Strategy: Sort by start time + min-heap of room end times.
     * For each meeting, if the room ending soonest is free (end <= start),
     * reuse it. Otherwise, allocate a new room.
     * Heap size = rooms in use; final heap size = minimum rooms needed.
     *
     * @param intervals array of [start, end] meeting time intervals
     * @return minimum number of conference rooms required
     *
     * Time:  O(n log n) — sort + n heap operations
     * Space: O(n) — heap stores up to n end times
     */
    public int minMeetingRooms(int[][] intervals) {
        if (intervals.length == 0) return 0;

        // Sort by start time
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        // Min-heap: tracks end times of currently occupied rooms
        PriorityQueue<Integer> heap = new PriorityQueue<>();

        for (int[] meeting : intervals) {
            int start = meeting[0], end = meeting[1];

            // If the earliest-ending room is free, reuse it
            if (!heap.isEmpty() && heap.peek() <= start) {
                heap.poll(); // free the room
            }

            heap.offer(end); // assign this meeting to a room (new or reused)
        }

        return heap.size(); // total rooms in use = minimum needed
    }

    // Two-pointer alternative: sort starts and ends separately
    public int minMeetingRooms_TwoPointer(int[][] intervals) {
        int n = intervals.length;
        int[] starts = new int[n], ends = new int[n];
        for (int i = 0; i < n; i++) {
            starts[i] = intervals[i][0];
            ends[i]   = intervals[i][1];
        }
        Arrays.sort(starts);
        Arrays.sort(ends);

        int rooms = 0, endPtr = 0;
        for (int i = 0; i < n; i++) {
            if (starts[i] < ends[endPtr]) {
                rooms++; // new room needed
            } else {
                endPtr++; // a room freed up
            }
        }
        return rooms;
    }

    public static void main(String[] args) {
        MeetingRoomsII sol = new MeetingRoomsII();

        // Example 1: 2 rooms needed
        System.out.println("[[0,30],[5,10],[15,25]] → " +
            sol.minMeetingRooms(new int[][]{{0,30},{5,10},{15,25}})); // 2

        // Example 2: 1 room suffices
        System.out.println("[[7,10],[2,4]] → " +
            sol.minMeetingRooms(new int[][]{{7,10},{2,4}})); // 1

        // Example 3: All overlap → n rooms
        System.out.println("[[1,5],[1,5],[1,5]] → " +
            sol.minMeetingRooms(new int[][]{{1,5},{1,5},{1,5}})); // 3

        // Example 4: None overlap → 1 room
        System.out.println("[[1,2],[3,4],[5,6]] → " +
            sol.minMeetingRooms(new int[][]{{1,2},{3,4},{5,6}})); // 1

        // Example 5: Touching meetings can share a room
        System.out.println("[[1,4],[4,7],[7,10]] → " +
            sol.minMeetingRooms(new int[][]{{1,4},{4,7},{7,10}})); // 1
    }
}
```

### Output

```
[[0,30],[5,10],[15,25]] → 2
[[7,10],[2,4]] → 1
[[1,5],[1,5],[1,5]] → 3
[[1,2],[3,4],[5,6]] → 1
[[1,4],[4,7],[7,10]] → 1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
import heapq

def min_meeting_rooms(intervals: list[list[int]]) -> int:
    """
    Find the minimum number of conference rooms for all meetings.

    Strategy: Sort by start time + min-heap of room end times.
    If the room with the earliest end time frees up (end <= current start),
    reuse it. Otherwise, allocate a new room.
    Final heap size = minimum rooms needed.

    Args:
        intervals: List of [start, end] meeting intervals

    Returns:
        Minimum number of conference rooms required

    Time:  O(n log n)
    Space: O(n)
    """
    if not intervals:
        return 0

    intervals.sort(key=lambda x: x[0])  # sort by start time

    heap = []  # min-heap of room end times

    for start, end in intervals:
        if heap and heap[0] <= start:
            heapq.heappop(heap)  # reuse the earliest-freed room

        heapq.heappush(heap, end)  # assign meeting to room

    return len(heap)  # rooms in use = minimum rooms needed


def min_meeting_rooms_two_pointer(intervals: list[list[int]]) -> int:
    """Two-pointer alternative: sort starts and ends separately."""
    starts = sorted(iv[0] for iv in intervals)
    ends   = sorted(iv[1] for iv in intervals)

    rooms, end_ptr = 0, 0
    for start in starts:
        if start < ends[end_ptr]:
            rooms += 1   # no room freed up → need a new one
        else:
            end_ptr += 1 # a room freed up → reuse it (net rooms unchanged)
    return rooms


if __name__ == "__main__":
    test_cases = [
        ([[0,30],[5,10],[15,25]], 2),
        ([[7,10],[2,4]],          1),
        ([[1,5],[1,5],[1,5]],     3),
        ([[1,2],[3,4],[5,6]],     1),
        ([[1,4],[4,7],[7,10]],    1),
    ]

    for intervals, expected in test_cases:
        heap_result = min_meeting_rooms([iv[:] for iv in intervals])
        tp_result   = min_meeting_rooms_two_pointer(intervals)
        status = "PASS" if heap_result == expected else "FAIL"
        print(f"[{status}] intervals={intervals}")
        print(f"  Heap: {heap_result}, Two-Pointer: {tp_result}, Expected: {expected}\n")
```

### Output

```
[PASS] intervals=[[0, 30], [5, 10], [15, 25]]
  Heap: 2, Two-Pointer: 2, Expected: 2

[PASS] intervals=[[7, 10], [2, 4]]
  Heap: 1, Two-Pointer: 1, Expected: 1

[PASS] intervals=[[1, 5], [1, 5], [1, 5]]
  Heap: 3, Two-Pointer: 3, Expected: 3

[PASS] intervals=[[1, 2], [3, 4], [5, 6]]
  Heap: 1, Two-Pointer: 1, Expected: 1

[PASS] intervals=[[1, 4], [4, 7], [7, 10]]
  Heap: 1, Two-Pointer: 1, Expected: 1
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why use a min-heap and not just count overlaps?**
A: A min-heap efficiently tracks which room frees up earliest. Without it, we'd need to scan all rooms for each new meeting (O(n) per meeting = O(n²) total). The heap gives O(log n) per meeting.

**Q: What does the heap represent at any point?**
A: The heap contains the end times of all currently occupied rooms. Its size = number of active rooms. Its minimum = the soonest any room will become free.

**Q: Why does heap size at the end equal the minimum rooms needed?**
A: Every push adds a room; every pop reuses one. After processing all meetings, the heap holds one entry per room that's still in use (occupied by the last meeting assigned to it). This count equals the total number of rooms allocated.

**Q: Explain the two-pointer approach.**
A: Sort starts and ends separately. Use two pointers `i` (starts) and `j` (ends). For each start time, if it's before `ends[j]`, no room is free — allocate one. Otherwise, advance `j` (a room freed up). This is equivalent to the heap approach but avoids the heap data structure.

**Q: What's the connection between this problem and the "maximum overlap" concept?**
A: The minimum rooms needed = maximum number of meetings happening simultaneously at any point. The heap approach computes this by tracking concurrent meetings; the two-pointer approach does the same by comparing sorted event times.

**Q: When would you prefer two-pointer over min-heap?**
A: Two-pointer uses O(n) extra space for two sorted arrays but avoids heap overhead. Min-heap is more intuitive for actual room-assignment questions (you can track which meeting is in which room). Both are O(n log n).


---

9. Summary

## Summary

| Attribute             | Value                                                               |
| --------------------- | ------------------------------------------------------------------- |
| **Problem**           | Minimum conference rooms to hold all meetings simultaneously        |
| **Optimal Algorithm** | Sort by start + Min-Heap of room end times                         |
| **Time Complexity**   | O(n log n)                                                          |
| **Space Complexity**  | O(n)                                                                |
| **Key Insight**       | Reuse room if earliest-ending room frees up before next meeting starts |

### Core Pattern

> Sort by start time. For each meeting, check if the room ending soonest (heap minimum) is free. If yes, reuse it; otherwise, open a new room. Heap size = rooms in use. Final heap size = answer.

### When to Use This Pattern

- Any "minimum resources to cover all intervals" problem.
- Related: Meeting Rooms (I), Non-overlapping Intervals, Task Scheduler, Car Pooling, Employee Free Time.

