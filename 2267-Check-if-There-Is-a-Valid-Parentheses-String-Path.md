# LeetCode 2267 — Check if There Is a Valid Parentheses String Path

## Problem

You are given an `m x n` grid. Every cell is `'('` or `')'`.

A path starts at `(0, 0)` and ends at `(m - 1, n - 1)`. From a cell you may only move **right** or **down**.

Return `true` if some path spells a **valid parentheses string**.

A string is valid when:

```text
balance never goes negative
balance ends at 0
```

`'('` means `balance + 1`. `')'` means `balance - 1`.

Example:

```text
grid = [["(","(","("],
        [")","(",")"],
        ["(","(",")"],
        [")",")",")"]]

answer = true
```

One valid path is `()()` along a route whose opens and closes stay balanced.

---

# Shared prune

Every path has the same length:

```text
path length = m + n - 1
```

A valid parentheses string has **even** length. If `m + n - 1` is odd, return `false` immediately.

---

# Approach 1 — Top-Down DFS + Memoization

### Core idea

State is the cell you are on and the open-count **after** reading that cell:

```text
dfs(i, j, balance)
```

`balance` arrives as the count from the path above or to the left. This cell updates it, then we try both moves.

### Transitions

```text
out of grid          → false
')' makes balance < 0 → false          (invalid prefix)
(i, j) is the end    → balance == 0
otherwise            → down OR right
```

Memo key is `(i, j, balance)` **after** this cell is applied. Reaching the same cell with the same open-count always has the same answer, so we store it.

`balance` never exceeds the path length, so the third dimension is `m + n`.

---

## Java

```java
class Solution {

    Boolean[][][] dp;
    int m, n;

    public boolean hasValidPath(char[][] grid) {

        m = grid.length;
        n = grid[0].length;

        // Valid parentheses need an even-length path.
        if ((m + n - 1) % 2 != 0) {
            return false;
        }

        dp = new Boolean[m][n][m + n];

        return dfs(grid, 0, 0, 0);
    }

    private boolean dfs(char[][] grid, int i, int j, int balance) {

        if (i >= m || j >= n) {
            return false;
        }

        if (grid[i][j] == '(') {
            balance++;
        } else {
            balance--;
        }

        // Too many closes. This prefix can never be valid.
        if (balance < 0) {
            return false;
        }

        if (i == m - 1 && j == n - 1) {
            return balance == 0;
        }

        if (dp[i][j][balance] != null) {
            return dp[i][j][balance];
        }

        boolean down = dfs(grid, i + 1, j, balance);
        boolean right = dfs(grid, i, j + 1, balance);

        return dp[i][j][balance] = down || right;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    bool hasValidPath(vector<vector<char>>& grid) {

        m = grid.size();
        n = grid[0].size();

        // Valid parentheses need an even-length path.
        if ((m + n - 1) % 2 != 0) {
            return false;
        }

        // -1 = not computed, 0 = false, 1 = true.
        dp.assign(m, vector<vector<int>>(n, vector<int>(m + n, -1)));

        return dfs(grid, 0, 0, 0);
    }

private:

    int m, n;
    vector<vector<vector<int>>> dp;

    bool dfs(vector<vector<char>>& grid, int i, int j, int balance) {

        if (i >= m || j >= n) {
            return false;
        }

        if (grid[i][j] == '(') {
            balance++;
        } else {
            balance--;
        }

        // Too many closes. This prefix can never be valid.
        if (balance < 0) {
            return false;
        }

        if (i == m - 1 && j == n - 1) {
            return balance == 0;
        }

        if (dp[i][j][balance] != -1) {
            return dp[i][j][balance];
        }

        bool down = dfs(grid, i + 1, j, balance);
        bool right = dfs(grid, i, j + 1, balance);

        dp[i][j][balance] = down || right;

        return dp[i][j][balance];
    }
};
```

### Complexity

```text
Time  : O(m · n · (m + n))
Space : O(m · n · (m + n))   memo table, plus O(m + n) recursion stack
```

Each state is computed once and branches to two neighbors.

---

