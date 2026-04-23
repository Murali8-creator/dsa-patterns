# Expanding from Center (Palindromes)

## When to Recognize It

You see this pattern when:
- The problem is about **palindromes** (substrings, subsequences that read the same forwards/backwards)
- You need to **find**, **count**, or **measure** palindromic substrings
- A brute-force "check every substring" would be O(n³)

**Clue words:** "palindrome", "palindromic substring", "palindromic subsequence", "longest palindromic X", "count palindromic X"

---

## Core Intuition

A palindrome is symmetric around its center. So if you stand at the center and move both hands outward simultaneously, each pair of characters you touch must be equal. The moment they're NOT equal, the palindrome ends.

### Analogy: Mirror Walkers

Imagine two people standing back-to-back at the center of a word. They walk outward in opposite directions, each looking at the character at their feet. At every step, they compare notes:
- **Same character** → palindrome continues, keep walking
- **Different character** → palindrome ends, they stop

The palindrome is whatever they walked through (including the center).

---

## Two Types of Centers

A palindrome can be **odd length** (centered on ONE character) or **even length** (centered in the GAP between two adjacent characters).

### Odd-length palindrome
```
indices:  0  1  2  3  4
chars:    a  b  c  b  a
                ↑
             center = 'c' at index 2
```
Center is a single character. Call `expand(i, i)` — both pointers start at the same spot.

### Even-length palindrome
```
indices:  0  1  2  3
chars:    a  b  b  a
             ↑ ↑
          center gap between indices 1 and 2
```
Center is the **gap** between two characters. Call `expand(i, i+1)` — pointers start adjacent.

### Total centers in a string of length n

- `n` odd-length centers (every character)
- `n - 1` even-length centers (every adjacent pair)
- **Total = 2n − 1 centers**

---

## Key Insight: Try BOTH at Every Index

For each `i`, do **two** expansions:
1. `expand(i, i)` — odd-length palindrome at index `i`
2. `expand(i, i+1)` — even-length palindrome at gap after `i`

Both types of palindromes can appear at ANY index. Missing either means you miss valid answers.

**Common mistake:** alternating odd/even by parity of `i` (e.g., `if i even, do even expansion`). **Wrong.** Try both at every position.

---

## Template — Longest Palindromic Substring (LC 5)

### Version A: Return length, compute start from formula

```java
public String longestPalindrome(String s) {
    char[] c = s.toCharArray();
    int n = c.length;
    int startIndex = 0, maxLen = 0;

    for (int i = 0; i < n; i++) {
        int oddLen  = expand(i, i, c);       // odd-length center
        int evenLen = expand(i, i + 1, c);   // even-length center

        if (oddLen > maxLen) {
            maxLen = oddLen;
            startIndex = i - (oddLen - 1) / 2;
        }
        if (evenLen > maxLen) {
            maxLen = evenLen;
            startIndex = i - (evenLen - 1) / 2;
        }
    }
    return s.substring(startIndex, startIndex + maxLen);
}

private int expand(int start, int end, char[] c) {
    while (start >= 0 && end < c.length && c[start] == c[end]) {
        start--;
        end++;
    }
    return end - start - 1;   // palindrome length after expansion
}
```

### Why `end - start - 1` for length?

When the loop exits, `start` went one step too far left and `end` went one step too far right (the mismatch or boundary step). Actual palindrome indices are `[start+1, end-1]`. Length = `(end-1) - (start+1) + 1 = end - start - 1`.

### The Start Index Formula — `startIndex = i - (len - 1) / 2`

In plain English: **the palindrome starts half its length to the left of the center position.**

- For a palindrome of length 5 centered at `i = 4`: starts at `4 - 2 = 2` (extends 2 positions to the left).
- For a palindrome of length 4 with left-of-center at `i = 2`: starts at `2 - 1 = 1` (extends 1 more to the left).

Why a single formula works for both odd and even:

| len | (len-1)/2 (Java int division) | Why correct |
|-----|:-----------------------------:|-------------|
| 1 (odd) | 0 | center only |
| 2 (even) | 0 | start = i (left of gap) |
| 3 (odd) | 1 | 1 step left from center |
| 4 (even) | 1 | 1 more step left from i |
| 5 (odd) | 2 | 2 steps left from center |
| 6 (even) | 2 | 2 more steps left from i |

