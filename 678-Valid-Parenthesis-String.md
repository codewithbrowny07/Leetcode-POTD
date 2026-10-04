# LeetCode 678 — Valid Parenthesis String

## Problem

Given a string `s` containing only `'('`, `')'`, and `'*'`, return `true` if `s` can be a valid parentheses string.

`'*'` may be used as one of three things:

```text
'('
')'
empty
```

A string is valid when every close matches an earlier open, and nothing is left unmatched. An empty string is valid.

Example:

```text
s      = "(*))"
answer = true
```

One reading is `(())`: the `'*'` is an open. Another is `())` with the `'*'` empty, which is invalid, but one valid reading is enough.

```text
s      = "(((*)"
answer = false
```

---

# Approach — Two Stacks of Indices

### Core idea

A `')'` must be matched immediately, from the left. An unmatched `'('` can wait, because a later `'*'` might close it.

Keep two stacks of **indices**:

```text
open  → indices of '(' that still need a close
star  → indices of '*' that have not been used yet
```

On each character:

```text
'('  → push its index onto open
'*'  → push its index onto star
')'  → pop an '(' if one exists
       otherwise pop a '*' and treat it as '('
       otherwise this ')' cannot be matched
```

Using a real `'('` before a `'*'` is safe. A `'*'` is more flexible, so it should be saved for later.

After the scan, every `')'` is handled. What remains is extra `'('` and extra `'*'`.

A leftover `'('` can only be closed by a `'*'` that sits **to its right**, because that `'*'` has to act as `')'`. Pair them from the end:

```text
pop the latest '(' and the latest '*'
if '(' index > '*' index, the star is too far left → false
```

Stars that are never used can be empty. Leftover `'('` with no star left cannot.

### Walkthrough

```text
s = "(*))"
     0123

i = 0  '('  open: [0]
i = 1  '*'  star: [1]
i = 2  ')'  pop open            open: []     star: [1]
i = 3  ')'  no '(', pop star    open: []     star: []

open is empty → true
```

```text
s = "(*("
     012

i = 0  '('  open: [0]
i = 1  '*'  star: [1]
i = 2  '('  open: [0, 2]

pair latest '(' at 2 with latest '*' at 1
2 > 1 → the star is before that '(', so it cannot close it → false
```

### State

```text
open  : unmatched '(' indices
star  : unused '*' indices
```

### Transitions

```text
'('                         → open.push(i)
'*'                         → star.push(i)
')' and open is not empty   → open.pop()
')' and star is not empty   → star.pop()          '*' used as '('
')' and both empty          → false
after the scan              → each leftover '(' needs a later '*'
done                        → true only if open is empty
```

---

## Java

```java
class Solution {
    public boolean checkValidString(String s) {

        Stack<Integer> open = new Stack<>();
        Stack<Integer> star = new Stack<>();

        for (int i = 0; i < s.length(); i++) {

            char ch = s.charAt(i);

            if (ch == '(') {
                open.push(i);
            } else if (ch == '*') {
                star.push(i);
            } else {

                // Prefer a real '(' . Save '*' for a later unmatched '('.
                if (!open.isEmpty()) {
                    open.pop();
                } else if (!star.isEmpty()) {
                    star.pop();
                } else {
                    return false;
                }
            }
        }

        // A leftover '(' can only be closed by a '*' to its right.
        while (!open.isEmpty() && !star.isEmpty()) {

            int openIndex = open.pop();
            int starIndex = star.pop();

            if (openIndex > starIndex) {
                return false;
            }
        }

        return open.isEmpty();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    bool checkValidString(string s) {

        stack<int> open;
        stack<int> star;

        for (int i = 0; i < (int)s.size(); i++) {

            char ch = s[i];

            if (ch == '(') {
                open.push(i);
            } else if (ch == '*') {
                star.push(i);
            } else {

                // Prefer a real '(' . Save '*' for a later unmatched '('.
                if (!open.empty()) {
                    open.pop();
                } else if (!star.empty()) {
                    star.pop();
                } else {
                    return false;
                }
            }
        }

        // A leftover '(' can only be closed by a '*' to its right.
        while (!open.empty() && !star.empty()) {

            int openIndex = open.top();
            open.pop();

            int starIndex = star.top();
            star.pop();

            if (openIndex > starIndex) {
                return false;
            }
        }

        return open.empty();
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    each index is pushed once and popped at most once
Space : O(n)    the two stacks hold indices
```

---

# Pattern Recognition

```text
Parentheses plus a wildcard that can be '(' , ')' , or empty
        ↓
')' must be matched now
'(' can wait for a later '*'
        ↓
Two stacks of indices
        ↓
On ')' use a '(' first, then a '*'
        ↓
Afterwards, every leftover '(' needs a '*' with a greater index
```

### Remember this trigger:

> **`'*'` in a parentheses string → store indices. Match `')'` immediately, then close leftover `'('` only with a star that comes after it.**
