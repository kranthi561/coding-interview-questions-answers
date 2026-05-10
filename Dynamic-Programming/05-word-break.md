# Word Break

---

<details>
<summary><strong>1. What is meant by Word Break?</strong></summary>

## What is Word Break?

**Word Break** is a DP problem where:

- You are given a string `s` and a dictionary of words `wordDict`.
- You need to determine if `s` can be completely segmented into one or more dictionary words.

**Goal:** Return `true` if `s` can be segmented, `false` otherwise.

**Constraints:**
- `1 <= s.length <= 300`
- `1 <= wordDict.length <= 1000`
- `1 <= wordDict[i].length <= 20`
- Words in `wordDict` may be reused.

**Example:**

```
Input:  s = "leetcode", wordDict = ["leet", "code"]
Output: true
Explanation: "leetcode" = "leet" + "code"
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Determine if the entire string `s` can be broken into valid dictionary words.
- Words can be reused multiple times.
- Every character of `s` must be covered — no leftovers.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | s up to 300 chars; dictionary up to 1000 words      |
| **Latency**     | O(n² × m) or O(n × L) — runs in milliseconds       |
| **Correctness** | Must explore all split points, not just greedy      |
| **Edge Cases**  | Single-character string, all chars in dict, empty   |

### Clarifying Questions to Ask

- Can words be reused? → Yes (unlimited).
- Must every character be used? → Yes, full segmentation required.
- Is the dictionary case-sensitive? → Yes (assume lowercase).
- Can `s` be empty? → Constraints say length ≥ 1.

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force Recursion (No Memoization)

| | Complexity |
| --- | --- |
| **Time** | O(2ⁿ) — exponential, re-explores same substrings |
| **Space** | O(n) — recursion stack |

### Approach 2: Memoization (Top-Down DP)

| | Complexity |
| --- | --- |
| **Time** | O(n² × m) — n positions × n substring lengths × m word check |
| **Space** | O(n) — memo array |

### Approach 3: Bottom-Up DP (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(n² × L) where L = average word length for set lookup |
| **Space** | O(n) — dp boolean array |

**With a HashSet for word lookup:** checking if a substring is in the dictionary is O(L) where L = substring length (hashing cost). Total: O(n² × L).

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Bottom-Up Dynamic Programming with HashSet

### Why?

| Option | Time | Space | Chosen? |
| --- | --- | --- | --- |
| Brute Force | O(2ⁿ) | O(n) | No |
| Top-Down Memo | O(n²×L) | O(n) | Equivalent |
| **Bottom-Up DP** | **O(n²×L)** | **O(n)** | **Yes — iterative, no stack** |

### Key Insight

`dp[i] = true` means the prefix `s[0..i-1]` can be segmented into dictionary words.

Recurrence:
```
dp[0] = true   (empty prefix is always valid)

dp[i] = true if:
    ∃ j in [0, i) such that:
        dp[j] == true  AND  s[j..i-1] ∈ wordDict
```

Check all split points `j` for each `i`. If you find a valid `j`, mark `dp[i] = true`.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Input Layer** — Receives string `s` and `wordDict`
2. **Word Set** — HashSet for O(L) word lookup
3. **DP Array** — Boolean array of size n+1
4. **Split Point Scanner** — Tries every `(j, i)` pair
5. **Output Layer** — Returns `dp[n]`

### Data Flow Diagram

```
s = "leetcode", wordDict = {"leet", "code"}

dp = [T, F, F, F, F, F, F, F, F]
      0  1  2  3  4  5  6  7  8

i=4: j=0→ dp[0]=T, s[0..3]="leet" ∈ dict → dp[4]=T
i=8: j=4→ dp[4]=T, s[4..7]="code" ∈ dict → dp[8]=T

dp = [T, F, F, F, T, F, F, F, T]

Output: dp[8] = true ✓
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.*;

public class WordBreak {

    /**
     * Determine if string s can be segmented into words from wordDict.
     *
     * Bottom-up DP: dp[i] = true if s[0..i-1] is fully segmentable.
     * For each end position i, check every split point j:
     *   dp[j] is true AND s[j..i-1] is in the dictionary → dp[i] = true.
     *
     * @param s        input string
     * @param wordDict list of valid dictionary words
     * @return true if s can be broken into dictionary words
     *
     * Time:  O(n² × L) where L = average word length
     * Space: O(n + W) where W = total chars in dictionary (for HashSet)
     */
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> wordSet = new HashSet<>(wordDict); // O(1) average lookup
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true; // empty prefix is always valid

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                // Check if prefix s[0..j-1] is segmentable AND s[j..i-1] is a word
                if (dp[j] && wordSet.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break; // no need to check further split points
                }
            }
        }

        return dp[n];
    }

    public static void main(String[] args) {
        WordBreak sol = new WordBreak();

        // Example 1: Valid segmentation
        System.out.println("s=leetcode, dict=[leet,code]");
        System.out.println("Output: " + sol.wordBreak("leetcode",
                Arrays.asList("leet", "code"))); // true

        // Example 2: Word reuse
        System.out.println("\ns=applepenapple, dict=[apple,pen]");
        System.out.println("Output: " + sol.wordBreak("applepenapple",
                Arrays.asList("apple", "pen"))); // true

        // Example 3: Impossible
        System.out.println("\ns=catsandog, dict=[cats,dog,sand,and,cat]");
        System.out.println("Output: " + sol.wordBreak("catsandog",
                Arrays.asList("cats", "dog", "sand", "and", "cat"))); // false

        // Example 4: Single char
        System.out.println("\ns=a, dict=[a]");
        System.out.println("Output: " + sol.wordBreak("a",
                Arrays.asList("a"))); // true
    }
}
```

### Output

```
s=leetcode, dict=[leet,code]
Output: true

