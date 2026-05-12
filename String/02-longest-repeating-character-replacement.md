# Longest Repeating Character Replacement

---

1. What is Longest Repeating Character Replacement?

## What is Longest Repeating Character Replacement?

Given a string `s` and an integer `k`, you can replace **at most k characters** in the string with any letter. Find the **length of the longest substring** where all characters are the same after those replacements.

**Example:**

```
Input:  s = "ABAB", k = 2
Output: 4
Explanation: Replace both 'A's (or both 'B's) → "BBBB" or "AAAA". Length = 4.

Input:  s = "AABABBA", k = 1
Output: 4
Explanation: Replace one 'B' in "AABA" → "AAAA". Length = 4.
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Given string `s` (uppercase letters) and integer `k`, return the length of the longest valid substring.
- A window is valid when `(window size − count of most frequent character) ≤ k`.

### Non-Functional Requirements

| Concern     | Expectation                                      |
| ----------- | ------------------------------------------------ |
| **Scale**   | String length up to 100,000                      |
| **Latency** | O(n) expected — single pass with sliding window  |
| **Space**   | O(1) — only 26 letter frequency counters needed  |

### Clarifying Questions to Ask

- Only uppercase English letters? → Yes (LeetCode constraint).
- Can k be 0? → Yes — then no replacements allowed; find longest run of same char.
- Can k ≥ len(s)? → Yes — then answer is len(s).


---

3. Time and Space Complexity

## Complexity Analysis

### Sliding Window (Optimal)

|           | Complexity                                            |
| --------- | ----------------------------------------------------- |
| **Time**  | O(n) — right pointer traverses string once            |
| **Space** | O(1) — frequency array of fixed size 26               |

**Key invariant:** `windowSize - maxFreq ≤ k`  
where `maxFreq` is the count of the most frequent character in the current window.


---

4. Algorithm Choice and Why

## Algorithm: Sliding Window with Max-Frequency Tracking

### Why This Algorithm?

| Option                        | Time   | Space  | Chosen?                              |
| ----------------------------- | ------ | ------ | ------------------------------------ |
| Brute force all substrings    | O(n²)  | O(1)   | No — too slow                        |
| **Sliding window + freq array** | **O(n)** | **O(1)** | **Yes**                        |

### Logic

- Maintain a frequency array `count[26]` for the current window.
- Track `maxFreq` — the highest frequency character count in the window.
- Window validity condition: `(right - left + 1) - maxFreq ≤ k`.
- If invalid, **shrink from left by 1** (decrement `count[s[left]]`, advance `left`).
- Because we never shrink the window below the current best size, the window only grows or stays the same — `right - left` monotonically increases.

**Why never shrink below current best?** We're looking for the maximum length. Once the window is size `W`, we only want to record a result when we find `W+1` — so we advance left by at most 1 when invalid, keeping the window size stable rather than collapsing it.


---

5. High-Level Design

## High-Level Design

### Components

1. **Frequency Counter** — `count[26]` tracks character frequencies in the current window.
2. **Max Frequency Tracker** — Stores the highest single-character count seen in the window.
3. **Window Validity Checker** — `windowSize - maxFreq ≤ k`.
4. **Left Shrinker** — Advances `left` by 1 when the window becomes invalid.

### Data Flow Diagram

```
s = "AABABBA", k = 1

right=0 'A': count={A:1}         maxFreq=1  window=1  valid(1-1=0≤1) ✓ max=1
right=1 'A': count={A:2}         maxFreq=2  window=2  valid(2-2=0≤1) ✓ max=2
right=2 'B': count={A:2,B:1}     maxFreq=2  window=3  valid(3-2=1≤1) ✓ max=3
right=3 'A': count={A:3,B:1}     maxFreq=3  window=4  valid(4-3=1≤1) ✓ max=4
right=4 'B': count={A:3,B:2}     maxFreq=3  window=5  valid(5-3=2>1) ✗
  shrink: left=1, count={A:2,B:2} window=4
right=5 'B': count={A:2,B:3}     maxFreq=3  window=5  valid(5-3=2>1) ✗
  shrink: left=2, count={A:1,B:3} window=4
right=6 'A': count={A:2,B:3}     maxFreq=3  window=5  valid(5-3=2>1) ✗
  shrink: left=3, count={A:2,B:2} window=4   max stays 4

Result: 4
```


---

6. Java Solution

## Java Implementation

```java
public class LongestRepeatingCharReplacement {

