# LeetCode 2778 — Sum of Squares of Special Elements

## Problem

You are given a **1-indexed** integer array `nums` of length `n`.

An element `nums[i]` is **special** when `i` divides `n`:

```text
n % i == 0
```

Return the sum of the squares of all special elements.

Example:

```text
nums   = [1, 2, 3, 4]
n      = 4

special positions (1-indexed): 1, 2, 4
answer = 1² + 2² + 4² = 1 + 4 + 16 = 21
```

```text
nums   = [2, 7, 1, 19, 18, 3]
n      = 6

special positions: 1, 2, 3, 6
answer = 2² + 7² + 1² + 3² = 4 + 49 + 1 + 9 = 63
```

---

# Approach — Check Every Position

### Core idea

The problem counts positions from `1`, but Java and C++ arrays start at `0`.

So array index `i` is problem position `i + 1`:

```text
array index   0   1   2   3
position      1   2   3   4
```

For each index, check whether that position divides `n`. If it does, add the square.

```text
n % (i + 1) == 0  →  sum += nums[i] * nums[i]
```

### State

```text
i    : 0-based array index
sum  : total of squares found so far
```

### Transitions

```text
n % (i + 1) == 0  → add nums[i]²
otherwise         → skip
```

---

## Java

```java
class Solution {
    public int sumOfSquares(int[] nums) {

        int sum = 0;

        for (int i = 0; i < nums.length; i++) {

            // The problem is 1-indexed, so index i is position i + 1.
            if (nums.length % (i + 1) == 0) {
                sum += nums[i] * nums[i];
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

    int sumOfSquares(vector<int>& nums) {

        int n = nums.size();
        int sum = 0;

        for (int i = 0; i < n; i++) {

            // The problem is 1-indexed, so index i is position i + 1.
            if (n % (i + 1) == 0) {
                sum += nums[i] * nums[i];
            }
        }

        return sum;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n)    one check per position
Space : O(1)    one running sum
```

`n ≤ 50` and `nums[i] ≤ 50` here, so the sum fits easily in an `int`.

---

# Pattern Recognition

```text
1-indexed problem, 0-indexed array
        ↓
Use position = index + 1
        ↓
Position divides n
        ↓
Add the square
```

### Remember this trigger:

> **When the problem says 1-indexed, convert every array index with `i + 1` before applying the condition.**
