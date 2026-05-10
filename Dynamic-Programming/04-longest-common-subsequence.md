# Longest Common Subsequence

---

<details>
<summary><strong>1. What is meant by Longest Common Subsequence?</strong></summary>

## What is Longest Common Subsequence (LCS)?

**LCS** is a classic DP problem where:

- You are given two strings `text1` and `text2`.
- A **common subsequence** is a sequence that appears in both strings (in order, but not necessarily contiguous).

**Goal:** Return the **length** of their longest common subsequence.

**Constraints:**
- `1 <= text1.length, text2.length <= 1000`
- Strings contain only lowercase English letters.

**Example:**

```
Input:  text1 = "abcde", text2 = "ace"
Output: 3
Explanation: LCS is "ace"
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Find the length of the longest subsequence common to both strings.
- Subsequences preserve relative order but can skip characters.
- No need to return the actual subsequence — just its length.

### Non-Functional Requirements

| Concern         | Expectation                                          |
| --------------- | ---------------------------------------------------- |
| **Scale**       | n, m up to 1000 → O(n×m) space and time acceptable  |
| **Latency**     | O(n×m) — runs in milliseconds                        |
| **Correctness** | Must handle all edge cases including no common chars |
| **Space**       | Can be optimized to O(min(n,m)) with rolling array   |

### Clarifying Questions to Ask

- Return length or actual subsequence? → Length only.
- Are strings case-sensitive? → Lowercase only per constraints.
- Can strings be empty? → Constraints say length ≥ 1; but handle defensively.
- Contiguous or non-contiguous? → Non-contiguous (that's Longest Common Substring for contiguous).

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force (All Subsequences)

| | Complexity |
| --- | --- |
| **Time** | O(2^m × n) — generate all subsequences of text1, check in text2 |
| **Space** | O(m) — recursion stack |

### Approach 2: Memoized Recursion (Top-Down DP)

| | Complexity |
| --- | --- |
| **Time** | O(n × m) — each (i,j) pair computed once |
| **Space** | O(n × m) — memoization table + call stack |

### Approach 3: Bottom-Up DP (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n × m) — fill the 2D table |
| **Space** | O(n × m) — 2D dp table |

### Approach 4: Space-Optimized (Two Rows)

| | Complexity |
| --- | --- |
| **Time** | O(n × m) |
| **Space** | O(min(n, m)) — only two rows needed |

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Bottom-Up 2D Dynamic Programming

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2^m × n) | O(m) | No |
| Top-Down Memo | O(n×m) | O(n×m) | Acceptable |
| **Bottom-Up DP** | **O(n×m)** | **O(n×m)** | **Yes — clear and standard** |
| Two-Row Optimized | O(n×m) | O(m) | Bonus optimization |

### Key Recurrence

Define `dp[i][j]` = LCS length of `text1[0..i-1]` and `text2[0..j-1]`:

```
if text1[i-1] == text2[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1      // characters match → extend LCS

else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])  // skip one character from either string
```

Base case: `dp[0][j] = dp[i][0] = 0` (empty string has LCS 0 with anything).

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Input Layer** — Receives `text1` and `text2`
2. **DP Table** — 2D array of size `(m+1) × (n+1)` initialized to 0
3. **Character Comparator** — Fills table cell by cell using the recurrence
4. **Output Layer** — Returns `dp[m][n]`

### Data Flow Diagram

```
text1 = "abcde"   (rows)
text2 = "ace"     (cols)

dp table:

     ""  a  c  e
  ""  0  0  0  0
  a   0  1  1  1
  b   0  1  1  1
  c   0  1  2  2
  d   0  1  2  2
  e   0  1  2  3

Output: dp[5][3] = 3  (LCS = "ace")
```

### Visual Logic

```
text1[i-1] == text2[j-1]:  dp[i][j] = dp[i-1][j-1] + 1  (diagonal + 1)
else:                       dp[i][j] = max(dp[i-1][j], dp[i][j-1])  (max of top and left)
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
public class LongestCommonSubsequence {

    /**
     * Find the length of the longest common subsequence of two strings.
     *
     * Bottom-up DP: dp[i][j] = LCS of text1[0..i-1] and text2[0..j-1].
     * Match → extend diagonal; mismatch → take best of top or left.
     *
     * @param text1 first string
     * @param text2 second string
     * @return length of LCS
     *
     * Time:  O(m × n)
     * Space: O(m × n)
     */
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1]; // dp[0][*] and dp[*][0] are 0 by default

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    // Characters match: extend the LCS from the diagonal
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    // Mismatch: take best by skipping one character from either string
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }

    // Reconstruct the actual LCS string (bonus)
    public String reconstructLCS(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (text1.charAt(i - 1) == text2.charAt(j - 1))
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                else
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);

        // Backtrack from dp[m][n]
        StringBuilder sb = new StringBuilder();
        int i = m, j = n;
        while (i > 0 && j > 0) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                sb.append(text1.charAt(i - 1));
                i--; j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                i--;
            } else {
                j--;
            }
        }
        return sb.reverse().toString();
    }

    public static void main(String[] args) {
        LongestCommonSubsequence sol = new LongestCommonSubsequence();

        // Example 1
        System.out.println("text1=abcde, text2=ace");
        System.out.println("LCS length: " + sol.longestCommonSubsequence("abcde", "ace")); // 3
        System.out.println("LCS string: " + sol.reconstructLCS("abcde", "ace")); // ace

        // Example 2
        System.out.println("\ntext1=abc, text2=abc");
        System.out.println("LCS length: " + sol.longestCommonSubsequence("abc", "abc")); // 3

        // Example 3: No common characters
        System.out.println("\ntext1=abc, text2=def");
        System.out.println("LCS length: " + sol.longestCommonSubsequence("abc", "def")); // 0

        // Example 4
        System.out.println("\ntext1=AGGTAB, text2=GXTXAYB");
        System.out.println("LCS length: " + sol.longestCommonSubsequence("AGGTAB", "GXTXAYB")); // 4
    }
}
```

### Output

```
text1=abcde, text2=ace
LCS length: 3
LCS string: ace

