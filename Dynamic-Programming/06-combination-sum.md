# Combination Sum

---

<details>
<summary><strong>1. What is meant by Combination Sum?</strong></summary>

## What is Combination Sum?

**Combination Sum** is a backtracking problem where:

- You are given an array of distinct candidate integers `candidates`.
- You are given a target integer `target`.
- You can use the same candidate **unlimited times**.

**Goal:** Return all **unique combinations** of candidates that sum to `target`.

**Constraints:**
- `1 <= candidates.length <= 30`
- `2 <= candidates[i] <= 40`
- All candidate values are distinct.
- `1 <= target <= 40`

**Example:**

```
Input:  candidates = [2, 3, 6, 7], target = 7
Output: [[2, 2, 3], [7]]
Explanation: 2+2+3=7, 7=7
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Find all unique combinations that sum exactly to `target`.
- Same candidate can be reused any number of times.
- Each unique combination should appear only once (no duplicate sets).
- Order within a combination doesn't matter: `[2,3]` is the same as `[3,2]`.

### Non-Functional Requirements

| Concern         | Expectation                                             |
| --------------- | ------------------------------------------------------- |
| **Scale**       | candidates ≤ 30; target ≤ 40 — small search space      |
| **Latency**     | Exponential worst-case, but practical for given limits  |
| **Correctness** | Must not return duplicate combinations                  |
| **Output**      | All combinations in any order                           |

### Clarifying Questions to Ask

- Can elements be reused? → Yes (unlimited).
- Can two combinations be the same in different orders? → No (treat as same).
- Are candidates guaranteed to be distinct? → Yes.
- Return count or actual combinations? → Actual combinations.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Time Complexity

Let `T = target`, `M = min(candidates)`.

| | Complexity |
| --- | --- |
| **Time** | O(N^(T/M)) — N candidates, max depth T/M |
| **Space** | O(T/M) — recursion call stack depth |

**Why?** In the worst case (candidates = [1], target = T), each path has depth T and branches N ways.

### Practical Behavior

With candidate values ≥ 2 and target ≤ 40:
- Max depth = T/2 = 20
- Total nodes ≤ N^20 in the worst case, but heavily pruned in practice

### Sorting Optimization

Sorting candidates allows early pruning: once a candidate exceeds the remaining target, skip all remaining (they're larger too). This significantly reduces actual runtime.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Backtracking with Start Index

### Why Backtracking?

| Option | Correct? | Notes | Chosen? |
| --- | --- | --- | --- |
| DP (count only) | Partial | Can count combos but not enumerate them | No |
| BFS | Yes | Uses large memory for storing partial states | No |
| **Backtracking** | **Yes** | **Efficiently explores all paths, prunes early** | **Yes** |

### Key Design Decisions

1. **Start Index (`start`):** Each recursive call only considers candidates from index `start` onward. This prevents duplicate combinations like `[2,3]` and `[3,2]`.

2. **Early Termination:** If `remaining < 0`, prune this branch immediately.

3. **Sorting:** Sort candidates first to enable early pruning — when `candidate > remaining`, all subsequent candidates are also too large.

### Algorithm Steps

```
sort(candidates)
backtrack(start=0, remaining=target, current=[]):
    if remaining == 0: add current to results
    for i from start to len(candidates):
        if candidates[i] > remaining: break  // pruning
        current.add(candidates[i])
        backtrack(i, remaining - candidates[i], current)  // i (not i+1) to reuse
        current.remove(last)  // backtrack
```

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Sorter** — Sort candidates to enable pruning
2. **Backtracking Engine** — DFS with start index and remaining sum
3. **Pruner** — Skip candidates larger than remaining sum
4. **Result Collector** — Add combination when remaining = 0
5. **Output Layer** — Return all collected combinations

### Data Flow Diagram

```
candidates = [2, 3, 6, 7], target = 7
After sort: [2, 3, 6, 7]

Recursion Tree (partial):

backtrack(start=0, rem=7, path=[])
├── pick 2 → backtrack(start=0, rem=5, path=[2])
│   ├── pick 2 → backtrack(start=0, rem=3, path=[2,2])
│   │   ├── pick 2 → backtrack(start=0, rem=1, path=[2,2,2])
│   │   │   ├── pick 2 → rem=-1  PRUNE
│   │   │   └── pick 3 → rem=-2  PRUNE
│   │   └── pick 3 → backtrack(start=1, rem=0, path=[2,2,3]) ✓ FOUND!
│   └── pick 3 → backtrack(start=1, rem=2, path=[2,3])
│       └── pick 3 → rem=-1  PRUNE
└── pick 7 → backtrack(start=3, rem=0, path=[7]) ✓ FOUND!
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class CombinationSum {

    private List<List<Integer>> results;

    /**
     * Find all combinations of candidates that sum to target (reuse allowed).
     *
     * Strategy: Backtracking with start index to avoid duplicates.
     * Sort first to enable early pruning when candidate > remaining target.
     *
     * @param candidates array of distinct integers
     * @param target     desired sum
     * @return list of all unique combinations summing to target
     *
     * Time:  O(N^(T/M)) where T=target, M=min candidate
     * Space: O(T/M) recursion depth
     */
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        results = new ArrayList<>();
        Arrays.sort(candidates); // sort for early pruning
        backtrack(candidates, target, 0, new ArrayList<>());
        return results;
    }

    private void backtrack(int[] candidates, int remaining, int start, List<Integer> current) {
        if (remaining == 0) {
            results.add(new ArrayList<>(current)); // found a valid combination
            return;
        }

        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) break; // sorted → all further are too large

            current.add(candidates[i]);
            // i (not i+1): same candidate can be reused
            backtrack(candidates, remaining - candidates[i], i, current);
            current.remove(current.size() - 1); // undo the choice
        }
    }

    public static void main(String[] args) {
        CombinationSum sol = new CombinationSum();

        // Example 1
        System.out.println("candidates=[2,3,6,7], target=7");
        System.out.println("Output: " + sol.combinationSum(new int[]{2, 3, 6, 7}, 7));
        // [[2,2,3],[7]]

        // Example 2
        System.out.println("\ncandidates=[2,3,5], target=8");
        System.out.println("Output: " + sol.combinationSum(new int[]{2, 3, 5}, 8));
        // [[2,2,2,2],[2,3,3],[3,5]]

        // Example 3: No combination possible
        System.out.println("\ncandidates=[2], target=3");
        System.out.println("Output: " + sol.combinationSum(new int[]{2}, 3));
        // []

        // Example 4: Single candidate matches
        System.out.println("\ncandidates=[5], target=5");
        System.out.println("Output: " + sol.combinationSum(new int[]{5}, 5));
        // [[5]]
    }
}
```

### Output

```
candidates=[2,3,6,7], target=7
Output: [[2, 2, 3], [7]]

