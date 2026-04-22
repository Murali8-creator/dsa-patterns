# Two Pointers — Converging Pointers

## When to Recognize It

You see this pattern when:
- Array is **sorted** (or you can sort it)
- You need to find **pairs/triplets** that satisfy a condition (sum, difference, etc.)
- Brute force would be O(n²) or O(n³) with nested loops

**Clue words in problem:** "two numbers that sum to", "triplets", "closest sum", "pair with target"

## Core Intuition

In a sorted array, if you place pointers at both ends:
- `sum < target` → left pointer must move right (need bigger values)
- `sum > target` → right pointer must move left (need smaller values)
- **Each move eliminates an entire row of possibilities**, not just one pair

That's why it's O(n) instead of O(n²).

## Template — Two Sum (sorted)

```java
int left = 0, right = n - 1;

while (left < right) {
    int sum = nums[left] + nums[right];

    if (sum == target) {
        // found it
        return new int[]{left, right};
    } else if (sum < target) {
        left++;
    } else {
        right--;
    }
}
```

## Template — Three Sum (reduce to Two Sum)

**Key idea:** Fix one element → the remaining two form a Two Sum problem.

```java
Arrays.sort(nums);

for (int i = 0; i < n - 2; i++) {
    // SKIP DUPLICATE i
    if (i > 0 && nums[i] == nums[i - 1]) continue;

    int left = i + 1, right = n - 1;

    while (left < right) {
        int sum = nums[i] + nums[left] + nums[right];

        if (sum == 0) {
            // add to result

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

## Duplicate Handling Rules

In a **sorted** array, duplicates sit next to each other. So:

1. **Outer loop:** `if (i > 0 && nums[i] == nums[i-1]) continue;` — already explored all results for this value
2. **After finding a match:** skip both pointers past duplicates before moving — avoids finding the same pair again
3. **You almost never need a Set for deduplication in sorted arrays** — just compare with neighbor

## Complexity

| Approach | Time | Space |
|----------|------|-------|
| Two Sum (sorted, two pointers) | O(n) | O(1) |
| Three Sum (sort + two pointers) | O(n²) | O(1) excluding result |

## Common Pitfalls

- Forgetting to **sort first** — two pointers only works on sorted data
- Skipping duplicates on **every** iteration instead of **only after a match** — this can skip valid pairs
- Inner duplicate skip compares with **next** element (`nums[left] == nums[left+1]`), done **before** the final `left++, right--`
- Off-by-one: loop runs while `left < right` (strict), not `left <= right`

## Problems Solved

| # | Problem | Key Takeaway |
|---|---------|--------------|
| 1 | Two Sum II (LC 167) | Pure two-pointer template on sorted array |
| 2 | Three Sum (LC 15) | Fix one + two pointer on rest. Duplicate skip at both levels |