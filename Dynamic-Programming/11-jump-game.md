# Jump Game

---

<details>
<summary><strong>1. What is meant by Jump Game?</strong></summary>

## What is Jump Game?

**Jump Game** is a greedy/DP problem where:

- You are given an integer array `nums`.
- You start at index 0.
- `nums[i]` represents the **maximum** number of steps you can jump forward from index `i`.

**Goal:** Return `true` if you can reach the **last index**, `false` otherwise.

**Constraints:**
- `1 <= nums.length <= 10⁴`
- `0 <= nums[i] <= 10⁵`

**Example:**

```
Input:  nums = [2, 3, 1, 1, 4]
Output: true
Explanation: Jump 1 step from 0 to 1, then 3 steps to the last index.

Input:  nums = [3, 2, 1, 0, 4]
Output: false
Explanation: Always stuck at index 3 — no way to reach index 4.
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Determine if the last index is reachable from index 0.
- At each index, you can jump any number of steps from 0 to `nums[i]`.
- You don't need to find the actual path — just a boolean answer.

### Non-Functional Requirements

| Concern         | Expectation                                       |
| --------------- | ------------------------------------------------- |
| **Scale**       | n up to 10⁴ — O(n) easily handles this           |
| **Latency**     | O(n) — single pass                                |
| **Correctness** | Must handle zeros mid-array that block all paths  |
| **Edge Cases**  | Single element (already at end), all zeros        |

### Clarifying Questions to Ask

- Can we jump exactly or at most `nums[i]` steps? → At most (can jump fewer).
- Must we use every element? → No, we can skip elements.
- What if n=1? → Already at last index, return true.
- Can values be 0? → Yes, zeros block forward progress from that position.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force / DFS

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — try all possible jump sequences |
| **Space** | O(n) — recursion stack |

### Approach 2: Dynamic Programming (Bottom-Up)

| | Complexity |
| --- | --- |
| **Time** | O(n²) — for each position, check all prior positions |
| **Space** | O(n) — dp boolean array |

### Approach 3: Greedy (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single left-to-right pass |
| **Space** | O(1) — only one variable (`maxReach`) |

**Why O(n)?** We scan once from left to right, updating the farthest reachable index at each step.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Greedy (Farthest Reach)

### Why Greedy?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2ⁿ) | O(n) | No |
| DP | O(n²) | O(n) | No — overkill |
| **Greedy** | **O(n)** | **O(1)** | **Yes — optimal** |

### Key Insight

Track the **farthest index reachable** at any point in the scan.

```
maxReach = 0

For each index i from 0 to n-1:
    if i > maxReach: return false   // i is beyond what we can reach
    maxReach = max(maxReach, i + nums[i])

return true
```

**Why it works:** At each index `i`, the farthest we can reach from there is `i + nums[i]`. If `maxReach >= n-1` at any point, we can reach the end. If we encounter an index `i > maxReach`, we're stranded.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Input Layer** — Receives `nums[]`
2. **Reach Tracker** — Single variable `maxReach` updated each step
3. **Stranded Detector** — Checks if current index exceeds `maxReach`
4. **Output Layer** — Returns `true` if scan completes without getting stranded

### Data Flow Diagram

```
nums = [2, 3, 1, 1, 4]  (can reach end)

i=0: maxReach = max(0, 0+2) = 2   — can reach up to index 2
i=1: 1 <= 2 ✓, maxReach = max(2, 1+3) = 4
i=2: 2 <= 4 ✓, maxReach = max(4, 2+1) = 4
i=3: 3 <= 4 ✓, maxReach = max(4, 3+1) = 4
i=4: 4 <= 4 ✓, maxReach = max(4, 4+4) = 8
Output: true ✓

---

nums = [3, 2, 1, 0, 4]  (blocked)

