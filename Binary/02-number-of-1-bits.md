# Number of 1 Bits

---

<details>
<summary><strong>1. What is meant by Number of 1 Bits?</strong></summary>

## What is Number of 1 Bits?

Write a function that takes an unsigned integer and returns the number of **'1' bits** it has (also known as the **Hamming weight**).

**Example:**
```
Input:  n = 00000000000000000000000000001011  (decimal: 11)
Output: 3
Explanation: Three '1' bits in the binary representation.

Input:  n = 00000000000000000000000010000000  (decimal: 128)
Output: 1

Input:  n = 11111111111111111111111111111101  (decimal: 4294967293)
Output: 31
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Count the number of set bits (1s) in a 32-bit unsigned integer.
- Return the count as an integer.
- Input is always an unsigned 32-bit integer.

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Input** | 32-bit unsigned integer (0 to 2³²-1) |
| **Time** | O(1) — fixed 32-bit input size |
| **Space** | O(1) |

### Clarifying Questions to Ask
- Is it always a 32-bit input? → Yes.
- Can the input be negative (signed)? → Treated as unsigned 32-bit.
- Do we need the positions of the 1 bits or just the count? → Just the count.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Check each of 32 bits with `>> 1` | O(1) — 32 steps | O(1) | Simple, always 32 iterations |
| **Brian Kernighan's `n & (n-1)`** | **O(1) — k steps** | **O(1)** | Only iterates as many times as there are 1 bits |
| Built-in `Integer.bitCount()` | O(1) | O(1) | Library call |

**Why O(1)?** The input is always a fixed 32-bit integer, so all approaches run in constant time regardless of the value.

**Brian Kernighan is preferred** because it iterates only `k` times where `k` = number of set bits — faster for sparse inputs.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Brian Kernighan's Bit Trick — `n & (n-1)`

### The Key Insight
`n & (n - 1)` **clears the lowest set bit** of `n`.

**Why?** Subtracting 1 from `n` flips the lowest set bit to 0 and all lower bits to 1. AND-ing with the original `n` clears those positions.

### Example: n = 12 (1100)
```
n     = 1100  (12)
n - 1 = 1011  (11)
n & (n-1) = 1000  (8)  ← lowest set bit cleared

Iteration 1: n=1100 → n=1000  (count=1)
Iteration 2: n=1000 → n=0000  (count=2)
Loop ends: return 2
```

### Alternative: Shift and Check
```
For each of 32 bits:
    if (n & 1) == 1: count++
    n >>= 1
```
Always 32 iterations regardless of how many bits are set.

### Why Brian Kernighan?
For numbers with few 1 bits, it's faster — only `k` iterations instead of always 32.

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Components
1. **Bit Clearer** — `n & (n-1)` clears the rightmost set bit each iteration
2. **Counter** — increments on every successful bit clear
3. **Loop Controller** — runs until `n == 0`

### Data Flow Diagram
```
Input: n = 11  (binary: 1011)

Step 1: n=1011, n-1=1010 → n & (n-1) = 1010, count=1
Step 2: n=1010, n-1=1001 → n & (n-1) = 1000, count=2
Step 3: n=1000, n-1=0111 → n & (n-1) = 0000, count=3
Step 4: n=0000 → loop ends

