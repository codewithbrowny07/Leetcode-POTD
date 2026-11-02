# LeetCode 462 — Minimum Moves to Equal Array Elements II

## Problem

Given an integer array `nums`, one move increments or decrements **one** element by `1`.

Return the minimum moves needed to make all array elements equal.

Example:

```text
nums = [1, 2, 3]
answer = 2

Make every value 2:
|1 - 2| + |2 - 2| + |3 - 2| = 1 + 0 + 1 = 2
```

```text
nums = [1, 10, 2, 9]
sorted = [1, 2, 9, 10]
answer = 16

Make every value 2:
1 + 0 + 7 + 8 = 16
```

---

# Approach — Sort and Use a Median

### Core idea

If every element becomes `target`, the total moves are:

```text
Σ |nums[i] - target|
```

This sum of absolute distances is minimized by any **median**.

After sorting:

```text
odd length   → the middle element is the median
even length  → any value between the two middle elements is optimal
```

The code chooses:

```text
nums[n / 2]
```

For even length this is the upper median. It gives the same minimum as the lower median.

### Why the median works

Imagine moving the target one step right:

```text
elements left of target   → each distance increases by 1
elements right of target  → each distance decreases by 1
```

The total stops improving exactly when at least half of the values are on each side. That is the median region.

The mean minimizes squared distance. The median minimizes absolute distance:

```text
sum of squares      → mean
sum of abs distance → median
```

### Walkthrough

```text
nums = [1, 10, 2, 9]

sort → [1, 2, 9, 10]
mid  → index 2, value 9

|1 - 9|  = 8
|2 - 9|  = 7
|9 - 9|  = 0
|10 - 9| = 1

answer = 16
```

Choosing `2` also gives:

```text
1 + 0 + 7 + 8 = 16
```

### State

```text
nums[mid]  : chosen median target
res        : total moves to reach that target
```

### Transitions

```text
sort nums
target = nums[n / 2]
res += |nums[i] - target| for every i
```

---

## Java

```java
class Solution {
    public int minMoves2(int[] nums) {

        Arrays.sort(nums);

        int mid = nums.length / 2;
        int target = nums[mid];
        int res = 0;

        for (int i = 0; i < nums.length; i++) {

            if (i < mid) {
                res += target - nums[i];
            } else {
                res += nums[i] - target;
            }
        }

        return res;
    }
}
```

After sorting, the explicit `if` avoids calling `Math.abs`. An equivalent, shorter loop is:

```java
for (int value : nums) {
    res += Math.abs(value - target);
}
```

---

## C++

```cpp
class Solution {
public:

    int minMoves2(vector<int>& nums) {

        sort(nums.begin(), nums.end());

        int mid = nums.size() / 2;
        int target = nums[mid];
        int result = 0;

        for (int i = 0; i < (int) nums.size(); i++) {

            if (i < mid) {
                result += target - nums[i];
            } else {
                result += nums[i] - target;
            }
        }

        return result;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n log n)    sorting dominates
Space : O(log n) to O(n), depending on the sorting implementation
```

Java's primitive `Arrays.sort(int[])` sorts in place with logarithmic auxiliary stack space. The returned move count is guaranteed to fit in a 32-bit `int`.

---

# Pattern Recognition

```text
Move individual values up or down
        ↓
Cost to a target = sum of absolute differences
        ↓
Absolute differences are minimized at the median
        ↓
Sort, choose a middle value, sum the gaps
```

### Remember this trigger:

> **Minimum total ±1 moves to one common value → choose a median. Mean is for squared error, median is for absolute error.**
