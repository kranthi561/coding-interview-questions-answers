# Longest Increasing Subsequence

---

1. What is meant by Longest Increasing Subsequence?

## What is Longest Increasing Subsequence (LIS)?

**LIS** is a classic DP problem where:

- You are given an integer array `nums`.
- A **subsequence** preserves the relative order of elements but skips some.
- A subsequence is **strictly increasing** if each element is greater than the previous.

**Goal:** Return the **length** of the longest strictly increasing subsequence.

**Constraints:**
- `1 <= nums.length <= 2500`
- `-10⁴ <= nums[i] <= 10⁴`

**Example:**

```
Input:  nums = [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4
Explanation: LIS is [2, 3, 7, 101] or [2, 5, 7, 101]
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Find the length (not the actual sequence) of the LIS.
- Subsequence must be **strictly increasing** (not non-decreasing).
- Elements don't need to be contiguous.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | n up to 2500 (O(n²) is acceptable; O(n log n) better)|
| **Latency**     | O(n log n) runs in sub-milliseconds                  |
| **Correctness** | Must count all valid increasing subsequences         |
| **Output**      | Length only, not the actual subsequence              |

### Clarifying Questions to Ask

- Strictly increasing or non-decreasing? → Strictly increasing (no equal values).
- Return length or the actual subsequence? → Length only.
- Can the array have duplicates? → Yes, but they cannot both be in the LIS.
- Multiple LIS of same length? → Return the length (any one is fine).


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Brute Force (All Subsequences)

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ × n) — generate all 2ⁿ subsequences and check |
| **Space** | O(n) — recursion stack |

### Approach 2: O(n²) DP

| | Complexity |
| --- | --- |
| **Time** | O(n²) — for each element, check all prior elements |
| **Space** | O(n) — dp array |

### Approach 3: O(n log n) Patience Sorting (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — binary search on a "tails" array |
| **Space** | O(n) — tails array |

**Why O(n log n)?** The `tails` array is always sorted, enabling binary search to find the position to update — O(log n) per element.


---

4. Which Algorithm and Why?

## Algorithm: Patience Sorting with Binary Search

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2ⁿ n) | O(n) | No |
| O(n²) DP | O(n²) | O(n) | Acceptable for n≤2500 |
| **Patience Sorting** | **O(n log n)** | **O(n)** | **Yes — optimal** |

### O(n²) DP Idea

`dp[i]` = length of LIS ending at index `i`.  
For each `i`, check all `j < i` where `nums[j] < nums[i]`: `dp[i] = max(dp[j] + 1)`.

### Patience Sorting Idea (O(n log n))

Maintain a `tails` array where `tails[i]` = smallest tail element of any increasing subsequence of length `i+1`.

For each number:
- If it's larger than all tails → append it (LIS grows by 1).
- Otherwise → binary search for the first tail ≥ num, replace it (keep tails as small as possible).

The length of `tails` at the end = LIS length.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Input Layer** — Receives `nums[]`
2. **Tails Array** — Maintains the "patience pile" sorted structure
3. **Binary Search Module** — Finds the position to place each element in `tails`
4. **Output Layer** — Returns `tails.length`

### Data Flow Diagram (Patience Sorting)

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

Start: tails = []

10  → tails=[], 10 > all → append    → tails=[10]
9   → bisect_left([10], 9)=0 → replace → tails=[9]
2   → bisect_left([9], 2)=0 → replace  → tails=[2]
5   → bisect_left([2], 5)=1 → append   → tails=[2,5]
3   → bisect_left([2,5], 3)=1 → replace → tails=[2,3]
7   → bisect_left([2,3], 7)=2 → append  → tails=[2,3,7]
101 → bisect_left([2,3,7], 101)=3 → append → tails=[2,3,7,101]
18  → bisect_left([2,3,7,101], 18)=3 → replace → tails=[2,3,7,18]

LIS length = len(tails) = 4
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.ArrayList;
import java.util.List;

public class LongestIncreasingSubsequence {

    /**
     * Returns LIS length using patience sorting + binary search.
     *
     * tails[i] = smallest tail of any increasing subsequence of length i+1.
     * We use binary search to find the right position for each element.
     *
     * @param nums input array
     * @return length of longest strictly increasing subsequence
     *
     * Time:  O(n log n)
     * Space: O(n)
     */
    public int lengthOfLIS(int[] nums) {
        List<Integer> tails = new ArrayList<>();

        for (int num : nums) {
            int pos = binarySearch(tails, num);

            if (pos == tails.size()) {
                tails.add(num);          // extend LIS
            } else {
                tails.set(pos, num);     // replace to keep smallest tail
            }
        }

        return tails.size();
    }

    // Binary search: find leftmost index where tails[index] >= target
    private int binarySearch(List<Integer> tails, int target) {
        int lo = 0, hi = tails.size();
        while (lo < hi) {
            int mid = (lo + hi) / 2;
            if (tails.get(mid) < target) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        return lo;
    }

    // O(n^2) DP version for comparison
    public int lengthOfLIS_DP(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        java.util.Arrays.fill(dp, 1); // every element is LIS of length 1 by itself

        int maxLen = 1;
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            maxLen = Math.max(maxLen, dp[i]);
        }
        return maxLen;
    }

    public static void main(String[] args) {
        LongestIncreasingSubsequence sol = new LongestIncreasingSubsequence();

        // Example 1
        int[] nums1 = {10, 9, 2, 5, 3, 7, 101, 18};
        System.out.println("Input: [10,9,2,5,3,7,101,18]");
        System.out.println("Output: " + sol.lengthOfLIS(nums1)); // 4

        // Example 2: Already sorted
        int[] nums2 = {0, 1, 0, 3, 2, 3};
        System.out.println("\nInput: [0,1,0,3,2,3]");
        System.out.println("Output: " + sol.lengthOfLIS(nums2)); // 4

        // Example 3: All same
        int[] nums3 = {7, 7, 7, 7};
        System.out.println("\nInput: [7,7,7,7]");
        System.out.println("Output: " + sol.lengthOfLIS(nums3)); // 1

        // Example 4: Strictly decreasing
        int[] nums4 = {5, 4, 3, 2, 1};
        System.out.println("\nInput: [5,4,3,2,1]");
        System.out.println("Output: " + sol.lengthOfLIS(nums4)); // 1
    }
}
```

