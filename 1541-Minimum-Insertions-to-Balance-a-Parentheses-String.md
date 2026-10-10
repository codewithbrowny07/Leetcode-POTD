# LeetCode 1541 — Minimum Insertions to Balance a Parentheses String

## Problem

`s` contains only `'('` and `')'`.

In this problem a balanced string needs **two** closes for every open:

```text
()        is not enough
())       is one balanced piece
A + B     two balanced pieces side by side
( A ))    a balanced piece inside one open and two closes
```

Return the **minimum number of insertions** that make `s` balanced. Insertions can be `'('` or `')'`.

Example:

```text
s      = "(()))"
answer = 1          one more ')'  →  "((()))"

s      = "())"
answer = 0          already balanced

s      = "))())("
answer = 3
```

---

# Approach — One Pass with an Open Count

### Core idea

Every `'('` is a debt of **two** `')'`.

`count` is how many `'('` are still unpaid. `result` is how many characters we have inserted.

A `')'` always starts a close-pair. Two decisions:

```text
1. Is there an open to attach this pair to?
   count > 0  → use one '('
   count == 0 → insert a '('

2. Do we already have both closes?
   next char is ')'  → this is "))", consume both
   otherwise         → we only have one ')', insert the other
```

After the scan, every leftover `'('` still needs two `')'`:

```text
answer = result + count * 2
```

### Walkthrough

```text
s = "(()))"

i = 0  '('   count = 1
i = 1  '('   count = 2
i = 2  ')'   use one '(', count = 1
             next is ')', so "))" is already here, i = 4
i = 4  ')'   use one '(', count = 0
             no second ')', insert one ')'
             result = 1

answer = 1 + 0 = 1
```

```text
s = "))())("

i = 0  ')'   no '(', insert '('          result = 1
             next is ')', consume "))"
i = 2  '('   count = 1
i = 3  ')'   use that '(', count = 0
             next is ')', consume "))"
i = 5  '('   count = 1

answer = 1 + 1 * 2 = 3
```

The first `"))"` needed an inserted `'('`. The last `'('` needs an inserted `"))"`. The middle `"())"` was already a complete piece.

### State

```text
count   : unmatched '('
result  : insertions already required
i       : current index
```

### Transitions

```text
'('                  → count++, i++
')' and count > 0    → count--
')' and count == 0   → result++                 insert '('
next char is ')'     → i += 2                   a full "))"
otherwise            → result++, i++            insert the missing ')'
end                  → result + count * 2
```

---

## Java

```java
class Solution {
    public int minInsertions(String s) {

        int n = s.length();
        int result = 0;
        int count = 0;
        int i = 0;

        while (i < n) {

            if (s.charAt(i) == '(') {
                count++;
                i++;
            } else {

                // This close-pair needs an open.
                if (count > 0) {
                    count--;
                } else {
                    result++;   // insert '('
                }

                // A close-pair is two ')' characters.
                if (i + 1 < n && s.charAt(i + 1) == ')') {
                    i += 2;     // "))" is already in the string
                } else {
                    result++;   // insert the missing ')'
                    i++;
                }
            }
        }

        // Each leftover '(' needs two ')'.
        return result + count * 2;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minInsertions(string s) {

        int n = s.size();
        int result = 0;
        int count = 0;
        int i = 0;

        while (i < n) {

            if (s[i] == '(') {
                count++;
                i++;
            } else {

                // This close-pair needs an open.
                if (count > 0) {
                    count--;
                } else {
                    result++;   // insert '('
                }

                // A close-pair is two ')' characters.
                if (i + 1 < n && s[i + 1] == ')') {
                    i += 2;     // "))" is already in the string
                } else {
                    result++;   // insert the missing ')'
                    i++;
                }
            }
        }

        // Each leftover '(' needs two ')'.
        return result + count * 2;
    }
};
```

---

# Complexity

Let `n = s.length()`.

```text
Time  : O(n)    i only moves forward
Space : O(1)    two counters, no stack and no output string
```

---

# Pattern Recognition

```text
Each '(' must be followed by "))"
        ↓
One open-count is the unpaid debt
        ↓
A ')' starts a pair: need an open, and need a second ')'
        ↓
Missing open → insert '('
Only one ')' present → insert ')'
        ↓
Leftover opens each cost two insertions
```

### Remember this trigger:

> **Balance where one open needs two closes → do not use a normal +1/−1 stack. On every `')'`, settle one open and check whether the next character completes `"))"`.**
