# String Comparison with Backspaces

## When to Recognize It

You see this pattern when:
- A string has **special characters that "undo" previous characters** (e.g., `#` = backspace)
- You need to compare two such strings **after applying the undo logic**
- The naive "build the string with a stack, then compare" works but uses O(n) space

**Clue words:** "backspace", "undo", "skip with marker", "type out with rules"

---

## Core Intuition

Two approaches:

1. **Stack approach (O(n) space):** simulate typing — push real chars, pop on `#`. Compare both final strings.
2. **Two-pointer from the END (O(1) space):** walk both strings backward simultaneously. When you see `#`, you "owe" yourself a deletion of the next non-`#` char. Compare only "surviving" chars from each side.

The two-pointer version is the goal — it teaches a powerful generalization: **when the operation deletes things ahead, walk backward to know exactly what to skip.**

---

## Why Walk Backward?

Going **forward**: when you see a char, you don't yet know if a future `#` will delete it. So you have to commit to keeping it, then maybe pop it later. That's the stack.

Going **backward**: when you see a `#`, you immediately know it'll delete **the next non-`#` char** (which is to your left now). You can carry that knowledge as a "skip counter" and consume it cleanly.

**Backward walking turns a stateful problem into a stateless one.**

---

## Template 1 — Stack Approach (O(n) space)

```java
public boolean backspaceCompare(String s, String t) {
    return build(s).equals(build(t));
}

private String build(String str) {
    Stack<Character> stack = new Stack<>();
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        if (c == '#') {
            if (!stack.isEmpty()) stack.pop();
            // else: silently skip (delete on empty = no-op)
        } else {
            stack.push(c);
        }
    }
    return stack.toString();
}
```

**Common pitfall:** the `else` for `#` on an empty stack should be a no-op, **not** an "add `#`" branch. Missing this turns a `#` into a real character.

**Java dangling-else warning:** always brace your outer `if` block when nesting another `if/else` inside, or the `else` will bind to the wrong `if`.

---

## Template 2 — Two Pointers Walking Backward (O(1) space)

```java
public boolean backspaceCompare(String s, String t) {
    int i = s.length() - 1, j = t.length() - 1;
    int skipS = 0, skipT = 0;

    while (i >= 0 || j >= 0) {
        // walk i back to next surviving char in s
        while (i >= 0 && (s.charAt(i) == '#' || skipS > 0)) {
            if (s.charAt(i) == '#') skipS++;
            else skipS--;
            i--;
        }

        // walk j back to next surviving char in t
        while (j >= 0 && (t.charAt(j) == '#' || skipT > 0)) {
            if (t.charAt(j) == '#') skipT++;
            else skipT--;
            j--;
        }

        // both pointers now at survivors OR below 0
        if (i < 0 && j < 0) return true;
        if (i < 0 || j < 0) return false;
        if (s.charAt(i) != t.charAt(j)) return false;

        i--;
        j--;
    }
    return true;
}
```

### The 4 critical pieces

1. **Outer loop is `||` not `&&`** — keep going as long as either string has chars left to process (so length mismatches get caught).

2. **Each pointer walks independently** — no lockstep `i--, j--` together. Each side has its own inner walk-back loop with its own skip counter.

3. **Inner walk-back covers TWO cases:**
   - Current char is `#` → `skip++`, step back (a deletion is now "pending")
   - Current char is real AND `skip > 0` → `skip--`, step back (this char was deleted)
   - Stop when current char is real AND `skip == 0` → surviving char found

4. **After both walks, three possible states:**
   - Both `< 0` → strings ended together → `return true`
   - Exactly one `< 0` → one string is longer effectively → `return false`
   - Both at survivors → compare the chars

---

## Mental Model: Two Sweepers Walking Backward

Two sweepers, each walking backward through their own string. Each carries a "ticket book" (skip counter):

- See a `#` → write an IOU in the ticket book (`skip++`). "I owe a deletion."
- See a real char with an IOU outstanding → cash in the IOU (`skip--`). "This char is the one I owed to delete."
- See a real char with no IOU → stop. This is a surviving char.

When both sweepers stop, compare their survivors. If they match, both step back and keep going. Length mismatches show up when one sweeper's string runs out before the other.

---

## Common Pitfalls

- **Lockstep walking (`i--, j--` together every iteration)** — wrong. Each pointer must walk independently based on its own skips.

- **Single skip counter for both strings** — wrong. Each string has its OWN skips. Two counters needed.

- **Outer loop `&&` instead of `||`** — misses length mismatches. If one string has chars left, we still need to process them (could be all `#`s that cancel out, or surviving chars that mean mismatch).

- **Treating `#` on empty stack as "add a `#` char"** (stack version) — turns `#` into a real character. Must be a silent no-op.

- **Java dangling-else** — without explicit braces, `else` binds to the nearest unmatched `if`, even if your indentation says otherwise. Always brace outer `if` bodies.

- **Inner walk-back only handles `#`, not deleted chars** — when `skip > 0` and current char is real, that char is deleted; the walk must consume it (`skip--`, step back), not stop.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Stack | O(n + m) | O(n + m) |
| Two pointers backward | O(n + m) | O(1) |

Both are linear time. The two-pointer version wins on space.

---

## Problems Solved

### 1. Backspace String Compare (LC 844)

- **Mental Model:** Two sweepers walking backward. Each carries a ticket book (skip counter). `#` writes an IOU; the next real char cashes it in. When both sweepers stop at survivors, compare. Length mismatches caught by the `||` outer loop.
- **Recognize it when:** any "X with #-style undo characters" problem where you compare results
- **Key takeaway:** Walking backward turns a stateful problem (forward: "is this char going to be deleted later?") into a stateless one ("when I see #, the next non-# back is gone — I just count").
- **Test cases:**
  - `s = "ab#c", t = "ad#c"` → `true` (both become `"ac"`)
  - `s = "a##c", t = "#a#c"` → `true` (both become `"c"`)
  - `s = "ab", t = "b"` → `false` (lengths differ after backspace — `"ab"` vs `"b"`)
  - `s = "y#fo##f", t = "y#f#o##f"` → `true` (both become `"f"`)
  - Edge: `s = "", t = "#"` → `true` (both empty)

---

### Summary Table

| # | Problem | Approach | One-line Mental Model |
|---|---------|----------|-----------------------|
| 1 | Backspace String Compare (LC 844) | Two pointers walking backward | Walk back, count IOUs (`#`), cash them in on real chars; compare survivors |

---

## Mental Checklist When You See a Problem

1. Does the problem have **undo/delete characters** that retroactively affect what was just kept?
2. Am I comparing two such strings/sequences after applying the undo?
3. Is O(n) space the simple solution, but O(1) is desired?

If YES → two pointers walking **backward**, each with its own skip counter.

---

## The Generalization Takeaway

The "walk backward + skip counter" technique generalizes to any problem where:
- An operation **deletes things ahead of itself** (or behind, when read in reverse)
- You need to know which elements actually "survive"
- You want O(1) space

You'll see this idea reappear in:
- Custom undo-like simulations
- String compression decoding
- Some stack-replacement optimizations

The pattern is: **if forward walking requires lookahead, backward walking often replaces it with a simple counter.**