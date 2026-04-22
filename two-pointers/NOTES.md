# Two Pointers — Converging Pointers

## When to Recognize It

You see this pattern when:
- Array is **sorted** (or you can sort it), OR
- Array has **positional meaning** (bars, walls, distances)
- You need to find **pairs/triplets** satisfying a condition, OR
- You need to measure something **between two ends** (area, water, distance)
- Brute force is O(n²) or O(n³) with nested loops

**Clue words in the problem:** "two numbers that sum to", "triplets", "closest sum", "pair with target", "max area", "container", "trap water"

---

## Core Intuition

Place pointers at both ends of the array, move them inward based on some rule.

Each move **eliminates an entire set of possibilities** — not just one pair. That's why O(n) works instead of O(n²).

---

## When to Sort and When NOT to Sort

| Problem type | Sort? | Why |
|--------------|:-----:|-----|
| Sum/comparison on **values** (Two Sum, Three Sum, 4Sum) | **Yes** | Need monotonic behavior — if sum is too small, moving left is the only way to grow it. |
| Physical **positions** matter (Container, Trapping Rain Water) | **No** | Indices ARE the problem — widths, distances, walls at specific locations. Sorting destroys that. |

**Rule of thumb:**
- Pattern depends on "what's bigger/smaller" → **sort**
- Pattern depends on "what's at this position" → **don't sort**

---

## Template 1 — Sum-Based (Two Sum on sorted array)

```java
Arrays.sort(nums);   // only if not already sorted

int left = 0, right = n - 1;

while (left < right) {
    int sum = nums[left] + nums[right];

    if (sum == target) {
        // found it
        return new int[]{left, right};
    } else if (sum < target) {
        left++;       // need a bigger sum
    } else {
        right--;      // need a smaller sum
    }
}
```

---

## Template 2 — Three Sum (reduce to Two Sum)

**Key idea:** Fix one element → the remaining two form a Two Sum problem.

```java
Arrays.sort(nums);
List<List<Integer>> res = new ArrayList<>();

for (int i = 0; i < n - 2; i++) {
    // SKIP DUPLICATE i
    if (i > 0 && nums[i] == nums[i - 1]) continue;

    int left = i + 1, right = n - 1;

    while (left < right) {
        int sum = nums[i] + nums[left] + nums[right];

        if (sum == 0) {
            res.add(List.of(nums[i], nums[left], nums[right]));

            // SKIP DUPLICATE left
            while (left < right && nums[left] == nums[left + 1]) left++;
            // SKIP DUPLICATE right
            while (left < right && nums[right] == nums[right - 1]) right--;

            left++;
            right--;
        } else if (sum < 0) {
            left++;
        } else {
            right--;
        }
    }
}
```

### Duplicate Handling Rules

In a **sorted** array, duplicates sit next to each other. So:

1. **Outer loop:** `if (i > 0 && nums[i] == nums[i-1]) continue;` — already explored all triplets for this value
2. **Inner match:** after finding a match, skip duplicates on both pointers **before** moving them
3. **Almost never need a `Set`** — just compare with neighbor

Why the inner skip is inside `if (sum == 0)`: only after a match do you need to avoid re-finding the same pair. Skipping unconditionally could hop over valid pairs.

---

## Template 3 — Bottleneck-Based (Container With Most Water)

Decision is NOT based on sum — it's based on **which side limits you**.

```java
int left = 0, right = n - 1, maxArea = 0;

while (left < right) {
    int area = (right - left) * Math.min(height[left], height[right]);
    maxArea = Math.max(maxArea, area);

    if (height[left] < height[right])
        left++;          // move the shorter side
    else
        right--;
}
```

**Why move the shorter side?**
- Area = width × min(heights)
- Moving the taller side: width decreases AND the min can't grow (shorter side still limits)
- Moving the shorter side: width decreases BUT the min *might* grow
- So shorter side is the only move with a chance to increase area

---

## Template 4 — Trapping Rain Water (Prefix-Max Combo)

**This is a combo pattern:** two-pointer thinking + prefix/suffix max (precomputation).

### The core formula

For each position `i`:
```
water[i] = min(maxLeft[i], maxRight[i]) - height[i]
```

Water rises up to the shorter of the two "walls" on each side.

### Brute Force — O(n²)

For each `i`, scan left and right to find max. Works but TLEs on large inputs.

### Optimized — Precompute Both Arrays (Prefix/Suffix Max)

