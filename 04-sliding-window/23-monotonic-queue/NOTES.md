# Monotonic Queue (Max/Min in a Sliding Window)

## When to Recognize It

You see this pattern when:
- You need the **max or min of every window** as it slides
- You need **max−min of a window** (range constraint)
- Naive "rescan the window each time" would be O(n × k)
- You want **O(n)**

**Clue words:** "maximum/minimum in each window of size k", "max sliding window", "absolute difference of any two ≤ limit", "longest subarray where max−min ≤ X"

---

## Core Intuition

A **monotonic deque** keeps only the "still-relevant candidates" for the answer, in sorted order. Useless elements are pruned eagerly, so the front of the deque is always the answer (max or min) for the current window.

### Why elements become "useless"

Consider tracking the **max**. If a new element `x` enters and there's a smaller element `y` already sitting behind it in the deque:
- As long as `x` is in the window, `y` can **never** be the max (x dominates it AND x will stay in the window longer, since it arrived later).
- So `y` is useless — evict it.

This keeps the deque **monotonically decreasing** (front = biggest). For min, keep it **monotonically increasing** (front = smallest).

### Analogy: A queue of job candidates

People line up holding numbers (their "score"). Rules:
1. A new person joins the back. Before they settle, everyone behind them with a **lower score** is kicked out — they'll never win while this stronger person is in line.
2. People at the front leave when they're "too old" (their index slid out of the window).
3. The front of the line is always the current champion (max).

---

## Why Store Indices, Not Values

The deque holds **indices**, not the values themselves. Why?

- To check if the front element is **still in the window**, you compare its index against the window's left boundary (`j - k + 1`). You can't do that with just a value.
- You read the value when needed via `nums[deque.peekFirst()]`.

---

## The Magic: Why It's O(n)

Each element is:
- **Added** to the deque exactly once (`addLast`)
- **Removed** at most once — either from the **front** (slid out of window) or the **back** (dominated by a newer, better element)

Total deque operations across the whole run = **2n**. Amortized **O(1) per element**, total **O(n)**.

The deque "self-prunes," so its size stays small and every operation is cheap.

---

## Template 1 — Sliding Window Maximum (LC 239)

