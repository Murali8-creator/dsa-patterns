# Fixed-Size Subarray (Sliding Window)

## When to Recognize It

You see this pattern when:
- The problem says "subarray of length **k**" or "substring of length **k**"
- The window size is **constant** (not "longest such that..." or "smallest such that...")
- You need to compute something (max sum, max count, average, etc.) over every length-`k` window

**Clue words:** "subarray of size k", "substring of length k", "every k-sized window", "exactly k consecutive"

---

## Core Intuition

A frame of **fixed width `k`** slides across the array one position at a time. At every position, you compute or check some property of what's inside the frame.

### Why brute force is wasteful

Naive: for each starting index `i`, compute the property of `nums[i..i+k-1]` from scratch. That's O(n × k) = O(n²) when k ≈ n.

### The sliding insight

When the window moves one step right:
- ONE element **enters** on the right
- ONE element **exits** on the left
- Everything in the middle is **unchanged**

So instead of recomputing the whole property, **update it incrementally**:
- New sum = old sum − exiting element + entering element
- New count = old count − exiting char (if it counted) + entering char (if it counts)

Result: **O(1) per slide**, total **O(n)** regardless of `k`.

### Analogy: A photo frame on a long mural

You're sliding a fixed-width photo frame across a long wall painting. At each position, you describe what's inside. Each slide reveals one new sliver on the right and hides one sliver on the left — you don't re-describe the whole frame.

---

## Canonical Template

```java
int i = 0;
int windowState = 0;            // sum, count, whatever the problem tracks
int answer = INITIAL_VALUE;     // MIN_VALUE for max-tracking, MAX_VALUE for min

for (int j = 0; j < n; j++) {
    // 1. Element nums[j] ENTERS the window
    windowState += /* effect of nums[j] entering */;

    // 2. Shrink the window if it exceeded size k
    if (j - i + 1 > k) {
        windowState -= /* effect of nums[i] exiting */;
        i++;
    }

    // 3. When window is exactly size k, evaluate
    if (j - i + 1 == k) {
        answer = better(answer, windowState);
    }
}

return answer;
```

### The three knobs

| Knob | What to plug in |
|------|------------------|
| `windowState` | Whatever's accumulated: sum, count, frequency map, etc. |
| "Effect of entering" | Add `nums[j]` to sum, increment count if it qualifies, etc. |
| "Effect of exiting" | Reverse of above — subtract, decrement, etc. |

---

## Examples

### LC 643 — Maximum Average Subarray I

Window state = running sum.

```java
public double findMaxAverage(int[] nums, int k) {
    int sum = 0, maxSum = Integer.MIN_VALUE;
    int i = 0;

    for (int j = 0; j < nums.length; j++) {
        sum += nums[j];

        if (j - i + 1 > k) {
            sum -= nums[i];
            i++;
        }
        if (j - i + 1 == k) {
            maxSum = Math.max(maxSum, sum);
        }
    }
    return (double) maxSum / k;
}
```

### LC 1456 — Max Vowels in Substring of Length K

Window state = vowel count.

```java
private static final Set<Character> VOWELS = Set.of('a','e','i','o','u');

public int maxVowels(String s, int k) {
    int count = 0, maxCount = 0;
    int i = 0;

    for (int j = 0; j < s.length(); j++) {
        if (VOWELS.contains(s.charAt(j))) count++;

        if (j - i + 1 > k) {
            if (VOWELS.contains(s.charAt(i))) count--;
            i++;
        }
        if (j - i + 1 == k) {
            maxCount = Math.max(maxCount, count);
        }
    }
    return maxCount;
}
```

Same skeleton, different state.

---

## Critical Off-by-One: Window Size

**Window size = `j - i + 1`**, not `j - i`.

When `i = 0` and `j = 3`, the window covers indices `0, 1, 2, 3` — that's **4 elements**, including both endpoints.

| Endpoints | Size |
|-----------|------|
| `i = 0, j = 0` | 1 |
| `i = 0, j = 2` | 3 |
| `i = 1, j = 5` | 5 |
| `i, j` (general) | `j - i + 1` |

The condition `j - i + 1 > k` correctly catches a window that's overgrown. `j - i > k` catches it one step too late.

---

## Common Pitfalls

- **Off-by-one on window size** — use `j - i + 1`, not `j - i`. This is the #1 bug in fixed-window problems.
- **Initializing `maxSum = 0`** when the array can contain all negatives — use `Integer.MIN_VALUE`, or initialize from the first full window.
- **Integer division in average problems** — cast to `double` BEFORE dividing: `(double) sum / k`, not `sum / k` (which truncates).
- **Recording the answer before the window reaches size `k`** — guard with `if (j - i + 1 == k)`.
- **Forgetting that the window slides by exactly 1** — `if` is enough, no `while` needed (the window grows at most 1 per iteration).

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force (recompute each window) | O(n × k) | O(1) |
| **Sliding window (incremental update)** | **O(n)** | **O(1)** or O(window-state size) |

The win is biggest when `k ≈ n` (10× to 1000× faster).

---

## Problems Solved

### 1. Maximum Average Subarray I (LC 643)

- **Mental Model:** A frame of width `k` slides across the array. Sum changes by one in, one out per slide.
- **Recognize it when:** find the best (max/min) sum/average over length-`k` subarrays
- **Key takeaway:** Window state = running sum. Update by `+= nums[j]` on enter, `-= nums[i]` on exit.
- **Test case:** `nums = [1, 12, -5, -6, 50, 3], k = 4` → `12.75` (window `[12, -5, -6, 50]`)

### 2. Maximum Number of Vowels in a Substring of Given Length (LC 1456)

- **Mental Model:** Same frame, but instead of summing, tally vowels in the window.
- **Recognize it when:** count of specific items in a fixed-length window
- **Key takeaway:** Window state = qualifying-element count. Increment/decrement only when the entering/exiting element matches the rule.
- **Test case:** `s = "abciiidef", k = 3` → `3` (substring `"iii"`)

---

### Summary Table

| # | Problem | Window State | One-line Mental Model |
|---|---------|--------------|-----------------------|
| 1 | Max Average Subarray I (LC 643) | running sum | Frame slides; sum updates by enter/exit |
| 2 | Max Vowels in Substring (LC 1456) | vowel count | Frame slides; count only flips on vowel-in or vowel-out |

---

## Mental Checklist When You See a Problem

1. Is the window size **fixed and given** (a specific `k`)?
2. Am I computing something (sum, count, max, min, average) over **every** length-`k` window?
3. Can the property be **updated incrementally** when one element enters and one exits?

If YES to all → fixed-size sliding window. Plug the right "enter/exit" effect into the canonical template.

---

## The Pattern in One Sentence

> Slide a fixed-width frame across the array, updating an incrementally-maintained "state" by adding the entering element and subtracting the exiting one. O(n) instead of O(n × k).