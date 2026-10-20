# LeetCode 3886 — Sum of Sortable Integers

## Problem

You are given an integer array `nums` of length `n`.

A positive integer `k` is **sortable** when:

```text
k divides n
```

and this operation can make `nums` non-decreasing:

```text
split nums into consecutive blocks of length k
cyclically rotate every block independently any number of times
```

Return the sum of every sortable `k`.

Example:

```text
nums = [3, 1, 2]
n = 3

divisors: 1, 3

k = 1  → [3], [1], [2] cannot change, so not sorted
k = 3  → [3, 1, 2] rotates to [1, 2, 3]

answer = 3
```

```text
nums = [5, 8]

k = 1 and k = 2 both work because nums is already sorted
answer = 1 + 2 = 3
```

```text
1 <= n <= 10^5
1 <= nums[i] <= 10^5
```

---

# Approach — Divisors + Descents + Prefix/Suffix Bounds

### 1. Only test divisors of n

The blocks must have exactly length `k`, so `k` must divide `n`.

Every divisor comes in a pair:

```text
d and n / d
```

Iterate only while `d <= n / d` to find both in `O(sqrt(n))` iterations.

---

### 2. When can one block be sorted by rotation?

Consider a block:

```text
[3, 1, 2]
```

It has one descent:

```text
3 > 1
```

Rotating after that descent gives:

```text
[1, 2, 3]
```

A block can be cyclically rotated into non-decreasing order **if and only if** it has at most one descent:

```text
a[i] > a[i + 1]
```

Why?

```text
0 descents  → it is already non-decreasing
1 descent   → rotate immediately after that descent
2+ descents → one circular cut cannot repair every drop
```

The edge from the last block element back to the first is also part of the circular order. If:

```text
a[end] > a[start]
```

that is one extra circular descent.

---

### 3. Blocks must join into one sorted array

Making every block sorted is not enough. For example:

```text
[5, 6] [1, 2]
```

Both blocks are sorted, but they cannot form one non-decreasing array in that order.

At the start `s` of each block, every earlier value must be no larger than every value from `s` onward:

```text
max(nums[0..s-1]) <= min(nums[s..n-1])
```

Precompute:

```text
prefMax[s] = maximum of nums[0..s-1]
suffMin[s] = minimum of nums[s..n-1]
```

Then a block boundary is valid in `O(1)`:

```text
prefMax[s] <= suffMin[s]
```

At `s = 0`, the left side is empty. `prefMax[0]` is `0`, which is safe because every `nums[i]` is positive.

---

### 4. Count descents in O(1)

`drops[i]` stores how many ordinary descents occur before index `i`:

```text
drops[i] = number of p where 0 <= p < i and nums[p] > nums[p + 1]
```

For block `[s..e]`, its internal descents are:

```text
drops[e] - drops[s]
```

Then add the circular edge:

```text
if nums[e] > nums[s], count++
```

The block works only when `count <= 1`.

### State

```text
prefMax[s]  : largest value before block start s
suffMin[s]  : smallest value from block start s onward
drops[i]    : ordinary descents before index i
```

### Conditions for every block `[s..e]`

```text
internal drops + circular drop <= 1
prefMax[s] <= suffMin[s]
```

If every block of length `k` passes, add `k` to the answer.

---

## Java

