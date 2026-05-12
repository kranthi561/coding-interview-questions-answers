# Climbing Stairs

---

1. What is meant by Climbing Stairs?

## What is Climbing Stairs?

**Climbing Stairs** is a classic dynamic programming problem where:

- You are climbing a staircase with `n` steps.
- Each time you can climb **1 or 2 steps**.

**Goal:** Return the number of **distinct ways** you can reach the top.

**Constraints:**
- `1 <= n <= 45`
- Each step must be 1 or 2 at a time.

**Example:**

```
Input:  n = 3
Output: 3
Explanation:
  1 step + 1 step + 1 step
  1 step + 2 steps
  2 steps + 1 step
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Given `n` steps, count all distinct ways to reach step `n`.
- Each move is either 1 step or 2 steps.
- Order of steps matters (1+2 ≠ 2+1).

### Non-Functional Requirements

| Concern         | Expectation                                      |
| --------------- | ------------------------------------------------ |
| **Scale**       | n up to 45 (small; fits in a single integer)     |
| **Latency**     | O(n) — runs in microseconds                      |
| **Correctness** | Must count all distinct ordered sequences        |
| **Space**       | Can be reduced to O(1) with two variables        |

### Clarifying Questions to Ask

- Can n be 0? → Conventionally 1 way (do nothing), but constraints say n ≥ 1.
- Are steps ordered or unordered? → Ordered (different order = different way).
- Can steps larger than 2 be used? → No, only 1 or 2.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

### Approach 1: Naive Recursion (No Memoization)

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — exponential, re-computes subproblems |
| **Space** | O(n) — recursion call stack depth |

### Approach 2: Memoization (Top-Down DP)

| | Complexity |
| --- | --- |
| **Time** | O(n) — each subproblem computed once |
| **Space** | O(n) — memo array + call stack |

### Approach 3: Bottom-Up DP (Tabulation)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single loop |
| **Space** | O(n) — dp array |

### Approach 4: Space-Optimized DP (Two Variables)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single loop |
| **Space** | O(1) — only two variables |


---

4. Which Algorithm and Why?

## Algorithm: Space-Optimized Fibonacci DP

### Why this approach?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Naive Recursion | O(2ⁿ) | O(n) | No — too slow |
| Memoization | O(n) | O(n) | No — extra space |
| Bottom-Up DP | O(n) | O(n) | No — full array not needed |
| **Two-Variable DP** | **O(n)** | **O(1)** | **Yes** |

### Key Insight

This is the **Fibonacci sequence** in disguise:
- `ways(1) = 1` (only one way: 1 step)
- `ways(2) = 2` (1+1 or 2)
- `ways(n) = ways(n-1) + ways(n-2)`

Because at step `n`, you arrived from step `n-1` (took 1 step) or step `n-2` (took 2 steps).

You only ever need the last two values, so two variables replace the full DP array.


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Input Validator** — Ensure `n >= 1`
2. **Base Case Handler** — Return directly for n=1 or n=2
3. **DP Stepper** — Iteratively compute the next Fibonacci value
4. **Output** — Return the count at step n

### Data Flow Diagram

```
n = 5

Initial: prev2 = 1 (ways to reach step 1)
         prev1 = 2 (ways to reach step 2)

Step 3: curr = prev1 + prev2 = 2 + 1 = 3  → prev2=2, prev1=3
Step 4: curr = prev1 + prev2 = 3 + 2 = 5  → prev2=3, prev1=5
Step 5: curr = prev1 + prev2 = 5 + 3 = 8  → prev2=5, prev1=8

Output: 8
```

### Visual — Fibonacci Tree (n=4)

```
              f(4)
            /      \
         f(3)      f(2)
        /    \    /    \
      f(2)  f(1) f(1) f(0)
      / \
   f(1) f(0)

Without memoization, f(2) is computed twice — DP avoids this.
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
public class ClimbingStairs {

    /**
     * Count distinct ways to climb n stairs (1 or 2 steps at a time).
     *
     * Pattern: Fibonacci — ways(n) = ways(n-1) + ways(n-2)
     * Space-optimized: track only the last two values.
     *
     * @param n number of stairs
     * @return number of distinct ways to reach the top
     *
     * Time:  O(n)
     * Space: O(1)
     */
    public int climbStairs(int n) {
        if (n <= 2) return n;  // Base cases: 1→1 way, 2→2 ways

        int prev2 = 1; // ways to reach step 1
        int prev1 = 2; // ways to reach step 2

        for (int step = 3; step <= n; step++) {
            int curr = prev1 + prev2; // arrive from step-1 or step-2
            prev2 = prev1;
            prev1 = curr;
        }

        return prev1;
    }

