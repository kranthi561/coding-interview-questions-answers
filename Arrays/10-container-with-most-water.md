# Container With Most Water

---

1. What is meant by Container With Most Water?

## What is Container With Most Water?

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `i`th line are `(i, 0)` and `(i, height[i])`.

Find two lines that together with the x-axis form a container, such that the container contains the **most water**.

**Return the maximum amount of water a container can store.**

**Example:**
```
Input:  height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
Output: 49
Explanation: Lines at index 1 (height 8) and index 8 (height 7).
             Width = 8 - 1 = 7. Height = min(8, 7) = 7. Area = 7 * 7 = 49.
```

```
Input:  height = [1, 1]
Output: 1
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements
- Given line heights, find two lines that maximize the water area.
- Area = `min(height[left], height[right]) × (right - left)`
- Return the maximum area.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 10⁵ lines |
| **Time** | O(n) expected |
| **Output** | Single integer (max area) |

### Clarifying Questions
- Can we use the same line twice? → No, must pick two different lines.
- Can heights be zero? → Yes, area would be zero.
- Do we need to return the indices or just the max area? → Just the area.


---

3. Time and Space Complexity

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Brute force (all pairs) | O(n²) | O(1) |
| **Two Pointers** | **O(n)** | **O(1)** |

**Two pointers** move from both ends toward the center — each step guaranteed to not miss a better answer.


---

4. Algorithm Choice and Why

## Algorithm: Two Pointers

### Key Insight
Start with `left = 0`, `right = n-1` (widest possible container).

**To increase area, we must increase height** (width can only shrink as we move inward).

Moving the **taller** pointer inward is pointless — the area is already limited by the shorter side, and width decreases. We must move the **shorter** pointer, hoping to find a taller line.

### Decision Rule
```
area = min(height[left], height[right]) × (right - left)

If height[left] < height[right]:
    move left pointer right → left++
Else:
    move right pointer left → right--
```

### Why This Is Correct (Proof by Contradiction)
If we skip a pair by moving a pointer, can we miss the optimal? No — the skipped pairs would all have a smaller or equal width AND the same limiting height, so they can't beat the current area.


---

5. High-Level Design

## High-Level Design

### Data Flow
```
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
          0  1  2  3  4  5  6  7  8

left=0, right=8: area=min(1,7)*8=8,  maxArea=8,  move left (1<7)
left=1, right=8: area=min(8,7)*7=49, maxArea=49, move right (7<8)
left=1, right=7: area=min(8,3)*6=18, maxArea=49, move right (3<8)
left=1, right=6: area=min(8,8)*5=40, maxArea=49, move either
...converge...

Result: 49
```

### Visual
```
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
              ↑                    ↑
            left=1              right=8
            h=8                  h=7
            area = 7 × 7 = 49 ← maximum
```


---

6. Java Solution with Explanation

## Java Implementation

```java
public class ContainerWithMostWater {

    /**
     * Finds maximum water area using two-pointer technique.
     * Start at widest; always move the shorter pointer inward
     * to try finding a taller line (only way area can increase).
     *
     * @param height array of line heights
     * @return maximum water the container can hold
     */
    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int maxArea = 0;

        while (left < right) {
            // Area = min height × width
            int area = Math.min(height[left], height[right]) * (right - left);
            maxArea = Math.max(maxArea, area);

            // Move the shorter line — only way to potentially increase area
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }

        return maxArea;
    }

    public static void main(String[] args) {
        ContainerWithMostWater solution = new ContainerWithMostWater();

        // Example 1
        int[] height1 = {1, 8, 6, 2, 5, 4, 8, 3, 7};
        System.out.println("Input: [1,8,6,2,5,4,8,3,7]");
        System.out.println("Output: " + solution.maxArea(height1)); // 49

        // Example 2
        int[] height2 = {1, 1};
        System.out.println("\nInput: [1,1]");
        System.out.println("Output: " + solution.maxArea(height2)); // 1

        // Example 3: All same height
        int[] height3 = {4, 4, 4, 4};
        System.out.println("\nInput: [4,4,4,4]");
        System.out.println("Output: " + solution.maxArea(height3)); // 12
    }
}
```

### Output
```
Input: [1,8,6,2,5,4,8,3,7]
Output: 49

Input: [1,1]
Output: 1

Input: [4,4,4,4]
Output: 12
```


---

7. Python Solution with Explanation

## Python Implementation

```python
def max_area(height: list[int]) -> int:
    """
    Find maximum water container using two pointers.

    Start from outermost edges (maximum width).
    Move the shorter pointer inward — only this can increase area.

    Time:  O(n)
    Space: O(1)
    """
    left, right = 0, len(height) - 1
    max_water = 0

    while left < right:
        # Compute current area
        water = min(height[left], height[right]) * (right - left)
        max_water = max(max_water, water)

        # Move the pointer with smaller height
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return max_water


if __name__ == "__main__":
    print(max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]))  # 49
    print(max_area([1, 1]))                          # 1
    print(max_area([4, 4, 4, 4]))                   # 12
```

### Output
```
49
1
12
```


---

8. Interview Q&A

## Q&A

**Q: Why start at the two ends?**  
A: The widest span gives the highest potential area for a given height. Starting wide and shrinking ensures we never miss the optimal pair.

**Q: Why move the shorter pointer, not the taller one?**  
A: The area is limited by the shorter line. Moving the taller line can only decrease width while the limiting height stays the same or gets smaller — area can only decrease. Moving the shorter line at least has a chance of finding a taller line.

**Q: Is it safe to skip the pair at each step?**  
A: Yes. When we move pointer `left` past a value, we're skipping all pairs `(left, j)` for `j < right`. But those pairs are all narrower and height-limited by `height[left]`, so they can't beat the area we already computed.

**Q: What if all heights are equal?**  
A: The maximum area is `height[0] × (n-1)` (widest container). The algorithm captures this on the first iteration.

**Q: Can this be extended to 3D (Trapping Rain Water)?**  
A: Trapping Rain Water (LeetCode 42) is a related problem but different — it sums water trapped at each bar between walls, not the area of a single container.


---

9. Summary

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Maximum water between two vertical lines |
| **Optimal Algorithm** | Two Pointers (start at extremes) |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) |
| **Key Insight** | Always move the shorter pointer — it's the only way area can grow |

### Core Pattern
> Greedy two-pointer convergence: start at maximum width, greedily try to increase height by moving the limiting (shorter) pointer.

### When to Use This Pattern
- Maximize/minimize a value that depends on two array positions
- The value is bounded by the smaller of the two sides
- Sorted or ordered structure is not required

