# Find First/Last Occurrence (Boundary Binary Search)

## ★ The Headline Insight ★

> **Binary search isn't always "find this exact thing."**
> **It's often "find the BOUNDARY between two zones."**

That mental shift is the real skill in this sub-pattern. If you remember nothing else, remember this.

---

## When to Recognize It

You see this pattern when the question is **not** "is this element present?" but instead:

- First occurrence / last occurrence of a target (with duplicates)
- First element **≥ target** (lower_bound)
- First element **> target** (upper_bound — "strictly greater")
- Last element **≤ target** (floor)
- Last element **< target**
- "First version that is bad", "first day capacity is enough", etc.

**Clue words:** "first", "last", "smallest greater than", "largest less than or equal", "leftmost", "rightmost", "smallest version where..."

---

## The Two-Zones Mental Model

The sorted array can always be split into **two zones** based on a yes/no condition:

```
[ FALSE  FALSE  FALSE  FALSE | TRUE  TRUE  TRUE  TRUE ]
                              ^
                         boundary
```

The condition is something like "is this element ≥ target?" or "is this version bad?" — anything where everything on the right satisfies it and everything on the left does not (or vice versa).

**Your answer is the boundary index** — usually the first TRUE index, or the last FALSE index, depending on the problem.

### The trick

Each iteration of binary search:
- If `arr[mid]` is in the LEFT zone (FALSE) → boundary is to the right → `start = mid + 1`
- If `arr[mid]` is in the RIGHT zone (TRUE) → boundary is here or to the left → `end = mid - 1`

When the loop exits, `start` has settled **exactly at the boundary** — the first index of the RIGHT zone.

---

## Worked Example

`letters = ['c', 'c', 'f', 'f', 'j'], target = 'c'` — find smallest letter strictly greater than `'c'`.

The two zones based on "is `arr[i] > 'c'`?":

```
[c, c | f, f, j]
 ↑ ↑    ↑  ↑  ↑
 F F    T  T  T
       boundary = index 2
```

The condition we use is `arr[mid] <= target`:
- LEFT zone: `arr[i] <= target` → keep moving past
- RIGHT zone: `arr[i] > target` → answer candidate

| Step | start | end | mid | arr[mid] | ≤ 'c'? | Action |
|:----:|:-----:|:---:|:---:|:--------:|:------:|--------|
| 1 | 0 | 4 | 2 | 'f' | No (RIGHT) | `end = 1` |
| 2 | 0 | 1 | 0 | 'c' | Yes (LEFT) | `start = 1` |
| 3 | 1 | 1 | 1 | 'c' | Yes (LEFT) | `start = 2` |
| 4 | 2 | 1 | — | — | — | exit |

`start = 2` = boundary index. Answer: `arr[2] = 'f'` ✓

---

## The Critical Rule: "Don't Stop on a Match"

Plain binary search returns immediately when `arr[mid] == target`. Boundary binary search **does not** — it records `mid` and keeps searching toward the boundary.

### Find FIRST occurrence

On match:
```
res = mid
end = mid - 1   // keep looking LEFT — there might be an earlier match
```

### Find LAST occurrence

On match:
```
res = mid
start = mid + 1   // keep looking RIGHT — there might be a later match
```

### Find strictly greater (upper_bound)

Treat equality as "not yet greater enough" — push it into the left-half move:
```
if arr[mid] <= target: start = mid + 1     // includes equality
else: end = mid - 1
```

After the loop, `start` is the first index where `arr[i] > target`.

---

## Templates

### Template A — Find First / Last Occurrence (record-and-narrow)

```java
private static int findFirst(int[] nums, int target) {
    int start = 0, end = nums.length - 1, res = -1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (nums[mid] == target) {
            res = mid;
            end = mid - 1;          // keep looking LEFT
        } else if (nums[mid] < target) {
            start = mid + 1;
        } else {
            end = mid - 1;
        }
    }
    return res;
}

private static int findLast(int[] nums, int target) {
    int start = 0, end = nums.length - 1, res = -1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (nums[mid] == target) {
            res = mid;
            start = mid + 1;        // keep looking RIGHT
        } else if (nums[mid] < target) {
            start = mid + 1;
        } else {
            end = mid - 1;
        }
    }
    return res;
}
```

The two helpers are **identical except for one line** — what to do on a match. That single difference flips first → last.

### Template B — Boundary on a Boolean Condition (cleaner for "first that satisfies X")

When you can phrase the question as "find the first index where P(i) is true," use a 2-branch template:

```java
int start = 0, end = n - 1;
while (start <= end) {
    int mid = start + (end - start) / 2;
    if (P(mid)) {
        end = mid - 1;     // mid satisfies — but there might be an earlier one
    } else {
        start = mid + 1;   // mid doesn't satisfy — answer is to the right
    }
}
return start;              // start is the boundary (first index satisfying P)
```

**Examples of `P(mid)`:**
- "is `arr[mid] >= target`?" → finds lower_bound
- "is `arr[mid] > target`?" → finds upper_bound
- "is `isBadVersion(mid)`?" → finds first bad version

