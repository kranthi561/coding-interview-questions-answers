# Group Anagrams

---

1. What is Group Anagrams?

## What is Group Anagrams?

Given an array of strings, group all strings that are **anagrams** of each other into the same sublist.

Two strings are anagrams if they contain the same characters with the same frequencies.

**Example:**

```
Input:  strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]
(Order within groups and order of groups do not matter.)

Input:  strs = [""]
Output: [[""]]

Input:  strs = ["a"]
Output: [["a"]]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Group strings by their anagram equivalence class.
- Return a list of groups; each group is a list of strings.
- Order within groups and order of groups in the output do not matter.

### Non-Functional Requirements

| Concern         | Expectation                                            |
| --------------- | ------------------------------------------------------ |
| **Scale**       | Up to 10,000 strings, each up to 100 characters        |
| **Latency**     | O(n · k log k) sorting key, or O(n · k) frequency key  |
| **Space**       | O(n · k) — all strings stored in output groups          |

### Clarifying Questions to Ask

- Only lowercase letters? → Yes (LeetCode base).
- Can strings be empty? → Yes — group empty strings together.
- Can the input array be empty? → Yes → return `[]`.


---

3. Time and Space Complexity

## Complexity Analysis

Let **n** = number of strings, **k** = maximum string length.

### Approach 1: Sort Each String as Key

|           | Complexity                                         |
| --------- | -------------------------------------------------- |
| **Time**  | O(n · k log k) — sorting each string of length k   |
| **Space** | O(n · k) — storing all strings in the hash map     |

### Approach 2: Frequency Tuple as Key (Optimal for large alphabets)

|           | Complexity                                               |
| --------- | -------------------------------------------------------- |
| **Time**  | O(n · k) — building frequency count per string           |
| **Space** | O(n · k) — same output storage                           |

For small alphabets (26 letters), both are practical. The sort approach is simpler to write; the frequency approach is asymptotically faster for long strings.


---

4. Algorithm Choice and Why

## Algorithm: Hash Map with Sorted String as Key

### Why This Algorithm?

| Option                        | Time          | Space    | Chosen?                               |
| ----------------------------- | ------------- | -------- | ------------------------------------- |
| Compare all pairs             | O(n² · k)     | O(n)     | No — quadratic                        |
| Sort key per string           | O(n·k log k)  | O(n·k)   | **Yes — simple and fast enough**      |
| Frequency tuple key           | O(n·k)        | O(n·k)   | Alternative; faster for very long strings |

### Logic

1. Create a hash map `{ canonical_key → [list of strings] }`.
2. For each string, compute its **canonical key** — the sorted version of the string (e.g., `"eat"` → `"aet"`).
3. Append the string to the list at `map[key]`.
4. Return all values of the map.

All anagrams share the same sorted form → they map to the same key → they end up in the same list.


---

5. High-Level Design

## High-Level Design

### Components

1. **Key Generator** — Sorts each string to produce a canonical key.
2. **Group Map** — Hash map from canonical key to list of original strings.
3. **Result Collector** — Returns all map values.

### Data Flow Diagram

```
Input: ["eat","tea","tan","ate","nat","bat"]

Process each string:
  "eat" → sort → "aet"  → map["aet"] = ["eat"]
  "tea" → sort → "aet"  → map["aet"] = ["eat","tea"]
  "tan" → sort → "ant"  → map["ant"] = ["tan"]
  "ate" → sort → "aet"  → map["aet"] = ["eat","tea","ate"]
  "nat" → sort → "ant"  → map["ant"] = ["tan","nat"]
  "bat" → sort → "abt"  → map["abt"] = ["bat"]

