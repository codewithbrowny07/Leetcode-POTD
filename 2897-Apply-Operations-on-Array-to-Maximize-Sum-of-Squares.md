# LeetCode 2897 — Apply Operations on Array to Maximize Sum of Squares

## Problem

You are given a 0-indexed array `nums` and a positive integer `k`.

Any number of times, pick two different indices `i` and `j` and replace the pair with:

```text
nums[i] = nums[i] AND nums[j]
nums[j] = nums[i] OR  nums[j]
```

The two writes happen from the original values.

After that, choose `k` elements from the final array. Return the **maximum** possible sum of their squares, modulo `10^9 + 7`.

Example:

```text
nums = [2, 6, 5, 8]
k    = 2
answer = 261
```

One sequence of operations turns the array into `[0, 6, 0, 15]`. The best `k` elements are `15` and `6`:

```text
15² + 6² = 225 + 36 = 261
```

```text
1 <= k <= nums.length <= 10^5
1 <= nums[i] <= 10^9
```

---

# Approach — Count Bits, Then Build the k Largest Numbers

### What the operation really does

Look at one bit position in both numbers:

```text
0 and 0  →  AND 0, OR 0     unchanged
1 and 1  →  AND 1, OR 1     unchanged
1 and 0  →  AND 0, OR 1     the 1 moves to the other number
```

A `1` can move onto a `0` in the same bit. Two `1`s in the same bit cannot merge into one number, because a number has only one slot for that bit.

So the operation does **not** change how many numbers have bit `b` set. It only moves those `1`s around.

### Why the largest numbers should be as large as possible

Squares grow faster as the value grows. If `a > b` and you move a positive chunk `c` from the smaller number onto the larger one:

```text
(a + c)² + (b - c)² = a² + b² + 2c(a - b) + 2c²
```

That is strictly bigger than `a² + b²`. Among the `k` chosen numbers, bits should be piled onto the biggest ones.

Each of those `k` numbers can take a bit at most once. If bit `b` is set in `freq[b]` numbers, then `min(freq[b], k)` of the chosen numbers can receive it.

### The construction

Count the `1`s:

```text
freq[b] = how many numbers have bit b set
```

Then build `k` numbers. Number `0` takes every bit that still has a count. Number `1` takes whatever is left, and so on. Each time a bit is used, its count drops by `1`.

```text
for each of the k numbers:
    turn on every bit whose remaining count is > 0
    decrease those counts
    add (number²) mod 10^9+7
```

The first number is the largest possible value the bits can form. Each later number is the largest value that can still be formed from what is left.

### Walkthrough

```text
nums = [2, 6, 5, 8]
2 = 0010
6 = 0110
5 = 0101
8 = 1000

bit 0: 1 number
bit 1: 2 numbers
bit 2: 2 numbers
bit 3: 1 number

k = 2

number 0 takes bits 0, 1, 2, 3  →  15
counts left: bit 1 = 1, bit 2 = 1

number 1 takes bits 1, 2        →  6

15² + 6² = 261
```

### State

```text
freq[b]  : remaining numbers that can still contribute bit b
num      : the next chosen value being built
ans      : sum of squares, kept modulo 10^9 + 7
```

### Transitions

```text
read nums     → freq[b]++ for every set bit
for each of k numbers:
    if freq[bit] > 0 → set that bit in num, freq[bit]--
    ans = (ans + num²) mod 10^9+7
```

---

## Java

```java
class Solution {
    public int maxSum(List<Integer> nums, int k) {

        int[] freq = new int[32];
        int MOD = 1_000_000_007;

        for (int num : nums) {
            for (int b = 0; b < 32; b++) {
                if (((num >> b) & 1) == 1) {
                    freq[b]++;
                }
            }
        }

        long ans = 0;

        // Build the k largest values the bits can still form.
        for (int i = 0; i < k; i++) {

            long num = 0;

            for (int bit = 0; bit < 32; bit++) {
                if (freq[bit] > 0) {
                    num |= (1L << bit);
                    freq[bit]--;
                }
            }

            ans = (ans + num * num) % MOD;
        }

        return (int) ans;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int maxSum(vector<int>& nums, int k) {

        int freq[32] = {0};
        const int MOD = 1000000007;

        for (int num : nums) {
            for (int b = 0; b < 32; b++) {
                if ((num >> b) & 1) {
                    freq[b]++;
                }
            }
        }

        long long ans = 0;

        // Build the k largest values the bits can still form.
        for (int i = 0; i < k; i++) {

            long long num = 0;

            for (int bit = 0; bit < 32; bit++) {
                if (freq[bit] > 0) {
                    num |= (1LL << bit);
                    freq[bit]--;
                }
            }

            ans = (ans + num * num) % MOD;
        }

        return (int) ans;
    }
};
```

`num` is 64-bit before it is squared. The largest value here is below `2^30`, so `num * num` fits in a signed 64-bit integer, and only then is the modulo applied.

---

# Complexity

Let `n = nums.length`. Every value uses at most 32 bits.

```text
Time  : O(32 · (n + k))    count the bits, then build k numbers
Space : O(1)               a fixed array of 32 counts
```

---

# Pattern Recognition

```text
AND / OR on a pair
        ↓
A 1 can move onto a 0 in the same bit
Two 1s in the same bit cannot merge
        ↓
The count of each bit is fixed
        ↓
Sum of squares wants values as unequal as possible
        ↓
Give every remaining bit to the next chosen number, k times
```

### Remember this trigger:

> **`(x AND y, x OR y)` does not create or destroy bits. Count each bit, then pack the remaining 1s into the k numbers you will square.**
