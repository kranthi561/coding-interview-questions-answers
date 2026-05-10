# Coin Change

---

<details>
<summary><strong>1. What is meant by Coin Change?</strong></summary>

## What is Coin Change?

**Coin Change** is a classic DP problem where:

- You are given an array of coin denominations `coins[]`.
- You have an infinite supply of each coin.
- You are given a target `amount`.

**Goal:** Return the **minimum number of coins** needed to make up `amount`. If it's impossible, return `-1`.

**Constraints:**
- `1 <= coins.length <= 12`
- `1 <= coins[i] <= 2³¹ - 1`
- `0 <= amount <= 10⁴`

**Example:**

```
Input:  coins = [1, 5, 10, 25], amount = 36
Output: 3
Explanation: 25 + 10 + 1 = 36 (3 coins)
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements — Functional and Non-Functional</strong></summary>

## Requirements

### Functional Requirements

- Given coin denominations (unlimited supply), find the minimum number of coins that sum to `amount`.
- Return `-1` if the amount cannot be reached.
- Coins may be reused any number of times.

### Non-Functional Requirements

| Concern         | Expectation                                         |
| --------------- | --------------------------------------------------- |
| **Scale**       | amount up to 10⁴; coins up to 12 denominations      |
| **Latency**     | O(amount × coins) — runs in milliseconds            |
| **Correctness** | Must find global minimum, not greedy local minimum  |
| **Edge Cases**  | amount = 0 → 0 coins; no valid combo → -1           |

### Clarifying Questions to Ask

- Can coins be reused? → Yes, unlimited supply.
- Can the amount be 0? → Yes, return 0.
- Are coin values always positive? → Yes.
- Is greedy always optimal? → No (e.g., coins=[1,3,4], amount=6: greedy picks 4+1+1=3 coins, DP finds 3+3=2 coins).

</details>

---

<details>
<summary><strong>3. Estimate Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Greedy

| | Complexity |
| --- | --- |
| **Time** | O(n log n) for sorting |
| **Space** | O(1) |
| **Correct?** | No — greedy fails for many coin sets |

### Approach 2: Brute Force Recursion

| | Complexity |
| --- | --- |
| **Time** | O(coins^(amount/min_coin)) — exponential |
| **Space** | O(amount) — recursion stack |

### Approach 3: Bottom-Up DP (Optimal)

| | Complexity |
| --- | --- |
| **Time** | O(amount × coins.length) |
| **Space** | O(amount) — dp array of size amount+1 |

**Why O(amount × coins)?** For each amount from 1 to target, we try every coin — two nested loops.

</details>

---

<details>
<summary><strong>4. Which Algorithm and Why?</strong></summary>

## Algorithm: Bottom-Up Dynamic Programming

### Why Bottom-Up DP?

| Option | Correct? | Time | Space | Chosen? |
| --- | --- | --- | --- | --- |
| Greedy | No | O(n log n) | O(1) | No |
| Brute Recursion | Yes | Exponential | O(n) | No |
| Top-Down Memo | Yes | O(A×C) | O(A) | Possible |
| **Bottom-Up DP** | **Yes** | **O(A×C)** | **O(A)** | **Yes** |

Where A = amount, C = number of coins.

### Key Insight

Build a table `dp[0..amount]` where `dp[i]` = minimum coins to make amount `i`.

- `dp[0] = 0` (0 coins to make amount 0)
- For every amount `i` and every coin `c`: `dp[i] = min(dp[i], dp[i - c] + 1)` if `i >= c`
- At the end, `dp[amount]` holds the answer.

This is an **unbounded knapsack** variant — coins can be reused, so we loop over all amounts and try all coins at each step.

</details>

---

<details>
<summary><strong>5. High-Level Design — Components and Data Flow</strong></summary>

## High-Level Design

### Components

1. **Input Layer** — Receives `coins[]` and `amount`
2. **DP Table Builder** — Fills `dp[0..amount]` iteratively
3. **Subproblem Resolver** — For each amount, tries every coin and picks minimum
4. **Output Layer** — Returns `dp[amount]` or -1 if unreachable

### Data Flow Diagram

```
coins = [1, 5, 10, 25], amount = 11

dp = [0, ∞, ∞, ∞, ∞, ∞, ∞, ∞, ∞, ∞, ∞, ∞]
      0   1   2   3   4   5   6   7   8   9  10  11

Fill dp[1]:  try coin 1: dp[0]+1=1 → dp[1]=1
Fill dp[5]:  try coin 5: dp[0]+1=1 → dp[5]=1
Fill dp[6]:  try coin 1: dp[5]+1=2 → dp[6]=2
Fill dp[10]: try coin 10: dp[0]+1=1 → dp[10]=1
Fill dp[11]: try coin 1: dp[10]+1=2 → dp[11]=2
             try coin 5: dp[6]+1=3 → not better

Final dp = [0,1,2,3,4,1,2,3,4,5,1,2]
Output: dp[11] = 2  (coins: 10 + 1)
```

</details>

---

<details>
<summary><strong>6. Java Solution with Comments and Examples</strong></summary>

## Java Implementation

```java
import java.util.Arrays;

public class CoinChange {

