# Search in Rotated Sorted Array

---

1. What is meant by Search in Rotated Sorted Array?

## What is Search in Rotated Sorted Array?

There is a sorted array that has been rotated at an unknown pivot. Given the rotated array and a target value, return the **index** of the target, or `-1` if it doesn't exist.

**Must run in O(log n) time.**

**Example:**
```
Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 0
Output: 4
```

```
Input:  nums = [4, 5, 6, 7, 0, 1, 2], target = 3
Output: -1
```

```
Input:  nums = [1], target = 0
Output: -1
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements
- Find and return the index of `target` in a rotated sorted array.
- Return -1 if not found.
- Must run in **O(log n)** time.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 5000 elements |
| **Time** | O(log n) — binary search |
| **Duplicates** | No duplicates (this variant) |

### Clarifying Questions
- Are there duplicates? → No.
- Can `target` equal multiple elements? → No duplicates guaranteed.
- What if array has one element? → Check it directly.


---

3. Time and Space Complexity

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Linear scan | O(n) | O(1) |
| **Modified Binary Search** | **O(log n)** | **O(1)** |


---

4. Algorithm Choice and Why

## Algorithm: Modified Binary Search

### Key Insight
In a rotated array, at least one half of `[left, mid]` or `[mid, right]` is always **fully sorted**. We determine which half is sorted and check if the target falls within that range. If yes, search there; otherwise search the other half.

### Decision Logic
```
If nums[left] <= nums[mid]:  (left half is sorted)
    If nums[left] <= target < nums[mid]:
        → search left half
    Else:
        → search right half
Else:  (right half is sorted)
    If nums[mid] < target <= nums[right]:
        → search right half
    Else:
        → search left half
```


---

5. High-Level Design

## High-Level Design

### Data Flow
```
nums = [4, 5, 6, 7, 0, 1, 2], target = 0

left=0, right=6
Iter 1: mid=3, nums[3]=7
  nums[0]=4 <= nums[3]=7 → left half [4,5,6,7] is sorted
  target=0 not in [4..7) → search right: left=4

left=4, right=6
Iter 2: mid=5, nums[5]=1
  nums[4]=0 <= nums[5]=1 → left half [0,1] is sorted
  target=0 in [0..1) → search left: right=5

left=4, right=5
Iter 3: mid=4, nums[4]=0
  nums[4]=0 <= nums[4]=0 → single-element left half
  target=0 in range → right=4

left=4, right=4 → loop ends
nums[4] = 0 == target → return 4
```


---

6. Java Solution with Explanation

## Java Implementation

```java
public class SearchInRotatedSortedArray {

    /**
     * Searches for target in a rotated sorted array.
     * Modified binary search: one half is always sorted — check if target is in sorted half.
     *
     * @param nums   rotated sorted array with distinct values
     * @param target value to search for
     * @return index of target, or -1 if not found
     */
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            }

            // Check if left half is sorted
            if (nums[left] <= nums[mid]) {
                if (nums[left] <= target && target < nums[mid]) {
                    right = mid - 1; // target in left sorted half
                } else {
                    left = mid + 1;  // target in right half
                }
            } else {
                // Right half is sorted
                if (nums[mid] < target && target <= nums[right]) {
                    left = mid + 1;  // target in right sorted half
                } else {
                    right = mid - 1; // target in left half
                }
            }
        }

        return -1;
    }

    public static void main(String[] args) {
        SearchInRotatedSortedArray solution = new SearchInRotatedSortedArray();

        System.out.println(solution.search(new int[]{4,5,6,7,0,1,2}, 0));  // 4
        System.out.println(solution.search(new int[]{4,5,6,7,0,1,2}, 3));  // -1
        System.out.println(solution.search(new int[]{1}, 0));               // -1
        System.out.println(solution.search(new int[]{1, 3}, 3));           // 1
    }
}
```

### Output
```
4
-1
-1
1
```


---

7. Python Solution with Explanation

## Python Implementation

```python
def search(nums: list[int], target: int) -> int:
    """
    Binary search in a rotated sorted array.

    One half is always sorted — determine which half, then
    check if target falls within it to decide which way to go.

    Time:  O(log n)
    Space: O(1)
    """
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = (left + right) // 2

        if nums[mid] == target:
            return mid

        if nums[left] <= nums[mid]:  # left half is sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # right half is sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1

    return -1


if __name__ == "__main__":
    print(search([4, 5, 6, 7, 0, 1, 2], 0))  # 4
    print(search([4, 5, 6, 7, 0, 1, 2], 3))  # -1
    print(search([1], 0))                      # -1
    print(search([1, 3], 3))                  # 1
```

### Output
```
4
-1
-1
1
```


---

8. Interview Q&A

## Q&A

**Q: How is this different from "Find Minimum in Rotated Sorted Array"?**  
A: Finding the minimum uses the invariant `nums[mid] vs nums[right]`. Searching for a target needs to know which half is sorted AND whether target is in that half.

**Q: Why use `nums[left] <= nums[mid]` (not `<`)?**  
A: When `left == mid` (single element), `nums[left] <= nums[mid]` is true, correctly identifying a "sorted" single-element left half.

**Q: What if there are duplicates?**  
A: When `nums[left] == nums[mid]`, we can't determine which half is sorted. We'd need to increment `left` linearly — degrading worst case to O(n).

**Q: Can we first find the pivot, then do two binary searches?**  
A: Yes, that's a valid O(log n) two-pass approach. But the single-pass method described here is cleaner and just as fast.


---

9. Summary

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Find target index in rotated sorted array |
| **Optimal Algorithm** | Modified Binary Search |
| **Time Complexity** | O(log n) |
| **Space Complexity** | O(1) |
| **Key Insight** | One half is always sorted — check if target is in the sorted half |

