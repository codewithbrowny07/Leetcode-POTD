# LeetCode 3942 — Minimum Operations to Sort a Permutation

## Problem

`nums` is a permutation of `0, 1, ..., n - 1`.

Only two operations are allowed:

```text
Reverse the whole array
Rotate left by one: move nums[0] to the end
```

Return the minimum number of operations that sorts `nums` into increasing order. If that is impossible, return `-1`.

Example:

```text
nums = [0, 2, 1]

rotate left → [2, 1, 0]
reverse     → [0, 1, 2]
answer = 2
```

```text
nums = [1, 0, 2]

reverse     → [2, 0, 1]
rotate left → [0, 1, 2]
answer = 2
```

```text
nums = [2, 0, 1, 3]
answer = -1
```

```text
1 <= n <= 10^5
nums is a permutation of 0 .. n - 1
```

---

# Approach — Circular Order

### Core idea

A left rotation does not change the circular order. It only chooses a new starting point.

A reverse flips that circular order.

Two reverses cancel, so an optimal plan uses a reverse either never or once. The useful plans are:

```text
rotations only
reverse, then rotations, then reverse
rotations, then one reverse
one reverse, then rotations
```

The target circular order is:

```text
0 → 1 → 2 → ... → n - 1 → 0
```

Therefore the current array must already be that cycle, or that cycle read backwards. Anything else cannot be repaired by rotations and one reversal.

### Detect the direction

Scan adjacent pairs:

```text
ascending pair:  nums[i] == (nums[i - 1] + 1) % n
descending pair: nums[i - 1] == (nums[i] + 1) % n
```

If all `n - 1` adjacent pairs ascend, the array is a rotation of `[0, 1, ..., n - 1]`.

If all `n - 1` adjacent pairs descend, it is a rotation of the reversed cycle.

Otherwise return `-1`.

### Ascending cycle

The array has the form:

```text
[k, k + 1, ..., n - 1, 0, 1, ..., k - 1]
```

where `k = nums[0]`.

If `k == 0`, it is already sorted. The answer is `0`.

Otherwise compare two plans:

```text
left rotations only
    0 is n - k positions from the front
    cost = n - nums[0]

reverse, rotate, reverse
    after the first reverse, rotate nums[n - 1] + 1 times,
    then reverse again
    cost = 1 + (nums[n - 1] + 1) + 1
```

```text
nums = [2, 0, 1]

left rotations: 3 - 2 = 1
    [2, 0, 1] → [0, 1, 2]

reverse plan: 1 + (1 + 1) + 1 = 4

answer = 1
```

### Descending cycle

The direction is backwards, so the plan must contain one reverse.

```text
reverse first, then rotate
    cost = n - nums[n - 1] + 1

rotate first, then reverse
    rotate (nums[0] + 1) % n times, then reverse once
    cost = (nums[0] + 1) % n + 1
```

```text
nums = [1, 0, 2]

reverse, then rotate: 3 - 2 + 1 = 2
    [1, 0, 2] → [2, 0, 1] → [0, 1, 2]

rotate, then reverse: (1 + 1) % 3 + 1 = 3

answer = 2
```

### State

```text
ascCount : adjacent pairs that increase by 1 modulo n
dscCount : adjacent pairs that decrease by 1 modulo n
leftRot  : cost of the rotation-first plan
rightRot : cost of the other plan
```

### Transitions

```text
ascCount == n - 1 and nums[0] == 0  → 0
ascCount == n - 1                   → min(n - nums[0], nums[n - 1] + 3)
dscCount == n - 1                   → min(n - nums[n - 1] + 1,
                                         (nums[0] + 1) % n + 1)
otherwise                           → -1
```

---

## Java

```java
class Solution {
    public int minOperations(int[] nums) {
        int n = nums.length;

        int ascCount = 0;
        int dscCount = 0;

        for (int i = 1; i < n; i++) {
            // Next value continues 0, 1, ..., n - 1, 0.
            ascCount += (nums[i] == (nums[i - 1] + 1) % n) ? 1 : 0;

            // Next value continues the reversed cycle.
            dscCount += (nums[i - 1] == (nums[i] + 1) % n) ? 1 : 0;
        }

        // Already [0, 1, 2, ..., n - 1].
        if (ascCount == n - 1 && nums[0] == 0) {
            return 0;
        }

        // Rotation of the sorted cycle.
        if (ascCount == n - 1) {
            int leftRot = n - nums[0];
            int rightRot = 1 + (nums[n - 1] + 1) + 1;
            return Math.min(leftRot, rightRot);
        }

        // Rotation of the reversed cycle. One reverse is required.
        if (dscCount == n - 1) {
            int reverseThenRotate = n - nums[n - 1] + 1;
            int rotateThenReverse = (nums[0] + 1) % n + 1;
            return Math.min(reverseThenRotate, rotateThenReverse);
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

    int minOperations(vector<int>& nums) {

        int n = nums.size();

        int ascCount = 0;
        int dscCount = 0;

        for (int i = 1; i < n; i++) {
            // Next value continues 0, 1, ..., n - 1, 0.
            ascCount += (nums[i] == (nums[i - 1] + 1) % n);

            // Next value continues the reversed cycle.
            dscCount += (nums[i - 1] == (nums[i] + 1) % n);
        }

        // Already [0, 1, 2, ..., n - 1].
        if (ascCount == n - 1 && nums[0] == 0) {
            return 0;
        }

        // Rotation of the sorted cycle.
        if (ascCount == n - 1) {
            int leftRot = n - nums[0];
            int rightRot = 1 + (nums[n - 1] + 1) + 1;
            return min(leftRot, rightRot);
        }

        // Rotation of the reversed cycle. One reverse is required.
        if (dscCount == n - 1) {
            int reverseThenRotate = n - nums[n - 1] + 1;
            int rotateThenReverse = (nums[0] + 1) % n + 1;
            return min(reverseThenRotate, rotateThenReverse);
        }

        return -1;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n)    one pass over adjacent pairs
Space : O(1)    four counters
```

The operations are not simulated. A rotation of `x` positions costs `x`, even when those `x` moves are counted in one expression.

---

# Pattern Recognition

```text
Only whole-array reverse and left rotation
        ↓
Rotation preserves circular order
Reverse flips it
        ↓
Sorted target is one specific cycle
        ↓
The array must be that cycle, or its reverse
        ↓
Compare the rotation-only plan with the one-reverse plan
```

### Remember this trigger:

> **Sort using only rotations and reversal → test whether the permutation is already the target cycle in either direction. If it is not, no sequence of these operations can create the missing order.**
