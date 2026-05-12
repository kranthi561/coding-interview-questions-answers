# Alien Dictionary

---

1. What is meant by Alien Dictionary?

## What is Alien Dictionary?

**Alien Dictionary** is a topological sort problem where:

- You receive a list of `words` sorted in **lexicographic order** according to an alien language's alphabet.
- The alien language uses the same letters as English, but in a different order.

**Goal:** Derive the **ordering of letters** in the alien language from the sorted word list. Return any valid ordering. Return `""` if the ordering is impossible (cycle in constraints).

**Constraints:**
- `1 <= words.length <= 100`
- `1 <= words[i].length <= 100`
- Words contain only lowercase English letters.

**Example:**

```
Input:  words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"
Explanation:
  wrt < wrf  → t < f
  wrf < er   → w < e
  er  < ett  → r < t
  ett < rftt → e < r
  Topological order: w, e, r, t, f → "wertf"
```


---

2. Clarify Requirements — Functional and Non-Functional

## Requirements

### Functional Requirements

- Extract ordering constraints by comparing adjacent word pairs.
- Build a directed graph of character orderings.
- Topologically sort the graph to produce a valid character ordering.
- Return `""` if a cycle exists (impossible ordering).

### Non-Functional Requirements

| Concern         | Expectation                                              |
| --------------- | -------------------------------------------------------- |
| **Scale**       | ≤100 words × ≤100 chars — very small, O(C) where C=total chars |
| **Latency**     | O(C) — linear in total characters across all words      |
| **Correctness** | Must detect invalid inputs: cycles, prefix violations   |
| **Edge Cases**  | Single word, all same letter, shorter word before longer prefix |

### Clarifying Questions to Ask

- Can there be multiple valid orderings? → Yes, return any one.
- What if it's impossible? → Return `""`.
- Can a word be its own prefix in the list? (e.g., ["abc","ab"]) → Invalid → return `""`.
- Are all 26 letters guaranteed to appear? → No, only return letters that appear.


---

3. Estimate Time and Space Complexity

## Complexity Analysis

Let `C` = total number of characters across all words, `U` = number of unique characters.

### BFS Topological Sort (Kahn's Algorithm)

| | Complexity |
| --- | --- |
| **Time** | O(C) — compare adjacent words + O(U) for BFS |
| **Space** | O(U + edges) — adjacency list and in-degree array |

### DFS Topological Sort

| | Complexity |
| --- | --- |
| **Time** | O(C + U) |
| **Space** | O(U) — visited state + recursion stack |

Both approaches are effectively **O(C)** since C ≥ U always.


---

4. Which Algorithm and Why?

## Algorithm: Kahn's BFS Topological Sort

### Why Kahn's?

| Option | Correct? | Notes | Chosen? |
| --- | --- | --- | --- |
| DFS Topological Sort | Yes | Produces reverse order — needs reversal | Acceptable |
| **Kahn's BFS** | **Yes** | **Direct order, easy cycle detection** | **Yes** |

### Steps

```
1. Build adjacency list and in-degree map from unique characters.
2. Compare each adjacent pair of words:
   a. Find first differing character position.
   b. Add edge: char[i] → char[j], increment in-degree[char[j]].
   c. If word1 is a prefix of word2 but longer, return "" (invalid).
3. Initialize queue with all chars of in-degree 0.
4. BFS: process queue, decrement neighbors' in-degrees, enqueue at 0.
5. If result length < unique chars count → cycle → return "".
6. Return result string.
```


---

5. High-Level Design — Components and Data Flow

## High-Level Design

### Components

1. **Character Collector** — Find all unique characters across words.
2. **Constraint Extractor** — Compare adjacent word pairs for ordering edges.
3. **Graph Builder** — Adjacency list + in-degree count.
4. **Cycle Detector** — If word A is a prefix of word B but appears after → invalid.
5. **BFS Topological Sorter** — Kahn's algorithm on the character graph.
6. **Output Validator** — If not all characters processed → cycle → return `""`.

### Data Flow Diagram

