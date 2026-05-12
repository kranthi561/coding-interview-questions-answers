# Minimum Window Substring

---

1. What is Minimum Window Substring?

## What is Minimum Window Substring?

Given strings `s` and `t`, return the **shortest substring of `s`** that contains **every character of `t`** (including duplicates). If no such window exists, return `""`.

**Example:**

```
Input:  s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
Explanation: "BANC" is the shortest substring containing A, B, and C.

Input:  s = "a", t = "a"
Output: "a"

Input:  s = "a", t = "aa"
Output: ""
Explanation: 't' needs two 'a's but 's' only has one.
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Find the smallest window in `s` containing all characters of `t` with correct frequencies.
- If no valid window exists, return `""`.
- If multiple windows of the same minimum length exist, return any one.

### Non-Functional Requirements

| Concern         | Expectation                                            |
| --------------- | ------------------------------------------------------ |
| **Scale**       | `s` and `t` lengths up to 100,000                      |
| **Latency**     | O(n + m) — linear in combined length                   |
| **Correctness** | `t` may have duplicate characters; frequencies matter  |

### Clarifying Questions to Ask

- Does character frequency in `t` matter? → Yes (e.g., `t = "AA"` requires two 'A's).
- Is the answer guaranteed to be unique? → Not necessarily; return any minimum-length window.
- Case-sensitive? → Yes.
- Can `t` be longer than `s`? → Yes → return `""`.


---

3. Time and Space Complexity

## Complexity Analysis

### Sliding Window (Optimal)

|           | Complexity                                                 |
| --------- | ---------------------------------------------------------- |
| **Time**  | O(n + m) — build freq map O(m); each char visited twice O(n) |
| **Space** | O(m) — frequency map for characters in `t`                |

**n** = len(s), **m** = len(t).


---

4. Algorithm Choice and Why

## Algorithm: Sliding Window with Two Frequency Maps

### Why This Algorithm?

| Option                           | Time      | Space  | Chosen?                          |
| -------------------------------- | --------- | ------ | -------------------------------- |
| Brute force all substrings       | O(n²·m)   | O(m)   | No — far too slow                |
| **Sliding window + freq maps**   | **O(n+m)**| **O(m)** | **Yes**                        |

### Logic

1. Build `need` — frequency map of all characters in `t`.
2. Use two pointers `left` and `right`, a `window` frequency map, and a counter `formed` tracking how many characters in `t` are currently satisfied in the window (correct frequency).
3. Expand `right`: add `s[right]` to `window`. If its frequency matches `need[s[right]]`, increment `formed`.
4. When `formed == required` (all characters satisfied), try contracting: record the window if it's the smallest so far, remove `s[left]` from `window`, decrement `formed` if needed, advance `left`.
5. Repeat until `right` reaches the end of `s`.


---

5. High-Level Design

## High-Level Design

### Components

1. **Need Map** — Desired character frequencies from `t`.
2. **Window Map** — Current character frequencies within `[left, right]`.
3. **Formed Counter** — Count of distinct characters fully satisfied.
4. **Min Window Recorder** — Tracks the start index and length of the best window.

### Data Flow Diagram

```
s = "ADOBECODEBANC",  t = "ABC"
need = {A:1, B:1, C:1},  required = 3

Expand right until formed == 3:
  right=0 'A': window={A:1}            formed=1
  right=1 'D': window={A:1,D:1}
  right=2 'O': ...
  right=3 'B': window={...,B:1}        formed=2
  right=4 'E': ...
  right=5 'C': window={...,C:1}        formed=3 ✓  best="ADOBEC" len=6

Contract left:
  left=0 'A': window={A:0,...}         formed=2  (A no longer satisfied)

Expand until formed == 3 again:
  right=9 'A': window={A:1,...}        formed=3 ✓  best="DOBECODEBA" — longer, skip

Continue contracting...
  Eventually: best="BANC" len=4

Result: "BANC"
```


---

6. Java Solution

## Java Implementation

```java
import java.util.HashMap;
import java.util.Map;

public class MinimumWindowSubstring {

    /**
     * Returns the shortest substring of s containing all characters of t.
     *
     * Sliding window: expand right to satisfy all character requirements,
     * then contract left to minimize the window while staying valid.
     *
     * @param s source string
     * @param t target string (characters to include)
     * @return minimum window substring, or "" if none exists
     */
    public String minWindow(String s, String t) {
        if (s.isEmpty() || t.isEmpty()) return "";

        // Frequency map for required characters
        Map<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) {
            need.merge(c, 1, Integer::sum);
        }

