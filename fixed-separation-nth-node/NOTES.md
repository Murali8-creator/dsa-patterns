# Fixed Separation (Nth Node from End)

## When to Recognize It

You see this pattern when:
- Working with a **linked list** (or any forward-walking structure)
- You need to find/remove/access **the Kth node from the end**
- You want a **single-pass** solution (no length counting)

**Clue words:** "Nth from end", "Kth to last", "remove last K", "swap last K"

---

## Core Intuition

Place two pointers at the same start. Move ONE pointer ahead by exactly **K steps**. Then walk **both at the same speed** until the lead pointer reaches the end.

When the lead is at the end, the trailing pointer is exactly **K nodes from the end**.

### Analogy: Two friends on a leash

Two friends start at the same spot. The first friend gets a head start of K steps. Then they walk at the same pace, side by side (K apart).

When the first friend hits the wall (end of list), the second friend is exactly K steps behind — i.e., K from the wall.

The gap is **fixed** the whole time, unlike Floyd's tortoise/hare where the gap *grows*.

---

## Difference from Fast & Slow Pointers

| | Fast & Slow (Floyd's) | Fixed Separation |
|---|---|---|
| Start | Both at head | Both at head (or dummy) |
| Speed | Different (1x, 2x) | **Same** (1x each) |
| Gap | Grows over time | **Constant** (K) |
| Used for | Cycles, middle | Kth from end |

---

## The Dummy Node Trick (Critical)

The Nth node from the end could be the **head** itself (when N == length). Without a dummy, removing the head is a special case.

**Solution:** start with a dummy node before head.

```
dummy → head → ... → tail
```

Set both pointers at the **dummy**, not at head. Advance the lead K steps. Walk both. Slow ends up at the node **just before** the one to remove (could even be the dummy itself, if removing the head).

Then `slow.next = slow.next.next` works uniformly — no special case.

Return `dummy.next` (the possibly-new head).

**Without the dummy:** you'd need an `if` to handle the "head removal" case separately. The dummy eliminates this.

---

## Canonical Template — Remove Nth From End (LC 19)

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(-1);
    dummy.next = head;

    ListNode fast = dummy;
    ListNode slow = dummy;

    // 1. Advance fast n steps ahead
    for (int i = 0; i < n; i++) {
        fast = fast.next;
    }

    // 2. Walk both at the same speed until fast.next is null
    while (fast.next != null) {
        fast = fast.next;
        slow = slow.next;
    }

    // 3. slow.next is the node to remove
    slow.next = slow.next.next;

    // 4. Return new head (might equal original head, or be head.next)
    return dummy.next;
}
```

### Why `while (fast.next != null)` and not `while (fast != null)`?

We want slow to end up at the node **before** the one to remove (so we can do `slow.next = slow.next.next`). Stopping when `fast.next == null` means fast is at the last node, and slow is at the (N+1)th-from-last — which is exactly the node before the Nth-from-last.

If we used `while (fast != null)`, fast would walk off the end (to null), and slow would be one step too far.

---

## Edge Cases — Why the Dummy Saves You

### Case 1: Remove the only node — `head = [1], n = 1`

- dummy → 1
- fast advances 1 step → at node 1
- `fast.next == null` → outer loop doesn't run
- slow still at dummy
- `slow.next = slow.next.next` → dummy.next = null
- Return dummy.next = null → `[]` ✓

### Case 2: Remove the head — `head = [1, 2], n = 2`

- dummy → 1 → 2
- fast advances 2 steps → at node 2
- `fast.next == null` → loop doesn't run
- slow still at dummy
- dummy.next = node 2
- Return dummy.next = node 2 → `[2]` ✓

### Case 3: Remove a middle node — `head = [1, 2, 3, 4, 5], n = 2`

- dummy → 1 → 2 → 3 → 4 → 5
- fast advances 2 steps → at node 2
- Walk both: fast=3 slow=1, fast=4 slow=2, fast=5 slow=3
- slow at node 3 → set slow.next = node 5 → `[1, 2, 3, 5]` ✓

### Case 4: Remove the tail — `head = [1, 2, 3], n = 1`

- dummy → 1 → 2 → 3
- fast advances 1 step → at node 1
- Walk both: fast=2 slow=1, fast=3 slow=2
- slow at node 2 → set slow.next = null → `[1, 2]` ✓

---

## Common Pitfalls

- **No dummy node** — forces a special case for "remove head," adding complexity and bugs.
- **`while (fast != null)` instead of `while (fast.next != null)`** — slow ends up one step too far.
- **Returning `head` instead of `dummy.next`** — if the original head was removed, you'd return a dangling pointer.
- **Counting length first (two passes)** — works but defeats the point; the pattern is single-pass.
- **Forgetting that the gap is `n`, not `n + 1`** — start fast `n` steps ahead, not `n + 1`. The `dummy` shifts everything by one, which is exactly what we want.

---

## Generalizations

The fixed-separation trick isn't only for "Nth from end." It works whenever you need:

- **A trailing pointer K steps behind a leading pointer**
- **A node that's K positions from a reference point**

Variations:
- **Find the Kth node from the end** (don't remove, just return it)
- **Swap two nodes at specific positions** in one pass
- **Split a list at the Kth from end** position
- **Detect/remove a "tail segment" of length K**

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Two-pass (count length first) | O(n) | O(1) |
| **One-pass fixed separation** | **O(n)** | **O(1)** |

Both are linear time and constant space. The one-pass version is preferred for elegance and works when length isn't known upfront.

---

## Problems Solved

### 1. Remove Nth Node From End of List (LC 19)

- **Mental Model:** Two friends with a K-step leash. The first friend hits the end → the second friend is K behind, exactly where we need to operate.
- **Recognize it when:** "Nth from end" of a linked list, single pass desired
- **Key takeaway:** Use a **dummy node** to make head-removal uniform with mid-list removal. Move fast N steps first, then walk both at 1x. Stop when `fast.next == null` — slow is at the node *before* the one to remove.
- **Test cases:**
  - `head = [1, 2, 3, 4, 5], n = 2` → `[1, 2, 3, 5]` (remove `4`)
  - `head = [1], n = 1` → `[]`
  - `head = [1, 2], n = 1` → `[1]`
  - `head = [1, 2], n = 2` → `[2]` (remove first — dummy handles this without a special case)

---

### Summary Table

| # | Problem | Trick | One-line Mental Model |
|---|---------|-------|-----------------------|
| 1 | Remove Nth Node From End (LC 19) | Dummy node + fixed gap | Two friends with a K-step leash; when first hits the wall, second is at the target |

---

## Mental Checklist When You See a Problem

1. Is it a **linked list** problem about "Nth from end" / "K to last"?
2. Do I need **one pass** (no length counting)?
3. Could the Nth node be the head? (If yes → **dummy node**)

If YES to all → fixed separation. Move fast N steps, then walk both together, dummy node to absorb edge cases.

---

## The Bigger Idea

Fast & slow with **different speeds** detects cycles and finds middles.
Fast & slow with **the same speed and a fixed gap** indexes from the end.

Same family of pattern, different parameter. Once you see this, "Nth from end" stops feeling like a special trick and becomes obvious.