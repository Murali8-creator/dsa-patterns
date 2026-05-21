# Character Frequency Matching (Sliding Window)

## When to Recognize It

You see this pattern when:
- The problem mentions **anagram**, **permutation**, or **rearrangement** of one string inside another
- You need to check if a string contains a **substring matching the character counts** of a pattern
- The **window size is fixed** (= length of the pattern)
- The alphabet is **bounded** (lowercase English → 26, ASCII → 128, etc.)

**Clue words:** "permutation in string", "anagrams of P in S", "substring with same chars as P", "rearrangement contained in"

---

## Core Intuition

> All permutations of a string share **exactly one property**: identical character frequencies.

So instead of generating all permutations (exponential — `n!` of them), you check the simpler question: **does this window have the same character frequencies as the pattern?**

Sliding a fixed-size window of length `|p|` across `s` and comparing frequencies turns an exponential problem into a linear one.

### Analogy: Comparing two ingredient lists

You have a recipe (pattern) requiring specific quantities of ingredients (characters). You walk along a buffet (text), grabbing a contiguous stretch of dishes (window). Question: does my window have **exactly** the recipe's ingredients in the right quantities?

You don't recompute the whole list every step. As the window slides, one ingredient leaves and one enters — you adjust the count and re-check.

---

## The Clever Trick — Single Hash + "All Zeros = Match"

Instead of maintaining TWO hashes (target + window) and comparing them every slide, use **ONE hash**:

1. **Build the pattern's frequency** into the hash (positive counts).
2. As the window slides, **decrement** on the character entering, **increment** on the character exiting.
3. **If the entire hash is all zeros**, the window is a perfect anagram of the pattern.

### Why "all zeros" means a match

- A character that appears `k` times in the pattern starts at `+k`.
- The window adds it `k` times → count becomes 0.
- Extra occurrences → count goes negative.
- Missing occurrences → count stays positive.
- Window matches ↔ every character's net count is 0.

### Why the array, not a HashMap

For lowercase English letters (26), an `int[26]` is:
- Faster than `HashMap` (no boxing, no hashing)
- Trivial "all-zeros" check (a simple loop)
- Constant space — array size doesn't grow with input

For other alphabets, scale the array to fit (`int[128]` for ASCII, etc.).

---

## Canonical Template

```java
public boolean checkInclusion(String pattern, String text) {
    int m = pattern.length();
    int n = text.length();
    if (m > n) return false;

    int[] hash = new int[26];

    // Step 1: build pattern frequency
    for (int i = 0; i < m; i++) {
        hash[pattern.charAt(i) - 'a']++;
    }

    // Step 2: slide a window of size m across text
    int i = 0;
    for (int j = 0; j < n; j++) {
        hash[text.charAt(j) - 'a']--;        // entering — decrement

        // shrink if window exceeded size m
        while (j - i + 1 > m) {
            hash[text.charAt(i) - 'a']++;    // exiting — increment
            i++;
        }

        // when window is exactly size m, check
        if (j - i + 1 == m && isAllZeroes(hash)) {
            return true;
        }
    }
    return false;
}

private static boolean isAllZeroes(int[] hash) {
    for (int v : hash) if (v != 0) return false;
    return true;
}
```

### The 4 critical pieces

1. **Build pattern hash first** — positive counts representing what we need.
2. **Decrement on enter** — each window char "consumes" one count from the target.
3. **Increment on exit** — chars leaving the window "release" their consumption.
4. **All-zeros check** — perfect match means every target count was exactly satisfied.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Generate all permutations + search | O(m! × n) | O(m!) |
| Two hashes + compare each slide | O(n × 26) = O(n) | O(26) = O(1) |
| **Single hash + decrement/increment + all-zeros** | **O(n × 26) = O(n)** | **O(26) = O(1)** |

Both linear approaches are equivalent in big-O. The "single hash" version is slightly more elegant and uses half the memory.

