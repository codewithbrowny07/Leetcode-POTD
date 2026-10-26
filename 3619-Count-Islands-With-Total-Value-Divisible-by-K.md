# LeetCode 3619 — Count Islands With Total Value Divisible by K

## Problem

You are given an integer grid `grid` and an integer `k`.

```text
0      → water
positive value → land with that value
```

An island is a 4-directionally connected group of positive cells. Its total value is the sum of every cell in that island.

Return how many islands have a total value divisible by `k`.

Example:

```text
grid =
[
  [1, 0, 2],
  [3, 0, 0],
  [0, 4, 5]
]
k = 3

island 1: 1 + 3 = 4       not divisible by 3
island 2: 2               not divisible by 3
island 3: 4 + 5 = 9       divisible by 3

answer = 1
```

---

# Approach — DFS Flood Fill That Returns the Island Sum

### Core idea

When the outer scan finds a positive cell, it has found an unvisited island.

Run DFS from that cell:

```text
read the current cell value
set the cell to 0       → mark it visited
return its value plus the sums from all four neighbors
```

When DFS finishes, it has both:

```text
visited every cell in this island
returned the island's total value
```

The outer loop checks:

```text
sum % k == 0
```

and increments the answer when it is true.

Setting visited cells to `0` is safe because `0` means water. It prevents the scan from starting a second DFS on the same island.

### Walkthrough

```text
grid =
[
  [1, 0, 2],
  [3, 0, 0],
  [0, 4, 5]
]

start at (0, 0):
1 + dfs(1, 0) = 1 + 3 = 4
4 % 3 != 0

start at (0, 2):
sum = 2
2 % 3 != 0

start at (2, 1):
4 + dfs(2, 2) = 4 + 5 = 9
9 % 3 == 0
count++
```

### State

```text
grid[r][c] > 0  : unvisited land
grid[r][c] == 0 : water or land already visited
sum             : value accumulated by one DFS
```

### Transitions

```text
outside grid or cell is 0  → return 0
positive cell              → save its value
                             mark it 0
                             return value + sums from up/down/left/right
after one DFS              → count it if sum % k == 0
```

---

## Java

```java
class Solution {
    public int countIslands(int[][] grid, int k) {

        int m = grid.length;
        int n = grid[0].length;
        int count = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {

                if (grid[i][j] > 0) {

                    // This DFS visits the full island and returns its sum.
                    long sum = dfs(grid, i, j);

                    if (sum % k == 0) {
                        count++;
                    }
                }
            }
        }

        return count;
    }

    private long dfs(int[][] grid, int r, int c) {

        int m = grid.length;
        int n = grid[0].length;

        // Water, visited land, or outside the island adds nothing.
        if (r < 0 || r >= m || c < 0 || c >= n
                || grid[r][c] == 0) {
            return 0;
        }

        int value = grid[r][c];
        grid[r][c] = 0; // Mark visited.

        long sum = value;
        sum += dfs(grid, r - 1, c);
        sum += dfs(grid, r + 1, c);
        sum += dfs(grid, r, c - 1);
        sum += dfs(grid, r, c + 1);

        return sum;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int countIslands(vector<vector<int>>& grid, int k) {

        int m = grid.size();
        int n = grid[0].size();
        int count = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {

                if (grid[i][j] > 0) {

                    // This DFS visits the full island and returns its sum.
                    long long sum = dfs(grid, i, j);

                    if (sum % k == 0) {
                        count++;
                    }
                }
            }
        }

        return count;
    }

private:

    long long dfs(vector<vector<int>>& grid, int r, int c) {

        int m = grid.size();
        int n = grid[0].size();

        // Water, visited land, or outside the island adds nothing.
        if (r < 0 || r >= m || c < 0 || c >= n
                || grid[r][c] == 0) {
            return 0;
        }

        int value = grid[r][c];
        grid[r][c] = 0; // Mark visited.

        long long sum = value;
        sum += dfs(grid, r - 1, c);
        sum += dfs(grid, r + 1, c);
        sum += dfs(grid, r, c - 1);
        sum += dfs(grid, r, c + 1);

        return sum;
    }
};
```

`long` / `long long` is used for the island total so it remains safe even when an island contains many positive values.

---

# Complexity

Let `m` be the number of rows and `n` the number of columns.

```text
Time  : O(m · n)    every cell is visited at most once
Space : O(m · n)    worst-case DFS recursion stack
```

The input grid doubles as the visited structure, so no extra `visited` matrix is needed.

---

# Pattern Recognition

```text
Grid with connected positive-value cells
        ↓
Need one aggregate per island
        ↓
DFS returns that aggregate
        ↓
Mark cells visited as you sum them
        ↓
Apply the condition after the whole component is collected
```

### Remember this trigger:

> **Count components whose total meets a condition → flood-fill each component once, return its sum from DFS, then test the sum.**
