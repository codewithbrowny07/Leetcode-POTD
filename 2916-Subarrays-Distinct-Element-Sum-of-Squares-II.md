# LeetCode 2916 — Subarrays Distinct Element Sum of Squares II

## Problem

You are given a 0-indexed integer array `nums`.

For every subarray, count its distinct values, square that count, and add it up.

Return that sum **modulo** `10^9 + 7`.

This is the same score as [2913](2913-Subarrays-Distinct-Element-Sum-of-Squares-I.md). The limits are larger, so scanning every subarray with a set is too slow.

Example:

```text
nums = [1, 2, 1]
answer = 15
```

The six subarrays contribute `1 + 4 + 4 + 1 + 4 + 1`.

```text
1 <= nums.length <= 10^5
1 <= nums[i] <= 10^5
```

Open [2916-segment-tree-visualizer.html](2916-segment-tree-visualizer.html) in a browser to step through this example. It draws `sum[]`, `squareSum[]`, and `lazy[]` for `nums = [1, 2, 1]`, and it highlights the update range at each right endpoint.

---

# Approach — Last Seen + Segment Tree

### Core idea

Group the subarrays by their **right** endpoint.

When the right end is `right`, the segment tree stores one number per start index `i`:

```text
tree[i] = distinct count of nums[i..right]
```

The contribution of this right endpoint is the sum of `tree[i]²` for every `i <= right`. That value lives at the root of `squareSum`.

Moving from `right - 1` to `right` does not rebuild those counts. `nums[right]` is a **new** distinct value only for starts that do not already contain it.

```text
previous = last index where nums[right] appeared, or -1
left     = previous + 1
```

Every subarray that starts in `[left, right]` and ends at `right` gains `+1` distinct. Starts at or before `previous` already contain this value, so their count stays the same.

```text
range add +1 on [left, right]
answer += squareSum[root]
lastSeen[nums[right]] = right
```

### Walkthrough

```text
nums = [1, 2, 1]

right = 0, value 1, previous = -1, update [0, 0]
  [1] has 1 distinct
  contribution = 1
  answer = 1

right = 1, value 2, previous = -1, update [0, 1]
  [1, 2] has 2 distinct
  [2]    has 1 distinct
  contribution = 4 + 1 = 5
  answer = 6

right = 2, value 1, previous = 0, update [1, 2]
  [1, 2, 1] still has 2 distinct     start 0 is left of the previous 1
  [2, 1]    has 2 distinct
  [1]       has 1 distinct
  contribution = 4 + 4 + 1 = 9
  answer = 15
```

### Why the square sum has its own update

A range add changes every stored count `x` in a segment into `x + delta`.

If the segment covers `L` starts, and those starts currently have total `sum` and total `squareSum`:

```text
newSum       = sum + L * delta

newSquareSum = Σ (x + delta)²
             = Σ x² + 2 * delta * Σ x + L * delta²
             = squareSum + 2 * delta * sum + L * delta²
```

Use the **old** `sum` in that formula, then update `sum`. The visualizer shows this on each `apply` step.

`delta` is stored in `lazy[]` so a fully covered node does not recurse immediately. `pushDown` applies that pending increment to the two children before a later partial update needs them.

### State

```text
sum[node]        : total distinct counts in this segment
squareSum[node]  : total of those counts squared
lazy[node]       : increment not yet pushed to the children
lastSeen[value]  : latest index of this value, or -1
```

### Transitions

```text
segment misses [left, right]     → return
segment sits inside [left, right] → apply +1 and stop
otherwise                        → pushDown, update both children, pull the sums back up
after the update                 → answer += squareSum[root]
```

---

## Java

