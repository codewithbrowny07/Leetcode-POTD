# LeetCode 1352 — Product of the Last K Numbers

## Problem

Design a class that keeps a stream of integers and answers product queries over the most recent numbers.

```text
ProductOfNumbers()        creates an empty stream
add(num)                  appends num to the stream
getProduct(k)             returns the product of the last k numbers
```

`getProduct(k)` is only called when the stream already holds at least `k` numbers.

Example:

```text
add(3)  add(0)  add(2)  add(5)  add(4)

getProduct(2) → 5 * 4         = 20
getProduct(3) → 2 * 5 * 4     = 40
getProduct(4) → 0 * 2 * 5 * 4 = 0

add(8)

getProduct(2) → 4 * 8         = 32
```

```text
0 <= num <= 100
1 <= k <= 4 * 10^4
at most 4 * 10^4 calls in total
every product fits in a 32-bit integer
```

A `0` in the last `k` numbers makes the whole product `0`. Every other product is a ratio of two running products.

---

# Approach — Prefix Products That Reset on Zero

### Core idea

Keep a running product, the same way a prefix sum is kept.

```text
list = [1, p1, p1·p2, p1·p2·p3, ...]
```

The product of the last `k` numbers is the newest prefix divided by the prefix that sits `k` places earlier:

```text
product of last k = list[last] / list[last - k]
```

Division is safe here because every stored prefix is non-zero.

A zero breaks that. Once `0` enters a running product, every later prefix is `0`, and `0 / 0` has no meaning. The fix is to **throw the list away on a zero**:

```text
add(0)  →  clear the list and start again from [1]
```

Everything before the zero can only matter to a query that reaches back across it, and that query is always `0`. The list then holds only the numbers added **after** the latest zero.

That also answers the zero case. After a reset, the list has `count + 1` entries, where `count` is the number of non-zero numbers since the last zero.

```text
k >= list.size()   →  k > count   →  the last k numbers include the zero   →  0
```

The sentinel `1` at index `0` is what lets `k == count` work: the divisor is `list[0] = 1`, and the answer is the whole running product.

### Walkthrough

```text
start          list = [1]

add(3)         list = [1, 3]
add(0)         list = [1]                 reset on zero
add(2)         list = [1, 2]
add(5)         list = [1, 2, 10]
add(4)         list = [1, 2, 10, 40]

getProduct(2)  n = 4, k = 2 < 4
               list[3] / list[4 - 1 - 2] = 40 / list[1] = 40 / 2 = 20

getProduct(3)  list[3] / list[0] = 40 / 1 = 40

getProduct(4)  k = 4 >= n = 4  →  0

add(8)         list = [1, 2, 10, 40, 320]

getProduct(2)  n = 5, list[4] / list[5 - 1 - 2] = 320 / list[2] = 320 / 10 = 32
```

### State

```text
nums : prefix products of the numbers since the last zero, starting with a sentinel 1
```

### Transitions

```text
add(0)       clear the list, then append 1
add(num)     append  lastPrefix * num

getProduct(k)
    n = list size
    if k >= n          return 0
    return list[n - 1] / list[n - 1 - k]
```

### Why integer division is exact

`list[n - 1]` equals `list[n - 1 - k]` multiplied by the last `k` numbers. Dividing by `list[n - 1 - k]` therefore returns exactly that product with no remainder.

The statement guarantees every product fits in a 32-bit integer, so each stored prefix stays in `int` range as well.

---

## Java

```java
class ProductOfNumbers {

    // Prefix products since the last zero, with a sentinel 1 at index 0.
    List<Integer> nums;

    public ProductOfNumbers() {
        nums = new ArrayList<>();
        nums.add(1);
    }

    public void add(int num) {

        if (num == 0) {
            // Any query that reaches this zero is 0, so older prefixes are no longer needed.
            nums.clear();
            nums.add(1);
        } else {
            int last = nums.get(nums.size() - 1);
            nums.add(last * num);
        }
    }

    public int getProduct(int k) {

        int n = nums.size();

        // Fewer than k non-zero numbers since the last zero → the window holds the zero.
        if (k >= n) {
            return 0;
        }

        return nums.get(n - 1) / nums.get(n - 1 - k);
    }
}
```

---

## C++

```cpp
class ProductOfNumbers {
public:

    // Prefix products since the last zero, with a sentinel 1 at index 0.
    vector<int> nums;

    ProductOfNumbers() {
        nums.push_back(1);
    }

    void add(int num) {

        if (num == 0) {
            // Any query that reaches this zero is 0, so older prefixes are no longer needed.
            nums.clear();
            nums.push_back(1);
        } else {
            nums.push_back(nums.back() * num);
        }
    }

    int getProduct(int k) {

        int n = nums.size();

        // Fewer than k non-zero numbers since the last zero → the window holds the zero.
        if (k >= n) {
            return 0;
        }

        return nums[n - 1] / nums[n - 1 - k];
    }
};
```

---

# Complexity

Let `m` be the total number of `add` calls.

```text
add            : O(1) amortized   (clear is O(size) once, and each element is cleared at most once)
getProduct     : O(1)
Space          : O(m)             the prefix list holds at most m + 1 values
```

---

# Pattern Recognition

```text
Product of the last k numbers in a stream
        ↓
Store prefix products, answer with one division
        ↓
A zero makes later prefixes useless for division
        ↓
Reset the list on a zero, and answer 0 when k reaches back across it
```

### Remember this trigger:

> **Product over a recent window with zeros → keep prefix products starting from a sentinel `1`, clear them on every `0`, return `0` when `k >= size`, otherwise `last / prefix[size - 1 - k]`.**
