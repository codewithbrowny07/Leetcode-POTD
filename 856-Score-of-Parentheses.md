# LeetCode 856 — Score of Parentheses

## Problem

Given a valid parentheses string `s`, return its **score**.

The scoring rules are:

```text
()      →  1
AB      →  A + B     two adjacent valid strings
(A)     →  2 * A     a valid string inside one extra pair
```

Example:

```text
s      = "()"
answer = 1

s      = "(())"
answer = 2

s      = "()()"
answer = 2

s      = "(()(()))"
answer = 6
```

`(()(()))` is `(A)` where `A` is `()(())` with score `1 + 2 = 3`, so the outer pair doubles it to `6`.

---

# Approach — Stack of Partial Scores

### Core idea

Every open `'('` starts a **new inner context**. Push `0` for the score of that context.

When a `')'` arrives, pop the inner score:

```text
inner == 0  → this pair was empty     ()  → score 1
inner  > 0  → this pair wrapped A     (A) → score 2 * inner
```

That score belongs to the **outer** context, which is now on top of the stack. Add it there:

```text
outer = outer + score
```

That is the `AB → A + B` rule. Adjacent pieces in the same context are summed.

A dummy `0` sits at the bottom so the whole string has a place to accumulate.

### Walkthrough

```text
s = "(()(()))"

start          [0]

'('            [0, 0]
'('            [0, 0, 0]
')'            inner 0 → 1, add to parent     [0, 1]
'('            [0, 1, 0]
'('            [0, 1, 0, 0]
')'            inner 0 → 1, add to parent     [0, 1, 1]
')'            inner 1 → 2, add to parent     [0, 3]
')'            inner 3 → 6, add to parent     [6]
```

The last pop is the score of the whole string.

### State

```text
stk  : score of each unclosed '(' context
       the bottom 0 is the whole-string total
```

### Transitions

```text
'('              → push 0          start a new inner score
')'              → inner = pop()
                   score = 1 if inner == 0 else 2 * inner
                   top += score    add this piece to the parent
end              → the remaining value is the answer
```

---

## Java

```java
class Solution {
    public int scoreOfParentheses(String s) {

        Stack<Integer> stk = new Stack<>();

        // Score of the whole string so far.
        stk.push(0);

        for (char ch : s.toCharArray()) {

            if (ch == '(') {
                // New inner context starts at 0.
                stk.push(0);
            } else {

                int inner = stk.pop();

                int score;
                if (inner == 0) {
                    score = 1;          // ()
                } else {
                    score = 2 * inner;  // (A)
                }

                // AB → A + B. Add this piece to the parent context.
                stk.push(stk.pop() + score);
            }
        }

        return stk.pop();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int scoreOfParentheses(string s) {

        stack<int> stk;

        // Score of the whole string so far.
        stk.push(0);

        for (char ch : s) {

            if (ch == '(') {
                // New inner context starts at 0.
                stk.push(0);
            } else {

                int inner = stk.top();
                stk.pop();

                int score;
                if (inner == 0) {
                    score = 1;          // ()
                } else {
                    score = 2 * inner;  // (A)
                }

                // AB → A + B. Add this piece to the parent context.
                int parent = stk.top();
                stk.pop();
                stk.push(parent + score);
            }
        }

        return stk.top();
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    each character is processed once
Space : O(n)    one stack frame per unmatched '('
```

The depth of the stack is the nesting depth of `s`.

---

# Pattern Recognition

```text
Score a valid parentheses string
        ↓
() is 1, (A) is 2A, AB is A + B
        ↓
Each '(' opens a new score bucket
        ↓
')' closes that bucket: 1 or 2 * inner
        ↓
Add the result to the parent bucket
```

### Remember this trigger:

> **Parentheses with a recursive score → stack of running totals. Empty pair is 1, nested pair doubles, siblings add.**
