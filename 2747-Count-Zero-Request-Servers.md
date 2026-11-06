# LeetCode 2747 — Count Zero Request Servers

## Problem

There are `n` servers, numbered `1` through `n`.

`logs[i] = [serverId, time]` means server `serverId` received a request at time `time`.

For each query `queries[i]`, count how many servers received **no** request in the inclusive window

```text
[queries[i] - x, queries[i]]
```

Return those counts in the original query order.

Example:

```text
n = 3
logs = [[1,3],[2,6],[1,5]]
x = 5
queries = [10, 11]

query 10 → window [5, 10]
  server 1 at time 5, server 2 at time 6
  server 3 is silent
  answer = 1

query 11 → window [6, 11]
  only server 2 at time 6
  answer = 2

result = [1, 2]
```

```text
n = 3
logs = [[2,4],[2,1],[1,2],[3,1]]
x = 2
queries = [3, 4]

query 3 → window [1, 3] → every server is active → 0
query 4 → window [2, 4] → server 3 is silent    → 1

result = [0, 1]
```

```text
1 <= n <= 10^5
1 <= logs.length <= 10^5
1 <= queries.length <= 10^5
1 <= logs[i][0] <= n
1 <= logs[i][1] <= 10^6
1 <= x <= 10^5
x < queries[i] <= 10^6
```

A server is active when it has **at least one** log inside the window. Extra logs from the same server do not change the count. The answer is `n - (number of active servers)`.

---

# Approach 1 — Scan Every Log for Every Query

### Core idea

Rebuild the active set from scratch for each query.

```text
window = [query - x, query]
put every server whose log falls inside that window into a set
answer = n - set size
```

The set collapses duplicate requests from the same server.

### State

```text
windowStart, windowEnd : inclusive range for this query
activeServers          : distinct servers with a log in that range
```

### Transitions

```text
for each query
    clear the set
    for each log
        if windowStart <= time <= windowEnd
            insert server
    answer = n - set size
```

This is correct. With `10^5` queries and `10^5` logs it does about `10^10` checks, so it does not pass the limits.

---

## Java

```java
class Solution {
    public int[] countServers(int n, int[][] logs, int x, int[] queries) {

        int totalQueries = queries.length;
        int[] answer = new int[totalQueries];

        for (int i = 0; i < totalQueries; i++) {

            int windowStart = queries[i] - x;
            int windowEnd = queries[i];

            // Distinct servers that received a request in this window.
            Set<Integer> activeServers = new HashSet<>();

            for (int[] log : logs) {

                int server = log[0];
                int time = log[1];

                if (time >= windowStart && time <= windowEnd) {
                    activeServers.add(server);
                }
            }

            answer[i] = n - activeServers.size();
        }

        return answer;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<int> countServers(int n, vector<vector<int>>& logs, int x, vector<int>& queries) {

        int totalQueries = queries.size();
        vector<int> answer(totalQueries);

        for (int i = 0; i < totalQueries; i++) {

            int windowStart = queries[i] - x;
            int windowEnd = queries[i];

            // Distinct servers that received a request in this window.
            unordered_set<int> activeServers;

            for (auto& log : logs) {

                int server = log[0];
                int time = log[1];

                if (time >= windowStart && time <= windowEnd) {
                    activeServers.insert(server);
                }
            }

            answer[i] = n - (int)activeServers.size();
        }

        return answer;
    }
};
```

---

# Complexity — Approach 1

Let `m = logs.length` and `q = queries.length`.

```text
Time  : O(q · m)   every query scans every log
Space : O(n)       the active set holds at most one entry per server
```

---

# Approach 2 — Sort Each Server, Then Binary Search

### Core idea

A server is active in `[left, right]` when **any** of its times falls in that range.

Group the times by server and sort each group. The first time that is `>= left` is the earliest request that could still be inside the window. If that time is also `<= right`, the server is active. Later times do not need to be checked.

```text
idx = first index with times[idx] >= left
active when idx exists and times[idx] <= right
```

### State

```text
serverLogs[server] : sorted request times for that server
left, right        : [query - x, query]
idx                : lower bound of left inside that server's times
```

