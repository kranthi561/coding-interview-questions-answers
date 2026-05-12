# Encode and Decode Strings

---

1. What is Encode and Decode Strings?

## What is Encode and Decode Strings?

Design an algorithm to **encode** a list of strings into a **single string** and **decode** that single string back into the original list — losslessly, for any input.

**Why is this non-trivial?** Strings can contain any character including the delimiter you might choose. A simple `join("|")` breaks when a string itself contains `"|"`.

**Example:**

```
Input:  ["lint", "code", "love", "you"]
Encoded: "4#lint4#code4#love3#you"
Decoded: ["lint", "code", "love", "you"]

Input:  ["we", "say", ":", "yes"]
Encoded: "2#we3#say1#:3#yes"
Decoded: ["we", "say", ":", "yes"]

Input:  ["", "hello", ""]
Encoded: "0#5#hello0#"
Decoded: ["", "hello", ""]
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- `encode(strs)`: Convert a list of strings into one string.
- `decode(s)`: Convert that string back into the original list.
- Must handle any characters in the strings (including `#`, `|`, spaces, etc.).
- Must handle empty strings in the list.

### Non-Functional Requirements

| Concern         | Expectation                                               |
| --------------- | --------------------------------------------------------- |
| **Scale**       | Up to 200 strings, each up to 200 characters              |
| **Latency**     | O(n) encode and decode, where n = total characters         |
| **Correctness** | `decode(encode(strs)) == strs` for all possible inputs    |

### Clarifying Questions to Ask

- Can strings contain any character? → Yes, including the separator character.
- Can the list be empty? → Yes.
- Can individual strings be empty? → Yes.
- Must the encoding be human-readable? → No — any lossless scheme works.


---

3. Time and Space Complexity

## Complexity Analysis

Let **n** = total number of characters across all strings, **k** = number of strings.

### Length-Prefixed Encoding (Optimal)

|                | Complexity                                             |
| -------------- | ------------------------------------------------------ |
| **Encode Time**| O(n) — visit each character once                       |
| **Decode Time**| O(n) — parse length prefix and extract each string     |
| **Space**      | O(n) — the encoded string holds all characters + prefixes |


---

4. Algorithm Choice and Why

## Algorithm: Length-Prefix Encoding (`len#string`)

### Why This Algorithm?

| Option                        | Problem                                        | Chosen? |
| ----------------------------- | ---------------------------------------------- | ------- |
| Delimiter (e.g. `"\|"`)       | Breaks if strings contain the delimiter         | No      |
| Escaped delimiter             | Complex, error-prone escaping logic             | No      |
| Fixed-width length prefix     | Wastes space for short strings                  | No      |
| **Variable-length prefix `len#`** | Length uniquely locates end of each string | **Yes** |

### Logic

**Encode:**
For each string `s`, prefix it with `len(s)` followed by a separator `#`:
```
encode(["ab", "c#d"]) → "2#ab3#c#d"
```

**Decode:**
Read characters until `#` to get the length `L`, then read exactly `L` characters as the next string. Repeat.

The key insight: by reading exactly `L` characters after `#`, we never confuse a `#` inside a string with the separator.


---

5. High-Level Design

## High-Level Design

### Components

1. **Encoder** — For each string: write `len(s)`, write `#`, write the string's characters.
2. **Decoder** — Scan for `#` to read the length, then slice exactly that many characters.
3. **Length Reader** — Advances an index pointer past digits and the `#` separator.

### Data Flow Diagram

```
Encode: ["lint", "code", "love", "you"]

  "lint" → len=4 → "4#lint"
  "code" → len=4 → "4#code"
  "love" → len=4 → "4#love"
  "you"  → len=3 → "3#you"

Encoded: "4#lint4#code4#love3#you"

---

Decode: "4#lint4#code4#love3#you"

  i=0: digits until '#' → "4", i advances to 2
        read 4 chars: "lint", i=6
  i=6: digits until '#' → "4", i advances to 8
        read 4 chars: "code", i=12
  i=12: "4" → "love", i=18
  i=18: "3" → "you",  i=23

Decoded: ["lint", "code", "love", "you"] ✓
```


---

6. Java Solution

## Java Implementation

```java
import java.util.ArrayList;
import java.util.List;

public class EncodeDecodeStrings {

    /**
     * Encodes a list of strings into a single string.
     * Format: for each string s, write len(s) + "#" + s.
     * The "#" separator is safe because the decoder reads exactly len(s)
     * characters after it, ignoring any "#" inside the string.
     *
     * @param strs list of strings to encode
     * @return encoded string
     */
    public String encode(List<String> strs) {
        StringBuilder sb = new StringBuilder();
        for (String s : strs) {
            sb.append(s.length()).append('#').append(s);
        }
        return sb.toString();
    }

    /**
     * Decodes a single string back into the original list.
     * Reads the length prefix before each '#', then extracts that many chars.
     *
     * @param s encoded string
     * @return original list of strings
     */
    public List<String> decode(String s) {
        List<String> result = new ArrayList<>();
        int i = 0;

        while (i < s.length()) {
            // Find the '#' separator to read the length
            int j = i;
            while (s.charAt(j) != '#') j++;

            int len = Integer.parseInt(s.substring(i, j));

            // Extract exactly len characters after '#'
            result.add(s.substring(j + 1, j + 1 + len));
            i = j + 1 + len;
        }

        return result;
    }

    public static void main(String[] args) {
        EncodeDecodeStrings codec = new EncodeDecodeStrings();

        // Example 1: standard strings
        List<String> input1 = List.of("lint", "code", "love", "you");
        String encoded1 = codec.encode(input1);
        System.out.println("Encoded: " + encoded1);               // 4#lint4#code4#love3#you
        System.out.println("Decoded: " + codec.decode(encoded1)); // [lint, code, love, you]

        // Example 2: strings with special characters
        List<String> input2 = List.of("we", "say", ":", "yes");
        String encoded2 = codec.encode(input2);
        System.out.println("\nEncoded: " + encoded2);             // 2#we3#say1#:3#yes
        System.out.println("Decoded: " + codec.decode(encoded2)); // [we, say, :, yes]

        // Example 3: empty strings
        List<String> input3 = List.of("", "hello", "");
        String encoded3 = codec.encode(input3);
        System.out.println("\nEncoded: '" + encoded3 + "'");      // '0#5#hello0#'
        System.out.println("Decoded: " + codec.decode(encoded3)); // [, hello, ]
    }
}
```

