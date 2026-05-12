# Counting Bits

---

1. What is meant by Counting Bits?

## What is Counting Bits?

Given an integer `n`, return an array `ans` of length `n + 1` such that for each `i` (0 ≤ i ≤ n), `ans[i]` is the **number of 1 bits** in the binary representation of `i`.

**Example:**
```
Input:  n = 2
Output: [0, 1, 1]
Explanation:
  0 → 0   → 0 ones
  1 → 1   → 1 one
  2 → 10  → 1 one

Input:  n = 5
Output: [0, 1, 1, 2, 1, 2]
Explanation:
  0 → 000 → 0 ones
  1 → 001 → 1 one
  2 → 010 → 1 one
  3 → 011 → 2 ones
  4 → 100 → 1 one
  5 → 101 → 2 ones
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements
- Return an array of length `n + 1`.
- Each element `ans[i]` = number of 1 bits in `i`.
- Index 0 is always 0 (zero has no set bits).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | `n` up to 10⁵ |
| **Time** | O(n) — single pass, no extra per-number work |
| **Space** | O(n) — the output array itself |

### Clarifying Questions to Ask
- Should I return counts or bit strings? → Integer counts.
- Is 0 always included? → Yes, output has n+1 elements from index 0 to n.
- Must it be O(n) or is O(n log n) acceptable? → O(n) is the follow-up challenge.


---

3. Time and Space Complexity

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Naive: call `hammingWeight(i)` for each i | O(n log n) | O(n) |
| **DP with right shift** | **O(n)** | **O(n)** |
| **DP with lowest set bit** | **O(n)** | **O(n)** |

**Why O(n)?** The DP recurrence computes each `ans[i]` in O(1) using a previously computed value — no per-number loop needed.


---

4. Algorithm Choice and Why

## Algorithm: Dynamic Programming with Bit Relationship

### Key Insight: Right Shift Relationship
Every number `i` is related to `i >> 1` (i divided by 2):
- `i >> 1` drops the last bit of `i`
- The last bit is either 0 or 1 (captured by `i & 1`)

**Recurrence:**
```
ans[i] = ans[i >> 1] + (i & 1)
```

| i | i >> 1 | i & 1 | ans[i] |
|---|--------|-------|--------|
| 0 | 0      | 0     | 0      |
| 1 | 0      | 1     | 1      |
| 2 | 1      | 0     | 1      |
| 3 | 1      | 1     | 2      |
| 4 | 2      | 0     | 1      |
| 5 | 2      | 1     | 2      |
| 6 | 3      | 0     | 2      |
| 7 | 3      | 1     | 3      |

### Alternative: Lowest Set Bit Relationship
Using `n & (n-1)` clears the lowest set bit:
```
ans[i] = ans[i & (i-1)] + 1
```

### Why DP?
The problem has **overlapping subproblems** — the bit count of `i` depends on the bit count of a smaller number. Memoizing these in an array makes each step O(1).


---

5. High-Level Design

## High-Level Design

### Components
1. **DP Array** — `ans[0..n]`, initialised to zero
2. **Recurrence Engine** — computes `ans[i] = ans[i >> 1] + (i & 1)` for each i
3. **Output** — the fully populated array

### Data Flow Diagram
```
n = 5

ans = [0, 0, 0, 0, 0, 0]  ← initialise

i=1: ans[1] = ans[0] + (1&1) = 0 + 1 = 1  → [0,1,0,0,0,0]
i=2: ans[2] = ans[1] + (2&1) = 1 + 0 = 1  → [0,1,1,0,0,0]
i=3: ans[3] = ans[1] + (3&1) = 1 + 1 = 2  → [0,1,1,2,0,0]
i=4: ans[4] = ans[2] + (4&1) = 1 + 0 = 1  → [0,1,1,2,1,0]
i=5: ans[5] = ans[2] + (5&1) = 1 + 1 = 2  → [0,1,1,2,1,2]

Result: [0, 1, 1, 2, 1, 2]
```

### Why Right Shift Works
```
i = 6  →  110 in binary
i >> 1 = 3  →  11 in binary  (same bits, minus last)
i & 1  = 0  (last bit is 0)

ans[6] = ans[3] + 0 = 2 + 0 = 2  ✓  (110 has two 1-bits)
```


---

6. Java Solution with Explanation

## Java Implementation

```java
public class CountingBits {

    /**
     * Returns array ans where ans[i] = number of 1-bits in i, for 0 <= i <= n.
     *
     * DP recurrence: ans[i] = ans[i >> 1] + (i & 1)
     *   ans[i >> 1] = count of 1-bits in i without its last bit
     *   (i & 1)     = whether the last bit is set
     *
     * @param n upper bound (inclusive)
     * @return bit count array of length n+1
     */
    public int[] countBits(int n) {
        int[] ans = new int[n + 1];
        // ans[0] = 0 by default (0 has no set bits)

        for (int i = 1; i <= n; i++) {
            ans[i] = ans[i >> 1] + (i & 1);
        }

        return ans;
    }

