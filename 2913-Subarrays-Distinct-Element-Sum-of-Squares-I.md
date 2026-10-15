# LeetCode 2913 — Subarrays Distinct Element Sum of Squares I

## Problem

You are given a 0-indexed integer array `nums`.

For every subarray `nums[i..j]`, count how many **distinct** values it contains. Square that count. Add the square into the answer.

Return the sum of those squares over **every** subarray.

Example:

```text
nums = [1, 2, 1]

[1]       → 1 distinct → 1
[1, 2]    → 2          → 4
[1, 2, 1] → 2          → 4
[2]       → 1          → 1
[2, 1]    → 2          → 4
[1]       → 1          → 1

answer = 1 + 4 + 4 + 1 + 4 + 1 = 15
```

```text
1 <= nums.length <= 100
1 <= nums[i] <= 100
```

`n` is at most 100, so looking at every subarray is fast enough.

---

# Approach — Fix the Left End, Grow the Right End

### Core idea

A subarray is a contiguous slice. There are `n(n + 1) / 2` of them.

Fix the start index `i`. Then move the end index `j` from `i` to `n - 1`.

The set of values in `nums[i..j]` is the set of `nums[i..j-1]` plus `nums[j]`. One `HashSet` for this start index records that. Each time `j` moves, insert `nums[j]` and add `size²` to the answer.

A new start index needs a new set, because the window no longer includes `nums[i]`.

### Walkthrough

```text
nums = [1, 2, 1]

i = 0
  j = 0  set {1}     size 1  add 1     answer 1
  j = 1  set {1, 2}  size 2  add 4     answer 5
  j = 2  set {1, 2}  size 2  add 4     answer 9

i = 1
  j = 1  set {2}     size 1  add 1     answer 10
  j = 2  set {2, 1}  size 2  add 4     answer 14

i = 2
  j = 2  set {1}     size 1  add 1     answer 15
```

### State

```text
i    : left end of the current subarray
j    : right end
st   : distinct values inside nums[i..j]
res  : sum of size² so far
```

### Transitions

```text
new i          → empty set
j moves right  → insert nums[j]
                 res += size * size
```

---

## Java

```java
class Solution {
    public int sumCounts(List<Integer> arr) {

        int res = 0;

        for (int i = 0; i < arr.size(); i++) {

            Set<Integer> st = new HashSet<>();

            for (int j = i; j < arr.size(); j++) {

                // nums[i..j] is nums[i..j-1] plus this one value.
                st.add(arr.get(j));

                res += st.size() * st.size();
            }
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

    int sumCounts(vector<int>& nums) {

        int n = nums.size();
        int res = 0;

        for (int i = 0; i < n; i++) {

            unordered_set<int> st;

            for (int j = i; j < n; j++) {

                // nums[i..j] is nums[i..j-1] plus this one value.
                st.insert(nums[j]);

                int distinct = (int) st.size();
                res += distinct * distinct;
            }
        }

        return res;
    }
};
```

The largest size is `n`, so `size²` is at most `10000`. There are at most `5050` subarrays, and the total fits in a 32-bit `int`.

---

# Complexity

Let `n = nums.length`.

```text
Time  : O(n²)    every subarray is visited once, and each insert is O(1) on average
Space : O(n)     the set holds the distinct values of the current window
```

---

# Pattern Recognition

```text
Answer depends on every subarray
        ↓
n is small
        ↓
Fix the left end, extend the right end
        ↓
Distinct count only grows when a new value appears
        ↓
One set per left end, add size² at every right end
```

### Remember this trigger:

> **Sum something over all subarrays, and n ≤ 100 → enumerate both ends. If the property grows as the right end moves, keep it in a set instead of rebuilding the window.**
