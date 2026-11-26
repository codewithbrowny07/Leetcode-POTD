# LeetCode 3360 — Stone Removal Game

## Problem

Alice and Bob play a game with a pile of `n` stones. Alice moves first.

```text
Alice's first move  : remove exactly 10 stones
every later move    : remove exactly 1 fewer stone than the opponent's previous move
```

The player who cannot make a move loses.

Return `true` if Alice wins, otherwise `false`.

Example:

```text
n = 12

Alice removes 10   → 2 stones left
Bob must remove 9  → only 2 left, Bob cannot move

answer = true
```

```text
n = 1

Alice must remove 10 → only 1 stone, Alice cannot move

answer = false
```

```text
n = 19

Alice removes 10   → 9 left
Bob removes 9      → 0 left
Alice must remove 8 → cannot move

answer = false
```

```text
1 <= n <= 50
```

---

# Approach — Simulate the Game

### Core idea

The move sizes are fixed in advance: `10, 9, 8, 7, ...`. Nobody picks how many to take, so there are no choices to search over. Walk through the moves in order, and the first player who faces a move larger than the remaining pile loses.

```text
n < 10               Alice cannot even take 10    → false
otherwise            Alice takes 10 and the pile becomes n - 10
```

After that, Bob moves with `moves = 9`, then Alice with `8`, and so on. Each step:

```text
if moves > n        the player to move cannot move → the other player wins
else                n -= moves, moves--, switch player
```

`turn` records who is to move: `0` is Bob's turn (Alice just moved) and `1` is Alice's turn.

```text
the player to move cannot move and turn == 0  → Bob is stuck  → Alice wins
the player to move cannot move and turn == 1  → Alice is stuck → Alice loses
```

### Walkthrough

```text
n = 12
n < 10? no
n = 12 - 10 = 2, moves = 9, turn = 0

moves 9 > n 2, turn = 0 → Bob cannot move → true
```

```text
n = 19
n = 19 - 10 = 9, moves = 9, turn = 0

moves 9 <= 9 → n = 0, moves = 8, turn = 1
moves 8 > 0, turn = 1 → Alice cannot move → false
```

```text
n = 27
n = 17, moves = 9, turn = 0

Bob   takes 9 → n = 8, moves = 8, turn = 1
Alice takes 8 → n = 0, moves = 7, turn = 0
moves 7 > 0, turn = 0 → Bob cannot move → true
```

### Who wins for every n

The total stones used after `m` moves is `10 + 9 + ... + (11 - m)`.

```text
m = 1 →  10
m = 2 →  19
m = 3 →  27
m = 4 →  34
m = 5 →  40
m = 6 →  45
m = 7 →  49
m = 8 →  52   (above the largest n)
```

The game lasts for the largest `m` whose total is at most `n`. Alice wins exactly when that `m` is odd, because then Alice made the last move.

```text
n in [ 1,  9]  m = 0   Alice loses
n in [10, 18]  m = 1   Alice wins
n in [19, 26]  m = 2   Alice loses
n in [27, 33]  m = 3   Alice wins
n in [34, 39]  m = 4   Alice loses
n in [40, 44]  m = 5   Alice wins
n in [45, 48]  m = 6   Alice loses
n in [49, 50]  m = 7   Alice wins
```

The sequence needs `55` stones to reach a move of size `1` (`10 + 9 + ... + 1`), and `n` is at most `50`. A player always gets stuck before `moves` reaches `0`, so the last line of the loop version never runs under these constraints.

### State

```text
n     : stones left
moves : stones the next player must remove
turn  : 0 means Bob moves next, 1 means Alice moves next
```

### Transitions

```text
if n < 10                    → false
n = n - 10, moves = 9, turn = 0

while moves > 0
    if moves > n             → return turn == 0   (Alice wins when Bob is stuck)
    n -= moves
    moves--
    flip turn
```

---

## Java

```java
class Solution {
    public boolean canAliceWin(int n) {

        // Alice's first move is exactly 10 stones.
        if (n < 10) {
            return false;
        }

        int moves = 9;
        n = n - 10;

        // 0 → Bob moves next, 1 → Alice moves next.
        int turn = 0;

        while (moves > 0) {

            // The player to move cannot take that many stones, so that player loses.
            if (moves > n) {
                return turn == 1 ? false : true;
            }

            n = n - moves;
            moves--;
            turn = (turn == 1) ? 0 : 1;
        }

        return turn == 1 ? false : true;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    bool canAliceWin(int n) {

        // Alice's first move is exactly 10 stones.
        if (n < 10) {
            return false;
        }

        int moves = 9;
        n = n - 10;

        // 0 → Bob moves next, 1 → Alice moves next.
        int turn = 0;

        while (moves > 0) {

            // The player to move cannot take that many stones, so that player loses.
            if (moves > n) {
                return turn == 1 ? false : true;
            }

            n = n - moves;
            moves--;
            turn = (turn == 1) ? 0 : 1;
        }

        return turn == 1 ? false : true;
    }
};
```

---

# Complexity

```text
Time  : O(1)
Space : O(1)
```

The loop starts at `moves = 9` and lowers it by `1` each pass. It runs at most `9` times, whatever the value of `n`. The pasted `O(n)` is a safe upper bound, and the tight bound is `O(1)`.

---

# Pattern Recognition

```text
A game where every move size is fixed in advance
        ↓
No choices, so no search or DP
        ↓
Simulate the shrinking move sizes and watch whose turn it is
        ↓
The player who cannot cover the next move loses
```

### Remember this trigger:

> **Fixed, decreasing move sizes in a two-player game → simulate move by move, and the first player facing `moves > n` loses. Alice wins exactly when the number of completed moves is odd.**
