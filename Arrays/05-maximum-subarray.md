# Maximum Subarray

---

<details>
<summary><strong>1. What is meant by Maximum Subarray?</strong></summary>

## What is Maximum Subarray?

Given an integer array `nums`, find the **contiguous subarray** (containing at least one number) which has the **largest sum**, and return its sum.

**Example:**
```
Input:  nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6
Explanation: The subarray [4, -1, 2, 1] has the largest sum = 6.
```

```
Input:  nums = [1]
Output: 1
```

```
Input:  nums = [5, 4, -1, 7, 8]
Output: 23
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Find the maximum sum of any contiguous subarray.
- Subarray must have at least one element.
- Return the sum (not the subarray itself, unless follow-up asked).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 10⁵ elements |
| **Time** | O(n) expected |
| **Values** | Can be negative |

### Clarifying Questions
- Can all numbers be negative? → Yes; return the largest single element.
- Do we need to return the subarray or just its sum? → Just the sum.
- Is an empty subarray valid? → No, at least one element required.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force (all subarrays) | O(n²) | O(1) |
| Divide and Conquer | O(n log n) | O(log n) |
| **Kadane's Algorithm** | **O(n)** | **O(1)** |

**Kadane's** is optimal — single pass, constant space.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Kadane's Algorithm

### Core Idea
At each index, decide: **extend the current subarray** or **start a new subarray here**?

If the current running sum becomes negative, it only hurts future sums — reset it to the current element.

### DP Relation
`currentSum = max(nums[i], currentSum + nums[i])`

- If `currentSum + nums[i] > nums[i]` → extend the subarray.
- Otherwise → start fresh at `nums[i]`.

Track the global maximum throughout.

### Why Not Divide and Conquer?
Correct but O(n log n) — Kadane's is simpler and O(n).

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Data Flow
```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

i=0: num=-2 → curr=max(-2, 0+(-2))=-2, maxSum=-2
i=1: num=1  → curr=max(1, -2+1)=1,    maxSum=1
i=2: num=-3 → curr=max(-3, 1-3)=-2,   maxSum=1
i=3: num=4  → curr=max(4, -2+4)=4,    maxSum=4
i=4: num=-1 → curr=max(-1, 4-1)=3,    maxSum=4
i=5: num=2  → curr=max(2, 3+2)=5,     maxSum=5
i=6: num=1  → curr=max(1, 5+1)=6,     maxSum=6  ← winner
i=7: num=-5 → curr=max(-5, 6-5)=1,    maxSum=6
i=8: num=4  → curr=max(4, 1+4)=5,     maxSum=6

Result: 6
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class MaximumSubarray {

    /**
     * Finds the maximum sum contiguous subarray using Kadane's Algorithm.
     * At each step, extend the current subarray or start fresh — whichever is larger.
     *
     * @param nums input array (may contain negatives)
     * @return maximum subarray sum
     */
    public int maxSubArray(int[] nums) {
        int currentSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.length; i++) {
            // Either extend the current subarray or start fresh from nums[i]
            currentSum = Math.max(nums[i], currentSum + nums[i]);
            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {
        MaximumSubarray solution = new MaximumSubarray();

        System.out.println(solution.maxSubArray(new int[]{-2,1,-3,4,-1,2,1,-5,4})); // 6
        System.out.println(solution.maxSubArray(new int[]{1}));                       // 1
        System.out.println(solution.maxSubArray(new int[]{5,4,-1,7,8}));              // 23
        System.out.println(solution.maxSubArray(new int[]{-3,-1,-2}));               // -1
    }
}
```

### Output
```
6
1
23
-1
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def max_sub_array(nums: list[int]) -> int:
    """
    Find the maximum sum contiguous subarray (Kadane's Algorithm).

    At each index: extend current subarray or start fresh.
    current_sum = max(nums[i], current_sum + nums[i])

    Time:  O(n)
    Space: O(1)
    """
    current_sum = max_sum = nums[0]

    for num in nums[1:]:
        current_sum = max(num, current_sum + num)
        max_sum = max(max_sum, current_sum)

    return max_sum


if __name__ == "__main__":
    print(max_sub_array([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # 6
    print(max_sub_array([1]))                                 # 1
    print(max_sub_array([5, 4, -1, 7, 8]))                   # 23
    print(max_sub_array([-3, -1, -2]))                       # -1
```

### Output
```
6
1
23
-1
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: What if all elements are negative?**  
A: Kadane's handles this — `maxSum` starts at `nums[0]` and gets updated to the least negative element.

**Q: How would you also return the subarray indices (not just the sum)?**  
A: Track `start`, `end`, and `tempStart`. When you start fresh, update `tempStart = i`. When you update `maxSum`, update `start = tempStart` and `end = i`.

**Q: Is this a dynamic programming problem?**  
A: Yes. `dp[i] = max subarray sum ending at index i`. The recurrence is `dp[i] = max(nums[i], dp[i-1] + nums[i])`. Kadane's is the space-optimized DP where we keep only `dp[i-1]`.

**Q: Can this be solved with divide and conquer?**  
A: Yes. Split in half; max subarray is either in left half, right half, or crosses midpoint. O(n log n) time.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Maximum sum contiguous subarray |
| **Optimal Algorithm** | Kadane's Algorithm |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) |
| **Key Insight** | Reset running sum when it goes negative — negative prefix only hurts |

### Core Pattern
> Greedy local decision: at each step, is the current element alone better than extending the previous subarray?

</details>