### Optional optimization: matched-counter

Instead of scanning the 26-element array every iteration, maintain a `matched` counter — number of characters whose frequency currently equals the target's. When `matched == distinctCharCount`, it's a perfect match. Reduces the inner check from O(26) to O(1) per slide, but adds bookkeeping. Skip unless you need every microsecond.

---

## Problems Solved

### 1. Permutation in String (LC 567)

- **Mental Model:** Slide a window of `|s1|` over `s2`. Build s1's char frequency. As the window slides, decrement entering chars and increment exiting ones. The instant the frequency array is all zeros, we have an anagram match.
- **Recognize it when:** "does s2 contain a permutation of s1" — i.e., return true if any substring of s2 is an anagram of s1
- **Key takeaway:** Don't generate permutations (factorial blowup). Permutation match = frequency match.
- **Test cases:**
  - `s1 = "ab", s2 = "eidbaooo"` → `true` (`"ba"` at index 3)
  - `s1 = "ab", s2 = "eidboaoo"` → `false`
  - `s1 = "abc", s2 = "cba"` → `true`

### 2. Find All Anagrams in a String (LC 438)

- **Mental Model:** Same as LC 567, but collect **every** starting index instead of returning at the first match. Pure copy-paste with `return true` replaced by `res.add(i)`.
- **Recognize it when:** find all start indices where pattern `p` appears as an anagram in `s`
- **Key takeaway:** This is LC 567 with a list output. The template doesn't change.
- **Test cases:**
  - `s = "cbaebabacd", p = "abc"` → `[0, 6]` (anagrams `"cba"` at 0, `"bac"` at 6)
  - `s = "abab", p = "ab"` → `[0, 1, 2]` (every position is an anagram)

---

### Summary Table

| # | Problem | What changes from the template | One-line Mental Model |
|---|---------|--------------------------------|-----------------------|
| 1 | Permutation in String (LC 567) | Return `true` on first match | Frequencies match → permutation present |
| 2 | Find All Anagrams (LC 438) | Collect indices, don't return early | Every all-zeros moment is a match — collect them |

---

## Common Pitfalls

- **Generating permutations of the pattern explicitly** — exponential blowup (`m!`). Reframe as frequency comparison.
- **Confusing `s1` and `s2`** when building vs. sliding — build the hash from the **pattern** (the smaller one), slide the **text** (the bigger one).
- **Mixing up enter and exit signs** — entering = decrement (consuming target), exiting = increment (releasing). Reverse this and the math doesn't track.
- **Checking "all zeros" before the window reaches full size** — guard with `if (j - i + 1 == m)`.
- **Using `HashMap` when an array suffices** — for bounded alphabets, `int[26]` is faster and cleaner.
- **Forgetting the `m > n` edge case** — if the pattern is longer than the text, return false immediately.

---

## Mental Checklist When You See a Problem

1. Does the problem involve **anagrams** / **permutations** as substrings?
2. Is the **alphabet bounded** (lowercase, ASCII, fixed set)?
3. Is the **window size fixed** (= length of the pattern)?

If YES to all → Character Frequency Matching. Apply the single-hash template with all-zeros check.

---

## Connection to Other Sub-patterns

| Sub-pattern | Window size | What you compare |
|-------------|:-----------:|------------------|
| Fixed-Size Subarray (20) | constant `k` | a numeric value (sum, max, count) |
| **Character Frequency (22)** | **constant `m`** | **frequency array** (anagram check) |
| Variable-Size Condition (21) | dynamic | a condition on state |

Sub-pattern 22 is a specialization of sub-pattern 20: same fixed-window template, but the "window state" is a 26-element frequency array instead of a scalar. Once you internalize 20, 22 is just plugging in a richer state object.

---

## The Pattern in One Sentence

> Slide a fixed-width window across the text and ask "does my window's character frequency match the pattern's?" — using a single hash array decremented on enter and incremented on exit, with "all zeros" as the success signal.