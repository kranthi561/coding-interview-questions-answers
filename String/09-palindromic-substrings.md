# Palindromic Substrings

---

1. What is Palindromic Substrings?

## What is Palindromic Substrings?

Given a string `s`, return the **total number of palindromic substrings** in it.

A **palindromic substring** is any contiguous sequence of characters that reads the same forward and backward. Single characters always count.

**Example:**

```
Input:  s = "abc"
Output: 3
Explanation: "a", "b", "c" are the three palindromic substrings.

Input:  s = "aaa"
Output: 6
Explanation: "a"(×3), "aa"(×2), "aaa"(×1) = 6 palindromic substrings.
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Count every distinct position-based palindromic substring (not unique strings — `"a"` at index 0 and `"a"` at index 2 count separately).
- Include single-character substrings (every character is a palindrome).

### Non-Functional Requirements

| Concern         | Expectation                                    |
| --------------- | ---------------------------------------------- |
| **Scale**       | String length up to 1,000                      |
| **Latency**     | O(n²) — expand-around-center approach          |
| **Space**       | O(1) — no extra structures needed              |

### Clarifying Questions to Ask

- Count positionally distinct occurrences or unique palindrome strings? → Positionally distinct (each position counts separately).
- Can the string be empty? → No, `s.length >= 1` per constraints.


---

3. Time and Space Complexity

## Complexity Analysis

### Expand Around Center (Optimal for interviews)

|           | Complexity                                          |
| --------- | --------------------------------------------------- |
| **Time**  | O(n²) — 2n-1 centers × O(n) expansion each          |
| **Space** | O(1)                                                |

### DP Approach

|           | Complexity                                                  |
| --------- | ----------------------------------------------------------- |
| **Time**  | O(n²)                                                       |
| **Space** | O(n²) — full 2D boolean table                               |

Expand around center is preferred — same time, far less space.


---

4. Algorithm Choice and Why

## Algorithm: Expand Around Center

### Why This Algorithm?

| Option                    | Time   | Space   | Chosen?                                  |
| ------------------------- | ------ | ------- | ---------------------------------------- |
| Brute force               | O(n³)  | O(1)    | No — too slow                            |
| DP table                  | O(n²)  | O(n²)   | Valid but wastes space                   |
| **Expand around center**  | O(n²)  | **O(1)** | **Yes — optimal space, same time**      |

### Logic

This is the same expand-around-center technique as Longest Palindromic Substring, but instead of tracking the maximum length, we **count every valid expansion step**.

For each of the `2n-1` centers:
1. Start at `left = center`, `right = center` (odd) or `left = center`, `right = center + 1` (even).
2. While `left >= 0` and `right < n` and `s[left] == s[right]`:
   - Increment the count by 1 (this expansion is a valid palindrome).
   - Expand outward: `left--`, `right++`.

Each successful expansion step is one palindromic substring.


---

5. High-Level Design

## High-Level Design

### Components

1. **Center Iterator** — Iterates over all 2n-1 centers.
2. **Expander-Counter** — Counts each valid outward expansion as one palindrome.
3. **Total Accumulator** — Sums all counts across all centers.

### Data Flow Diagram

```
s = "aaa"
Indices: 0='a', 1='a', 2='a'

Odd centers:
  center=0: expand(0,0): "a" ✓ count=1; s[-1] out of bounds → stop
  center=1: expand(1,1): "a" ✓ count=1; s[0]==s[2]='a' → "aaa" ✓ count=2; out of bounds → stop
  center=2: expand(2,2): "a" ✓ count=1; s[3] out of bounds → stop

Even centers:
  gap 0-1: expand(0,1): s[0]==s[1]='a' → "aa" ✓ count=1; out of bounds → stop
  gap 1-2: expand(1,2): s[1]==s[2]='a' → "aa" ✓ count=1; out of bounds → stop

Total = 1+2+1+1+1 = 6 ✓
```


---

6. Java Solution

## Java Implementation

```java
public class PalindromicSubstrings {

