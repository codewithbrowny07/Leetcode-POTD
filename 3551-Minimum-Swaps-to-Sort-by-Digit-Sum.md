# LeetCode 3551 — Minimum Swaps to Sort by Digit Sum

## Problem

`nums` contains **distinct** positive integers.

Sort it by the **sum of digits**, smallest sum first. If two numbers have the same digit sum, the smaller number comes first.

Return the **minimum number of swaps** that puts `nums` into that order. A swap exchanges the values at two different indices.

Example:

```text
nums = [37, 100]
digit sums = [10, 1]
target = [100, 37]
answer = 1

nums = [22, 14, 33, 7]
digit sums = [4, 5, 6, 7]
target = [22, 14, 33, 7]
answer = 0

nums = [18, 43, 34, 16]
digit sums = [9, 7, 7, 7]
target = [16, 34, 43, 18]
answer = 2
```

```text
1 <= nums.length <= 10^5
1 <= nums[i] <= 10^9
every value is unique
```

---

# Approach — Target Order, Then Cycles

### Core idea

Any two positions may be swapped, so the minimum number of swaps is the number of places an element is out of position, counted by **cycles**.

Build the target order:

```text
sort by (digitSum, value)
```

`destination[i]` is the index where `nums[i]` belongs in that order.

Following `destination` splits the array into cycles. Inside one cycle, each swap can place one element into its final index and leave the displaced element still in the cycle. The last element falls into place by itself.

```text
cycle length L  →  L - 1 swaps
a cycle of 1    →  0 swaps     already home
```

If the permutation has `c` cycles, the same total is `n - c`.

### Walkthrough

```text
nums = [18, 43, 34, 16]

value   18  43  34  16
sum      9   7   7   7

target, smallest digit sum first, then smaller value:
index  0   1   2   3
value  16  34  43  18

where each original value must go:
18 → 3
43 → 2
34 → 1
16 → 0

destination = [3, 2, 1, 0]

cycle 0 → 3 → 0     length 2    swaps += 1
cycle 1 → 2 → 1     length 2    swaps += 1

answer = 2
```

One swap exchanges `18` and `16`. The next exchanges `43` and `34`.

### State

```text
arr[i]          : {value, digitSum}
sortedIndex     : value → its index after sorting
destination[i]  : target index of the value currently at i
visited         : indexes already counted in a cycle
```

### Transitions

```text
compute digit sums
sort by digit sum, then by value
destination[i] = sorted index of nums[i]
walk each unvisited cycle
swaps += cycleLength - 1
```

---

## Java

```java
class Solution {
    public int minSwaps(int[] nums) {
        int n = nums.length;
        int[][] arr = new int[n][2];

        // Store {value, digitSum}.
        for (int i = 0; i < n; i++) {
            arr[i][0] = nums[i];
            arr[i][1] = digitSum(nums[i]);
        }

        // Smaller digit sum first. Same sum keeps the smaller value first.
        Arrays.sort(arr, (a, b) -> {
            if (a[1] != b[1]) {
                return Integer.compare(a[1], b[1]);
            }
            return Integer.compare(a[0], b[0]);
        });

        // Values are distinct, so each value has one target index.
        Map<Integer, Integer> sortedIndex = new HashMap<>();

        for (int i = 0; i < n; i++) {
            sortedIndex.put(arr[i][0], i);
        }

        int[] destination = new int[n];

        for (int i = 0; i < n; i++) {
            destination[i] = sortedIndex.get(nums[i]);
        }

        boolean[] visited = new boolean[n];
        int swaps = 0;

        for (int i = 0; i < n; i++) {
            if (visited[i]) {
                continue;
            }

            int curr = i;
            int cycleLength = 0;

            // 0 → 3 → 0 is one cycle. Length 2 needs 1 swap.
            while (!visited[curr]) {
                visited[curr] = true;
                curr = destination[curr];
                cycleLength++;
            }

            if (cycleLength > 1) {
                swaps += cycleLength - 1;
            }
        }

        return swaps;
    }

    private int digitSum(int num) {
        int sum = 0;

        while (num > 0) {
            sum += num % 10;
            num /= 10;
        }

        return sum;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minSwaps(vector<int>& nums) {

        int n = nums.size();
        vector<pair<int, int>> arr(n);

        // Store {value, digitSum}.
        for (int i = 0; i < n; i++) {
            arr[i] = {nums[i], digitSum(nums[i])};
        }

        // Smaller digit sum first. Same sum keeps the smaller value first.
        sort(arr.begin(), arr.end(), [](const pair<int, int>& a, const pair<int, int>& b) {
            if (a.second != b.second) {
                return a.second < b.second;
            }
            return a.first < b.first;
        });

        // Values are distinct, so each value has one target index.
        unordered_map<int, int> sortedIndex;

        for (int i = 0; i < n; i++) {
            sortedIndex[arr[i].first] = i;
        }

        vector<int> destination(n);

        for (int i = 0; i < n; i++) {
            destination[i] = sortedIndex[nums[i]];
        }

        vector<char> visited(n, 0);
        int swaps = 0;

        for (int i = 0; i < n; i++) {
            if (visited[i]) {
                continue;
            }

            int curr = i;
            int cycleLength = 0;

            // 0 → 3 → 0 is one cycle. Length 2 needs 1 swap.
            while (!visited[curr]) {
                visited[curr] = 1;
                curr = destination[curr];
                cycleLength++;
            }

            if (cycleLength > 1) {
                swaps += cycleLength - 1;
            }
        }

        return swaps;
    }

private:

    int digitSum(int num) {
        int sum = 0;

        while (num > 0) {
            sum += num % 10;
            num /= 10;
        }

        return sum;
    }
};
```

---

# Complexity

Let `n = nums.length`. Each value has at most 10 digits.

```text
Time  : O(n log n)    sorting decides the target order
Space : O(n)          the pairs, the map, destination, and visited
```

Walking the cycles is linear. The `O(n log n)` bound comes from the sort, not from the cycle count.

---

# Pattern Recognition

```text
Minimum swaps into a required order
        ↓
Swaps may use any two indexes
        ↓
Build the target permutation
        ↓
A cycle of length L needs L - 1 swaps
        ↓
Digit-sum order is the sort key, with the number itself as the tie break
```

### Remember this trigger:

> **Minimum swaps to reach a known order → map each item to its target index and count cycles. Do not swap neighbors one step at a time.**
