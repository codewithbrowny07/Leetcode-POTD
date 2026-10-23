# LeetCode 3644 — Maximum K to Sort a Permutation

## Problem

`nums` is a permutation of `0, 1, ..., n - 1`.

You may swap values `nums[i]` and `nums[j]` only when:

```text
nums[i] AND nums[j] == k
```

`AND` is bitwise AND. Return the **largest** non-negative `k` for which some sequence of allowed swaps can sort the permutation.

If `nums` is already sorted, return `0`.

Example:

```text
nums = [0, 3, 2, 1]

k = 1
3 AND 1 = 1, so swap 3 and 1:
[0, 3, 2, 1] → [0, 1, 2, 3]

answer = 1
```

```text
nums = [0, 1, 3, 2]

k = 2
3 AND 2 = 2, so swap 3 and 2:
[0, 1, 3, 2] → [0, 1, 2, 3]

answer = 2
```

```text
nums = [3, 2, 1, 0]
answer = 0
```

```text
1 <= n <= 10^5
nums is a permutation of 0 .. n - 1
```

---

# Approach — AND Every Misplaced Value

### Core idea

In the final sorted permutation, value `i` must be at index `i`.

```text
nums[i] == i  → already correct, never needs a swap
nums[i] != i  → misplaced, must participate in the sorting
```

If a bit is `1` in a feasible `k`, every number used in a swap with AND exactly `k` must also have that bit set:

```text
x AND y == k
k has bit b = 1
therefore x has bit b = 1 and y has bit b = 1
```

Every misplaced value must be movable. Therefore each `1` bit in `k` must be present in **every misplaced value**.

The largest mask with that property is:

```text
k = AND of all nums[i] where nums[i] != i
```

Correct positions do not restrict `k` because they do not need to move.

### Why this AND is achievable

Let:

```text
mask = AND of every misplaced value
```

Every misplaced value contains all `1` bits in `mask`. Any extra bits are not shared by every misplaced value, and the misplaced values can be routed through swaps so only the common mask remains in the AND condition.

The important conclusion for this problem is:

```text
all misplaced values share mask
mask is the greatest possible k
```

So there is no need to simulate swaps or find cycles.

### Walkthrough

```text
nums = [0, 3, 2, 1]

index:  0  1  2  3
value:  0  3  2  1

misplaced values: 3, 1

3 = 11₂
1 = 01₂

3 AND 1 = 01₂ = 1
```

`2` is already at index `2`, so it is not included.

```text
nums = [0, 1, 3, 2]

misplaced values: 3, 2

3 = 11₂
2 = 10₂

3 AND 2 = 10₂ = 2
```

### Already sorted

Initialize the running AND to `-1`:

```text
-1 in two's-complement has every bit set to 1
-1 AND x = x
```

If no value is misplaced, it stays `-1`. The answer for an already sorted permutation is explicitly `0`, so return:

```text
k == -1 ? 0 : k
```

### State

```text
k  : AND of every misplaced value seen so far
```

### Transition

```text
nums[i] == i  → skip
nums[i] != i  → k &= nums[i]
```

---

## Java

```java
class Solution {
    public int sortPermutation(int[] nums) {

        int k = -1; // All bits are 1, the identity value for AND.

        for (int i = 0; i < nums.length; i++) {

            // Only values that must move restrict the largest possible k.
            if (nums[i] != i) {
                k &= nums[i];
            }
        }

        // No mismatches means nums was already sorted.
        return k == -1 ? 0 : k;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int sortPermutation(vector<int>& nums) {

        int k = -1; // All bits are 1, the identity value for AND.

        for (int i = 0; i < (int) nums.size(); i++) {

            // Only values that must move restrict the largest possible k.
            if (nums[i] != i) {
                k &= nums[i];
            }
        }

        // No mismatches means nums was already sorted.
        return k == -1 ? 0 : k;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n)    one scan
Space : O(1)    one integer mask
```

---

# Pattern Recognition

```text
Largest bitwise condition that lets every misplaced value move
        ↓
Correct values do not matter
        ↓
Every 1 bit in k must appear in every misplaced value
        ↓
Take the AND of all misplaced values
```

### Remember this trigger:

> **Maximum shared bitmask across elements that must change → AND only those elements. Start at all ones, and handle the empty set separately.**
