# LeetCode 1569 — Number of Ways to Reorder Array to Get Same BST

## Problem

`nums` is a permutation of `1 .. n`. Insert the values into an empty BST in the order they appear. The first value becomes the root.

Return how many **other** orders of the same values build that **same** BST. Take the answer modulo `10^9 + 7`.

Example:

```text
nums = [2, 1, 3]

      2
     / \
    1   3

[2, 3, 1] builds the same tree
answer = 1
```

```text
nums = [3, 4, 5, 1, 2]
answer = 5

nums = [1, 2, 3]
answer = 0          only the original order builds this chain
```

```text
1 <= nums.length <= 1000
nums is a permutation of 1 .. n
```

---

# Approach — Root Is Fixed, Then Interleave the Subtrees

### Core idea

BST insertion always uses the first remaining value as the root of that subtree. So the root of the whole tree is fixed as `nums[0]`.

Every later value goes left or right by comparison with that root, and it keeps its relative order with the other values on the same side:

```text
left  = values < root, in the order they appear
right = values > root, in the order they appear
```

The left subtree can itself be produced in `solve(left)` orders. The right subtree can be produced in `solve(right)` orders.

After the root is written, the remaining `n - 1` positions are a mix of the left values and the right values. Choose `left.size()` of those positions for the left group. The rest belong to the right group. Inside the chosen positions, the left values must appear in one of their valid orders. Same for the right.

```text
ways = C(n - 1, left.size()) * leftWays * rightWays
```

`C(n - 1, left.size())` is the number of ways to pick the slots. It does not depend on which side we count, because `C(n - 1, left) = C(n - 1, right)`.

A list of length `0`, `1`, or `2` has only one valid order, so those calls return `1`. The empty side is the `1` in a product: it does not change the count.

The recursion counts the original order too. The problem wants the other orders, so the final answer is `ways - 1`.

### Walkthrough

```text
nums = [3, 4, 5, 1, 2]

root 3
left  = [1, 2]     one order
right = [4, 5]     one order

remaining slots = 4
choose 2 of them for the left values

C(4, 2) * 1 * 1 = 6
answer = 6 - 1 = 5
```

The six orders are the original `[3, 4, 5, 1, 2]` plus the five reorderings that still put `3` first, keep `1` before `2`, and keep `4` before `5`.

```text
nums = [1, 2, 3]

root 1
left is empty
right = [2, 3]

C(2, 0) * 1 * 1 = 1
answer = 1 - 1 = 0
```

### Combinations

Pascal's rule fills the table before the recursion starts:

```text
C(i, 0) = C(i, i) = 1
C(i, j) = C(i - 1, j - 1) + C(i - 1, j)
```

Every entry is stored modulo `10^9 + 7`.

### State

```text
nums[0]   : root of this subtree
left      : values that must stay in the left subtree
right     : values that must stay in the right subtree
comb[i][j]: C(i, j) mod 10^9 + 7
```

### Transitions

```text
n <= 2                         → 1 way
otherwise                      → split by the root
ways = C(n - 1, left.size()) * solve(left) * solve(right)
final answer                   → (ways - 1) mod 10^9 + 7
```

---

## Java

```java
class Solution {
    static final long MOD = 1_000_000_007;
    long[][] comb;

    private long solve(List<Integer> nums) {
        int n = nums.size();

        // Empty, one node, or root plus one child: only one order.
        if (n <= 2) {
            return 1;
        }

        int root = nums.get(0);
        List<Integer> left = new ArrayList<>();
        List<Integer> right = new ArrayList<>();

        for (int i = 1; i < n; i++) {
            if (nums.get(i) < root) {
                left.add(nums.get(i));
            } else {
                right.add(nums.get(i));
            }
        }

        long leftWays = solve(left);
        long rightWays = solve(right);

        // Choose slots for the left values among the n - 1 positions after the root.
        long ways = comb[n - 1][left.size()];
        ways = ways * leftWays % MOD;
        ways = ways * rightWays % MOD;

        return ways;
    }

    public int numOfWays(int[] nums) {
        int n = nums.length;

        comb = new long[n + 1][n + 1];

        for (int i = 0; i <= n; i++) {
            comb[i][0] = comb[i][i] = 1;
            for (int j = 1; j < i; j++) {
                comb[i][j] =
                    (comb[i - 1][j - 1] + comb[i - 1][j]) % MOD;
            }
        }

        List<Integer> list = new ArrayList<>();
        for (int num : nums) {
            list.add(num);
        }

        // solve counts the original order too. The problem wants the others.
        return (int) ((solve(list) - 1 + MOD) % MOD);
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int numOfWays(vector<int>& nums) {

        int n = nums.size();
        comb.assign(n + 1, vector<long long>(n + 1, 0));

        for (int i = 0; i <= n; i++) {
            comb[i][0] = comb[i][i] = 1;
            for (int j = 1; j < i; j++) {
                comb[i][j] =
                    (comb[i - 1][j - 1] + comb[i - 1][j]) % MOD;
            }
        }

        // solve counts the original order too. The problem wants the others.
        return (int) ((solve(nums) - 1 + MOD) % MOD);
    }

private:

    static const long long MOD = 1000000007LL;
    vector<vector<long long>> comb;

    long long solve(const vector<int>& nums) {
        int n = nums.size();

        // Empty, one node, or root plus one child: only one order.
        if (n <= 2) {
            return 1;
        }

        int root = nums[0];
        vector<int> left, right;

        for (int i = 1; i < n; i++) {
            if (nums[i] < root) {
                left.push_back(nums[i]);
            } else {
                right.push_back(nums[i]);
            }
        }

        long long leftWays = solve(left);
        long long rightWays = solve(right);

        // Choose slots for the left values among the n - 1 positions after the root.
        long long ways = comb[n - 1][left.size()];
        ways = ways * leftWays % MOD;
        ways = ways * rightWays % MOD;

        return ways;
    }
};
```

---

# Complexity

Let `n = nums.length`. `n` is at most `1000`.

```text
Time  : O(n²)    the combination table, and each partition scans its list
Space : O(n²)    the combination table
```

A skewed tree also uses `O(n)` recursion depth. Each call copies its left and right values into new lists.

---

# Pattern Recognition

```text
Same BST from a different insertion order
        ↓
The first value is the root and cannot move
        ↓
Smaller values stay on the left, in a valid order for that subtree
Larger values stay on the right
        ↓
Interleave those two sequences in the remaining slots
        ↓
C(n - 1, leftCount) * leftWays * rightWays
        ↓
Subtract 1, because the original order is not counted
```

### Remember this trigger:

> **Count insertion orders that rebuild one BST → the root is fixed, recurse on the two sides, and multiply by the ways to choose their slots.**
