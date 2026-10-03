# LeetCode 32 — Longest Valid Parentheses

## Problem

Given a string `s` of `'('` and `')'`, return the length of the **longest valid parentheses substring**.

A valid substring is well-formed: every open has a matching close, in order, and the balance never goes negative inside it.

Example:

```text
s      = ")()())"
answer = 4
```

The longest piece is `"()()"`, from index `1` through `4`.

```text
s      = "(()"
answer = 2
```

---

# Approach — Stack of Indices

### Core idea

Store **indices**, not the brackets themselves.

The stack top is the start of the current open segment. A sentinel index `-1` sits under everything, so the first valid pair `"()"` at indices `0` and `1` has length:

```text
1 - (-1) = 2
```

Scan left to right:

```text
'('  → push this index. It might start a new pair.
')'  → pop. This close matches the open on top.
```

After the pop, two things can happen:

```text
stack is empty
    This ')' had no matching '('.
    It ends every valid substring that reached it.
    Push this index. It is the new base.

stack is not empty
    The index under the top is the last boundary
    before the valid substring that just ended at i.
    length = i - stack.top()
```

That length can cover several pairs in a row, because earlier opens stay on the stack until their own closes arrive. `"()()"` keeps the base `-1`, so the second `')'` measures all the way back and gets length `4`.

### Walkthrough

```text
s = ")()())"
     012345

start stack: [-1]

i = 0  ')'  pop -1, stack empty, push 0     stack: [0]
i = 1  '('  push 1                           stack: [0, 1]
i = 2  ')'  pop 1, length = 2 - 0 = 2       stack: [0]     max = 2
i = 3  '('  push 3                           stack: [0, 3]
i = 4  ')'  pop 3, length = 4 - 0 = 4       stack: [0]     max = 4
i = 5  ')'  pop 0, stack empty, push 5      stack: [5]
```

Index `0` and index `5` are unmatched closes. They become walls. The valid run between them has length `4`.

### State

```text
stk  : indices of unmatched '(' , plus the last unmatched ')' as a base
max  : best valid length seen
```

### Transitions

```text
'('                  → push i
')' and stack empty  → push i          (new base)
')' otherwise        → length = i - peek, update max
```

---

## Java

```java
class Solution {
    public int longestValidParentheses(String s) {

        int n = s.length();
        int max = 0;

        Stack<Integer> stk = new Stack<>();

        // Base index before the string, so the first pair has length 2.
        stk.push(-1);

        for (int i = 0; i < n; i++) {

            char ch = s.charAt(i);

            if (ch == '(') {
                stk.push(i);
            } else {

                // Match this ')' with the latest '('.
                stk.pop();

                if (stk.isEmpty()) {
                    // Unmatched ')'. It becomes the new boundary.
                    stk.push(i);
                } else {
                    int len = i - stk.peek();
                    max = Math.max(max, len);
                }
            }
        }

        return max;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int longestValidParentheses(string s) {

        int n = s.size();
        int maxLen = 0;

        stack<int> stk;

        // Base index before the string, so the first pair has length 2.
        stk.push(-1);

        for (int i = 0; i < n; i++) {

            if (s[i] == '(') {
                stk.push(i);
            } else {

                // Match this ')' with the latest '('.
                stk.pop();

                if (stk.empty()) {
                    // Unmatched ')'. It becomes the new boundary.
                    stk.push(i);
                } else {
                    int len = i - stk.top();
                    maxLen = max(maxLen, len);
                }
            }
        }

        return maxLen;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    each index is pushed once and popped at most once
Space : O(n)    the stack holds unmatched '(' indices
```

---

# Pattern Recognition

```text
Longest valid parentheses substring
        ↓
An unmatched ')' splits the string into independent pieces
        ↓
Stack of indices, with -1 as the first base
        ↓
'(' pushes its index
')' pops, then length = i - new top
        ↓
If the pop empties the stack, that ')' is the next base
```

### Remember this trigger:

> **Valid parentheses substring, not just a yes/no check → store indices. The top after a pop is the wall before the current valid run.**
