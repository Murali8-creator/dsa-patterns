# In-Place Array Modification (Read + Write Pointer)

## When to Recognize It

You see this pattern when:
- You need to **modify an array in-place** (no extra array allocation)
- You're **removing, filtering, or rearranging** elements based on a condition
- The problem says "do it in-place" or "return the new length, mutate the array"

**Clue words:** "remove in-place", "return the new length", "move X to the end", "remove duplicates", "keep at most K copies"

---

## Core Intuition

Use **two pointers moving in the same direction**:
- **Read pointer (`j`)** — scans every element, always moves forward
- **Write pointer (`i`)** — tracks where the next "kept" element goes, only advances when we keep something

`j` reads, `i` writes. `i ≤ j` always. At the end, indices `0..i-1` contain the final result.

### Analogy: Sorting Papers In-Place

You have a stack of papers you're sorting through. In your right hand, one paper at a time (the read pointer). Next to you is a "keepers" pile that grows as you go (the write pointer).

For each paper:
- Does it pass your filter? Put it on the keepers pile (advance write pointer).
- Doesn't pass? Drop it, move on (only advance read pointer).

When you finish, the keepers pile is the cleaned result — in the same box you started with.

---

## Difference From Converging Two Pointers

| | Converging (Template 1) | In-Place Modification |
|---|---|---|
| Start positions | Opposite ends (0 and n-1) | Both at 0 |
| Direction | Toward each other | Same direction (forward) |
| Typical use | Search pairs in sorted arrays | Filter/compact an array |

Same "two pointers" family, different movement style.

---

## Canonical Template

```java
int i = 0;                          // write pointer
for (int j = 0; j < n; j++) {       // read pointer — always moves
    if (/* nums[j] passes filter */) {
        nums[i] = nums[j];          // keep it: write at i
        i++;                        // advance write pointer
    }
    // else: skip, i stays put, only j advances
}
return i;                           // new length
```

Three knobs to tune per problem:
1. **What "passes filter" means** (the `if` condition)
2. **What you write** (usually `nums[j]`, sometimes a modified value)
3. **What you return** (new length, void, the array, etc.)

---

## Variants

### 1. Remove by Value (LC 27)

Filter: `nums[j] != val`

```java
int i = 0;
for (int j = 0; j < n; j++) {
    if (nums[j] != val) {
        nums[i++] = nums[j];
    }
}
return i;
```

### 2. Remove Duplicates, Keep 1 (LC 26)

Filter: `i == 0 || nums[j] != nums[i - 1]` (different from last kept)

```java
int i = 0;
for (int j = 0; j < n; j++) {
    if (i == 0 || nums[j] != nums[i - 1]) {
        nums[i++] = nums[j];
    }
}
return i;
```

**Why it works:** the array is **sorted**, so duplicates sit next to each other. Comparing with `nums[i-1]` (the last kept value) is enough.

### 3. Remove Duplicates, Keep At Most K (LC 80 for K=2)

Filter: `i < K || nums[j] != nums[i - K]` (different from K-back)

```java
int i = 0, K = 2;
for (int j = 0; j < n; j++) {
    if (i < K || nums[j] != nums[i - K]) {
        nums[i++] = nums[j];
    }
}
return i;
```

**The `i - K` trick:** if `nums[j]` equals the element K positions back from the write pointer, it means you already have K copies of that value in the kept portion (because the array is sorted — K consecutive equal values). Skip.

**Generalizes:** change K to any value to "keep at most K copies."

### 4. Move Zeros to End (LC 283) — Two-pass version

Two passes:
1. First pass: copy non-zeros forward using the template.
2. Second pass: fill remaining positions with 0.

```java
int i = 0;
for (int j = 0; j < n; j++) {
    if (nums[j] != 0) {
        nums[i++] = nums[j];
    }
}
for (int k = i; k < n; k++) {
    nums[k] = 0;
}
```

### 5. Move Zeros to End (LC 283) — One-pass version (swap)

Instead of copying, **swap** when you keep:

```java
int i = 0;
for (int j = 0; j < n; j++) {
    if (nums[j] != 0) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
        i++;
    }
}
```

**Why swap works:** swap preserves both values — the non-zero moves forward, the zero moves to where the non-zero was. Zeros naturally migrate to the end.

---

## Common Pitfalls

- **Making the two pointers "cross"** — letting `i` advance past `j` by using separate inner `while` loops to skip. Don't do this. Stick to the single-loop template where `j` always moves and `i` only moves on keeps. Then `i ≤ j` is automatic.