Integer division rounds down, which neatly collapses both cases into one formula.

### Version B: Return substring directly (cleaner for debugging, slower)

```java
public String longestPalindrome(String s) {
    char[] c = s.toCharArray();
    int n = c.length;
    String maxRes = "";

    for (int i = 0; i < n; i++) {
        String odd  = expand(i, i, s, c);
        String even = expand(i, i + 1, s, c);
        if (odd.length()  > maxRes.length()) maxRes = odd;
        if (even.length() > maxRes.length()) maxRes = even;
    }
    return maxRes;
}

private String expand(int start, int end, String s, char[] c) {
    String res = "";
    while (start >= 0 && end < c.length && c[start] == c[end]) {
        res = s.substring(start, end + 1);
        start--;
        end++;
    }
    return res;
}
```

**Tradeoff:** Version B is easier to reason about but allocates a fresh substring on every expansion step. Version A is faster — recommended for production / interviews.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force (check every substring) | O(n³) | O(1) |
| **Expand from center** | **O(n²)** | **O(1)** |
| Manacher's algorithm | O(n) | O(n) |

- **Brute force:** O(n²) substrings × O(n) palindrome check = O(n³)
- **Expand from center:** O(n) centers × O(n) expansion per center = O(n²)
- **Manacher's:** linear time but interview-impossible to derive — don't bother unless you've memorized it

For LC 5 constraints (n ≤ 1000), O(n²) is plenty fast.

---

## Common Pitfalls

- **Only handling one type (odd OR even)** — missing the other means you miss palindromes. Always do both per index.
- **Alternating odd/even by index parity** — wrong; try BOTH at every index.
- **Not stopping on mismatch** — the while loop must include `c[start] == c[end]` so it exits immediately on a mismatch.
- **Double-counting the center** — if you add `c[i]` before the loop AND the loop also adds it (when start==end), the center gets duplicated.
- **Building the palindrome character-by-character into a Deque/StringBuilder** — overcomplicated. Track indices (or length) and slice the original string once at the end.
- **Off-by-one in length calculation** — after the loop, `start` and `end` are one beyond the palindrome, so length is `end - start - 1`, not `end - start + 1`.

---

## Problems Solved

### 1. Longest Palindromic Substring (LC 5)

- **Mental Model:** Two walkers at every possible center of the string. They walk outward until a mismatch. Longest walk wins.
- **Recognize it when:** find the longest palindrome in a string
- **Key takeaway:** O(n) centers × O(n) expansion = O(n²). Try BOTH odd and even expansion at every index. Track start + length (or start + end indices) — don't rebuild the string char by char.
- **Test cases:**
  - `s = "babad"` → `"bab"` or `"aba"` (both valid). Odd expansion at index 1 finds `"bab"`; odd expansion at index 2 finds `"aba"`.
  - `s = "cbbd"` → `"bb"`. Even expansion at index 1 (between the two b's) finds `"bb"`.
  - `s = "a"` → `"a"`. Single char is trivially a palindrome.
  - `s = "ac"` → `"a"` or `"c"`. No multi-char palindrome exists.

---

### Summary Table

| # | Problem | Core Trick | One-line Mental Model |
|---|---------|-----------|-----------------------|
| 1 | Longest Palindromic Substring (LC 5) | Two expansions per index | Walk outward from every center; longest walk wins |

---

## Pattern Template (Recipe for Any Expand-from-Center Problem)

1. For each index `i` from 0 to `n-1`:
   a. Expand from `(i, i)` — odd-length palindrome
   b. Expand from `(i, i+1)` — even-length palindrome
   c. Update your "best answer" based on the result (longest? count? etc.)

2. `expand(left, right)` helper:
   - While `left >= 0 && right < n && s[left] == s[right]`: move outward
   - Return what you need (length, count, or start/end indices)

---

## Mental Checklist When You See a Problem

1. Does it involve a **palindromic** substring or subsequence?
2. Could the answer exist **anywhere** in the string?
3. Is brute force O(n³)?

If YES to these → expand from center.

### Related patterns (for when this isn't enough)

- **Palindromic subsequences** (LC 516) — different problem shape (non-contiguous), solved with 2D DP, not expand-from-center.
- **Palindrome Partitioning** (LC 131) — uses expand-from-center as a subroutine with backtracking.
- **Manacher's algorithm** — O(n) optimization; skip unless you're already strong here.