# Top K Frequent Elements

---

1. What is meant by Top K Frequent Elements?

## What is Top K Frequent Elements?

**Top K Frequent Elements** is a heap-based problem where you are given:

- An integer array `nums`
- A positive integer `k`

**Goal:** Return the `k` most frequently occurring elements in the array.

**Constraints:**

- `1 <= k <= number of unique elements`
- The answer is guaranteed to be unique.
- You may return the answer in any order.

**Example:**

```
Input:  nums = [1, 1, 1, 2, 2, 3], k = 2
Output: [1, 2]
Explanation: 1 appears 3 times, 2 appears 2 times, 3 appears 1 time.
             Top 2 frequent → [1, 2]

Input:  nums = [1], k = 1
Output: [1]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Given an integer array and `k`, find the `k` elements with the highest occurrence counts.
- Return the elements (not their counts).
- Order of output does not matter.

### Non-Functional Requirements

| Concern         | Expectation                                                     |
| --------------- | --------------------------------------------------------------- |
| **Scale**       | Array can have up to 10⁵ elements                               |
| **Latency**     | Better than O(n log n); O(n log k) is the target               |
| **Correctness** | Must return exactly k elements with the highest frequencies     |
| **Uniqueness**  | Answer is guaranteed unique — no tie-breaking edge case         |

### Clarifying Questions to Ask

- Can `k` equal the total number of unique elements? → Yes
- Can there be negative numbers? → Yes
- Is the array sorted? → No
- Do we need to return results sorted by frequency? → No
- Can k = 0? → No, k ≥ 1 by constraints


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Sorting by Frequency

Sort all unique elements by frequency descending, take the first k.

| | Complexity |
| --- | --- |
| **Time** | O(n log n) — dominated by sort |
| **Space** | O(n) — frequency map + sorted list |

### Approach 2: Min-Heap of Size k (Optimal)

Count frequencies, maintain a min-heap of size k — evict the smallest when heap exceeds k.

| | Complexity |
| --- | --- |
| **Time** | O(n log k) — n insertions each O(log k) |
| **Space** | O(n + k) — frequency map O(n) + heap O(k) |

### Approach 3: Bucket Sort (O(n))

Use an array of buckets where index = frequency; scan from high to low.

| | Complexity |
| --- | --- |
| **Time** | O(n) — two linear passes |
| **Space** | O(n) — buckets array of size n+1 |

**Best practical choice:** Min-Heap O(n log k) — general, clean, interview-standard.  
**Best theoretical:** Bucket Sort O(n) — works only when n is bounded.


---

4. Algorithm Choice and Why

## Algorithm: Min-Heap of Size k

### Why Min-Heap?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force Sort | O(n log n) | O(n) | No — slower than needed |
| **Min-Heap (size k)** | **O(n log k)** | **O(n)** | **Yes** |
| Bucket Sort | O(n) | O(n) | Bonus — shown below |
| QuickSelect (partial sort) | O(n) avg | O(n) | Advanced, complex to implement |

### Why Min-Heap and not Max-Heap?

- A **max-heap** of all n elements costs O(n) to build, then O(k log n) to extract k — total O(n + k log n).
- A **min-heap of size k** keeps only k elements at any time. The minimum in the heap is the "weakest" candidate. When a new element beats it, evict the minimum and insert the new element.
- At the end, everything left in the heap belongs to the top-k.

### Min-Heap Logic

1. Build a frequency map: `num → count` — O(n)
2. Iterate over all unique elements:
   - Push `(count, num)` onto the min-heap
   - If heap size exceeds k, pop the smallest
3. Return all elements remaining in the heap

```
freq = {1:3, 2:2, 3:1}, k = 2

Push (3,1) → heap: [(3,1)]
Push (2,2) → heap: [(2,2), (3,1)]
Push (1,3) → heap size 3 > k=2 → pop min (1,3) → heap: [(2,2),(3,1)]

Result: [1, 2]
```


---

5. High-Level Design

## High-Level Design

### Components

1. **Input Layer** — Receives `nums[]` and `k`
2. **Frequency Counter** — Hash map: `value → occurrence count`
3. **Min-Heap (size k)** — Stores `(frequency, value)` pairs; maintains the top-k candidates
4. **Output Layer** — Extracts values from the final heap

### Data Flow Diagram

```
Input: nums = [1, 1, 1, 2, 2, 3], k = 2

Step 1 — Build Frequency Map
  {1: 3, 2: 2, 3: 1}

Step 2 — Process each (freq, num) through Min-Heap
  Insert (3, 1) → heap: [(3, 1)]
  Insert (2, 2) → heap: [(2, 2), (3, 1)]
  Insert (1, 3) → heap size = 3 > k = 2 → evict min (1, 3)
               → heap: [(2, 2), (3, 1)]

