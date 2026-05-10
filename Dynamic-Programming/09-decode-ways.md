# Decode Ways

---

<details>
<summary><strong>1. What is meant by Decode Ways?</strong></summary>

## What is Decode Ways?

**Decode Ways** is a DP problem where:

- A message is encoded as a digit string where 'A'=1, 'B'=2, ..., 'Z'=26.
- Each digit or pair of digits can represent a letter.

**Goal:** Return the **total number of ways** to decode the encoded string.

**Constraints:**
- `1 <= s.length <= 100`
- `s` contains only digits.
- `s` may contain leading zeros.

**Example:**

```
Input:  s = "226"
Output: 3
Explanation:
  "2 2 6" → BBA
  "22 6"  → VF
  "2 26"  → BZ
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Count all valid ways to decode the digit string.
- A single digit decodes to 'A' (1) to 'I' (9) — '0' alone is invalid.
- A two-digit pair decodes to 'J' (10) to 'Z' (26) — must be in range [10, 26].
- Leading zeros in a grouping are invalid (e.g., "06" is invalid).

### Non-Functional Requirements

| Concern         | Expectation                                      |
| --------------- | ------------------------------------------------ |
| **Scale**       | n up to 100 — O(n) is trivially fast             |
| **Latency**     | O(n) — single pass                               |
| **Correctness** | Must handle '0' carefully (can't decode alone)   |
| **Edge Cases**  | Starts with '0', contains "00", all same digits  |

### Clarifying Questions to Ask

- What if the string starts with '0'? → Return 0 (invalid encoding).
- Can "00" appear? → It can appear in the string but is always 0 decodings.
- Are numbers beyond 26 valid two-digit codes? → No, only 10–26.
- Is the answer always a valid integer? → Yes, fits in 32-bit int for n ≤ 100.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force Recursion

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — at each position, try 1 or 2 digits |
| **Space** | O(n) — call stack |

### Approach 2: Top-Down DP (Memoization)

| | Complexity |
| --- | --- |
| **Time** | O(n) — each index computed once |
| **Space** | O(n) — memo array + call stack |

### Approach 3: Bottom-Up DP (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n) — single pass |
| **Space** | O(n) — dp array |

### Approach 4: Space-Optimized (Two Variables)

| | Complexity |
| --- | --- |
| **Time** | O(n) |
| **Space** | O(1) — only prev and curr |

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Bottom-Up Dynamic Programming

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2ⁿ) | O(n) | No |
| Top-Down Memo | O(n) | O(n) | Equivalent |
| **Bottom-Up DP** | **O(n)** | **O(n)** | **Yes — standard** |

### Key Recurrence

`dp[i]` = number of ways to decode the first `i` characters `s[0..i-1]`.

```
dp[0] = 1  (empty string: one way to decode nothing)
dp[1] = 0 if s[0]=='0' else 1

For i from 2 to n:
    oneDigit = int(s[i-1])
    twoDigit = int(s[i-2:i])

    if oneDigit != 0:
        dp[i] += dp[i-1]   // s[i-1] decodes as a single character

    if 10 <= twoDigit <= 26:
        dp[i] += dp[i-2]   // s[i-2..i-1] decodes as a two-character code
```

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Input Validator** — Early return if `s[0] == '0'`
2. **DP Table Builder** — Array `dp[0..n]` filled iteratively
3. **One-Digit Checker** — Adds `dp[i-1]` if current digit is non-zero
4. **Two-Digit Checker** — Adds `dp[i-2]` if two-digit value is in [10, 26]
5. **Output Layer** — Returns `dp[n]`

### Data Flow Diagram

```
s = "226"

dp = [1, 1, 0, 0]  (initial: dp[0]=1, dp[1]=1 since s[0]='2'≠0)

i=2: oneDigit=2 (≠0) → dp[2] += dp[1] = 1
     twoDigit=22 ∈[10,26] → dp[2] += dp[0] = 1
     dp[2] = 2

i=3: oneDigit=6 (≠0) → dp[3] += dp[2] = 2
     twoDigit=26 ∈[10,26] → dp[3] += dp[1] = 1
     dp[3] = 3

Output: dp[3] = 3
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class DecodeWays {

    /**
     * Count the number of ways to decode a digit string.
     *
     * dp[i] = ways to decode s[0..i-1].
     * At each position, try decoding the last 1 digit and last 2 digits.
     * Add to dp[i] only when the digit(s) represent a valid letter (1-26, no leading zero).
     *
     * @param s encoded digit string
     * @return total number of valid decodings
     *
     * Time:  O(n)
     * Space: O(n)  — reducible to O(1) with two variables
     */
    public int numDecodings(String s) {
        int n = s.length();
        if (s.charAt(0) == '0') return 0; // no valid decoding

        int[] dp = new int[n + 1];
        dp[0] = 1; // empty prefix: one way (decode nothing)
        dp[1] = 1; // first char is non-zero (checked above)

        for (int i = 2; i <= n; i++) {
            int oneDigit = s.charAt(i - 1) - '0';        // current single digit
            int twoDigit = Integer.parseInt(s.substring(i - 2, i)); // last two digits

            // Single-digit decode: valid if non-zero (0 has no mapping)
            if (oneDigit != 0) {
                dp[i] += dp[i - 1];
            }

            // Two-digit decode: valid only in range [10, 26]
            if (twoDigit >= 10 && twoDigit <= 26) {
                dp[i] += dp[i - 2];
            }
        }

        return dp[n];
    }

    public static void main(String[] args) {
        DecodeWays sol = new DecodeWays();

        // Example 1: Multiple ways
        System.out.println("Input: \"226\"");
        System.out.println("Output: " + sol.numDecodings("226")); // 3

        // Example 2: Zero mid-string
        System.out.println("\nInput: \"12\"");
        System.out.println("Output: " + sol.numDecodings("12")); // 2 (AB or L)

        // Example 3: Starts with 0 → 0 ways
        System.out.println("\nInput: \"06\"");
        System.out.println("Output: " + sol.numDecodings("06")); // 0

        // Example 4: Single digit
        System.out.println("\nInput: \"7\"");
        System.out.println("Output: " + sol.numDecodings("7")); // 1 (G)

        // Example 5: Contains 10 (must be decoded as 'J')
        System.out.println("\nInput: \"10\"");
        System.out.println("Output: " + sol.numDecodings("10")); // 1

        // Example 6: "2101"
        System.out.println("\nInput: \"2101\"");
        System.out.println("Output: " + sol.numDecodings("2101")); // 1
    }
}
```

### Output

```
Input: "226"
Output: 3

