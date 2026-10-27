# LeetCode 3963 — Create Grid With Exactly One Path

## Problem

You are given two integers `m` and `n`.

Build any `m x n` grid using only:

```text
'.'  → free cell
'#'  → obstacle
```

A valid path:

```text
starts at (0, 0)
ends at (m - 1, n - 1)
moves only right or down
uses only free cells
```

Return any grid that has **exactly one** such path.

Example:

```text
m = 3, n = 4

....
###.
###.

only path:
(0,0) → (0,1) → (0,2) → (0,3) → (1,3) → (2,3)
```

```text
1 <= m, n <= 25
```

---

# Approach — Open the Top Row and the Right Column

### Core idea

Right and down moves cannot go back. If every other route is walled off, one polyline is the only path.

Build this shape:

```text
row 0        : every cell is '.'
rows 1..m-1  : '#' everywhere except the last column, which is '.'
```

```text
. . . . .
# # # # .
# # # # .
```

From `(0, 0)` the cell below is `'#'` whenever another row exists. The same is true for every cell in row `0` except the last column. So the path must travel right across the whole first row:

```text
(0, 0) → (0, 1) → ... → (0, n - 1)
```

At `(0, n - 1)` the only useful move is down. Every later row has a free cell only in column `n - 1`, so the path continues straight down to `(m - 1, n - 1)`.

There is no second path. Leaving row `0` early lands on `'#'`, and there is no way to move left or up to recover.

### Small grids

```text
m = 1, n = 4
....
```

One row of free cells. The only rightward walk is the path.

```text
m = 3, n = 1
.
.
.
```

One column of free cells. The only downward walk is the path. The loop that writes `'#'` does not run, because `n - 1 == 0`.

### State

```text
temp  : first row, n copies of '.'
strb  : every later row, '#' in columns 0..n-2 and '.' in the last column
```

### Transitions

```text
fill a string of n dots
save that string as row 0
turn the first n - 1 characters into '#'
copy that string into every remaining row
```

---

## Java

```java
class Solution {
    public String[] createGrid(int m, int n) {

        String[] res = new String[m];
        StringBuilder strb = new StringBuilder();

        for (int i = 0; i < n; i++) {
            strb.append('.');
        }

        // Row 0 stays completely open.
        String temp = strb.toString();

        // Later rows are blocked except the last column.
        for (int i = 0; i < n - 1; i++) {
            strb.setCharAt(i, '#');
        }

        res[0] = temp;

        for (int i = 1; i < m; i++) {
            res[i] = strb.toString();
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

    vector<string> createGrid(int m, int n) {

        vector<string> res(m);

        // Row 0 stays completely open.
        string open(n, '.');

        // Later rows are blocked except the last column.
        string blocked(n, '#');
        blocked[n - 1] = '.';

        res[0] = open;

        for (int i = 1; i < m; i++) {
            res[i] = blocked;
        }

        return res;
    }
};
```

---

# Complexity

```text
Time  : O(m · n)    every cell of the returned grid is written
Space : O(m · n)    the grid itself
```

`m` and `n` are at most `25`, so the construction is tiny. The answer string array is the space that grows with the grid. Besides that array, only one or two length-`n` builders are used.

---

# Pattern Recognition

```text
Construct a grid with exactly one right/down path
        ↓
Do not search for a path
        ↓
Force the walk onto one polyline
        ↓
Open the first row
        ↓
Open only the last column below it
```

### Remember this trigger:

> **Exactly one monotone path → wall off every early down move. Across the top, then down the last column, is one path and cannot branch.**
