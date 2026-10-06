# LeetCode 921 — Minimum Add to Make Parentheses Valid

## Problem

Given a parentheses string `s` of `'('` and `')'`, return the **minimum number of parentheses** you must insert so that `s` becomes valid.

A string is valid when every open has a matching close, in order, and nothing is left unmatched.

Example:

```text
s      = "())"
answer = 1          insert one '(' at the front → "(())"

s      = "((("
answer = 3          insert three ')' at the end → "((()))"
```

---

# Approach — Stack of Unmatched Opens

### Core idea

Each insertion fixes one unmatched bracket.

```text
')' with no '(' before it   → insert one '('
'(' left over at the end    → insert one ')' for each
```

The stack holds `'('` that have not been closed yet.

```text
'('  → push it. It still needs a close.
')'  → if an open is waiting, pop it. They form a pair.
       if the stack is empty, this ')' needs an inserted '('
```

`cnt` counts those extra opens we had to imagine. At the end, every `'('` still on the stack needs an inserted `')'`.

```text
answer = leftover opens + unmatched closes
       = stk.size() + cnt
```

### Walkthrough

```text
s = "())"

'('  push                 stack: ['(']     cnt = 0
')'  pop                  stack: []        cnt = 0
')'  stack empty          stack: []        cnt = 1

answer = 0 + 1 = 1
```

```text
s = "((("

three pushes, no closes
answer = 3 + 0 = 3
```

### State

```text
stk  : '(' that are still open
cnt  : ')' that arrived with nothing to match
```

### Transitions

```text
'(' and we can match it     → push
')' and stack is not empty  → pop
')' and stack is empty      → cnt++
end                         → stk.size() + cnt
```

---

## Java

```java
class Solution {
    public int minAddToMakeValid(String s) {

        int cnt = 0;
        Stack<Character> stk = new Stack<>();

        for (char ch : s.toCharArray()) {

            if (ch == '(') {
                stk.push(ch);
            } else {

                // This ')' has no open waiting, so insert one '('.
                if (stk.isEmpty()) {
                    cnt++;
                } else {
                    stk.pop();
                }
            }
        }

        // Each leftover '(' needs one inserted ')'.
        return stk.size() + cnt;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minAddToMakeValid(string s) {

        int cnt = 0;
        stack<char> stk;

        for (char ch : s) {

            if (ch == '(') {
                stk.push(ch);
            } else {

                // This ')' has no open waiting, so insert one '('.
                if (stk.empty()) {
                    cnt++;
                } else {
                    stk.pop();
                }
            }
        }

        // Each leftover '(' needs one inserted ')'.
        return (int)stk.size() + cnt;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    one pass, each character is pushed or counted once
Space : O(n)    the stack holds unmatched '('
```

The stack only ever stores `'('`. A single open-count replaces it and drops the extra space to `O(1)`, with the same `cnt` for unmatched `')'`. The logic above is the same either way.

---

# Pattern Recognition

```text
Minimum inserts to make parentheses valid
        ↓
')' with balance 0 needs an inserted '('
leftover '(' each need an inserted ')'
        ↓
Stack of unmatched opens, plus a counter for bad closes
        ↓
answer = leftover opens + unmatched closes
```

### Remember this trigger:

> **Count missing parentheses, not the valid pairs. A close with an empty stack is one insert. Whatever is still open at the end is the rest.**