    /**
     * Alternative using lowest-set-bit relationship:
     * ans[i] = ans[i & (i-1)] + 1
     */
    public int[] countBitsAlt(int n) {
        int[] ans = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            ans[i] = ans[i & (i - 1)] + 1;
        }
        return ans;
    }

    public static void main(String[] args) {
        CountingBits solution = new CountingBits();

        // Example 1: n=2
        int[] result1 = solution.countBits(2);
        System.out.print("n=2: ");
        for (int v : result1) System.out.print(v + " ");  // 0 1 1
        System.out.println();

        // Example 2: n=5
        int[] result2 = solution.countBits(5);
        System.out.print("n=5: ");
        for (int v : result2) System.out.print(v + " ");  // 0 1 1 2 1 2
        System.out.println();

        // Example 3: n=8
        int[] result3 = solution.countBits(8);
        System.out.print("n=8: ");
        for (int v : result3) System.out.print(v + " ");  // 0 1 1 2 1 2 2 3 1
        System.out.println();
    }
}
```

### Output
```
n=2: 0 1 1
n=5: 0 1 1 2 1 2
n=8: 0 1 1 2 1 2 2 3 1
```


---

7. Python Solution with Explanation

## Python Implementation

```python
def count_bits(n: int) -> list[int]:
    """
    Return array where ans[i] = number of 1-bits in i, for 0 <= i <= n.

    DP recurrence: ans[i] = ans[i >> 1] + (i & 1)
    - i >> 1 is i with the last bit removed (already computed)
    - i & 1 is whether the last bit is 1

    Time:  O(n) — each index computed in O(1)
    Space: O(n) — output array
    """
    ans = [0] * (n + 1)

    for i in range(1, n + 1):
        ans[i] = ans[i >> 1] + (i & 1)

    return ans


def count_bits_alt(n: int) -> list[int]:
    """Alternative using n & (n-1) lowest-bit-clear trick."""
    ans = [0] * (n + 1)
    for i in range(1, n + 1):
        ans[i] = ans[i & (i - 1)] + 1
    return ans


if __name__ == "__main__":
    print(f"n=2: {count_bits(2)}")   # [0, 1, 1]
    print(f"n=5: {count_bits(5)}")   # [0, 1, 1, 2, 1, 2]
    print(f"n=8: {count_bits(8)}")   # [0, 1, 1, 2, 1, 2, 2, 3, 1]

    # Verify alternative
    print(f"\nAlt n=5: {count_bits_alt(5)}")  # [0, 1, 1, 2, 1, 2]
```

### Output
```
n=2: [0, 1, 1]
n=5: [0, 1, 1, 2, 1, 2]
n=8: [0, 1, 1, 2, 1, 2, 2, 3, 1]

Alt n=5: [0, 1, 1, 2, 1, 2]
```


---

8. Interview Q&A

## Q&A

**Q: Why is `ans[i] = ans[i >> 1] + (i & 1)` correct?**
A: Shifting `i` right by 1 removes its last bit. The bit count of `i >> 1` is already in the array. Adding `i & 1` accounts for whether the last bit was set — giving the full count for `i`.

**Q: What's the difference between the two DP recurrences?**
A: `ans[i >> 1] + (i & 1)` uses the "halved number" relationship. `ans[i & (i-1)] + 1` uses the "clear lowest set bit" relationship. Both are O(n) and O(1) per step; the shift approach is slightly more intuitive.

**Q: How is this different from calling `hammingWeight` for each number?**
A: Calling `hammingWeight` per number is O(log n) per call (up to 32 iterations), giving O(n log n) total. The DP approach reuses prior results, reducing each step to O(1) — total O(n).

**Q: Can you do it without extra space (beyond the output)?**
A: The output array itself is O(n) and unavoidable. The only extra space is O(1) for loop variables — already achieved.

**Q: Does the pattern `[0,1,1,2,1,2,2,3,1,2,...]` have a name?**
A: Yes — this is the **Stern–Brocot sequence** variant related to Hamming weight. The sequence for powers of 2 always has exactly one 1-bit, and numbers between powers of 2 build on smaller values.


---

9. Summary

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Count 1-bits for every number from 0 to n |
| **Optimal Algorithm** | DP with right-shift recurrence |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(n) — output array only |
| **Key Insight** | `ans[i] = ans[i >> 1] + (i & 1)` — reuse the count of the halved number |

### Core Pattern
> Recognise that the bit count of `i` depends on `i >> 1` (a previously solved subproblem). This transforms an O(n log n) approach into O(n) via DP.

### Related Problems
- Number of 1 Bits — counts bits in a single number
- Hamming Distance — XOR two numbers and count 1-bits
- Power of Two — `n > 0 && (n & (n-1)) == 0`

