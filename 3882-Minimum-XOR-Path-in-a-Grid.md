# LeetCode 3882 — Minimum XOR Path in a Grid

## Problem

You start at the top-left cell `(0, 0)` and want to reach the bottom-right cell `(m - 1, n - 1)`.

You may move only:

```text
right
down
```

The cost of a path is the XOR of every cell value visited, including both endpoints.

Return the minimum possible path XOR.

Example:

```text
grid =
[
  [1, 2],
  [4, 3]
]

paths:
1 → 2 → 3   = 1 ^ 2 ^ 3 = 0
1 → 4 → 3   = 1 ^ 4 ^ 3 = 6

answer = 0
```

```text
1 <= m, n <= 1000
m * n <= 1000
0 <= grid[i][j] <= 1023
```

---

# Approach — DFS + Memoization on `(row, col, prefixXor)`

### Why normal grid DP is not enough

For ordinary path sums, keeping only the smallest cost at each cell often works.

XOR is different. A smaller prefix XOR is not always better after the same suffix is added:

```text
1 < 2

1 ^ 3 = 2
2 ^ 3 = 1
```

The prefix `2` was larger, but becomes better after XOR with `3`.

So a state cannot be only:

```text
(row, col)
```

It must also remember the XOR accumulated before entering this cell:

```text
func(row, col, curr)
```

### Why 1024 XOR states?

Every grid value is at most `1023`, which fits in 10 bits:

```text
0 <= grid[i][j] < 2^10
```

XOR never creates a bit that is not present in one of its operands. Therefore every path XOR also stays in:

```text
0 .. 1023
```

That gives exactly 1024 possible XOR states per cell:

```text
dp[m][n][1024]
```

### Recurrence

The function receives `curr`, the XOR **before** including `grid[i][j]`.

First include the current cell:

```text
curr ^= grid[i][j]
```

If this is the bottom-right cell, `curr` is the XOR of the full path.

Otherwise try both valid next moves:

```text
answer = min(
    func(i + 1, j, curr),
    func(i, j + 1, curr)
)
```

The memo key uses the XOR **after** the current cell is included, because that is the exact state passed to the next move.

### Walkthrough

```text
grid =
[
  [1, 2],
  [4, 3]
]

func(0, 0, 0)
  curr = 0 ^ 1 = 1

  right:
    func(0, 1, 1)
    curr = 1 ^ 2 = 3
    func(1, 1, 3)
    curr = 3 ^ 3 = 0     → path XOR 0

  down:
    func(1, 0, 1)
    curr = 1 ^ 4 = 5
    func(1, 1, 5)
    curr = 5 ^ 3 = 6     → path XOR 6

min(0, 6) = 0
```

### State

```text
i, j  : current cell
curr  : XOR after all earlier cells, before this one
dp    : best final XOR after processing this cell with a given XOR state
```

### Transitions

```text
curr ^= grid[i][j]

bottom-right cell  → return curr
right exists       → try right
down exists        → try down
return minimum of those valid moves
```

---

## Java

```java
class Solution {
    private int[][][] dp;
    private int m, n;

    public int minCost(int[][] grid) {

        m = grid.length;
        n = grid[0].length;

        // Every prefix XOR is between 0 and 1023.
        dp = new int[m][n][1024];

        for (int[][] layer : dp) {
            for (int[] row : layer) {
                Arrays.fill(row, -1);
            }
        }

        return func(grid, 0, 0, 0);
    }

    private int func(int[][] grid, int i, int j, int curr) {

        // Include the cell we just entered.
        curr ^= grid[i][j];

        if (i == m - 1 && j == n - 1) {
            return curr;
        }

        // curr now represents the XOR after processing grid[i][j].
        if (dp[i][j][curr] != -1) {
            return dp[i][j][curr];
        }

        int right = Integer.MAX_VALUE;
        int down = Integer.MAX_VALUE;

        if (j + 1 < n) {
            right = func(grid, i, j + 1, curr);
        }

        if (i + 1 < m) {
            down = func(grid, i + 1, j, curr);
        }

        return dp[i][j][curr] = Math.min(right, down);
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int minCost(vector<vector<int>>& grid) {

        m = grid.size();
        n = grid[0].size();

        // Every prefix XOR is between 0 and 1023.
        dp.assign(m, vector<vector<int>>(n, vector<int>(1024, -1)));

        return func(grid, 0, 0, 0);
    }

private:

    int m, n;
    vector<vector<vector<int>>> dp;

    int func(vector<vector<int>>& grid, int i, int j, int curr) {

        // Include the cell we just entered.
        curr ^= grid[i][j];

        if (i == m - 1 && j == n - 1) {
            return curr;
        }

        // curr now represents the XOR after processing grid[i][j].
        if (dp[i][j][curr] != -1) {
            return dp[i][j][curr];
        }

        int right = INT_MAX;
        int down = INT_MAX;

        if (j + 1 < n) {
            right = func(grid, i, j + 1, curr);
        }

        if (i + 1 < m) {
            down = func(grid, i + 1, j, curr);
        }

        return dp[i][j][curr] = min(right, down);
    }
};
```

---

# Complexity

Let `V = 1024`, the number of possible XOR values.

```text
Time  : O(m · n · V)
Space : O(m · n · V)
```

Each `(row, col, xor)` state is computed at most once. The recursion stack is `O(m + n)`, which is smaller than the memo table.

Since `m * n <= 1000`, the memo table has at most about one million entries.

---

# Pattern Recognition

```text
Grid path with XOR cost
        ↓
XOR does not preserve numeric ordering
        ↓
The smallest prefix XOR is not enough information
        ↓
Keep every possible prefix XOR at each cell
        ↓
Values fit in 10 bits, so use 1024 states
```

### Remember this trigger:

> **Path XOR optimization → add the running XOR to the DP state. A numerically smaller prefix can become worse after the next XOR.**
