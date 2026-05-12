# Valid Anagram

---

1. What is Valid Anagram?

## What is Valid Anagram?

Given two strings `s` and `t`, return `true` if `t` is an **anagram** of `s`, and `false` otherwise.

An **anagram** is a word formed by rearranging the letters of another word — using all original letters exactly once.

**Example:**

```
Input:  s = "anagram", t = "nagaram"
Output: true

Input:  s = "rat", t = "car"
Output: false
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Return `true` if `s` and `t` contain the same characters with the same frequencies.
- Both strings must be the same length (otherwise immediately false).

### Non-Functional Requirements

| Concern         | Expectation                                      |
| --------------- | ------------------------------------------------ |
| **Scale**       | String length up to 50,000                       |
| **Latency**     | O(n) — single or double pass                     |
| **Space**       | O(1) for lowercase ASCII (26 fixed counters)     |

### Clarifying Questions to Ask

- Only lowercase letters? → Yes (LeetCode base). Follow-up: Unicode?
- Are strings guaranteed non-null? → Yes.
- Are spaces significant? → Yes, treated as characters.
- What about case sensitivity? → Case-sensitive by default.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Sort Both Strings

|           | Complexity                             |
| --------- | -------------------------------------- |
| **Time**  | O(n log n) — sorting dominates         |
| **Space** | O(n) — sorted copies                   |

### Approach 2: Frequency Count Array (Optimal)

|           | Complexity                                               |
| --------- | -------------------------------------------------------- |
| **Time**  | O(n) — two passes (one per string)                       |
| **Space** | O(1) — fixed-size array of 26 integers                   |

### Approach 3: Single-Pass with One Array

|           | Complexity                                                     |
| --------- | -------------------------------------------------------------- |
| **Time**  | O(n) — one pass incrementing for `s`, decrementing for `t`     |
| **Space** | O(1)                                                           |


---

4. Algorithm Choice and Why

## Algorithm: Single Frequency Array (One-Pass)

### Why This Algorithm?

| Option                  | Time       | Space  | Chosen?                             |
| ----------------------- | ---------- | ------ | ----------------------------------- |
| Sort and compare        | O(n log n) | O(n)   | No — sorting is unnecessary         |
| Two frequency maps      | O(n)       | O(1)   | Works but uses two data structures  |
| **One frequency array** | **O(n)**   | **O(1)** | **Yes — cleanest O(n)/O(1) solution** |

### Logic

1. If `len(s) != len(t)`, return `false` immediately.
2. Create `count[26]`, initialized to zero.
3. Iterate over both strings simultaneously: increment `count[s[i] - 'a']`, decrement `count[t[i] - 'a']`.
4. After the loop, if all values in `count` are zero, the strings are anagrams.

Any non-zero entry means a character has unequal frequency between `s` and `t`.


---

5. High-Level Design

## High-Level Design

### Components

1. **Length Guard** — Early exit if lengths differ.
2. **Frequency Delta Array** — Single array incremented for `s`, decremented for `t`.
3. **Zero Checker** — Confirms all 26 entries are zero.

### Data Flow Diagram

```
s = "anagram",  t = "nagaram"

Initialize: count[26] = {0, 0, ..., 0}

Simultaneous pass:
  i=0: s='a'(+1), t='n'(-1)  → count[a]=1, count[n]=-1
  i=1: s='n'(+1), t='a'(-1)  → count[n]=0, count[a]=0
  i=2: s='a'(+1), t='g'(-1)  → count[a]=1, count[g]=-1
  i=3: s='g'(+1), t='a'(-1)  → count[g]=0, count[a]=0
  i=4: s='r'(+1), t='r'(-1)  → count[r]=0
  i=5: s='a'(+1), t='a'(-1)  → count[a]=0
  i=6: s='m'(+1), t='m'(-1)  → count[m]=0

All zeros → true ✓
```


---

6. Java Solution

## Java Implementation

```java
public class ValidAnagram {

    /**
     * Returns true if t is an anagram of s.
     *
     * One-pass: increment count for each char in s, decrement for t.
     * If all 26 entries are zero at the end, frequencies match exactly.
     *
     * @param s first string
     * @param t second string
     * @return true if t is an anagram of s
     */
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;

        int[] count = new int[26];

        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a']++;
            count[t.charAt(i) - 'a']--;
        }

        for (int freq : count) {
            if (freq != 0) return false;
        }

        return true;
    }

    public static void main(String[] args) {
        ValidAnagram solution = new ValidAnagram();

        System.out.println(solution.isAnagram("anagram", "nagaram")); // true
        System.out.println(solution.isAnagram("rat", "car"));         // false
        System.out.println(solution.isAnagram("a", "ab"));            // false
        System.out.println(solution.isAnagram("", ""));               // true
    }
}
```

### Output

```
true
false
false
true
```


---

7. Python Solution

## Python Implementation

```python
def is_anagram(s: str, t: str) -> bool:
    """
    Return True if t is an anagram of s.

    One-pass frequency delta: increment for s, decrement for t.
    If all 26 values are zero, characters and frequencies match.

    Args:
        s: first string
        t: second string

    Returns:
        True if t is an anagram of s

    Time:  O(n)
    Space: O(1) — fixed 26-entry array
    """
    if len(s) != len(t):
        return False

    count = [0] * 26

    for cs, ct in zip(s, t):
        count[ord(cs) - ord('a')] += 1
        count[ord(ct) - ord('a')] -= 1

    return all(c == 0 for c in count)


# ---- Example usage ----
if __name__ == "__main__":
    print(is_anagram("anagram", "nagaram"))  # True
    print(is_anagram("rat", "car"))          # False
    print(is_anagram("a", "ab"))             # False
    print(is_anagram("", ""))               # True
```

### Output

```
True
False
False
True
```


---

8. Interview Q&A

## Q&A

**Q: What if the strings contain Unicode characters?**  
A: Use a `HashMap<Character, Integer>` instead of a fixed-size array. Increment for `s`, decrement for `t`. Time stays O(n); space becomes O(k) where k is the character set size.

**Q: Why check `len(s) != len(t)` first?**  
A: If lengths differ, they can't be anagrams — this is an O(1) early exit that avoids unnecessary work.

**Q: Can you use `Counter` subtraction in Python?**  
A: Yes — `Counter(s) == Counter(t)` is clean and idiomatic. It's O(n) time and O(k) space, same as the array approach but with more overhead from the hash map.

**Q: What is the difference between an anagram and a permutation?**  
A: In this context, none — a string is an anagram of another if it is a permutation of its characters. The terms are used interchangeably in interview problems.

**Q: How would you check if one string is an anagram of any substring of another?**  
A: Use a sliding window of length `len(t)` over `s`, maintaining a frequency delta. This is the "Find All Anagrams in a String" problem (LeetCode 438).


---

9. Summary

## Summary

| Attribute             | Value                                                         |
| --------------------- | ------------------------------------------------------------- |
| **Problem**           | Check if two strings are anagrams (same chars, same counts)   |
| **Optimal Algorithm** | Single-pass frequency delta array                             |
| **Time Complexity**   | O(n)                                                          |
| **Space Complexity**  | O(1) — 26-entry array for lowercase ASCII                     |
| **Key Insight**       | Increment for s, decrement for t — all zeros means a match    |

### Core Pattern

> Frequency delta: use a single counter array, +1 for one string and −1 for the other. Zero across all entries confirms equal frequencies.

### When to Use This Pattern

- Anagram detection and permutation problems.
- Any "do two collections have the same elements?" check.