### Output

```
Input: [10,9,2,5,3,7,101,18]
Output: 4

Input: [0,1,0,3,2,3]
Output: 4

Input: [7,7,7,7]
Output: 1

Input: [5,4,3,2,1]
Output: 1
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
import bisect

def length_of_lis(nums: list[int]) -> int:
    """
    Find the length of the longest strictly increasing subsequence.

    Strategy: Patience sorting with binary search (O(n log n)).
    tails[i] = smallest ending element of any increasing subsequence of length i+1.
    For each number, binary-search its position in tails:
      - If larger than all → extend (LIS grows).
      - Otherwise → replace the first element >= num (keep tails minimal).

    Args:
        nums: Input integer array

    Returns:
        Length of the longest strictly increasing subsequence

    Time:  O(n log n)
    Space: O(n)
    """
    tails = []

    for num in nums:
        pos = bisect.bisect_left(tails, num)  # find leftmost position where tails[pos] >= num
        if pos == len(tails):
            tails.append(num)   # num is larger than all tails → extend
        else:
            tails[pos] = num    # replace to maintain smallest possible tail

    return len(tails)


def length_of_lis_dp(nums: list[int]) -> int:
    """O(n^2) DP version for clarity."""
    n = len(nums)
    dp = [1] * n  # each element is an LIS of length 1 by itself

    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)


if __name__ == "__main__":
    test_cases = [
        ([10, 9, 2, 5, 3, 7, 101, 18], 4),
        ([0, 1, 0, 3, 2, 3], 4),
        ([7, 7, 7, 7], 1),
        ([5, 4, 3, 2, 1], 1),
        ([1, 3, 6, 7, 9, 4, 10, 5, 6], 6),
    ]

    for nums, expected in test_cases:
        result = length_of_lis(nums)
        status = "✓" if result == expected else "✗"
        print(f"{status} Input: {nums}")
        print(f"  Output: {result}, Expected: {expected}\n")
```

### Output

```
✓ Input: [10, 9, 2, 5, 3, 7, 101, 18]
  Output: 4, Expected: 4

✓ Input: [0, 1, 0, 3, 2, 3]
  Output: 4, Expected: 4

✓ Input: [7, 7, 7, 7]
  Output: 1, Expected: 1

✓ Input: [5, 4, 3, 2, 1]
  Output: 1, Expected: 1

✓ Input: [1, 3, 6, 7, 9, 4, 10, 5, 6]
  Output: 6, Expected: 6
```


---

8. Interview Questions and Answers

## Q&A

**Q: What is the difference between a subsequence and a subarray?**
A: A subarray must be contiguous. A subsequence preserves order but can skip elements. LIS uses subsequences.

**Q: What does the `tails` array actually represent?**
A: `tails[i]` stores the smallest possible last element of any increasing subsequence of length `i+1`. This greedy minimization is what enables O(n log n) efficiency.

**Q: Why is the `tails` array always sorted?**
A: By construction — we only append elements larger than the current max (sorted extension), or replace elements in a way that maintains the sorted invariant.

**Q: Why do we use `bisect_left` (not `bisect_right`)?**
A: We want strictly increasing sequences. `bisect_left` finds the first position where `tails[pos] >= num`, replacing equal elements (not allowing duplicates in the subsequence).

**Q: Can you reconstruct the actual LIS (not just its length)?**
A: Yes, but it requires tracking parent pointers — for each element, record which element preceded it in the LIS. This makes reconstruction O(n) after the main O(n²) or O(n log n) pass.

**Q: When would you use O(n²) DP instead of O(n log n)?**
A: When you need to reconstruct the actual subsequence (parent tracking is simpler with DP), or when n is small and clarity matters more than performance.


---

9. Summary

## Summary

| Attribute             | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| **Problem**           | Find length of longest strictly increasing subsequence                |
| **Optimal Algorithm** | Patience Sorting + Binary Search                                      |
| **Time Complexity**   | O(n log n)                                                            |
| **Space Complexity**  | O(n)                                                                  |
| **Key Insight**       | Maintain a sorted `tails` array; binary search to place each element |

### Core Pattern

> Keep the smallest possible ending element for each subsequence length. Binary search the right position for each new element. The array length at the end = LIS length.

### When to Use This Pattern

- Finding the longest increasing subsequence (or non-decreasing variant).
- Related: Russian Doll Envelopes, Longest Chain of Pairs, Patience Sorting.
- Any "longest chain satisfying a monotone constraint" formulation.

