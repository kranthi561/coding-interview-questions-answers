# Two Sum

---

**1. What is meant by Two Sum?**

## What is Two Sum?

**Two Sum** is a foundational array problem where you are given:

- An integer array `nums`
- A target integer `target`

**Goal:** Return the **indices** of the two numbers that add up to the target.

**Constraints:**

- Exactly one valid solution exists.
- You may not use the same element twice.
- Return the answer in any order.

**Example:**

```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
Explanation: nums[0] + nums[1] = 2 + 7 = 9
```



---

**2. Clarify Requirements**

## Requirements

### Functional Requirements

- Given an array of integers and a target sum, find two numbers that add up to the target.
- Return the **indices** (not the values) of the two numbers.
- Each input has exactly **one valid solution**.
- The same element cannot be used twice.

### Non-Functional Requirements


| Concern         | Expectation                                           |
| --------------- | ----------------------------------------------------- |
| **Scale**       | Array can have up to 10⁴ elements                     |
| **Latency**     | Should run in sub-millisecond for typical input sizes |
| **Correctness** | Must return exact indices, not values                 |
| **Uniqueness**  | Only one solution guaranteed                          |


### Clarifying Questions to Ask

- Can there be negative numbers? → Yes
- Is the array sorted? → Not necessarily
- Can the array have duplicates? → Yes, but only one valid pair
- Must we return indices or values? → Indices



---

**3. Time and Space Complexity**

## Complexity Analysis

### Approach 1: Brute Force (Nested Loop)


|           | Complexity               |
| --------- | ------------------------ |
| **Time**  | O(n²) — check every pair |
| **Space** | O(1) — no extra storage  |


### Approach 2: Hash Map (Optimal)


|           | Complexity                              |
| --------- | --------------------------------------- |
| **Time**  | O(n) — single pass through array        |
| **Space** | O(n) — hash map stores up to n elements |


**Why O(n) time?** We iterate once. For each element, a hash map lookup is O(1) average case.

**Trade-off:** We spend O(n) space to save O(n) time — a classic space-time trade-off.



---

**4. Algorithm Choice and Why**

## Algorithm: Hash Map (One-Pass)

### Why Hash Map?


| Option                    | Time       | Space    | Chosen?                     |
| ------------------------- | ---------- | -------- | --------------------------- |
| Brute Force (nested loop) | O(n²)      | O(1)     | No                          |
| Sorting + Two Pointers    | O(n log n) | O(n)     | No — loses original indices |
| **Hash Map (one-pass)**   | **O(n)**   | **O(n)** | **Yes**                     |


### Why NOT Two Pointers?

Sorting rearranges elements, which destroys the original indices needed for the answer.

### Hash Map Logic

1. For each element `nums[i]`, compute `complement = target - nums[i]`
2. Check if `complement` exists in the hash map
3. If yes → return `[map[complement], i]`
4. If no → store `nums[i] → i` in the map and continue

This is "look back as you go forward" — you check if the needed partner has already been seen.



---

**5. High-Level Design**

## High-Level Design

### Components

1. **Input Layer** — Receives `nums[]` and `target`
2. **Hash Map Store** — Maps `value → index` for O(1) lookup
3. **Complement Checker** — For each element, checks if its pair is already stored
4. **Output Layer** — Returns the two indices

### Data Flow Diagram

```
nums = [2, 7, 11, 15], target = 9

Step 1: i=0, num=2
        complement = 9 - 2 = 7
        map = {}  → 7 not found
        store: map = {2: 0}

Step 2: i=1, num=7
        complement = 9 - 7 = 2
        map = {2: 0}  → 2 found at index 0!
        return [0, 1]
```

### Visual

```
Index:  [0]  [1]  [2]  [3]
Array:   2    7   11   15
         ↑         
         stored    ↑
                   found complement → MATCH!
```



---

**6. Java Solution with Explanation**