    /**
     * Find minimum coins to make up the target amount.
     *
     * Bottom-up DP: dp[i] = min coins to reach amount i.
     * For each amount, try every coin and take the best.
     *
     * @param coins  available denominations (reusable)
     * @param amount target sum
     * @return minimum number of coins, or -1 if impossible
     *
     * Time:  O(amount * coins.length)
     * Space: O(amount)
     */
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1); // Initialize with "infinity" (impossible marker)
        dp[0] = 0; // Base case: 0 coins needed for amount 0

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    // Use this coin: best way to reach (i - coin) + 1 more coin
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }

        // If dp[amount] is still "infinity", amount is unreachable
        return dp[amount] > amount ? -1 : dp[amount];
    }

    public static void main(String[] args) {
        CoinChange solution = new CoinChange();

        // Example 1: Standard case
        System.out.println("Input: coins=[1,5,10,25], amount=36");
        System.out.println("Output: " + solution.coinChange(new int[]{1, 5, 10, 25}, 36)); // 3

        // Example 2: Greedy would fail here
        System.out.println("\nInput: coins=[1,3,4], amount=6");
        System.out.println("Output: " + solution.coinChange(new int[]{1, 3, 4}, 6)); // 2 (3+3)

        // Example 3: Impossible
        System.out.println("\nInput: coins=[2], amount=3");
        System.out.println("Output: " + solution.coinChange(new int[]{2}, 3)); // -1

        // Example 4: Amount is 0
        System.out.println("\nInput: coins=[1,2,5], amount=0");
        System.out.println("Output: " + solution.coinChange(new int[]{1, 2, 5}, 0)); // 0
    }
}
```

### Output

```
Input: coins=[1,5,10,25], amount=36
Output: 3

Input: coins=[1,3,4], amount=6
Output: 2

Input: coins=[2], amount=3
Output: -1

Input: coins=[1,2,5], amount=0
Output: 0
```

</details>

---

<details>
<summary><strong>7. Python Solution with Comments and Examples</strong></summary>

## Python Implementation

```python
def coin_change(coins: list[int], amount: int) -> int:
    """
    Find minimum coins needed to make up the target amount.

    Strategy: Bottom-up DP.
    dp[i] = minimum coins to make amount i.
    For each amount, try every coin and keep the minimum.

    Args:
        coins: List of coin denominations (reusable)
        amount: Target sum

    Returns:
        Minimum coin count, or -1 if impossible

    Time:  O(amount * len(coins))
    Space: O(amount)
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0  # Base case: 0 coins to make 0

    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1


if __name__ == "__main__":
    # Example 1: Standard
    print(f"coins=[1,5,10,25], amount=36 → {coin_change([1,5,10,25], 36)}")  # 3

    # Example 2: Greedy fails, DP succeeds
    print(f"coins=[1,3,4], amount=6 → {coin_change([1,3,4], 6)}")  # 2 (3+3)

    # Example 3: Impossible
    print(f"coins=[2], amount=3 → {coin_change([2], 3)}")  # -1

    # Example 4: Amount zero
    print(f"coins=[1,2,5], amount=0 → {coin_change([1,2,5], 0)}")  # 0

    # Example 5: Single coin exact match
    print(f"coins=[5], amount=10 → {coin_change([5], 10)}")  # 2
```

### Output

```
coins=[1,5,10,25], amount=36 → 3
coins=[1,3,4], amount=6 → 2
coins=[2], amount=3 → -1
coins=[1,2,5], amount=0 → 0
coins=[5], amount=10 → 2
```

</details>

---

<details>
<summary><strong>8. Interview Questions and Answers</strong></summary>

## Q&A

**Q: Why doesn't greedy work for Coin Change?**
A: Greedy always picks the largest coin first. For `coins=[1,3,4], amount=6`: greedy picks 4+1+1=3 coins. But DP finds 3+3=2 coins. Greedy is only optimal when coin denominations form a canonical system (like US coins).

**Q: What is the time complexity and why?**
A: O(amount × coins). The outer loop runs `amount` times; the inner loop runs `coins.length` times. Total iterations = their product.

**Q: What does `dp[amount+1]` represent initially?**
A: We fill it with a sentinel value (`amount+1` in Java or `inf` in Python) meaning "this amount is unreachable." If it's unchanged at the end, return -1.

**Q: Can you use top-down memoization instead?**
A: Yes. Define `solve(remaining)` recursively, memoize by `remaining`. Both give the same O(A×C) time. Bottom-up avoids recursion stack overhead.

**Q: What if each coin can only be used once (0/1 knapsack variant)?**
A: Change the inner loop direction — iterate amounts from right to left to prevent reuse of the same coin.

**Q: How does this relate to the knapsack problem?**
A: This is the **unbounded knapsack** problem — items (coins) have unlimited supply, and we minimize count (instead of maximizing value).

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute             | Value                                                                  |
| --------------------- | ---------------------------------------------------------------------- |
| **Problem**           | Minimum coins to make a target amount (unlimited coin supply)          |
| **Optimal Algorithm** | Bottom-Up Dynamic Programming (unbounded knapsack)                     |
| **Time Complexity**   | O(amount × coins.length)                                               |
| **Space Complexity**  | O(amount)                                                              |
| **Key Insight**       | `dp[i] = min(dp[i - coin] + 1)` for all coins ≤ i                    |

### Core Pattern

> Fill a DP array from 0 to amount. At each step, "buy" any coin you can afford and check if it improves the minimum count. Greedy is wrong here — always use DP.

### When to Use This Pattern

- Minimum cost to reach a target with unlimited use of items.
- Related: Word Break, Combination Sum, Unbounded Knapsack.
- Any "minimum steps/operations to reach a goal" formulation.

</details>
