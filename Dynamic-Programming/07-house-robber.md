# House Robber

---

1. What is meant by House Robber?

## What is House Robber?

**House Robber** is a classic DP problem where:

- You are a robber planning to rob houses along a street.
- Each house has a certain amount of money `nums[i]`.
- You cannot rob two **adjacent** houses (alarm will trigger).

**Goal:** Return the **maximum amount of money** you can rob without triggering the alarm.

**Constraints:**
- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 400`

**Example:**

```
Input:  nums = [2, 7, 9, 3, 1]
Output: 12
Explanation: Rob houses 0, 2, 4 → 2 + 9 + 1 = 12
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Maximize the total amount robbed.
- Cannot rob two adjacent (consecutive) houses.
- Must consider all valid non-adjacent subsets.

### Non-Functional Requirements

| Concern         | Expectation                                      |
| --------------- | ------------------------------------------------ |
| **Scale**       | n up to 100 — O(n) is trivially fast             |
| **Latency**     | O(n) — runs in microseconds                      |
| **Correctness** | Must find global maximum, not just alternating   |
| **Space**       | Reducible to O(1) with two variables             |

### Clarifying Questions to Ask

- Can we skip more than one house? → Yes, we can skip any number.
- All values non-negative? → Yes (`nums[i] >= 0`).
- Can we rob zero houses? → Yes (if all values are 0, result is 0).
- Houses arranged in a line or a circle? → Line (circle = House Robber II).


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Brute Force (All Subsets)

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — try all subsets of non-adjacent houses |
| **Space** | O(n) — recursion stack |

### Approach 2: DP with Array

| | Complexity |
| --- | --- |
| **Time** | O(n) — single pass |
| **Space** | O(n) — dp array |

### Approach 3: Space-Optimized DP (Two Variables)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single pass |
| **Space** | O(1) — only prev and curr |

**Why O(1) space?** At each step, we only need the two most recent DP values — no need to store the entire array.


---

4. Which Algorithm and Why?

## Algorithm: Space-Optimized Dynamic Programming

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2ⁿ) | O(n) | No |
| DP Array | O(n) | O(n) | Works, but wasteful |
| **Two-Variable DP** | **O(n)** | **O(1)** | **Yes — optimal** |

### Key Recurrence

`dp[i]` = maximum money from robbing houses `0..i`.

```
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

Meaning: at house `i`, either:
- **Skip it** → take `dp[i-1]` (best from previous house)
- **Rob it** → take `dp[i-2] + nums[i]` (best from 2 houses back + current)

Since we only need `dp[i-1]` and `dp[i-2]`, use two variables `prev` and `curr`.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Input Layer** — Receives `nums[]`
2. **Base Case Handler** — Handle n=1 or n=2 directly
3. **DP Stepper** — Iteratively updates `prev` and `curr`
4. **Output Layer** — Returns `curr` (max money)

### Data Flow Diagram

```
nums = [2, 7, 9, 3, 1]

prev = 0 (max before any house)
curr = 0 (current max)

House 0 (val=2): new_curr = max(curr=0, prev=0 + 2) = 2  → prev=0, curr=2
House 1 (val=7): new_curr = max(curr=2, prev=0 + 7) = 7  → prev=2, curr=7
House 2 (val=9): new_curr = max(curr=7, prev=2 + 9) = 11 → prev=7, curr=11
House 3 (val=3): new_curr = max(curr=11, prev=7 + 3) = 11 → prev=11, curr=11
House 4 (val=1): new_curr = max(curr=11, prev=11 + 1) = 12 → prev=11, curr=12