```java
class Solution {
    static final long MOD = 1_000_000_007L;

    long[] sum;
    long[] squareSum;
    long[] lazy;
    int n;

    // Every distinct count in this segment increases by delta.
    void apply(int node, int left, int right, long delta) {
        long length = right - left + 1L;

        // Uses the old sum. (x + delta)² = x² + 2*delta*x + delta².
        squareSum[node] =
            (squareSum[node] + 2 * delta * sum[node]
             + length * delta * delta) % MOD;

        sum[node] = (sum[node] + length * delta) % MOD;
        lazy[node] += delta;
    }

    void pushDown(int node, int left, int right) {
        if (lazy[node] == 0 || left == right) {
            return;
        }

        int mid = left + (right - left) / 2;

        apply(node * 2 + 1, left, mid, lazy[node]);
        apply(node * 2 + 2, mid + 1, right, lazy[node]);

        lazy[node] = 0;
    }

    void update(int node, int left, int right,
                int queryLeft, int queryRight) {
        if (right < queryLeft || left > queryRight) {
            return;
        }

        if (queryLeft <= left && right <= queryRight) {
            apply(node, left, right, 1);
            return;
        }

        pushDown(node, left, right);

        int mid = left + (right - left) / 2;

        update(node * 2 + 1, left, mid, queryLeft, queryRight);
        update(node * 2 + 2, mid + 1, right, queryLeft, queryRight);

        sum[node] =
            (sum[node * 2 + 1] + sum[node * 2 + 2]) % MOD;

        squareSum[node] =
            (squareSum[node * 2 + 1] + squareSum[node * 2 + 2]) % MOD;
    }

    public int sumCounts(int[] nums) {
        n = nums.length;

        sum = new long[4 * n];
        squareSum = new long[4 * n];
        lazy = new long[4 * n];

        int[] lastSeen = new int[100001];
        java.util.Arrays.fill(lastSeen, -1);

        long answer = 0;

        for (int right = 0; right < n; right++) {
            int left = lastSeen[nums[right]] + 1;

            update(0, 0, n - 1, left, right);

            answer = (answer + squareSum[0]) % MOD;
            lastSeen[nums[right]] = right;
        }

        return (int) answer;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int sumCounts(vector<int>& nums) {

        n = nums.size();

        sum.assign(4 * n, 0);
        squareSum.assign(4 * n, 0);
        lazy.assign(4 * n, 0);

        vector<int> lastSeen(100001, -1);
        long long answer = 0;

        for (int right = 0; right < n; right++) {
            int left = lastSeen[nums[right]] + 1;

            update(0, 0, n - 1, left, right);

            answer = (answer + squareSum[0]) % MOD;
            lastSeen[nums[right]] = right;
        }

        return (int) answer;
    }

private:

    static const long long MOD = 1000000007LL;

    vector<long long> sum, squareSum, lazy;
    int n;

    // Every distinct count in this segment increases by delta.
    void apply(int node, int left, int right, long long delta) {
        long long length = right - left + 1LL;

        // Uses the old sum. (x + delta)² = x² + 2*delta*x + delta².
        squareSum[node] =
            (squareSum[node] + 2 * delta * sum[node]
             + length * delta * delta) % MOD;

        sum[node] = (sum[node] + length * delta) % MOD;
        lazy[node] += delta;
    }

    void pushDown(int node, int left, int right) {
        if (lazy[node] == 0 || left == right) {
            return;
        }

        int mid = left + (right - left) / 2;

        apply(node * 2 + 1, left, mid, lazy[node]);
        apply(node * 2 + 2, mid + 1, right, lazy[node]);

        lazy[node] = 0;
    }

    void update(int node, int left, int right,
                int queryLeft, int queryRight) {
        if (right < queryLeft || left > queryRight) {
            return;
        }

        if (queryLeft <= left && right <= queryRight) {
            apply(node, left, right, 1);
            return;
        }

        pushDown(node, left, right);

        int mid = left + (right - left) / 2;

        update(node * 2 + 1, left, mid, queryLeft, queryRight);
        update(node * 2 + 2, mid + 1, right, queryLeft, queryRight);

        sum[node] =
            (sum[node * 2 + 1] + sum[node * 2 + 2]) % MOD;

        squareSum[node] =
            (squareSum[node * 2 + 1] + squareSum[node * 2 + 2]) % MOD;
    }
};
```

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n log n)    one range update per right endpoint
Space : O(n)          three segment-tree arrays of length 4n, plus lastSeen
```

`lastSeen` has length `100001` because each value is at most `10^5`.

---

# Pattern Recognition

```text
Sum of (distinct count)² over every subarray, n = 10^5
        ↓
Fix the right endpoint
        ↓
A value increases the count only for starts after its previous occurrence
        ↓
Range add +1 on [lastSeen + 1, right]
        ↓
Also maintain the sum of squares, because (x + 1)² = x² + 2x + 1
        ↓
Lazy segment tree
```

### Remember this trigger:

> **Distinct counts of all subarrays ending at `r` → add 1 on `[lastSeen[nums[r]] + 1, r]`. If the score is a square, update `sum` and `squareSum` together with `squareSum + 2·delta·sum + L·delta²`.**
