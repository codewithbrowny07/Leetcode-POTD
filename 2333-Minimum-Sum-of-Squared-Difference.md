# LeetCode 2333 — Minimum Sum of Squared Difference

## Problem

You are given two arrays `nums1` and `nums2`, both of length `n`, and two integers `k1` and `k2`.

The sum of squared difference is:

```text
Σ (nums1[i] - nums2[i])²
```

You may change any element of `nums1` by `+1` or `-1`, at most `k1` times in total. The same applies to `nums2` with `k2`. Values may become negative.

Return the **minimum** sum of squared difference after those changes.

Example:

```text
nums1 = [1, 4, 10, 12]
nums2 = [5, 8, 6, 9]
k1 = 1, k2 = 1

diffs before = [4, 4, 4, 3]
answer       = 43
```

One optimal change turns the diffs into `[4, 3, 3, 3]`:

```text
4² + 3² + 3² + 3² = 16 + 9 + 9 + 9 = 43
```

---

# Approach — Reduce the Largest Diffs First

### Core idea

Only the **absolute difference** of each pair matters. One operation on either array can shrink one difference by `1`, if you move the larger value down or the smaller value up.

So `k1 + k2` is one shared budget. Each unit of budget subtracts `1` from one difference.

Squaring punishes large numbers more than small ones:

```text
d² - (d - 1)² = 2d - 1
```

A bigger `d` saves more when it is reduced. Always spend the next operation on a current maximum difference.

If the budget is at least the sum of all differences, every difference can be driven to `0`, and the answer is `0`.

### Frequency buckets

Differences are at most `100000`, so count how many pairs have each difference:

```text
freq[d] = number of pairs whose |nums1[i] - nums2[i]| is d
```

Walk from the largest `d` downward. Every pair in `freq[d]` can spend one operation and fall into `freq[d - 1]`.

```text
operations = min(budget, freq[d])
freq[d]     -= operations
freq[d - 1] += operations
budget      -= operations
```

The next loop step is `d - 1`, so those pairs are reduced again if budget remains. That is the same as repeatedly trimming the current largest difference.

### Walkthrough

```text
diffs  = [4, 4, 4, 3]
budget = 2

freq[4] = 3
freq[3] = 1

d = 4, spend 2
freq[4] = 1
freq[3] = 3
budget  = 0

answer = 1·4² + 3·3² = 43
```

### State

```text
freq    : count of each difference
total   : sum of differences, used for the all-zero check
budget  : k1 + k2 operations still available
```

### Transitions

```text
budget >= total     → return 0
otherwise, for d from large to small:
    move min(budget, freq[d]) pairs from d down to d - 1
answer = Σ d² · freq[d]
```

---

## Java

```java
class Solution {
    public long minSumSquareDiff(int[] nums1, int[] nums2, int k1, int k2) {

        int n = nums1.length;
        int[] freq = new int[100001];
        long total = 0;

        for (int i = 0; i < n; i++) {
            int diff = Math.abs(nums1[i] - nums2[i]);
            freq[diff]++;
            total += diff;
        }

        long budget = (long) k1 + k2;

        // Enough operations to erase every difference.
        if (budget >= total) {
            return 0;
        }

        // Always shrink the current largest difference by 1.
        for (int d = 100000; d > 0 && budget > 0; d--) {

            if (freq[d] == 0) {
                continue;
            }

            int count = freq[d];
            long operations = Math.min(budget, (long) count);

            freq[d] -= (int) operations;
            freq[d - 1] += (int) operations;
            budget -= operations;
        }

        long answer = 0;

        for (int d = 1; d <= 100000; d++) {
            answer += (long) d * d * freq[d];
        }

        return answer;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    long long minSumSquareDiff(vector<int>& nums1, vector<int>& nums2, int k1, int k2) {

        int n = nums1.size();
        vector<int> freq(100001, 0);
        long long total = 0;

        for (int i = 0; i < n; i++) {
            int diff = abs(nums1[i] - nums2[i]);
            freq[diff]++;
            total += diff;
        }

        long long budget = (long long) k1 + k2;

        // Enough operations to erase every difference.
        if (budget >= total) {
            return 0;
        }

        // Always shrink the current largest difference by 1.
        for (int d = 100000; d > 0 && budget > 0; d--) {

            if (freq[d] == 0) {
                continue;
            }

            int count = freq[d];
            long long operations = min(budget, (long long) count);

            freq[d] -= (int) operations;
            freq[d - 1] += (int) operations;
            budget -= operations;
        }

        long long answer = 0;

        for (int d = 1; d <= 100000; d++) {
            answer += (long long) d * d * freq[d];
        }

        return answer;
    }
};
```

---

# Complexity

Let `n` be the array length and `D = 100000`, the largest possible difference.

```text
Time  : O(n + D)    one pass over the pairs, then one pass over the difference buckets
Space : O(D)        the frequency array
```

`k1` and `k2` can each be `10^9`, and the sum of differences can be up to `n · D`. Both `budget` and the squared sum are accumulated in 64-bit integers. `d * d` is computed after the cast to 64-bit, so the square does not overflow a 32-bit `int`.

---

# Pattern Recognition

```text
Minimize a sum of squares with a limited number of ±1 edits
        ↓
Each edit can reduce one absolute difference by 1
        ↓
k1 + k2 is one budget
        ↓
Reducing a larger value saves more, because d² - (d - 1)² = 2d - 1
        ↓
Count differences, then pour the budget from the largest bucket downward
```

### Remember this trigger:

> **A fixed number of ±1 changes, and the cost is a square → spend every operation on the current largest value. A frequency array is enough when the value range is small.**