```
words = ["wrt","wrf","er","ett","rftt"]

Step 1 — Extract edges:
  "wrt" vs "wrf" → first diff at pos 2: t→f   edge: t→f
  "wrf" vs "er"  → first diff at pos 0: w→e   edge: w→e
  "er"  vs "ett" → first diff at pos 1: r→t   edge: r→t
  "ett" vs "rftt"→ first diff at pos 0: e→r   edge: e→r

Step 2 — Graph:
  w → e
  e → r
  r → t
  t → f

In-degrees: w=0, e=1, r=1, t=1, f=1

Step 3 — BFS:
  Queue: [w]
  Process w → reduce e: in-degree=0 → Queue: [e]
  Process e → reduce r: in-degree=0 → Queue: [r]
  Process r → reduce t: in-degree=0 → Queue: [t]
  Process t → reduce f: in-degree=0 → Queue: [f]
  Process f → done

Result: "w e r t f" → "wertf"
```


---

6. Java Solution with Comments and Examples

## Java Implementation

```java
import java.util.*;

public class AlienDictionary {

    /**
     * Determine the character ordering of an alien language from sorted words.
     *
     * Strategy: Extract ordering edges from adjacent word comparisons,
     * build a directed graph, then topologically sort using Kahn's BFS.
     * If a cycle exists, return "".
     *
     * @param words list of words sorted in the alien language's order
     * @return valid character ordering string, or "" if impossible
     *
     * Time:  O(C) where C = total characters across all words
     * Space: O(U) where U = unique characters
     */
    public String alienOrder(String[] words) {
        // Step 1: Initialize adjacency list and in-degree for every unique character
        Map<Character, Set<Character>> adj = new HashMap<>();
        Map<Character, Integer> inDegree = new HashMap<>();
        for (String word : words) {
            for (char c : word.toCharArray()) {
                adj.putIfAbsent(c, new HashSet<>());
                inDegree.putIfAbsent(c, 0);
            }
        }

        // Step 2: Extract ordering edges by comparing adjacent word pairs
        for (int i = 0; i < words.length - 1; i++) {
            String w1 = words[i], w2 = words[i + 1];
            int minLen = Math.min(w1.length(), w2.length());
            boolean foundDiff = false;

            for (int j = 0; j < minLen; j++) {
                char c1 = w1.charAt(j), c2 = w2.charAt(j);
                if (c1 != c2) {
                    if (!adj.get(c1).contains(c2)) {  // avoid duplicate edges
                        adj.get(c1).add(c2);
                        inDegree.put(c2, inDegree.get(c2) + 1);
                    }
                    foundDiff = true;
                    break;
                }
            }

            // Invalid: "abc" before "ab" — longer word can't come first
            if (!foundDiff && w1.length() > w2.length()) return "";
        }

        // Step 3: Kahn's BFS topological sort
        Queue<Character> queue = new LinkedList<>();
        for (char c : inDegree.keySet()) {
            if (inDegree.get(c) == 0) queue.offer(c);
        }

        StringBuilder result = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            result.append(c);
            for (char neighbor : adj.get(c)) {
                inDegree.put(neighbor, inDegree.get(neighbor) - 1);
                if (inDegree.get(neighbor) == 0) queue.offer(neighbor);
            }
        }

        // If not all characters processed → cycle exists
        return result.length() == inDegree.size() ? result.toString() : "";
    }

    public static void main(String[] args) {
        AlienDictionary sol = new AlienDictionary();

        // Example 1: Valid ordering
        System.out.println("words=[wrt,wrf,er,ett,rftt]");
        System.out.println("Output: " + sol.alienOrder(new String[]{"wrt","wrf","er","ett","rftt"}));
        // "wertf"

        // Example 2: Cycle → impossible
        System.out.println("\nwords=[z,x,z]");
        System.out.println("Output: '" + sol.alienOrder(new String[]{"z","x","z"}) + "'"); // ""

        // Example 3: Single word
        System.out.println("\nwords=[abc]");
        System.out.println("Output: " + sol.alienOrder(new String[]{"abc"}));
        // "abc" or any permutation of a,b,c

        // Example 4: Invalid — prefix violation
        System.out.println("\nwords=[abc,ab]");
        System.out.println("Output: '" + sol.alienOrder(new String[]{"abc","ab"}) + "'"); // ""
    }
}
```

### Output

```
words=[wrt,wrf,er,ett,rftt]
Output: wertf

words=[z,x,z]
Output: ''

words=[abc]
Output: abc

words=[abc,ab]
Output: ''
```


---

7. Python Solution with Comments and Examples

## Python Implementation

