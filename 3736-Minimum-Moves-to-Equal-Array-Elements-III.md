# LeetCode 3736 — Minimum Moves to Equal Array Elements III

## Problem

You are given an integer array `nums`.

In one move, increase **one** element by `1`. Decreasing is not allowed.

Return the minimum number of moves that makes every element equal.

Example:

```text
nums = [2, 1, 3]

2 → 3
1 → 2 → 3

answer = 3
```

```text
nums = [4, 4, 5]

4 → 5
4 → 5

answer = 2
```

```text
1 <= nums.length <= 100
1 <= nums[i] <= 100
```

---

# Approach — Raise Everything to the Maximum

### Core idea

A move can only make a value larger. The largest value cannot move down, so the common final value must be at least the maximum.

Choosing anything above the maximum also increments the old maximum. That adds one extra move for every element, including values that were already at the old maximum.

The minimum target is therefore the current maximum:

```text
moves for nums[i] = max - nums[i]
total moves       = sum of those gaps
```

That is the same formula as:

```text
n * max - sum(nums)
```

### Walkthrough

```text
nums = [2, 1, 3]
max  = 3

3 - 2 = 1
3 - 1 = 2
3 - 3 = 0

answer = 3
```

```text
nums = [4, 4, 5]
max  = 5

5 - 4 = 1
5 - 4 = 1
5 - 5 = 0

answer = 2
```

Starting `max` at `0` is valid for this problem because every `nums[i]` is at least `1`. The first value replaces `0`, and later values can only raise it.

### How this differs from the earlier equal-array problems

```text
453  increment n - 1 elements   → meet at the minimum
462  increment or decrement one → meet at a median
3736 increment one element      → meet at the maximum
```

The allowed direction decides the target.

### State

```text
max : largest value in nums
res : total increments needed to reach max
```

### Transitions

```text
scan once for the maximum
scan again and add max - nums[i]
```

---

## Java

```java
class Solution {
    public int minMoves(int[] nums) {

        int max = 0;

        // Every nums[i] is at least 1, so this finds the real maximum.
        for (int value : nums) {
            if (value > max) {
                max = value;
            }
        }

        int res = 0;

        for (int value : nums) {
            res += max - value;
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

        int maxValue = 0;

        // Every nums[i] is at least 1, so this finds the real maximum.
        for (int value : nums) {
            maxValue = max(maxValue, value);
        }

        int result = 0;

        for (int value : nums) {
            result += maxValue - value;
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
Space : O(1)    the maximum and the running total
```

---

# Pattern Recognition

```text
Only increments are allowed
        ↓
The largest value cannot decrease
        ↓
Every smaller value must climb to that maximum
        ↓
answer = Σ (max - nums[i])
```

### Remember this trigger:

> **Equalize an array when each move increments one element → the target is the maximum, and the answer is the sum of the gaps below it.**
