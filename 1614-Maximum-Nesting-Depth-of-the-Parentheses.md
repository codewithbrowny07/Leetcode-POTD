# LeetCode 1614 — Maximum Nesting Depth of the Parentheses

## Problem

A string `s` is a **valid parentheses string** (VPS). It contains digits, the operators `+ - * /`, and parentheses.

Return the **maximum nesting depth** of the parentheses in `s`.

```text
depth("")           = 0
depth(single char)  = 0          a digit or an operator
depth(A + B)        = max(depth(A), depth(B))
depth("(" + A + ")") = 1 + depth(A)
```

The parentheses are guaranteed to be balanced. Other characters do not affect depth.

Example:

```text
s      = "(1+(2*3)+((8)/4))+1"
answer = 3
```

The `8` sits inside three pairs: `((8)/4)` inside `(1+... )`.

```text
( 1 + ( 2 * 3 ) + ( ( 8 ) / 4 ) ) + 1
1     2         1   2 3     2 1     0
                      ↑
                   depth 3
```

---

# Approach — Running Depth ⭐

### Core idea

A valid string never has a `)` without a matching `(`. A stack is not required.

Keep a counter:

```text
'('  → depth goes up by 1
')'  → depth goes down by 1
other characters are ignored
```

The answer is the **largest** value `depth` reaches. Record that maximum **after** an open parenthesis, while the depth is still high.

### Walkthrough

```text
s = "(1+(2*3)+((8)/4))+1"

(   depth 1   max 1
(   depth 2   max 2
)   depth 1
(   depth 2
(   depth 3   max 3
)   depth 2
)   depth 1
)   depth 0

answer = 3
```

### State

```text
depth : how many '(' are still open
res   : deepest value seen so far
```

### Transitions

```text
ch == '('  → depth++, res = max(res, depth)
ch == ')'  → depth--
otherwise  → skip
```

### One decrement only

Closing a pair drops the depth by **one**.

```text
if (ch == ')') {
    if (!stk.isEmpty() && stk.peek() == '(') {
        stk.pop();
        cnt--;
    }
    cnt--;   // second drop — depth falls by 2
}
```

That extra `cnt--` runs on every `)`, so `"(1+(2*3)+((8)/4))+1"` reports `2` instead of `3`. The matching `(` already accounts for the close. Pop (or just decrement) once.

The stack only repeats what `depth` already knows. Drop it and the scan is `O(1)` extra space.

---

## Java

```java
class Solution {
    public int maxDepth(String s) {
        int depth = 0;
        int res = 0;

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);

            if (ch == '(') {
                depth++;
                res = Math.max(res, depth);
            } else if (ch == ')') {
                depth--;
            }
        }

        return res;
    }
}
```

---

## C++

```cpp
class Solution {
public:
    int maxDepth(string s) {
        int depth = 0;
        int res = 0;

        for (char ch : s) {
            if (ch == '(') {
                depth++;
                res = max(res, depth);
            } else if (ch == ')') {
                depth--;
            }
        }

        return res;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)   one pass
Space : O(1)   two integers
```

`n ≤ 100` on this problem, so any linear scan is enough. The counter is the version to keep.

---

# Pattern Recognition

```text
Balanced parentheses, ask only for depth
        ↓
Do not store the pairs
        ↓
'(' increments, ')' decrements
        ↓
Answer is the max value of that counter
```

### Remember this trigger:

> **Valid brackets and you only need how deep they nest → one counter. A stack is for when you must remember what is inside the pair.**
