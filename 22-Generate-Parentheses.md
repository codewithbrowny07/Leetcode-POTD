# LeetCode 22 — Generate Parentheses

## Problem

Given `n` pairs of parentheses, return every **well-formed** string that uses exactly `n` pairs.

Example:

```text
n = 3

[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

`1 <= n <= 8`, so generating the full set is fine.

The count of these strings is the **nth Catalan number**.

---

# Approach 1 — Generate every string, then check

### Core idea

A string of `n` pairs has length `2n`. Every position is either `'('` or `')'`, so there are `2^(2n)` candidates.

Build each one in a char array, then keep it only if the balance rules pass:

```text
'('  → balance++
')'  → balance--
balance < 0 at any point  → invalid
balance == 0 at the end   → valid
```

Most candidates are thrown away. This is the brute-force baseline.

### State

```text
arr  : the string being built, length 2n
idx  : next position to fill
```

### Transitions

```text
idx == 2n        → keep the string only if isValid
otherwise        → try '(' at idx, then ')' at idx
```

---

## Java

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();

        char[] arr = new char[2 * n];

        generate(arr, 0, ans);

        return ans;
    }

    private void generate(char[] arr, int idx, List<String> ans) {

        // Every position is filled. Keep it only if the brackets match.
        if (idx == arr.length) {
            if (isValid(arr)) {
                ans.add(new String(arr));
            }
            return;
        }

        arr[idx] = '(';
        generate(arr, idx + 1, ans);

        arr[idx] = ')';
        generate(arr, idx + 1, ans);
    }

    private boolean isValid(char[] arr) {
        int balance = 0;

        for (char ch : arr) {
            if (ch == '(') {
                balance++;
            } else {
                balance--;
            }

            // More ')' than '(' so far.
            if (balance < 0) {
                return false;
            }
        }

        return balance == 0;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<string> generateParenthesis(int n) {

        vector<string> ans;
        vector<char> arr(2 * n);

        generate(arr, 0, ans);

        return ans;
    }

private:

    void generate(vector<char>& arr, int idx, vector<string>& ans) {

        // Every position is filled. Keep it only if the brackets match.
        if (idx == (int)arr.size()) {
            if (isValid(arr)) {
                ans.push_back(string(arr.begin(), arr.end()));
            }
            return;
        }

        arr[idx] = '(';
        generate(arr, idx + 1, ans);

        arr[idx] = ')';
        generate(arr, idx + 1, ans);
    }

    bool isValid(const vector<char>& arr) {

        int balance = 0;

        for (char ch : arr) {
            if (ch == '(') {
                balance++;
            } else {
                balance--;
            }

            // More ')' than '(' so far.
            if (balance < 0) {
                return false;
            }
        }

        return balance == 0;
    }
};
```

### Complexity

```text
Time  : O(4^n · n)    2^(2n) = 4^n strings, each checked in O(n)
Space : O(n)          recursion depth and the char buffer, besides the answer
```

---

# Approach 2 — Backtracking ⭐

### Core idea

Do not build invalid strings at all. Track how many opens and closes are already used.

```text
place '('  only while open < n
place ')'  only while close < open
```

`close < open` is the same balance check as approach 1, applied **while building** instead of after. Every string that reaches length `2n` is valid, and every valid string is reached once.

### State

```text
curr   : string built so far
open   : number of '(' used
close  : number of ')' used
```

### Transitions

```text
length == 2n   → save curr
open < n       → append '('
close < open   → append ')'
```

---

