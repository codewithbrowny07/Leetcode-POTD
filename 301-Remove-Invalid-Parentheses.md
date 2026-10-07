# LeetCode 301 — Remove Invalid Parentheses

## Problem

Given a string `s` of letters, `'('`, and `')'`, remove the **minimum** number of parentheses so the string is valid.

Return **every** unique string that can be formed that way.

A string is valid when the balance never goes negative and ends at `0`. Letters are never removed.

Example:

```text
s      = "()())()"
answer = ["(())()", "()()()"]

s      = "(a)())()"
answer = ["(a())()", "(a)()()"]
```

Both answers remove one `')'`. Removing two would also be valid, but it is not minimum.

---

# Approach 1 — Backtracking

### Core idea

Walk the string once. A letter must stay. A parenthesis has two choices: **keep** it or **remove** it.

`balance` is the number of unmatched `'('` in `curr`.

```text
balance < 0  → this prefix already has too many ')'  → stop
```

At the end of `s`:

```text
balance == 0 and curr is longer than anything saved  → this is a new minimum-removal answer
                                                         clear the old set
balance == 0 and curr has that same length           → another answer with the same removals
```

Longest valid `curr` means fewest deletions, because every letter is kept and only parentheses are optional.

A `HashSet` drops duplicates. `"()())()"` can reach `"()()()"` by deleting different copies of `')'`.

### State

```text
i        : index in s
curr     : string kept so far
balance  : unmatched '(' in curr
maxLen   : longest valid length found
ans      : unique strings of that length
```

### Transitions

```text
letter        → append it, move on, undo
'(' keep      → balance + 1
')' keep      → balance - 1
either remove → balance unchanged
i == n        → save curr only when balance == 0 and length == maxLen
```

---

## Java

```java
class Solution {

    Set<String> ans = new HashSet<>();
    int maxLen = 0;

    void backtrack(String s, int i, StringBuilder curr, int balance) {

        // Too many ')' already. No way to fix this prefix.
        if (balance < 0) {
            return;
        }

        if (i == s.length()) {

            if (balance == 0) {

                // Longer string means fewer deletions. Drop the old answers.
                if (curr.length() > maxLen) {
                    maxLen = curr.length();
                    ans.clear();
                }

                if (curr.length() == maxLen) {
                    ans.add(curr.toString());
                }
            }

            return;
        }

        char ch = s.charAt(i);

        // Letters are never removed.
        if (ch != '(' && ch != ')') {
            curr.append(ch);
            backtrack(s, i + 1, curr, balance);
            curr.deleteCharAt(curr.length() - 1);
            return;
        }

        // Choice 1: keep this parenthesis.
        curr.append(ch);

        if (ch == '(') {
            backtrack(s, i + 1, curr, balance + 1);
        } else {
            backtrack(s, i + 1, curr, balance - 1);
        }

        curr.deleteCharAt(curr.length() - 1);

        // Choice 2: delete this parenthesis.
        backtrack(s, i + 1, curr, balance);
    }

    public List<String> removeInvalidParentheses(String s) {
        backtrack(s, 0, new StringBuilder(), 0);
        return new ArrayList<>(ans);
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<string> removeInvalidParentheses(string s) {

        maxLen = 0;
        ans.clear();

        string curr;
        backtrack(s, 0, curr, 0);

        return vector<string>(ans.begin(), ans.end());
    }

private:

    unordered_set<string> ans;
    int maxLen = 0;

    void backtrack(const string& s, int i, string& curr, int balance) {

        // Too many ')' already. No way to fix this prefix.
        if (balance < 0) {
            return;
        }

        if (i == (int)s.size()) {

            if (balance == 0) {

                // Longer string means fewer deletions. Drop the old answers.
                if ((int)curr.size() > maxLen) {
                    maxLen = curr.size();
                    ans.clear();
                }

                if ((int)curr.size() == maxLen) {
                    ans.insert(curr);
                }
            }

            return;
        }

        char ch = s[i];

        // Letters are never removed.
        if (ch != '(' && ch != ')') {
            curr.push_back(ch);
            backtrack(s, i + 1, curr, balance);
            curr.pop_back();
            return;
        }

        // Choice 1: keep this parenthesis.
        curr.push_back(ch);

        if (ch == '(') {
            backtrack(s, i + 1, curr, balance + 1);
        } else {
            backtrack(s, i + 1, curr, balance - 1);
        }

        curr.pop_back();

        // Choice 2: delete this parenthesis.
        backtrack(s, i + 1, curr, balance);
    }
};
```