Output: 12
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class HouseRobber {

    /**
     * Find the maximum amount robbed from a row of houses without robbing adjacent ones.
     *
     * DP: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
     * Space-optimized using two variables instead of a full array.
     *
     * @param nums array of house values
     * @return maximum money that can be robbed
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public int rob(int[] nums) {
        int prev = 0; // best money up to 2 houses back
        int curr = 0; // best money up to the previous house

        for (int num : nums) {
            int next = Math.max(curr, prev + num); // skip or rob current house
            prev = curr;
            curr = next;
        }

        return curr;
    }

    public static void main(String[] args) {
        HouseRobber sol = new HouseRobber();

        // Example 1
        System.out.println("Input: [1,2,3,1]");
        System.out.println("Output: " + sol.rob(new int[]{1, 2, 3, 1})); // 4 (rob 0+2)

        // Example 2
        System.out.println("\nInput: [2,7,9,3,1]");
        System.out.println("Output: " + sol.rob(new int[]{2, 7, 9, 3, 1})); // 12 (rob 0+2+4)

        // Example 3: Single house
        System.out.println("\nInput: [5]");
        System.out.println("Output: " + sol.rob(new int[]{5})); // 5

        // Example 4: Two houses — pick larger
        System.out.println("\nInput: [3, 10]");
        System.out.println("Output: " + sol.rob(new int[]{3, 10})); // 10

        // Example 5: All same value
        System.out.println("\nInput: [4,4,4,4,4]");
        System.out.println("Output: " + sol.rob(new int[]{4, 4, 4, 4, 4})); // 12 (rob 3 houses)
    }
}
```

### Output

```
Input: [1,2,3,1]
Output: 4

Input: [2,7,9,3,1]
Output: 12

Input: [5]
Output: 5

Input: [3, 10]
Output: 10

Input: [4,4,4,4,4]
Output: 12
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def rob(nums: list[int]) -> int:
    """
    Find the maximum money from robbing non-adjacent houses.

    Strategy: Space-optimized DP.
    At each house, decide: skip it (keep prev max) or rob it (prev_prev + current value).
    Track only the last two DP values — no full array needed.

    Args:
        nums: List of house money values

    Returns:
        Maximum money robbed without triggering adjacent-house alarm

    Time:  O(n)
    Space: O(1)
    """
    prev, curr = 0, 0  # prev = dp[i-2], curr = dp[i-1]

    for num in nums:
        prev, curr = curr, max(curr, prev + num)

    return curr


if __name__ == "__main__":
    test_cases = [
        ([1, 2, 3, 1],    4),
        ([2, 7, 9, 3, 1], 12),
        ([5],             5),
        ([3, 10],         10),
        ([4, 4, 4, 4, 4], 12),
        ([0, 0, 0],       0),
    ]

    for nums, expected in test_cases:
        result = rob(nums)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] nums={nums} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] nums=[1, 2, 3, 1] → Output: 4, Expected: 4
[PASS] nums=[2, 7, 9, 3, 1] → Output: 12, Expected: 12
[PASS] nums=[5] → Output: 5, Expected: 5
[PASS] nums=[3, 10] → Output: 10, Expected: 10
[PASS] nums=[4, 4, 4, 4, 4] → Output: 12, Expected: 12
[PASS] nums=[0, 0, 0] → Output: 0, Expected: 0
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why is `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` correct?**
A: At house `i`, you have exactly two choices: skip it (best from i-1) or rob it (must skip i-1, so take best from i-2 plus current value). The max of these two is optimal for position `i`.

**Q: What if all values are 0?**
A: The algorithm returns 0 correctly — robbing 0-value houses yields 0, which is the maximum.

**Q: Can we skip more than one house?**
A: Yes, but we don't need to explicitly handle it. If skipping two is better, `dp[i-2]` will reflect the best result from two positions back, which may itself have skipped houses.

**Q: How do you handle the edge case where n=1?**
A: The loop runs once: `prev=0, curr=0`, then `curr = max(0, 0 + nums[0]) = nums[0]`. Returns correctly.

**Q: How does this extend to House Robber II (circular arrangement)?**
A: Run the linear House Robber algorithm twice: once on `nums[0..n-2]` (exclude last) and once on `nums[1..n-1]` (exclude first). Return the maximum — this handles the constraint that first and last houses are adjacent.

**Q: Is there a greedy solution?**
A: No. Consider `[3, 10, 3, 1, 2]`. Greedy (always pick highest available non-adjacent) picks 10+2=12, but 3+3+2=8 or 10+1=11. The greedy picks 10 then 2 for 12, but DP confirms this is correct here. However, in general, greedy can fail (e.g., `[5, 1, 1, 5]`: greedy picks 5+1=6, DP finds 5+5=10).


---

9. Summary

## Summary

| Attribute             | Value                                                         |
| --------------------- | ------------------------------------------------------------- |
| **Problem**           | Max money from non-adjacent houses in a line                  |
| **Optimal Algorithm** | Space-Optimized DP (two variables)                            |
| **Time Complexity**   | O(n)                                                          |
| **Space Complexity**  | O(1)                                                          |
| **Key Insight**       | `dp[i] = max(skip current, rob current + dp[i-2])`            |

### Core Pattern

> At every step, decide: take the current item (and skip the previous) or skip the current item (keep the previous best). This "include vs. exclude" decision is the heart of most 1D DP problems.

### When to Use This Pattern

- Any "maximize/minimize over a sequence with an exclusion constraint" problem.
- Related: House Robber II (circular), Maximum Alternating Subsequence, Stock Buy/Sell with Cooldown.

