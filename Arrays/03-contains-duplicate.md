# Contains Duplicate

---

<details>
<summary><strong>1. What is meant by Contains Duplicate?</strong></summary>

## What is Contains Duplicate?

Given an integer array `nums`, return `true` if **any value appears at least twice**, and `false` if every element is distinct.

**Example:**
```
Input:  nums = [1, 2, 3, 1]
Output: true
Explanation: 1 appears at index 0 and index 3.
```

```
Input:  nums = [1, 2, 3, 4]
Output: false
Explanation: All elements are distinct.
```

```
Input:  nums = [1, 1, 1, 3, 3, 4, 3, 2, 4, 2]
Output: true
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Return `true` if any duplicate exists in the array.
- Return `false` if all elements are unique.
- Array can contain any integers (positive, negative, zero).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 10⁵ elements |
| **Latency** | O(n) expected |
| **Memory** | O(n) acceptable for hash set approach |

### Clarifying Questions to Ask
- Can the array be empty? → Return false (no duplicates in empty array)
- What integer range? → Any 32-bit integers
- Do we need to find which element duplicates? → No, just return true/false

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute Force (nested loop) | O(n²) | O(1) |
| Sorting | O(n log n) | O(1) or O(n) |
| **Hash Set (optimal)** | **O(n)** | **O(n)** |

**Hash Set:** Insert each element; if insertion fails (element already exists), return `true`.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Hash Set

### Why Hash Set?
- A set only stores unique elements.
- Attempting to add a duplicate is detected in O(1) average time.
- Single pass: O(n) total.

### Logic
```
Create an empty set
For each number in nums:
    If number is already in set → return true
    Add number to set
Return false
```

### Why Not Sorting?
Sorting changes the index structure and takes O(n log n). Hash set is faster and simpler.

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Data Flow
```
nums = [1, 2, 3, 1]

Step 1: num=1 → set={},    not in set → add → set={1}
Step 2: num=2 → set={1},   not in set → add → set={1,2}
Step 3: num=3 → set={1,2}, not in set → add → set={1,2,3}
Step 4: num=1 → set={1,2,3}, IN SET! → return true
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
import java.util.HashSet;
import java.util.Set;

public class ContainsDuplicate {

    /**
     * Returns true if any value appears at least twice in the array.
     * Uses a HashSet for O(1) average-case membership checks.
     *
     * @param nums input integer array
     * @return true if duplicate exists, false otherwise
     */
    public boolean containsDuplicate(int[] nums) {
        Set<Integer> seen = new HashSet<>();

        for (int num : nums) {
            // add() returns false if element already in set
            if (!seen.add(num)) {
                return true;
            }
        }

        return false;
    }

    public static void main(String[] args) {
        ContainsDuplicate solution = new ContainsDuplicate();

        System.out.println(solution.containsDuplicate(new int[]{1, 2, 3, 1})); // true
        System.out.println(solution.containsDuplicate(new int[]{1, 2, 3, 4})); // false
        System.out.println(solution.containsDuplicate(new int[]{1, 1, 1, 3, 3, 4, 3, 2, 4, 2})); // true
    }
}
```

### Output
```
true
false
true
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def contains_duplicate(nums: list[int]) -> bool:
    """
    Check if any value appears at least twice.

    Strategy: Use a set to track seen values.
    If current value already in set → duplicate found.

    Time:  O(n)
    Space: O(n)
    """
    seen = set()

    for num in nums:
        if num in seen:
            return True
        seen.add(num)

    return False


# Pythonic one-liner alternative:
def contains_duplicate_v2(nums: list[int]) -> bool:
    return len(nums) != len(set(nums))


if __name__ == "__main__":
    print(contains_duplicate([1, 2, 3, 1]))        # True
    print(contains_duplicate([1, 2, 3, 4]))        # False
    print(contains_duplicate_v2([1, 1, 1, 3, 3]))  # True
```

### Output
```
True
False
True
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: What is the space complexity of the one-liner `len(nums) != len(set(nums))`?**  
A: O(n) — creates a new set with all elements. Same as the explicit approach.

**Q: Can we solve it in O(1) space?**  
A: Yes, by sorting first (O(n log n) time) and checking adjacent elements. But this is slower.

**Q: What if the array is empty?**  
A: Return false. The loop doesn't execute; `set` is empty; we return false.

**Q: What if array has one element?**  
A: Return false. One element can't duplicate itself.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Detect if any duplicate exists in array |
| **Optimal Algorithm** | Hash Set membership check |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(n) |
| **Key Insight** | Set insertion failure = duplicate found |

</details>
