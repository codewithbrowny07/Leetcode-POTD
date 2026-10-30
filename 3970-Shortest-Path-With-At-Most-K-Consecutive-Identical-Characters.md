# LeetCode 3970 — Shortest Path With At Most K Consecutive Identical Characters

## Problem

You are given a **directed**, weighted graph with `n` nodes:

```text
edges[i] = [from, to, weight]
```

`labels[node]` is the lowercase character assigned to that node.

Find the minimum total edge weight of a path from node `0` to node `n - 1` such that the labels along the path never contain more than `k` consecutive copies of the same character.

Return `-1` when no valid path exists.

Example:

```text
labels = "aabb"
k = 2

Path labels: a → a → b → b    valid
Path labels: a → a → a        invalid, 3 consecutive 'a' values
```

```text
1 <= n <= 5 * 10^4
0 <= edges.length <= 5 * 10^4
1 <= edge weight <= 10^4
1 <= k <= 50
```

---

# Approach — Dijkstra on `(node, consecutiveCount)`

### Why one distance per node is not enough

In ordinary Dijkstra:

```text
dist[node] = cheapest cost to reach node
```

That is not enough here. The same node can be reached with different lengths of the current label run:

```text
labels[node] = 'a'

reach node with ...aa     consecutiveCount = 2
reach node with ...a      consecutiveCount = 1
```

The count `1` state can still move to another `'a'` if `k = 2`. The count `2` state cannot. Even if count `2` was cheaper, it may be worse for all valid continuations.

So the actual state is:

```text
(node, consecutiveCount)
```

```text
dist[node][count] = minimum cost to reach node
                    with exactly count copies of labels[node] at the end
```

### Transition

From state `(node, count)`, follow a directed edge to `v`.

```text
labels[node] == labels[v]  → newCount = count + 1
otherwise                  → newCount = 1
```

If `newCount > k`, that move is invalid and is skipped.

Otherwise this is a normal Dijkstra relaxation:

```text
newCost = cost + edgeWeight

if newCost < dist[v][newCount]:
    update dist
    push (newCost, v, newCount) into the min-heap
```

### Start state

The path includes node `0`, so its label already begins a run of length `1`:

```text
dist[0][1] = 0
```

Not `0`. If `k = 1`, the very next node cannot have the same label as node `0`.

### Why the first destination state is the answer

All edge weights are positive. Dijkstra removes states from the min-heap in non-decreasing cost order.

The first state removed for node `n - 1` is the cheapest valid destination state across every possible trailing count, so return its cost immediately.

### Walkthrough

```text
labels = "aaba"
k = 2

At node 0:
state = (0, 1), cost = 0

move to a node labeled 'a':
state = (next, 2)

move to another node labeled 'a':
newCount = 3
3 > k, so skip that edge

move to a node labeled 'b':
labels differ, newCount = 1
state = (next, 1)
```

Changing labels resets the run. Matching labels extends it.

### State

```text
graph[u]        : directed edges {v, weight} from u
dist[u][count]  : cheapest valid cost for state (u, count)
pq              : {cost, node, consecutiveCount}, ordered by cost
```

### Transitions

```text
same next label      → count + 1
different next label → 1
newCount > k         → reject edge
otherwise            → normal Dijkstra relaxation
```

---

## Java

```java
class Solution {

    public int shortestPath(int n, int[][] edges, String labels, int k) {

        List<int[]>[] graph = new ArrayList[n];

        for (int i = 0; i < n; i++) {
            graph[i] = new ArrayList<>();
        }

        // Directed graph: e[0] -> e[1].
        for (int[] e : edges) {
            graph[e[0]].add(new int[]{e[1], e[2]});
        }

        // dist[node][count] = minimum cost for this complete state.
        int[][] dist = new int[n][k + 1];

        for (int[] row : dist) {
            Arrays.fill(row, Integer.MAX_VALUE);
        }

        // {cost, node, consecutiveCount}
        PriorityQueue<int[]> pq = new PriorityQueue<>(
            (a, b) -> Integer.compare(a[0], b[0])
        );

        // Node 0 itself starts a run of one copy of labels[0].
        dist[0][1] = 0;
        pq.offer(new int[]{0, 0, 1});

        while (!pq.isEmpty()) {

            int[] curr = pq.poll();

            int cost = curr[0];
            int node = curr[1];
            int count = curr[2];

            // Ignore a heap entry made stale by a better relaxation.
            if (cost > dist[node][count]) {
                continue;
            }

            // First valid destination state popped is globally cheapest.
            if (node == n - 1) {
                return cost;
            }

            for (int[] next : graph[node]) {

                int v = next[0];
                int weight = next[1];

                int newCount;

                if (labels.charAt(node) == labels.charAt(v)) {
                    newCount = count + 1;
                } else {
                    newCount = 1;
                }

                if (newCount > k) {
                    continue;
                }

                int newCost = cost + weight;

                if (newCost < dist[v][newCount]) {
                    dist[v][newCount] = newCost;
                    pq.offer(new int[]{newCost, v, newCount});
                }
            }
        }

        return -1;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int shortestPath(int n, vector<vector<int>>& edges, string labels, int k) {

        vector<vector<pair<int, int>>> graph(n);

        // Directed graph: edge[0] -> edge[1].
        for (const auto& edge : edges) {
            graph[edge[0]].push_back({edge[1], edge[2]});
        }

        // dist[node][count] = minimum cost for this complete state.
        vector<vector<int>> dist(n, vector<int>(k + 1, INT_MAX));

        // {cost, node, consecutiveCount}; greater makes this a min-heap.
        using State = tuple<int, int, int>;
        priority_queue<State, vector<State>, greater<State>> pq;

        // Node 0 itself starts a run of one copy of labels[0].
        dist[0][1] = 0;
        pq.push({0, 0, 1});

        while (!pq.empty()) {

            auto [cost, node, count] = pq.top();
            pq.pop();

            // Ignore a heap entry made stale by a better relaxation.
            if (cost > dist[node][count]) {
                continue;
            }

            // First valid destination state popped is globally cheapest.
            if (node == n - 1) {
                return cost;
            }

            for (auto [v, weight] : graph[node]) {

                int newCount;

                if (labels[node] == labels[v]) {
                    newCount = count + 1;
                } else {
                    newCount = 1;
                }

                if (newCount > k) {
                    continue;
                }

                int newCost = cost + weight;

                if (newCost < dist[v][newCount]) {
                    dist[v][newCount] = newCost;
                    pq.push({newCost, v, newCount});
                }
            }
        }

        return -1;
    }
};
```

---

# Complexity

Let `E = edges.length`.

```text
States : O(n · k)
Time   : O((n · k + E · k) · log(n · k))
Space  : O(n · k + E)
```

There can be up to `k` states for each node. Each state can examine the node's outgoing edges, so the graph work is `O(E · k)`.

`k <= 50`, which keeps the expanded graph manageable.

---

# Pattern Recognition

```text
Shortest path with a path-history restriction
        ↓
The current node alone loses important information
        ↓
Add the smallest history needed for the next decision
        ↓
state = (node, run length of the current label)
        ↓
Run Dijkstra over those expanded states
```

### Remember this trigger:

> **Shortest path with a bounded consecutive-run rule → Dijkstra on `(node, runLength)`. Reset the run when the label changes and discard states over the limit.**
