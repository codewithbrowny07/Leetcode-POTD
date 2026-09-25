# LeetCode 1096 — Brace Expansion II

## Problem

You are given a string `expression` representing a brace expansion.

Return the **sorted list of unique words** after expanding it.

Rules:

```text
{a,b,c}     → union of options
ab          → concatenation
{a,b}{c,d}  → cartesian product  (ac, ad, bc, bd)
nested {}   → evaluate inner expression first
```

Example:

```text
expression = "{a,b}{c,{d,e}}"
answer     = ["ac", "ad", "ae", "bc", "bd", "be"]
```

---

# Approach — Recursive Expand + Combine

### Core idea

Scan a slice `expression[start, end)` and maintain two sets:

```text
current  → strings built for the current union-piece
result   → finished pieces after a ',' union
```

At every character there are three cases:

```text
','   → union: dump current into result, reset current to {""}
'{'   → find matching '}', recurse, then cartesian-product with current
letter → append that letter to every string in current
```

`,` is **union**. Adjacent groups (letters or `{...}`) are **concatenation**, implemented as:

```text
combine(A, B) = { a + b  for a in A, b in B }
```

A `HashSet` / `unordered_set` automatically drops duplicates.

After the whole expression is expanded, copy into a list and sort.

### State

```text
expand(start, end)
  current : set of prefixes for the current comma-separated part
  result  : union of completed parts
```

### Transitions

```text
current = {""}

for each char:
  if ','     → result ∪= current; current = {""}
  if '{'     → nested = expand(inner)
               current = current × nested
  else       → current = { prefix + ch  for prefix in current }

result ∪= current
return result
```

---

## Java

```java
import java.util.*;

class Solution {

    public List<String> braceExpansionII(String expression) {

        Set<String> expanded = expand(expression, 0, expression.length());

        List<String> result = new ArrayList<>(expanded);
        Collections.sort(result);

        return result;
    }

    private Set<String> expand(String str, int start, int end) {

        Set<String> result = new HashSet<>();

        // Strings built so far for the current union piece.
        Set<String> current = new HashSet<>();
        current.add("");

        int index = start;

        while (index < end) {

            char ch = str.charAt(index);

            // Handle union.
            if (ch == ',') {

                result.addAll(current);

                current = new HashSet<>();
                current.add("");

                index++;
            }

            // Handle nested expression.
            else if (ch == '{') {

                int balance = 1;
                int next = index + 1;

                // Find matching '}'.
                while (next < end && balance > 0) {

                    if (str.charAt(next) == '{') {
                        balance++;
                    } else if (str.charAt(next) == '}') {
                        balance--;
                    }

                    next++;
                }

                Set<String> nested =
                    expand(str, index + 1, next - 1);

                // Combine previous × nested (concatenation).
                current = combine(current, nested);

                index = next;
            }

            // Normal character.
            else {

                Set<String> updated = new HashSet<>();

                for (String prefix : current) {
                    updated.add(prefix + ch);
                }

                current = updated;

                index++;
            }
        }

        result.addAll(current);

        return result;
    }

    private Set<String> combine(Set<String> first,
                                Set<String> second) {

        Set<String> result = new HashSet<>();

        for (String left : first) {
            for (String right : second) {
                result.add(left + right);
            }
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

    vector<string> braceExpansionII(string expression) {

        unordered_set<string> expanded =
            expand(expression, 0, (int)expression.size());

        vector<string> result(expanded.begin(), expanded.end());
        sort(result.begin(), result.end());

        return result;
    }

private:

    unordered_set<string> expand(const string& str, int start, int end) {

        unordered_set<string> result;

        // Strings built so far for the current union piece.
        unordered_set<string> current;
        current.insert("");

        int index = start;

        while (index < end) {

            char ch = str[index];

            // Handle union.
            if (ch == ',') {

                result.insert(current.begin(), current.end());

                current.clear();
                current.insert("");

                index++;
            }

            // Handle nested expression.
            else if (ch == '{') {

                int balance = 1;
                int next = index + 1;

                // Find matching '}'.
                while (next < end && balance > 0) {

                    if (str[next] == '{') {
                        balance++;
                    } else if (str[next] == '}') {
                        balance--;
                    }

                    next++;
                }

                unordered_set<string> nested =
                    expand(str, index + 1, next - 1);

                // Combine previous × nested (concatenation).
                current = combine(current, nested);

                index = next;
            }

            // Normal character.
            else {

                unordered_set<string> updated;

                for (const string& prefix : current) {
                    updated.insert(prefix + ch);
                }

                current = move(updated);

                index++;
            }
        }

        result.insert(current.begin(), current.end());

        return result;
    }

    unordered_set<string> combine(const unordered_set<string>& first,
                                  const unordered_set<string>& second) {

        unordered_set<string> result;

        for (const string& left : first) {
            for (const string& right : second) {
                result.insert(left + right);
            }
        }

        return result;
    }
};
```

---

# Complexity

Let `k` be the number of unique expanded words, and `L` the length of the expression.

```text
Time  : O(k · wordLength + L)   cartesian products dominate
Space : O(k · wordLength)       sets of generated strings
```

Worst case is exponential in the size of nested products, which matches the output size.

---

# Pattern Recognition

```text
Braces + commas + concatenation
        ↓
',' is union
adjacent groups are cartesian product
        ↓
Recurse on matching { }
        ↓
Dedup with a set, then sort
```

### Remember this trigger:

> **Brace expansion / RLE-style grammar → recursive parse of matching braces, union on `,`, product on juxtaposition.**