candidates=[2,3,5], target=8
Output: [[2, 2, 2, 2], [2, 3, 3], [3, 5]]

candidates=[2], target=3
Output: []

candidates=[5], target=5
Output: [[5]]
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def combination_sum(candidates: list[int], target: int) -> list[list[int]]:
    """
    Find all combinations of candidates summing to target (reuse allowed).

    Strategy: Backtracking with a start index.
    Sort candidates for early pruning.
    Only pick candidates from index `start` onward to avoid duplicates.

    Args:
        candidates: List of distinct integers
        target: Desired sum

    Returns:
        All unique combinations that sum to target

    Time:  O(N^(T/M))
    Space: O(T/M)
    """
    results = []
    candidates.sort()  # enable early termination

    def backtrack(start: int, remaining: int, path: list[int]) -> None:
        if remaining == 0:
            results.append(path[:])  # copy the current path
            return

        for i in range(start, len(candidates)):
            if candidates[i] > remaining:
                break  # sorted: all further candidates are also too large

            path.append(candidates[i])
            backtrack(i, remaining - candidates[i], path)  # i, not i+1 (reuse allowed)
            path.pop()  # backtrack

    backtrack(0, target, [])
    return results


if __name__ == "__main__":
    test_cases = [
        ([2, 3, 6, 7], 7),
        ([2, 3, 5],    8),
        ([2],          3),
        ([5],          5),
    ]

    for candidates, target in test_cases:
        result = combination_sum(candidates, target)
        print(f"candidates={candidates}, target={target}")
        print(f"  Output: {result}\n")
```

### Output

```
candidates=[2, 3, 6, 7], target=7
  Output: [[2, 2, 3], [7]]

candidates=[2, 3, 5], target=8
  Output: [[2, 2, 2, 2], [2, 3, 3], [3, 5]]

candidates=[2], target=3
  Output: []

candidates=[5], target=5
  Output: [[5]]
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why do we pass `i` (not `i+1`) in the recursive call?**
A: Because each candidate can be reused. Passing `i` means "we can still pick the same candidate again." Passing `i+1` would mean "move on to the next candidate" — used in Combination Sum II where elements can't be reused.

**Q: Why sort the candidates before backtracking?**
A: Sorting allows the early `break` when `candidates[i] > remaining`. Without sorting, you'd need to continue checking all remaining candidates. This is a performance optimization, not a correctness requirement.

**Q: How is Combination Sum II different from Combination Sum I?**
A: In Combination Sum II, each element can be used only once (0/1 usage) and the array may have duplicates. You pass `i+1` in the recursive call and skip duplicate values at the same recursion level.

**Q: Why do we append a copy of `path` (not `path` itself)?**
A: Because `path` is a mutable list that gets modified as we backtrack. If we append the reference, all entries in results will point to the same (eventually empty) list. Copying captures the current state.

**Q: What is the difference between Combination Sum and Subset Sum?**
A: Subset Sum asks "can we reach the target?" (boolean). Combination Sum asks "give all ways to reach the target" (enumeration). Combination Sum also allows element reuse; standard Subset Sum does not.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                               |
| --------------------- | ------------------------------------------------------------------- |
| **Problem**           | Find all unique combinations summing to target (reuse allowed)      |
| **Optimal Algorithm** | Backtracking with start index and early pruning                     |
| **Time Complexity**   | O(N^(T/M)) — N candidates, depth T/M                               |
| **Space Complexity**  | O(T/M) — recursion depth                                           |
| **Key Insight**       | Use `start` index to avoid duplicate combos; pass `i` (not i+1) to reuse |

### Core Pattern

> Backtracking explores all paths. The `start` index prevents counting `[2,3]` and `[3,2]` as different. Sort candidates to prune branches where no valid completion exists.

### When to Use This Pattern

- Enumerate all subsets/combinations satisfying a constraint.
- Related: Combination Sum II (no reuse), Subset Sum, Permutations, Subsets.

</details>
