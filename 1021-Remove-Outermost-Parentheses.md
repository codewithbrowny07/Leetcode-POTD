# LeetCode 1021 — Remove Outermost Parentheses

## Problem

`s` is a valid parentheses string. It splits into **primitive** pieces: valid strings that cannot be split further into two non-empty valid strings.

Each primitive piece has the shape `(A)`. Remove the outer `'('` and `')'` of every piece, then concatenate what is left.

Example:

```text
s      = "(()())(())"
pieces = "(()())" + "(())"
answer = "()()" + "()" = "()()()"

s      = "()()"
answer = ""
```

---

# Approach — Depth Counter

### Core idea

A primitive piece starts when the depth goes from `0` to `1`, and ends when it comes back to `0`.

Those two parentheses are the outer pair. Everything written while the depth is already above `0` stays.

```text
'('  → if depth is already > 0, this open is inside, so keep it
       then depth++

')'  → depth-- first
       if depth is still > 0, this close is inside, so keep it
```

The `'('` that moves depth from `0` to `1` is skipped. The `')'` that moves depth from `1` to `0` is skipped, because the check happens after the decrement.

No stack is needed. The depth alone knows when a primitive piece starts and ends.

### Walkthrough

```text
s = "(()())(())"

char   (  (  )  (  )  )  (  (  )  )
depth  1  2  1  2  1  0  1  2  1  0
keep      (  )  (  )        (  )
```

The two chars where depth lands on `0`, and the two opens that left `0`, are the outer pairs.

### State

```text
depth  : how many '(' are still open
strb   : parentheses that are not an outer pair
```

### Transitions

```text
'(' and depth > 0  → append, then depth++
'(' and depth == 0 → skip, then depth++
')'                → depth--
')' and depth > 0  → append
')' and depth == 0 → skip     end of this primitive piece
```

---

## Java

```java
class Solution {
    public String removeOuterParentheses(String s) {

        StringBuilder strb = new StringBuilder();
        int depth = 0;

        for (char ch : s.toCharArray()) {

            if (ch == '(') {

                // depth > 0 means this '(' is inside a primitive piece.
                if (depth > 0) {
                    strb.append(ch);
                }

                depth++;
            } else {

                depth--;

                // depth > 0 means this ')' is not the closer of the piece.
                if (depth > 0) {
                    strb.append(ch);
                }
            }
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

    string removeOuterParentheses(string s) {

        string result;
        int depth = 0;

        for (char ch : s) {

            if (ch == '(') {

                // depth > 0 means this '(' is inside a primitive piece.
                if (depth > 0) {
                    result.push_back(ch);
                }

                depth++;
            } else {

                depth--;

                // depth > 0 means this ')' is not the closer of the piece.
                if (depth > 0) {
                    result.push_back(ch);
                }
            }
        }

        return result;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    one pass
Space : O(1)    extra memory besides the answer string
```

---

# Pattern Recognition

```text
Valid parentheses split into primitive pieces
        ↓
A piece starts at depth 0 and ends when depth returns to 0
        ↓
Skip the parenthesis that crosses 0
        ↓
Keep everything written while depth stays positive
```

### Remember this trigger:

> **Strip the outer pair of each primitive group → one depth counter. Append `'('` before incrementing only if depth is already positive, and append `')'` after decrementing only if depth is still positive.**
