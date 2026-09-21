# 🧩 Arrays — Full DSA Interview Revision

**Mission:** be able to open *any* array problem in a live DSA round, name its pattern within 30 seconds, and write correct, panic-free Rust the first time.

Here's the secret interviewers won't tell you: arrays only really have **8 tricks**. Almost every "new" array problem you'll ever see is one of these tricks wearing a costume — a two-pointer problem dressed up as "container with most water," a sliding window dressed up as "longest substring." Learn the 8 tricks once, and you stop seeing 500 different LeetCode problems and start seeing 8 familiar friends. That's what this file trains.

**How to use this file:**
- **Night before an interview:** read [§9 Speed Read](#9-5-minute-speed-read) only.
- **Learning a new pattern:** read the section, the analogy, then type the code by hand without copy/paste.
- **After every mock/LeetCode session:** append a dated entry to [§10 Dated Log](#10-dated-revision-log) with whatever tripped you up.

This file assumes you've read the Rust-specific landmines in [`../README.md` Part 1](../README.md#part-1-the-rust-mental-model) (indexing with `usize`, mutability, move semantics, etc). Everything here is array-specific.

---

## Table of Contents
1. [Pattern Recognition Map](#1-pattern-recognition-map)
2. [Two Pointers](#2-two-pointers)
3. [Sliding Window](#3-sliding-window)
4. [Prefix Sum / Difference Array](#4-prefix-sum--difference-array)
5. [Kadane's Algorithm (Max Subarray Family)](#5-kadanes-algorithm-max-subarray-family)
6. [Binary Search on Arrays](#6-binary-search-on-arrays)
7. [Sorting-Based Patterns](#7-sorting-based-patterns)
8. [In-Place Rearrangement Tricks](#8-in-place-rearrangement-tricks)
9. [5-Minute Speed Read](#9-5-minute-speed-read)
10. [Dated Revision Log](#10-dated-revision-log)

Cross-references to structures/algorithms that overlap arrays but live in their own files once added: `hashing.md` (frequency maps), `two-pointers-sliding-window.md` if split out further, `bitwise.md` (XOR tricks), `heaps.md` (top-K), `stacks.md` (monotonic stack), `dynamic-programming.md` (subarray/subsequence DP). See [§0 index rule](../README.md#how-new-topics-get-added).

---

## 1. Pattern Recognition Map

The single most valuable skill in an array round is mapping the *problem smell* to a *pattern* before you write a line of code.

| Problem smell | Pattern | Section |
|---|---|---|
| "pair/triplet that sums to X", sorted array, opposite ends closing in | Two pointers (converging) | [§2](#2-two-pointers) |
| "remove duplicates in place", "partition array", slow/fast same direction | Two pointers (same direction / read-write) | [§2](#2-two-pointers) |
| "longest/shortest subarray with condition", contiguous + variable length | Sliding window | [§3](#3-sliding-window) |
| "subarray sum equals K", "range sum queries", "equal sum split" | Prefix sum | [§4](#4-prefix-sum--difference-array) |
| "max subarray sum", "max/min contiguous product" | Kadane's / running best | [§5](#5-kadanes-algorithm-max-subarray-family) |
| Sorted (or rotated-sorted) array, need O(log N) | Binary search variants | [§6](#6-binary-search-on-arrays) |
| "k-th largest/smallest", frequency-based ordering | Sort or heap (heap → `heaps.md` once added) | [§7](#7-sorting-based-patterns) |
| "sort 0s/1s/2s", "move negatives to one side" | Dutch National Flag / partitioning | [§7](#7-sorting-based-patterns), [§8](#8-in-place-rearrangement-tricks) |
| "find missing/duplicate number in `1..n`" | Cyclic sort / index-as-hashmap / XOR / Floyd's | [§8](#8-in-place-rearrangement-tricks) |
| "rotate array by k" | Reversal trick or juggling | [§8](#8-in-place-rearrangement-tricks) |
| "next permutation", "spiral/diagonal traversal" | Simulation with careful bounds | [§8](#8-in-place-rearrangement-tricks) |
| 2D grid, "search a matrix", "set matrix zeroes" | Treat rows/cols as 1D arrays + in-place markers | [§8](#8-in-place-rearrangement-tricks) |

---

## 2. Two Pointers

**Where you'll meet it:** any time you catch yourself about to write a nested loop over a *sorted* array ("for every i, for every j, check pair i,j..."), stop — that's an O(N²) alarm bell, and two pointers is almost always the O(N) fix.

**The analogy:** two people start at opposite ends of a sorted bookshelf looking for two books whose page counts add up to a target. If the sum is too big, the person on the right steps in (fewer pages). If too small, the person on the left steps in (more pages). They can never pass each other, so the search space shrinks every step, O(N) instead of O(N²).

### 2.1 Converging pointers (sorted array)

```rust
/// Two Sum on a SORTED array — O(N) time, O(1) space
/// (For unsorted arrays, prefer the HashMap two-sum in README Part 3.6, O(N) time O(N) space)
pub fn two_sum_sorted(nums: &[i32], target: i32) -> Option<(usize, usize)> {
    if nums.len() < 2 {
        return None;
    }
    let (mut left, mut right) = (0, nums.len() - 1);

    while left < right {
        let sum = nums[left] + nums[right];
        match sum.cmp(&target) {
            std::cmp::Ordering::Equal => return Some((left, right)),
            std::cmp::Ordering::Less => left += 1,
            std::cmp::Ordering::Greater => right -= 1,
        }
    }
    None
}
```

**Three Sum** (classic extension — fix one pointer, two-sum the rest, skip duplicates):

```rust
pub fn three_sum(nums: &mut [i32]) -> Vec<[i32; 3]> {
    nums.sort_unstable();
    let n = nums.len();
    let mut result = Vec::new();

    for i in 0..n {
        // Skip duplicate anchors
        if i > 0 && nums[i] == nums[i - 1] {
            continue;
        }
        if nums[i] > 0 {
            break; // sorted ascending, no triplet can sum to 0 from here on
        }

        let (mut left, mut right) = (i + 1, n.saturating_sub(1));
        while left < right {
            let sum = nums[i] + nums[left] + nums[right];
            if sum == 0 {
                result.push([nums[i], nums[left], nums[right]]);
                left += 1;
                right -= 1;
                // Skip duplicate pointer positions
                while left < right && nums[left] == nums[left - 1] {
                    left += 1;
                }
                while left < right && nums[right] == nums[right + 1] {
                    right -= 1;
                }
            } else if sum < 0 {
                left += 1;
            } else {
                right -= 1;
            }
        }
    }
    result
}
```

### 2.2 Same-direction pointers (read/write, in place)

**The analogy:** a slow "write head" and a fast "read head" moving through the same tape. The read head scans everything; the write head only advances when it finds something worth keeping.

```rust
/// Remove duplicates from a SORTED array in place, return new length.
/// Time O(N), Space O(1).
pub fn remove_duplicates_sorted(nums: &mut Vec<i32>) -> usize {
    if nums.is_empty() {
        return 0;
    }
    let mut write = 1;
    for read in 1..nums.len() {
        if nums[read] != nums[write - 1] {
            nums[write] = nums[read];
            write += 1;
        }
    }
    nums.truncate(write);
    write
}
```

**Container With Most Water** (converging pointers, greedy elimination):

```rust
/// The shorter wall always limits the area, so it's always safe to move
/// the shorter pointer inward — moving the taller one can never help.
pub fn max_area(heights: &[i32]) -> i32 {
    let (mut left, mut right) = (0usize, heights.len() - 1);
    let mut best = 0;

    while left < right {
        let width = (right - left) as i32;
        let area = width * heights[left].min(heights[right]);
        best = best.max(area);

        if heights[left] < heights[right] {
            left += 1;
        } else {
            right -= 1;
        }
    }
    best
}
```

---

## 3. Sliding Window

**Where you'll meet it:** the phrase "contiguous subarray/substring" plus "longest", "shortest", or "count of" is basically a neon sign pointing at this section. It's two pointers' close cousin — same O(N) trick, but both pointers now move in the *same* direction instead of converging.

**The analogy:** a rubber band stretched over the array. The right edge always stretches forward to include new elements. The left edge only snaps forward when the window breaks its rule (too many distinct chars, sum too big, etc). Every element is added once and removed at most once → O(N), not O(N²).

### 3.1 Fixed-size window

```rust
/// Max sum of any contiguous subarray of exactly size k. O(N) time, O(1) space.
pub fn max_sum_fixed_window(nums: &[i32], k: usize) -> Option<i32> {
    if nums.len() < k || k == 0 {
        return None;
    }
    let mut window_sum: i32 = nums[..k].iter().sum();
    let mut best = window_sum;

    for i in k..nums.len() {
        window_sum += nums[i] - nums[i - k]; // add new right edge, drop old left edge
        best = best.max(window_sum);
    }
    Some(best)
}
```

### 3.2 Variable-size window (shrink on violation)

```rust
/// Smallest subarray length with sum >= target. O(N) time, O(1) space.
/// (Requires non-negative numbers — negative values break window monotonicity.)
pub fn min_subarray_len(target: i32, nums: &[i32]) -> usize {
    let mut window_start = 0;
    let mut window_sum = 0;
    let mut best_len = usize::MAX;

    for window_end in 0..nums.len() {
        window_sum += nums[window_end];

        // Shrink from the left while the window still satisfies the condition
        while window_sum >= target {
            best_len = best_len.min(window_end - window_start + 1);
            window_sum -= nums[window_start];
            window_start += 1;
        }
    }

    if best_len == usize::MAX { 0 } else { best_len }
}
```

### 3.3 Variable-size window with a frequency map

See `length_of_longest_substring` in [`../README.md` §3.2](../README.md#32-sliding-window-longest-substring-without-repeats) — same skeleton, `HashMap<char, usize>` swapped for `HashMap<i32, usize>` handles "longest subarray with at most K distinct values" style problems identically.

---

## 4. Prefix Sum / Difference Array

**Where you'll meet it:** "sum of range [l, r]" asked *many times*, or "subarray sum equals K" — the moment you see repeated range-sum questions, doing a fresh loop each time is the trap; pay the O(N) cost once up front instead.

**The concept:** precompute cumulative sums once (O(N)) so any range-sum query afterward is O(1) instead of O(N) per query. Think of it like a running odometer in a car — instead of measuring the distance between mile marker 12 and mile marker 47 by driving it again, you just subtract the two odometer readings you already logged.

```
prefix[i] = nums[0] + nums[1] + ... + nums[i-1]
range_sum(l, r) = prefix[r+1] - prefix[l]   // inclusive [l, r]
```

```rust
pub struct PrefixSum {
    prefix: Vec<i64>, // i64 guards against overflow on large arrays
}

impl PrefixSum {
    pub fn new(nums: &[i32]) -> Self {
        let mut prefix = Vec::with_capacity(nums.len() + 1);
        prefix.push(0);
        for &n in nums {
            prefix.push(prefix.last().unwrap() + n as i64);
        }
        Self { prefix }
    }

    /// Inclusive range sum [left, right]
    pub fn range_sum(&self, left: usize, right: usize) -> i64 {
        self.prefix[right + 1] - self.prefix[left]
    }
}
```

**Subarray Sum Equals K** — the prefix-sum + HashMap combo, one of the most tested array patterns:

```rust
use std::collections::HashMap;

/// Count of contiguous subarrays summing to k. O(N) time, O(N) space.
/// Key idea: if prefix[j] - prefix[i] == k, subarray (i, j] sums to k.
/// So for each running prefix, check how many earlier prefixes equal (prefix - k).
pub fn subarray_sum_equals_k(nums: &[i32], k: i32) -> i32 {
    let mut seen: HashMap<i32, i32> = HashMap::new();
    seen.insert(0, 1); // empty prefix, handles subarrays starting at index 0
    let mut running_sum = 0;
    let mut count = 0;

    for &num in nums {
        running_sum += num;
        if let Some(&freq) = seen.get(&(running_sum - k)) {
            count += freq;
        }
        *seen.entry(running_sum).or_insert(0) += 1;
    }
    count
}
```

**Difference array** — for "apply +v to range [l, r] many times, then read final array":

```rust
/// Apply range-increment updates in O(1) each, materialize once at the end in O(N).
pub fn apply_range_updates(n: usize, updates: &[(usize, usize, i32)]) -> Vec<i32> {
    let mut diff = vec![0i32; n + 1];
    for &(l, r, val) in updates {
        diff[l] += val;
        diff[r + 1] -= val; // cancel the effect right after r
    }

    let mut result = vec![0i32; n];
    let mut running = 0;
    for i in 0..n {
        running += diff[i];
        result[i] = running;
    }
    result
}
```

Even/pivot-index prefix-sum trick (with the negative-number binary-search trap explained) lives in [`../README.md` §3.5](../README.md#35-prefix-sum--pivot-index).

---

## 5. Kadane's Algorithm (Max Subarray Family)

**Where you'll meet it:** "maximum sum/product of a contiguous subarray," or anything that smells like "best run of consecutive good decisions." It's dynamic programming in disguise — but the state is so small (just "current" and "best") that it collapses into a one-pass loop.

**The analogy:** you're carrying a running total on a hike. The moment your running total drops below zero, it can only drag future sums down, so you drop your pack and start fresh from the next step.

```rust
/// Maximum sum of any contiguous subarray. O(N) time, O(1) space.
pub fn max_subarray(nums: &[i32]) -> i32 {
    let mut current = nums[0];
    let mut best = nums[0];

    for &num in &nums[1..] {
        // Either extend the previous subarray, or start fresh at `num`
        current = num.max(current + num);
        best = best.max(current);
    }
    best
}
```

**Max Product Subarray** — the twist: a negative number can flip the smallest product into the largest, so track *both* running max and running min.

```rust
pub fn max_product_subarray(nums: &[i32]) -> i32 {
    let mut current_max = nums[0];
    let mut current_min = nums[0];
    let mut best = nums[0];

    for &num in &nums[1..] {
        if num < 0 {
            std::mem::swap(&mut current_max, &mut current_min);
        }
        current_max = num.max(current_max * num);
        current_min = num.min(current_min * num);
        best = best.max(current_max);
    }
    best
}
```

Also see [`../README.md` §3.3](../README.md#33-one-pass-running-minmax-tracking) — Best Time to Buy/Sell Stock is the same "carry running best forward" family, one pass, no nested loops.

---

## 6. Binary Search on Arrays

**Where you'll meet it:** any sorted array, obviously — but also the sneaky version where the *array* isn't sorted but the *answer space* is monotonic ("minimum X such that Y holds"). If halving the search space every step is even remotely possible, this is your O(log N) weapon.

Requires a sorted (or sorted-with-a-twist, e.g. rotated) array to get O(log N).

```rust
/// Classic binary search, returns the index if found.
pub fn binary_search(arr: &[i32], target: i32) -> Option<usize> {
    let (mut lo, mut hi) = (0isize, arr.len() as isize - 1);

    while lo <= hi {
        let mid = lo + (hi - lo) / 2; // avoids overflow vs (lo + hi) / 2
        let mid_val = arr[mid as usize];

        if mid_val == target {
            return Some(mid as usize);
        } else if mid_val < target {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    None
}
```

**Search in Rotated Sorted Array** — the key insight: at least one half of `[lo, mid]` / `[mid, hi]` is always properly sorted; check which half is sorted, then decide which side to search.

```rust
pub fn search_rotated(nums: &[i32], target: i32) -> Option<usize> {
    let (mut lo, mut hi) = (0isize, nums.len() as isize - 1);

    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        let mid_val = nums[mid as usize];

        if mid_val == target {
            return Some(mid as usize);
        }

        if nums[lo as usize] <= mid_val {
            // Left half [lo, mid] is sorted
            if nums[lo as usize] <= target && target < mid_val {
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            // Right half [mid, hi] is sorted
            if mid_val < target && target <= nums[hi as usize] {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
    }
    None
}
```

**Find First/Last Position** (lower/upper bound) — use Rust's built-in `partition_point` (see [`../README.md` §3.8](../README.md#38-search-patterns)) instead of hand-rolling it:

```rust
pub fn find_first_and_last(nums: &[i32], target: i32) -> (Option<usize>, Option<usize>) {
    let first = nums.partition_point(|&x| x < target);
    if first >= nums.len() || nums[first] != target {
        return (None, None);
    }
    let last = nums.partition_point(|&x| x <= target) - 1;
    (Some(first), Some(last))
}
```

**Binary search on the *answer*** (not the array index) — when the problem asks "minimum capacity/speed/days such that X holds", binary search over the value range and use a greedy feasibility check as the predicate. Recognize this smell: "minimize the maximum", "maximize the minimum", answer space is monotonic even though the array itself isn't sorted.

---

## 7. Sorting-Based Patterns

**Where you'll meet it:** whenever "order doesn't matter for the input, only for the output" — if you're free to shuffle the array first, sorting often turns a hard problem into an easy scan.

See [`../README.md` §3.7](../README.md#37-sorting-based-patterns) for wave sort and merge-and-deduplicate. Additional array-specific sorting patterns:

**Dutch National Flag** (sort an array of only 0s, 1s, 2s in one O(N) pass, O(1) space — no full sort needed):

```rust
/// Three pointers: everything before `low` is 0, everything after `high` is 2,
/// the unexplored region is [low, high]. `mid` scans through it.
pub fn sort_colors(nums: &mut [i32]) {
    let (mut low, mut mid, mut high) = (0usize, 0usize, nums.len().saturating_sub(1));

    while mid <= high {
        match nums[mid] {
            0 => {
                nums.swap(low, mid);
                low += 1;
                mid += 1;
            }
            1 => mid += 1,
            2 => {
                nums.swap(mid, high);
                // Don't advance mid: the swapped-in value from `high` is unexamined
                if high == 0 { break; } // guard against usize underflow
                high -= 1;
            }
            _ => unreachable!(),
        }
    }
}
```

**Merge Intervals** (sort by start, then merge overlaps in one pass):

```rust
pub fn merge_intervals(mut intervals: Vec<[i32; 2]>) -> Vec<[i32; 2]> {
    intervals.sort_unstable_by_key(|iv| iv[0]);
    let mut merged: Vec<[i32; 2]> = Vec::with_capacity(intervals.len());

    for interval in intervals {
        match merged.last_mut() {
            Some(last) if interval[0] <= last[1] => {
                last[1] = last[1].max(interval[1]); // overlap: extend the end
            }
            _ => merged.push(interval),
        }
    }
    merged
}
```

**Kth Largest via sort** (O(N log N), fine unless the interviewer wants O(N) via quickselect or O(N log K) via a heap — heap-based selection will live in `heaps.md` once that topic file exists):

```rust
pub fn kth_largest_by_sort(nums: &mut [i32], k: usize) -> i32 {
    nums.sort_unstable_by(|a, b| b.cmp(a)); // descending
    nums[k - 1]
}
```

---

## 8. In-Place Rearrangement Tricks

**Where you'll meet it:** the interviewer adds the magic words "without using extra space" or "in place" — that's your cue that a clever index trick exists, and allocating a second array is the answer they're trying to steer you away from.

### 8.1 Rotate Array by K (reversal trick, O(N) time, O(1) space)

**The analogy:** to rotate a line of people right by k, flip the whole line, then flip each of the two resulting segments back — three flips net out to one rotation, no extra array needed.

```rust
pub fn rotate_right(nums: &mut [i32], k: usize) {
    let n = nums.len();
    if n == 0 {
        return;
    }
    let k = k % n;
    nums.reverse();
    nums[..k].reverse();
    nums[k..].reverse();
}
```

### 8.2 Cyclic Sort (values in range `1..=n`, find missing/duplicate)

**The analogy:** if every value from 1 to n should live at index `value - 1`, walk the array placing each number in its rightful seat; whatever seat is still wrong (or empty) at the end tells you the answer.

```rust
/// Find the missing number in [0, n] given n distinct numbers. O(N), O(1).
pub fn missing_number(nums: &mut [i32]) -> i32 {
    let n = nums.len() as i32;
    let mut i = 0;
    while (i as i32) < n {
        let correct = nums[i];
        // nums[i] belongs at index nums[i], unless it's the "extra" value n
        if correct < n && nums[i] != nums[correct as usize] {
            nums.swap(i, correct as usize);
        } else {
            i += 1;
        }
    }
    for i in 0..nums.len() {
        if nums[i] != i as i32 {
            return i as i32;
        }
    }
    n
}
```

**XOR trick** for the same "find the missing number 0..n" problem, O(1) space, no mutation of input:

```rust
pub fn missing_number_xor(nums: &[i32]) -> i32 {
    let n = nums.len() as i32;
    let mut result = n; // account for the index n, which has no array slot
    for (i, &num) in nums.iter().enumerate() {
        result ^= i as i32 ^ num; // a ^ a cancels; only the missing value survives
    }
    result
}
```

Duplicate detection via array-as-hashmap (negate the value at the seen index): see Floyd's Cycle Detection in [`../README.md` §3.4](../README.md#34-floyds-cycle-detection-pointer-jumping) for the pointer-jumping variant of "find the duplicate."

### 8.3 Matrix as a flattened array

**Set Matrix Zeroes in place** (use row 0 and column 0 themselves as the marker storage, O(1) extra space beyond two booleans):

```rust
pub fn set_zeroes(matrix: &mut Vec<Vec<i32>>) {
    let rows = matrix.len();
    let cols = matrix[0].len();
    let mut first_row_has_zero = false;
    let mut first_col_has_zero = false;

    for r in 0..rows {
        if matrix[r][0] == 0 {
            first_col_has_zero = true;
        }
    }
    for c in 0..cols {
        if matrix[0][c] == 0 {
            first_row_has_zero = true;
        }
    }

    // Use row 0 / col 0 as markers for the rest of the matrix
    for r in 1..rows {
        for c in 1..cols {
            if matrix[r][c] == 0 {
                matrix[r][0] = 0;
                matrix[0][c] = 0;
            }
        }
    }

    for r in 1..rows {
        for c in 1..cols {
            if matrix[r][0] == 0 || matrix[0][c] == 0 {
                matrix[r][c] = 0;
            }
        }
    }

    if first_row_has_zero {
        for c in 0..cols {
            matrix[0][c] = 0;
        }
    }
    if first_col_has_zero {
        for r in 0..rows {
            matrix[r][0] = 0;
        }
    }
}
```

**Spiral Traversal** (four shrinking boundaries — the pattern to memorize for any "traverse in a shape" matrix problem):

```rust
pub fn spiral_order(matrix: &Vec<Vec<i32>>) -> Vec<i32> {
    if matrix.is_empty() {
        return Vec::new();
    }
    let (mut top, mut bottom) = (0isize, matrix.len() as isize - 1);
    let (mut left, mut right) = (0isize, matrix[0].len() as isize - 1);
    let mut result = Vec::new();

    while top <= bottom && left <= right {
        for c in left..=right {
            result.push(matrix[top as usize][c as usize]);
        }
        top += 1;

        for r in top..=bottom {
            result.push(matrix[r as usize][right as usize]);
        }
        right -= 1;

        if top <= bottom {
            for c in (left..=right).rev() {
                result.push(matrix[bottom as usize][c as usize]);
            }
            bottom -= 1;
        }

        if left <= right {
            for r in (top..=bottom).rev() {
                result.push(matrix[r as usize][left as usize]);
            }
            left += 1;
        }
    }
    result
}
```

### 8.4 In-place matrix transform, wave sort, move-zeroes, vector relocation

Already covered in `README.md` and cross-linked here rather than duplicated:
- Flip-and-invert image (two-pointer, O(1) aux space) — [§3.1](../README.md#31-in-place-matrix-transform-two-pointer-o1-aux-space)
- Wave sort (`sort` + `step_by(2)` swap) — [§3.7](../README.md#37-sorting-based-patterns)
- Move Zeroes (two-pointer write index) — [§3.9](../README.md#39-vector-removal-relocation--the-move-zeroes-trap)
- `remove` vs `swap_remove` vs `retain` decision table — [§3.9](../README.md#39-vector-removal-relocation--the-move-zeroes-trap)

---

## 9. 5-Minute Speed Read

**Step 1 — classify the problem** using the [Pattern Recognition Map](#1-pattern-recognition-map) table before writing any code.

**Step 2 — pick the right invariant:**

| Pattern | Invariant to hold in your head |
|---|---|
| Two pointers (converging) | Array must be sorted; pointers never cross |
| Two pointers (read/write) | Write pointer only ever moves forward, and only on a "keep" decision |
| Sliding window | Window only grows right; shrinks left *only* when it breaks the rule |
| Prefix sum | `range_sum(l, r) = prefix[r+1] - prefix[l]`, always `i64` for large sums |
| Kadane's | `current = max(num, current + num)` — reset, don't go negative into the future |
| Binary search | Loop while `lo <= hi`; compute `mid = lo + (hi - lo) / 2` to dodge overflow |
| Cyclic sort | Values live at index `value - 1` (or `value`); swap until every seat is correct |

**Step 3 — panic-proof before you submit:**
- Empty array / single element handled explicitly?
- `usize` subtraction that could underflow? Use `.saturating_sub()` or check first.
- Sum that could overflow `i32`? Cast to `i64`.
- 2D access: did you bound-check both `rows` and `matrix[0].len()` (ragged arrays are rare but real)?
- Binary search: does `mid` calculation avoid `(lo + hi) / 2` overflow?

**Step 4 — state complexity out loud.** Interviewers weight "I chose sliding window for O(N) over brute-force O(N²)" as much as the working code itself.

---

## 10. Dated Revision Log

Keep appending dated entries below whenever a mock interview or LeetCode session teaches you something new about arrays specifically. General Rust/compiler lessons still go in `../README.md` §7.

### 📅 September 21, 2026
Split the array-specific material out of the single-file revision vault into this dedicated topic file as part of restructuring `revision.MD` into a per-topic `revision/` folder. Consolidated and expanded: full two-pointer family (converging + read/write, Three Sum, Container With Most Water), fixed and variable sliding windows, prefix sum + difference array + Subarray Sum Equals K, Kadane's + max product subarray, binary search (classic, rotated array, first/last position, binary-search-on-answer), Dutch National Flag, Merge Intervals, rotate-by-k reversal trick, cyclic sort + XOR missing-number, Set Matrix Zeroes, and Spiral Traversal.

<!-- Add your next entry below this line -->
