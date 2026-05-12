# Product of Array Except Self

---

1. What is meant by Product of Array Except Self?

## What is Product of Array Except Self?

Given an integer array `nums`, return an array `answer` such that `answer[i]` equals the **product of all elements of `nums` except `nums[i]`**.

**Constraints:**
- Must run in **O(n)** time.
- Cannot use the **division** operation.

**Example:**
```
Input:  nums = [1, 2, 3, 4]
Output: [24, 12, 8, 6]
Explanation:
  answer[0] = 2 * 3 * 4 = 24
  answer[1] = 1 * 3 * 4 = 12
  answer[2] = 1 * 2 * 4 = 8
  answer[3] = 1 * 2 * 3 = 6
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements
- Return array where each position holds the product of all other elements.
- **No division** allowed.
- Must be O(n) time.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 10⁵ elements |
| **Time** | Strictly O(n) |
| **Space** | O(n) for output; O(1) extra if output array doesn't count |

### Clarifying Questions
- Can the array contain zeros? → Yes
- Can it contain negative numbers? → Yes
- Minimum length? → At least 2 elements


---

3. Time and Space Complexity

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force (nested loop) | O(n²) | O(1) |
| Division by total product | O(n) | O(1) | (not allowed + fails with zeros) |
| **Prefix + Suffix Products** | **O(n)** | **O(n)** |
| Optimized (single output array) | O(n) | O(1) extra |


---

4. Algorithm Choice and Why

## Algorithm: Prefix and Suffix Products

### Key Insight
`answer[i] = product of all elements to LEFT of i × product of all elements to RIGHT of i`

### Why Not Division?
- Fails when zeros are present.
- Explicitly prohibited by the problem.

### Two-Pass Approach
1. **Left pass:** Build prefix products where `prefix[i]` = product of `nums[0..i-1]`
2. **Right pass:** Multiply each position by the suffix product (tracked as a running variable)

```
nums     = [1,  2,  3,  4]
prefix   = [1,  1,  2,  6]   (product of everything left of index i)
suffix   = [24, 12, 4,  1]   (product of everything right of index i)
answer   = [24, 12, 8,  6]   (prefix[i] * suffix[i])
```


---

5. High-Level Design

## High-Level Design

### Components
1. **Prefix pass** — left-to-right, accumulate running product into output array
2. **Suffix pass** — right-to-left, multiply output array by running right product

### Data Flow
```
nums = [1, 2, 3, 4]

Pass 1 (left → right, prefix):
  output = [1, 1, 2, 6]
  (output[i] = product of nums[0..i-1])

Pass 2 (right → left, suffix):
  rightProduct starts at 1
  i=3: output[3] = 6 * 1 = 6,  rightProduct = 4
  i=2: output[2] = 2 * 4 = 8,  rightProduct = 12
  i=1: output[1] = 1 * 12 = 12, rightProduct = 24
  i=0: output[0] = 1 * 24 = 24, rightProduct = 24

Final: [24, 12, 8, 6]
```


---

6. Java Solution with Explanation

## Java Implementation

```java
public class ProductOfArrayExceptSelf {

    /**
     * Returns array where each index holds product of all other elements.
     * Uses two passes: left prefix products, then right suffix products.
     * No division used. O(n) time, O(1) extra space.
     *
     * @param nums input array
     * @return result array where result[i] = product of all nums except nums[i]
     */
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] output = new int[n];

        // Pass 1: Fill output[i] with product of all elements to the LEFT of i
        output[0] = 1;
        for (int i = 1; i < n; i++) {
            output[i] = output[i - 1] * nums[i - 1];
        }

        // Pass 2: Multiply each output[i] by the product of all elements to the RIGHT
        int rightProduct = 1;
        for (int i = n - 1; i >= 0; i--) {
            output[i] *= rightProduct;
            rightProduct *= nums[i]; // accumulate right product as we go left
        }

        return output;
    }

    public static void main(String[] args) {
        ProductOfArrayExceptSelf solution = new ProductOfArrayExceptSelf();

        // Example 1
        int[] nums1 = {1, 2, 3, 4};
        int[] result1 = solution.productExceptSelf(nums1);
        System.out.print("Input: [1,2,3,4] → Output: ");
        for (int v : result1) System.out.print(v + " "); // 24 12 8 6
        System.out.println();

        // Example 2: With zero
        int[] nums2 = {-1, 1, 0, -3, 3};
        int[] result2 = solution.productExceptSelf(nums2);
        System.out.print("Input: [-1,1,0,-3,3] → Output: ");
        for (int v : result2) System.out.print(v + " "); // 0 0 9 0 0
        System.out.println();
    }
}
```

### Output
```
Input: [1,2,3,4] → Output: 24 12 8 6
Input: [-1,1,0,-3,3] → Output: 0 0 9 0 0
```


---

7. Python Solution with Explanation

## Python Implementation

```python
def product_except_self(nums: list[int]) -> list[int]:
    """
    Return array where output[i] = product of all elements except nums[i].

    Strategy: Two-pass prefix/suffix product.
    Pass 1 (L→R): output[i] = product of all elements left of i.
    Pass 2 (R→L): multiply output[i] by product of all elements right of i.

    Time:  O(n)
    Space: O(1) extra (output array doesn't count)
    """
    n = len(nums)
    output = [1] * n

    # Pass 1: prefix products
    prefix = 1
    for i in range(n):
        output[i] = prefix
        prefix *= nums[i]

    # Pass 2: multiply by suffix products
    suffix = 1
    for i in range(n - 1, -1, -1):
        output[i] *= suffix
        suffix *= nums[i]

    return output


if __name__ == "__main__":
    print(product_except_self([1, 2, 3, 4]))       # [24, 12, 8, 6]
    print(product_except_self([-1, 1, 0, -3, 3]))  # [0, 0, 9, 0, 0]
```

### Output
```
[24, 12, 8, 6]
[0, 0, 9, 0, 0]
```


---

8. Interview Q&A

## Q&A

**Q: Why can't we use total product divided by nums[i]?**  
A: Division fails when `nums[i] = 0`. Also, the problem explicitly forbids it.

**Q: How does the algorithm handle zeros?**  
A: Naturally — prefix and suffix products propagate zeros correctly. If there's one zero, only that position gets a non-zero answer.

**Q: How is space O(1) extra if we use an output array?**  
A: By convention, the output array doesn't count toward extra space. The only additional variable is `rightProduct`.

**Q: What if two zeros exist?**  
A: All products become 0, since every product includes at least one zero.


---

9. Summary

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Product of all elements except current index |
| **Optimal Algorithm** | Two-pass prefix × suffix products |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) extra |
| **Key Insight** | `answer[i] = left_product[i] × right_product[i]` |