Input: "12"
Output: 2

Input: "06"
Output: 0

Input: "7"
Output: 1

Input: "10"
Output: 1

Input: "2101"
Output: 1
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def num_decodings(s: str) -> int:
    """
    Count the number of ways to decode a digit string into letters (A=1..Z=26).

    Strategy: Bottom-up DP.
    dp[i] = ways to decode s[:i].
    At each step, try decoding the last 1 digit and last 2 digits.

    Args:
        s: Encoded string of digits

    Returns:
        Number of valid decodings (0 if none possible)

    Time:  O(n)
    Space: O(n) — reducible to O(1)
    """
    if s[0] == '0':
        return 0

    n = len(s)
    dp = [0] * (n + 1)
    dp[0] = 1  # empty prefix
    dp[1] = 1  # s[0] is non-zero (checked above)

    for i in range(2, n + 1):
        one_digit = int(s[i - 1])       # last single digit
        two_digit = int(s[i - 2:i])     # last two digits

        if one_digit != 0:              # non-zero → valid single-digit decode
            dp[i] += dp[i - 1]

        if 10 <= two_digit <= 26:       # valid two-digit decode (J..Z)
            dp[i] += dp[i - 2]

    return dp[n]


if __name__ == "__main__":
    test_cases = [
        ("226",  3),
        ("12",   2),
        ("06",   0),
        ("7",    1),
        ("10",   1),
        ("2101", 1),
        ("111",  3),
    ]

    for s, expected in test_cases:
        result = num_decodings(s)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] s={s!r} → Output: {result}, Expected: {expected}")
```

### Output

```
[PASS] s='226' → Output: 3, Expected: 3
[PASS] s='12' → Output: 2, Expected: 2
[PASS] s='06' → Output: 0, Expected: 0
[PASS] s='7' → Output: 1, Expected: 1
[PASS] s='10' → Output: 1, Expected: 1
[PASS] s='2101' → Output: 1, Expected: 1
[PASS] s='111' → Output: 3, Expected: 3
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why is `dp[0] = 1` when the empty string has "one decoding"?**
A: It's a base case trick. `dp[0]=1` means "there is one way to decode an empty prefix" — this ensures `dp[2]` accumulates correctly when a valid two-digit pair spans positions 0 and 1.

**Q: Why does '0' alone have no valid decoding?**
A: The mapping is A=1 to Z=26. There is no letter for 0. A '0' must always be paired with the digit before it (as '10' or '20') to be valid.

**Q: What happens with "00"?**
A: `dp[2]` gets: one_digit = 0 → skip; two_digit = 0 → not in [10,26] → skip. `dp[2] = 0`. Correctly returns 0 decodings.

**Q: How do you reduce space to O(1)?**
A: Since `dp[i]` only depends on `dp[i-1]` and `dp[i-2]`, use two variables `prev2` and `prev1` instead of the full array — same as Fibonacci optimization.

**Q: What's the maximum possible answer for n=100?**
A: For `s = "11111...1"` (100 ones), the answer follows the Fibonacci sequence, which grows exponentially — the result can exceed 64-bit int for large n, though constraints here keep n ≤ 100.

**Q: How does this relate to Climbing Stairs?**
A: The structure is identical: at each step, combine the count from one step back and two steps back. The difference is the validity checks (`digit != 0`, `10 <= twoDigit <= 26`) that can block accumulation.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                               |
| --------------------- | ------------------------------------------------------------------- |
| **Problem**           | Count all ways to decode a digit string into letters (A=1..Z=26)   |
| **Optimal Algorithm** | Bottom-Up Dynamic Programming                                       |
| **Time Complexity**   | O(n)                                                                |
| **Space Complexity**  | O(n) — reducible to O(1) with two variables                        |
| **Key Insight**       | At each position, try 1-digit and 2-digit decodings with validity checks |

### Core Pattern

> Like Climbing Stairs, but with conditions: single steps are valid only for non-zero digits, two-step jumps only for values in [10, 26]. Invalid steps contribute 0, effectively blocking those paths.

### When to Use This Pattern

- Counting arrangements with conditional step sizes.
- Related: Climbing Stairs, Word Break, Fibonacci variants, Number of Ways to Decode.

</details>