text1=abc, text2=abc
LCS length: 3

text1=abc, text2=def
LCS length: 0

text1=AGGTAB, text2=GXTXAYB
LCS length: 4
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def longest_common_subsequence(text1: str, text2: str) -> int:
    """
    Find the length of the longest common subsequence of two strings.

    Strategy: Bottom-up 2D DP.
    dp[i][j] = LCS length of text1[:i] and text2[:j].
    Match → dp[i-1][j-1] + 1 (diagonal).
    Mismatch → max(dp[i-1][j], dp[i][j-1]) (skip one char from either).

    Args:
        text1: First string
        text2: Second string

    Returns:
        Length of the longest common subsequence

    Time:  O(m × n)
    Space: O(m × n)
    """
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    return dp[m][n]


def reconstruct_lcs(text1: str, text2: str) -> str:
    """Reconstruct the actual LCS string by backtracking through the DP table."""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    # Backtrack
    result = []
    i, j = m, n
    while i > 0 and j > 0:
        if text1[i - 1] == text2[j - 1]:
            result.append(text1[i - 1])
            i -= 1; j -= 1
        elif dp[i - 1][j] > dp[i][j - 1]:
            i -= 1
        else:
            j -= 1

    return ''.join(reversed(result))


if __name__ == "__main__":
    cases = [
        ("abcde", "ace"),
        ("abc", "abc"),
        ("abc", "def"),
        ("AGGTAB", "GXTXAYB"),
    ]
    for t1, t2 in cases:
        length = longest_common_subsequence(t1, t2)
        lcs = reconstruct_lcs(t1, t2)
        print(f"text1={t1!r}, text2={t2!r}")
        print(f"  LCS length: {length}, LCS: {lcs!r}\n")
```

### Output

```
text1='abcde', text2='ace'
  LCS length: 3, LCS: 'ace'

text1='abc', text2='abc'
  LCS length: 3, LCS: 'abc'

text1='abc', text2='def'
  LCS length: 0, LCS: ''

text1='AGGTAB', text2='GXTXAYB'
  LCS length: 4, LCS: 'GTAB'
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: What is the difference between LCS and Longest Common Substring?**
A: LCS allows skipping characters (non-contiguous). Longest Common Substring requires the matching characters to be contiguous. LCS uses the recurrence above; Longest Common Substring resets to 0 on mismatch.

**Q: Why initialize dp[0][*] and dp[*][0] to 0?**
A: They represent the LCS of an empty string with any string — which is always 0. This is the base case that makes the recurrence work.

**Q: How do you reconstruct the actual LCS?**
A: Start at `dp[m][n]` and trace back: if characters match, go diagonal and collect the char; otherwise go in the direction of the larger value (top or left).

**Q: Can space be reduced below O(m×n)?**
A: Yes. Since each row only depends on the previous row, you can use two 1D arrays (current and previous row), reducing space to O(n). This is the "rolling array" optimization.

**Q: Is LCS related to Edit Distance?**
A: Yes. Edit Distance (Levenshtein) is a generalization. LCS can be used to compute the minimum edit distance: `edit_distance = m + n - 2 * LCS(text1, text2)` when only insertions and deletions are allowed.

**Q: What are real-world applications of LCS?**
A: Diff tools (git diff), bioinformatics (DNA sequence alignment), plagiarism detection, spell checkers.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                                   |
| --------------------- | ----------------------------------------------------------------------- |
| **Problem**           | Find length of longest subsequence common to two strings                |
| **Optimal Algorithm** | Bottom-Up 2D Dynamic Programming                                        |
| **Time Complexity**   | O(m × n)                                                                |
| **Space Complexity**  | O(m × n) — reducible to O(min(m,n)) with rolling array                 |
| **Key Insight**       | Match → diagonal+1; mismatch → max(top, left)                          |

### Core Pattern

> Use a 2D table where each cell answers "what's the LCS of these two prefixes?" Fill left-to-right, top-to-bottom. The answer lives in the bottom-right corner.

### When to Use This Pattern

- Any problem comparing two sequences character by character.
- Related: Edit Distance, Shortest Common Supersequence, Diff algorithms, Sequence Alignment.

</details>
