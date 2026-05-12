# Valid Palindrome

---

1. What is Valid Palindrome?

## What is Valid Palindrome?

Given a string `s`, return `true` if it is a **palindrome** after:
- Converting all uppercase letters to lowercase.
- Removing all non-alphanumeric characters.

A palindrome reads the same forward and backward.

**Example:**

```
Input:  s = "A man, a plan, a canal: Panama"
Output: true
Cleaned: "amanaplanacanalpanama" — same forward and backward.

Input:  s = "race a car"
Output: false
Cleaned: "raceacar" — not a palindrome.

Input:  s = " "
Output: true
Cleaned: "" — empty string is a palindrome.
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Ignore non-alphanumeric characters (spaces, punctuation, symbols).
- Compare case-insensitively.
- Return `true` if the filtered string is a palindrome.

### Non-Functional Requirements

| Concern         | Expectation                                  |
| --------------- | -------------------------------------------- |
| **Scale**       | String length up to 200,000                  |
| **Latency**     | O(n) — single pass with two pointers         |
| **Space**       | O(1) — no new string allocation needed       |

### Clarifying Questions to Ask

- What counts as alphanumeric? → Letters and digits only.
- Is comparison case-sensitive? → No, case-insensitive.
- Is an empty string a palindrome? → Yes.


---

3. Time and Space Complexity

## Complexity Analysis

### Approach 1: Clean String Then Check

|           | Complexity                                       |
| --------- | ------------------------------------------------ |
| **Time**  | O(n) — filter + reverse/compare                  |
| **Space** | O(n) — allocates a cleaned copy of the string    |

### Approach 2: Two Pointers on Original String (Optimal)

|           | Complexity                                                  |
| --------- | ----------------------------------------------------------- |
| **Time**  | O(n) — each pointer moves at most n steps total             |
| **Space** | O(1) — no extra string; only two integer pointers           |


---

4. Algorithm Choice and Why

## Algorithm: Two-Pointer on Original String

### Why This Algorithm?

| Option                     | Time  | Space  | Chosen?                              |
| -------------------------- | ----- | ------ | ------------------------------------ |
| Clean string + reverse     | O(n)  | O(n)   | No — unnecessary allocation          |
| **Two pointers in-place**  | O(n)  | **O(1)** | **Yes — no extra space**           |

### Logic

1. Set `left = 0`, `right = len(s) - 1`.
2. While `left < right`:
   - Skip non-alphanumeric characters by advancing `left` rightward or `right` leftward.
   - Compare `s[left]` and `s[right]` case-insensitively.
   - If they differ → return `false`.
   - Otherwise, advance both pointers inward.
3. Return `true`.


---

5. High-Level Design

## High-Level Design

### Components

1. **Left Pointer** — Starts at 0; skips non-alphanumeric characters moving right.
2. **Right Pointer** — Starts at end; skips non-alphanumeric characters moving left.
3. **Character Comparator** — Compares pointers' characters case-insensitively.

### Data Flow Diagram

```
s = "A man, a plan, a canal: Panama"

left=0  'A'  right=29 'a'  → lowercased: 'a'=='a' ✓ → left=1, right=28
left=1  ' '  → skip (non-alphanumeric)  left=2
right=28 'm' → 'm'
left=2  'm'  right=28 'm'  → 'm'=='m' ✓ → left=3, right=27
...
(all characters match)

→ true ✓
```


---

6. Java Solution

## Java Implementation

```java
public class ValidPalindrome {

    /**
     * Returns true if the string is a palindrome ignoring non-alphanumeric
     * characters and case differences.
     *
     * Two pointers: skip non-alphanumeric from both ends, compare remaining
     * characters case-insensitively.
     *
     * @param s input string
     * @return true if s is a valid palindrome
     */
    public boolean isPalindrome(String s) {
        int left = 0, right = s.length() - 1;

        while (left < right) {
            // Advance left past non-alphanumeric characters
            while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
                left++;
            }
            // Advance right past non-alphanumeric characters
            while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
                right--;
            }

            // Compare case-insensitively
            if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }

    public static void main(String[] args) {
        ValidPalindrome solution = new ValidPalindrome();

        System.out.println(solution.isPalindrome("A man, a plan, a canal: Panama")); // true
        System.out.println(solution.isPalindrome("race a car"));                     // false
        System.out.println(solution.isPalindrome(" "));                              // true
        System.out.println(solution.isPalindrome("0P"));                             // false
    }
}
```

### Output

```
true
false
true
false
```


---

7. Python Solution

## Python Implementation

```python
def is_palindrome(s: str) -> bool:
    """
    Return True if s is a palindrome ignoring non-alphanumeric characters
    and case.

    Two-pointer approach: skip non-alphanumeric characters from both ends,
    compare remaining characters case-insensitively.

    Args:
        s: input string

    Returns:
        True if s is a valid palindrome

    Time:  O(n)
    Space: O(1)
    """
    left, right = 0, len(s) - 1

    while left < right:
        # Skip non-alphanumeric from the left
        while left < right and not s[left].isalnum():
            left += 1
        # Skip non-alphanumeric from the right
        while left < right and not s[right].isalnum():
            right -= 1

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True


# ---- Example usage ----
if __name__ == "__main__":
    print(is_palindrome("A man, a plan, a canal: Panama"))  # True
    print(is_palindrome("race a car"))                      # False
    print(is_palindrome(" "))                               # True
    print(is_palindrome("0P"))                              # False
```

### Output

```
True
False
True
False
```


---

8. Interview Q&A

## Q&A

**Q: Why use two pointers instead of building a cleaned string?**  
A: Building a cleaned string allocates O(n) space. Two pointers skip non-alphanumeric characters in place — O(1) space with the same O(n) time.

**Q: What does `isalnum()` check?**  
A: Returns true if the character is a letter (a–z, A–Z) or a digit (0–9). Spaces, commas, colons, etc. return false.

**Q: How would you solve "Valid Palindrome II" where you can remove at most one character?**  
A: When a mismatch is found at `(left, right)`, try skipping one character from either side: check `isPalin(s, left+1, right)` and `isPalin(s, left, right-1)`. Return true if either is a palindrome.

**Q: What if the string is empty after filtering?**  
A: `left` starts at 0 and `right` at -1 (or both advance past each other immediately) — the while loop doesn't execute, and we return `true`. Empty string is a palindrome.

**Q: How would you check for a palindrome in a linked list?**  
A: Find the midpoint (slow/fast pointers), reverse the second half, then compare both halves node by node.


---

9. Summary

## Summary

| Attribute             | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| **Problem**           | Check if string is a palindrome ignoring non-alphanumeric and case |
| **Optimal Algorithm** | Two pointers with in-place alphanumeric skipping                  |
| **Time Complexity**   | O(n)                                                              |
| **Space Complexity**  | O(1)                                                              |
| **Key Insight**       | Skip non-alphanumeric characters by advancing pointers, not filtering |

### Core Pattern

> Two-pointer palindrome check: skip irrelevant characters in-place to avoid copying. Compare from both ends toward the center.

### When to Use This Pattern

- Palindrome validation with preprocessing rules.
- Any "check symmetry" problem where some characters can be ignored.

