# LeetCode 2409 — Count Days Spent Together

## Problem

Alice stays from `arriveAlice` through `leaveAlice`. Bob stays from `arriveBob` through `leaveBob`.

Every date is a string `MM-DD` in a **non-leap** year, so February has 28 days. Both endpoints are inclusive.

Return how many days they are both present. If their stays do not overlap, return `0`.

Example:

```text
Alice: 08-15 through 08-18
Bob:   08-16 through 08-19

shared days: 08-16, 08-17, 08-18
answer = 3
```

```text
Alice: 10-01 through 10-31
Bob:   11-01 through 12-31

answer = 0
```

---

# Approach — Map Each Date onto One Number Line

### Core idea

Comparing month-day strings directly is awkward. Turn each date into its day number in the year:

```text
day number = days in every earlier month + day of this month
```

The month lengths are fixed:

```text
31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31
```

January 1 is day `1`. August 15 is day `227`, because the first seven months contain `212` days and August 15 adds `15`.

Each person is then one closed interval:

```text
Alice: [aliceStart, aliceEnd]
Bob:   [bobStart, bobEnd]
```

The overlap of two closed intervals is:

```text
start = max(aliceStart, bobStart)
end   = min(aliceEnd, bobEnd)
```

If `end < start`, the intervals miss each other and the answer is `0`. Otherwise the inclusive length is:

```text
end - start + 1
```

`Math.max(0, end - start + 1)` covers both situations.

### Walkthrough

```text
Alice: 08-15 → 227,  08-18 → 230
Bob:   08-16 → 228,  08-19 → 231

start = max(227, 228) = 228
end   = min(230, 231) = 230
days  = 230 - 228 + 1 = 3
```

```text
Alice ends on day 304
Bob starts on day 305

end - start + 1 = 304 - 305 + 1 = 0
```

### State

```text
monthDays : fixed length of each month
start     : later of the two arrivals
end       : earlier of the two departures
```

### Transitions

```text
MM-DD → day of year
overlap start = max of the two starts
overlap end   = min of the two ends
answer        = max(0, end - start + 1)
```

---

## Java

```java
class Solution {
    public int countDaysTogether(String arriveAlice, String leaveAlice,
                                 String arriveBob, String leaveBob) {

        int[] monthDays = {31, 28, 31, 30, 31, 30,
                           31, 31, 30, 31, 30, 31};

        int aliceStart = yearLineMapping(arriveAlice, monthDays);
        int aliceEnd = yearLineMapping(leaveAlice, monthDays);
        int bobStart = yearLineMapping(arriveBob, monthDays);
        int bobEnd = yearLineMapping(leaveBob, monthDays);

        // Later arrival through earlier departure.
        int start = Math.max(aliceStart, bobStart);
        int end = Math.min(aliceEnd, bobEnd);

        return Math.max(0, end - start + 1);
    }

    private int yearLineMapping(String date, int[] monthDays) {
        int month = Integer.parseInt(date.substring(0, 2));
        int day = Integer.parseInt(date.substring(3));

        int total = 0;

        // Add every complete month before this one.
        for (int i = 0; i < month - 1; i++) {
            total += monthDays[i];
        }

        total += day;

        return total;
    }
}
```

---

## C++

```cpp
class Solution {
public:

    int countDaysTogether(string arriveAlice, string leaveAlice,
                          string arriveBob, string leaveBob) {

        int monthDays[12] = {31, 28, 31, 30, 31, 30,
                             31, 31, 30, 31, 30, 31};

        int aliceStart = yearLineMapping(arriveAlice, monthDays);
        int aliceEnd = yearLineMapping(leaveAlice, monthDays);
        int bobStart = yearLineMapping(arriveBob, monthDays);
        int bobEnd = yearLineMapping(leaveBob, monthDays);

        // Later arrival through earlier departure.
        int start = max(aliceStart, bobStart);
        int end = min(aliceEnd, bobEnd);

        return max(0, end - start + 1);
    }

private:

    int yearLineMapping(const string& date, const int monthDays[]) {
        int month = stoi(date.substr(0, 2));
        int day = stoi(date.substr(3));

        int total = 0;

        // Add every complete month before this one.
        for (int i = 0; i < month - 1; i++) {
            total += monthDays[i];
        }

        total += day;

        return total;
    }
};
```

---

# Complexity

There are only 12 months, and each date string has length 5.

```text
Time  : O(1)    four dates, each adding at most 11 month lengths
Space : O(1)    the month table and four day numbers
```

---

# Pattern Recognition

```text
Inclusive date ranges
        ↓
Convert MM-DD to a day-of-year number
        ↓
Overlap of [a, b] and [c, d]
        ↓
max(0, min(b, d) - max(a, c) + 1)
```

### Remember this trigger:

> **Days shared by two inclusive stays → put both ranges on one number line. The overlap starts at the later arrival and ends at the earlier departure.**
