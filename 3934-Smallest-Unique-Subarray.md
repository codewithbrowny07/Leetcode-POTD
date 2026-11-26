# LeetCode 3934 — Smallest Unique Subarray

## Problem

You are given an integer array `nums`.

Find the minimum length of a subarray that is **not identical to any other subarray** in `nums`.

Two subarrays are identical when they have the same length and the same elements in the same positions.

Return that minimum length.

Example:

```text
nums = [3, 3, 3]

length 1 : [3]       appears 3 times
length 2 : [3, 3]    appears 2 times
length 3 : [3, 3, 3] appears 1 time

answer = 3
```

```text
nums = [2, 1, 2, 3, 3]

[1] appears once

answer = 1
```

```text
nums = [1, 1, 2, 2, 1]

length 1 : [1] x3, [2] x2          no unique one
length 2 : [1,1] [1,2] [2,2] [2,1]  each appears once

answer = 2
```

```text
1 <= nums.length <= 10^5
1 <= nums[i] <= 10^5
```

The whole array appears exactly once, so a unique subarray always exists and the answer is at most `n`.

---

# Approach — Binary Search on the Length + Rolling Hash

### Core idea

**Monotone check.** If some subarray of length `L` appears exactly once, the subarray of length `L + 1` that extends it also appears exactly once. A longer window that contains a unique window cannot match any other window. So

```text
"a unique subarray of length L exists"
```

is false for small `L`, then true from some length onward. Binary search finds the first `L` where it becomes true.

**One check.** To test a length `m`, slide a window of size `m` over the array and count how many times each window appears. Comparing windows directly costs `O(m)` each. A **rolling hash** turns every window into one number in `O(1)` per slide:

```text
hash = hash * base + nums[j]                 add the new element on the right
hash = hash - nums[i] * base^(m-1)           remove the old element on the left
```

A map from hash to "seen exactly once?" records the result. If any hash is still marked `true` after the pass, a unique window of length `m` exists.

`base = max(nums) + 1` keeps every element a valid digit in that base.

`p = base^(m-1)` is the weight of the leftmost element of a full window. It is computed with binary exponentiation in `O(log m)`.

### Binary exponentiation

```text
a^b = (a^(b/2))^2           when b is even
a^b = a * (a^(b/2))^2       when b is odd
```

Each step halves `b`, so the cost is `O(log b)`. The recursive form and the iterative form below are equivalent.

### Walkthrough

```text
nums = [1, 1, 2, 2, 1]
l = 1, r = 5, result = 5

m = 3
  windows [1,1,2] [1,2,2] [2,2,1]  all distinct → unique exists
  result = 3, r = 2

m = 1
  windows 1, 1, 2, 2, 1   →  1 appears 3 times, 2 appears 2 times
  no unique window → l = 2

m = 2
  windows [1,1] [1,2] [2,2] [2,1]  all distinct → unique exists
  result = 2, r = 1

l > r → answer 2
```

```text
nums = [3, 3, 3]

m = 2
  windows [3,3] [3,3]   same hash twice → no unique → l = 3

m = 3
  window [3,3,3]  → unique → result = 3

answer = 3
```

### Fixing the pasted code

In the pasted version the line

```java
// long p = getPow(base % modulo, sizeOfSubArr - 1);
```

is commented out, but `p` is used when removing the left element. That line must be active (and `base % modulo` is still needed as `p`). The code below restores it.

### State

```text
m     : window length being tested
base  : max(nums) + 1
p     : base^(m-1) mod modulo
hash  : hash of the current window
mp    : hash → true when seen once, false when seen two or more times
```

### Transitions

```text
func(m):
    p = base^(m-1) mod modulo
    slide a window of size m
        add nums[j] to the hash
        once the window has m elements
            mark the hash true if new, false if already seen
            remove nums[i] from the hash, move i and j
    return true if any hash is still marked true

binary search:
    if func(m) → result = m, search smaller
    else       → search larger
```

---

## Java