```java
class Solution {
    public int sortableIntegers(int[] nums) {
        int n = nums.length;
        int[] prefMax = new int[n + 1];
        int[] suffMin = new int[n + 1];
        int[] drops = new int[n];

        // prefMax[s] = max of nums[0..s-1].
        // prefMax[0] = 0 is valid because nums contains positive values.
        for (int i = 0; i < n; i++) {
            prefMax[i + 1] = Math.max(prefMax[i], nums[i]);
        }

        // suffMin[s] = min of nums[s..n-1].
        suffMin[n] = Integer.MAX_VALUE;
        for (int i = n - 1; i >= 0; i--) {
            suffMin[i] = Math.min(nums[i], suffMin[i + 1]);
        }

        // drops[i] counts nums[p] > nums[p + 1] for p < i.
        for (int i = 1; i < n; i++) {
            drops[i] = drops[i - 1] + (nums[i - 1] > nums[i] ? 1 : 0);
        }

        int ans = 0;

        // Test divisor pairs d and n / d.
        for (int d = 1; d <= n / d; d++) {
            if (n % d != 0) {
                continue;
            }

            if (check(nums, prefMax, suffMin, drops, d)) {
                ans += d;
            }

            int other = n / d;
            if (other != d && check(nums, prefMax, suffMin, drops, other)) {
                ans += other;
            }
        }

        return ans;
    }

    private boolean check(int[] nums, int[] prefMax, int[] suffMin,
                          int[] drops, int k) {
        int n = nums.length;

        for (int start = 0; start < n; start += k) {
            int end = start + k - 1;

            // Ordinary drops inside this block.
            int count = drops[end] - drops[start];

            // The last-to-first edge is the circular drop.
            if (nums[end] > nums[start]) {
                count++;
            }

            // The block cannot be fixed with one rotation, or this
            // boundary cannot join into a globally sorted array.
            if (count > 1 || prefMax[start] > suffMin[start]) {
                return false;
            }
        }

        return true;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int sortableIntegers(vector<int>& nums) {

        int n = nums.size();
        vector<int> prefMax(n + 1, 0);
        vector<int> suffMin(n + 1);
        vector<int> drops(n, 0);

        // prefMax[s] = max of nums[0..s-1].
        // prefMax[0] = 0 is valid because nums contains positive values.
        for (int i = 0; i < n; i++) {
            prefMax[i + 1] = max(prefMax[i], nums[i]);
        }

        // suffMin[s] = min of nums[s..n-1].
        suffMin[n] = INT_MAX;
        for (int i = n - 1; i >= 0; i--) {
            suffMin[i] = min(nums[i], suffMin[i + 1]);
        }

        // drops[i] counts nums[p] > nums[p + 1] for p < i.
        for (int i = 1; i < n; i++) {
            drops[i] = drops[i - 1] + (nums[i - 1] > nums[i]);
        }

        int answer = 0;

        // Test divisor pairs d and n / d.
        for (int d = 1; d <= n / d; d++) {
            if (n % d != 0) {
                continue;
            }

            if (check(nums, prefMax, suffMin, drops, d)) {
                answer += d;
            }

            int other = n / d;
            if (other != d && check(nums, prefMax, suffMin, drops, other)) {
                answer += other;
            }
        }

        return answer;
    }

private:

    bool check(const vector<int>& nums, const vector<int>& prefMax,
               const vector<int>& suffMin, const vector<int>& drops, int k) {

        int n = nums.size();

        for (int start = 0; start < n; start += k) {
            int end = start + k - 1;

            // Ordinary drops inside this block.
            int count = drops[end] - drops[start];

            // The last-to-first edge is the circular drop.
            if (nums[end] > nums[start]) {
                count++;
            }

            // The block cannot be fixed with one rotation, or this
            // boundary cannot join into a globally sorted array.
            if (count > 1 || prefMax[start] > suffMin[start]) {
                return false;
            }
        }

        return true;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Preprocessing : O(n)
Checks        : O(n log n) worst case
Space         : O(n)
```

For a divisor `k`, `check` visits `n / k` blocks. Summed across all divisors, this is no more than the harmonic-series bound:

```text
Σ(n / k) <= n · (1 + 1/2 + ... + 1/n) = O(n log n)
```

Only divisors are checked, so the practical total is smaller.

---

# Pattern Recognition

```text
All block lengths that can sort after independent rotations
        ↓
Only divisors of n matter
        ↓
One block is a sorted rotation iff its circular descent count is at most 1
        ↓
Adjacent blocks must also obey:
max(prefix before block) <= min(suffix from block)
        ↓
Precompute prefix max, suffix min, and drop-prefix counts
```

### Remember this trigger:

> **Rotate each fixed-size block to sort globally → test divisors. A block needs at most one circular drop, and every block boundary must separate smaller prefix values from larger suffix values.**
