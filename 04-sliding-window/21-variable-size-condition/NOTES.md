# Variable-Size Condition (Sliding Window)

## When to Recognize It

You see this pattern when:
- The window size is **NOT given** — you need to find it
- The problem asks for the **longest** or **smallest** contiguous subarray/substring satisfying some condition
- Brute force = check every subarray = O(n²) or worse

**Clue words:** "longest substring with...", "smallest subarray with...", "at most K...", "no more than...", "containing all..."

---

## ★ The Two Flavors ★

This is the most important distinction in variable-size sliding window:

### Flavor 1 — Longest window satisfying X

> **Expand greedily. Shrink only when invariant breaks. Record after shrinking.**

Examples: longest substring without duplicates, longest with at most K zeros, longest with at most K distinct.

### Flavor 2 — Smallest window satisfying Y

> **Expand until valid. Then shrink while still valid (find the tightest fit). Record inside the shrink loop.**

Examples: smallest subarray with sum ≥ target, smallest substring containing all chars of a pattern.

### Why the answer-recording difference matters

| Flavor | Record answer WHEN | WHY |
|--------|--------------------|-----|
| **Longest** | After shrink loop (window now valid) | Only valid windows count; you record the size after restoring validity |
| **Smallest** | Inside shrink loop (each step still valid) | Each successful shrink might be the tightest fit; you want to capture every valid-but-smaller window |

---

## Core Intuition

Two pointers `i` and `j`, both moving forward, never backward. The window between them **grows and contracts** based on a condition (the **invariant**).

### Analogy: A trombone slide

A trombone player moves their slide outward to lengthen a note, then pulls it back to shorten it — both ends move independently in the same direction. That's variable-size sliding window.

- `j` (right end) **always extends** the window each iteration.
- `i` (left end) **may contract** the window when the condition demands.

---

## Canonical Templates

### Flavor 1 — Longest Window Satisfying X

```java
int i = 0;
int maxLen = 0;

for (int j = 0; j < n; j++) {
    // 1. Expand: add nums[j] to window state

    // 2. Shrink while invariant is broken
    while (/* invariant violated */) {
        // remove nums[i] from window state
        i++;
    }

    // 3. Record (window is now valid)
    maxLen = Math.max(maxLen, j - i + 1);
}
return maxLen;
```

### Flavor 2 — Smallest Window Satisfying Y

```java
int i = 0;
int minLen = Integer.MAX_VALUE;

for (int j = 0; j < n; j++) {
    // 1. Expand: add nums[j] to window state

    // 2. Shrink while STILL valid (tighten the window)
    while (/* invariant satisfied */) {
        minLen = Math.min(minLen, j - i + 1);   // record BEFORE shrinking
        // remove nums[i] from window state
        i++;
    }
}
return minLen == Integer.MAX_VALUE ? 0 : minLen;
```

### Side-by-side comparison

| | Flavor 1 (Longest) | Flavor 2 (Smallest) |
|---|---|---|
| `while` condition | `invariant violated` | `invariant satisfied` |
| Record location | **After** the while loop | **Inside** the while loop |
| Goal of shrinking | Restore validity | Find tightest valid window |
| Initial answer | `0` | `Integer.MAX_VALUE` (then convert) |

---

## Problems Solved

### 1. Longest Substring Without Repeating Characters (LC 3) — Flavor 1

- **Mental Model:** Walk forward with `j`. The window is the current "no-duplicates" stretch. When a duplicate enters, shrink from the left until it's gone, then continue.
- **Recognize it when:** longest substring with the "all-unique" property
- **Window state:** `HashSet<Character>` of chars currently in window
- **Invariant:** no duplicates (no char appears twice)
- **Key takeaway:** When duplicate enters, shrink past all instances of it. Use `hs.contains(...)` as the violation check.
- **Test cases:**
  - `s = "abcabcbb"` → `3` (`"abc"`)
  - `s = "bbbbb"` → `1`
  - `s = "pwwkew"` → `3` (`"wke"`)

```java
public int lengthOfLongestSubstring(String s) {
    HashSet<Character> hs = new HashSet<>();
    int maxLen = 0, i = 0;

    for (int j = 0; j < s.length(); j++) {
        while (hs.contains(s.charAt(j))) {
            hs.remove(s.charAt(i));
            i++;
        }
        hs.add(s.charAt(j));
        maxLen = Math.max(maxLen, hs.size());
    }
    return maxLen;
}
```

### 2. Minimum Size Subarray Sum (LC 209) — Flavor 2

- **Mental Model:** Expand until sum ≥ target. Then shrink as much as possible while still ≥ target — each shrink might give a tighter answer. Continue.
- **Recognize it when:** smallest subarray with sum at least X
- **Window state:** running sum
- **Invariant:** `sum >= target`
- **Key takeaway:** This is the canonical "smallest with Y" pattern. Record `minLen` **before** each shrink because shrinking might invalidate the window in the next step.
- **Test cases:**
  - `target = 7, nums = [2,3,1,2,4,3]` → `2` (subarray `[4,3]`)
  - `target = 11, nums = [1,1,1,1,1,1,1,1]` → `0` (impossible)