```python
from collections import defaultdict, deque

def alien_order(words: list[str]) -> str:
    """
    Determine the alien language character ordering from a sorted word list.

    Strategy:
    1. Extract ordering edges by comparing adjacent word pairs.
    2. Build a directed graph (char → set of chars that come after).
    3. Topological sort using Kahn's BFS.
    4. If cycle detected (not all chars processed), return "".

    Args:
        words: Words sorted in alien language order

    Returns:
        Valid character ordering string, or "" if impossible

    Time:  O(C) — C = total characters across all words
    Space: O(U) — U = unique characters
    """
    adj = {c: set() for word in words for c in word}
    in_degree = {c: 0 for word in words for c in word}

    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i+1]
        min_len = min(len(w1), len(w2))
        found_diff = False

        for j in range(min_len):
            if w1[j] != w2[j]:
                if w2[j] not in adj[w1[j]]:  # avoid counting duplicate edges
                    adj[w1[j]].add(w2[j])
                    in_degree[w2[j]] += 1
                found_diff = True
                break

        if not found_diff and len(w1) > len(w2):
            return ""  # "abc" before "ab" is invalid

    queue = deque(c for c in in_degree if in_degree[c] == 0)
    result = []

    while queue:
        c = queue.popleft()
        result.append(c)
        for neighbor in adj[c]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return "".join(result) if len(result) == len(in_degree) else ""


if __name__ == "__main__":
    test_cases = [
        (["wrt","wrf","er","ett","rftt"], True),  # valid (multiple correct answers)
        (["z","x","z"],                  False),  # cycle → ""
        (["abc","ab"],                   False),  # prefix violation → ""
        (["z","x"],                      True),   # z < x
    ]

    for words, should_be_nonempty in test_cases:
        result = alien_order(words)
        is_nonempty = result != ""
        status = "PASS" if is_nonempty == should_be_nonempty else "FAIL"
        print(f"[{status}] words={words} → Output: '{result}'")
```

### Output

```
[PASS] words=['wrt', 'wrf', 'er', 'ett', 'rftt'] → Output: 'wertf'
[PASS] words=['z', 'x', 'z'] → Output: ''
[PASS] words=['abc', 'ab'] → Output: ''
[PASS] words=['z', 'x'] → Output: 'zx'
```


---

8. Interview Questions and Answers

## Q&A

**Q: How do you extract ordering edges from adjacent words?**
A: Compare position by position. The first position where they differ gives `word1[j] → word2[j]` (word1's character comes before word2's character in the alien ordering). Stop at the first difference.

**Q: What is the prefix violation case?**
A: If `word1 = "abc"` and `word2 = "ab"`, they share the same prefix for the length of word2, but word1 is longer. Since word1 comes first in sorted order, this is impossible — `"abc"` cannot precede `"ab"` in any valid dictionary.

**Q: How do you detect a cycle with Kahn's algorithm?**
A: After BFS, if the result string's length is less than the number of unique characters, some characters were never processed (their in-degree never reached 0 because of a cycle).

**Q: What if multiple valid orderings exist?**
A: Return any one — the problem says "return any valid ordering." Kahn's BFS may give a different result depending on queue order (e.g., using a priority queue gives lexicographically smallest).

**Q: How does this differ from Course Schedule?**
A: Alien Dictionary first extracts the graph from raw data (word comparisons), then performs topological sort. Course Schedule receives the graph directly. The topological sort logic is identical.

**Q: What if all words are the same?**
A: No edges are extracted. All characters have in-degree 0. Any permutation of the characters is a valid answer.


---

9. Summary

## Summary

| Attribute             | Value                                                                      |
| --------------------- | -------------------------------------------------------------------------- |
| **Problem**           | Derive character ordering from a sorted alien word list                    |
| **Optimal Algorithm** | Extract edges from adjacent words + Kahn's BFS Topological Sort            |
| **Time Complexity**   | O(C) — total characters across all words                                   |
| **Space Complexity**  | O(U) — unique characters                                                   |
| **Key Insight**       | Compare adjacent words to find ordering edges; topological sort = answer  |

### Core Pattern

> Implicit graph problem: extract directed constraints from data, build the graph, then apply topological sort. Cycle = impossible answer.

### When to Use This Pattern

- Deriving an ordering from pairwise comparison data.
- Related: Course Schedule II, Task Scheduler, Sequence Reconstruction.

