# Sum of Two Integers

---

1. What is meant by Sum of Two Integers?

## What is Sum of Two Integers?

Given two integers `a` and `b`, return their **sum without using the `+` or `-` operators**.

You must implement addition using only **bitwise operations**.

**Example:**
```
Input:  a = 1, b = 2
Output: 3

Input:  a = 2, b = 3
Output: 5

Input:  a = -1, b = 1
Output: 0
```

**Why is this interesting?**
At the hardware level, addition is implemented with logic gates using XOR and AND operations. This problem asks you to simulate that directly.


---

2. Clarify Requirements

## Requirements

### Functional Requirements
- Add two integers `a` and `b` and return the result.
- Cannot use `+`, `-`, `*`, or `/` operators.
- Must handle negative integers (two's complement representation).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Works on any 32-bit signed integers |
| **Correctness** | Must handle overflow, negatives, zeros |
| **Operators** | Only bitwise operations allowed |

### Clarifying Questions to Ask
- Are the inputs 32-bit integers? → Yes (standard Java `int` / Python `int`)
- Can either input be negative? → Yes
- Can we use multiplication or division? → No, only bitwise operations


---

3. Time and Space Complexity

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| Built-in `+` (not allowed) | O(1) | O(1) |
| **Bitwise XOR + AND loop** | **O(1)** | **O(1)** |

**Why O(1)?**
For 32-bit integers, the carry propagation loop runs at most 32 iterations — a fixed constant regardless of input values.

**Space:** Only two variables (`sum`, `carry`) are needed.


---

4. Algorithm Choice and Why

## Algorithm: Bitwise XOR and AND

### How Binary Addition Works in Hardware

Addition of two bits has two outputs:
| Operation | Result |
|-----------|--------|
| `a XOR b` | Sum bit (without carry) |
| `(a AND b) << 1` | Carry bit (shifted left to next position) |

### Iterative Carry Propagation

```
Repeat until carry == 0:
    carry = (a AND b) << 1     ← carry produced at each bit position
    a     = a XOR b            ← partial sum without carry
    b     = carry              ← carry becomes new addend

Return a
```

### Example: 5 + 3 = 8
```
a = 0101 (5)
b = 0011 (3)

Iter 1:
  carry = (0101 AND 0011) << 1 = 0001 << 1 = 0010
  a     = 0101 XOR 0011        = 0110
  b     = 0010

Iter 2:
  carry = (0110 AND 0010) << 1 = 0010 << 1 = 0100
  a     = 0110 XOR 0010        = 0100
  b     = 0100

Iter 3:
  carry = (0100 AND 0100) << 1 = 0100 << 1 = 1000
  a     = 0100 XOR 0100        = 0000
  b     = 1000

Iter 4:
  carry = (0000 AND 1000) << 1 = 0000
  a     = 0000 XOR 1000        = 1000
  b     = 0000  ← carry is 0, stop

Result: a = 1000 = 8 ✓
```

### Why Not Other Approaches?
- Counting increments one by one would use `+1` implicitly.
- String conversion doesn't meet the bitwise requirement.
- This XOR/AND loop is the exact logic used inside CPUs for addition.


---

5. High-Level Design

## High-Level Design

### Components
1. **XOR Unit** — Computes the sum bits (no carry): `a XOR b`
2. **AND + Shift Unit** — Computes the carry bits: `(a AND b) << 1`
3. **Iteration Controller** — Repeats until carry = 0
4. **Output** — Final value of `a` is the result

### Data Flow Diagram
```
Input: a, b

┌─────────────────────────────────┐
│  LOOP while b != 0:             │
│                                 │
│  carry ← (a AND b) << 1        │
│                                 │
│  a     ← a XOR b  (partial sum)│
│                                 │
│  b     ← carry                  │
└─────────────────────────────────┘

Output: a
```

### Bit-Level Truth Table for One Position
```
a bit | b bit | XOR (sum) | AND (carry-out)
  0   |   0   |     0     |       0
  0   |   1   |     1     |       0
  1   |   0   |     1     |       0
  1   |   1   |     0     |       1  ← carry into next bit
```


---

6. Java Solution with Explanation

## Java Implementation

```java
public class SumOfTwoIntegers {

    /**
     * Returns the sum of a and b without using + or - operators.
     *
     * Uses bitwise XOR for sum bits and AND-with-shift for carry bits.
     * Java int is 32-bit two's complement, so negatives are handled correctly.
     *
     * @param a first integer
     * @param b second integer
     * @return a + b computed via bit manipulation
     */
    public int getSum(int a, int b) {
        while (b != 0) {
            int carry = (a & b) << 1;  // carry bits shifted left
            a = a ^ b;                  // sum bits without carry
            b = carry;                  // carry becomes the new addend
        }
        return a;
    }

    public static void main(String[] args) {
        SumOfTwoIntegers solution = new SumOfTwoIntegers();

        // Example 1: positive + positive
        System.out.println("1 + 2 = " + solution.getSum(1, 2));    // 3

        // Example 2: larger numbers
        System.out.println("5 + 3 = " + solution.getSum(5, 3));    // 8

        // Example 3: negative number
        System.out.println("-1 + 1 = " + solution.getSum(-1, 1));  // 0

        // Example 4: both negative
        System.out.println("-2 + -3 = " + solution.getSum(-2, -3)); // -5

        // Example 5: zero
        System.out.println("0 + 5 = " + solution.getSum(0, 5));    // 5
    }
}
```

### Output
```
1 + 2 = 3
5 + 3 = 8
-1 + 1 = 0
-2 + -3 = -5
0 + 5 = 5
```

### Why Java Handles Negatives Correctly
Java `int` uses **two's complement** representation. The XOR/AND bit operations work identically for negative numbers — the sign bit is just another bit in the computation.


---

7. Python Solution with Explanation

## Python Implementation

```python
def get_sum(a: int, b: int) -> int:
    """
    Return a + b without using + or - operators.

    Uses XOR for sum bits and AND-with-shift for carry.
    Python integers are arbitrary precision (not 32-bit), so we must
    manually mask to 32 bits to simulate overflow behavior.

    Time:  O(1) — at most 32 iterations for 32-bit integers
    Space: O(1)
    """
    MASK = 0xFFFFFFFF    # 32-bit mask
    MAX  = 0x7FFFFFFF    # max positive 32-bit int

    while b & MASK:
        carry = ((a & b) << 1) & MASK
        a = (a ^ b) & MASK
        b = carry

    # If result fits in positive 32-bit int, return as-is.
    # Otherwise convert from unsigned 32-bit to Python negative int.
    return a if a <= MAX else ~(a ^ MASK)


if __name__ == "__main__":
    print(f"1 + 2   = {get_sum(1, 2)}")     # 3
    print(f"5 + 3   = {get_sum(5, 3)}")     # 8
    print(f"-1 + 1  = {get_sum(-1, 1)}")    # 0
    print(f"-2 + -3 = {get_sum(-2, -3)}")   # -5
    print(f"0 + 5   = {get_sum(0, 5)}")     # 5
```

### Output
```
1 + 2   = 3
5 + 3   = 8
-1 + 1  = 0
-2 + -3 = -5
0 + 5   = 5
```

### Why Python Needs a Mask
Python integers have **arbitrary precision** — they don't overflow at 32 bits. Without masking, left shifts on negative numbers grow infinitely. The `MASK = 0xFFFFFFFF` keeps values within 32 bits. After the loop, `~(a ^ MASK)` converts an unsigned 32-bit value back to a Python signed negative integer.


---

8. Interview Q&A

## Q&A

**Q: Why does XOR compute sum bits?**
A: XOR is exactly the single-bit addition truth table: `0+0=0`, `0+1=1`, `1+0=1`, `1+1=0 (with carry)`. It gives the sum without the carry.

**Q: Why does AND + left shift compute carry bits?**
A: A carry is produced only when both bits are 1 (`1 AND 1 = 1`). The carry feeds into the **next higher** bit position, hence the left shift by 1.

**Q: How many iterations does the loop run?**
A: At most 32 for 32-bit integers. Each iteration resolves at least one carry bit, and there are at most 32 bit positions.

**Q: How does this handle negative numbers?**
A: Java uses two's complement, where the sign bit is treated like any other bit. The XOR/AND logic is identical — no special case needed.

**Q: Why does Python need a 32-bit mask?**
A: Python integers are arbitrary precision. Without masking, the left shift in `(a & b) << 1` can grow endlessly for large values. The mask clamps results to 32 bits to simulate hardware behavior.

**Q: Can you implement subtraction similarly?**
A: Yes. `a - b` = `a + (-b)` = `a + (~b + 1)`. You negate `b` using bitwise NOT plus 1 (two's complement negation), then add.


---

9. Summary

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Add two integers without `+` or `-` |
| **Optimal Algorithm** | Bitwise XOR (sum) + AND-shift (carry) loop |
| **Time Complexity** | O(1) — at most 32 iterations |
| **Space Complexity** | O(1) |
| **Key Insight** | XOR = sum without carry; AND << 1 = carry; repeat until carry is zero |

### Core Pattern
> Simulate hardware full-adder logic: XOR for partial sum, AND with left shift for carry propagation, iterate until no carry remains.

### When to Use This Pattern
- Any problem requiring arithmetic without arithmetic operators
- Understanding low-level CPU addition
- Implementing addition in restricted environments (embedded, HDL logic)

