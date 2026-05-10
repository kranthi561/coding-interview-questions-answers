# Find Minimum in Rotated Sorted Array

---

<details>
<summary><strong>1. What is meant by Find Minimum in Rotated Sorted Array?</strong></summary>

## What is Find Minimum in Rotated Sorted Array?

Suppose a sorted array has been **rotated** at some unknown pivot index. Given this rotated array, find the minimum element.

**You must write an algorithm that runs in O(log n) time.**

**Example:**
```
Input:  nums = [3, 4, 5, 1, 2]
Output: 1
Explanation: Original: [1,2,3,4,5]. Rotated at index 3.
```

```
Input:  nums = [4, 5, 6, 7, 0, 1, 2]
Output: 0
```

```
Input:  nums = [11, 13, 15, 17]
Output: 11
Explanation: No rotation — array is already sorted.
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Array was originally sorted in ascending order, then rotated at some pivot.
- Find the minimum element.
- **Must be O(log n)** — linear scan is not acceptable.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 5000 elements |
| **Time** | O(log n) — binary search |
| **Duplicates** | No duplicates (LeetCode 153; LeetCode 154 has duplicates) |

### Clarifying Questions
- Are there duplicates? → No (for this variant).
- What if not rotated at all? → Still works; return nums[0].
- Can the array be empty? → Guaranteed at least one element.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Linear scan | O(n) | O(1) |
| **Binary Search** | **O(log n)** | **O(1)** |

**Binary search** eliminates half the array each iteration using the invariant: the minimum lies in the unsorted half.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Binary Search

### Key Insight
In a rotated sorted array, comparing `mid` element with `right` element tells us which half is sorted:
- If `nums[mid] > nums[right]` → the minimum is in the **right half** (the left is sorted but doesn't contain min)
- If `nums[mid] <= nums[right]` → the minimum is in the **left half** (including mid)

### Why This Works
The rotation creates exactly one "drop point" where the array goes from high to low. Binary search finds this drop.

```
[3, 4, 5, 1, 2]
        ↑
      drop (5→1) — minimum is to the right of here
```

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Data Flow
```
nums = [3, 4, 5, 1, 2]
left=0, right=4

Iteration 1:
  mid = 2, nums[mid]=5, nums[right]=2
  5 > 2 → min is in right half → left = mid+1 = 3

Iteration 2:
  mid = 3, nums[mid]=1, nums[right]=2
  1 <= 2 → min is in left half (include mid) → right = mid = 3

Loop ends: left == right == 3
Answer: nums[3] = 1
```

### Visual
```
Index:  0  1  2  3  4
Array:  3  4  5  1  2
                ↑
             minimum
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class FindMinimumInRotatedSortedArray {

    /**
     * Finds the minimum element in a rotated sorted array using binary search.
     * Compares mid with right to determine which half contains the minimum.
     *
     * @param nums rotated sorted array with distinct values
     * @return the minimum element
     */
    public int findMin(int[] nums) {
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                // Left half is sorted; minimum is in the right half
                left = mid + 1;
            } else {
                // Right half is sorted (or no rotation); minimum is at mid or left of it
                right = mid;
            }
        }

        // left == right, pointing to the minimum
        return nums[left];
    }

    public static void main(String[] args) {
        FindMinimumInRotatedSortedArray solution = new FindMinimumInRotatedSortedArray();

        System.out.println(solution.findMin(new int[]{3, 4, 5, 1, 2}));        // 1
        System.out.println(solution.findMin(new int[]{4, 5, 6, 7, 0, 1, 2})); // 0
        System.out.println(solution.findMin(new int[]{11, 13, 15, 17}));       // 11
        System.out.println(solution.findMin(new int[]{2, 1}));                 // 1
    }
}
```

### Output
```
1
0
11
1
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def find_min(nums: list[int]) -> int:
    """
    Find minimum in a rotated sorted array using binary search.

    Compare nums[mid] with nums[right]:
    - If nums[mid] > nums[right]: min is in right half → left = mid + 1
    - Otherwise: min is in left half (including mid) → right = mid

    Time:  O(log n)
    Space: O(1)
    """
    left, right = 0, len(nums) - 1

    while left < right:
        mid = (left + right) // 2

        if nums[mid] > nums[right]:
            left = mid + 1  # min is in right half
        else:
            right = mid     # min is at mid or in left half

    return nums[left]


if __name__ == "__main__":
    print(find_min([3, 4, 5, 1, 2]))        # 1
    print(find_min([4, 5, 6, 7, 0, 1, 2])) # 0
    print(find_min([11, 13, 15, 17]))       # 11
    print(find_min([2, 1]))                 # 1
```

### Output
```
1
0
11
1
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why compare `mid` with `right` instead of `left`?**  
A: Comparing with `right` cleanly identifies which half is unsorted. Comparing with `left` leads to edge cases because `nums[left] <= nums[mid]` is true in both the sorted and partially rotated cases.

**Q: Why use `right = mid` and not `right = mid - 1`?**  
A: Because `mid` itself could be the minimum. We must not discard it.

**Q: What if the array is not rotated?**  
A: `nums[mid] <= nums[right]` is always true for a sorted array, so `right` converges to 0 — returning `nums[0]`, which is correct.

**Q: How would you handle duplicates (LeetCode 154)?**  
A: When `nums[mid] == nums[right]`, we can't determine which half is sorted. The safest approach is `right--`, falling back to O(n) worst case.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Find minimum in rotated sorted array |
| **Optimal Algorithm** | Binary Search |
| **Time Complexity** | O(log n) |
| **Space Complexity** | O(1) |
| **Key Insight** | If `nums[mid] > nums[right]`, min is in right half; otherwise in left |

</details>
