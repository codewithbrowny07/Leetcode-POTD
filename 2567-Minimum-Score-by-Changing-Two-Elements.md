# LeetCode 2567 — Minimum Score by Changing Two Elements

## Problem

You are given an integer array `nums`.

The **score** of the array is `max(nums) - min(nums)`.

You may change **at most two** elements to any integers.

Return the minimum possible score after those changes.

Example:

```text
nums = [1, 4, 3]

Change 4 to 1 → [1, 1, 3]
score = 3 - 1 = 2

answer = 2
```

```text
nums = [1, 4, 7, 8, 5]

After sorting: [1, 4, 5, 7, 8]

Change 1 and 8 → remaining span [4, 7] = 3
Change 7 and 8 → remaining span [1, 5] = 4
Change 1 and 4 → remaining span [5, 8] = 3

answer = 3
```

```text
3 <= nums.length <= 10^5
1 <= nums[i] <= 10^9
```

The two new values can sit anywhere inside the remaining range, so they never widen it. The score after the changes is exactly the range of the **untouched** values.

---

# Approach — Sort, Then Cover Two Extremes

### Core idea

Sort the array. The original score is `nums[n - 1] - nums[0]`. Each change can overwrite one of those extremes (or any other value). After two overwrites, three sorted values stay, and the new score is the gap between the smallest and largest of those survivors.

The two positions that still affect the max and min after a sort are always at the ends. The useful pairs of overwrites are therefore:

```text
overwrite the two smallest          leftover range = nums[n - 1] - nums[2]
overwrite the two largest           leftover range = nums[n - 3] - nums[0]
overwrite the smallest and largest  leftover range = nums[n - 2] - nums[1]
```

Any other pair of indices sits strictly inside one of these three remaining windows, so the window can only be wider. The answer is the minimum of those three gaps.

Setting both changed values equal to some leftover `nums[i]` is always enough. The problem allows any integers, including values already in the array.

### Walkthrough

```text
nums = [1, 4, 7, 8, 5]
sorted = [1, 4, 5, 7, 8]
n = 5

nums[n-2] - nums[1] = 7 - 4 = 3   change 1 and 8
nums[n-3] - nums[0] = 5 - 1 = 4   change 7 and 8
nums[n-1] - nums[2] = 8 - 5 = 3   change 1 and 4

answer = 3
```

```text
nums = [1, 4, 3]
sorted = [1, 3, 4]
n = 3

every leftover range is a single value
answer = 0
```

With three elements, two changes can make every value equal, so the score is `0`. All three formulas collapse to `0` in that case.

### State

```text
nums : sorted in non-decreasing order
n    : nums.length
```

### Transitions

```text
sort nums
answer = min(
    nums[n - 2] - nums[1],
    nums[n - 3] - nums[0],
    nums[n - 1] - nums[2]
)
```

---

## Java

```java
class Solution {
    public int minimizeSum(int[] nums) {

        int n = nums.length;
        Arrays.sort(nums);

        // After two changes, the score is the range of the leftover values.
        // The three leftover windows sit at the ends of the sorted array.
        return Math.min(
            nums[n - 2] - nums[1],
            Math.min(nums[n - 3] - nums[0], nums[n - 1] - nums[2])
        );
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minimizeSum(vector<int>& nums) {

        int n = nums.size();
        sort(nums.begin(), nums.end());

        // After two changes, the score is the range of the leftover values.
        // The three leftover windows sit at the ends of the sorted array.
        return min({
            nums[n - 2] - nums[1],
            nums[n - 3] - nums[0],
            nums[n - 1] - nums[2]
        });
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n log n)   the sort
Space : O(log n)     Java dual-pivot quicksort / C++ introsort recursion
                     extra besides the input array
```

The three comparisons after the sort are `O(1)`. The pasted `O(1)` space comment counts only the few integers used after sorting. The sort itself still uses logarithmic stack space on the input array.

---

# Pattern Recognition

```text
Score is max - min
        ↓
Two values may be rewritten to anything
        ↓
Those two should cover the current extremes
        ↓
Sort, then try covering 2 left / 2 right / 1 of each
        ↓
answer = min of the three leftover gaps
```

### Remember this trigger:

> **Minimize `max - min` after changing two elements → sort, then take the minimum of the three leftover end-gaps `nums[n-1]-nums[2]`, `nums[n-3]-nums[0]`, and `nums[n-2]-nums[1]`.**