Result: 3
```

### Visual Bit Clearing
```
1 0 1 1   (n = 11)
    ↑ ↑ ↑  ← three 1-bits
    │ │ └── cleared in step 1
    │ └──── cleared in step 2
    └────── cleared in step 3
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class NumberOf1Bits {

    /**
     * Returns the number of set bits (1s) in the binary representation of n.
     *
     * Uses Brian Kernighan's trick: n & (n-1) clears the lowest set bit.
     * Loop count equals exactly the number of 1 bits.
     *
     * @param n unsigned 32-bit integer
     * @return count of 1 bits (Hamming weight)
     */
    public int hammingWeight(int n) {
        int count = 0;
        while (n != 0) {
            n = n & (n - 1);  // clear the lowest set bit
            count++;
        }
        return count;
    }

    /**
     * Alternative: check each bit using right shift.
     * Always runs exactly 32 iterations.
     */
    public int hammingWeightShift(int n) {
        int count = 0;
        while (n != 0) {
            count += (n & 1);  // check last bit
            n >>>= 1;           // unsigned right shift (fills with 0, not sign bit)
        }
        return count;
    }

    public static void main(String[] args) {
        NumberOf1Bits solution = new NumberOf1Bits();

        // 11 in binary = 1011 → three 1-bits
        System.out.println("n=11:         " + solution.hammingWeight(11));   // 3

        // 128 in binary = 10000000 → one 1-bit
        System.out.println("n=128:        " + solution.hammingWeight(128));  // 1

        // All 1s except bit 1: 11111111111111111111111111111101 → 31 bits
        System.out.println("n=4294967293: " + solution.hammingWeight(-3));   // 31

        // Zero: no set bits
        System.out.println("n=0:          " + solution.hammingWeight(0));    // 0
    }
}
```

### Output
```
n=11:         3
n=128:        1
n=4294967293: 31
n=0:          0
```

> **Note:** Java's `int` is signed. For unsigned 32-bit behavior, use `>>>` (unsigned right shift) instead of `>>`. The `n & (n-1)` approach works correctly for both signed and unsigned inputs.

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def hamming_weight(n: int) -> int:
    """
    Count the number of 1-bits in the 32-bit unsigned integer n.

    Brian Kernighan's trick: n & (n - 1) clears the lowest set bit.
    Each iteration removes exactly one 1-bit; count iterations.

    Time:  O(1) — at most 32 iterations for 32-bit input
    Space: O(1)
    """
    count = 0
    while n:
        n &= n - 1  # clear lowest set bit
        count += 1
    return count


def hamming_weight_builtin(n: int) -> int:
    """Pythonic one-liner using bin() string counting."""
    return bin(n).count('1')


if __name__ == "__main__":
    print(f"n=11:         {hamming_weight(11)}")          # 3
    print(f"n=128:        {hamming_weight(128)}")         # 1
    print(f"n=4294967293: {hamming_weight(4294967293)}")  # 31
    print(f"n=0:          {hamming_weight(0)}")           # 0

    # Verify with built-in
    print(f"\nBuilt-in: n=11 → {hamming_weight_builtin(11)}")  # 3
```

### Output
```
n=11:         3
n=128:        1
n=4294967293: 31
n=0:          0

Built-in: n=11 → 3
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why does `n & (n-1)` clear the lowest set bit?**
A: Subtracting 1 from `n` converts the lowest set bit to 0 and all bits below it to 1. AND-ing the original `n` (where those lower bits are 0) with `n-1` clears exactly the lowest set bit and leaves everything else unchanged.

**Q: What is the difference between `>>` and `>>>` in Java?**
A: `>>` is arithmetic right shift (preserves the sign bit). `>>>` is logical right shift (fills with 0). For unsigned bit counting, `>>>` is correct — `>>` on a negative number would extend 1s infinitely.

**Q: What is the Hamming weight used for in real life?**
A: Hamming weight (popcount) is used in error correction codes, cryptography, approximate string matching (Hamming distance), and population count instructions in CPUs (`POPCNT`).

**Q: Can you do this in O(1) even more efficiently?**
A: Yes — using a **lookup table** approach: split the 32-bit number into four 8-bit chunks, precompute counts for all 256 values, and sum four table lookups. This avoids any loop entirely.

**Q: What's the built-in way in Java and Python?**
A: Java: `Integer.bitCount(n)`. Python: `bin(n).count('1')` or `n.bit_count()` (Python 3.10+).

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Count set bits (1s) in a 32-bit integer |
| **Optimal Algorithm** | Brian Kernighan's `n & (n-1)` trick |
| **Time Complexity** | O(1) — at most k iterations (k = number of 1-bits) |
| **Space Complexity** | O(1) |
| **Key Insight** | `n & (n-1)` removes the lowest set bit — count iterations until n is zero |

### Core Pattern
> Use `n & (n-1)` whenever you need to clear or count set bits. This is one of the most fundamental bit manipulation tricks.

### When to Use This Pattern
- Counting set bits (Hamming weight / popcount)
- Checking if a number is a power of 2: `n > 0 && (n & (n-1)) == 0`
- Finding the lowest set bit: `n & (-n)`

</details>