### Output

```
Encoded: 4#lint4#code4#love3#you
Decoded: [lint, code, love, you]

Encoded: 2#we3#say1#:3#yes
Decoded: [we, say, :, yes]

Encoded: '0#5#hello0#'
Decoded: [, hello, ]
```


---

7. Python Solution

## Python Implementation

```python
def encode(strs: list[str]) -> str:
    """
    Encode a list of strings into a single string.
    Format: "{length}#{string}" for each string.
    Reading exactly 'length' characters after '#' safely handles
    any '#' characters within the strings themselves.

    Args:
        strs: list of strings to encode

    Returns:
        Single encoded string

    Time:  O(n) total characters
    Space: O(n)
    """
    return "".join(f"{len(s)}#{s}" for s in strs)


def decode(s: str) -> list[str]:
    """
    Decode a single string back into the original list.
    Reads the length prefix before each '#', then slices exactly that many chars.

    Args:
        s: encoded string

    Returns:
        Original list of strings

    Time:  O(n)
    Space: O(n) for the output list
    """
    result = []
    i = 0

    while i < len(s):
        # Find '#' to determine the length prefix
        j = i
        while s[j] != '#':
            j += 1

        length = int(s[i:j])

        # Extract exactly 'length' characters after '#'
        result.append(s[j + 1: j + 1 + length])
        i = j + 1 + length

    return result


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1
    strs1 = ["lint", "code", "love", "you"]
    enc1 = encode(strs1)
    print(f"Encoded: {enc1}")           # 4#lint4#code4#love3#you
    print(f"Decoded: {decode(enc1)}")   # ['lint', 'code', 'love', 'you']

    # Example 2: special characters
    strs2 = ["we", "say", ":", "yes"]
    enc2 = encode(strs2)
    print(f"\nEncoded: {enc2}")         # 2#we3#say1#:3#yes
    print(f"Decoded: {decode(enc2)}")   # ['we', 'say', ':', 'yes']

    # Example 3: empty strings
    strs3 = ["", "hello", ""]
    enc3 = encode(strs3)
    print(f"\nEncoded: '{enc3}'")       # '0#5#hello0#'
    print(f"Decoded: {decode(enc3)}")   # ['', 'hello', '']
```

### Output

```
Encoded: 4#lint4#code4#love3#you
Decoded: ['lint', 'code', 'love', 'you']

Encoded: 2#we3#say1#:3#yes
Decoded: ['we', 'say', ':', 'yes']

Encoded: '0#5#hello0#'
Decoded: ['', 'hello', '']
```


---

8. Interview Q&A

## Q&A

**Q: Why can't we just use a delimiter like `"|"`?**  
A: If any string in the list contains `"|"`, the split will produce incorrect results. For example, `["a|b", "c"]` encoded as `"a|b|c"` decodes as `["a", "b", "c"]` — wrong.

**Q: Why is `#` safe as the separator here even though strings can contain `#`?**  
A: The decoder doesn't split on `#`. It reads digits until `#` to get the length, then advances exactly that many characters. Any `#` inside the string body is part of the `length` characters extracted — it is never interpreted as a separator.

**Q: What if a string length itself contains `#`, e.g., the digit "3" followed by "#"?**  
A: The length prefix only contains digits (0–9). We scan until we hit the first `#` — which marks the end of the length. Digits cannot be `#`, so there is no ambiguity.

**Q: How would you encode an empty list?**  
A: `encode([])` produces an empty string `""`. `decode("")` returns `[]` because the while loop doesn't execute.

**Q: Is there an alternative approach that doesn't use `#`?**  
A: Yes — use a 4-byte fixed-length header for each string's length (big-endian integer). This avoids the `#` entirely but requires binary encoding. The `len#` approach is simpler and purely text-based.


---

9. Summary

## Summary

| Attribute             | Value                                                                   |
| --------------------- | ----------------------------------------------------------------------- |
| **Problem**           | Losslessly encode/decode a list of arbitrary strings                    |
| **Optimal Algorithm** | Length-prefix encoding: `"len#string"` per entry                        |
| **Encode Time**       | O(n) total characters                                                   |
| **Decode Time**       | O(n)                                                                    |
| **Space**             | O(n)                                                                    |
| **Key Insight**       | Prefix each string with its length so the decoder never needs to guess  |

### Core Pattern

> Self-describing encoding: each record carries its own length. The reader uses the length to know exactly how many bytes to consume, making any byte value safe inside the payload.

### When to Use This Pattern

- Any serialization problem where the alphabet includes every possible character.
- Network protocols, file formats, and RPC frames (TLV — Type-Length-Value encoding).

