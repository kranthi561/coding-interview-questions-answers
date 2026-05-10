# Best Time to Buy and Sell Stock

---

<details>
<summary><strong>1. What is meant by Best Time to Buy and Sell Stock?</strong></summary>

## What is Best Time to Buy and Sell Stock?

You are given an array `prices` where `prices[i]` is the price of a stock on day `i`.

**Goal:** Find the maximum profit by choosing a **single day to buy** and a **later day to sell**.

- You must buy **before** you sell.
- If no profit is possible, return `0`.

**Example:**
```
Input:  prices = [7, 1, 5, 3, 6, 4]
Output: 5
Explanation: Buy on day 2 (price=1), sell on day 5 (price=6). Profit = 6-1 = 5.
```

```
Input:  prices = [7, 6, 4, 3, 1]
Output: 0
Explanation: Prices only decrease — no profitable transaction possible.
```

</details>

---

<details>
<summary><strong>2. Clarify Requirements</strong></summary>

## Requirements

### Functional Requirements
- Given array of stock prices per day, find the maximum profit from one buy-sell transaction.
- Buy must come before sell (chronological order).
- Return `0` if no profit is achievable.
- Only **one transaction** allowed (not multiple buy/sell cycles).

### Non-Functional Requirements
| Concern | Expectation |
|---------|-------------|
| **Scale** | Up to 10⁵ prices |
| **Latency** | Single-pass O(n) expected |
| **Correctness** | Return 0 for non-profitable inputs |

### Clarifying Questions to Ask
- Can we make multiple transactions? → No, only one buy-sell pair.
- Can prices be negative? → No, prices are non-negative.
- What if all prices are equal? → Return 0.

</details>

---

<details>
<summary><strong>3. Time and Space Complexity</strong></summary>

## Complexity Analysis

### Approach 1: Brute Force
| | Complexity |
|---|---|
| **Time** | O(n²) — check every buy-sell pair |
| **Space** | O(1) |

### Approach 2: Single Pass (Optimal)
| | Complexity |
|---|---|
| **Time** | O(n) — one iteration |
| **Space** | O(1) — only two variables |

**Why O(n)?** We track the minimum price seen so far. At each step, profit = current price − min price so far. Update max profit if larger.

</details>

---

<details>
<summary><strong>4. Algorithm Choice and Why</strong></summary>

## Algorithm: Single-Pass with Min Tracking

### Options Compared
| Approach | Time | Space | Chosen? |
|----------|------|-------|---------|
| Brute force (all pairs) | O(n²) | O(1) | No |
| **Single pass (min tracking)** | **O(n)** | **O(1)** | **Yes** |

### Why Single Pass?
- We want the maximum of `prices[j] - prices[i]` where `j > i`.
- Equivalently: for each day `j` (sell day), find the minimum price in `prices[0..j-1]` (best buy day).
- We can track the running minimum in a single pass — no need to look back.

### Logic
```
minPrice = infinity
maxProfit = 0

for each price:
    if price < minPrice:
        minPrice = price          ← found a cheaper buy day
    elif price - minPrice > maxProfit:
        maxProfit = price - minPrice  ← found a better profit
```

</details>

---

<details>
<summary><strong>5. High-Level Design</strong></summary>

## High-Level Design

### Components
1. **Input** — Array of daily prices
2. **Min Price Tracker** — Maintains the lowest price seen so far
3. **Max Profit Tracker** — Maintains the best profit seen so far
4. **Output** — Maximum profit (or 0)

### Data Flow Diagram
```
prices = [7, 1, 5, 3, 6, 4]

Day 0: price=7  → minPrice=7,  maxProfit=0
Day 1: price=1  → minPrice=1,  maxProfit=0   (new min, no profit yet)
Day 2: price=5  → minPrice=1,  maxProfit=4   (5-1=4)
Day 3: price=3  → minPrice=1,  maxProfit=4   (3-1=2, not better)
Day 4: price=6  → minPrice=1,  maxProfit=5   (6-1=5, new best!)
Day 5: price=4  → minPrice=1,  maxProfit=5   (4-1=3, not better)

Result: 5
```

### Visual
```
Price: 7   1   5   3   6   4
           ↑           ↑
         buy=1       sell=6  → profit=5
```

</details>

---

<details>
<summary><strong>6. Java Solution with Explanation</strong></summary>

## Java Implementation