Step 3 — Extract results from heap
  Output: [1, 2]
```

### Visual

```
Frequency Map          Min-Heap (size k=2)
┌─────┬───────┐        ┌──────────┐
│ num │ count │        │ (2, 2)   │  ← smallest in top-k
├─────┼───────┤        │ (3, 1)   │
│  1  │   3   │        └──────────┘
│  2  │   2   │
│  3  │   1   │  ← evicted (below the cutoff)
└─────┴───────┘
```


---

6. Java Solution

## Java Implementation

```java
import java.util.*;

public class TopKFrequentElements {

    /**
     * Returns the k most frequent elements using a min-heap of size k.
     *
     * Strategy:
     *  1. Build a frequency map in O(n).
     *  2. Use a min-heap (PriorityQueue ordered by frequency ascending).
     *  3. Push each unique element; if heap exceeds k, pop the minimum.
     *  4. The heap retains only the top-k frequent elements.
     *
     * Time:  O(n log k)
     * Space: O(n + k)
     *
     * @param nums input array
     * @param k    number of top frequent elements to return
     * @return array of k most frequent elements
     */
    public int[] topKFrequent(int[] nums, int k) {
        // Step 1: Count frequencies
        Map<Integer, Integer> freq = new HashMap<>();
        for (int num : nums) {
            freq.put(num, freq.getOrDefault(num, 0) + 1);
        }

        // Step 2: Min-heap ordered by frequency (smallest freq at top)
        // Entry: (frequency, element)
        PriorityQueue<int[]> minHeap = new PriorityQueue<>(
            (a, b) -> a[0] - b[0]  // ascending by frequency
        );

        for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
            minHeap.offer(new int[]{entry.getValue(), entry.getKey()});
            // Evict the element with lowest frequency when heap exceeds k
            if (minHeap.size() > k) {
                minHeap.poll();
            }
        }

        // Step 3: Extract results — everything in the heap belongs to top-k
        int[] result = new int[k];
        for (int i = 0; i < k; i++) {
            result[i] = minHeap.poll()[1];
        }
        return result;
    }

    // ---- Main: Example usage ----
    public static void main(String[] args) {
        TopKFrequentElements solution = new TopKFrequentElements();

        // Example 1
        int[] nums1 = {1, 1, 1, 2, 2, 3};
        int k1 = 2;
        System.out.println("Input: nums=[1,1,1,2,2,3], k=2");
        System.out.println("Output: " + Arrays.toString(solution.topKFrequent(nums1, k1)));
        // Output: [1, 2] (any order)

        // Example 2
        int[] nums2 = {1};
        int k2 = 1;
        System.out.println("\nInput: nums=[1], k=1");
        System.out.println("Output: " + Arrays.toString(solution.topKFrequent(nums2, k2)));
        // Output: [1]

        // Example 3: all equal frequency except one
        int[] nums3 = {4, 4, 4, 6, 6, 7};
        int k3 = 1;
        System.out.println("\nInput: nums=[4,4,4,6,6,7], k=1");
        System.out.println("Output: " + Arrays.toString(solution.topKFrequent(nums3, k3)));
        // Output: [4]
    }
}
```

### Output

```
Input: nums=[1,1,1,2,2,3], k=2
Output: [2, 1]

Input: nums=[1], k=1
Output: [1]

Input: nums=[4,4,4,6,6,7], k=1
Output: [4]
```

> Note: Output order may vary — any ordering of the top-k elements is valid.

### Bonus: Bucket Sort — O(n)

```java
public int[] topKFrequentBucket(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) freq.put(num, freq.getOrDefault(num, 0) + 1);

    // Buckets indexed by frequency (0..n)
    List<List<Integer>> buckets = new ArrayList<>();
    for (int i = 0; i <= nums.length; i++) buckets.add(new ArrayList<>());

    for (Map.Entry<Integer, Integer> entry : freq.entrySet()) {
        buckets.get(entry.getValue()).add(entry.getKey());
    }

    // Collect top-k from high-frequency buckets downward
    int[] result = new int[k];
    int idx = 0;
    for (int f = nums.length; f >= 0 && idx < k; f--) {
        for (int num : buckets.get(f)) {
            if (idx == k) break;
            result[idx++] = num;
        }
    }
    return result;
}
```


---

7. Python Solution

## Python Implementation

```python
import heapq
from collections import Counter
from typing import List


