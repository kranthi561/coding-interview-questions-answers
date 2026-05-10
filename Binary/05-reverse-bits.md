# Reverse Bits

---

<details>
<summary><strong>1. What is meant by Reverse Bits?</strong></summary>

## What is Reverse Bits?

Reverse the **bits** of a given 32-bit unsigned integer.

**Example:**
```
Input:  n = 00000010100101000001111010011100
Output:    00111001011110000010100101000000
Decimal: 43261596 → 964176192

Input:  n = 11111111111111111111111111111101
Output:    10111111111111111111111111111111
Decimal: 4294967293 → 3221225471
```

> This reverses the **binary representation** bit by bit — not the decimal digits.

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Reverse all 32 bits of the input unsigned integer.
- The result must also be a 32-bit unsigned integer.
- Pad with leading zeros if necessary (input is always treated as exactly 32 bits).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Input** | 32-bit unsigned integer |
| **Output** | 32-bit unsigned integer |
| **Time** | O(1) — fixed 32 bits |
| **Space** | O(1) |

### Clarifying Questions to Ask
- Is the input always 32 bits? → Yes, treat as unsigned 32-bit.
- Should leading zeros be preserved? → Yes — reversing `1` (00..001) gives `10..000` (2147483648).
- What if called many times? → Can optimise with a cache (byte-level reverse lookup table).

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| **Bit-by-bit loop (32 iterations)** | **O(1)** | **O(1)** | Simple, direct |
| Divide and conquer (mask + shift) | O(1) | O(1) | Branchless, faster in practice |
| Lookup table (byte-level) | O(1) | O(256) | Best when called repeatedly |

All approaches are O(1) because the input is fixed at 32 bits.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Bit-by-Bit Reversal

### Core Idea
Extract bits from the right of `n` one at a time and build the reversed result from right to left:

```
For 32 iterations:
  1. Extract the rightmost bit of n:    bit = n & 1
  2. Append it to the LEFT of result:  result = (result << 1) | bit
  3. Move to the next bit:             n >>= 1
```

### Why Shift Left on Result?
Each new bit goes to the far left of the reversed number. Shifting `result` left by 1 makes room at the rightmost position for the current bit.

### Trace: Reversing 13 (1101) as 4-bit example
```
n      = 1101
result = 0000

Step 1: bit=1, result = 0001, n=0110
Step 2: bit=0, result = 0010, n=0011
Step 3: bit=1, result = 0101, n=0001
Step 4: bit=1, result = 1011, n=0000

Reversed: 1011 = 11  ✓  (1101 reversed is 1011)
```

### Alternative: Divide and Conquer (Swap Groups of Bits)
Swap adjacent bits, then pairs, then nibbles, then bytes — in 5 masking steps. This is branchless and very fast but harder to read.

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Components
1. **Bit Extractor** — `n & 1` pulls the rightmost bit of `n`
2. **Result Builder** — `result << 1 | bit` appends the bit on the right of result (which becomes the reversed left)
3. **Input Shifter** — `n >>>= 1` (unsigned) moves to the next bit
4. **Loop Controller** — runs exactly 32 times

### Data Flow (32-bit n = 43261596)
```
n      = 00000010100101000001111010011100
result = 00000000000000000000000000000000

Iteration 1:  bit=0, result=...0,  n shifts right
Iteration 2:  bit=0, result=...00, n shifts right
...
(Each step: right bit of n → left bit of result)
...
Iteration 32: result = 00111001011110000010100101000000
                     = 964176192
```

### Visual
```
n:      [b31][b30]...[b1][b0]   (read right-to-left)
result: [b0] [b1] ...[b30][b31] (written left-to-right)
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class ReverseBits {

    /**
     * Reverses the bits of a 32-bit unsigned integer.
     *
     * Extracts the rightmost bit of n and appends it to the
     * left of the result in each of 32 iterations.
     *
     * Uses >>> (unsigned right shift) to avoid sign extension.
     *
     * @param n 32-bit unsigned integer
     * @return n with all bits reversed
     */
    public int reverseBits(int n) {
        int result = 0;

        for (int i = 0; i < 32; i++) {
            result = (result << 1) | (n & 1);  // append last bit of n to result
            n >>>= 1;                            // unsigned shift: fill with 0
        }

        return result;
    }

    /**
     * Optimised divide-and-conquer approach.
     * Swaps: adjacent bits → pairs → nibbles → bytes → half-words.
     * Five mask-and-shift operations, no loop.
     */
    public int reverseBitsDivideConquer(int n) {
        n = (n >>> 16) | (n << 16);                          // swap halves
        n = ((n & 0xff00ff00) >>> 8)  | ((n & 0x00ff00ff) << 8);   // swap bytes
        n = ((n & 0xf0f0f0f0) >>> 4)  | ((n & 0x0f0f0f0f) << 4);   // swap nibbles
        n = ((n & 0xcccccccc) >>> 2)  | ((n & 0x33333333) << 2);   // swap bit pairs
        n = ((n & 0xaaaaaaaa) >>> 1)  | ((n & 0x55555555) << 1);   // swap bits
        return n;
    }

    public static void main(String[] args) {
        ReverseBits solution = new ReverseBits();

        // Example 1: 43261596
        int n1 = 43261596;
        System.out.printf("Input:  %d%n", n1);
        System.out.printf("Binary: %s%n", String.format("%32s", Integer.toBinaryString(n1)).replace(' ', '0'));
        int r1 = solution.reverseBits(n1);
        System.out.printf("Output: %d%n", r1);  // 964176192
        System.out.printf("Binary: %s%n%n", String.format("%32s", Integer.toBinaryString(r1)).replace(' ', '0'));

        // Example 2: 4294967293 (treated as -3 in signed Java)
        int n2 = 0xFFFFFFFD; // -3 in signed, 4294967293 unsigned
        System.out.printf("Input:  %d (unsigned)%n", Integer.toUnsignedLong(n2));
        int r2 = solution.reverseBits(n2);
        System.out.printf("Output: %d (unsigned)%n", Integer.toUnsignedLong(r2)); // 3221225471
    }
}
```

