# Binary Search on Sorted Array (Classic)

## When to Recognize It

You see this pattern when:
- The data is **sorted** (or monotonic in some property)
- You're searching for a target, an insertion point, or a value satisfying a condition
- A linear O(n) scan is too slow; you want **O(log n)**

**Clue words:** "sorted array", "find target", "insert position", "search in O(log n)", a guessing game over a numeric range

---

## Core Intuition

The array is sorted, so comparing the **middle** element to the target tells you which **half** to discard. Repeat, halving the search space each time → O(log n).

### Analogy: Dictionary lookup

You don't read a dictionary page by page. You flip to the middle, see if your word comes before or after, and throw away half the book. Repeat until you land on the word.

---

## The Canonical Template (memorize this exact shape)

```java
int start = 0, end = n - 1;
while (start <= end) {
    int mid = start + (end - start) / 2;   // overflow-safe
    if (nums[mid] == target) {
        return mid;
    } else if (nums[mid] < target) {
        start = mid + 1;    // target is in the right half
    } else {
        end = mid - 1;      // target is in the left half
    }
}
return -1;   // not found
```

---

## The Three Decisions (get these consistent or it breaks)

Binary search bugs almost always come from mixing these up. The template above uses the **"exclusive" style** — once you check `mid`, you exclude it from the next window.

### 1. Loop condition: `start <= end`

Because the updates exclude `mid` (`mid + 1` / `mid - 1`), the window can shrink to a single element where `start == end`. You still need to check it, so the condition is `<=`, not `<`.

**Pairing rule:**

| Update style | Loop condition |
|--------------|----------------|
| `end = mid - 1`, `start = mid + 1` (exclusive) | `start <= end` |
| `end = mid` (inclusive) | `start < end` |

Mixing an exclusive update with a `<` condition skips the last element. Mixing inclusive update with `<=` causes an infinite loop.

### 2. Mid: `start + (end - start) / 2`

NOT `(start + end) / 2`. For very large arrays, `start + end` can exceed `Integer.MAX_VALUE` and overflow to a negative number. The `start + (end - start) / 2` form is mathematically identical but never overflows. Interviewers notice this.

### 3. Updates: `mid + 1` / `mid - 1`

Since you've already compared `mid` and it wasn't the answer, exclude it from the next window. This guarantees the window strictly shrinks each iteration — no infinite loops.

---

## Variants

### LC 704 — Find Target

Pure template. Return `mid` on match, `-1` if the loop exits.

### LC 35 — Search Insert Position

Same search, but `return start` instead of `-1`.

**Why `start` is the insertion point:** when the loop exits, `start > end`. The invariant is:
- Everything left of `start` is `< target` (we advanced `start` past them)
- Everything at/right of `start` is `> target` (we pulled `end` below them)

So `start` is the first index where `nums[i] >= target` — exactly where `target` should go.

```java
// identical loop, only the final return changes:
return start;
```

### LC 374 — Guess Number (search over a range, not an array)

No array — the search space is `1..n`. Instead of comparing `nums[mid]` to a target, you call an API `guess(mid)`:
- `0` → correct, return `mid`
- `1` → pick is higher → `start = mid + 1`
- `-1` → pick is lower → `end = mid - 1`

**Efficiency note:** cache the API result in a variable — don't call `guess(mid)` twice in one iteration:

```java
int res = guess(mid);
if (res == 0) return mid;
else if (res == 1) start = mid + 1;
else end = mid - 1;
```

This generalizes binary search: the "thing you compare against" doesn't have to be an array element — it can be any monotonic signal.

---

## Common Pitfalls

- **`start < end` with exclusive updates** — skips the final single-element window. Use `<=`.
- **`(start + end) / 2`** — integer overflow on large inputs. Use `start + (end - start) / 2`.
- **`start = mid` or `end = mid` with `<=` condition** — infinite loop (window never shrinks past `mid`). Exclusive style must use `mid ± 1`.
- **Calling an expensive comparison/API twice per iteration** — cache the result.
- **Forgetting the array must be sorted** — binary search is meaningless on unsorted data.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Linear scan | O(n) | O(1) |
| **Binary search** | **O(log n)** | **O(1)** |

Each step halves the search space: `n → n/2 → n/4 → ... → 1`, which is `log₂(n)` steps.

---

## Problems Solved

### 1. Binary Search (LC 704)

- **Mental Model:** Dictionary lookup — open to the middle, discard the wrong half, repeat.
- **Recognize it when:** find an exact target in a sorted array
- **Key takeaway:** The canonical template. Three consistent decisions: `start <= end`, overflow-safe mid, `mid ± 1` updates.
- **Test cases:**
  - `nums = [-1,0,3,5,9,12], target = 9` → `4`
  - `nums = [-1,0,3,5,9,12], target = 2` → `-1`

### 2. Search Insert Position (LC 35)

- **Mental Model:** Same search; when not found, `start` has settled exactly where the target belongs.
- **Recognize it when:** find target OR where it would be inserted
- **Key takeaway:** `return start` instead of `-1`. When the loop exits, `start` is the first index with `nums[i] >= target`.
- **Test cases:**
  - `nums = [1,3,5,6], target = 2` → `1`
  - `nums = [1,3,5,6], target = 7` → `4`
  - `nums = [1,3,5,6], target = 0` → `0`

### 3. Guess Number Higher or Lower (LC 374)

- **Mental Model:** Binary search over the range `1..n`, where an API call (not an array) tells you which way to go.
- **Recognize it when:** narrowing a numeric range using a monotonic yes/no (or higher/lower) signal
- **Key takeaway:** The "array" can be an abstract range; the comparison can be an API. Cache the API result to avoid duplicate calls.
- **Test cases:**
  - `n = 10, pick = 6` → `6`
  - `n = 1, pick = 1` → `1`

---

### Summary Table

| # | Problem | What changes from template | One-line Mental Model |
|---|---------|----------------------------|-----------------------|
| 1 | Binary Search (LC 704) | nothing — pure template | Open to middle, toss the wrong half |
| 2 | Search Insert Position (LC 35) | `return start` not `-1` | Where the search settles is where it belongs |
| 3 | Guess Number (LC 374) | compare via API, not array | Halve a range using a higher/lower signal |

---

## Mental Checklist When You See a Problem

1. Is the data **sorted** (or monotonic in the property I care about)?
2. Am I searching for a target, a boundary, or an insertion point?
3. Can I, from the middle element, decide which **half** to discard?

If YES → binary search. Apply the canonical template; change only the final return or the comparison source.

---

## The Pattern in One Sentence

> On sorted data, compare the middle to your target and discard the half that can't contain the answer — halving the search space each step for O(log n).