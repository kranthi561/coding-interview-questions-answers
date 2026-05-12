# Longest Substring Without Repeating Characters

---

1. What is Longest Substring Without Repeating Characters?

## What is Longest Substring Without Repeating Characters?

Given a string `s`, find the **length of the longest substring** that contains **no duplicate characters**.

A **substring** is a contiguous sequence of characters within the string (not a subsequence).

**Example:**

```
Input:  s = "abcabcbb"
Output: 3
Explanation: "abc" is the longest substring without repeating characters.

Input:  s = "bbbbb"
Output: 1
Explanation: "b" is the only non-repeating substring.

Input:  s = "pwwkew"
Output: 3
Explanation: "wke" is the answer; "pwke" is a subsequence, not a substring.
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Given a string, return the length of the longest substring with all unique characters.
- A substring is contiguous — gaps are not allowed.

### Non-Functional Requirements

| Concern         | Expectation                                             |
| --------------- | ------------------------------------------------------- |
| **Scale**       | String length up to 50,000 characters                   |
| **Latency**     | O(n) expected — single pass through the string          |
| **Correctness** | Handle empty string (return 0) and all-same chars (return 1) |

### Clarifying Questions to Ask

- Is the input ASCII only or Unicode? → ASCII (LeetCode: printable characters).
- Is the string guaranteed non-null? → Yes.
- What is returned for an empty string? → 0.
- Are spaces and punctuation treated as characters? → Yes.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Brute Force (check all substrings)

|           | Complexity                                       |
| --------- | ------------------------------------------------ |
| **Time**  | O(n³) — O(n²) substrings × O(n) uniqueness check |
| **Space** | O(min(n, charset)) — set to track characters     |

### Approach 2: Sliding Window with Hash Set (Optimal)

|           | Complexity                                               |
| --------- | -------------------------------------------------------- |
| **Time**  | O(n) — each character is added and removed at most once  |
| **Space** | O(min(n, charset)) — set holds at most charset size keys |

**Why O(n)?** The left and right pointers each move at most n steps in total — the combined movement is bounded by 2n.


---

4. Algorithm Choice and Why

## Algorithm: Sliding Window with Hash Map

### Why This Algorithm?

| Option                     | Time   | Space              | Chosen?                           |
| -------------------------- | ------ | ------------------ | --------------------------------- |
| Brute Force                | O(n³)  | O(charset)         | No — too slow                     |
| Sliding Window + Hash Set  | O(n)   | O(charset)         | Good but shrinks one char at a time |
| **Sliding Window + Hash Map** | **O(n)** | **O(charset)** | **Yes — jump left pointer directly** |

### Logic

Use a hash map `{ character → last seen index }`.

- Maintain a `left` pointer marking the start of the current valid window.
- Expand `right` pointer one step at a time.
- If `s[right]` was seen before **and** its last index is within `[left, right)`, jump `left` to `lastSeen[s[right]] + 1` to skip past the duplicate.
- Track the maximum `right - left + 1`.

This avoids shrinking the window one character at a time — we jump directly to the correct position.


---

5. High-Level Design

## High-Level Design

### Components

1. **Window Manager** — Two pointers `left` and `right` defining the current valid window.
2. **Character Index Store** — Hash map recording the most recent index of each character.
3. **Max Tracker** — Updates the best window length seen so far.

### Data Flow Diagram

```
s = "abcabcbb"

right=0 'a': map={a:0}          window=[0,0] len=1  max=1
right=1 'b': map={a:0,b:1}      window=[0,1] len=2  max=2
right=2 'c': map={a:0,b:1,c:2}  window=[0,2] len=3  max=3
right=3 'a': 'a' seen at 0 ≥ left(0) → left=1
             map={a:3,b:1,c:2}  window=[1,3] len=3  max=3
right=4 'b': 'b' seen at 1 ≥ left(1) → left=2
             map={a:3,b:4,c:2}  window=[2,4] len=3  max=3
right=5 'c': 'c' seen at 2 ≥ left(2) → left=3
             map={a:3,b:4,c:5}  window=[3,5] len=3  max=3
right=6 'b': 'b' seen at 4 ≥ left(3) → left=5
right=7 'b': 'b' seen at 6 ≥ left(5) → left=7

Result: 3
```


---

6. Java Solution

## Java Implementation

```java
import java.util.HashMap;
import java.util.Map;

