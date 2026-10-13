# LeetCode 3727 — Maximum Alternating Sum of Squares

## Problem

You are given an integer array `nums`. You may rearrange it in any order.

The alternating score of an array `arr` is:

```text
score = arr[0]² - arr[1]² + arr[2]² - arr[3]² + ...
```

Even indices are added. Odd indices are subtracted.

Return the **maximum** score over every rearrangement.

Example:

```text
nums   = [1, 2, 3]
order  = [2, 1, 3]
score  = 2² - 1² + 3² = 4 - 1 + 9 = 12

nums   = [1, -1, 2, -2, 3, -3]
score  = 16
```

```text
1 <= nums.length <= 10^5
-4 * 10^4 <= nums[i] <= 4 * 10^4
```

---

# Approach — Sort the Squares

### Core idea

Squaring removes the original sign. `3` and `-3` both contribute `9`. The only choice left is which square gets a plus and which gets a minus.

There are a fixed number of each sign:

```text
plus slots   = ceil(n / 2)     even indices
minus slots  = floor(n / 2)    odd indices
```

A larger square helps more in a plus slot and hurts more in a minus slot. So:

```text
smallest floor(n / 2) squares  →  minus
largest  ceil(n / 2)  squares  →  plus
```

Sort the squares ascending. The first `n / 2` values are the ones to subtract. The rest are the ones to add.

### Walkthrough

```text
nums    = [1, 2, 3]
squares = [1, 4, 9]          already sorted
n / 2   = 1

subtract 1
add      4 + 9
score    = -1 + 13 = 12
```

```text
nums    = [1, -1, 2, -2, 3, -3]
squares = [1, 1, 4, 4, 9, 9]
n / 2   = 3

subtract 1 + 1 + 4 = 6
add      4 + 9 + 9 = 22
score    = 16
```

One arrangement that produces this is `[-3, -1, -2, 1, 3, 2]`. The signs of the originals do not matter. Only the squares do.

### State

```text
sqrArr  : nums[i]² for every i
sum     : plus squares minus the smaller squares
```

### Transitions

```text
square every value
sort ascending
i < n / 2   → sum -= sqrArr[i]
i >= n / 2  → sum += sqrArr[i]
```

---

## Java

```java
class Solution {
    public long maxAlternatingSum(int[] nums) {

        int n = nums.length;
        long[] sqrArr = new long[n];

        for (int i = 0; i < n; i++) {
            sqrArr[i] = (long) nums[i] * nums[i];
        }

        long sum = 0;
        Arrays.sort(sqrArr);

        for (int i = 0; i < n; i++) {

            if (i < (n / 2)) {
                // Smaller squares go into the minus slots.
                sum -= sqrArr[i];
            } else {
                // Larger squares go into the plus slots.
                sum += sqrArr[i];
            }
        }

        return sum;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    long long maxAlternatingSum(vector<int>& nums) {

        int n = nums.size();
        vector<long long> sqrArr(n);

        for (int i = 0; i < n; i++) {
            sqrArr[i] = 1LL * nums[i] * nums[i];
        }

        sort(sqrArr.begin(), sqrArr.end());

        long long sum = 0;

        for (int i = 0; i < n; i++) {

            if (i < n / 2) {
                // Smaller squares go into the minus slots.
                sum -= sqrArr[i];
            } else {
                // Larger squares go into the plus slots.
                sum += sqrArr[i];
            }
        }

        return sum;
    }
};
```

The square is computed in 64-bit arithmetic. Under these constraints one square still fits in a 32-bit `int` (`40000² = 1_600_000_000`), but the total of up to `10^5` squares does not, so the accumulator is 64-bit.

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n log n)    sorting the squares dominates the two linear passes
Space : O(n)          the array of squares
```

---

# Pattern Recognition

```text
Rearrange, then alternate +x² - x² + x² - x²
        ↓
The original sign disappears after squaring
        ↓
floor(n / 2) minus slots, ceil(n / 2) plus slots
        ↓
Sort squares
        ↓
Subtract the small half, add the large half
```

### Remember this trigger:

> **Alternating sum after a free rearrangement → sort, put the largest values on the plus signs and the smallest values on the minus signs.**
