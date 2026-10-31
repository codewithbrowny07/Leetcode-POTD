# LeetCode 3665 — Twisted Mirror Path Count

## Problem

You are given a binary grid:

```text
0 → empty cell
1 → mirror
```

Start at `(0, 0)` and reach `(m - 1, n - 1)` by moving only right or down.

When a move would enter a mirror:

```text
moving right into a mirror  → turn down and move below that mirror
moving down into a mirror   → turn right and move right of that mirror
```

If the redirected move enters another mirror, it reflects again immediately. A path that reflects outside the grid is invalid.

Return the number of unique valid paths modulo `10^9 + 7`.

Example:

```text
grid =
[
  [0, 1, 0],
  [0, 0, 1],
  [1, 0, 0]
]

answer = 5
```

```text
2 <= m, n <= 500
grid[0][0] == grid[m - 1][n - 1] == 0
```

---

# Approach — DFS + Memoization, Simulate the Mirror Chain

### Core idea

`func(i, j)` counts valid paths from an **empty** cell `(i, j)` to the destination.

From an empty cell there are two choices:

```text
try to move right
try to move down
```

Each attempted move may enter one or more mirrors. Simulate that chain until it:

```text
reaches an empty cell  → recurse from there
goes outside the grid  → this direction contributes 0
```

Once the robot reaches an empty cell, its previous direction no longer matters. It may choose right or down again. That is why the memo state only needs:

```text
dp[row][col]
```

and does not need a direction dimension.

### Simulating a move

To explore right from `(i, j)`:

```text
x = i
y = j + 1
dir = right
```

While `(x, y)` is a mirror:

```text
entered moving right  → leave below it:  x++, direction becomes down
entered moving down   → leave right of it: y++, direction becomes right
```

The loop stops at an empty cell or when it leaves the grid.

The down move is symmetric:

```text
x = i + 1
y = j
dir = down
```

### Walkthrough

```text
grid =
[
  [0, 1, 1],
  [1, 1, 0]
]

start at (0, 0)

try RIGHT:
  enter (0, 1), a mirror, while moving right
  turn down → move to (1, 1), also a mirror
  entered while moving down, turn right → move to (1, 2)
  (1, 2) is empty and is the destination

try DOWN:
  enter (1, 0), a mirror, while moving down
  turn right → move to (1, 1), a mirror
  turn down → move to (2, 1), outside

answer = 1
```

### State

```text
dp[i][j]  : number of valid paths from empty cell (i, j) to the end
dir       : direction used only while resolving a mirror chain
```

### Transitions

```text
at destination          → 1
right chain reaches empty cell → func(empty cell)
down chain reaches empty cell  → func(empty cell)
chain leaves the grid   → 0
answer                  → rightWays + downWays
```

---

## Java

```java
class Solution {
    private static final long MOD = 1_000_000_007L;
    private int m, n;
    private long[][] dp;

    public int uniquePaths(int[][] grid) {

        m = grid.length;
        n = grid[0].length;

        dp = new long[m][n];

        for (int i = 0; i < m; i++) {
            Arrays.fill(dp[i], -1);
        }

        return (int) func(grid, 0, 0);
    }

    public long func(int[][] grid, int i, int j) {

        if (i >= m || j >= n) {
            return 0;
        }

        if (i == m - 1 && j == n - 1) {
            return 1;
        }

        if (dp[i][j] != -1) {
            return dp[i][j];
        }

        long countPaths = 0;

        // Explore a move to the right.
        int x = i;
        int y = j + 1;
        int dir = 0; // 0 = right, 1 = down

        // Resolve every chained reflection.
        while (x < m && y < n && grid[x][y] == 1) {

            if (dir == 0) {
                x++;       // right into mirror → move below it
                dir = 1;
            } else {
                y++;       // down into mirror → move right of it
                dir = 0;
            }
        }

        if (x < m && y < n) {
            countPaths += func(grid, x, y);
        }

        // Explore a move downward.
        x = i + 1;
        y = j;
        dir = 1; // 0 = right, 1 = down

        // Resolve every chained reflection.
        while (x < m && y < n && grid[x][y] == 1) {

            if (dir == 0) {
                x++;
                dir = 1;
            } else {
                y++;
                dir = 0;
            }
        }

        if (x < m && y < n) {
            countPaths += func(grid, x, y);
        }

        return dp[i][j] = countPaths % MOD;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int uniquePaths(vector<vector<int>>& grid) {

        m = grid.size();
        n = grid[0].size();

        dp.assign(m, vector<long long>(n, -1));

        return (int) func(grid, 0, 0);
    }

private:

    static const long long MOD = 1000000007LL;
    int m, n;
    vector<vector<long long>> dp;

    long long func(vector<vector<int>>& grid, int i, int j) {

        if (i >= m || j >= n) {
            return 0;
        }

        if (i == m - 1 && j == n - 1) {
            return 1;
        }

        if (dp[i][j] != -1) {
            return dp[i][j];
        }

        long long countPaths = 0;

        // Explore a move to the right.
        int x = i;
        int y = j + 1;
        int dir = 0; // 0 = right, 1 = down

        // Resolve every chained reflection.
        while (x < m && y < n && grid[x][y] == 1) {

            if (dir == 0) {
                x++;       // right into mirror → move below it
                dir = 1;
            } else {
                y++;       // down into mirror → move right of it
                dir = 0;
            }
        }

        if (x < m && y < n) {
            countPaths += func(grid, x, y);
        }

        // Explore a move downward.
        x = i + 1;
        y = j;
        dir = 1; // 0 = right, 1 = down

        // Resolve every chained reflection.
        while (x < m && y < n && grid[x][y] == 1) {

            if (dir == 0) {
                x++;
                dir = 1;
            } else {
                y++;
                dir = 0;
            }
        }

        if (x < m && y < n) {
            countPaths += func(grid, x, y);
        }

        return dp[i][j] = countPaths % MOD;
    }
};
```

---

# Complexity

The memo table has `m · n` states, but each state may scan a chain of up to `m + n` mirrors.

```text
Time  : O(m · n · (m + n)) worst case
Space : O(m · n)
```

The space is the `dp` table plus an `O(m + n)` recursion stack.

This is the accurate bound for the supplied simulation approach. The code memoizes empty-cell states, but it does not memoize the mirror-chain destination for each `(cell, direction)` pair.

---

# Pattern Recognition

```text
Right/down grid paths with forced mirror turns
        ↓
At an empty cell, choose right or down
        ↓
One attempted move can pass through a chain of mirrors
        ↓
Simulate the chain until empty or outside
        ↓
Memoize the number of paths from each empty cell
```

### Remember this trigger:

> **Forced movement between decision cells → simulate until the next decision cell, then memoize from there. Here, an empty cell is a decision point and mirrors only redirect the current move.**
