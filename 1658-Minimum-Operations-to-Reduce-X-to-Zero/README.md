# LeetCode 1658 — Minimum Operations to Reduce X to Zero

## Problem

Given an array `nums` and integer `x`, remove elements **only from the left or right**. Each removed element decreases `x`.

Return the **minimum number of operations** required to make `x = 0`.

If impossible, return `-1`.

---

# Approach 1 — Top-Down DP / Memoization

### Core idea

At every step we have two choices:

```text
Take nums[left]
OR
Take nums[right]
```

So the recursion is:

```text
f(left, right, x)
```

However, `x` does **not** need to be part of the DP state.

For a particular `(left, right)`, the elements already removed are uniquely determined. Therefore the remaining `x` is also determined.

So:

```text
dp[left][right]
```

stores the minimum operations needed for that state.

### Java

```java
class Solution {

    static int[][] dp;

    // Large SAFE value.
    // Don't use Integer.MAX_VALUE because 1 + MAX_VALUE overflows.
    static final int INF = 1_000_000;

    public int solve(int[] nums, int x, int left, int right) {

        // We successfully reduced x to zero.
        if (x == 0) {
            return 0;
        }

        // No elements left OR x became negative.
        // This path is impossible.
        if (left > right || x < 0) {
            return INF;
        }

        // Already calculated this state.
        if (dp[left][right] != -1) {
            return dp[left][right];
        }

        // Choice 1: remove from LEFT.
        int takeLeft = 1 + solve(
            nums,
            x - nums[left],
            left + 1,
            right
        );

        // Choice 2: remove from RIGHT.
        int takeRight = 1 + solve(
            nums,
            x - nums[right],
            left,
            right - 1
        );

        // Store the better choice.
        dp[left][right] = Math.min(takeLeft, takeRight);

        return dp[left][right];
    }

    public int minOperations(int[] nums, int x) {

        int n = nums.length;

        dp = new int[n][n];

        // -1 means "state has not been calculated".
        for (int[] row : dp) {
            java.util.Arrays.fill(row, -1);
        }

        int ans = solve(nums, x, 0, n - 1);

        return ans >= INF ? -1 : ans;
    }
}
```

### C++

```cpp
class Solution {

    static const int INF = 1000000;

    vector<vector<int>> dp;

    int solve(vector<int>& nums, int x, int left, int right) {

        // Successfully reduced x to zero.
        if (x == 0) {
            return 0;
        }

        // Impossible state.
        if (left > right || x < 0) {
            return INF;
        }

        // Already calculated.
        if (dp[left][right] != -1) {
            return dp[left][right];
        }

        // Take from LEFT.
        int takeLeft = 1 + solve(
            nums,
            x - nums[left],
            left + 1,
            right
        );

        // Take from RIGHT.
        int takeRight = 1 + solve(
            nums,
            x - nums[right],
            left,
            right - 1
        );

        // Choose minimum operations.
        return dp[left][right] =
            min(takeLeft, takeRight);
    }

public:

    int minOperations(vector<int>& nums, int x) {

        int n = nums.size();

        dp.assign(n, vector<int>(n, -1));

        int ans = solve(nums, x, 0, n - 1);

        return ans >= INF ? -1 : ans;
    }
};
```

### DP Complexity

```text
Time  : O(n²)
Space : O(n²)
```

⚠️ For `n = 10⁵`, this is too large.

---

# Approach 2 — Sliding Window ⭐

This is the optimal approach.

## The Big Transformation

Suppose:

```text
nums = [1, 1, 4, 2, 3]
x = 5
```

Total:

```text
1 + 1 + 4 + 2 + 3 = 11
```

If we remove elements totaling `5`, the elements **remaining** must total:

```text
11 - 5 = 6
```

So instead of:

```text
Find minimum elements to REMOVE with sum x
```

we solve:

```text
Find LONGEST subarray with sum total - x
```

Why longest?

If we keep the longest possible subarray, we remove the fewest elements.

```text
answer = n - longestSubarrayLength
```

---

## Why Sliding Window Works

All numbers are positive.

Therefore:

```text
Expand right → sum increases
Move left    → sum decreases
```

This gives us a monotonic window.

---

## Java

```java
class Solution {

    public int minOperations(int[] nums, int x) {

        int n = nums.length;

        // ----------------------------------
        // Step 1: Calculate total sum
        // ----------------------------------

        int total = 0;

        for (int num : nums) {
            total += num;
        }

        // ----------------------------------
        // Step 2: Find the sum we want
        //
        // Removed sum = x
        // Remaining sum = total - x
        // ----------------------------------

        int target = total - x;

        // Target cannot be negative.
        if (target < 0) {
            return -1;
        }

        // target == 0 means we need to
        // remove the entire array.
        if (target == 0) {
            return n;
        }

        // ----------------------------------
        // Step 3: Sliding Window
        // Find longest subarray
        // whose sum == target
        // ----------------------------------

        int left = 0;
        int sum = 0;
        int maxLen = -1;

        for (int right = 0; right < n; right++) {

            // Expand window.
            sum += nums[right];

            // If sum becomes too large,
            // shrink window from left.
            while (sum > target && left <= right) {
                sum -= nums[left];
                left++;
            }

            // We found a valid window.
            if (sum == target) {

                int currentLength =
                    right - left + 1;

                maxLen = Math.max(
                    maxLen,
                    currentLength
                );
            }
        }

        // No valid subarray exists.
        if (maxLen == -1) {
            return -1;
        }

        // Everything outside the window
        // must be removed.
        return n - maxLen;
    }
}
```

---

## C++

```cpp
class Solution {

public:

    int minOperations(vector<int>& nums, int x) {

        int n = nums.size();

        // ----------------------------------
        // Step 1: Calculate total sum
        // ----------------------------------

        int total = 0;

        for (int num : nums) {
            total += num;
        }

        // ----------------------------------
        // Step 2: Remaining subarray target
        // ----------------------------------

        int target = total - x;

        if (target < 0) {
            return -1;
        }

        // If target == 0,
        // remove everything.
        if (target == 0) {
            return n;
        }

        // ----------------------------------
        // Step 3: Sliding Window
        // ----------------------------------

        int left = 0;
        int sum = 0;
        int maxLen = -1;

        for (int right = 0; right < n; right++) {

            // Expand window.
            sum += nums[right];

            // Shrink if sum is too large.
            while (sum > target && left <= right) {
                sum -= nums[left];
                left++;
            }

            // Valid subarray found.
            if (sum == target) {

                int currentLength =
                    right - left + 1;

                maxLen = max(
                    maxLen,
                    currentLength
                );
            }
        }

        // No valid subarray.
        if (maxLen == -1) {
            return -1;
        }

        // Minimum removals.
        return n - maxLen;
    }
};
```

---

# Pattern Recognition

This problem teaches a **very reusable transformation**:

```text
Remove from both ends
        ↓
Remaining elements are contiguous
        ↓
Removed sum = x
        ↓
Remaining sum = total - x
        ↓
Find LONGEST subarray with that sum
        ↓
Sliding Window
```

### Remember this trigger:

> **Positive array + subarray sum + longest/shortest + large N → think Sliding Window.**

And the DP version is useful for understanding **state + memoization**, while sliding window is the production-grade solution here:

```text
DP              → O(n²) time / O(n²) space
Sliding Window  → O(n)  time / O(1)  space
```