```java
class Solution {

    private static final long modulo = 1_000_000_007L;

    // Binary exponentiation: a^exp in O(log exp).
    private long getPow(long a, long exp) {

        if (exp == 0) {
            return 1L;
        }

        long half = getPow(a, exp / 2);
        long res = (half * half) % modulo;

        if ((exp % 2) == 1) {
            res = (res * a) % modulo;
        }

        return res;
    }

    // True when some window of length sizeOfSubArr appears exactly once.
    private boolean func(int[] nums, int sizeOfSubArr, long base) {

        int n = nums.length;

        long hash = 0;
        Map<Long, Boolean> mp = new HashMap<>();

        // Weight of the leftmost element in a full window.
        long p = getPow(base % modulo, sizeOfSubArr - 1);

        int i = 0;
        int j = 0;

        while (j < n) {

            hash = (hash * base + nums[j]) % modulo;

            if (j - i + 1 < sizeOfSubArr) {
                j++;
            } else {

                // Seen once → true, seen again → false.
                if (mp.containsKey(hash)) {
                    mp.put(hash, false);
                } else {
                    mp.put(hash, true);
                }

                // Remove nums[i] so the next window slides one step right.
                hash = (hash - (p * nums[i]) % modulo + modulo) % modulo;

                i++;
                j++;
            }
        }

        for (boolean unique : mp.values()) {
            if (unique) {
                return true;
            }
        }

        return false;
    }

    public int smallestUniqueSubarray(int[] nums) {

        int n = nums.length;
        int max = 0;

        for (int num : nums) {
            max = Math.max(max, num);
        }

        long base = max + 1L;

        int l = 1;
        int r = n;
        int result = n;

        while (l <= r) {

            int m = l + (r - l) / 2;

            if (func(nums, m, base)) {
                result = m;
                r = m - 1;
            } else {
                l = m + 1;
            }
        }

        return result;
    }
}
```

The iterative power (commented out in the original) is equivalent:

```java
private long getPow(long a, long exp) {
    long result = 1;
    while (exp > 0) {
        if (exp % 2 == 1) {
            result = (result * a) % modulo;
        }
        a = (a * a) % modulo;
        exp /= 2;
    }
    return result;
}
```

---

## C++

```cpp
class Solution {
public:

    static constexpr long long modulo = 1000000007LL;

    // Binary exponentiation: a^exp in O(log exp).
    long long getPow(long long a, long long exp) {

        long long result = 1;

        while (exp > 0) {
            if (exp % 2 == 1) {
                result = (result * a) % modulo;
            }
            a = (a * a) % modulo;
            exp /= 2;
        }

        return result;
    }

    // True when some window of length sizeOfSubArr appears exactly once.
    bool func(vector<int>& nums, int sizeOfSubArr, long long base) {

        int n = nums.size();

        long long hash = 0;
        unordered_map<long long, bool> mp;

        // Weight of the leftmost element in a full window.
        long long p = getPow(base % modulo, sizeOfSubArr - 1);

        int i = 0;
        int j = 0;

        while (j < n) {

            hash = (hash * base + nums[j]) % modulo;

            if (j - i + 1 < sizeOfSubArr) {
                j++;
            } else {

                // Seen once → true, seen again → false.
                if (mp.count(hash)) {
                    mp[hash] = false;
                } else {
                    mp[hash] = true;
                }

                // Remove nums[i] so the next window slides one step right.
                hash = (hash - (p * nums[i]) % modulo + modulo) % modulo;

                i++;
                j++;
            }
        }

        for (auto& entry : mp) {
            if (entry.second) {
                return true;
            }
        }

        return false;
    }

    int smallestUniqueSubarray(vector<int>& nums) {

        int n = nums.size();
        int maxValue = *max_element(nums.begin(), nums.end());

        long long base = maxValue + 1LL;

        int l = 1;
        int r = n;
        int result = n;

        while (l <= r) {

            int m = l + (r - l) / 2;

            if (func(nums, m, base)) {
                result = m;
                r = m - 1;
            } else {
                l = m + 1;
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
Time  : O(n log n)
        binary search runs O(log n) checks
        each check slides n windows in O(1) each, plus O(log n) for p

Space : O(n)
        the hash map holds at most n windows
        getPow recursion adds O(log n) stack, which is absorbed by O(n)
```

---

# Hash Collisions — Making It Safe

A single modulus can send two **different** windows to the same hash. A collision marks two distinct windows as the same one, and the check can then miss a truly unique window. The probability is small on random data, and test sets can contain arrays built to collide.

Combining two independent moduli makes a collision astronomically unlikely. Keep two hashes, then merge them into one `long` key:

```text
key = h1 * MOD2 + h2        (both below about 10^9, so the key stays under 9 * 10^18)
```