```java
public class BestTimeToBuyAndSellStock {

    /**
     * Finds the maximum profit from a single buy-sell transaction.
     * Tracks the minimum price seen so far and computes potential profit at each step.
     *
     * @param prices array of stock prices indexed by day
     * @return maximum profit possible, or 0 if no profit
     */
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;

        for (int price : prices) {
            if (price < minPrice) {
                // Found a cheaper day to buy
                minPrice = price;
            } else if (price - minPrice > maxProfit) {
                // Found a better sell day relative to current min buy
                maxProfit = price - minPrice;
            }
        }

        return maxProfit;
    }

    // ---- Main: Example usage ----
    public static void main(String[] args) {
        BestTimeToBuyAndSellStock solution = new BestTimeToBuyAndSellStock();

        // Example 1: Profitable
        int[] prices1 = {7, 1, 5, 3, 6, 4};
        System.out.println("Input: [7,1,5,3,6,4]");
        System.out.println("Output: " + solution.maxProfit(prices1)); // 5

        // Example 2: No profit (decreasing prices)
        int[] prices2 = {7, 6, 4, 3, 1};
        System.out.println("\nInput: [7,6,4,3,1]");
        System.out.println("Output: " + solution.maxProfit(prices2)); // 0

        // Example 3: Single element
        int[] prices3 = {5};
        System.out.println("\nInput: [5]");
        System.out.println("Output: " + solution.maxProfit(prices3)); // 0
    }
}
```

### Output
```
Input: [7,1,5,3,6,4]
Output: 5

Input: [7,6,4,3,1]
Output: 0

Input: [5]
Output: 0
```

</details>

---

<details>
<summary><strong>7. Python Solution with Explanation</strong></summary>

## Python Implementation

```python
def max_profit(prices: list[int]) -> int:
    """
    Find maximum profit from one buy-sell stock transaction.

    Strategy: Single pass, track the minimum price seen.
    For each price, compute profit if sold today vs min buy price so far.

    Args:
        prices: List of daily stock prices

    Returns:
        Maximum profit, or 0 if no profitable transaction exists

    Time:  O(n)
    Space: O(1)
    """
    min_price = float('inf')
    max_profit = 0

    for price in prices:
        if price < min_price:
            min_price = price                    # Update cheapest buy day
        elif price - min_price > max_profit:
            max_profit = price - min_price       # Update best profit

    return max_profit


# ---- Example usage ----
if __name__ == "__main__":
    # Example 1: Profitable
    prices1 = [7, 1, 5, 3, 6, 4]
    print(f"Input: {prices1}")
    print(f"Output: {max_profit(prices1)}")  # 5

    # Example 2: Decreasing — no profit
    prices2 = [7, 6, 4, 3, 1]
    print(f"\nInput: {prices2}")
    print(f"Output: {max_profit(prices2)}")  # 0

    # Example 3: All same
    prices3 = [3, 3, 3]
    print(f"\nInput: {prices3}")
    print(f"Output: {max_profit(prices3)}")  # 0
```

### Output
```
Input: [7, 1, 5, 3, 6, 4]
Output: 5

Input: [7, 6, 4, 3, 1]
Output: 0

Input: [3, 3, 3]
Output: 0
```

</details>

---

<details>
<summary><strong>8. Interview Q&A</strong></summary>

## Q&A

**Q: Why can't we use the global minimum and global maximum directly?**  
A: The minimum must come before the maximum in time (buy before sell). The global min might be after the global max. Example: `[3, 1]` — min=1 at day 1, max=3 at day 0, but we can't sell before we buy.

**Q: How does this differ from the "multiple transactions" variant?**  
A: In multiple transactions (LeetCode 122), you can buy and sell repeatedly. The greedy approach adds all upward slopes. This problem restricts to one transaction.

**Q: What if the array has only one element?**  
A: No transaction can be made, so return 0. The loop runs once but `price - minPrice = 0`.

**Q: Is this a dynamic programming problem?**  
A: It can be framed as DP (`dp[i] = max profit ending on day i`), but the greedy/single-pass approach is simpler and equivalent here.

**Q: What happens with negative prices (hypothetically)?**  
A: The logic still works — `minPrice` would track the most negative, and profit would be calculated correctly. The constraint `prices[i] >= 0` in LeetCode is a simplification.

</details>

---

<details>
<summary><strong>9. Summary</strong></summary>

## Summary

| Attribute | Value |
|-----------|-------|
| **Problem** | Maximum profit from one stock buy-sell |
| **Optimal Algorithm** | Single-pass with min price tracking |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(1) |
| **Key Insight** | At each step, profit = current price − min price seen so far |

### Core Pattern
> Track a "best so far" metric (min buy price) as you iterate. At each step compute the best outcome if you acted today.

### When to Use This Pattern
- Problems that ask for max/min difference with ordering constraint
- Sliding window minimum/maximum variants
- Any "best decision up to this point" problems

</details>