```java
public int minSubArrayLen(int target, int[] nums) {
    int sum = 0, minLen = Integer.MAX_VALUE, i = 0;

    for (int j = 0; j < nums.length; j++) {
        sum += nums[j];
        while (sum >= target) {
            minLen = Math.min(minLen, j - i + 1);
            sum -= nums[i];
            i++;
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
```

### 3. Max Consecutive Ones III (LC 1004) — Flavor 1

- **Mental Model:** Window of 0s and 1s where you can "flip" up to K zeros. Reframe as: longest window with at most K zeros. When zero count exceeds K, shrink.
- **Recognize it when:** "max consecutive after at most K modifications" / "longest with at most K bad elements"
- **Window state:** count of 0s in window
- **Invariant:** `zeros <= k`
- **Key takeaway:** Don't mutate `k` (the parameter). Track an explicit `zeros` counter — clearer and safer.
- **Test cases:**
  - `nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2` → `6`
  - `nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k = 3` → `10`

```java
public int longestOnes(int[] nums, int k) {
    int zeros = 0, maxLen = 0, i = 0;

    for (int j = 0; j < nums.length; j++) {
        if (nums[j] == 0) zeros++;

        while (zeros > k) {
            if (nums[i] == 0) zeros--;
            i++;
        }
        maxLen = Math.max(maxLen, j - i + 1);
    }
    return maxLen;
}
```

### 4. Fruit Into Baskets (LC 904) — Flavor 1

- **Mental Model:** You walk along a row of trees with 2 baskets, each fitting one fruit type. Reframe: longest contiguous subarray with at most 2 distinct elements.
- **Recognize it when:** "at most K distinct" — generalize to K with the same template
- **Window state:** `HashMap<fruit, count>` of distinct types and their counts
- **Invariant:** `map.size() <= 2`
- **Key takeaway:** When shrinking, if a fruit's count drops to 0, **remove the key from the map** — otherwise `map.size()` stays high and the invariant check is wrong.
- **Test cases:**
  - `fruits = [1,2,1]` → `3`
  - `fruits = [0,1,2,2]` → `3`
  - `fruits = [1,2,3,2,2]` → `4`
  - `fruits = [3,3,3,1,2,1,1,2,3,3,4]` → `5`

```java
public int totalFruit(int[] fruits) {
    HashMap<Integer, Integer> map = new HashMap<>();
    int maxLen = 0, i = 0;

    for (int j = 0; j < fruits.length; j++) {
        map.put(fruits[j], map.getOrDefault(fruits[j], 0) + 1);

        while (map.size() > 2) {
            map.put(fruits[i], map.get(fruits[i]) - 1);
            if (map.get(fruits[i]) == 0) map.remove(fruits[i]);
            i++;
        }
        maxLen = Math.max(maxLen, j - i + 1);
    }
    return maxLen;
}
```

---

### Summary Table

| # | Problem | Flavor | Window State | Invariant |
|---|---------|--------|---------------|-----------|
| 1 | Longest Substring Without Repeating (LC 3) | Longest | `HashSet<Char>` | no duplicates |
| 2 | Minimum Size Subarray Sum (LC 209) | **Smallest** | running sum | `sum >= target` |
| 3 | Max Consecutive Ones III (LC 1004) | Longest | zero count | `zeros <= k` |
| 4 | Fruit Into Baskets (LC 904) | Longest | `HashMap<Int, Count>` | `map.size() <= 2` |

---

## Common Pitfalls

- **Mixing up the two flavors** — using `while (invariant violated)` for a "smallest" problem will only shrink to validity, never tighten further. Use `while (invariant satisfied)` for "smallest" to keep shrinking.
- **Recording the answer in the wrong place** — for "smallest," record **inside** the shrink loop; for "longest," record **after**.
- **Forgetting to remove keys from the map when count hits 0** — `map.size()` stays inflated, and the invariant check `size > K` becomes wrong.
- **Mutating the input parameter (like `k`)** — works but loses the original. Track an explicit window-state counter instead.
- **Using `if` instead of `while` for shrink** — `if` shrinks only once per outer iteration. Some invariants need multiple shrinks before the window is restored.
- **Wrong initial value for `minLen`** — use `Integer.MAX_VALUE`, then convert to `0` (or whatever the problem says) if the loop never finds a valid window.
- **Forgetting to `++i` after a shrink step** — infinite loop.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force (check every subarray) | O(n²) or O(n³) | O(1) |
| **Variable-size sliding window** | **O(n)** | O(window-state-size) — typically O(1) or O(distinct chars) |

Why O(n)? Each pointer (`i` and `j`) moves at most n times. Total pointer movements ≤ 2n.

---

## Mental Checklist When You See a Problem

1. Am I asked for **longest** or **smallest** (or "is there any") contiguous subarray/substring?
2. Is there a **condition** that the window must satisfy?
3. Can I describe the **window state** (a set, a map, a counter, a sum)?
4. Can I describe the **invariant** (what makes the window "valid")?

If YES to all → variable-size sliding window. Then:
- Longest → Flavor 1 template
- Smallest → Flavor 2 template

---

## The Pattern in One Sentence

> Two pointers, both moving forward. Right pointer expands, left pointer shrinks based on the invariant. The window's job is to maintain a property; the answer comes from tracking the window's size at the right moment.