Fixed window of size `k`. Return the max of each window.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    Deque<Integer> dq = new ArrayDeque<>();   // stores indices, decreasing by value
    int[] res = new int[n - k + 1];

    for (int j = 0; j < n; j++) {
        // 1. Evict front if it slid out of the window
        while (!dq.isEmpty() && dq.peekFirst() < j - k + 1) {
            dq.removeFirst();
        }

        // 2. Evict smaller elements from the back (they're dominated)
        while (!dq.isEmpty() && nums[dq.peekLast()] < nums[j]) {
            dq.removeLast();
        }

        // 3. Add current index
        dq.addLast(j);

        // 4. Record the max once the window is full
        if (j >= k - 1) {
            res[j - k + 1] = nums[dq.peekFirst()];
        }
    }
    return res;
}
```

### The 4 steps (memorize this order)

1. **Evict front** — remove indices that fell out of the window (`< j - k + 1`)
2. **Evict back** — remove dominated elements (smaller than the incoming, for max-queue)
3. **Add** — push the current index to the back
4. **Record** — front of deque is the window max, once `j >= k - 1`

For a **min**-queue, flip step 2's comparison: evict elements that are **larger** than the incoming.

### Why `j >= k - 1`?

The window only has `k` elements once `j` reaches index `k - 1`. Before that, the window is partial — nothing to record. `res[j - k + 1]` is the position of the current window's answer in the output array.

---

## Template 2 — Two Deques: Longest Subarray with max−min ≤ limit (LC 1438)

Combines a max-deque AND a min-deque with a **variable-size** window.

### The reframing (the hard insight)

> "Absolute difference between **any two** elements ≤ limit" ⟺ "**max − min** of the window ≤ limit"

If the biggest and smallest are within `limit`, every pair is automatically within `limit`. So you only need to track max and min — that's two monotonic deques.

```java
public int longestSubarray(int[] nums, int limit) {
    Deque<Integer> maxDq = new ArrayDeque<>();  // decreasing — front is max
    Deque<Integer> minDq = new ArrayDeque<>();  // increasing — front is min
    int res = 0, i = 0;

    for (int j = 0; j < nums.length; j++) {
        // push j into max-deque (evict smaller from back)
        while (!maxDq.isEmpty() && nums[maxDq.peekLast()] < nums[j]) maxDq.removeLast();
        maxDq.addLast(j);

        // push j into min-deque (evict larger from back)
        while (!minDq.isEmpty() && nums[minDq.peekLast()] > nums[j]) minDq.removeLast();
        minDq.addLast(j);

        // shrink window while invariant (max - min <= limit) is violated
        while (nums[maxDq.peekFirst()] - nums[minDq.peekFirst()] > limit) {
            if (maxDq.peekFirst() == i) maxDq.removeFirst();
            if (minDq.peekFirst() == i) minDq.removeFirst();
            i++;
        }

        res = Math.max(res, j - i + 1);
    }
    return res;
}
```

### Key points

- **Two deques, opposite monotonicity** — maxDq decreasing, minDq increasing.
- **Variable-size shrink** — this is Flavor 1 (longest-with-X) from sub-pattern 21. Shrink while invariant broken, record after.
- **Conditional front eviction** — when `i` advances, evict from a deque's front **only if** the leaving index `i` is actually at that deque's front. Otherwise it was already pruned from the back earlier.
- **Recompute max−min inside the shrink loop** — the window changes each shrink step, so the difference must be re-read each iteration.

---

## Common Pitfalls

- **Storing values instead of indices** — you can't check "is the front still in the window?" without the index.
- **Comparing indices when you mean values** — `dq.peekFirst()` is an index; wrap it in `nums[...]` to get the value.
- **Using `dq.remove(i)` (the `Object` overload) to evict** — it does an O(n) linear scan, dropping you back to O(n × k). Use `removeFirst()` (O(1)) and let the deque manage the window boundary.
- **Forgetting to recompute the diff inside the shrink loop** (LC 1438) — stale `diff` means the loop never re-checks the shrunken window.
- **Evicting from the front unconditionally** — only evict if the leaving index equals the deque's front. Otherwise that index is already gone.
- **Wrong comparison direction** — max-queue evicts smaller from the back; min-queue evicts larger. Mixing these up gives wrong answers.
- **Recording before the window is full** (LC 239) — guard with `j >= k - 1`.

---

## Complexity

| Approach | Time | Space |
|----------|:----:|:-----:|
| Brute force (rescan each window) | O(n × k) | O(1) |
| Heap (priority queue) | O(n log n) | O(n) |
| **Monotonic deque** | **O(n)** | **O(k)** |

The deque beats the heap because it prunes dominated elements entirely, rather than carrying them around in a heap.

---

## Problems Solved

### 1. Sliding Window Maximum (LC 239)

- **Mental Model:** A line of candidates; the front is always the current champion. New arrivals kick out weaker people behind them; the champion steps aside when too old (out of window).
- **Recognize it when:** max (or min) of every fixed-size window
- **Key takeaway:** Store indices. 4 steps in order: evict front (out of window) → evict back (dominated) → add → record. O(n) because each element enters/exits the deque at most once.
- **Test case:** `nums = [1,3,-1,-3,5,3,6,7], k = 3` → `[3,3,5,5,6,7]`

### 2. Longest Continuous Subarray With Absolute Diff ≤ Limit (LC 1438)

- **Mental Model:** Two champions tracked at once — the biggest and the smallest in the window. If they're within `limit`, the window is valid. Grow greedily; shrink when the spread is too big.
- **Recognize it when:** "any two elements differ by at most X" / "longest where max−min ≤ limit"
- **Key takeaway:** Reframe "any two ≤ limit" as "max − min ≤ limit." Use TWO deques (one max, one min) plus the variable-size template. Recompute max−min each shrink step; evict fronts conditionally on the leaving index.
- **Test cases:**
  - `nums = [8,2,4,7], limit = 4` → `2` (subarray `[2,4]` or `[4,7]`)
  - `nums = [10,1,2,4,7,2], limit = 5` → `4` (subarray `[2,4,7,2]`, max 7 − min 2 = 5)

---

### Summary Table

| # | Problem | Deques | Window | One-line Mental Model |
|---|---------|--------|--------|-----------------------|
| 1 | Sliding Window Maximum (LC 239) | 1 (max) | fixed `k` | Front of the candidate line is the champion |
| 2 | Longest Subarray, max−min ≤ limit (LC 1438) | 2 (max + min) | variable | Two champions; valid if they're within limit |

---

## Mental Checklist When You See a Problem

1. Do I need the **max or min of a window** repeatedly as it slides?
2. Do I need **max − min** (a range/spread constraint)?
3. Is a naive solution O(n × k) and I want O(n)?

If YES → monotonic deque. One deque for max-or-min; two deques if you need both (range constraints).

---

## The Pattern in One Sentence

> Keep a deque of indices whose values are monotonic; evict the front when it slides out of the window and evict the back when a new element dominates it — so the front is always the window's max (or min) in O(1).