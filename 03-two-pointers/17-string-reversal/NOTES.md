# String Reversal (Converging Two Pointers — Swap Variant)

## When to Recognize It

You see this pattern when:
- You need to **reverse** a string, array, or part of one **in-place**
- The operation is "swap pairs starting from both ends and move inward"
- No extra space allocation allowed (O(1) space)

**Clue words:** "reverse", "reverse in-place", "reverse every K characters", "reverse only X characters"

---

## Core Intuition

Converging two pointers, but the action between them is **swap** (not compare). No filter, no decision — just swap and step inward.

### Analogy: Conveyor Belt

Two workers at opposite ends of a conveyor belt. Each grabs one item, they trade items with each other, then step one position toward the middle. When they meet, the belt is reversed.

---

## Core Template

```java
int left = 0, right = n - 1;
while (left < right) {
    // swap
    char tmp = c[left];
    c[left] = c[right];
    c[right] = tmp;

    left++;
    right--;
}
```

That's the whole skeleton. Three variants below tweak when/where the swap happens.

---

## Variants

### 1. Reverse Everything (LC 344)

Apply template directly to the full string.

```java
public void reverseString(char[] s) {
    int n = s.length;
    int left = 0, right = n - 1;
    while (left < right) {
        char tmp = s[left];
        s[left] = s[right];
        s[right] = tmp;
        left++;
        right--;
    }
}
```

### 2. Conditional Swap — Reverse Only Some Characters (LC 345)

Same template, but only swap when **both** pointers point to "valid" characters (vowels, in this case). Otherwise advance the non-vowel side.

```java
private static final Set<Character> VOWELS =
    Set.of('a','e','i','o','u','A','E','I','O','U');

public String reverseVowels(String s) {
    char[] c = s.toCharArray();
    int left = 0, right = c.length - 1;

    while (left < right) {
        if (VOWELS.contains(c[left]) && VOWELS.contains(c[right])) {
            char tmp = c[left]; c[left] = c[right]; c[right] = tmp;
            left++;
            right--;
        } else if (!VOWELS.contains(c[left])) {
            left++;
        } else {
            right--;
        }
    }
    return String.valueOf(c);
}
```

**Key insight:** The "vowel set" is a **static constant** — don't recreate it per call.

### 3. Stride-Based — Reverse First K of Every 2K (LC 541)

Outer loop jumps by `2k`. For each chunk, reverse the first `k` characters (or fewer if at the end).

```java
public String reverseStr(String s, int k) {
    char[] c = s.toCharArray();
    int n = c.length;

    for (int i = 0; i < n; i += 2 * k) {
        int left = i;
        int right = Math.min(i + k - 1, n - 1);  // clamp to end of string
        while (left < right) {
            char tmp = c[left]; c[left] = c[right]; c[right] = tmp;
            left++;
            right--;
        }
    }
    return String.valueOf(c);
}
```

**Three pieces:**
- Outer loop strides by `2k` — jumps to start of next chunk
- `right = min(i + k - 1, n - 1)` — clamps when fewer than `k` chars remain
- Inner loop is the standard swap-and-converge

---

## Common Pitfalls

- **Forgetting the loop condition is `left < right`, not `left <= right`** — at `left == right` (odd-length array), there's nothing to swap.
- **Creating a fresh `HashSet` of vowels per call** — allocate it once as a static constant.
- **Inconsistent stride in LC 541** — must jump by `2k` each iteration, not `k`. Easiest formula: `i += 2*k` in the for update.
- **Forgetting to clamp `right`** in LC 541 — if fewer than `k` chars remain in the last chunk, `right` would go past the array's end without clamping to `n - 1`.

---

## Problems Solved

### 1. Reverse String (LC 344)

- **Mental Model:** Two workers at opposite ends of a conveyor belt. Trade items, step inward, repeat until they meet.
- **Recognize it when:** reverse an entire string/array in-place
- **Key takeaway:** Simplest application of converging two pointers — no condition, just swap.
- **Test case:** `s = ['h','e','l','l','o']` → `['o','l','l','e','h']`

### 2. Reverse Vowels of a String (LC 345)

- **Mental Model:** Same conveyor belt, but workers only swap when BOTH have a vowel in hand. Otherwise the one without a vowel passes their item (advances) without swapping.
- **Recognize it when:** reverse only certain characters in a string, others stay put
- **Key takeaway:** The template extends with a conditional — swap only when both pointers satisfy the filter; otherwise advance only the failing side.
- **Test case:** `s = "hello"` → `"holle"` (e and o swapped)
  - Another: `s = "leetcode"` → `"leotcede"` (vowels `e, e, o, e` reversed to `e, o, e, e`)

### 3. Reverse String II (LC 541)

- **Mental Model:** Walk the string in chunks of `2k`. For each chunk, reverse only its first `k` chars (the rest of the chunk stays put). Clamp at the end if the last chunk is short.
- **Recognize it when:** reverse pieces of a string at fixed intervals
- **Key takeaway:** Stride math is the only tricky part. Inner reverse is identical to LC 344. Always increment outer by `2k`, clamp `right` to `n-1` for the final partial chunk.
- **Test case:** `s = "abcdefg", k = 2` → `"bacdfeg"`
  - Chunks of `2k=4`: `"ab|cd"` and `"ef|g"`. Reverse first 2 of each: `"ba"cd` and `"fe"g`. Result: `"bacdfeg"`.
  - Edge: `s = "abc", k = 5` → `"cba"` (reverse entire string since fewer than k chars).

---

### Summary Table

| # | Problem | Twist | One-line Mental Model |
|---|---------|-------|-----------------------|
| 1 | Reverse String (LC 344) | None | Swap pairs from both ends, walk inward |
| 2 | Reverse Vowels (LC 345) | Filter | Swap only when both pointers on vowels |
| 3 | Reverse String II (LC 541) | Stride | Reverse first k chars of every 2k-sized chunk |

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Any variant above | O(n) | O(1) |

Each character is visited at most twice (once when its pointer reaches it, once when it gets swapped or stepped over). Linear time, constant space.

---

## Mental Checklist When You See a Problem

1. Does the problem say "in-place" + "reverse"?
2. Am I swapping pairs symmetrically (start ↔ end, then move inward)?
3. Is there a filter / stride / boundary on which positions actually get swapped?

If YES → converging two pointers + swap. Pick the variant based on the twist:
- No twist → LC 344 template
- Conditional swap → LC 345 template
- Chunk-based → LC 541 template