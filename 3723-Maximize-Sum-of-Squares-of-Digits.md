# LeetCode 3723 — Maximize Sum of Squares of Digits

## Problem

You are given two positive integers `num` and `sum`.

A positive integer is **good** when:

```text
it has exactly num digits
its digits add up to exactly sum
```

The score of a good integer is the **sum of the squares of its digits**.

Return the good integer with the **maximum score**, as a string. If several integers share that score, return the **largest** one. If none exists, return `""`.

The string has exactly `num` characters, so a trailing zero stays. `"30"` is a 2-digit answer, not `"3"`.

Example:

```text
num = 2, sum = 3
good integers: 12, 21, 30
scores:        5,  5,  9
answer = "30"

num = 2, sum = 17
good integers: 89, 98
scores:        145, 145
answer = "98"          same score, larger number

num = 1, sum = 10
answer = ""            one digit cannot add up to 10
```

```text
1 <= num <= 2 * 10^5
1 <= sum <= 2 * 10^6
```

---

# Approach 1 — Backtracking

### Core idea

Fill the `num` positions from the left. Each position is a digit `0` through `9`.

Keep a running digit sum and a running score. Skip a digit that would push the sum past `sum`.

At the last position, the digit sum must be exactly `sum`. Among those strings, keep the one with the larger score. If the scores match, keep the lexicographically larger string.

### Why it is too slow

`num` can be `2 * 10^5`. The search tries up to 10 choices at every position, so the tree is exponential. The recursion is also `num` calls deep, which is far past a normal call stack.

This finds the right answer on tiny inputs and times out on the real limits.

### State

```text
idx      : next digit position
currSum  : digits chosen so far
score    : sum of squares so far
curr     : the number built so far
```

### Transitions

```text
idx == num and currSum == sum  → keep curr if the score is better,
                                  or the score ties and curr is larger
digit d with currSum + d <= sum → append d and recurse
```

---

## Java

```java
class Solution {

    String best = "";
    int maxScore = -1;

    public String maxSumOfSquares(int num, int sum) {
        solve(0, num, sum, 0, 0, new StringBuilder());
        return best;
    }

    void solve(int idx, int num, int target,
               int currSum, int score, StringBuilder curr) {

        if (idx == num) {

            if (currSum == target) {
                String candidate = curr.toString();

                // Higher score wins. Same score keeps the larger number.
                if (score > maxScore
                        || (score == maxScore && candidate.compareTo(best) > 0)) {
                    maxScore = score;
                    best = candidate;
                }
            }

            return;
        }

        for (int digit = 0; digit <= 9; digit++) {

            if (currSum + digit > target) {
                continue;
            }

            curr.append((char) ('0' + digit));

            solve(idx + 1, num, target,
                    currSum + digit,
                    score + digit * digit, curr);

            curr.deleteCharAt(curr.length() - 1);
        }
    }
}
```

---

## C++

```cpp
class Solution {
public:

    string maxSumOfSquares(int num, int sum) {

        best = "";
        maxScore = -1;

        string curr;
        solve(0, num, sum, 0, 0, curr);

        return best;
    }

private:

    string best;
    int maxScore = -1;

    void solve(int idx, int num, int target,
               int currSum, int score, string& curr) {

        if (idx == num) {

            if (currSum == target) {

                // Higher score wins. Same score keeps the larger number.
                if (score > maxScore
                        || (score == maxScore && curr > best)) {
                    maxScore = score;
                    best = curr;
                }
            }

            return;
        }

        for (int digit = 0; digit <= 9; digit++) {

            if (currSum + digit > target) {
                continue;
            }

            curr.push_back('0' + digit);

            solve(idx + 1, num, target,
                  currSum + digit,
                  score + digit * digit, curr);

            curr.pop_back();
        }
    }
};
```

### Complexity

```text
Time  : exponential     up to 10 choices at each of the num positions
Space : O(num)          the recursion depth, which overflows when num is 2 * 10^5
```

---

# Approach 2 — Greedy ⭐

### Core idea

A digit cannot be larger than `9`. If the largest possible total is still too small, no answer exists:

```text
sum > 9 * num  →  ""
```

For a fixed digit sum, the sum of squares is largest when the digits are as unbalanced as possible. Moving `1` from a smaller digit `b` onto a larger digit `a` changes the score by:

```text
(a + 1)² + (b - 1)² - a² - b² = 2(a - b) + 2
```

That change is positive whenever `a >= b` and `b >= 1`. Keep moving until digits are `9`s, at most one leftover digit, and zeros.

The largest such number writes those digits from the left:

```text
9, 9, ..., 9, remainder, 0, 0, ..., 0
```

That one string is both the best score and the largest number. No search is required.

### Walkthrough

```text
num = 2, sum = 3

3 < 9, so the first digit is 3 and the rest is 0
answer = "30"
score  = 9 + 0 = 9
```

```text
num = 2, sum = 17

17 = 9 + 8
answer = "98"
score  = 81 + 64 = 145
```

`"89"` has the same score, but `"98"` is larger.

```text
num = 1, sum = 10

10 > 9 * 1, so the answer is ""
```

### State

```text
currSum  : digit sum still left to place
strb     : digits written from the left
```

### Transitions

```text
sum > 9 * num     → return ""
currSum >= 9      → append '9' and subtract 9
otherwise         → append the leftover digit, then fill the rest with '0'
```

---

## Java

```java
class Solution {
    public String maxSumOfSquares(int num, int sum) {

        StringBuilder strb = new StringBuilder();
        int currSum = sum;

        // Even all 9s cannot reach this digit sum.
        if (sum > 9 * num) {
            return "";
        }

        for (int i = 0; i < num; i++) {

            if (currSum >= 9) {
                strb.append('9');
                currSum -= 9;
            } else {
                // One leftover digit, then the rest must be 0.
                strb.append((char) ('0' + currSum));
                currSum = 0;
                break;
            }
        }

        while (strb.length() < num) {
            strb.append('0');
        }

        return strb.toString();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    string maxSumOfSquares(int num, int sum) {

        // Even all 9s cannot reach this digit sum.
        if (sum > 9 * num) {
            return "";
        }

        string result;
        result.reserve(num);

        int currSum = sum;

        for (int i = 0; i < num; i++) {

            if (currSum >= 9) {
                result.push_back('9');
                currSum -= 9;
            } else {
                // One leftover digit, then the rest must be 0.
                result.push_back('0' + currSum);
                currSum = 0;
                break;
            }
        }

        while ((int) result.size() < num) {
            result.push_back('0');
        }

        return result;
    }
};
```

### Complexity

```text
Time  : O(num)    each of the num digits is written once
Space : O(num)    the returned string has length num
```

---

# Pattern Recognition

```text
Fixed length, fixed digit sum, maximize sum of digit squares
        ↓
Squares prefer unbalanced digits, and a digit stops at 9
        ↓
Use as many 9s as possible, then one remainder, then zeros
        ↓
Write them from the left so the string is also the largest
        ↓
sum > 9 * num means no such number exists
```

### Remember this trigger:

> **Maximize a sum of squares under a digit cap → pack 9s first. The same left-to-right packing is the lexicographically largest number.**
