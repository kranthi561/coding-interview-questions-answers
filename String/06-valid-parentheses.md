# Valid Parentheses

---

1. What is Valid Parentheses?

## What is Valid Parentheses?

Given a string containing only `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the input string is **valid**.

A string is valid if:
1. Every open bracket is closed by the **same type** of bracket.
2. Open brackets are closed in the **correct (LIFO) order**.
3. Every close bracket has a corresponding open bracket.

**Example:**

```
Input:  s = "()"       → Output: true
Input:  s = "()[]{}"   → Output: true
Input:  s = "(]"       → Output: false
Input:  s = "([)]"     → Output: false
Input:  s = "{[]}"     → Output: true
```


---

2. Clarify Requirements

## Requirements

### Functional Requirements

- Only characters `()[]{}'` appear in the string.
- Return `true` if brackets are properly nested and matched; `false` otherwise.

### Non-Functional Requirements

| Concern         | Expectation                                  |
| --------------- | -------------------------------------------- |
| **Scale**       | String length up to 10,000                   |
| **Latency**     | O(n) — single pass                           |
| **Space**       | O(n) worst case — entire string on the stack |

### Clarifying Questions to Ask

- Can the string be empty? → Yes → return `true`.
- Only bracket characters? → Yes (LeetCode constraint).
- Are unmatched open brackets at the end invalid? → Yes.


---

3. Time and Space Complexity

## Complexity Analysis

### Stack-Based (Optimal)

|           | Complexity                                                      |
| --------- | --------------------------------------------------------------- |
| **Time**  | O(n) — each character pushed or popped at most once             |
| **Space** | O(n) — stack can hold up to n/2 open brackets in the worst case |


---

4. Algorithm Choice and Why

## Algorithm: Stack (LIFO Matching)

### Why Stack?

| Option                  | Time  | Space  | Chosen?                                         |
| ----------------------- | ----- | ------ | ----------------------------------------------- |
| Counter (just counts)   | O(n)  | O(1)   | No — can't distinguish types or nesting order   |
| Recursive               | O(n)  | O(n)   | Possible but awkward and no cleaner than stack  |
| **Stack**               | **O(n)** | **O(n)** | **Yes — naturally models LIFO nesting**      |

### Logic

1. Use a hash map to pair each closing bracket with its corresponding opening bracket.
2. Iterate through the string:
   - If the character is an **opening bracket**, push it onto the stack.
   - If the character is a **closing bracket**:
     - If the stack is empty → unmatched closing bracket → return `false`.
     - If the top of the stack is not the matching opener → mismatch → return `false`.
     - Otherwise, pop the top.
3. After the loop, return `true` only if the stack is **empty** (no unmatched openers).


---

5. High-Level Design

## High-Level Design

### Components

1. **Pair Map** — `{ ')': '(', '}': '{', ']': '[' }` for O(1) match lookup.
2. **Stack** — Holds unmatched opening brackets in order.
3. **Validator** — Checks stack top on each closing bracket; returns empty-stack check at end.

### Data Flow Diagram

```
s = "({[]})"

Stack state after each character:
  '(' → push  → stack: ['(']
  '{' → push  → stack: ['(', '{']
  '[' → push  → stack: ['(', '{', '[']
  ']' → close → top='[' matches ']' ✓ pop → stack: ['(', '{']
  '}' → close → top='{' matches '}' ✓ pop → stack: ['(']
  ')' → close → top='(' matches ')' ✓ pop → stack: []

Stack empty → true ✓

---

s = "([)]"

  '(' → push  → stack: ['(']
  '[' → push  → stack: ['(', '[']
  ')' → close → top='[' does NOT match ')' → false ✗
```


---

6. Java Solution

## Java Implementation