        int required = need.size(); // distinct characters we need to satisfy
        Map<Character, Integer> window = new HashMap<>();

        int left = 0, formed = 0;
        int minLen = Integer.MAX_VALUE, minLeft = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            window.merge(c, 1, Integer::sum);

            // Check if this character's requirement is now fully met
            if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
                formed++;
            }

            // Try to contract the window while it's valid
            while (formed == required) {
                // Record if this is the smallest window so far
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    minLeft = left;
                }

                // Remove leftmost character from window
                char leftChar = s.charAt(left);
                window.merge(leftChar, -1, Integer::sum);
                if (need.containsKey(leftChar) && window.get(leftChar) < need.get(leftChar)) {
                    formed--;
                }
                left++;
            }
        }

        return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
    }

    public static void main(String[] args) {
        MinimumWindowSubstring solution = new MinimumWindowSubstring();

        System.out.println(solution.minWindow("ADOBECODEBANC", "ABC")); // "BANC"
        System.out.println(solution.minWindow("a", "a"));               // "a"
        System.out.println(solution.minWindow("a", "aa"));              // ""
        System.out.println(solution.minWindow("AA", "AA"));             // "AA"
    }
}
```

### Output

```
BANC
a

AA
```


---

7. Python Solution

## Python Implementation

```python
from collections import Counter

def min_window(s: str, t: str) -> str:
    """
    Return the smallest substring of s containing all characters of t.

    Sliding window: expand right to cover all requirements, then contract
    left to minimize — repeat until right reaches end of s.

    Args:
        s: source string
        t: target characters (with frequencies)

    Returns:
        Minimum window substring, or "" if none exists

    Time:  O(n + m)
    Space: O(m)
    """
    if not s or not t:
        return ""

    need = Counter(t)          # required character frequencies
    required = len(need)       # number of distinct chars to satisfy
    window = {}

    left = 0
    formed = 0
    best = float('inf'), 0, 0  # (length, left, right)

    for right, char in enumerate(s):
        window[char] = window.get(char, 0) + 1

        # Check if this character's requirement is now met
        if char in need and window[char] == need[char]:
            formed += 1

        # Contract window while all requirements are satisfied
        while formed == required:
            if right - left + 1 < best[0]:
                best = (right - left + 1, left, right)

            left_char = s[left]
            window[left_char] -= 1
            if left_char in need and window[left_char] < need[left_char]:
                formed -= 1
            left += 1

    return "" if best[0] == float('inf') else s[best[1]: best[2] + 1]


# ---- Example usage ----
if __name__ == "__main__":
    print(min_window("ADOBECODEBANC", "ABC"))  # "BANC"
    print(min_window("a", "a"))                # "a"
    print(min_window("a", "aa"))               # ""
    print(min_window("AA", "AA"))              # "AA"
```

### Output

```
BANC
a

AA
```


---

8. Interview Q&A

## Q&A

**Q: What does `formed` track and why?**  
A: `formed` counts how many distinct characters in `t` currently have their frequency fully satisfied in the window. When `formed == required` (len of need map), the window is valid. This avoids re-scanning the window map every step.

**Q: Why track distinct characters rather than total character count?**  
A: We need each character's frequency to be at least the required count. A single counter for total characters wouldn't capture whether each individual requirement is met.

**Q: What if `t` has repeated characters like "AAB"?**  
A: `need = {A:2, B:1}`. `formed` increments for 'A' only when `window['A'] == 2` (not at 1), and again would not re-increment — so frequency matching is exact.

**Q: Why use `required = len(need)` instead of `len(t)`?**  
A: `len(t)` counts all characters (including duplicates). We track satisfaction per distinct character, so `len(need)` (number of distinct required chars) is the correct target.

**Q: Can the window map hold characters not in `t`?**  
A: Yes. Characters in `s` but not in `t` are added to `window` but ignored in `formed` checks — they don't affect validity.


---

9. Summary

## Summary

| Attribute             | Value                                                                     |
| --------------------- | ------------------------------------------------------------------------- |
| **Problem**           | Smallest substring of `s` containing all characters of `t`               |
| **Optimal Algorithm** | Sliding window with two frequency maps and a `formed` counter             |
| **Time Complexity**   | O(n + m)                                                                  |
| **Space Complexity**  | O(m)                                                                      |
| **Key Insight**       | Track satisfaction per distinct character; contract greedily once valid   |

### Core Pattern

> Expand until valid, contract until invalid, record minimum at each valid state. The `formed` counter makes validity checks O(1) per step.

### When to Use This Pattern

- "Find smallest window satisfying a multi-character constraint" problems.
- Anagram detection, substring permutation, and coverage problems.

