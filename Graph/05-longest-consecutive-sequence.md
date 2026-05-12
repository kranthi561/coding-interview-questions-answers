# Longest Consecutive Sequence

---

1. What is meant by Longest Consecutive Sequence?

## What is Longest Consecutive Sequence?

**Longest Consecutive Sequence** is a problem where:

- You are given an unsorted integer array `nums`.

**Goal:** Find the length of the **longest sequence of consecutive integers** (e.g., `4, 5, 6, 7`) present in the array, in **O(n)** time.

**Constraints:**
- `0 <= nums.length <= 10⁵`
- `-10⁹ <= nums[i] <= 10⁹`

**Example:**

```
Input:  nums = [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: Consecutive sequence is [1, 2, 3, 4]

Input:  nums = [0, 3, 7, 2, 5, 8, 4, 6, 0, 1]
Output: 9
Explanation: [0, 1, 2, 3, 4, 5, 6, 7, 8]
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Elements in the sequence must be consecutive integers (differ by exactly 1).
- Elements don't need to be contiguous in the original array.
- Duplicate values are allowed but count as one occurrence.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | n up to 10⁵ — must be O(n), not O(n log n)         |
| **Latency**     | O(n) — single HashSet-based pass                   |
| **Correctness** | Duplicates should not extend the sequence length   |
| **Edge Cases**  | Empty array → 0, all same values → 1               |

### Clarifying Questions to Ask

- Must elements be distinct? → No, but duplicates are treated as one.
- Must the sequence appear consecutively in the array? → No, just in value.
- Return length or the sequence itself? → Length only.
- Can values be negative? → Yes.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Sort and Scan

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — sorting cost |
| **Space** | O(1) or O(n) depending on sort implementation |

### Approach 2: HashSet (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — each number is visited at most twice |
| **Space** | O(n) — HashSet stores all values |

**Why O(n)?** Each element is the starting point of a sequence at most once (only when `n-1` is not in the set). The inner while-loop across all starting points runs at most `n` times total — amortized O(1) per element.


---

4. Which Algorithm and Why?

## Algorithm: HashSet with Sequence Start Detection

### Why HashSet?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(n³) | O(1) | No |
| Sort + Scan | O(n log n) | O(1) | No — violates O(n) requirement |
| **HashSet** | **O(n)** | **O(n)** | **Yes** |

### Key Insight

Only start counting a sequence from a **sequence start** — a number `n` where `n-1` is NOT in the set. This avoids counting the same sequence multiple times.

```
numSet = {100, 4, 200, 1, 3, 2}

For n=100: 99 not in set → start sequence: 100,101... → length=1
For n=4:   3 in set → skip (not a start)
For n=200: 199 not in set → start sequence: 200,201... → length=1
For n=1:   0 not in set → start sequence: 1,2,3,4 → length=4 ✓
For n=3:   2 in set → skip
For n=2:   1 in set → skip

Max = 4
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **HashSet Builder** — O(n) insertion of all values.
2. **Start Detector** — Check if `n-1` is missing (this is a sequence start).
3. **Sequence Counter** — Extend from start while `n+1, n+2, ...` are in the set.
4. **Max Tracker** — Keep track of the longest sequence seen.
5. **Output** — Return the max length.

### Data Flow Diagram

```
nums = [100, 4, 200, 1, 3, 2]
numSet = {1, 2, 3, 4, 100, 200}

Iteration:
  n=100 → 99 ∉ set → count: 100 ✓, 101 ✗ → length=1
  n=4   → 3 ∈ set → skip
  n=200 → 199 ∉ set → count: 200 ✓, 201 ✗ → length=1
  n=1   → 0 ∉ set → count: 1✓,2✓,3✓,4✓,5✗ → length=4 ← max!
  n=3   → 2 ∈ set → skip
  n=2   → 1 ∈ set → skip

Output: 4
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class LongestConsecutiveSequence {

    /**
     * Find the length of the longest consecutive integer sequence in an array.
     *
     * Strategy: HashSet for O(1) lookup.
     * Only start counting from a sequence start (n where n-1 is not in the set).
     * This ensures each number is counted in at most one sequence.
     *
     * @param nums unsorted integer array (may have duplicates)
     * @return length of the longest consecutive sequence
     *
     * Time:  O(n)
     * Space: O(n)
     */
    public int longestConsecutive(int[] nums) {
        Set<Integer> numSet = new HashSet<>();
        for (int n : nums) numSet.add(n); // deduplicate and enable O(1) lookup

        int maxLen = 0;

        for (int n : numSet) {
            // Only start counting if n is the beginning of a sequence
            if (!numSet.contains(n - 1)) {
                int currentLen = 1;
                int current = n;

                // Extend the sequence as far as possible
                while (numSet.contains(current + 1)) {
                    current++;
                    currentLen++;
                }

                maxLen = Math.max(maxLen, currentLen);
            }
        }

        return maxLen;
    }

    public static void main(String[] args) {
        LongestConsecutiveSequence sol = new LongestConsecutiveSequence();

        // Example 1
        System.out.println("Input: [100,4,200,1,3,2]");
        System.out.println("Output: " + sol.longestConsecutive(new int[]{100,4,200,1,3,2})); // 4

        // Example 2: Longer sequence
        System.out.println("\nInput: [0,3,7,2,5,8,4,6,0,1]");
        System.out.println("Output: " + sol.longestConsecutive(new int[]{0,3,7,2,5,8,4,6,0,1})); // 9

        // Example 3: Empty array
        System.out.println("\nInput: []");
        System.out.println("Output: " + sol.longestConsecutive(new int[]{})); // 0

        // Example 4: All duplicates
        System.out.println("\nInput: [5,5,5]");
        System.out.println("Output: " + sol.longestConsecutive(new int[]{5,5,5})); // 1

        // Example 5: Negative numbers
        System.out.println("\nInput: [-3,-2,-1,0,1]");
        System.out.println("Output: " + sol.longestConsecutive(new int[]{-3,-2,-1,0,1})); // 5
    }
}
```

