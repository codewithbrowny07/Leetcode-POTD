# LeetCode 3411 — Maximum Subarray With Equal Products

## Problem

You are given an array of positive integers `nums`.

An array `arr` is **product equivalent** when

```text
prod(arr) == lcm(arr) * gcd(arr)
```

Here `prod` is the product of all elements, `lcm` is their least common multiple, and `gcd` is their greatest common divisor.

Return the length of the **longest** product equivalent subarray of `nums`.

Example:

```text
nums = [1, 2, 1, 2, 1, 1, 1]

[1, 2, 1, 1, 1]
prod = 2    gcd = 1    lcm = 2    2 == 1 * 2

answer = 5
```

```text
nums = [2, 3, 4, 5, 6]

[3, 4, 5]
prod = 60   gcd = 1    lcm = 60   60 == 1 * 60

answer = 3
```

```text
nums = [1, 2, 3, 1, 4, 5, 1]

answer = 5
```

```text
2 <= nums.length <= 100
1 <= nums[i] <= 10
```

For any two numbers `a` and `b`, `a * b == gcd(a, b) * lcm(a, b)`. Every subarray of length `2` is therefore product equivalent, and since `n >= 2` the answer is always at least `2`.

---

# Approach — Fix the Start, Extend the End, Track Three Values

### Core idea

The array is short (`n <= 100`), so every subarray can be checked.

For each start `i`, extend the end `j` one element at a time and update three running values in `O(1)` each:

```text
prd = prd * nums[j]
g   = gcd(g, nums[j])
l   = lcm(l, nums[j]) = (l * nums[j]) / gcd(l, nums[j])
```

After each update, test `prd == g * l`. On a match, the subarray `[i .. j]` is valid, so record `j - i + 1` if it is the longest so far.

The starting values make the first element behave naturally:

```text
g starts at 0   gcd(0, x) = x
l starts at 1   lcm(1, x) = x
prd starts at 1
```

### Bounding the product

Every value is at most `10`, and `lcm(1..10) = 2520`. For any subarray:

```text
gcd <= 10
lcm <= 2520
gcd * lcm <= 25200
```

The right side `g * l` never exceeds `25200`, while `prd` never decreases (every `nums[j] >= 1`). Once `prd > 25200`, no longer subarray from this start can match, so the inner loop stops.

This also keeps `prd` inside the range of an `int`:

```text
largest prd before the next multiply = 25200
25200 * 10 = 252000, which fits in int
```

Without the stop, a product of `100` numbers up to `10` reaches `10^100` and wraps around in `int`. A wrapped value could land on `g * l` by accident, so the stop is what keeps the result trustworthy.

### Walkthrough

```text
nums = [2, 3, 4, 5, 6]

i = 1
  j = 1  nums = 3   prd = 3     g = 3   l = 3    3 vs 9      no
  j = 2  nums = 4   prd = 12    g = 1   l = 12   12 vs 12    yes → length 2
  j = 3  nums = 5   prd = 60    g = 1   l = 60   60 vs 60    yes → length 3
  j = 4  nums = 6   prd = 360   g = 1   l = 60   360 vs 60   no

maxLen = 3
```

```text
nums = [1, 2, 1, 2, 1, 1, 1]

i = 0
  [1]              prd 1   g 1   l 1   1 vs 1   yes → length 1
  [1, 2]           prd 2   g 1   l 2   2 vs 2   yes → length 2
  [1, 2, 1]        prd 2   g 1   l 2   2 vs 2   yes → length 3
  [1, 2, 1, 2]     prd 4   g 1   l 2   4 vs 2   no
  [1, 2, 1, 2, 1]  prd 4   g 1   l 2   no
  ...

i = 2
  [1, 2, 1, 1, 1]  prd 2   g 1   l 2   2 vs 2   yes → length 5

maxLen = 5
```

A single element `x` matches only when `x == x * x`, which means `x == 1`.

### State

```text
i, j : start and end of the subarray
prd  : product of nums[i..j]
g    : gcd of nums[i..j]
l    : lcm of nums[i..j]
```

### Transitions

```text
for each start i
    prd = 1, g = 0, l = 1
    for each end j from i
        prd = prd * nums[j]
        if prd > 25200 → stop this start
        g = gcd(g, nums[j])
        l = lcm(l, nums[j])
        if prd == g * l → maxLen = max(maxLen, j - i + 1)
```

---

## Java

```java
class Solution {

    private static int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    public int maxLength(int[] nums) {

        int n = nums.length;
        int maxLen = 0;

        // gcd <= 10 and lcm <= lcm(1..10) = 2520, so gcd * lcm <= 25200.
        final int LIMIT = 25200;

        for (int i = 0; i < n; i++) {

            int l = 1;
            int g = 0;
            int prd = 1;

            for (int j = i; j < n; j++) {

                int num = nums[j];

                prd *= num;

                // The product only grows, so it can never come back down to g * l.
                if (prd > LIMIT) {
                    break;
                }

                g = gcd(g, num);
                l = (l * num) / gcd(l, num);

                if (prd == g * l) {
                    maxLen = Math.max(maxLen, j - i + 1);
                }
            }
        }

        return maxLen;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    int maxLength(vector<int>& nums) {

        int n = nums.size();
        int maxLen = 0;

        // gcd <= 10 and lcm <= lcm(1..10) = 2520, so gcd * lcm <= 25200.
        const int LIMIT = 25200;

        for (int i = 0; i < n; i++) {

            int l = 1;
            int g = 0;
            int prd = 1;

            for (int j = i; j < n; j++) {

                int num = nums[j];

                prd *= num;

                // The product only grows, so it can never come back down to g * l.
                if (prd > LIMIT) {
                    break;
                }

                g = gcd(g, num);
                l = (l * num) / gcd(l, num);

                if (prd == g * l) {
                    maxLen = max(maxLen, j - i + 1);
                }
            }
        }

        return maxLen;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n^2)   n starts, up to n ends each, and each gcd works on values <= 2520 (a few steps)
Space : O(1)     prd, g, l and a few counters
```

The pasted `O(n^2)` and `O(1)` are both right. The `LIMIT` stop only trims work on subarrays that already have a large product.

---

# Pattern Recognition

```text
Subarray condition built from product, gcd and lcm
        ↓
n is small (100) and values are tiny (<= 10)
        ↓
Fix the start, extend the end
        ↓
Update prod, gcd and lcm incrementally in O(1)
        ↓
Stop once the product exceeds the largest possible gcd * lcm
```

### Remember this trigger:

> **Check `prod == gcd * lcm` over every subarray → expand from each start, keep `prd`, `g`, `l` as running values, and stop when `prd` passes `25200` because `gcd * lcm` can never reach it.**