### Complexity

Let `n = s.length()`.

```text
Time  : O(2^n · n)    each parenthesis is kept or removed, and each string is copied into the set
Space : O(n)          recursion depth and the builder, besides the answer set
```

The `balance < 0` prune cuts a lot of branches. The search is still exponential.

---

# Approach 2 — BFS

### Core idea

Minimum deletions means the valid strings **closest** to `s`.

BFS removes one parenthesis per step. Every string in the same queue level has had the **same number of deletions**.

```text
level 0  → original string, 0 removals
level 1  → delete exactly one parenthesis
level 2  → delete exactly two
```

The first level that contains a valid string is the minimum. Collect every valid string on that level, and do not go deeper.

`seen` stops the same string from being built twice. Deleting index `1` then `3`, or `3` then `1`, can produce one string.

Once any string on the current level is valid, later invalid strings on that level do not generate children. Those children would need one more deletion.

### State

```text
q      : strings reached with the current number of deletions
seen   : strings already queued
result : valid strings on the first valid level
```

### Transitions

```text
valid string     → save it, mark this level as done
invalid string   → delete one '(' or ')' at a time and queue the new string
level finished   → if any valid string was found, stop
```

---

## Java

```java
class Solution {

    public List<String> removeInvalidParentheses(String s) {

        List<String> result = new ArrayList<>();

        Queue<String> q = new LinkedList<>();
        Set<String> seen = new HashSet<>();

        q.add(s);
        seen.add(s);

        while (!q.isEmpty()) {

            int levelSize = q.size();
            boolean validFound = false;

            while (levelSize-- > 0) {

                String str = q.remove();

                if (valid(str)) {
                    result.add(str);
                    validFound = true;
                    continue;
                }

                // This level already has a minimum answer.
                // Children would delete one more character.
                if (validFound) {
                    continue;
                }

                for (int i = 0; i < str.length(); i++) {

                    char ch = str.charAt(i);

                    if (ch != '(' && ch != ')') {
                        continue;
                    }

                    String next = str.substring(0, i) + str.substring(i + 1);

                    if (seen.add(next)) {
                        q.add(next);
                    }
                }
            }

            // First valid level is the minimum number of removals.
            if (validFound) {
                break;
            }
        }

        return result;
    }

    private boolean valid(String s) {

        int balance = 0;

        for (char ch : s.toCharArray()) {

            if (ch == '(') {
                balance++;
            } else if (ch == ')') {
                balance--;

                if (balance < 0) {
                    return false;
                }
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

    vector<string> removeInvalidParentheses(string s) {

        vector<string> result;

        queue<string> q;
        unordered_set<string> seen;

        q.push(s);
        seen.insert(s);

        while (!q.empty()) {

            int levelSize = q.size();
            bool validFound = false;

            while (levelSize-- > 0) {

                string str = q.front();
                q.pop();

                if (valid(str)) {
                    result.push_back(str);
                    validFound = true;
                    continue;
                }

                // This level already has a minimum answer.
                // Children would delete one more character.
                if (validFound) {
                    continue;
                }

                for (int i = 0; i < (int)str.size(); i++) {

                    char ch = str[i];

                    if (ch != '(' && ch != ')') {
                        continue;
                    }

                    string next = str.substr(0, i) + str.substr(i + 1);

                    if (seen.insert(next).second) {
                        q.push(next);
                    }
                }
            }

            // First valid level is the minimum number of removals.
            if (validFound) {
                break;
            }
        }

        return result;
    }

private:

    bool valid(const string& s) {

        int balance = 0;

        for (char ch : s) {

            if (ch == '(') {
                balance++;
            } else if (ch == ')') {
                balance--;

                if (balance < 0) {
                    return false;
                }
            }
        }

        return balance == 0;
    }
};
```

### Complexity

Let `n = s.length()`.

```text
Time  : O(2^n · n²)   each subset of deletions can be built, and each string is scanned and sliced
Space : O(2^n · n)    the queue and the seen set store those strings
```

BFS stops at the first valid level, so it does not explore deeper deletions. The bound is still exponential in the worst case.

---

# Pattern Recognition

```text
Remove the fewest parentheses, return every way
        ↓
Minimum deletions = longest valid string that keeps every letter
        ↓
Backtracking: keep or delete each parenthesis, prune balance < 0
        ↓
BFS: each level deletes one more parenthesis
     the first valid level is the answer
```

### Remember this trigger:

> **Minimum removals, all unique results → either build the longest valid string, or BFS outward and stop at the first valid level.**
