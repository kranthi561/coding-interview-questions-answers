# Maximum Product Subarray

---

<details>
<summary><strong>1. What is meant by Maximum Product Subarray?</strong></summary>

## What is Maximum Product Subarray?

Given an integer array `nums`, find the contiguous subarray that has the **largest product**, and return that product.

**Example:**
```
Input:  nums = [2, 3, -2, 4]
Output: 6
Explanation: Subarray [2, 3] has the largest product = 6.
```

```
Input:  nums = [-2, 0, -1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not contiguous.
```

**Why is this harder than Maximum Subarray (sum)?**  
Negative × negative = positive. A very negative number can become the maximum if another negative appears later.

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Find the contiguous subarray with the maximum product.
- Return the product value (not the subarray).
- Subarray must have at least one element.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 2×10⁴ elements |
| **Values** | Can be negative (key challenge) |
| **Zeros** | Act as reset points |

### Clarifying Questions
- Can the array contain zeros? → Yes; zeros reset the product.
- Can all values be negative? → Yes; must handle even count of negatives.
- Return value or indices? → Just the product value.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force | O(n²) | O(1) |
| **Track max & min** | **O(n)** | **O(1)** |

**Why track both max and min?**  
A large negative (current min) multiplied by the next negative number can become the new maximum.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Track Current Max and Min

### Key Insight
Unlike sum, negative values flip the sign of the product. So we must track **both the max and min** products ending at the current index:
- `maxProd = max(num, maxProd * num, minProd * num)`
- `minProd = min(num, maxProd * num, minProd * num)`

**Why min?** If `num` is negative and `minProd` is very negative, `minProd * num` becomes a large positive — a new maximum.

### Zero Handling
When `num = 0`, both max and min reset to 0. Starting fresh at `num` handles this since `max(0, 0, 0) = 0` and a fresh subarray from next element begins.

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Data Flow
```
nums = [2, 3, -2, 4]

i=0: num=2  → maxP=2,   minP=2,   result=2
i=1: num=3  → maxP=6,   minP=3,   result=6
i=2: num=-2 → maxP=-2,  minP=-12, result=6  (max flips to negative)
i=3: num=4  → maxP=-8,  minP=-48, result=6  (4*(-2) still negative)

Result: 6

---

nums = [-2, 3, -4]

i=0: num=-2  → maxP=-2, minP=-2,  result=-2
i=1: num=3   → maxP=3,  minP=-6,  result=3
i=2: num=-4  → maxP=24, minP=-12, result=24
              (minP*num = (-6)*(-4) = 24 → new max!)
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class MaximumProductSubarray {

    /**
     * Finds the contiguous subarray with the maximum product.
     * Tracks both the maximum and minimum product at each step,
     * because a negative min * negative num = large positive.
     *
     * @param nums input array
     * @return maximum product of any contiguous subarray
     */
    public int maxProduct(int[] nums) {
        int maxProd = nums[0];
        int minProd = nums[0];
        int result = nums[0];

        for (int i = 1; i < nums.length; i++) {
            int num = nums[i];

            // When multiplied by a negative, max and min swap — compute all candidates
            int tempMax = Math.max(num, Math.max(maxProd * num, minProd * num));
            int tempMin = Math.min(num, Math.min(maxProd * num, minProd * num));

            maxProd = tempMax;
            minProd = tempMin;

            result = Math.max(result, maxProd);
        }

        return result;
    }

    public static void main(String[] args) {
        MaximumProductSubarray solution = new MaximumProductSubarray();

        System.out.println(solution.maxProduct(new int[]{2, 3, -2, 4}));   // 6
        System.out.println(solution.maxProduct(new int[]{-2, 0, -1}));     // 0
        System.out.println(solution.maxProduct(new int[]{-2, 3, -4}));     // 24
        System.out.println(solution.maxProduct(new int[]{-2}));            // -2
    }
}
```

### Output
```
6
0
24
-2
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def max_product(nums: list[int]) -> int:
    """
    Find maximum product contiguous subarray.

    Track both max and min products at each step.
    Negative numbers can flip max→min and min→max.

    Time:  O(n)
    Space: O(1)
    """
    max_prod = min_prod = result = nums[0]

    for num in nums[1:]:
        # Must compute both before updating (avoid using updated values)
        candidates = (num, max_prod * num, min_prod * num)
        max_prod = max(candidates)
        min_prod = min(candidates)
        result = max(result, max_prod)

    return result


if __name__ == "__main__":
    print(max_product([2, 3, -2, 4]))   # 6
    print(max_product([-2, 0, -1]))     # 0
    print(max_product([-2, 3, -4]))     # 24
    print(max_product([-2]))            # -2
```

### Output
```
6
0
24
-2
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why do we track minimum product?**  
A: A large negative minimum, when multiplied by a negative number, produces a large positive maximum.

**Q: How does zero affect the algorithm?**  
A: Zero resets the subarray. `max(0, prev_max*0, prev_min*0) = 0`. The algorithm naturally starts fresh from the next element.

**Q: How is this different from Maximum Subarray Sum?**  
A: Sum is monotone — adding a negative always decreases. Product is not monotone — a negative can flip the sign and create a new max.

**Q: What if all elements are negative?**  
A: The answer is the product of all negatives if count is even, or drop the first or last negative if count is odd.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Maximum product contiguous subarray |
| **Optimal Algorithm** | Track current max and min products |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) |
| **Key Insight** | Negatives flip max↔min — track both to handle sign changes |

</details>
