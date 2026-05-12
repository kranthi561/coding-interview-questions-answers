# Longest Palindromic Substring

---

1. What is Longest Palindromic Substring?

## What is Longest Palindromic Substring?

Given a string `s`, return the **longest palindromic substring** within it.

A **palindromic substring** is a contiguous sequence of characters that reads the same forward and backward.

**Example:**

```
Input:  s = "babad"
Output: "bab"  (or "aba" — both are valid)

Input:  s = "cbbd"
Output: "bb"

Input:  s = "a"
Output: "a"

Input:  s = "racecar"
Output: "racecar"
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Return the longest contiguous palindromic substring.
- If multiple substrings share the same maximum length, return any one.

### Non-Functional Requirements

| Concern         | Expectation                                        |
| --------------- | -------------------------------------------------- |
| **Scale**       | String length up to 1,000                          |
| **Latency**     | O(n²) expand-around-center; O(n) Manacher's        |
| **Space**       | O(1) for expand-around-center approach             |

### Clarifying Questions to Ask

- Can the string be empty? → Constraints say `s.length >= 1`.
- Return the substring or its indices? → The substring itself.
- If tie in length, which to return? → Any is accepted.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Brute Force (check all substrings)

|           | Complexity                                   |
| --------- | -------------------------------------------- |
| **Time**  | O(n³) — O(n²) substrings × O(n) palindrome check |
| **Space** | O(1)                                         |

### Approach 2: Expand Around Center (Practical Optimal)

|           | Complexity                                          |
| --------- | --------------------------------------------------- |
| **Time**  | O(n²) — 2n-1 centers × O(n) expansion each          |
| **Space** | O(1)                                                |

### Approach 3: Manacher's Algorithm

|           | Complexity                              |
| --------- | --------------------------------------- |
| **Time**  | O(n) — linear scan with precomputed radii |
| **Space** | O(n)                                    |

For interviews, Expand Around Center is the standard expected answer. Manacher's is a bonus.


---

4. Algorithm Choice and Why

## Algorithm: Expand Around Center

### Why This Algorithm?

| Option                    | Time   | Space  | Chosen?                                    |
| ------------------------- | ------ | ------ | ------------------------------------------ |
| Brute force               | O(n³)  | O(1)   | No — too slow                              |
| DP table                  | O(n²)  | O(n²)  | Valid but uses quadratic space              |
| **Expand around center**  | O(n²)  | **O(1)** | **Yes — simple, no extra space**         |
| Manacher's algorithm      | O(n)   | O(n)   | Optimal but complex; reserved for follow-up |

### Logic

Every palindrome has a **center** — either a single character (odd length) or between two characters (even length). There are `2n - 1` such centers for a string of length `n`.

For each center:
1. Start with `left = center`, `right = center` (odd) or `left = center`, `right = center + 1` (even).
2. Expand outward while `s[left] == s[right]` and pointers are in bounds.
3. The palindrome extends from `left + 1` to `right - 1` after the loop ends.
4. Track the longest found.


---

5. High-Level Design

## High-Level Design

### Components

1. **Center Iterator** — Iterates over all 2n-1 centers (each character + each gap).
2. **Expander** — Grows the window outward from a center as long as characters match.
3. **Best Tracker** — Compares the current palindrome length to the running maximum.

### Data Flow Diagram

```
s = "babad"
Indices: 0='b', 1='a', 2='b', 3='a', 4='d'

Centers and best expansions:
  center=0 (odd):  'b'               len=1
  center=0-1 (even): 'b','a' ≠       len=0
  center=1 (odd):  'a' → expand: 'b','b'? s[0]='b',s[2]='b' ✓ → "bab" len=3
  center=1-2 (even): 'a','b' ≠       len=0
  center=2 (odd):  'b' → expand: s[1]='a',s[3]='a' ✓ → "aba" len=3
  ...

Best: "bab" (or "aba"), length 3
```


---

6. Java Solution

## Java Implementation

```java
public class LongestPalindromicSubstring {

    private String s;
    private int bestStart, bestLen;

    /**
     * Returns the longest palindromic substring in s.
     *
     * For each of the 2n-1 centers, expand outward while characters match.
     * Track the start index and length of the longest palindrome found.
     *
     * @param s input string
     * @return longest palindromic substring
     */
    public String longestPalindrome(String s) {
        this.s = s;
        bestStart = 0;
        bestLen = 1;

        for (int i = 0; i < s.length(); i++) {
            expand(i, i);     // odd-length palindromes (single character center)
            expand(i, i + 1); // even-length palindromes (gap center)
        }

        return s.substring(bestStart, bestStart + bestLen);
    }