s=applepenapple, dict=[apple,pen]
Output: true

s=catsandog, dict=[cats,dog,sand,and,cat]
Output: false

s=a, dict=[a]
Output: true
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def word_break(s: str, word_dict: list[str]) -> bool:
    """
    Determine if string s can be segmented into words from word_dict.

    Strategy: Bottom-up DP.
    dp[i] = True if s[:i] can be fully segmented.
    For each end i, try all split points j:
      dp[j] is True AND s[j:i] is in the word set → dp[i] = True.

    Args:
        s: Input string
        word_dict: List of valid words

    Returns:
        True if full segmentation is possible, False otherwise

    Time:  O(n² × L)
    Space: O(n)
    """
    word_set = set(word_dict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True  # empty prefix is always valid

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break  # found one valid split — no need to continue

    return dp[n]


if __name__ == "__main__":
    test_cases = [
        ("leetcode",      ["leet", "code"],                     True),
        ("applepenapple", ["apple", "pen"],                     True),
        ("catsandog",     ["cats", "dog", "sand", "and", "cat"], False),
        ("a",             ["a"],                                 True),
        ("cars",          ["car", "ca", "rs"],                  True),
    ]

    for s, words, expected in test_cases:
        result = word_break(s, words)
        status = "PASS" if result == expected else "FAIL"
        print(f"[{status}] s={s!r}, dict={words}")
        print(f"  Output: {result}, Expected: {expected}\n")
```

### Output

```
[PASS] s='leetcode', dict=['leet', 'code']
  Output: True, Expected: True

[PASS] s='applepenapple', dict=['apple', 'pen']
  Output: True, Expected: True

[PASS] s='catsandog', dict=['cats', 'dog', 'sand', 'and', 'cat']
  Output: False, Expected: False

[PASS] s='a', dict=['a']
  Output: True, Expected: True

[PASS] s='cars', dict=['car', 'ca', 'rs']
  Output: True, Expected: True
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why doesn't greedy work for Word Break?**
A: Greedy picks the longest (or first) matching word at each position, which can fail. Example: `s="cars"`, `dict=["car","ca","rs"]`. Greedy picks "car" leaving "s" unmatched, but "ca"+"rs" works.

**Q: What does `dp[0] = true` represent?**
A: The empty string is always a valid segmentation (trivially). It serves as the base case that allows the first word in the string to be recognized.

**Q: Can the word dictionary have duplicate words?**
A: Yes, but converting to a `HashSet` deduplicates them automatically and provides O(1) average lookup.

**Q: How does this differ from Combination Sum?**
A: Both use DP with backtracking ideas, but Word Break only needs a boolean answer. Combination Sum enumerates all combinations. If Word Break needed all segmentations (Word Break II), it becomes a backtracking problem.

**Q: What is Word Break II?**
A: Return all possible segmentations. Requires DFS/backtracking with memoization: for each valid split, recursively find all valid segmentations of the remainder.

**Q: How do you handle very long strings efficiently?**
A: Trie optimization: build a Trie from `wordDict`. Instead of `O(L)` substring hashing, traverse the Trie from each split point — prune early when no word starts with the current character.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                               |
| --------------------- | ------------------------------------------------------------------- |
| **Problem**           | Determine if a string can be fully segmented into dictionary words  |
| **Optimal Algorithm** | Bottom-Up DP with HashSet word lookup                               |
| **Time Complexity**   | O(n² × L)                                                          |
| **Space Complexity**  | O(n)                                                                |
| **Key Insight**       | `dp[i] = true` if any `dp[j] = true` and `s[j..i-1]` is a word   |

### Core Pattern

> Think of `dp[i]` as "can I build a valid sentence up to position i?" Every position scans all prior valid positions and checks if the segment in between is a dictionary word.

### When to Use This Pattern

- Any "can the entire input be partitioned into valid pieces?" problem.
- Related: Word Break II (all solutions), Palindrome Partitioning, Decode Ways.

</details>
