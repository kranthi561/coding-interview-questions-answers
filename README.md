# Coding Interview Questions And Answers

---

## 1. Array (10)

1. [Two Sum](Arrays/two-sum.md)
   Given an array and a target, return indices of two numbers that add up to the target.
   Classic one-pass hash map problem — store each value's index and check for its complement on every step.

2. [Best Time to Buy and Sell Stock](Arrays/best-time-to-buy-and-sell-stock.md)
   Find the maximum profit from a single buy-sell stock transaction.
   Single pass tracking the minimum price seen so far; profit at each step = current price − running minimum.

3. [Contains Duplicate](Arrays/contains-duplicate.md)
   Determine if any value appears more than once in an array.
   Insert elements into a hash set; if insertion fails the element is already present — return true immediately.

4. [Product of Array Except Self](Arrays/product-of-array-except-self.md)
   Return an array where each index holds the product of all other elements, without using division.
   Two-pass prefix/suffix product approach: first pass builds left products, second pass multiplies in right products.

5. [Maximum Subarray](Arrays/maximum-subarray.md)
   Find the contiguous subarray that has the largest sum.
   Kadane's algorithm: at each position decide to extend the current subarray or start fresh — O(n) time, O(1) space.

6. [Maximum Product Subarray](Arrays/maximum-product-subarray.md)
   Find the contiguous subarray that has the largest product.
   Track both the current maximum and minimum product; a large negative minimum can become the new maximum when multiplied by another negative.

7. [Find Minimum in Rotated Sorted Array](Arrays/find-minimum-in-rotated-sorted-array.md)
   Locate the minimum element in a sorted array rotated at an unknown pivot, in O(log n) time.
   Binary search compares the mid element to the right boundary; if mid > right, the minimum is in the right half.

8. [Search in Rotated Sorted Array](Arrays/search-in-rotated-sorted-array.md)
   Find the index of a target value in a rotated sorted array or return -1, in O(log n) time.
   Modified binary search identifies the sorted half at each step and checks whether the target falls within it.

9. [3Sum](Arrays/3sum.md)
   Find all unique triplets in an array that sum to zero.
   Sort the array, fix one element per outer loop, then use two pointers on the remainder for an O(n²) solution with duplicate skipping.

10. [Container With Most Water](Arrays/container-with-most-water.md)
    Find two lines that form the container that holds the most water.
    Two-pointer technique starts at the widest span and always moves the shorter pointer inward — the only direction area can possibly increase.

---

## 2. Binary (5)

1. [Sum of Two Integers](Binary/sum-of-two-integers.md)
   Calculate the sum of two integers without using the `+` or `-` operators.
   XOR computes sum bits without carry; AND shifted left computes the carry; repeat until carry is zero.

2. [Number of 1 Bits](Binary/number-of-1-bits.md)
   Count the number of set bits (1s) in the binary representation of an unsigned integer.
   `n & (n - 1)` clears the lowest set bit each iteration; count how many times this can be done before n reaches zero.

3. [Counting Bits](Binary/counting-bits.md)
   Return an array where each index `i` holds the count of 1-bits in the binary representation of `i`, for all i from 0 to n.
   DP relation: `bits[i] = bits[i >> 1] + (i & 1)` — reuse the count of i shifted right, then add the last bit.

4. [Missing Number](Binary/missing-number.md)
   Find the one missing number in an array containing n distinct values from the range [0, n].
   Gauss formula gives the expected sum `n*(n+1)/2`; subtract the actual array sum to find the missing value.

5. [Reverse Bits](Binary/reverse-bits.md)
   Reverse the bits of a 32-bit unsigned integer.
   Process each bit from right to left: shift the result left by 1, OR in the current bit, then shift n right by 1 — repeat 32 times.

---

## 3. Dynamic Programming (11)

1. [Climbing Stairs](Dynamic-Programming/01-climbing-stairs.md)
   Count the number of distinct ways to climb n stairs when you can take 1 or 2 steps at a time.
   This is the Fibonacci sequence: `dp[i] = dp[i-1] + dp[i-2]` because you arrive at step i from either i-1 or i-2.

2. [Coin Change](Dynamic-Programming/02-coin-change.md)
   Find the minimum number of coins needed to make up a given amount using coins of given denominations.
   Bottom-up DP: `dp[amount] = min(dp[amount - coin] + 1)` for each coin, building solutions from amount 0 upward.