## Java Implementation

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSum {

    /**
     * Finds two indices whose values sum to target using a hash map.
     * One-pass: store each value as we go, check if complement was seen before.
     *
     * @param nums   input integer array
     * @param target the desired sum
     * @return int[] with two indices [i, j] where nums[i] + nums[j] == target
     */
    public int[] twoSum(int[] nums, int target) {
        // Map: value -> index
        Map<Integer, Integer> seen = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            // Check if complement was already encountered
            if (seen.containsKey(complement)) {
                return new int[]{seen.get(complement), i};
            }

            // Store current value with its index
            seen.put(nums[i], i);
        }

        // Problem guarantees one solution, this line is never reached
        throw new IllegalArgumentException("No valid pair found");
    }

    // ---- Main: Example usage ----
    public static void main(String[] args) {
        TwoSum solution = new TwoSum();

        // Example 1
        int[] nums1 = {2, 7, 11, 15};
        int target1 = 9;
        int[] result1 = solution.twoSum(nums1, target1);
        System.out.println("Input: nums=[2,7,11,15], target=9");
        System.out.println("Output: [" + result1[0] + ", " + result1[1] + "]"); // [0, 1]

        // Example 2
        int[] nums2 = {3, 2, 4};
        int target2 = 6;
        int[] result2 = solution.twoSum(nums2, target2);
        System.out.println("\nInput: nums=[3,2,4], target=6");
        System.out.println("Output: [" + result2[0] + ", " + result2[1] + "]"); // [1, 2]

        // Example 3
        int[] nums3 = {3, 3};
        int target3 = 6;
        int[] result3 = solution.twoSum(nums3, target3);
        System.out.println("\nInput: nums=[3,3], target=6");
        System.out.println("Output: [" + result3[0] + ", " + result3[1] + "]"); // [0, 1]
    }
}
```

### Output

```
Input: nums=[2,7,11,15], target=9
Output: [0, 1]

Input: nums=[3,2,4], target=6
Output: [1, 2]

Input: nums=[3,3], target=6
Output: [0, 1]
```



---

**7. Python Solution with Explanation**

## Python Implementation

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    """
    Find two indices whose values sum to target.

    Strategy: One-pass hash map.
    For each number, check if its complement (target - num) was seen before.
    Store value -> index as we iterate.

    Args:
        nums: List of integers
        target: Target sum

    Returns:
        List of two indices [i, j] where nums[i] + nums[j] == target

    Time:  O(n)
    Space: O(n)
    """
    seen = {}  # value -> index

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    raise ValueError("No valid pair found")


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: Standard case
    nums1, target1 = [2, 7, 11, 15], 9
    print(f"Input: nums={nums1}, target={target1}")
    print(f"Output: {two_sum(nums1, target1)}")  # [0, 1]

    # Example 2: Answer not at start
    nums2, target2 = [3, 2, 4], 6
    print(f"\nInput: nums={nums2}, target={target2}")
    print(f"Output: {two_sum(nums2, target2)}")  # [1, 2]

    # Example 3: Duplicate values
    nums3, target3 = [3, 3], 6
    print(f"\nInput: nums={nums3}, target={target3}")
    print(f"Output: {two_sum(nums3, target3)}")  # [0, 1]
```

### Output

```
Input: nums=[2, 7, 11, 15], target=9
Output: [0, 1]

Input: nums=[3, 2, 4], target=6
Output: [1, 2]

Input: nums=[3, 3], target=6
Output: [0, 1]
```



---

**8. Interview Q&A**

## Q&A

**Q: Why use a hash map instead of brute force?**  
A: Brute force is O(n²) because it checks every pair. A hash map reduces lookup to O(1), making the overall solution O(n).

**Q: Why not sort and use two pointers?**  
A: Sorting is O(n log n) and destroys original indices. Since we need to return indices, sorting would require tracking the original positions separately.

**Q: What if there are duplicate values in the array?**  
A: The hash map stores `value → index`. Since the problem guarantees exactly one valid pair, duplicates won't conflict with the answer. We also check for the complement before storing, so same-index reuse is avoided.

**Q: What if the array is empty or has one element?**  
A: The problem guarantees a valid solution exists, implying at least 2 elements. But defensively, the loop returns nothing and the guard clause throws.

**Q: Can you solve it in O(1) space?**  
A: Only with brute force (O(n²) time). There's an inherent trade-off: reducing time below O(n²) requires extra space.

**Q: What is the worst-case for the hash map?**  
A: Hash collisions can degrade lookup to O(n), but Java's `HashMap` and Python's `dict` handle this with rehashing, keeping average O(1).



---

**9. Summary**

## Summary


| Attribute             | Value                                                      |
| --------------------- | ---------------------------------------------------------- |
| **Problem**           | Find two indices that sum to target                        |
| **Optimal Algorithm** | One-pass Hash Map                                          |
| **Time Complexity**   | O(n)                                                       |
| **Space Complexity**  | O(n)                                                       |
| **Key Insight**       | For each element, check if its complement was already seen |


### Core Pattern

> Store elements as you iterate, check for the needed partner on each step — "look back as you go forward."

### When to Use This Pattern

- Any "find two elements that satisfy a condition" problem
- When index preservation matters (can't sort)
- When O(n) time is required