public class LongestSubstringWithoutRepeating {

    /**
     * Returns the length of the longest substring without repeating characters.
     *
     * Sliding window: expand right freely; when a duplicate is found within
     * the window, jump left past the previous occurrence of that character.
     *
     * @param s input string
     * @return length of the longest unique-character substring
     */
    public int lengthOfLongestSubstring(String s) {
        // Map: character → most recent index seen
        Map<Character, Integer> lastSeen = new HashMap<>();
        int maxLen = 0;
        int left = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);

            // If character was seen inside the current window, move left past it
            if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
                left = lastSeen.get(c) + 1;
            }

            // Always update to the latest index of this character
            lastSeen.put(c, right);

            // Update the best window size
            maxLen = Math.max(maxLen, right - left + 1);
        }

        return maxLen;
    }

    public static void main(String[] args) {
        LongestSubstringWithoutRepeating solution = new LongestSubstringWithoutRepeating();

        System.out.println(solution.lengthOfLongestSubstring("abcabcbb")); // 3
        System.out.println(solution.lengthOfLongestSubstring("bbbbb"));    // 1
        System.out.println(solution.lengthOfLongestSubstring("pwwkew"));   // 3
        System.out.println(solution.lengthOfLongestSubstring(""));         // 0
        System.out.println(solution.lengthOfLongestSubstring("au"));       // 2
    }
}
```

### Output

```
3
1
3
0
2
```


---

7. Python Solution

## Python Implementation

```python
def length_of_longest_substring(s: str) -> int:
    """
    Return the length of the longest substring without repeating characters.

    Sliding window with a hash map storing the most recent index of each
    character. When a duplicate is found inside the window, jump left
    directly past the previous occurrence.

    Args:
        s: input string

    Returns:
        Length of the longest substring with all unique characters

    Time:  O(n)
    Space: O(min(n, charset))
    """
    last_seen = {}  # character → most recent index
    max_len = 0
    left = 0

    for right, char in enumerate(s):
        # If duplicate is within the current window, advance left
        if char in last_seen and last_seen[char] >= left:
            left = last_seen[char] + 1

        last_seen[char] = right
        max_len = max(max_len, right - left + 1)

    return max_len


# ---- Example usage ----
if __name__ == "__main__":
    print(length_of_longest_substring("abcabcbb"))  # 3
    print(length_of_longest_substring("bbbbb"))     # 1
    print(length_of_longest_substring("pwwkew"))    # 3
    print(length_of_longest_substring(""))          # 0
    print(length_of_longest_substring("au"))        # 2
```

### Output

```
3
1
3
0
2
```


---

8. Interview Q&A

## Q&A

**Q: Why use a hash map instead of a hash set?**  
A: A set requires shrinking the window one character at a time when a duplicate is found — O(n) worst case per step. The map stores the last-seen index, letting us jump `left` directly to `lastSeen[c] + 1` in O(1).

**Q: Why check `lastSeen[c] >= left` before updating left?**  
A: The map may contain stale entries for characters that were in a previous window but are no longer relevant. Without this check, we might incorrectly shrink the window backward (move `left` to the left of its current position).

**Q: What is the maximum possible answer?**  
A: For ASCII (128 characters), the longest unique window is at most 128. For Unicode it can be much larger.

**Q: Can we solve this with O(1) space?**  
A: Only if we restrict to ASCII — use an integer array of size 128 instead of a hash map. Functionally equivalent but avoids hash overhead.

**Q: What happens if the entire string has unique characters?**  
A: `left` never moves right (no duplicates found), and the answer equals the full string length. O(n) with no extra branching.


---

9. Summary

## Summary

| Attribute             | Value                                                              |
| --------------------- | ------------------------------------------------------------------ |
| **Problem**           | Find length of longest substring with all unique characters        |
| **Optimal Algorithm** | Sliding window with last-seen index hash map                       |
| **Time Complexity**   | O(n)                                                               |
| **Space Complexity**  | O(min(n, charset))                                                 |
| **Key Insight**       | Jump `left` directly past the duplicate instead of shrinking by 1  |

### Core Pattern

> Sliding window: expand right freely; on constraint violation, contract left to restore validity — do it in O(1) by storing the position of the violating element.

### When to Use This Pattern

- "Longest substring satisfying property X" problems.
- Any fixed-constraint window where you can determine the new left boundary in O(1).