## Java

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();
        backtrack("", 0, 0, n, ans);
        return ans;
    }

    private void backtrack(String curr, int open, int close, int n, List<String> ans) {

        // Every complete string that reaches here is valid.
        if (curr.length() == 2 * n) {
            ans.add(curr);
            return;
        }

        if (open < n) {
            backtrack(curr + "(", open + 1, close, n, ans);
        }

        if (close < open) {
            backtrack(curr + ")", open, close + 1, n, ans);
        }
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<string> generateParenthesis(int n) {

        vector<string> ans;
        string curr;

        backtrack(curr, 0, 0, n, ans);

        return ans;
    }

private:

    void backtrack(string& curr, int open, int close, int n, vector<string>& ans) {

        // Every complete string that reaches here is valid.
        if ((int)curr.size() == 2 * n) {
            ans.push_back(curr);
            return;
        }

        if (open < n) {
            curr.push_back('(');
            backtrack(curr, open + 1, close, n, ans);
            curr.pop_back();
        }

        if (close < open) {
            curr.push_back(')');
            backtrack(curr, open, close + 1, n, ans);
            curr.pop_back();
        }
    }
};
```

The C++ version mutates one buffer and undoes the choice with `pop_back`. The Java version copies `curr` on each call. Both explore the same tree.

### Complexity

Let `C(n)` be the nth Catalan number, about `4^n / (n^(3/2) · sqrt(π))`.

```text
Time  : O(C(n) · n)    only valid strings are built, each of length 2n
Space : O(n)           recursion depth, besides the answer
```

The Java concatenations copy the prefix at every call, so that version is `O(C(n) · n²)` in the constant factors of string copying. The search tree is the same.

---

# Approach 3 — Catalan DP

### The recurrence

Every valid string with `pairs` pairs has this shape:

```text
( A ) B
```

- `A` is a valid string
- `B` is a valid string
- the outer `(` `)` are one pair

If `A` uses `i` pairs, then `B` uses the rest:

```text
i + 1 + (pairs - 1 - i) = pairs
A has i pairs
B has pairs - 1 - i pairs
```

`i` runs from `0` to `pairs - 1`.

That is the Catalan recurrence:

```text
C(0) = 1                         one empty string

C(pairs) = Σ C(i) · C(pairs - 1 - i)
           i = 0 .. pairs - 1
```

`C(n)` is exactly how many answers this problem has.

```text
n    C(n)
0    1
1    1          ()
2    2          ()()  (())
3    5
```

### Core idea

`dp[k]` stores every valid string that uses `k` pairs.

```text
dp[0] = [""]

for pairs = 1 .. n:
    for i = 0 .. pairs - 1:
        for every A in dp[i]:
            for every B in dp[pairs - 1 - i]:
                dp[pairs] += "(" + A + ")" + B
```

Nothing invalid is ever created, and each valid string is created once, because every valid string has exactly one way to peel off the pair that matches the first `'('`.

### Walkthrough for `n = 2`

```text
dp[0] = [""]

pairs = 1
  i = 0 → "(" + "" + ")" + "" = "()"
dp[1] = ["()"]

pairs = 2
  i = 0 → "(" + "" + ")" + "()" = "()()"
  i = 1 → "(" + "()" + ")" + "" = "(())"
dp[2] = ["()()", "(())"]
```

---

## Java

```java
class Solution {
    public List<String> generateParenthesis(int n) {

        List<List<String>> dp = new ArrayList<>();

        // dp[i] = all valid strings with i pairs.
        for (int i = 0; i <= n; i++) {
            dp.add(new ArrayList<>());
        }

        // C(0) = 1. The empty string is the only string with 0 pairs.
        dp.get(0).add("");

        for (int pairs = 1; pairs <= n; pairs++) {

            // i = pairs inside the first ().
            // pairs - 1 - i = pairs after it.
            for (int i = 0; i < pairs; i++) {

                List<String> inside = dp.get(i);
                List<String> after = dp.get(pairs - 1 - i);

                for (String a : inside) {
                    for (String b : after) {
                        dp.get(pairs).add("(" + a + ")" + b);
                    }
                }
            }
        }

        return dp.get(n);
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<string> generateParenthesis(int n) {

        // dp[i] = all valid strings with i pairs.
        vector<vector<string>> dp(n + 1);

        // C(0) = 1. The empty string is the only string with 0 pairs.
        dp[0].push_back("");

        for (int pairs = 1; pairs <= n; pairs++) {

            // i = pairs inside the first ().
            // pairs - 1 - i = pairs after it.
            for (int i = 0; i < pairs; i++) {

                for (const string& a : dp[i]) {
                    for (const string& b : dp[pairs - 1 - i]) {
                        dp[pairs].push_back("(" + a + ")" + b);
                    }
                }
            }
        }

        return dp[n];
    }
};
```

### Complexity

```text
Time  : O(C(n) · n)    each answer, and each smaller string, is built once
Space : O(C(n) · n)    dp stores every valid string of every size up to n
```

`C(n)` is about `4^n / (n^(3/2) · sqrt(π))`. For `n <= 8` that is 1430 strings.

---

# Pattern Recognition

```text
All valid parentheses of length 2n
        ↓
Brute force: 4^n candidates, filter with a balance scan
        ↓
Backtracking: place ')' only when it cannot make the balance negative
        ↓
Catalan DP: every valid string is (A)B
            C(n) = Σ C(i) · C(n - 1 - i)
```

### Remember this trigger:

> **Count or list valid parentheses → Catalan. Either prune with `close < open`, or build `(A)B` from smaller answers.**