i=0: maxReach = max(0, 0+3) = 3
i=1: maxReach = max(3, 1+2) = 3
i=2: maxReach = max(3, 2+1) = 3
i=3: maxReach = max(3, 3+0) = 3
i=4: 4 > 3 → STRANDED  → return false ✗
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class JumpGame {

    /**
     * Determine if the last index is reachable from index 0.
     *
     * Greedy: track the farthest index reachable as we scan left to right.
     * If the current index exceeds maxReach, we are stranded → return false.
     * If we scan through the entire array, → return true.
     *
     * @param nums max jump lengths at each index
     * @return true if last index is reachable
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public boolean canJump(int[] nums) {
        int maxReach = 0; // farthest index we can reach so far

        for (int i = 0; i < nums.length; i++) {
            if (i > maxReach) return false; // stranded — can't reach index i
            maxReach = Math.max(maxReach, i + nums[i]); // extend reach from here
        }

        return true; // successfully scanned all reachable positions
    }

    // DP version for comparison (O(n^2))
    public boolean canJump_DP(int[] nums) {
        int n = nums.length;
        boolean[] dp = new boolean[n];
        dp[0] = true; // always start at index 0

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && j + nums[j] >= i) {
                    dp[i] = true;
                    break;
                }
            }
        }

        return dp[n - 1];
    }

    public static void main(String[] args) {
        JumpGame sol = new JumpGame();

        // Example 1: Can reach end
        System.out.println("Input: [2,3,1,1,4]");
        System.out.println("Output: " + sol.canJump(new int[]{2, 3, 1, 1, 4})); // true

        // Example 2: Blocked by zero
        System.out.println("\nInput: [3,2,1,0,4]");
        System.out.println("Output: " + sol.canJump(new int[]{3, 2, 1, 0, 4})); // false

        // Example 3: Single element — already at end
        System.out.println("\nInput: [0]");
        System.out.println("Output: " + sol.canJump(new int[]{0})); // true

        // Example 4: Starts at zero — can't move
        System.out.println("\nInput: [0, 1]");
        System.out.println("Output: " + sol.canJump(new int[]{0, 1})); // false

        // Example 5: Large jump at start
        System.out.println("\nInput: [5, 0, 0, 0, 0]");
        System.out.println("Output: " + sol.canJump(new int[]{5, 0, 0, 0, 0})); // true
    }
}
```

### Output

```
Input: [2,3,1,1,4]
Output: true

Input: [3,2,1,0,4]
Output: false

Input: [0]
Output: true

Input: [0, 1]
Output: false

Input: [5, 0, 0, 0, 0]
Output: true
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def can_jump(nums: list[int]) -> bool:
    """
    Determine if the last index is reachable from index 0.

    Strategy: Greedy — track farthest reachable index.
    At each position i, update maxReach = max(maxReach, i + nums[i]).
    If i ever exceeds maxReach, we are stranded → return False.

    Args:
        nums: Max jump lengths at each index

    Returns:
        True if last index is reachable, False otherwise

    Time:  O(n)
    Space: O(1)
    """
    max_reach = 0

    for i, jump in enumerate(nums):
        if i > max_reach:
            return False  # current position is unreachable
        max_reach = max(max_reach, i + jump)

    return True  # scanned all reachable positions successfully


if __name__ == "__main__":
    test_cases = [
        ([2, 3, 1, 1, 4], True),
        ([3, 2, 1, 0, 4], False),
        ([0],             True),
        ([0, 1],          False),
        ([5, 0, 0, 0, 0], True),
        ([1, 0, 1, 0],    False),
    ]

    for nums, expected in test_cases:
        result = can_jump(nums)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] nums={nums} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] nums=[2, 3, 1, 1, 4] → Output: True, Expected: True
[PASS] nums=[3, 2, 1, 0, 4] → Output: False, Expected: False
[PASS] nums=[0] → Output: True, Expected: True
[PASS] nums=[0, 1] → Output: False, Expected: False
[PASS] nums=[5, 0, 0, 0, 0] → Output: True, Expected: True
[PASS] nums=[1, 0, 1, 0] → Output: False, Expected: False
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why is the greedy approach correct here?**
A: We're asking a reachability question — can we get to the end? The greedy insight is: if we can reach index `i`, we can try jumping from `i`. Tracking the farthest reachable index covers all possible jump combinations at once without needing to enumerate them.

**Q: What if all values are 0 except the first one?**
A: If `nums[0] > 0`, we can reach index 1 at minimum. Whether we reach the end depends on subsequent values. If `nums[0] = 0` and n > 1, we return false immediately.

**Q: How does Jump Game II differ?**
A: Jump Game II asks for the **minimum number of jumps** to reach the last index (assumes it's always reachable). Uses a greedy approach tracking the current jump's coverage range and counting jumps when the range is exhausted.

**Q: Why does `i > maxReach` detect being stranded?**
A: `maxReach` is the farthest index reachable from any previously visited position. If the current index `i` is greater, no jump from any prior position can reach `i` — we're stranded.

**Q: Can nums contain very large jump values?**
A: Yes, but they don't affect correctness — `maxReach = i + nums[i]` may exceed `n-1`, but we cap the scan at `n-1`. We could add `Math.min(maxReach, n-1)` for clarity, but it's not required.

**Q: What is the DP recurrence for this problem?**
A: `dp[i] = true` if any `j < i` has `dp[j] = true` and `j + nums[j] >= i`. This is O(n²) and less efficient than greedy, but follows the same logic.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Problem**           | Determine if the last index is reachable (max-jump per position)  |
| **Optimal Algorithm** | Greedy — track farthest reachable index in a single pass          |
| **Time Complexity**   | O(n)                                                              |
| **Space Complexity**  | O(1)                                                              |
| **Key Insight**       | If `i > maxReach` at any point, we're stranded → return false    |

### Core Pattern

> Track the furthest position reachable so far. At each step, update it. If you encounter a position beyond what's reachable, you're stuck. If you finish the scan, you can reach the end.

### When to Use This Pattern

- Reachability / coverage problems on a linear array.
- Related: Jump Game II (min jumps), Video Stitching, Interval Coverage, Boat Passengers.

</details>