# Approach 2 — Bottom-Up DP

Same state, built cell by cell so there is no recursion.

```text
reach[j][b] = true
```

means: on the current row, column `j` can be reached with balance `b` **after** reading that cell.

Only two previous places can enter `(i, j)`:

```text
from above  (i - 1, j)   previous row, same column
from left   (i, j - 1)   same row, previous column
```

Apply this cell’s delta (`+1` or `-1`) to every balance those neighbors already have. Drop any result that is negative.

Because we walk left to right, the left neighbor is already filled in the **current** row. The cell above is still in the **previous** row, so keep two rows and swap them.

The answer is whether the bottom-right cell can be reached with balance `0`.

---

## Java

```java
class Solution {

    public boolean hasValidPath(char[][] grid) {

        int m = grid.length;
        int n = grid[0].length;

        if ((m + n - 1) % 2 != 0) {
            return false;
        }

        int limit = m + n;

        // dp[col][balance] for the previous row.
        boolean[][] dp = new boolean[n][limit];

        for (int i = 0; i < m; i++) {

            boolean[][] next = new boolean[n][limit];

            for (int j = 0; j < n; j++) {

                int delta = grid[i][j] == '(' ? 1 : -1;

                if (i == 0 && j == 0) {
                    if (delta >= 0) {
                        next[j][delta] = true;
                    }
                    continue;
                }

                // Came from above.
                if (i > 0) {
                    for (int b = 0; b < limit; b++) {
                        if (!dp[j][b]) {
                            continue;
                        }
                        int nb = b + delta;
                        if (nb >= 0) {
                            next[j][nb] = true;
                        }
                    }
                }

                // Came from the left. That cell is already in this row.
                if (j > 0) {
                    for (int b = 0; b < limit; b++) {
                        if (!next[j - 1][b]) {
                            continue;
                        }
                        int nb = b + delta;
                        if (nb >= 0) {
                            next[j][nb] = true;
                        }
                    }
                }
            }

            dp = next;
        }

        return dp[n - 1][0];
    }
}
```

---

## C++

```cpp
class Solution {
public:

    bool hasValidPath(vector<vector<char>>& grid) {

        int m = grid.size();
        int n = grid[0].size();

        if ((m + n - 1) % 2 != 0) {
            return false;
        }

        int limit = m + n;

        // dp[col][balance] for the previous row.
        vector<vector<char>> dp(n, vector<char>(limit, 0));

        for (int i = 0; i < m; i++) {

            vector<vector<char>> next(n, vector<char>(limit, 0));

            for (int j = 0; j < n; j++) {

                int delta = grid[i][j] == '(' ? 1 : -1;

                if (i == 0 && j == 0) {
                    if (delta >= 0) {
                        next[j][delta] = 1;
                    }
                    continue;
                }

                // Came from above.
                if (i > 0) {
                    for (int b = 0; b < limit; b++) {
                        if (!dp[j][b]) {
                            continue;
                        }
                        int nb = b + delta;
                        if (nb >= 0) {
                            next[j][nb] = 1;
                        }
                    }
                }

                // Came from the left. That cell is already in this row.
                if (j > 0) {
                    for (int b = 0; b < limit; b++) {
                        if (!next[j - 1][b]) {
                            continue;
                        }
                        int nb = b + delta;
                        if (nb >= 0) {
                            next[j][nb] = 1;
                        }
                    }
                }
            }

            dp.swap(next);
        }

        return dp[n - 1][0];
    }
};
```

### Complexity

```text
Time  : O(m · n · (m + n))
Space : O(n · (m + n))     two rolling rows, not the full grid
```

Same number of states as the DFS. The rolling row drops the extra `m` factor from the table.

---

# Pattern Recognition

```text
Grid path + parentheses balance
        ↓
Only right and down, so every path has length m + n - 1
        ↓
Odd length is impossible
        ↓
State = (row, col, open count)
        ↓
Never let the count go negative
        ↓
End cell must finish at 0
```

### Remember this trigger:

> **Path that must be a valid parentheses string → track the open-count in the DP state, and reject a negative prefix immediately.**
