# Missing Number

---

<details>
<summary><strong>1. What is meant by Missing Number?</strong></summary>

## What is Missing Number?

Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return the **one number in that range that is missing** from the array.

**Example:**
```
Input:  nums = [3, 0, 1]
Output: 2
Explanation: n=3, range is [0,1,2,3]. The number 2 is missing.

Input:  nums = [0, 1]
Output: 2
Explanation: n=2, range is [0,1,2]. The number 2 is missing.

Input:  nums = [9,6,4,2,3,5,7,0,1]
Output: 8
Explanation: n=9, range is [0..9]. The number 8 is missing.
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Array has `n` elements containing distinct values from `[0, n]`.
- Exactly **one** number is missing — find and return it.
- Values are non-negative integers.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | `n` up to 10⁴ |
| **Time** | O(n) |
| **Space** | O(1) preferred |

### Clarifying Questions to Ask
- Is exactly one number missing? → Yes, always.
- Can the array be unsorted? → Yes.
- Are all numbers distinct? → Yes.
- Can the missing number be 0 or n? → Yes, the endpoints are valid answers.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Sorting then scan | O(n log n) | O(1) | Modifies array |
| Hash set membership | O(n) | O(n) | Extra space |
| **Gauss formula (sum)** | **O(n)** | **O(1)** | Arithmetic |
| **XOR approach** | **O(n)** | **O(1)** | Bit manipulation |

Both the Gauss formula and XOR approaches achieve optimal O(n) time and O(1) space.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Gauss Formula (Primary) + XOR (Alternative)

### Approach 1: Gauss Sum Formula

The sum of integers from 0 to n is `n * (n + 1) / 2`.
The missing number = expected sum − actual sum.

```
expectedSum = n * (n + 1) / 2
actualSum   = sum of all elements in nums
missing     = expectedSum - actualSum
```

**Why this works:** Every number 0..n is accounted for except one. The difference between what should be there and what is there is the missing number.

### Approach 2: XOR

XOR every number 0..n AND every element in the array.
Numbers that appear in both cancel out (`x ^ x = 0`).
The missing number appears only once — it survives.

```
result = 0
XOR in each index i:   result ^= i
XOR in each value:     result ^= nums[i]
XOR in n itself:       result ^= n
remaining = missing number
```

### Why Gauss Is Preferred Here
- Simpler to read and explain.
- No risk of overflow confusion (though XOR has none either).
- Arithmetic intent is directly clear.

### Watch Out for Overflow
For large `n`, `n * (n + 1)` can overflow a 32-bit integer. In Java use `long`; in Python integers are arbitrary precision.

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Gauss Formula Flow
```
nums = [3, 0, 1]
n = 3

expectedSum = 3 * 4 / 2 = 6
actualSum   = 3 + 0 + 1 = 4
missing     = 6 - 4 = 2
```

### XOR Flow
```
nums = [3, 0, 1],  n = 3

Start: result = 0

XOR with indices and values simultaneously:
  i=0: result ^= 0 (index) ^= 3 (value)  → result = 3
  i=1: result ^= 1 (index) ^= 0 (value)  → result = 2
  i=2: result ^= 2 (index) ^= 1 (value)  → result = 1

XOR with n:
  result ^= 3                              → result = 2