### Transitions

```text
bucket every log by server id, then sort each bucket
for each query
    active = 0
    for each server
        if the first time >= left is still <= right
            active++
    answer = n - active
```

`lowerBound` returns the insertion point, which is `times.size()` when every time is smaller than `left`. That server is silent.

This is still too slow. Every query walks all `n` servers, and both `n` and `q` can be `10^5`.

---

## Java

```java
class Solution {

    // First index where arr[index] >= target.
    public int lowerBound(List<Integer> arr, int target) {

        int low = 0;
        int high = arr.size();

        while (low < high) {

            int mid = low + (high - low) / 2;

            if (arr.get(mid) >= target) {
                high = mid;
            } else {
                low = mid + 1;
            }
        }

        return low;
    }

    public int[] countServers(int n, int[][] logs, int x, int[] queries) {

        List<Integer>[] serverLogs = new ArrayList[n + 1];

        for (int i = 1; i <= n; i++) {
            serverLogs[i] = new ArrayList<>();
        }

        for (int[] log : logs) {
            serverLogs[log[0]].add(log[1]);
        }

        for (int i = 1; i <= n; i++) {
            Collections.sort(serverLogs[i]);
        }

        int[] ans = new int[queries.length];

        for (int i = 0; i < queries.length; i++) {

            int left = queries[i] - x;
            int right = queries[i];
            int active = 0;

            for (int j = 1; j <= n; j++) {

                List<Integer> times = serverLogs[j];
                int idx = lowerBound(times, left);

                if (idx < times.size() && times.get(idx) <= right) {
                    active++;
                }
            }

            ans[i] = n - active;
        }

        return ans;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    // First index where arr[index] >= target.
    int lowerBound(const vector<int>& arr, int target) {

        int low = 0;
        int high = arr.size();

        while (low < high) {

            int mid = low + (high - low) / 2;

            if (arr[mid] >= target) {
                high = mid;
            } else {
                low = mid + 1;
            }
        }

        return low;
    }

    vector<int> countServers(int n, vector<vector<int>>& logs, int x, vector<int>& queries) {

        vector<vector<int>> serverLogs(n + 1);

        for (auto& log : logs) {
            serverLogs[log[0]].push_back(log[1]);
        }

        for (int i = 1; i <= n; i++) {
            sort(serverLogs[i].begin(), serverLogs[i].end());
        }

        vector<int> ans(queries.size());

        for (int i = 0; i < (int)queries.size(); i++) {

            int left = queries[i] - x;
            int right = queries[i];
            int active = 0;

            for (int j = 1; j <= n; j++) {

                vector<int>& times = serverLogs[j];
                int idx = lowerBound(times, left);

                if (idx < (int)times.size() && times[idx] <= right) {
                    active++;
                }
            }

            ans[i] = n - active;
        }

        return ans;
    }
};
```

---

# Complexity — Approach 2

```text
Time  : O(m log m + q · n · log m)
        sorting the buckets is O(m log m)
        each query binary-searches every server

Space : O(m)   the time lists
```

The `q · n` walk is the part that fails the limits.

---

# Approach 3 — Sort Queries and Slide One Window ⭐

### Core idea

Answer the queries from earliest to latest. Then both ends of the window only move forward:

```text
query grows
    → end   = query       grows
    → start = query - x   grows
```

Sort the logs by time as well. Two pointers maintain the logs that sit inside the current window:

```text
j walks forward and adds logs with time <= end
i walks forward and drops logs with time <  start
```

Logs between `i` and `j` are exactly the requests in `[start, end]`.

A server can have several logs in that range, so the map stores a **count**, not just presence. When the count falls to `0`, the server leaves the window and is erased. `map.size()` is then the number of active servers.

Queries are sorted, so the answer must be written back at the original index.

### Walkthrough

```text
n = 3
logs sorted by time = [1,3], [1,5], [2,6]
queries sorted      = (10, index 0), (11, index 1)
x = 5

query 10, window [5, 10]
    add [1,3], [1,5], [2,6]
    drop [1,3] because 3 < 5
    map = {1: 1, 2: 1}
    result[0] = 3 - 2 = 1

query 11, window [6, 11]
    nothing new to add
    drop [1,5] because 5 < 6
    map = {2: 1}
    result[1] = 3 - 1 = 2
```