    /**
     * Returns the length of the longest substring achievable by replacing
     * at most k characters so all characters in the window are the same.
     *
     * Validity: (windowSize - maxFrequency) <= k.
     * When invalid, shrink from the left by 1 — never collapse the window
     * below the current best size.
     *
     * @param s input string (uppercase letters)
     * @param k maximum replacements allowed
     * @return length of the longest valid substring
     */
    public int characterReplacement(String s, int k) {
        int[] count = new int[26];   // frequency of each letter in window
        int left = 0;
        int maxFreq = 0;             // highest single-char count in window

        for (int right = 0; right < s.length(); right++) {
            int idx = s.charAt(right) - 'A';
            count[idx]++;

            // maxFreq only increases — we don't bother recomputing on shrink
            // because we only want windows larger than the current best
            maxFreq = Math.max(maxFreq, count[idx]);

            // If window is invalid, shrink by 1 from the left
            int windowSize = right - left + 1;
            if (windowSize - maxFreq > k) {
                count[s.charAt(left) - 'A']--;
                left++;
            }
        }

        // Window size at end = right - left + 1 = s.length() - left
        return s.length() - left;
    }

    public static void main(String[] args) {
        LongestRepeatingCharReplacement solution = new LongestRepeatingCharReplacement();

        System.out.println(solution.characterReplacement("ABAB", 2));     // 4
        System.out.println(solution.characterReplacement("AABABBA", 1));  // 4
        System.out.println(solution.characterReplacement("AAAA", 0));     // 4
        System.out.println(solution.characterReplacement("ABCDE", 1));    // 2
    }
}
```

### Output

```
4
4
4
2
```


---

7. Python Solution

## Python Implementation

```python
def character_replacement(s: str, k: int) -> int:
    """
    Return the length of the longest substring where all characters match
    after replacing at most k characters.

    Validity condition: (window_size - max_freq) <= k.
    Shrink from left by 1 when invalid — never collapse below current best.

    Args:
        s: uppercase English letter string
        k: max replacements allowed

    Returns:
        Length of the longest valid substring

    Time:  O(n)
    Space: O(1) — 26-entry frequency array
    """
    count = [0] * 26
    left = 0
    max_freq = 0

    for right, char in enumerate(s):
        count[ord(char) - ord('A')] += 1
        max_freq = max(max_freq, count[ord(char) - ord('A')])

        # Window invalid: shrink by 1 from left
        if (right - left + 1) - max_freq > k:
            count[ord(s[left]) - ord('A')] -= 1
            left += 1

    return len(s) - left


# ---- Example usage ----
if __name__ == "__main__":
    print(character_replacement("ABAB", 2))     # 4
    print(character_replacement("AABABBA", 1))  # 4
    print(character_replacement("AAAA", 0))     # 4
    print(character_replacement("ABCDE", 1))    # 2
```

### Output

```
4
4
4
2
```


---

8. Interview Q&A

## Q&A

**Q: Why do we never recompute maxFreq when shrinking the window?**  
A: We only want windows larger than the current best. If shrinking causes maxFreq to decrease, the window is still the same size — it won't produce a new maximum. We only update maxFreq upward (when the window grows), so this is safe.

**Q: What does `windowSize - maxFreq` represent?**  
A: It's the number of characters in the window that are NOT the most frequent character — i.e., the minimum number of replacements needed to make all characters in the window the same.

**Q: Why shrink by exactly 1 instead of finding the new left boundary?**  
A: We're maximizing window size. Once we've found a window of size W, we advance both pointers together, keeping the window at size W until we find something better at W+1.

**Q: What if k ≥ s.length()?**  
A: We can replace every character, so the answer is `s.length()`. The algorithm handles this naturally — the window never becomes invalid.

**Q: Does this work for lowercase letters?**  
A: Change `- 'A'` to `- 'a'`. The logic is identical.


---

9. Summary

## Summary

| Attribute             | Value                                                                    |
| --------------------- | ------------------------------------------------------------------------ |
| **Problem**           | Longest substring where all chars match after ≤ k replacements          |
| **Optimal Algorithm** | Sliding window with max-frequency tracking                               |
| **Time Complexity**   | O(n)                                                                     |
| **Space Complexity**  | O(1) — fixed 26-char frequency array                                     |
| **Key Insight**       | `windowSize - maxFreq ≤ k`; shrink by 1 to maintain current best size   |

### Core Pattern

> Sliding window with a constraint on the "minority" count. Only shrink enough to keep the window size stable — never collapse it.

### When to Use This Pattern

- "Longest window with at most k changes" problems.
- Character frequency problems where you need the dominant character count.