    /**
     * Expands outward from (left, right) as long as characters match.
     * Updates bestStart and bestLen if a longer palindrome is found.
     */
    private void expand(int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }
        // After loop: s[left+1..right-1] is the palindrome
        int len = right - left - 1;
        if (len > bestLen) {
            bestLen = len;
            bestStart = left + 1;
        }
    }

    public static void main(String[] args) {
        LongestPalindromicSubstring solution = new LongestPalindromicSubstring();

        System.out.println(solution.longestPalindrome("babad"));   // "bab" or "aba"
        System.out.println(solution.longestPalindrome("cbbd"));    // "bb"
        System.out.println(solution.longestPalindrome("a"));       // "a"
        System.out.println(solution.longestPalindrome("racecar")); // "racecar"
        System.out.println(solution.longestPalindrome("ac"));      // "a" or "c"
    }
}
```

### Output

```
bab
bb
a
racecar
a
```


---

7. Python Solution

## Python Implementation

```python
def longest_palindrome(s: str) -> str:
    """
    Return the longest palindromic substring.

    Expand-around-center: for each of the 2n-1 possible centers,
    expand outward while characters match. Track the best window.

    Args:
        s: input string

    Returns:
        Longest palindromic substring

    Time:  O(n²)
    Space: O(1)
    """
    best_start, best_end = 0, 0

    def expand(left: int, right: int) -> None:
        nonlocal best_start, best_end
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        # palindrome is s[left+1 : right]
        if right - left - 1 > best_end - best_start:
            best_start = left + 1
            best_end = right  # exclusive

    for i in range(len(s)):
        expand(i, i)      # odd-length center
        expand(i, i + 1)  # even-length center

    return s[best_start:best_end]


# ---- Example usage ----
if __name__ == "__main__":
    print(longest_palindrome("babad"))   # "bab" or "aba"
    print(longest_palindrome("cbbd"))    # "bb"
    print(longest_palindrome("a"))       # "a"
    print(longest_palindrome("racecar")) # "racecar"
    print(longest_palindrome("ac"))      # "a"
```

### Output

```
bab
bb
a
racecar
a
```


---

8. Interview Q&A

## Q&A

**Q: Why are there 2n-1 centers?**  
A: Each character is a center for odd-length palindromes (n centers). Each gap between adjacent characters is a center for even-length palindromes (n-1 gaps). Total: 2n-1.

**Q: After the expand loop, why is the palindrome `s[left+1 : right-1]`?**  
A: The loop exits when `s[left] != s[right]` or a pointer goes out of bounds — meaning the last valid expansion was one step before. So `left+1` and `right-1` are the actual palindrome boundaries.

**Q: What is Manacher's algorithm and when would you use it?**  
A: Manacher's achieves O(n) by reusing palindrome radius information from previously computed centers — similar to KMP's failure function. Use it in production code for large strings, but in an interview, Expand Around Center is the expected answer unless O(n) is explicitly required.

**Q: How does the DP approach work?**  
A: `dp[i][j] = true` if `s[i..j]` is a palindrome. Base cases: single chars are true; adjacent equal chars are true. Transition: `dp[i][j] = (s[i] == s[j]) && dp[i+1][j-1]`. O(n²) time and space.

**Q: Can you find all palindromic substrings with this approach?**  
A: Yes — instead of tracking only the maximum, collect all palindromes found during expansion. This is the basis for "Palindromic Substrings" (count version).


---

9. Summary

## Summary

| Attribute             | Value                                                            |
| --------------------- | ---------------------------------------------------------------- |
| **Problem**           | Find the longest palindromic substring                           |
| **Optimal Algorithm** | Expand around center (2n-1 centers)                              |
| **Time Complexity**   | O(n²)                                                            |
| **Space Complexity**  | O(1)                                                             |
| **Key Insight**       | Every palindrome has a center; expand outward to find its radius |

### Core Pattern

> Center-expansion: enumerate all possible palindrome centers and grow each one outward until a mismatch. Track the widest expansion.

### When to Use This Pattern

- Finding or counting palindromic substrings.
- Any problem where a structure's extent must be measured from its center outward.

