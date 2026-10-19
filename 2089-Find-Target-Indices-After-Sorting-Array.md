# LeetCode 2089 — Find Target Indices After Sorting Array

## Problem

You are given a 0-indexed array `nums` and an integer `target`.

Sort `nums` in non-decreasing order. Return every index where the sorted value equals `target`. The indexes come out in increasing order.

Example:

```text
nums   = [1, 2, 5, 2, 3]
target = 2

sorted = [1, 2, 2, 3, 5]
answer = [1, 2]

target = 3
answer = [3]

target = 4
answer = []
```

```text
1 <= nums.length <= 100
1 <= nums[i], target <= 100
```

---

# Approach 1 — Sort, Then Scan

### Core idea

After a non-decreasing sort, every copy of `target` sits in one contiguous block. One pass records the index of each match. The indexes are already sorted because the scan goes left to right.

If `target` never appears, the list stays empty.

### State

```text
nums  : sorted in place
lst   : indexes where the sorted value equals target
```

### Transitions

```text
sort nums
nums[i] == target  → append i
```

---

## Java

```java
class Solution {
    public List<Integer> targetIndices(int[] nums, int target) {

        int n = nums.length;
        List<Integer> lst = new ArrayList<>();

        Arrays.sort(nums);

        for (int i = 0; i < n; i++) {
            if (nums[i] == target) {
                lst.add(i);
            }
        }

        return lst;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<int> targetIndices(vector<int>& nums, int target) {

        sort(nums.begin(), nums.end());

        vector<int> result;

        for (int i = 0; i < (int) nums.size(); i++) {
            if (nums[i] == target) {
                result.push_back(i);
            }
        }

        return result;
    }
};
```

### Complexity

Let `n = nums.length`.

```text
Time  : O(n log n)    the sort
Space : O(n)          the returned list, which holds one index per match
```

The scan after the sort is linear.

---

# Approach 2 — Count, Do Not Sort

### Core idea

The sorted position of `target` does not need the sorted array.

```text
less   = how many values are smaller than target
equal  = how many values are equal to target
```

In the sorted array those `equal` copies occupy the indexes

```text
less, less + 1, ..., less + equal - 1
```

If `equal` is `0`, the answer is empty.

```text
nums   = [1, 2, 5, 2, 3]
target = 2

less  = 1          the single 1
equal = 2
answer = [1, 2]
```

---

## Java

```java
class Solution {
    public List<Integer> targetIndices(int[] nums, int target) {

        int less = 0;
        int equal = 0;

        for (int num : nums) {
            if (num < target) {
                less++;
            } else if (num == target) {
                equal++;
            }
        }

        List<Integer> result = new ArrayList<>();

        for (int i = 0; i < equal; i++) {
            result.add(less + i);
        }

        return result;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<int> targetIndices(vector<int>& nums, int target) {

        int less = 0;
        int equal = 0;

        for (int num : nums) {
            if (num < target) {
                less++;
            } else if (num == target) {
                equal++;
            }
        }

        vector<int> result;

        for (int i = 0; i < equal; i++) {
            result.push_back(less + i);
        }

        return result;
    }
};
```

### Complexity

```text
Time  : O(n)    one count, then one write per match
Space : O(n)    the returned list
```

The two counters are the only extra memory besides that list.

---

# Pattern Recognition

```text
Indexes of a value after sorting
        ↓
Equal values form one block
        ↓
Sort and collect the block
        ↓
Or count how many values are smaller
   the block starts there and has length equal to the number of matches
```

### Remember this trigger:

> **Target indexes in the sorted array → count the values strictly less than `target`. That count is the first index, and each extra copy is the next index.**