```java
import java.util.Deque;
import java.util.ArrayDeque;
import java.util.Map;

public class ValidParentheses {

    /**
     * Returns true if the bracket string is valid.
     *
     * Push opening brackets; on a closing bracket verify it matches the top
     * of the stack. Valid only if the stack is empty at the end.
     *
     * @param s bracket string containing ()[]{}
     * @return true if all brackets are correctly matched and nested
     */
    public boolean isValid(String s) {
        // Maps each closing bracket to its required opening bracket
        Map<Character, Character> pair = Map.of(
            ')', '(',
            '}', '{',
            ']', '['
        );

        Deque<Character> stack = new ArrayDeque<>();

        for (char c : s.toCharArray()) {
            if (!pair.containsKey(c)) {
                // Opening bracket — push onto stack
                stack.push(c);
            } else {
                // Closing bracket — must match the top of the stack
                if (stack.isEmpty() || stack.peek() != pair.get(c)) {
                    return false;
                }
                stack.pop();
            }
        }

        // Valid only if all open brackets were matched
        return stack.isEmpty();
    }

    public static void main(String[] args) {
        ValidParentheses solution = new ValidParentheses();

        System.out.println(solution.isValid("()"));      // true
        System.out.println(solution.isValid("()[]{}"));  // true
        System.out.println(solution.isValid("(]"));      // false
        System.out.println(solution.isValid("([)]"));    // false
        System.out.println(solution.isValid("{[]}"));    // true
        System.out.println(solution.isValid(""));        // true
        System.out.println(solution.isValid("("));       // false
    }
}
```

### Output

```
true
true
false
false
true
true
false
```


---

7. Python Solution

## Python Implementation

```python
def is_valid(s: str) -> bool:
    """
    Return True if the bracket string has valid, correctly nested brackets.

    Push opening brackets; on a closing bracket, verify it matches the top
    of the stack. Valid only if the stack is empty at the end.

    Args:
        s: string containing only ()[]{}

    Returns:
        True if all brackets are correctly matched

    Time:  O(n)
    Space: O(n)
    """
    pair = {')': '(', '}': '{', ']': '['}
    stack = []

    for c in s:
        if c not in pair:
            stack.append(c)       # opening bracket
        else:
            # closing bracket — must match top of stack
            if not stack or stack[-1] != pair[c]:
                return False
            stack.pop()

    return len(stack) == 0


# ---- Example usage ----
if __name__ == "__main__":
    print(is_valid("()"))      # True
    print(is_valid("()[]{}"))  # True
    print(is_valid("(]"))      # False
    print(is_valid("([)]"))    # False
    print(is_valid("{[]}"))    # True
    print(is_valid(""))        # True
    print(is_valid("("))       # False
```

### Output

```
True
True
False
False
True
True
False
```


---

8. Interview Q&A

## Q&A

**Q: Why do we need a stack instead of just counting?**  
A: A simple counter can't distinguish bracket types or track nesting order. `"(]"` has equal counts but is invalid. The stack captures both type and order.

**Q: What if the string has only closing brackets?**  
A: The stack is empty when we hit the first closing bracket → `stack.isEmpty()` check returns `false` immediately.

**Q: What if the string has only opening brackets?**  
A: They all get pushed. At the end, the stack is non-empty → return `false`.

**Q: Can we solve this with O(1) space?**  
A: Not in general — `"(((((...)))))"` requires tracking depth. You could handle a single bracket type with a counter, but for multiple types you need the stack.

**Q: How would you extend this to validate XML/HTML tags?**  
A: Same stack approach — push tag names on open tags, verify and pop on close tags. The matching condition changes from character comparison to string comparison.


---

9. Summary

## Summary

| Attribute             | Value                                                            |
| --------------------- | ---------------------------------------------------------------- |
| **Problem**           | Check if bracket string is validly nested and matched            |
| **Optimal Algorithm** | Stack-based LIFO matching                                        |
| **Time Complexity**   | O(n)                                                             |
| **Space Complexity**  | O(n)                                                             |
| **Key Insight**       | Push openers, match and pop on closers, empty stack means valid  |

### Core Pattern

> Use a stack to track unmatched openers. Any closer that doesn't match the most recent opener is invalid. Leftover openers at the end are invalid.

### When to Use This Pattern

- Any nested structure validation: brackets, tags, function calls.
- Expression parsing and syntax checking.

