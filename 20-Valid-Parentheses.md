# LeetCode 20 — Valid Parentheses

## Problem

Given a string `s` containing only `'('`, `')'`, `'{'`, `'}'`, `'['`, and `']'`, return `true` if the brackets are valid.

A string is valid when:

```text
Open brackets are closed by the same type
Open brackets are closed in the correct order
Every close bracket has a matching open bracket
```

Example:

```text
s      = "()[]{}"
answer = true

s      = "(]"
answer = false

s      = "([)]"
answer = false
```

---

# Approach — Stack

### Core idea

The bracket that must close next is always the **most recent** one that is still open. A stack stores open brackets in that order.

```text
open bracket    → push it
close bracket   → the stack top must be its match, then pop
```

Three failures:

```text
close bracket and the stack is empty     → nothing to match
close bracket and the top is the wrong type
loop ends and the stack still has opens  → some bracket was never closed
```

If none of those happen, the string is valid.

### Walkthrough

```text
s = "([)]"

push '('
push '['
see ')'  top is '['  → mismatch → false
```

```text
s = "{[]}"

push '{'
push '['
see ']'  top is '['  → pop
see '}'  top is '{'  → pop
stack empty → true
```

### State

```text
stk  : unmatched open brackets, most recent on top
```

### Transitions

```text
'(' '{' '['  → push
')' '}' ']'  → empty stack, or top is not the pair → false
               otherwise pop
end          → true only if the stack is empty
```

---

## Java

```java
class Solution {
    public boolean isValid(String s) {

        Stack<Character> stk = new Stack<>();

        for (char ch : s.toCharArray()) {

            if (ch == '(' || ch == '{' || ch == '[') {
                stk.push(ch);
            } else {

                // A close bracket with nothing open is invalid.
                if (stk.isEmpty()) {
                    return false;
                }

                char top = stk.pop();

                if ((ch == '}' && top != '{')
                        || (ch == ']' && top != '[')
                        || (ch == ')' && top != '(')) {
                    return false;
                }
            }
        }

        // Every open bracket must have been closed.
        return stk.isEmpty();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    bool isValid(string s) {

        stack<char> stk;

        for (char ch : s) {

            if (ch == '(' || ch == '{' || ch == '[') {
                stk.push(ch);
            } else {

                // A close bracket with nothing open is invalid.
                if (stk.empty()) {
                    return false;
                }

                char top = stk.top();
                stk.pop();

                if ((ch == '}' && top != '{')
                        || (ch == ']' && top != '[')
                        || (ch == ')' && top != '(')) {
                    return false;
                }
            }
        }

        // Every open bracket must have been closed.
        return stk.empty();
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    each character is pushed or popped at most once
Space : O(n)    the stack holds unmatched open brackets
```

Worst case is a string of only open brackets. The stack then holds all `n` characters.

---

# Pattern Recognition

```text
Brackets must close in the reverse order they opened
        ↓
Stack
        ↓
Push opens
        ↓
A close must match the top, then pop
        ↓
Valid only if the stack is empty at the end
```

### Remember this trigger:

> **Last opened bracket must close first → stack. Check the match on the way out, and reject leftover opens.**