Same server, two times: server `1` stays active after `[1,3]` is dropped, because `[1,5]` is still inside `[5, 10]`. The count is what makes that correct.

### State

```text
logs            : sorted by time
timeIndex       : (query time, original index), sorted by time
i               : first log that is still inside the window
j               : first log not yet considered
mp              : server → how many of its logs are inside the window
```

### Transitions

```text
while the next log's time <= end
    increment that server's count
    j++

while the oldest included log's time < start
    decrement that server's count
    erase the server when the count hits 0
    i++

result[original index] = n - mp.size()
```

`i` and `j` never move backward, so every log is added once and removed once.

---

## Java

```java
class Solution {
    public int[] countServers(int n, int[][] logs, int x, int[] queries) {

        int m = logs.length;

        // Times fit in 1..10^6, so subtracting them cannot overflow.
        Arrays.sort(logs, (a, b) -> a[1] - b[1]);

        int q = queries.length;
        int[][] timeIndex = new int[q][2];

        for (int i = 0; i < q; i++) {
            timeIndex[i][0] = queries[i];
            timeIndex[i][1] = i;
        }

        Arrays.sort(timeIndex, (a, b) -> a[0] - b[0]);

        // server → number of its logs currently inside the window
        Map<Integer, Integer> mp = new HashMap<>();
        int[] result = new int[q];

        int i = 0;
        int j = 0;

        for (int[] pair : timeIndex) {

            int curtime = pair[0];
            int ind = pair[1];

            int start = curtime - x;
            int end = curtime;

            // Pull in every log that has reached the right edge.
            while (j < m && logs[j][1] <= end) {
                int serverId = logs[j][0];
                mp.put(serverId, mp.getOrDefault(serverId, 0) + 1);
                j++;
            }

            // Drop logs that have fallen behind the left edge.
            while (i < m && logs[i][1] < start) {
                int serverId = logs[i][0];
                mp.put(serverId, mp.get(serverId) - 1);
                if (mp.get(serverId) == 0) {
                    mp.remove(serverId);
                }
                i++;
            }

            result[ind] = n - mp.size();
        }

        return result;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    vector<int> countServers(int n, vector<vector<int>>& logs, int x, vector<int>& queries) {

        int m = logs.size();

        sort(logs.begin(), logs.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[1] < b[1];
        });

        int q = queries.size();
        vector<pair<int, int>> timeIndex(q);

        for (int i = 0; i < q; i++) {
            timeIndex[i] = {queries[i], i};
        }

        sort(timeIndex.begin(), timeIndex.end());

        // server → number of its logs currently inside the window
        unordered_map<int, int> mp;
        vector<int> result(q);

        int i = 0;
        int j = 0;

        for (auto& entry : timeIndex) {

            int curtime = entry.first;
            int ind = entry.second;

            int start = curtime - x;
            int end = curtime;

            while (j < m && logs[j][1] <= end) {
                mp[logs[j][0]]++;
                j++;
            }

            while (i < m && logs[i][1] < start) {
                int serverId = logs[i][0];
                mp[serverId]--;
                if (mp[serverId] == 0) {
                    mp.erase(serverId);
                }
                i++;
            }

            result[ind] = n - (int)mp.size();
        }

        return result;
    }
};
```

---

# Complexity — Approach 3

```text
Time  : O(m log m + q log q)
        both sorts dominate
        each log is inserted once and removed once

Space : O(m + q)
        the index pairs, plus the map of servers currently inside the window
```

---

# Pattern Recognition

```text
Many range queries on a time line
        ↓
Sort the events
        ↓
Sort the queries too, and remember their original index
        ↓
Both window edges only move forward
        ↓
Two pointers + a count of who is still inside
```

### Remember this trigger:

> **Count how many ids appear inside `[query - x, query]` for many queries → sort by time and slide one window, keeping a frequency map so an id stays active until its last event leaves.**