    /**
     * Counts all palindromic substrings in s.
     *
     * For each of the 2n-1 centers, expand outward while characters match.
     * Every successful expansion step is one palindromic substring.
     *
     * @param s input string
     * @return total count of palindromic substrings
     */
    public int countSubstrings(String s) {
        int count = 0;
        int n = s.length();

        for (int i = 0; i < n; i++) {
            count += expand(s, i, i);     // odd-length palindromes
            count += expand(s, i, i + 1); // even-length palindromes
        }

        return count;
    }

    /**
     * Expands from (left, right) and counts every valid palindrome found.
     */
    private int expand(String s, int left, int right) {
        int count = 0;
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            count++;
            left--;
            right++;
        }
        return count;
    }

    public static void main(String[] args) {
        PalindromicSubstrings solution = new PalindromicSubstrings();

        System.out.println(solution.countSubstrings("abc")); // 3
        System.out.println(solution.countSubstrings("aaa")); // 6
        System.out.println(solution.countSubstrings("a"));   // 1
        System.out.println(solution.countSubstrings("aba")); // 4  ("a","b","a","aba")
    }
}
```

### Output

```
3
6
1
4
```


---

7. Python Solution

## Python Implementation

```python
def count_substrings(s: str) -> int:
    """
    Count all palindromic substrings in s.

    Expand-around-center: for each of the 2n-1 centers, count every
    valid outward expansion as one palindromic substring.

    Args:
        s: input string

    Returns:
        Total number of palindromic substrings

    Time:  O(n²)
    Space: O(1)
    """
    def expand(left: int, right: int) -> int:
        count = 0
        while left >= 0 and right < len(s) and s[left] == s[right]:
            count += 1
            left -= 1
            right += 1
        return count

    total = 0
    for i in range(len(s)):
        total += expand(i, i)      # odd-length
        total += expand(i, i + 1)  # even-length
    return total


# ---- Example usage ----
if __name__ == "__main__":
    print(count_substrings("abc"))  # 3
    print(count_substrings("aaa"))  # 6
    print(count_substrings("a"))    # 1
    print(count_substrings("aba"))  # 4
```

### Output

```
3
6
1
4
```


---

8. Interview Q&A

## Q&A

**Q: Why is the answer for "aaa" equal to 6?**  
A: The palindromic substrings are: `"a"` at position 0, `"a"` at position 1, `"a"` at position 2, `"aa"` at positions 0-1, `"aa"` at positions 1-2, and `"aaa"` at positions 0-2. Each position-based occurrence counts separately.

**Q: How is this different from the Longest Palindromic Substring problem?**  
A: In that problem, we track the maximum-length palindrome. Here, we count every palindrome found during every expansion step. The expand function increments a counter instead of comparing lengths.

**Q: Can we use a DP table instead?**  
A: Yes. Build `dp[i][j]` = true if `s[i..j]` is a palindrome. Iterate all substrings in increasing length order. Count all `dp[i][j] == true` entries. O(n²) time and space.

**Q: Is there an O(n) solution?**  
A: Manacher's algorithm computes palindrome radii at each center in O(n) total. Count is then summed from the radii array. For interviews, the O(n²) expand approach is the expected answer.

**Q: What is the formula for the count in a string of all-same characters like "aaaa"?**  
A: For a string of n identical characters, the count is `n*(n+1)/2`. For "aaaa": 4×5/2 = 10. ("a"×4, "aa"×3, "aaa"×2, "aaaa"×1 = 4+3+2+1 = 10.)


---

9. Summary

## Summary

| Attribute             | Value                                                               |
| --------------------- | ------------------------------------------------------------------- |
| **Problem**           | Count all palindromic substrings (positionally distinct)            |
| **Optimal Algorithm** | Expand around center — count each expansion step                    |
| **Time Complexity**   | O(n²)                                                               |
| **Space Complexity**  | O(1)                                                                |
| **Key Insight**       | Each successful expansion step is exactly one palindromic substring |

### Core Pattern

> Same center-expansion as Longest Palindromic Substring, but count instead of compare lengths. Every match inside the expand loop is one result.

### When to Use This Pattern

- Counting palindromic substrings.
- Any "how many valid windows of type X exist?" problem where validity is checked by center expansion.

