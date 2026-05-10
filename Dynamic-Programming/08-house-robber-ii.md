# House Robber II

---

<details>
<summary><strong>1. What is meant by House Robber II?</strong></summary>

## What is House Robber II?

**House Robber II** extends the original House Robber problem:

- Houses are arranged in a **circle** (the last house is adjacent to the first).
- You cannot rob two **adjacent** houses (alarm triggers).
- All other rules from House Robber I apply.

**Goal:** Return the **maximum amount of money** you can rob without triggering the alarm.

**Constraints:**
- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 1000`

**Example:**

```
Input:  nums = [2, 3, 2]
Output: 3
Explanation: Rob house 1 (3). Cannot rob both 0 and 2 (they're adjacent via circle).
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Houses form a **circular** arrangement — house 0 and house n-1 are neighbors.
- Cannot rob two adjacent houses.
- Maximize total money robbed.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | n up to 100 — O(n) trivially fast                   |
| **Latency**     | O(n) — two linear passes                            |
| **Correctness** | Must handle the circular adjacency constraint       |
| **Edge Cases**  | n=1 (rob the only house), n=2 (rob the larger one) |

### Clarifying Questions to Ask

- Is this strictly circular or a line? → Circular (first and last are neighbors).
- n=1 edge case? → Just return `nums[0]`.
- Can all values be 0? → Yes, return 0.
- Same adjacency rules as House Robber I? → Yes.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Brute Force (Try All Subsets)

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — exponential |
| **Space** | O(n) — recursion stack |

### Optimal: Two Linear Passes of House Robber I

| | Complexity |
| --- | --- |
| **Time** | O(n) — two O(n) passes |
| **Space** | O(1) — two-variable DP in each pass |

**Why O(n)?** The circular constraint is handled by running the linear House Robber twice on overlapping subarrays — each pass is O(n).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Two-Pass Linear DP

### Key Insight

The circular constraint means house 0 and house n-1 cannot **both** be robbed.

Therefore, at least one of them is always excluded. The optimal solution is:

```
max(
    rob(nums[0..n-2]),   // exclude last house
    rob(nums[1..n-1])    // exclude first house
)
```

Run the House Robber I algorithm on both subarrays and return the larger result.

### Why This Works

If the optimal solution includes house 0, it cannot include house n-1, so the answer is in `rob(nums[0..n-2])`.  
If the optimal solution includes house n-1, it cannot include house 0, so the answer is in `rob(nums[1..n-1])`.  
If neither is included, both runs would still find the correct subset.

| Option | Correct? | Time | Space | Chosen? |
| --- | --- | --- | --- | --- |
| Brute Force | Yes | O(2ⁿ) | O(n) | No |
| **Two-Pass Linear DP** | **Yes** | **O(n)** | **O(1)** | **Yes** |

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Edge Case Handler** — Return `nums[0]` if n=1
2. **Pass 1** — Run `robLinear` on `nums[0..n-2]` (exclude last)
3. **Pass 2** — Run `robLinear` on `nums[1..n-1]` (exclude first)
4. **Output Layer** — Return `max(pass1, pass2)`

### Data Flow Diagram

```
nums = [2, 3, 2]  (n=3)

Pass 1: rob(nums[0..1]) = rob([2, 3])
  prev=0, curr=0
  num=2: curr=max(0, 0+2)=2 → prev=0, curr=2
  num=3: curr=max(2, 0+3)=3 → prev=2, curr=3
  result1 = 3

Pass 2: rob(nums[1..2]) = rob([3, 2])
  prev=0, curr=0
  num=3: curr=max(0, 0+3)=3 → prev=0, curr=3
  num=2: curr=max(3, 0+2)=3 → prev=3, curr=3
  result2 = 3

Output: max(3, 3) = 3
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class HouseRobberII {

    /**
     * Max money from a circular arrangement of houses (no adjacent robbing).
     *
     * Strategy: Break the circle by running linear House Robber on two subarrays:
     *   1. nums[0..n-2] (exclude last house)
     *   2. nums[1..n-1] (exclude first house)
     * Return the maximum of both runs.
     *
     * @param nums array of house values (circular arrangement)
     * @return maximum money robbed
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0]; // only one house — rob it
        if (n == 2) return Math.max(nums[0], nums[1]); // two adjacent — rob the larger

        // Run linear robber on both ranges and return the max
        return Math.max(
            robLinear(nums, 0, n - 2), // exclude last house
            robLinear(nums, 1, n - 1)  // exclude first house
        );
    }

    /**
     * House Robber I on nums[start..end] (inclusive).
     * Two-variable DP: O(n) time, O(1) space.
     */
    private int robLinear(int[] nums, int start, int end) {
        int prev = 0, curr = 0;
        for (int i = start; i <= end; i++) {
            int next = Math.max(curr, prev + nums[i]);
            prev = curr;
            curr = next;
        }
        return curr;
    }

    public static void main(String[] args) {
        HouseRobberII sol = new HouseRobberII();

        // Example 1: Classic circular case
        System.out.println("Input: [2,3,2]");
        System.out.println("Output: " + sol.rob(new int[]{2, 3, 2})); // 3

        // Example 2
        System.out.println("\nInput: [1,2,3,1]");
        System.out.println("Output: " + sol.rob(new int[]{1, 2, 3, 1})); // 4

        // Example 3: Single house
        System.out.println("\nInput: [5]");
        System.out.println("Output: " + sol.rob(new int[]{5})); // 5

        // Example 4: Two houses — pick the larger
        System.out.println("\nInput: [3, 7]");
        System.out.println("Output: " + sol.rob(new int[]{3, 7})); // 7

        // Example 5: All equal
        System.out.println("\nInput: [4,4,4,4]");
        System.out.println("Output: " + sol.rob(new int[]{4, 4, 4, 4})); // 8
    }
}
```

### Output

```
Input: [2,3,2]
Output: 3

Input: [1,2,3,1]
Output: 4

Input: [5]
Output: 5

Input: [3, 7]
Output: 7

Input: [4,4,4,4]
Output: 8
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def rob(nums: list[int]) -> int:
    """
    Max money from robbing a circular row of houses (no adjacent robbing).

    Strategy: Break the circle into two linear problems:
      1. Rob houses 0 to n-2 (exclude last)
      2. Rob houses 1 to n-1 (exclude first)
    Return the max of both.

    Args:
        nums: List of house money values (circular arrangement)

    Returns:
        Maximum money robbed without triggering the alarm

    Time:  O(n)
    Space: O(1)
    """
    n = len(nums)

    if n == 1:
        return nums[0]

    def rob_linear(houses: list[int]) -> int:
        prev, curr = 0, 0
        for num in houses:
            prev, curr = curr, max(curr, prev + num)
        return curr

    return max(
        rob_linear(nums[:-1]),  # exclude last house
        rob_linear(nums[1:])    # exclude first house
    )


if __name__ == "__main__":
    test_cases = [
        ([2, 3, 2],    3),
        ([1, 2, 3, 1], 4),
        ([5],          5),
        ([3, 7],       7),
        ([4, 4, 4, 4], 8),
        ([1, 1, 1, 2], 3),
    ]

    for nums, expected in test_cases:
        result = rob(nums)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] nums={nums} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] nums=[2, 3, 2] → Output: 3, Expected: 3
[PASS] nums=[1, 2, 3, 1] → Output: 4, Expected: 4
[PASS] nums=[5] → Output: 5, Expected: 5
[PASS] nums=[3, 7] → Output: 7, Expected: 7
[PASS] nums=[4, 4, 4, 4] → Output: 8, Expected: 8
[PASS] nums=[1, 1, 1, 2] → Output: 3, Expected: 3
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why does splitting into two subarrays solve the circular problem?**
A: The circular adjacency means house 0 and house n-1 can't both be in the solution. So we run two independent linear passes — one where house 0 is allowed (but n-1 is excluded), and one where n-1 is allowed (but 0 is excluded). The optimal answer must fall in one of these two cases.

**Q: What if n=1?**
A: There's only one house — rob it (return `nums[0]`). With n=1, there's no adjacency constraint.

**Q: What if n=2?**
A: The two houses are adjacent (and also form a 2-house circle). Rob the larger one: `max(nums[0], nums[1])`.

**Q: Is it possible that the optimal solution avoids both house 0 and house n-1?**
A: Yes. In that case, both subarrays (`nums[0..n-2]` and `nums[1..n-1]`) will find the same optimal subset (since the excluded house wasn't chosen anyway), and the max of the two is still correct.

**Q: How does this extend to House Robber III (binary tree)?**
A: Each node has two children. DFS returns a pair: `(rob_this_node, skip_this_node)`. For each node: `rob = node.val + skip_left + skip_right`; `skip = max(rob_left, skip_left) + max(rob_right, skip_right)`.

**Q: Can you solve this in a single pass?**
A: Not directly without extra tracking, because the circular constraint depends on a global choice (include house 0 or not). Two passes is the standard clean approach.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Max money from non-adjacent houses in a circular arrangement       |
| **Optimal Algorithm** | Two-pass linear DP (run House Robber I on two subarrays)           |
| **Time Complexity**   | O(n)                                                               |
| **Space Complexity**  | O(1)                                                               |
| **Key Insight**       | Circular → house 0 and n-1 can't both be chosen → two linear cases |

### Core Pattern

> Break a circular constraint into two simpler linear problems by excluding one boundary element each time. Run the simpler algorithm on both and take the maximum.

### When to Use This Pattern

- Circular variants of linear DP problems.
- Related: House Robber I (linear), House Robber III (tree), Jump Game II, Circular Array Loop.

</details>