```java
int n = height.length;
int[] leftMax = new int[n];
int[] rightMax = new int[n];

leftMax[0] = height[0];
for (int i = 1; i < n; i++)
    leftMax[i] = Math.max(leftMax[i-1], height[i]);

rightMax[n-1] = height[n-1];
for (int i = n-2; i >= 0; i--)
    rightMax[i] = Math.max(rightMax[i+1], height[i]);

int total = 0;
for (int i = 0; i < n; i++)
    total += Math.min(leftMax[i], rightMax[i]) - height[i];

return total;
```

**Complexity:** O(n) time, O(n) space.

### The Prefix Sum / Running Max Insight

Whenever you see:
- "for each index, I need something about the whole prefix/suffix"
- "I'm re-computing something that doesn't change"

→ **Think precomputation.** Prefix sum, prefix max, prefix min, prefix XOR — all the same technique with different operations.

| | Prefix Sum | Trapping Rain Water |
|---|---|---|
| Preprocess | Running sum | Running max (both directions) |
| Preprocess cost | O(n) | O(n) |
| Per-query cost | O(1) | O(1) |

### Optimized Further — O(1) Space with Two Pointers (advanced)

Instead of two arrays, maintain running `leftMax` and `rightMax` variables and move the pointer on the **shorter side** (same bottleneck logic as Container With Most Water).

The insight: when `height[left] < height[right]`, we know the right side has a bar at least as tall as `rightMax`. So the bottleneck at `left` is `leftMax` — we can safely compute water there.

```java
int left = 0, right = n - 1;
int leftMax = 0, rightMax = 0, total = 0;

while (left < right) {
    if (height[left] < height[right]) {
        if (height[left] >= leftMax) leftMax = height[left];
        else total += leftMax - height[left];
        left++;
    } else {
        if (height[right] >= rightMax) rightMax = height[right];
        else total += rightMax - height[right];
        right--;
    }
}
```

---

## Pattern Variants Summary

Converging two pointers can decide moves based on different things:

| Decision Logic | Example Problem | Move Rule |
|----------------|-----------------|-----------|
| Sum vs target | Two Sum II, Three Sum, 4Sum | `sum < target` → `left++`, `sum > target` → `right--` |
| Bottleneck side | Container With Most Water | Move the pointer with the smaller value |
| Character match | Valid Palindrome | Move both inward when matched |
| Prefix/suffix max | Trapping Rain Water | Move the shorter side, accumulate as you go |

---

## Complexity Summary

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force (nested loops) | O(n²) or O(n³) | O(1) |
| Two Sum (sorted, two pointers) | O(n) | O(1) |
| Three Sum (sort + two pointers) | O(n²) | O(1) excluding result |
| Container With Most Water | O(n) | O(1) |
| Trapping Rain Water (prefix max) | O(n) | O(n) |
| Trapping Rain Water (two pointers) | O(n) | O(1) |

---

## Common Pitfalls

- Forgetting to **sort first** in sum-based problems
- Sorting when it destroys the problem (Container, Trapping Rain Water)
- Skipping duplicates **unconditionally** instead of **only after a match** — can skip valid pairs
- Inner duplicate skip: compare with **next** element (`nums[left] == nums[left+1]`), done **before** the final `left++, right--`
- Off-by-one: loop runs while `left < right` (strict), not `left <= right`
- In Trapping Rain Water: confusing "water between two walls" (Container logic) with "water above each position" (TRW logic)

---

## Problems Solved

| # | Problem | Category | Key Takeaway |
|---|---------|----------|--------------|
| 1 | Two Sum II (LC 167) | Sum-based | Pure converging template on sorted array |
| 2 | Three Sum (LC 15) | Sum-based | Fix one + two pointer on rest. Duplicate skip at BOTH levels (outer `i` + inner match) |
| 3 | Container With Most Water (LC 11) | Bottleneck | Move the **shorter** pointer — only move with a chance to improve |
| 4 | Trapping Rain Water (LC 42) | Prefix max + two pointers | `water[i] = min(leftMax, rightMax) - height[i]`. Solve with O(n) precomputed arrays OR O(1) two-pointer |

---

## Mental Checklist When You See a Problem

1. Is there a sorted array (or one I can sort without losing info)?
2. Am I looking for a pair/triplet with a condition?
3. Does the problem involve two ends + something between them (area, water, distance)?
4. Am I re-computing the same prefix/suffix info in a nested loop?

If YES to any → converging two pointers (possibly combined with prefix/suffix precomputation).