Result: 2 ✓
```

### Why XOR Cancels Pairs
```
0 XOR 0 = 0
1 XOR 1 = 0
3 XOR 3 = 0
Only 2 (present in indices but not in nums) remains.
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class MissingNumber {

    /**
     * Finds the missing number using Gauss sum formula.
     * Expected sum of [0..n] minus actual array sum = missing number.
     * Uses long to avoid 32-bit overflow for large n.
     *
     * @param nums array of n distinct values from [0, n] with one missing
     * @return the missing number
     */
    public int missingNumber(int[] nums) {
        int n = nums.length;
        long expectedSum = (long) n * (n + 1) / 2;
        long actualSum = 0;

        for (int num : nums) {
            actualSum += num;
        }

        return (int) (expectedSum - actualSum);
    }

    /**
     * Alternative: XOR approach.
     * XOR all indices 0..n with all values; paired numbers cancel out.
     * The survivor is the missing number.
     */
    public int missingNumberXOR(int[] nums) {
        int result = nums.length; // start with n

        for (int i = 0; i < nums.length; i++) {
            result ^= i;        // XOR index
            result ^= nums[i];  // XOR value
        }

        return result;
    }

    public static void main(String[] args) {
        MissingNumber solution = new MissingNumber();

        // Example 1
        System.out.println("Input [3,0,1]:         " + solution.missingNumber(new int[]{3, 0, 1}));         // 2
        // Example 2
        System.out.println("Input [0,1]:            " + solution.missingNumber(new int[]{0, 1}));            // 2
        // Example 3
        System.out.println("Input [9,6,4,2,3,5,7,0,1]: " + solution.missingNumber(new int[]{9,6,4,2,3,5,7,0,1})); // 8

        System.out.println("\nXOR approach:");
        System.out.println("Input [3,0,1]:  " + solution.missingNumberXOR(new int[]{3, 0, 1})); // 2
    }
}
```

### Output
```
Input [3,0,1]:         2
Input [0,1]:            2
Input [9,6,4,2,3,5,7,0,1]: 8

XOR approach:
Input [3,0,1]:  2
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def missing_number(nums: list[int]) -> int:
    """
    Find the missing number in range [0, n] using the Gauss sum formula.

    Expected sum = n*(n+1)//2
    Missing = expected - actual

    Time:  O(n)
    Space: O(1)
    """
    n = len(nums)
    expected = n * (n + 1) // 2
    return expected - sum(nums)


def missing_number_xor(nums: list[int]) -> int:
    """
    XOR approach: XOR all indices 0..n with all values.
    Paired numbers cancel; only the missing number survives.

    Time:  O(n)
    Space: O(1)
    """
    result = len(nums)
    for i, num in enumerate(nums):
        result ^= i ^ num
    return result


if __name__ == "__main__":
    print(missing_number([3, 0, 1]))            # 2
    print(missing_number([0, 1]))               # 2
    print(missing_number([9, 6, 4, 2, 3, 5, 7, 0, 1]))  # 8

    print("\nXOR:")
    print(missing_number_xor([3, 0, 1]))        # 2
```

### Output
```
2
2
8

XOR:
2
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why does the Gauss formula work?**
A: The range [0, n] has a known closed-form sum: `n*(n+1)/2`. Since the array contains all numbers in that range except one, subtracting the actual sum gives the missing value.

**Q: Why does the XOR approach work?**
A: XOR is self-inverse (`x ^ x = 0`) and commutative. XOR-ing all numbers 0..n with all array values pairs every present number with itself, cancelling to 0. The missing number has no pair, so it survives.

**Q: Which approach is better — Gauss or XOR?**
A: Both are O(n) time and O(1) space. Gauss is more readable. XOR avoids any risk of integer overflow entirely (addition can overflow, XOR cannot).

**Q: What if multiple numbers were missing?**
A: The Gauss formula only works for a single missing number. For multiple missing numbers, you'd need the sum AND sum of squares (two equations, two unknowns) or a hash set approach.

**Q: Can the missing number be 0 or n?**
A: Yes. If `nums = [1, 2, 3]`, the missing number is 0. If `nums = [0, 1, 2]`, the missing number is 3. Both approaches handle this correctly.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Find the one missing number in range [0, n] |
| **Optimal Algorithm** | Gauss sum formula or XOR |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) |
| **Key Insight (Gauss)** | Expected sum − actual sum = missing number |
| **Key Insight (XOR)** | Every present number cancels with its pair; only the missing survives |

### Core Pattern
> When a set is "almost complete" with exactly one element missing, the Gauss formula or XOR cancellation are the go-to O(1)-space approaches.

</details>
