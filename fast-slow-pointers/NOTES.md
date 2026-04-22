# Fast & Slow Pointers (Floyd's Tortoise & Hare)

## When to Recognize It

You see this pattern when:
- Working with a **linked list** (or any structure where you can only walk forward, one step at a time)
- Need to detect a **cycle**
- Need to find the **middle** of a list
- Need to find the **kth from end** without knowing the length
- Structure has no indexing / no way to know the length cheaply

**Clue words:** "cycle", "loop", "middle of the list", "nth from end", "palindromic linked list"

---

## Core Intuition

Two pointers, **both starting at head**, both moving **forward**, but at **different speeds**:
- `slow` moves **1 step** at a time
- `fast` moves **2 steps** at a time

### Analogy: Two Runners on a Track

Imagine a jogging track. A slow runner and a fast runner start at the same spot. The fast runner jogs at twice the speed.

- **Straight track (no cycle):** the fast runner reaches the finish line first. Done.
- **Circular track (cycle):** the fast runner goes around and around. Eventually they catch up to the slow runner **from behind** and tap them on the shoulder. That's the cycle being detected.

The gap between them closes by 1 unit per step (fast moves 2, slow moves 1 → net +1 gain per step). Since the cycle has a finite length, the gap MUST reach 0 — i.e., they MUST meet.

### For finding the middle

Same two runners, straight track. When the fast runner finishes, the slow runner is at the halfway mark — because they've been running for the same amount of time, at half the speed.

**Space: O(1)** — this is the main win over the HashSet-based brute force.

---

## Java Concept: HashSet Reference Equality

For the brute-force "have I seen this node?" approach:

```java
Set<ListNode> visited = new HashSet<>();
```

**Key:** `HashSet<ListNode>` uses default `hashCode()` and `equals()`, which rely on **memory address** (reference equality) — NOT on the node's value.

- Two nodes with the same value but different memory addresses → considered **different** by the set.
- Only the **exact same node object** reached again → cycle.
- This is why duplicates in value don't break cycle detection.

Contrast with `HashSet<Integer>`: that would track values, which is wrong here (duplicate values could falsely signal a cycle).

---

## Template 1 — Cycle Detection (LC 141)

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (fast == slow) return true;
    }
    return false;
}
```

### Null Safety — The Critical Detail

Loop condition checks `fast != null && fast.next != null`.

Why both?
- `fast.next.next` reads `fast.next` first. If `fast.next` is null, NPE.
- `fast != null` alone doesn't save you.

### Move First, Check After

Both pointers start at `head`. If you check before moving, you'd return `true` at step 0 (they're at the same node). So: **move, then check**.

---

## Template 2 — Finding the Middle (LC 876)

Same structure, just walk until the end. When `fast` reaches the end, `slow` is at the middle.

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

**Behavior on even length:** returns the **second** middle (e.g., for `[1,2,3,4]`, returns node 3). This matches LC 876's spec.

If you wanted the **first** middle instead: start `fast = head.next` (fast is one step ahead from the start).

---

## Template 3 — Floyd's Cycle Start (LC 142)

### Analogy: Two Detectives

A suspect walks down a road, then enters a circular neighborhood and keeps looping around it. We know they're looping, but we need to find the **exact street where they entered the neighborhood** (the cycle entrance).

Two detectives work together:
- **Detective 1** starts at the original starting point (head of the list).
- **Detective 2** starts at a point we already know is **inside the loop** — the spot where our fast/slow runners met in Phase 1.

Both detectives walk at the **same pace**, one step at a time. Here's the wonderful thing: **they will meet for the first time at the cycle entrance.**

Why? Because of a mathematical identity (proved below): the distance from head to cycle-entrance is the same as the distance from the meeting point to cycle-entrance (when measured along the cycle). So both detectives, starting from their respective points and walking in sync, arrive at the entrance at the exact same step.

Two phases in code:
1. **Phase 1:** Detect cycle (same as Template 1). Remember the **meeting point**.
2. **Phase 2:** Reset one pointer to `head`. Move BOTH at 1 step/turn. Where they meet = cycle start.

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            // Phase 2: find cycle start
            slow = head;
            while (slow != fast) {
                slow = slow.next;
                fast = fast.next;
            }
            return slow;
        }
    }
    return null;
}
```

### Why Phase 2 Works (The Magic Math)

**Example list:**
```
1 → 2 → 3 → 4 → 5 → 6 → 7
        ↑               |
        └───────────────┘
```
- Tail length **L = 2** (nodes before cycle start)
- Cycle length **C = 5** (nodes 3, 4, 5, 6, 7)
- Cycle starts at node **3**

**Phase 1 trace:**

| Step | slow | fast |
|:----:|:----:|:----:|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| 3 | 4 | 7 |
| 4 | 5 | 4 (7→3→4) |
| 5 | 6 | 6 (4→5→6) |

They meet at node **6**. That's `k = 3` steps into the cycle from the cycle start.

**Phase 2 trace:**

Reset slow to head (node 1). Fast stays at node 6.

| Step | slow | fast |
|:----:|:----:|:----:|
| 0 | 1 | 6 |
| 1 | 2 | 7 |
| 2 | **3** | **3** (7→3) |

They meet at node 3 — the cycle start. ✓

**The math:**

When slow and fast meet in Phase 1:
- `slow` walked `L + k`
- `fast` walked `2(L + k)` but also `L + k + n*C` (some loops around)

Setting equal:
```
2(L + k) = L + k + n*C
L + k    = n*C
L        = n*C − k
```