3. [Longest Increasing Subsequence](Dynamic-Programming/03-longest-increasing-subsequence.md)
   Find the length of the longest strictly increasing subsequence in an array.
   O(n²) DP: for each element check all prior elements; O(n log n) variant uses patience sorting with binary search.

4. [Longest Common Subsequence](Dynamic-Programming/04-longest-common-subsequence.md)
   Find the length of the longest subsequence common to two strings.
   2D DP table: `dp[i][j] = dp[i-1][j-1] + 1` if characters match, otherwise `max(dp[i-1][j], dp[i][j-1])`.

5. [Word Break](Dynamic-Programming/05-word-break.md)
   Determine if a string can be segmented into one or more dictionary words.
   DP: `dp[i] = true` if any split point `j < i` has `dp[j] = true` and `s[j..i]` is a dictionary word.

6. [Combination Sum](Dynamic-Programming/06-combination-sum.md)
   Find all combinations of candidates that sum to a target value, reusing elements as many times as needed.
   Backtracking explores each candidate from the current index onward, reducing the remaining target until zero or negative.

7. [House Robber](Dynamic-Programming/07-house-robber.md)
   Maximize the money robbed from houses in a row without robbing two adjacent houses.
   DP: `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — at each house either skip it or take it plus the best two houses back.

8. [House Robber II](Dynamic-Programming/08-house-robber-ii.md)
   Same as House Robber but houses form a circle, so first and last are adjacent.
   Run the linear House Robber algorithm twice: once on `nums[0..n-2]` and once on `nums[1..n-1]`; return the larger result.

9. [Decode Ways](Dynamic-Programming/09-decode-ways.md)
   Count the number of ways to decode a digit string into letters (A=1 … Z=26).
   DP: `dp[i]` accumulates decodings where the current digit is valid alone and/or forms a valid two-digit code with the prior digit.

10. [Unique Paths](Dynamic-Programming/10-unique-paths.md)
    Count all unique paths from the top-left to bottom-right corner of an m×n grid moving only right or down.
    DP: each cell's path count = paths from above + paths from left; `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.

11. [Jump Game](Dynamic-Programming/11-jump-game.md)
    Determine if you can reach the last index starting from the first, given each element's maximum jump length.
    Greedy: track the furthest reachable index; if you reach a position beyond it at any point, return false.

---

## 4. Graph (8)

1. [Clone Graph](Graph/01-clone-graph.md)
   Deep copy a connected undirected graph where each node has a value and a list of neighbors.
   BFS/DFS with a hash map records original → clone mappings and connects cloned neighbors as it traverses.

2. [Course Schedule](Graph/02-course-schedule.md)
   Given course prerequisites, determine if it is possible to finish all courses (detect a directed cycle).
   Topological sort via DFS with three states (unvisited, visiting, visited) detects back edges that form cycles.

3. [Pacific Atlantic Water Flow](Graph/03-pacific-atlantic-water-flow.md)
   Find all grid cells from which water can flow to both the Pacific (top/left) and Atlantic (bottom/right) oceans.
   BFS/DFS inward from each ocean's border marks reachable cells; the answer is the intersection of both sets.

4. [Number of Islands](Graph/04-number-of-islands.md)
   Count the number of islands (connected groups of '1's) in a 2D binary grid.
   DFS/BFS from each unvisited '1' floods the entire island by marking cells as visited, incrementing the count per flood.

5. [Longest Consecutive Sequence](Graph/05-longest-consecutive-sequence.md)
   Find the length of the longest consecutive integer sequence in an unsorted array, in O(n) time.
   Insert all values into a hash set; only start counting sequences from elements with no left neighbor (`n-1` not in set).

6. [Alien Dictionary](Graph/06-alien-dictionary.md)
   Derive the character ordering of an alien language from a list of words sorted in that language's order.
   Compare adjacent words to extract ordering edges, build a directed graph, then topologically sort to get the character order.

7. [Graph Valid Tree](Graph/07-graph-valid-tree.md)
   Determine whether n nodes and a list of edges form a valid tree (connected and acyclic).
   A valid tree has exactly n-1 edges; Union-Find or DFS verifies no cycle exists and all nodes are reachable.

8. [Number of Connected Components in an Undirected Graph](Graph/08-number-of-connected-components.md)
   Count the number of distinct connected components in an undirected graph with n nodes.
   Union-Find merges nodes sharing an edge; the number of remaining distinct roots equals the component count.

---