### Output

```
Input: [100,4,200,1,3,2]
Output: 4

Input: [0,3,7,2,5,8,4,6,0,1]
Output: 9

Input: []
Output: 0

Input: [5,5,5]
Output: 1

Input: [-3,-2,-1,0,1]
Output: 5
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def longest_consecutive(nums: list[int]) -> int:
    """
    Find the length of the longest consecutive integer sequence.

    Strategy: HashSet for O(1) lookup.
    For each number n, only start counting if n-1 is NOT in the set
    (meaning n is the start of a new sequence).
    Extend the sequence while n+1, n+2, ... are found in the set.

    Args:
        nums: Unsorted integer array (may have duplicates)

    Returns:
        Length of the longest consecutive sequence

    Time:  O(n)  — each number processed at most twice
    Space: O(n)  — HashSet
    """
    num_set = set(nums)  # deduplication + O(1) lookup
    max_len = 0

    for n in num_set:
        if n - 1 not in num_set:  # n is the start of a sequence
            current_len = 1
            current = n

            while current + 1 in num_set:
                current += 1
                current_len += 1

            max_len = max(max_len, current_len)

    return max_len


if __name__ == "__main__":
    test_cases = [
        ([100, 4, 200, 1, 3, 2],        4),
        ([0, 3, 7, 2, 5, 8, 4, 6, 0, 1], 9),
        ([],                             0),
        ([5, 5, 5],                      1),
        ([-3, -2, -1, 0, 1],             5),
        ([1],                            1),
    ]

    for nums, expected in test_cases:
        result = longest_consecutive(nums)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] nums={nums} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] nums=[100, 4, 200, 1, 3, 2] → Output: 4, Expected: 4
[PASS] nums=[0, 3, 7, 2, 5, 8, 4, 6, 0, 1] → Output: 9, Expected: 9
[PASS] nums=[] → Output: 0, Expected: 0
[PASS] nums=[5, 5, 5] → Output: 1, Expected: 1
[PASS] nums=[-3, -2, -1, 0, 1] → Output: 5, Expected: 5
[PASS] nums=[1] → Output: 1, Expected: 1
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why iterate over `numSet` instead of `nums`?**
A: Duplicates in `nums` would trigger the inner while-loop multiple times for the same sequence start, wasting work. Iterating over `numSet` (deduplicated) ensures each start is processed exactly once.

**Q: Why is the time complexity O(n) even though there's a nested while loop?**
A: The key insight is amortized analysis. Each number is added to at most one sequence (the one where it is the start or an extension). The inner while-loop, summed across all starting points, runs at most `n` times total — making the overall complexity O(n).

**Q: Can sorting solve this in O(n)?**
A: No. Sorting is O(n log n) regardless. The HashSet approach achieves O(n) by trading space for time.

**Q: How do you handle duplicates?**
A: Converting to a HashSet automatically deduplicates. Even if `[1, 1, 2]` is the input, the set is `{1, 2}` and the sequence length is 2.

**Q: What if the problem required returning the actual sequence, not just the length?**
A: Store the start of the longest sequence and its length. Then iterate from `start` to `start + length - 1` to reconstruct it in O(length) time.

**Q: Why is this classified under Graph?**
A: Conceptually, consecutive integer relationships form an implicit graph (edges between n and n±1). The problem is essentially finding the longest connected path in this implicit integer graph.


---

9. Summary

## Summary

| Attribute             | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Problem**           | Find the length of the longest consecutive integer sequence       |
| **Optimal Algorithm** | HashSet with sequence-start detection                             |
| **Time Complexity**   | O(n)                                                              |
| **Space Complexity**  | O(n)                                                              |
| **Key Insight**       | Only count from `n` if `n-1` is not in the set (sequence start) |

### Core Pattern

> Load all values into a HashSet for O(1) lookup. For each value, check if it's a sequence start (no left neighbor). If so, extend rightward and count. Amortized O(n) total.

### When to Use This Pattern

- Finding contiguous ranges in an unordered set of values.
- Related: Missing Ranges, Find All Numbers Disappeared in an Array, Union-Find-based range merging.

