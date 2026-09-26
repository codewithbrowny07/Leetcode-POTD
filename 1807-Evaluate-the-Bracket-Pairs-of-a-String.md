# LeetCode 1807 — Evaluate the Bracket Pairs of a String

## Problem

You are given a string `s` that contains bracket pairs, each holding a non-empty key.

You are also given `knowledge`, where each entry is `[key, value]`.

Evaluate every bracket pair:

```text
(key)  →  value     if the key is in knowledge
(key)  →  ?         if the key is unknown
```

The parentheses are removed too. There are **no nested brackets**. Each key in `knowledge` is unique.

Example:

```text
s         = "(name)is(age)yearsold"
knowledge = [["name","bob"],["age","two"]]
answer    = "bobistwoyearsold"
```

---

# Approach 1 — Collect ranges, then replace

### Core idea

1. Scan `s` and record every `[start, end)` of a key inside `(...)`.
2. Put `knowledge` into a map.
3. Replace each key range in a `StringBuilder`.

### Why this version fails

Two bugs show up if you replace **left to right** and only the key (not the parentheses):

```text
1. replace(start, end, value) leaves '(' and ')' in the string.
   "(name)" becomes "(bob)", not "bob".

2. A longer or shorter value shifts every later index.
   The ranges you stored were for the original string, so the next
   replace hits the wrong characters.
```

Replacing from the **right** and covering the parentheses (`start - 1` through `end + 1`) fixes both, because earlier indices stay valid.

That fix works, but it is easy to get wrong. The one-pass builder below is the version to use.

### Java (broken left-to-right replace)

```java
class Solution {
    public String evaluate(String s, List<List<String>> knowledge) {
        StringBuilder res = new StringBuilder(s);
        ArrayList<ArrayList<Integer>> idxs = new ArrayList<>();

        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (ch == '(') {
                int start = i + 1;
                while (i < s.length() && s.charAt(i) != ')') {
                    i++;
                }
                int end = i;
                ArrayList<Integer> curr = new ArrayList<>();
                curr.add(start);
                curr.add(end);
                idxs.add(curr);
            }
        }

        Map<String, String> mp = new HashMap<>();
        for (int i = 0; i < knowledge.size(); i++) {
            mp.put(knowledge.get(i).get(0), knowledge.get(i).get(1));
        }

        // Left-to-right replace on the original indices is wrong:
        // parentheses stay, and later ranges shift.
        for (int i = 0; i < idxs.size(); i++) {
            int start = idxs.get(i).get(0);
            int end = idxs.get(i).get(1);
            String key = s.substring(start, end);
            String value = mp.get(key);

            if (value != null) {
                res.replace(start, end, value);
            } else {
                res.replace(start, end, "?");
            }
        }

        return res.toString();
    }
}
```

---

# Approach 2 — One-pass builder ⭐

This is the correct approach.

### Core idea

Build a **new** string. Never edit the original in place, so indices never shift.

```text
normal char  → append it
'('          → read until ')', look up the key, append value or "?"
               do not append '(' or ')'
```

`knowledge` goes into a hash map first, so each lookup is `O(1)` average.

### State

```text
mp   : key → value
res  : answer built so far
i    : current index in s
```

### Transitions

```text
s[i] == '('  → key = s[i+1 .. ')'), append mp.getOrDefault(key, "?"), jump i to ')'
otherwise    → append s[i]
```

---

## Java

```java
class Solution {
    public String evaluate(String s, List<List<String>> knowledge) {

        Map<String, String> mp = new HashMap<>();

        for (List<String> k : knowledge) {
            mp.put(k.get(0), k.get(1));
        }

        StringBuilder res = new StringBuilder();

        for (int i = 0; i < s.length(); i++) {

            if (s.charAt(i) == '(') {

                int start = i + 1;

                while (s.charAt(i) != ')') {
                    i++;
                }

                int end = i;

                String key = s.substring(start, end);

                String value = mp.getOrDefault(key, "?");

                res.append(value);

            } else {
                res.append(s.charAt(i));
            }
        }

        return res.toString();
    }
}
```

---

## C++

```cpp
class Solution {
public:

    string evaluate(string s, vector<vector<string>>& knowledge) {

        unordered_map<string, string> mp;

        for (auto& k : knowledge) {
            mp[k[0]] = k[1];
        }

        string res;
        res.reserve(s.size());

        for (int i = 0; i < (int)s.size(); i++) {

            if (s[i] == '(') {

                int start = i + 1;

                while (s[i] != ')') {
                    i++;
                }

                int end = i;

                string key = s.substr(start, end - start);

                auto it = mp.find(key);
                res += (it == mp.end() ? "?" : it->second);

            } else {
                res.push_back(s[i]);
            }
        }

        return res;
    }
};
```

---

# Complexity

Let `n = s.length()` and `m = knowledge.length()`.

```text
Time  : O(n + m)     one scan of s, one pass over knowledge
Space : O(n + m)     map plus the answer string
```

Keys and values are short (length ≤ 10 under the usual constraints), so substring and lookup cost stays small.

---

# Pattern Recognition

```text
Replace marked spans inside a string
        ↓
Do not edit the original left-to-right
        ↓
Hash the replacements
        ↓
Scan once and append into a new buffer
```

### Remember this trigger:

> **In-place replace shifts later indices. When the output length can change, build a new string in one pass.**