- **Overwriting data you haven't read yet** — e.g., writing to `nums[i+2]` when `i+2` hasn't been scanned by `j` yet destroys future values. Always write to `nums[i]` (the write pointer itself), not some offset.

- **Comparing the wrong index in dedup problems** — use `nums[j]` (candidate) vs `nums[i-K]` (K-back from write pointer). Don't use `nums[i]` on the left — `nums[i]` is where you're about to write; it has no useful info yet.

- **Flipping `!=` and `==`** — you want to KEEP when values are DIFFERENT (no extra copy coming in). Easy to flip under mental load.

- **Missing the `i < K` edge case** — the first K elements are always kept unconditionally (you haven't accumulated K yet). Without this guard, you'd try to read `nums[i-K]` with a negative index.

---

## Problems Solved

### 1. Remove Element (LC 27)

- **Mental Model:** Walk every paper. If it's not the target value, put it on your keepers pile.
- **Filter:** `nums[j] != val`
- **Return:** new length (`i`)
- **Test case:** `nums = [3, 2, 2, 3], val = 3` → returns `2`, array becomes `[2, 2, _, _]`
  - Another: `nums = [0, 1, 2, 2, 3, 0, 4, 2], val = 2` → returns `5`, array becomes `[0, 1, 3, 0, 4, _, _, _]`

### 2. Remove Duplicates from Sorted Array (LC 26)

- **Mental Model:** Same walk — but since the array is sorted, duplicates sit next to each other. Keep only if different from the last one you kept.
- **Filter:** `i == 0 || nums[j] != nums[i - 1]`
- **Return:** new length (`i`)
- **Test case:** `nums = [1, 1, 2]` → returns `2`, array becomes `[1, 2, _]`
  - Another: `nums = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]` → returns `5`, array becomes `[0, 1, 2, 3, 4, _, _, _, _, _]`

### 3. Move Zeroes (LC 283)

- **Mental Model:** Same walk — copy non-zeros forward, then fill the tail with zeros. OR one-pass swap so zeros naturally migrate to the back.
- **Filter:** `nums[j] != 0`
- **Return:** void (mutate in place)
- **Test case:** `nums = [0, 1, 0, 3, 12]` → array becomes `[1, 3, 12, 0, 0]`
  - Edge: `nums = [0]` → `[0]` (nothing to move).

### 4. Remove Duplicates II — Keep At Most 2 (LC 80)

- **Mental Model:** Same walk — but this time, compare with what's 2 positions back on the write side. If it matches, you already have 2 copies — skip.
- **Filter:** `i < 2 || nums[j] != nums[i - 2]`
- **Return:** new length (`i`)
- **Test case:** `nums = [1, 1, 1, 2, 2, 3]` → returns `5`, array becomes `[1, 1, 2, 2, 3, _]`
  - Another: `nums = [0, 0, 1, 1, 1, 1, 2, 3, 3]` → returns `7`, array becomes `[0, 0, 1, 1, 2, 3, 3, _, _]`
  - Tricky: `nums = [1, 2, 3, 4]` (all unique) → returns `4`, array stays `[1, 2, 3, 4]`.

---

### Summary Table

| # | Problem | Filter | What's written | Returns |
|---|---------|--------|----------------|---------|
| 1 | Remove Element (LC 27) | `nums[j] != val` | `nums[j]` | new length |
| 2 | Remove Duplicates (LC 26) | `i == 0 \|\| nums[j] != nums[i-1]` | `nums[j]` | new length |
| 3 | Move Zeroes (LC 283) | `nums[j] != 0` | `nums[j]` (or swap) | void |
| 4 | Remove Duplicates II (LC 80) | `i < 2 \|\| nums[j] != nums[i-2]` | `nums[j]` | new length |

---

## Complexity

| Metric | Value |
|--------|:-----:|
| Time | O(n) — single pass |
| Space | O(1) — in-place |

Two-pass variants (like Move Zeroes v1) are still O(n) time — two passes is 2n operations = still linear.

---

## Mental Checklist When You See a Problem

1. Does the problem say "in-place" or "return the new length"?
2. Am I filtering/removing/compacting based on some condition?
3. Can I describe the keep rule as "this element passes the filter"?

If YES → use the read/write pointer template. The whole problem reduces to picking the filter condition.

---

## The Generalization Takeaway

Once you see this template a few times, you realize nearly every "array in-place" problem is:

> **Walk with `j`. Define a filter. On "keep", write to `nums[i]` and advance `i`. Return `i`.**

You'll see it again in:
- Kadane-style running maximums
- Partitioning problems (Dutch National Flag, etc.)
- Backtracking-adjacent problems where you compact a grid/row
- String compression

The skeleton stays the same; only the filter logic changes.