This means:
- Walking **L steps from head** → lands at cycle start.
- Walking **L steps from meeting point** → walks `(C − k)` to complete the cycle, then `(n−1)` more full loops = total `L`. Also lands at cycle start.

Both pointers moving 1 step/turn → they arrive at the cycle start together.

### The "Same Forward Distance" Clarification

The two pointers aren't equidistant in some abstract sense — they have the same **forward distance** to reach the cycle start:

- From `head`: forward distance to cycle start = **L**
- From meeting point: forward distance to cycle start = **C − k** (complete what's left of the current loop) — or `nC − k` if you want to loop extra times

The identity **`L = nC − k`** literally says "these two forward distances are equal." Same speed, same distance → arrive together.

### Why Slow Gets Reset to Head (Not Somewhere Else)

Once Phase 1 ends, we know two locations:
- `head` (where we started)
- the meeting point (inside the cycle)

To find the cycle start (a third location), the simplest approach is to walk from both known locations toward it. The math guarantees they meet at the cycle start when walking at the same pace. That's why we reset slow to head — not randomness, just the only other useful anchor we have.

### Where the Identity Is "Applied" in the Code

It's **not** an explicit line. It's the **reason** the Phase 2 loop terminates exactly at the cycle start:
- Loop condition: `while (slow != fast)`
- Both walk 1 step at a time
- They meet at cycle start **because** of `L = nC − k`

No code change uses the formula — the formula just guarantees the loop does the right thing.

### Interview-Ready Explanation (Rehearse This)

If an interviewer asks "why does Phase 2 work?", say this:

> "When the fast and slow pointers first meet, slow has walked `L + k` steps — where `L` is the tail length and `k` is how far into the cycle we are from its start. Fast has walked exactly twice that: `2(L + k)`. But fast also traveled some whole number of cycles, so `2(L + k) = L + k + n·C`, which simplifies to `L = n·C − k`. That equation tells me the distance from head to the cycle start is the same as the forward distance from the meeting point to the cycle start — so if I walk from both at the same speed, they arrive at the cycle start together."

4 sentences. Draw the list, label L, k, C. Done.

### One-sentence Backup (If You Blank)

> "Head and the meeting point are equidistant (in forward direction) from the cycle start — so two pointers walking in sync from those positions meet exactly at the cycle start."

---

## Pattern Variants Summary

| Problem | Slow/Fast Speed | What You Return |
|---------|----------------|-----------------|
| Cycle detection | 1x / 2x | `true`/`false` |
| Middle of list | 1x / 2x | `slow` when fast ends |
| Cycle start (Floyd's) | 1x / 2x + Phase 2 | `slow` after Phase 2 |
| Nth from end | `slow` follows `fast` N steps behind | `slow` when fast ends |
| Happy number (LC 202) | 1x / 2x over `digitSquareSum` | check if cycle reaches 1 |

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| HashSet (brute force) | O(n) | O(n) |
| Fast & Slow | O(n) | O(1) |

Fast & Slow is always preferred when it applies.

---

## Common Pitfalls

- **NPE on `fast.next.next`** — always check `fast != null && fast.next != null` in the loop condition.
- **Checking equality BEFORE moving** — false positive at step 0 (both at head). Move first, check after.
- **Forgetting to handle empty list** (`head == null`) — the `fast != null` check covers this.
- **Returning wrong type for "no cycle"** — return `null`, not a sentinel node.
- **Confusing which middle to return** for even-length lists (LC 876 wants the second middle).
- **Using `HashSet<Integer>` to track "seen nodes"** — wrong, because duplicate values would trigger false cycles. Must use `HashSet<ListNode>` for reference equality.

---

## Problems Solved

### 1. Linked List Cycle (LC 141)

- **Mental Model:** Two runners on a track. If the track has a loop, the fast runner eventually laps the slow one.
- **Recognize it when:** detect whether a linked list has a cycle
- **Key takeaway:** O(1) space beats O(n) HashSet. Null checks on both `fast` and `fast.next` are critical.

### 2. Middle of the Linked List (LC 876)

- **Mental Model:** Fast runs twice as fast. When fast finishes the track, slow is exactly halfway.
- **Recognize it when:** find the middle without knowing the length
- **Key takeaway:** Same template as cycle detection, just walk until end.

### 3. Linked List Cycle II (LC 142)

- **Mental Model:** Two-phase detective work. Phase 1: tortoise-hare to find any meeting point inside the loop. Phase 2: reset slow to head, both walk at 1x — they meet at the cycle entrance (by the `L = n*C − k` identity).
- **Recognize it when:** find the node where a cycle begins
- **Key takeaway:** Phase 2 is Floyd's magic — trust the math: walking L from head = walking L from meeting point (both land at cycle start).

---

### Summary Table

| # | Problem | Speed Trick | One-line Mental Model |
|---|---------|-------------|-----------------------|
| 1 | Linked List Cycle (LC 141) | 1x / 2x | Fast laps slow if there's a loop |
| 2 | Middle of the Linked List (LC 876) | 1x / 2x | Fast at end → slow at midpoint |
| 3 | Linked List Cycle II (LC 142) | 1x / 2x + Phase 2 | Phase 1 meet, reset slow to head, walk 1x each — meet at cycle start |

---

## Mental Checklist When You See a Problem

1. Is it a **linked list** or a structure where you can only walk forward?
2. Do I need to detect a **cycle**?
3. Do I need the **middle**, or **kth from end**, or something I'd otherwise need the length for?
4. Can I avoid O(n) extra space?

If YES to any → fast & slow pointers.