After the loop, `start` is the boundary. If `start == n`, no element satisfies P (handle as needed — return `-1`, wrap with `% n`, etc.).

---

## Common Pitfalls

- **Stopping on the first match** — plain binary search returns immediately on `==`. Boundary search **doesn't**. Record the index and keep narrowing.
- **Wrong direction after recording the match** — to find FIRST, after recording you must go LEFT (`end = mid - 1`). To find LAST, you must go RIGHT. Easy to mix up.
- **Linear scan after the binary search finds any match** — defeats the O(log n) guarantee. With duplicate values, two biased searches are O(log n) total; expanding from a hit is worst-case O(n).
- **Modulo trickery inside the loop** — for wraparound problems (LC 744), keep `end = n - 1` and **only apply `% n` at the return**. Wrap-modulo inside comparisons reads wrong indices.
- **Forgetting the `start == n` edge case in Template B** — when no element satisfies P, `start` falls off the right end. Handle it explicitly.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Linear scan for first/last | O(n) | O(1) |
| Plain binary search + expand outward | O(n) worst case (defeats the point) | O(1) |
| **Two biased binary searches** | **O(log n)** | **O(1)** |

The biased technique stays log-time even when duplicates fill the array.

---

## Problems Solved

### 1. Find First and Last Position (LC 34)

- **Mental Model:** Two zones — "less than target" vs "≥ target" (for first), or "≤ target" vs "greater than target" (for last). Run boundary search twice, once biased left, once biased right. Equality is a "record and keep going" signal, not a stop.
- **Recognize it when:** sorted array with duplicates; need first AND last index of a value
- **Key takeaway:** Same skeleton, run twice — the only line that differs between the two is the on-match direction (`end = mid - 1` for first, `start = mid + 1` for last).
- **Test cases:**
  - `nums = [5,7,7,8,8,10], target = 8` → `[3, 4]`
  - `nums = [5,7,7,8,8,10], target = 6` → `[-1, -1]`
  - `nums = [], target = 0` → `[-1, -1]`

### 2. First Bad Version (LC 278)

- **Mental Model:** Two zones — good versions on the left, bad versions on the right. Boundary = first bad version. The "array" is the abstract range `1..n`; the comparison is an API call.
- **Recognize it when:** monotonic boolean condition — "all true after some boundary point"
- **Key takeaway:** Template B applied verbatim with `P(mid) = isBadVersion(mid)`. Use `start + (end - start) / 2` to avoid overflow (n can be near `Integer.MAX_VALUE`).
- **Test case:**
  - `n = 5, first bad = 4` → `4` (versions 4, 5 are bad)

### 3. Find Smallest Letter Greater Than Target (LC 744)

- **Mental Model:** Two zones — letters ≤ target (LEFT) vs letters strictly > target (RIGHT). Boundary = answer. Wraparound when the RIGHT zone is empty (all letters ≤ target).
- **Recognize it when:** "strictly greater than" / "upper_bound" / "ceiling" with possible wraparound
- **Key takeaway:** Treat equality as LEFT zone (`<= target` → `start = mid + 1`). After the loop, return `arr[start % n]` — the modulo handles the wraparound edge case where `start` falls off the right end.
- **Test cases:**
  - `['c','f','j'], 'a'` → `'c'`
  - `['c','f','j'], 'c'` → `'f'` (strictly greater, not equal)
  - `['c','f','j'], 'j'` → `'c'` (wraparound)
  - `['x','x','y','y'], 'z'` → `'x'` (wraparound when no letter is >)

---

### To revisit later

- **LC 658 — Find K Closest Elements** — combines boundary binary search with expand-from-center two pointers. Genuinely a combo problem — comes back to this after more solo practice on boundary problems alone.

---

### Summary Table

| # | Problem | Boundary You're Finding | One-line Mental Model |
|---|---------|--------------------------|-----------------------|
| 1 | First & Last Position (LC 34) | First and last index of target | Two biased searches; on match, record and keep narrowing in one direction |
| 2 | First Bad Version (LC 278) | First true index of `isBadVersion` | Find the boundary between good and bad |
| 3 | Smallest Letter > Target (LC 744) | First index where letter > target | Find the boundary; wrap modulo if no such letter |

---

## Mental Checklist When You See a Problem

1. Is the data sorted (or monotonic in some property)?
2. Is the question phrased with words like "**first**", "**last**", "**smallest greater than**", "**largest less than**", "**leftmost / rightmost**"?
3. Can I phrase the answer as "the boundary index between two zones"?

If YES → boundary binary search. Pick:
- Template A if equality matters (record-and-narrow with a 3-branch comparison)
- Template B if the question is naturally "first index satisfying condition P" (cleaner 2-branch)

---

## The Pattern in One Sentence

> The array splits into two monotonic zones; binary search finds the boundary between them. On a match, don't stop — record and keep narrowing in the direction of the boundary you want.