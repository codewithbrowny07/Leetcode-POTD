# LeetCode 3550 — Smallest Index With Digit Sum Equal to Index

## Problem

Given an integer array `nums`, return the **smallest index** `i` such that the **sum of digits** of `nums[i]` is equal to `i`.

If no such index exists, return `-1`.

---

# Approach — Linear Scan + Digit Sum

### Core idea

We need the **smallest** valid index, so scan from left to right.

For each index `i`:

```text
digitSum(nums[i]) == i  →  return i
```

If the loop ends with no match, return `-1`.

Digit sum is computed by repeatedly taking the last digit:

```text
sum += n % 10
n   /= 10
```

until `n` becomes `0`.

If `nums[i] == 0`, the loop does not run, so `sum` stays `0`. That correctly matches only when `i == 0`.

---

## Java

```java
class Solution {

    public int smallestIndex(int[] nums) {

        for (int i = 0; i < nums.length; i++) {

            int sum = 0;
            int n = nums[i];

            // Extract digits from right to left.
            // Example: 123 → 3 + 2 + 1 = 6
            while (n > 0) {
                sum += n % 10;
                n /= 10;
            }

            // First (smallest) index whose digit sum equals the index.
            if (sum == i) {
                return i;
            }
        }

        return -1;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int smallestIndex(vector<int>& nums) {

        for (int i = 0; i < (int)nums.size(); i++) {

            int sum = 0;
            int n = nums[i];

            // Extract digits from right to left.
            // Example: 123 → 3 + 2 + 1 = 6
            while (n > 0) {
                sum += n % 10;
                n /= 10;
            }

            // First (smallest) index whose digit sum equals the index.
            if (sum == i) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Complexity

```text
Time  : O(n · d)   d = number of digits in nums[i]
Space : O(1)
```

Each number has at most a few digits (for typical constraints, `d ≤ 10`), so this is effectively linear in `n`.

---

# Pattern Recognition

```text
Need smallest index matching a property
        ↓
Scan left → right, return on first hit
        ↓
Property = digit sum of nums[i]
        ↓
Repeated n % 10 / n / 10
```

### Remember this trigger:

> **Smallest index that satisfies a condition → one left-to-right pass. Digit work → modulo 10.**
