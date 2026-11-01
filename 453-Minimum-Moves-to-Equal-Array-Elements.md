# LeetCode 453 — Minimum Moves to Equal Array Elements

## Problem

You are given an integer array `nums` of length `n`.

In one move, increment **any** `n - 1` elements by `1`.

Return the minimum number of moves that makes every element equal.

Example:

```text
nums = [1, 2, 3]

[1, 2, 3]
[2, 3, 3]
[3, 4, 3]
[4, 4, 4]

answer = 3
```

```text
nums = [1, 1, 1]
answer = 0
```

```text
1 <= n <= 10^5
-10^9 <= nums[i] <= 10^9
the answer fits in a 32-bit integer
```

---

# Approach — Decrement the One Element Left Out

### Core idea

Incrementing every element except one changes the array in exactly the same relative way as decrementing that one element.

```text
[1, 2, 3]
increment the first two → [2, 3, 3]

same gaps as
decrement the last one  → [1, 2, 2]
```

Both arrays still need the same number of moves to become equal. Only the final common value changes.

So the original operation can be replaced by:

```text
choose one element and decrease it by 1
```

The elements must meet at the current minimum. Nothing needs to go below it, and every larger value must come down to it.

```text
moves for nums[i] = nums[i] - min
total moves       = sum of those gaps
```

That is the same formula as:

```text
sum(nums) - n * min
```

Adding the gaps one by one is safer in Java and C++. The final answer fits in a 32-bit integer, so the running total of non-negative gaps does too. Computing `sum(nums)` and `n * min` separately can overflow a 32-bit integer before the subtraction.

### Walkthrough

```text
nums = [1, 2, 3]
min  = 1

1 - 1 = 0
2 - 1 = 1
3 - 1 = 2

answer = 0 + 1 + 2 = 3
```

```text
nums = [1, 1, 1]
min  = 1

every gap is 0
answer = 0
```

A negative minimum does not change the rule. `[-2, -1, 3]` still needs `0 + 1 + 5 = 6` moves.

### State

```text
min : smallest value in nums
res : total gap above that minimum
```

### Transitions

```text
scan once for the minimum
scan again and add nums[i] - min
```

---

## Java

```java
class Solution {
    public int minMoves(int[] nums) {

        int min = Integer.MAX_VALUE;

        for (int value : nums) {
            if (value < min) {
                min = value;
            }
        }

        // Each increment of n - 1 elements is one decrement of the excluded element.
        int res = 0;

        for (int value : nums) {
            res += value - min;
        }

        return res;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minMoves(vector<int>& nums) {

        int minValue = INT_MAX;

        for (int value : nums) {
            minValue = min(minValue, value);
        }

        // Each increment of n - 1 elements is one decrement of the excluded element.
        int result = 0;

        for (int value : nums) {
            result += value - minValue;
        }

        return result;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n)    two linear scans
Space : O(1)    the minimum and the running total
```

---

# Pattern Recognition

```text
Increment all but one element
        ↓
Same relative change as decrementing the excluded element
        ↓
Everything must come down to the minimum
        ↓
answer = Σ (nums[i] - min)
```

### Remember this trigger:

> **A move that increments every element except one is a decrement of the leftover element. Equalize the array by summing the gaps above the minimum.**