### Output
```
Input:  43261596
Binary: 00000010100101000001111010011100
Output: 964176192
Binary: 00111001011110000010100101000000

Input:  4294967293 (unsigned)
Output: 3221225471 (unsigned)
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def reverse_bits(n: int) -> int:
    """
    Reverse the 32 bits of an unsigned integer.

    Strategy: Extract bits from the right of n one at a time,
    appending each to the right of result (which fills from left).

    Time:  O(1) — always 32 iterations
    Space: O(1)
    """
    result = 0

    for _ in range(32):
        result = (result << 1) | (n & 1)  # shift result left, OR in last bit of n
        n >>= 1                            # move to next bit

    return result


def reverse_bits_builtin(n: int) -> int:
    """
    Python one-liner: convert to 32-bit binary string, reverse, convert back.
    """
    return int(format(n, '032b')[::-1], 2)


if __name__ == "__main__":
    # Example 1
    n1 = 43261596
    r1 = reverse_bits(n1)
    print(f"Input:  {n1}")
    print(f"Binary: {n1:032b}")
    print(f"Output: {r1}")       # 964176192
    print(f"Binary: {r1:032b}")
    print()

    # Example 2
    n2 = 4294967293
    r2 = reverse_bits(n2)
    print(f"Input:  {n2}")
    print(f"Output: {r2}")       # 3221225471

    # Builtin verification
    print(f"\nBuiltin: {reverse_bits_builtin(43261596)}")  # 964176192
```

### Output
```
Input:  43261596
Binary: 00000010100101000001111010011100
Output: 964176192
Binary: 00111001011110000010100101000000

Input:  4294967293
Output: 3221225471

Builtin: 964176192
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why use `>>>` in Java instead of `>>`?**
A: `>>` is arithmetic right shift — it fills the leftmost bit with the sign bit (1 for negative numbers), which would corrupt the bit pattern. `>>>` is logical right shift — it always fills with 0, preserving the unsigned interpretation.

**Q: Why does Python not need `>>>`?**
A: Python integers are arbitrary precision and signed. For right shifts of non-negative numbers, `>>` fills with 0. Since the 32-bit unsigned input is always non-negative, `n >>= 1` works correctly here.

**Q: How would you optimize for repeated calls (follow-up)?**
A: Use a **byte-level lookup table**: precompute `reversed_byte[0..255]`. For a 32-bit integer, reverse each byte and reassemble. This turns 32 iterations into 4 lookups.

**Q: What is the divide-and-conquer approach?**
A: Swap bit-level groups in 5 rounds: halves → bytes → nibbles → bit pairs → individual bits, each using a bitmask and shift. No loop, no branches — compiles to very fast code.

**Q: What is the real-world application of bit reversal?**
A: Bit reversal is used in **Fast Fourier Transform (FFT)** algorithms to reorder elements (bit-reversal permutation). It also appears in network byte-order conversion and certain hash functions.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Reverse all 32 bits of an unsigned integer |
| **Optimal Algorithm** | 32-iteration bit extraction loop |
| **Time Complexity** | O(1) — always exactly 32 steps |
| **Space Complexity** | O(1) |
| **Key Insight** | Extract rightmost bit of n, append to left of result; shift n right each step |

### Core Pattern
> Read bits from input right-to-left; write them to result left-to-right. Two shifts (input right, result left) per step.

### Comparison of Approaches
| Approach | Code Clarity | Speed | Best For |
|----------|-------------|-------|---------|
| 32-step loop | High | Good | Interviews, clarity |
| Divide & conquer | Low | Best | Performance-critical code |
| Lookup table | Medium | Excellent | Repeated calls |

</details>