def top_k_frequent(nums: List[int], k: int) -> List[int]:
    """
    Return the k most frequent elements using a min-heap of size k.

    Strategy:
      1. Count frequencies with Counter — O(n).
      2. Maintain a min-heap of (frequency, element) pairs capped at size k.
         The heap's minimum is always the weakest candidate — evict it when
         a new element arrives and the heap is full.
      3. Return the elements remaining in the heap.

    Args:
        nums: Input integer list.
        k:    Number of top-frequent elements to return.

    Returns:
        List of k elements with highest frequencies (any order).

    Time:  O(n log k)
    Space: O(n + k)
    """
    # Step 1: frequency map in O(n)
    freq = Counter(nums)  # {element: count}

    # Step 2: min-heap of size k
    # heapq in Python is a min-heap by default
    # Push (frequency, element) so smallest frequency is at the top
    min_heap: list = []
    for num, count in freq.items():
        heapq.heappush(min_heap, (count, num))
        if len(min_heap) > k:
            heapq.heappop(min_heap)  # evict least frequent

    # Step 3: extract values
    return [num for count, num in min_heap]


# ---- Bonus: O(n) Bucket Sort approach ----
def top_k_frequent_bucket(nums: List[int], k: int) -> List[int]:
    """
    O(n) solution using bucket sort.
    Buckets are indexed by frequency; scan from highest to lowest to pick top-k.

    Time:  O(n)
    Space: O(n)
    """
    freq = Counter(nums)
    # buckets[i] holds all elements with frequency == i
    buckets: List[List[int]] = [[] for _ in range(len(nums) + 1)]

    for num, count in freq.items():
        buckets[count].append(num)

    result: List[int] = []
    for f in range(len(buckets) - 1, 0, -1):
        result.extend(buckets[f])
        if len(result) >= k:
            break

    return result[:k]


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: standard case
    nums1, k1 = [1, 1, 1, 2, 2, 3], 2
    print(f"Input: nums={nums1}, k={k1}")
    print(f"Output (heap):   {top_k_frequent(nums1, k1)}")        # [1, 2]
    print(f"Output (bucket): {top_k_frequent_bucket(nums1, k1)}") # [1, 2]

    # Example 2: single element
    nums2, k2 = [1], 1
    print(f"\nInput: nums={nums2}, k={k2}")
    print(f"Output: {top_k_frequent(nums2, k2)}")  # [1]

    # Example 3: three elements, pick top 1
    nums3, k3 = [4, 4, 4, 6, 6, 7], 1
    print(f"\nInput: nums={nums3}, k={k3}")
    print(f"Output: {top_k_frequent(nums3, k3)}")  # [4]
```

### Output

```
Input: nums=[1, 1, 1, 2, 2, 3], k=2
Output (heap):   [2, 1]
Output (bucket): [1, 2]

Input: nums=[1], k=1
Output: [1]

Input: nums=[4, 4, 4, 6, 6, 7], k=1
Output: [4]
```


---

8. Interview Q&A

## Q&A

**Q: Why use a min-heap instead of a max-heap?**  
A: A max-heap of all n elements would require O(n) space and O(k log n) extractions. A min-heap capped at size k costs O(n log k) total and uses only O(k) heap space — better when k << n.

**Q: Why can't we just sort the frequency map?**  
A: Sorting gives O(n log n). A min-heap of size k gives O(n log k). When k is small (e.g., k=5 out of 10⁶ elements), the heap is significantly faster.

**Q: What is the bucket sort approach, and when is it better?**  
A: Bucket sort uses an array of size n+1 where index = frequency. We scan from highest index down and collect k elements. It runs in O(n) time — optimal — but only works when frequencies are bounded by n (always true here). Use it when k is large relative to n.

**Q: Can there be ties in frequency for the k-th position?**  
A: The problem guarantees a unique answer, so tie-breaking is not required. In real scenarios you'd need a secondary sort criterion.

**Q: What happens if all elements have the same frequency?**  
A: The heap still works; it picks any k elements arbitrarily (since all frequencies are equal). The answer is valid because the problem guarantees uniqueness.

**Q: Is the output order guaranteed?**  
A: No — the problem allows returning top-k elements in any order. The heap does not maintain a sorted order of frequencies.

**Q: What is the space complexity breakdown?**  
A: O(n) for the frequency Counter/HashMap, plus O(k) for the heap — total O(n + k), which simplifies to O(n) since k ≤ n.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Find k elements with highest occurrence counts                     |
| **Optimal Algorithm** | Min-Heap of size k                                                 |
| **Time Complexity**   | O(n log k)                                                         |
| **Space Complexity**  | O(n + k)                                                           |
| **Key Insight**       | Keep a size-k min-heap so the weakest candidate is always evictable |
| **Bonus O(n)**        | Bucket sort when frequency is bounded by array length              |

### Core Pattern

> Use a **min-heap of fixed size k** to efficiently maintain the top-k candidates: the minimum of the heap is always the current weakest element — evict it whenever a stronger candidate arrives.

### When to Use This Pattern

- Any "top-k by some score" problem (frequent elements, largest values, closest points)
- When k << n and you want better than O(n log n)
- Stream processing where you can't hold all elements in memory