## 5. Interval (5)

1. [Insert Interval](Interval/01-insert-interval.md)
   Insert a new interval into a sorted list of non-overlapping intervals and merge any overlaps.
   Collect all intervals that end before the new one starts, merge all that overlap, then append the rest.

2. [Merge Intervals](Interval/02-merge-intervals.md)
   Merge all overlapping intervals in an unsorted list and return the merged result.
   Sort by start time; if the current interval's start ≤ last merged interval's end, extend the end; otherwise add a new interval.

3. [Non-overlapping Intervals](Interval/03-non-overlapping-intervals.md)
   Find the minimum number of intervals to remove so that no two intervals overlap.
   Greedy: sort by end time and always keep the interval ending earliest, removing ones that conflict with it.

4. [Meeting Rooms](Interval/04-meeting-rooms.md)
   Determine if a person can attend all meetings given their start and end times.
   Sort meetings by start time and check whether any consecutive pair overlaps (`meetings[i].start < meetings[i-1].end`).

5. [Meeting Rooms II](Interval/05-meeting-rooms-ii.md)
   Find the minimum number of conference rooms required to hold all meetings.
   Use a min-heap of end times; for each new meeting, reuse a room if the earliest-ending room is already free.

---

## 6. Linked List (6)

1. **Reverse Linked List**
   Reverse a singly linked list in place and return the new head.
   Iterative: use three pointers (prev, current, next) to redirect each node's next pointer backward as you traverse.

2. **Linked List Cycle**
   Detect whether a linked list contains a cycle.
   Floyd's cycle detection: a slow pointer (1 step) and fast pointer (2 steps) will eventually meet if and only if a cycle exists.

3. **Merge Two Sorted Lists**
   Merge two sorted linked lists into a single sorted linked list.
   Compare current heads, attach the smaller node to the result, and advance that pointer — repeat until one list is exhausted.

4. **Merge k Sorted Lists**
   Merge k sorted linked lists into one sorted linked list.
   Min-heap (priority queue) holds the current head of each list; repeatedly extract the minimum and enqueue the next node from that list.

5. **Remove Nth Node From End**
   Remove the nth node from the end of a linked list in a single pass.
   Two-pointer trick: advance the first pointer n steps ahead, then move both together until the first reaches the end.

6. **Reorder List**
   Reorder a linked list as L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → ...
   Split the list at the midpoint, reverse the second half, then interleave nodes from both halves alternately.

---

## 7. Matrix (4)

1. **Set Matrix Zeroes**
   Given a matrix, set an entire row and column to zero if any element in that row or column is zero, in place.
   Use the first row and first column as zero-markers to record which rows/columns need zeroing, avoiding O(mn) extra space.

2. **Spiral Matrix**
   Return all elements of an m×n matrix traversed in spiral (clockwise) order.
   Maintain four boundaries (top, bottom, left, right) and shrink them inward after traversing each side.

3. **Rotate Image**
   Rotate an n×n matrix 90 degrees clockwise in place.
   Transpose the matrix by swapping `matrix[i][j]` with `matrix[j][i]`, then reverse each row.

4. **Word Search**
   Check if a given word exists in a 2D character grid using adjacent (up/down/left/right) cells.
   DFS with backtracking starts from each matching cell and explores all paths, marking cells visited and restoring them on backtrack.

---

## 8. String (10)

1. **Longest Substring Without Repeating Characters**
   Find the length of the longest substring that contains no duplicate characters.
   Sliding window with a hash set: expand right pointer freely, shrink left pointer when a duplicate is detected.

2. **Longest Repeating Character Replacement**
   Find the longest substring achievable by replacing at most k characters to make all characters the same.
   Sliding window: a window is valid when `(window length − max character frequency) ≤ k`; expand right and shrink left as needed.

3. **Minimum Window Substring**
   Find the smallest substring of s that contains every character of t.
   Sliding window expands to cover all required characters, then contracts from the left to find the minimum valid window.

4. **Valid Anagram**
   Determine whether two strings are anagrams of each other (same characters, same counts).
   Count character frequencies of both strings with a hash map (or frequency array); they must be identical.

5. **Group Anagrams**
   Group an array of strings by their anagram equivalence class.
   Use the sorted version of each string (or a frequency tuple) as a hash map key; strings sharing the same key are anagrams.

6. **Valid Parentheses**
   Check whether a string of brackets is validly opened and closed in correct order.
   Stack-based: push every open bracket; on a close bracket verify it matches the top of the stack and pop it.