Return map values:
  [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```


---

6. Java Solution

## Java Implementation

```java
import java.util.*;

public class GroupAnagrams {

    /**
     * Groups strings that are anagrams of each other.
     *
     * For each string, sort its characters to produce a canonical key.
     * All anagrams share the same key and are collected in the same list.
     *
     * @param strs array of strings
     * @return list of anagram groups
     */
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();

        for (String s : strs) {
            // Sort characters to produce canonical key
            char[] chars = s.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            // Add to the corresponding group
            map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        }

        return new ArrayList<>(map.values());
    }

    public static void main(String[] args) {
        GroupAnagrams solution = new GroupAnagrams();

        // Example 1
        String[] strs1 = {"eat","tea","tan","ate","nat","bat"};
        System.out.println(solution.groupAnagrams(strs1));
        // [["eat","tea","ate"],["tan","nat"],["bat"]] (order may vary)

        // Example 2
        String[] strs2 = {""};
        System.out.println(solution.groupAnagrams(strs2));
        // [[""]]

        // Example 3
        String[] strs3 = {"a"};
        System.out.println(solution.groupAnagrams(strs3));
        // [["a"]]
    }
}
```

### Output

```
[[eat, tea, ate], [tan, nat], [bat]]
[[]]
[[a]]
```


---

7. Python Solution

## Python Implementation

```python
from collections import defaultdict

def group_anagrams(strs: list[str]) -> list[list[str]]:
    """
    Group strings that are anagrams of each other.

    Canonical key: sorted string. All anagrams map to the same key.

    Args:
        strs: list of strings

    Returns:
        List of anagram groups

    Time:  O(n · k log k)  n = number of strings, k = max string length
    Space: O(n · k)
    """
    groups = defaultdict(list)

    for s in strs:
        key = tuple(sorted(s))   # tuple is hashable; sorted gives canonical form
        groups[key].append(s)

    return list(groups.values())


# ---- Alternative: frequency-tuple key (O(n·k) time) ----
def group_anagrams_freq(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for c in s:
            count[ord(c) - ord('a')] += 1
        groups[tuple(count)].append(s)
    return list(groups.values())


# ---- Example usage ----
if __name__ == "__main__":
    print(group_anagrams(["eat","tea","tan","ate","nat","bat"]))
    # [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]

    print(group_anagrams([""]))
    # [['']]

    print(group_anagrams(["a"]))
    # [['a']]
```

### Output

```
[['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
[['']]
[['a']]
```


---

8. Interview Q&A

## Q&A

**Q: Why use the sorted string as the key instead of a frequency map?**  
A: Sorted strings are directly hashable as strings. A frequency map would need to be converted to a tuple or string first. Both work; sorted is simpler to write.

**Q: What if strings contain uppercase or mixed-case characters?**  
A: Normalize to lowercase (`s.toLowerCase()`) before computing the key, if case-insensitive grouping is required.

**Q: Can we use a frequency array as the key in Java?**  
A: Arrays are not hashable by value in Java. Convert to a String like `"1#0#0#..."` or use a `List<Integer>` as key. Python tuples are hashable, making the frequency approach cleaner there.

**Q: What is the time complexity if we use the frequency-tuple key?**  
A: O(n · k) — we count 26 characters per string instead of sorting. For long strings this is faster than O(n · k log k).

**Q: How would you handle Unicode strings?**  
A: Use a `HashMap<Character, Integer>` frequency map, convert to a sorted list of `(char, count)` pairs as the key. The core logic is identical.


---

9. Summary

## Summary

| Attribute             | Value                                                         |
| --------------------- | ------------------------------------------------------------- |
| **Problem**           | Group strings by anagram equivalence class                    |
| **Optimal Algorithm** | Hash map with sorted-string canonical key                     |
| **Time Complexity**   | O(n · k log k) with sort key; O(n · k) with frequency key     |
| **Space Complexity**  | O(n · k)                                                      |
| **Key Insight**       | All anagrams share the same canonical form (sorted string)    |

### Core Pattern

> Reduce each element to its canonical form, then group by that form using a hash map.

### When to Use This Pattern

- Any "group by equivalence" problem (anagrams, isomorphic strings, similar strings).
- When two objects are "equivalent" under some transformation — hash the transformation, not the object.

