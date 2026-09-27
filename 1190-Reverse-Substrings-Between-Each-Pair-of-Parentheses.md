# LeetCode 1190 — Reverse Substrings Between Each Pair of Parentheses

## Problem

You are given a string `s` made of lowercase letters and parentheses.

Reverse the substring inside **every pair of matching parentheses**, starting from the **innermost** pair.

The final answer must **not** contain any parentheses.

Parentheses are guaranteed to be balanced.

Example:

```text
s      = "(u(love)i)"
answer = "iloveu"
```

Innermost `(love)` becomes `evol`, so the string is `(uevoli)`.
The outer pair then reverses that to `iloveu`.

---

# Approach — Stack

### Core idea

A stack naturally handles nested pairs: the **last** `(` you saw is the **innermost** one still open.

Scan left to right:

```text
letter or '('  → push it
')'            → pop until '(', that pop reverses the inside
                 drop the '('
                 push the reversed characters back
```

Pushing the reversed piece back is what makes the **outer** reverse work. The next `)` will pop that piece again and flip it once more.

At the end the stack holds the answer, but in reverse, because `pop` walks from the top. Pop everything into a builder, then reverse it.

### Walkthrough

```text
s = "(u(love)i)"

push ( u ( l o v e
hit ')'
  pop until '('  → "evol"     (stack pop is already a reverse)
  drop '('
  push e v o l
stack is now: ( u e v o l

push i
hit ')'
  pop until '('  → "iloveu"
  drop '('
  push i l o v e u

final pop + reverse → "iloveu"
```

### State

```text
st    : characters still waiting, including open '('
temp  : the reversed segment between the current ')' and its '('
```

### Transitions

```text
ch != ')'  → st.push(ch)
ch == ')'  → pop into temp until '('
             pop '('
             push temp back onto st
```

---

## Java

```java
class Solution {
    public String reverseParentheses(String s) {

        Stack<Character> st = new Stack<>();

        for (char ch : s.toCharArray()) {

            if (ch != ')') {
                st.push(ch);
            } else {

                // Popping into a builder reverses this pair.
                StringBuilder temp = new StringBuilder();

                while (st.peek() != '(') {
                    temp.append(st.pop());
                }

                st.pop(); // remove '('

                // Put the reversed piece back so an outer ')' can flip it again.
                for (char c : temp.toString().toCharArray()) {
                    st.push(c);
                }
            }
        }

        // Stack pops from the end, so reverse once more.
        StringBuilder ans = new StringBuilder();

        while (!st.isEmpty()) {
            ans.append(st.pop());
        }

        return ans.reverse().toString();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    string reverseParentheses(string s) {

        stack<char> st;

        for (char ch : s) {

            if (ch != ')') {
                st.push(ch);
            } else {

                // Popping into a string reverses this pair.
                string temp;

                while (st.top() != '(') {
                    temp.push_back(st.top());
                    st.pop();
                }

                st.pop(); // remove '('

                // Put the reversed piece back so an outer ')' can flip it again.
                for (char c : temp) {
                    st.push(c);
                }
            }
        }

        // Stack pops from the end, so reverse once more.
        string ans;

        while (!st.empty()) {
            ans.push_back(st.top());
            st.pop();
        }

        reverse(ans.begin(), ans.end());

        return ans;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n²)   a character can be popped and pushed once per enclosing pair
Space : O(n)    the stack holds the current string
```

`n ≤ 2000` on this problem, so the quadratic bound is fine.

---

# Pattern Recognition

```text
Nested parentheses, process innermost first
        ↓
Stack
        ↓
')' means: reverse everything back to the matching '('
        ↓
Push that reversed piece back for the next outer pair
```

### Remember this trigger:

> **Matching brackets + “do the inside first” → stack. Popping until the opener is already a reverse.**