7. **Valid Palindrome**
   Determine if a string is a palindrome ignoring non-alphanumeric characters and case.
   Two-pointer approach from both ends, skipping non-alphanumeric characters, comparing characters case-insensitively.

8. **Longest Palindromic Substring**
   Find the longest palindromic substring within a given string.
   Expand around each of the 2n-1 possible centers (each character and each gap) tracking the widest palindrome found.

9. **Palindromic Substrings**
   Count the total number of palindromic substrings in a string.
   Expand around each center (2n-1 centers) and count every valid palindrome found, including single characters.

10. **Encode and Decode Strings**
    Design functions to encode a list of strings into a single string and decode it back losslessly.
    Prefix each string with its character length and a separator (e.g., `"4#word"`); decoding reads the prefix to extract each string exactly.

---

## 9. Tree (14)

1. **Maximum Depth of Binary Tree**
   Find the maximum depth (number of nodes along the longest root-to-leaf path) of a binary tree.
   Recursive DFS: `depth = 1 + max(depth(left), depth(right))`; base case returns 0 for null nodes.

2. **Same Tree**
   Determine if two binary trees are structurally identical with the same node values at every position.
   Recursive comparison: both null → true; one null → false; values differ → false; recurse on both children.

3. **Invert Binary Tree**
   Produce the mirror image of a binary tree by swapping left and right children at every node.
   Recursive: swap left and right children at the current node, then recursively invert both subtrees.

4. **Binary Tree Maximum Path Sum**
   Find the maximum sum of any path between any two nodes in a binary tree (path need not pass through root).
   DFS computes the best single-arm gain from each node; at each step update a global max that includes both arms crossing the node.

5. **Binary Tree Level Order Traversal**
   Return node values grouped by level, from root to leaves.
   BFS with a queue: process all nodes at the current level, collect their values, then enqueue their children for the next level.

6. **Serialize and Deserialize Binary Tree**
   Convert a binary tree to a string and reconstruct the original tree from that string.
   Pre-order DFS serializes node values with null markers; deserialization replays the same pre-order sequence to rebuild the tree.

7. **Subtree of Another Tree**
   Check if a given tree `subRoot` is an identical subtree of tree `root`.
   DFS visits every node of root; at each node call `isSameTree(node, subRoot)` to check for an exact structural match.

8. **Construct Binary Tree from Preorder and Inorder Traversal**
   Rebuild a binary tree uniquely given its preorder and inorder traversal sequences.
   The first preorder element is always the root; its position in inorder splits the array into left and right subtrees — recurse.

9. **Validate Binary Search Tree**
   Verify that a binary tree satisfies the BST property: left subtree values < node < right subtree values, for all nodes.
   DFS propagates valid (min, max) bounds; if any node's value falls outside its bounds the tree is invalid.

10. **Kth Smallest Element in a BST**
    Find the kth smallest value in a Binary Search Tree.
    In-order traversal (left → node → right) visits BST nodes in ascending order; decrement a counter and return at k = 0.

11. **Lowest Common Ancestor of a BST**
    Find the lowest (deepest) common ancestor of two given nodes in a BST.
    Navigate left if both targets are smaller than current, right if both are larger; otherwise the current node is the LCA.

12. **Implement Trie (Prefix Tree)**
    Build a trie supporting `insert`, `search` (exact), and `startsWith` (prefix) operations.
    Each node holds a children map (character → node) and an `isEndOfWord` flag; insert/search traverse character by character.

13. **Design Add and Search Words Data Structure**
    Support `addWord` and `searchWord` where `'.'` in a search pattern matches any single character.
    Trie with DFS for wildcard: when `'.'` is encountered, recursively try all existing children nodes.

14. **Word Search II**
    Find all words from a given list that exist as paths in a 2D character grid.
    Build a trie from all target words; DFS on the grid prunes entire branches early using the trie, far faster than searching each word separately.

---

## 10. Heap (3)

1. **Top K Frequent Elements**
   Return the k most frequently occurring elements in an integer array.
   Count frequencies with a hash map, then use a min-heap of size k — evict the lowest-frequency element whenever the heap exceeds k.

2. **Find Median from Data Stream**
   Design a data structure that supports adding integers one at a time and querying the current median efficiently.
   Maintain a max-heap for the lower half and a min-heap for the upper half; rebalance after each insertion so sizes differ by at most one.