    public static void main(String[] args) {
        ClimbingStairs solution = new ClimbingStairs();

        // Example 1
        System.out.println("Input: n=2");
        System.out.println("Output: " + solution.climbStairs(2)); // 2

        // Example 2
        System.out.println("\nInput: n=3");
        System.out.println("Output: " + solution.climbStairs(3)); // 3

        // Example 3
        System.out.println("\nInput: n=5");
        System.out.println("Output: " + solution.climbStairs(5)); // 8

        // Example 4
        System.out.println("\nInput: n=10");
        System.out.println("Output: " + solution.climbStairs(10)); // 89
    }
}
```

### Output

```
Input: n=2
Output: 2

Input: n=3
Output: 3

Input: n=5
Output: 8

Input: n=10
Output: 89
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
def climb_stairs(n: int) -> int:
    """
    Count distinct ways to climb n stairs (1 or 2 steps at a time).

    Observation: ways(n) = ways(n-1) + ways(n-2) — identical to Fibonacci.
    Use two variables to avoid O(n) space of a full dp array.

    Args:
        n: Number of stairs (n >= 1)

    Returns:
        Number of distinct ways to reach step n

    Time:  O(n)
    Space: O(1)
    """
    if n <= 2:
        return n  # 1 stair = 1 way, 2 stairs = 2 ways

    prev2, prev1 = 1, 2  # ways to reach steps 1 and 2

    for _ in range(3, n + 1):
        curr = prev1 + prev2  # sum of the two preceding values
        prev2, prev1 = prev1, curr

    return prev1


if __name__ == "__main__":
    test_cases = [1, 2, 3, 5, 10, 45]

    for n in test_cases:
        print(f"Input: n={n}  →  Output: {climb_stairs(n)}")
```

### Output

```
Input: n=1  →  Output: 1
Input: n=2  →  Output: 2
Input: n=3  →  Output: 3
Input: n=5  →  Output: 8
Input: n=10  →  Output: 89
Input: n=45  →  Output: 1836311903
```


---

8. Interview Questions and Answers

## Q&A

**Q: Why is this the Fibonacci sequence?**
A: At any step `n`, you can only arrive from `n-1` (one step back) or `n-2` (two steps back). So `ways(n) = ways(n-1) + ways(n-2)` — the exact Fibonacci recurrence.

**Q: Why use two variables instead of a full DP array?**
A: Each step only depends on the previous two. We never look back further, so storing the full array wastes O(n) space unnecessarily.

**Q: What if you could take 1, 2, or 3 steps?**
A: Extend to three variables: `ways(n) = ways(n-1) + ways(n-2) + ways(n-3)`, tracking `prev3`, `prev2`, `prev1`.

**Q: Can recursion with memoization be used instead?**
A: Yes, it's equivalent in time complexity O(n). But iterative DP avoids recursion call-stack overhead and potential stack overflow for large n.

**Q: What is the maximum possible output for n=45?**
A: `1,836,311,903` — fits in a 32-bit int (Java `int` or Python `int`).

**Q: How does this relate to the Fibonacci sequence exactly?**
A: `climbStairs(n) = Fibonacci(n+1)`, where `Fibonacci(1)=1, Fibonacci(2)=1, Fibonacci(3)=2, ...`.


---

9. Summary

## Summary

| Attribute             | Value                                                        |
| --------------------- | ------------------------------------------------------------ |
| **Problem**           | Count distinct ways to climb n stairs (1 or 2 steps)         |
| **Optimal Algorithm** | Space-Optimized Fibonacci DP (two variables)                 |
| **Time Complexity**   | O(n)                                                         |
| **Space Complexity**  | O(1)                                                         |
| **Key Insight**       | ways(n) = ways(n-1) + ways(n-2) — arrive from one step back or two |

### Core Pattern

> Whenever the answer to step `n` depends on answers to `n-1` and `n-2`, you're looking at Fibonacci-style DP. Replace the array with two variables.

### When to Use This Pattern

- Count distinct ways to reach a goal with fixed step sizes.
- Any DP recurrence that only looks back a constant number of steps.
- Related: House Robber, Decode Ways, Unique Paths.

