# LeetCode 2243 — Calculate Digit Sum of a String

## Problem

You are given a digit string `s` and an integer `k`.

While `s.length() > k`, replace `s` with a shorter digit string:

1. Split `s` into consecutive groups of length `k`. The last group may be shorter.
2. Replace each group with the decimal form of the sum of its digits.
3. Join those sums, in order, to form the next `s`.

Stop when the length is at most `k`, and return that string.

Example:

```text
s = "11111222223"
k = 3

"111" "112" "222" "23"
  3     4     6     5     → "3465"

"346" "5"
 13    5                  → "135"

"135" has length 3, which is not greater than k

answer = "135"
```

```text
s = "00000000"
k = 3

"000" "000" "00"
  0     0     0           → "000"

answer = "000"
```

```text
2 <= s.length <= 100
2 <= k <= 100
s consists of digits
```

A sum of `10` or more stays a multi-digit string. `"346"` becomes `"13"`, not `"1"` and `"3"`. A sum of `0` stays the character `'0'`, so leading zeros are preserved.

---

# Approach — Fold Groups Until the String Is Short Enough

### Core idea

One round walks the string in steps of `k`. Each step reads one group, adds `ch - '0'` for every character, and appends that integer to a builder. Appending an `int` writes every digit of the sum, so `13` contributes two characters.

The builder becomes the next `s`. The loop repeats while the length is still greater than `k`. If the original length is already at most `k`, the loop never runs and `s` is returned unchanged.

### Walkthrough

```text
s = "11111222223", k = 3

round 1
  i = 0  group "111"  sum 3
  i = 3  group "112"  sum 4
  i = 6  group "222"  sum 6
  i = 9  group "23"   sum 5
  next s = "3465"

round 2
  i = 0  group "346"  sum 13
  i = 3  group "5"    sum 5
  next s = "135"

length 3 <= k, return "135"
```

`Math.min(i + k, s.length())` is what keeps the last group `"23"` inside the string.

### State

```text
s    : digit string for the current round
strb : sums of this round, joined in order
sum  : digit sum of the group that starts at i
```

### Transitions

```text
while s.length() > k
    clear the builder
    for i = 0, k, 2k, ...
        sum the digits in s[i .. min(i + k, length))
        append the decimal form of sum
    s = builder
return s
```

---

## Java

```java
class Solution {
    public String digitSum(String s, int k) {

        while (s.length() > k) {

            StringBuilder strb = new StringBuilder();

            for (int i = 0; i < s.length(); i += k) {

                // Last group stops at the end of the string.
                String group = s.substring(i, Math.min(i + k, s.length()));

                int sum = 0;
                for (char ch : group.toCharArray()) {
                    sum += ch - '0';
                }

                // append(int) writes the full decimal value, so 13 stays "13".
                strb.append(sum);
            }

            s = strb.toString();
        }

        return s;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    string digitSum(string s, int k) {

        while ((int)s.size() > k) {

            string next;

            for (int i = 0; i < (int)s.size(); i += k) {

                int sum = 0;
                int end = min(i + k, (int)s.size());

                for (int j = i; j < end; j++) {
                    sum += s[j] - '0';
                }

                // to_string writes the full decimal value, so 13 stays "13".
                next += to_string(sum);
            }

            s = next;
        }

        return s;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)
Space : O(n)
```

One round reads each character of the current string a constant number of times. `substring` and `toCharArray` copy that group, and those copies still add up to the current length.

The pasted comment says `O(n^3)`, from treating the `while` loop, the group loop, and the character loop as three independent `O(n)` factors. Inside one round the character loops together touch each character once, so that round is `O(current length)`.

The length also shrinks. A group of `k` digits (`k >= 2`) becomes the decimal form of a sum at most `9k`, which is at most three digits. With `k = 2` and large digits, one round can keep the length the same, because `"99"` sums to `18`. The next round turns `"18"` into `"9"`, and the length then drops by about half each pair of rounds. The scanned lengths form a geometric series, so the total work is `O(n)`.

The builder and the next string hold at most a few characters per group. That is `O(n)` extra space. With `n <= 100` the bound is also easy to see directly: a handful of rounds, each scanning at most about `100` characters.

---

# Pattern Recognition

```text
A digit string longer than k
        ↓
Cut it into consecutive blocks of k
        ↓
Replace each block by the decimal form of its digit sum
        ↓
Repeat until the length is at most k
```

### Remember this trigger:

> **Replace a digit string by group sums of size `k` until it is short enough → one round is a linear scan that appends each sum as a decimal string, and the rounds together are `O(n)` because the string shrinks geometrically.**
