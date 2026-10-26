# LeetCode 1254 — Number of Closed Islands

## Problem

`grid` contains:

```text
0 → land
1 → water
```

An island is a 4-directionally connected group of land cells. It is **closed** only if it is surrounded by water and does **not** touch the grid boundary.

Return the number of closed islands.

Example:

```text
grid =
[
  [1,1,1,1,1,1,1,0],
  [1,0,0,0,0,1,1,0],
  [1,0,1,0,1,1,1,0],
  [1,0,0,0,0,1,0,1],
  [1,1,1,1,1,1,1,0]
]

answer = 2
```

Land connected to an outside edge is not closed. An island can still be closed if it is near the edge, as long as every path from it stops at water before reaching that edge.

---

# Approach — DFS Returns Whether the Island Is Closed

### Core idea

Start a DFS from every unvisited land cell.

The DFS marks the whole island visited, and returns:

```text
true  → this direction did not escape the grid
false → this direction stepped outside the grid
```

Stepping outside means this island reaches the boundary, so it is not closed.

```text
out of bounds        → false
water / visited land → true
land                 → visit every neighbor and AND their answers
```

The `AND` is the key:

```text
up && down && left && right
```

The island is closed only if **all** four searches remain inside the grid or end at water.

The DFS changes visited land from `0` to `1`. Treating it as water prevents a later starting cell from counting the same island again.

### Why an out-of-bounds call is `false`

Consider land at `(0, 3)`. The recursive call upward visits `(-1, 3)`.

```text
(-1, 3) is outside → false
```

That `false` propagates through the `&&` calls and makes the whole island false. It is exactly the definition of an island that touches the boundary.

### Walkthrough

```text
0 0 0
1 0 1
1 1 1
```

Start at the upper-left `0`:

```text
DFS can move above or left of the grid
out-of-bounds returns false
the whole component returns false
```

Now compare:

```text
1 1 1
1 0 1
1 1 1
```

Every outward recursive path hits `1` before leaving the grid. Water returns `true`, every direction is true, and this island is counted.

### State

```text
grid[r][c] = 0  : unvisited land
grid[r][c] = 1  : water or already visited
```

### Transitions

```text
outside the grid  → false
cell is 1         → true
cell is 0         → mark 1
                    return dfs(up) && dfs(down) && dfs(left) && dfs(right)
```

---

## Java

```java
class Solution {
    public int closedIsland(int[][] grid) {

        int m = grid.length;
        int n = grid[0].length;
        int count = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {

                if (grid[i][j] == 0) {

                    // DFS marks this entire island as visited.
                    if (dfs(grid, i, j)) {
                        count++;
                    }
                }
            }
        }

        return count;
    }

    private boolean dfs(int[][] grid, int r, int c) {

        int m = grid.length;
        int n = grid[0].length;

        // The island escaped the grid, so it touches the boundary.
        if (r < 0 || r >= m || c < 0 || c >= n) {
            return false;
        }

        // Water or land already visited by this DFS cannot make it open.
        if (grid[r][c] == 1) {
            return true;
        }

        // Mark land as visited.
        grid[r][c] = 1;

        boolean up = dfs(grid, r - 1, c);
        boolean down = dfs(grid, r + 1, c);
        boolean left = dfs(grid, r, c - 1);
        boolean right = dfs(grid, r, c + 1);

        // One escaping direction makes the complete island non-closed.
        return up && down && left && right;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int closedIsland(vector<vector<int>>& grid) {

        int m = grid.size();
        int n = grid[0].size();
        int count = 0;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {

                if (grid[i][j] == 0) {

                    // DFS marks this entire island as visited.
                    if (dfs(grid, i, j)) {
                        count++;
                    }
                }
            }
        }

        return count;
    }

private:

    bool dfs(vector<vector<int>>& grid, int r, int c) {

        int m = grid.size();
        int n = grid[0].size();

        // The island escaped the grid, so it touches the boundary.
        if (r < 0 || r >= m || c < 0 || c >= n) {
            return false;
        }

        // Water or land already visited by this DFS cannot make it open.
        if (grid[r][c] == 1) {
            return true;
        }

        // Mark land as visited.
        grid[r][c] = 1;

        bool up = dfs(grid, r - 1, c);
        bool down = dfs(grid, r + 1, c);
        bool left = dfs(grid, r, c - 1);
        bool right = dfs(grid, r, c + 1);

        // One escaping direction makes the complete island non-closed.
        return up && down && left && right;
    }
};
```

---

# Complexity

Let `m` be the number of rows and `n` the number of columns.

```text
Time  : O(m · n)    every cell is marked visited at most once
Space : O(m · n)    worst-case DFS recursion stack
```

The input grid is reused as the visited array, so no separate `visited` matrix is allocated.

---

# Pattern Recognition

```text
Grid islands
        ↓
Need to know whether a component reaches the boundary
        ↓
DFS returns false when it goes out of bounds
        ↓
Water and visited cells return true
        ↓
AND every neighbor result
        ↓
Count a component only when its DFS returns true
```

### Remember this trigger:

> **Closed island → flood-fill each land component. If DFS ever escapes the grid, return false; otherwise the component is enclosed.**
