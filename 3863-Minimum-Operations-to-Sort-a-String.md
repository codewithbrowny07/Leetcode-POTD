# LeetCode 3863 — Minimum Operations to Sort a String

## Problem

You are given a lowercase string `s`.

In one operation, choose any **proper substring**—a substring that is **not the whole string**—and sort its characters in non-decreasing order.

Return the minimum operations needed to make the entire string non-decreasing. Return `-1` if it is impossible.

Example:

```text
s = "dog"
answer = 1

Sort "og" → "go"
"dog" → "dgo"
```

```text
s = "card"
answer = 2

Sort "car" → "acr"     "card" → "acrd"
Sort "rd"  → "dr"      "acrd" → "acdr"
```

```text
s = "gf"
answer = -1
```

The only substring containing both characters is the entire string, which cannot be selected.

```text
1 <= s.length <= 10^5
```

---

# Approach — Case Analysis

### Core idea

You may sort any substring except the entire string. The answer is always one of:

```text
0, 1, 2, 3, or -1
```

The cases depend on:

```text
whether s is already sorted
the smallest character in s
the largest character in s
where those extreme characters appear
```

### Case 0 — Already sorted

```text
s[i - 1] <= s[i] for every i
```

No operation is needed.

---

### Case -1 — Unsorted string of length 2

The only substring that could swap both characters is the entire string. That operation is forbidden.

```text
"gf" → -1
```

---

### Case 1 — Minimum is already first, or maximum is already last

If the global smallest character already occupies index `0`, sort the suffix:

```text
minimum at s[0]  → sort s[1..n-1]
```

That leaves the minimum in front and sorts everything after it.

Similarly, if the global largest character is already at the end, sort the prefix:

```text
maximum at s[n-1]  → sort s[0..n-2]
```

```text
"dog"
min = 'd', and s[0] is 'd'
sort suffix "og" → "go"
answer = 1
```

---

### Case 2 — An extreme character appears in the middle

If the string was not covered by the one-operation case, but an occurrence of the global minimum or maximum is somewhere in the interior:

```text
s[i] is min or max for some 0 < i < n - 1
```

two proper-substring sorts are enough: one can move that extreme value to its correct edge, and the other sorts the remaining side.

This case includes duplicates. For example, if the first character is a maximum and the last character is a minimum, but another minimum or maximum occurs in the middle, the answer is `2`.

---

### Case 3 — Unique maximum at the front and unique minimum at the end

If none of the previous cases worked, then:

```text
s[0]     is the only global maximum
s[n - 1] is the only global minimum
```

Neither endpoint can be included in a one-operation sort that preserves the other required edge. No interior copy of either extreme exists to help. Three operations are necessary.

```text
s = "cba"
min = 'a' at the end
max = 'c' at the front
no min/max in the middle
answer = 3
```

### Decision tree

```text
already non-decreasing?                 → 0
unsorted and n == 2?                    → -1
first character is global min?          → 1
last character is global max?           → 1
an interior min or max exists?          → 2
otherwise                               → 3
```

---

## Java — Linear Scan ⭐

```java
class Solution {
    public int minOperations(String s) {

        int n = s.length();
        char minChar = s.charAt(0);
        char maxChar = s.charAt(0);
        boolean sorted = true;

        // Find the extremes and check whether s is already sorted.
        for (int i = 1; i < n; i++) {
            char ch = s.charAt(i);

            minChar = (char) Math.min(minChar, ch);
            maxChar = (char) Math.max(maxChar, ch);

            if (ch < s.charAt(i - 1)) {
                sorted = false;
            }
        }

        if (sorted) {
            return 0;
        }

        // An unsorted two-character string cannot sort its whole string.
        if (n == 2) {
            return -1;
        }

        // Sort the other proper substring once.
        if (s.charAt(0) == minChar || s.charAt(n - 1) == maxChar) {
            return 1;
        }

        // A middle copy of either extreme makes two operations enough.
        for (int i = 1; i < n - 1; i++) {
            if (s.charAt(i) == minChar || s.charAt(i) == maxChar) {
                return 2;
            }
        }

        // Only maximum at the front and minimum at the end remain.
        return 3;
    }
}
```

---

## C++ — Linear Scan ⭐

```cpp
class Solution {
public:

    int minOperations(string s) {

        int n = s.size();
        char minChar = s[0];
        char maxChar = s[0];
        bool sorted = true;

        // Find the extremes and check whether s is already sorted.
        for (int i = 1; i < n; i++) {
            minChar = min(minChar, s[i]);
            maxChar = max(maxChar, s[i]);

            if (s[i] < s[i - 1]) {
                sorted = false;
            }
        }

        if (sorted) {
            return 0;
        }

        // An unsorted two-character string cannot sort its whole string.
        if (n == 2) {
            return -1;
        }

        // Sort the other proper substring once.
        if (s[0] == minChar || s[n - 1] == maxChar) {
            return 1;
        }

        // A middle copy of either extreme makes two operations enough.
        for (int i = 1; i < n - 1; i++) {
            if (s[i] == minChar || s[i] == maxChar) {
                return 2;
            }
        }

        // Only maximum at the front and minimum at the end remain.
        return 3;
    }
};
```

---

# Your Sorting Version

Your code has the same logical case split:

```text
sort a copy to detect “already sorted” and find min/max
count min/max occurrences
use the unique max-front / min-end case for answer 3
```

It is correct, but these complexity comments:

```text
// TC : O(NLOGN)
// SC : O(1)
```

need one correction:

```text
Time  : O(n log n)   Arrays.sort(chars) dominates
Space : O(n)         toCharArray() creates a length-n copy
```

The direct scan above computes the same facts without sorting:

```text
Time  : O(n)
Space : O(1)
```

---

# Complexity

For the linear-scan solution:

```text
Time  : O(n)    up to two full scans
Space : O(1)    only characters and booleans
```

---

# Pattern Recognition

```text
Sort any proper substring, but never the full string
        ↓
The global min must reach the front
The global max must reach the end
        ↓
Already sorted? 0
Unsorted length 2? impossible
Min already first or max already last? 1
Interior min/max available? 2
Otherwise 3
```

### Remember this trigger:

> **A proper-substring sort cannot move both endpoints at once. For the minimum operation count, classify where the global minimum and maximum occur.**
