# LeetCode 1111 — Maximum Nesting Depth of Two Valid Parentheses Strings

## Problem

`seq` is a valid parentheses string.

Split it into two **disjoint subsequences** `A` and `B` that together use every character. Both `A` and `B` must also be valid parentheses strings. One of them may be empty.

Return an array `answer` of the same length as `seq`:

```text
answer[i] = 0  →  seq[i] goes to A
answer[i] = 1  →  seq[i] goes to B
```

Any split that **minimizes** `max(depth(A), depth(B))` is accepted.

Example:

```text
seq    = "(()())"
answer = [0, 1, 1, 1, 1, 0]
```

`A` is `()` at depth 1. `B` is `()()` at depth 1. The max of the two depths is 1.

---

# Approach — Depth Parity

### Core idea

Nesting depth is what we are trying to cut in half.

Give **odd** depths to one string and **even** depths to the other. A parenthesis and the parenthesis that closes it must get the **same** group, otherwise that string is not valid.

```text
'('  → this pair lives at the new depth, so increment first, then assign depth % 2
')'  → this pair still lives at the current depth, so assign depth % 2, then decrement
```

Consecutive levels go to different groups, so neither string is deeper than about half the original depth. That is optimal: two groups cannot both stay shallower than `ceil(depth / 2)`.

### Walkthrough

```text
seq = ( ( ) ( ) )

i        0  1  2  3  4  5
char     (  (  )  (  )  )
depth    1  2  2  2  2  1     depth used for the assignment
answer   1  0  0  0  0  1     depth % 2
```

Group `0` receives the inner `()()`. Group `1` receives the outer `()`.

### State

```text
d    : current nesting depth
res  : group id for each index
```

### Transitions

```text
'('  → d++, res[i] = d % 2
')'  → res[i] = d % 2, d--
```

---

## Java

```java
class Solution {
    public int[] maxDepthAfterSplit(String seq) {
        int n = seq.length();
        int[] res = new int[n];
        int d = 0;

        for (int i = 0; i < n; i++) {
            if (seq.charAt(i) == '(') {
                // This '(' opens a new depth. Assign that depth's group.
                d++;
                res[i] = d % 2;
            } else {
                // This ')' closes the current depth, so it shares that group.
                res[i] = d % 2;
                d--;
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

    vector<int> maxDepthAfterSplit(string seq) {

        int n = seq.size();
        vector<int> res(n);
        int d = 0;

        for (int i = 0; i < n; i++) {

            if (seq[i] == '(') {
                // This '(' opens a new depth. Assign that depth's group.
                d++;
                res[i] = d % 2;
            } else {
                // This ')' closes the current depth, so it shares that group.
                res[i] = d % 2;
                d--;
            }
        }

        return res;
    }
};
```

---

# Complexity

Let `n = seq.length()`.

```text
Time  : O(n)    one pass
Space : O(1)    extra memory besides the answer array
```

The answer array is required output, so it is not counted as extra space.

---

# Pattern Recognition

```text
Split one valid parentheses string into two valid ones
        ↓
A pair must stay in the same group
        ↓
Assign by nesting depth % 2
        ↓
Odd levels → one string, even levels → the other
        ↓
max depth becomes ceil(original depth / 2)
```

### Remember this trigger:

> **Minimize the deeper of two parentheses groups → color depths by parity. Increment before assigning `'('`, decrement after assigning `')'`.**