## Java — double hash

```java
class Solution {

    private static final long MOD1 = 1_000_000_007L;
    private static final long MOD2 = 1_000_000_009L;

    private long power(long a, long exp, long mod) {
        long result = 1;
        a %= mod;
        while (exp > 0) {
            if ((exp & 1) == 1) {
                result = (result * a) % mod;
            }
            a = (a * a) % mod;
            exp >>= 1;
        }
        return result;
    }

    private boolean hasUnique(int[] nums, int len, long base) {

        int n = nums.length;

        long p1 = power(base, len - 1, MOD1);
        long p2 = power(base, len - 1, MOD2);

        long h1 = 0;
        long h2 = 0;

        Map<Long, Integer> count = new HashMap<>();

        for (int j = 0; j < n; j++) {

            h1 = (h1 * base + nums[j]) % MOD1;
            h2 = (h2 * base + nums[j]) % MOD2;

            if (j >= len - 1) {

                long key = h1 * MOD2 + h2;
                count.merge(key, 1, Integer::sum);

                int i = j - len + 1;
                h1 = (h1 - (p1 * nums[i]) % MOD1 + MOD1) % MOD1;
                h2 = (h2 - (p2 * nums[i]) % MOD2 + MOD2) % MOD2;
            }
        }

        for (int c : count.values()) {
            if (c == 1) {
                return true;
            }
        }

        return false;
    }

    public int smallestUniqueSubarray(int[] nums) {

        int n = nums.length;
        int max = 0;

        for (int num : nums) {
            max = Math.max(max, num);
        }

        long base = max + 1L;

        int l = 1;
        int r = n;
        int result = n;

        while (l <= r) {

            int m = l + (r - l) / 2;

            if (hasUnique(nums, m, base)) {
                result = m;
                r = m - 1;
            } else {
                l = m + 1;
            }
        }

        return result;
    }
}
```

## C++ — double hash

```cpp
class Solution {
public:

    static constexpr long long MOD1 = 1000000007LL;
    static constexpr long long MOD2 = 1000000009LL;

    long long power(long long a, long long exp, long long mod) {
        long long result = 1;
        a %= mod;
        while (exp > 0) {
            if (exp & 1) {
                result = (result * a) % mod;
            }
            a = (a * a) % mod;
            exp >>= 1;
        }
        return result;
    }

    bool hasUnique(vector<int>& nums, int len, long long base) {

        int n = nums.size();

        long long p1 = power(base, len - 1, MOD1);
        long long p2 = power(base, len - 1, MOD2);

        long long h1 = 0;
        long long h2 = 0;

        unordered_map<long long, int> count;

        for (int j = 0; j < n; j++) {

            h1 = (h1 * base + nums[j]) % MOD1;
            h2 = (h2 * base + nums[j]) % MOD2;

            if (j >= len - 1) {

                long long key = h1 * MOD2 + h2;
                count[key]++;

                int i = j - len + 1;
                h1 = (h1 - (p1 * nums[i]) % MOD1 + MOD1) % MOD1;
                h2 = (h2 - (p2 * nums[i]) % MOD2 + MOD2) % MOD2;
            }
        }

        for (auto& entry : count) {
            if (entry.second == 1) {
                return true;
            }
        }

        return false;
    }

    int smallestUniqueSubarray(vector<int>& nums) {

        int n = nums.size();
        long long base = *max_element(nums.begin(), nums.end()) + 1LL;

        int l = 1;
        int r = n;
        int result = n;

        while (l <= r) {

            int m = l + (r - l) / 2;

            if (hasUnique(nums, m, base)) {
                result = m;
                r = m - 1;
            } else {
                l = m + 1;
            }
        }

        return result;
    }
};
```

The double-hash version has the same complexity: `O(n log n)` time and `O(n)` space.

---

# Pattern Recognition

```text
Shortest subarray that appears exactly once
        ↓
Longer containing windows stay unique, so the answer is monotone in length
        ↓
Binary search the length
        ↓
For each length, hash every window with a rolling hash and count occurrences
        ↓
Combine two moduli to avoid collisions
```

### Remember this trigger:

> **Find the shortest subarray or substring that is unique → binary search the length, slide a rolling hash, and mark each hash seen once or many times. A unique window of length `L` stays unique at `L + 1`.**
