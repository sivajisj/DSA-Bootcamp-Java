# 🦀 Rust DSA Interview Revision Vault

**Mission:** crack high paying Rust backend / systems interviews by never losing marks to compiler friction. Every pattern here is built so you can recall it under pressure, not just recognize it.

**How to use this file:**
- **Night before an interview:** read Part 6 only (5-minute speed read).
- **While learning a new pattern:** read the full section, the analogy, and type out the code once by hand.
- **After every mock interview or LeetCode session:** append a new `Revision Entry` at the bottom with whatever tripped you up, exactly like the dated entries already in this file.

---

## Table of Contents
1. [The Rust Mental Model](#part-1-the-rust-mental-model)
2. [Data Structure Playbook](#part-2-data-structure-playbook)
3. [Algorithm Pattern Library](#part-3-algorithm-pattern-library)
4. [Compiler Error Fix-It Table](#part-4-compiler-error-fix-it-table)
5. [Local Test Harness Boilerplate](#part-5-local-test-harness-boilerplate)
6. [5-Minute Pre-Interview Speed Read](#part-6-5-minute-pre-interview-speed-read)
7. [Dated Revision Log](#part-7-dated-revision-log)

---

## Part 1: The Rust Mental Model

Rust interviewers are not just testing whether you know the algorithm. They are testing whether the borrow checker beats you in front of them. This part is your armor against that.

### 1.1 Type Safety: `i32` vs `usize`

**The analogy:** think of `usize` as the only currency accepted at the "memory access" border crossing. It does not matter how much `i32` money you are carrying, the border guard (the compiler) will not let you in with the wrong currency. You must exchange it first.

- Rust never implicitly converts numeric types.
- Indexing arrays, vectors, and slices strictly requires `usize`.
- Algorithmic logic (deltas, negative offsets, math) is usually done in `i32` or `i64`.

```rust
// Cast explicitly whenever an i32-typed index touches memory
let value = nums[idx as usize];
```

### 1.2 Range Iteration Types

Writing `0..n` where `n: i32` gives you `i32` loop variables, which then fail to index anything.

```rust
// ❌ n is i32, loop variable i is i32, cannot index with it
// for i in 0..n { arr[i] } 

// ✅ .len() returns usize, so the loop variable is already correctly typed
for i in 0..nums.len() {
    let _ = nums[i];
}
```

### 1.3 Ownership & Parameter Mutability

**The analogy:** a function parameter by default is like a library book. You can read it, but you cannot scribble on the pages. If you want to write on it, you have to explicitly check it out as "mutable" first.

```rust
// Re-bind the parameter with `mut` to modify the buffer in place, O(1) aux space
pub fn build_array(mut nums: Vec<i32>) -> Vec<i32> {
    nums.reverse();
    nums
}
```

### 1.4 Statements vs. Expressions

The missing semicolon is not a typo, it is a return statement.

```rust
fn square(x: i32) -> i32 {
    x * x   // no semicolon -> this is the return value
}

fn log_and_divide(nums: &mut [i32], n: i32) {
    for x in nums.iter_mut() {
        *x /= n;   // semicolon -> pure side effect, nothing returned
    }
}
```

### 1.5 Move Semantics Inside Closures (`E0507`)

**The analogy:** `|&x|` in a closure tries to physically hand you the original photo out of someone's photo album. If the photo is not a duplicate-able type (`Copy`), the album owner will not let you rip it out, because that would leave a hole. Ask for a photocopy (`|x|`, a reference) instead.

```rust
// ❌ Fails for non-Copy types like String, Vec, custom structs
// a.retain(|&x| x != b[i]);

// ✅ Works universally: compare references
a.retain(|x| x != &b[i]);

// ✅ Most idiomatic for "array difference" style problems
a.retain(|item| !b.contains(item));
```

### 1.6 Exponentiation

Rust has no `**` operator.

```rust
let squared = x * x;      // for squaring, just multiply
let powered = x.pow(3);   // for general integer exponents
```

### 1.7 String Formatting Landmines

**Problem A: expressions inside `format!` braces**

```rust
// ❌ format! only accepts simple identifiers inside {}
// format!("{names[0]} likes this");

// ✅ pass indices as positional arguments
format!("{} likes this", names[0]);
```

**Problem B: `{}` (Display) vs `{:?}` (Debug)**

`{:?}` on a string prints literal escape quotes (`"\"Peter\" likes this"`), which silently breaks test assertions that expect clean text. Use `{}` for user-facing strings, `{:?}` only when you are inspecting a data structure's shape.

**Problem C: `if` without `else` must resolve to `()`**

```rust
// ❌ if-only block returning a value causes "expected (), found String"
// if n <= 0 { format!("no one likes this") }

// ✅ use an explicit return as a guard clause
if n <= 0 {
    return format!("no one likes this");
}
```

---

## Part 2: Data Structure Playbook

### 2.1 Stack Arrays `[T; N]` vs Heap Vectors `Vec<T>`

**The analogy:** a fixed array is a parking lot with a painted, fixed number of spaces, fast because there is no office to call when you need a new spot, but you can never add a space. A `Vec` is a parking lot with a manager who can buy the empty lot next door whenever you need more room, more flexible, but that phone call (reallocation) costs time.

| Type | Location | Growable | Best for |
|---|---|---|---|
| `[T; N]` | Stack | No | Fixed-size returns: coordinates, directions, matrix bounds |
| `Vec<T>` | Heap | Yes | Variable-length algorithm outputs |

Concatenating two fixed arrays without heap allocation:

```rust
let arr1 = [1, 2, 3];
let arr2 = [4, 5];

let mut combined = [0; 5];        // exact stack buffer, pre-sized
combined[..3].copy_from_slice(&arr1);
combined[3..].copy_from_slice(&arr2);
```

### 2.2 Array / Slice / Vec Conversion Cheat Sheet

| Method | Syntax | Ownership | Best used for |
|---|---|---|---|
| `.to_vec()` | `arr.to_vec()` | Clones into a new `Vec`, original stays intact | Slices or arrays you still need afterward |
| `Vec::from()` | `Vec::from(arr)` | Consumes or copies | Explicit, readable conversion |
| `.into()` | `arr.into()` | Consumes the array | Idiomatic code where the target type is inferable |

> 🚨 **Gotcha:** `vec![arr]` does **not** flatten. It produces `Vec<[T; N]>`, a vector containing one giant array, not a flat `Vec<T>`.

Accept slices, not concrete types, in function signatures for maximum caller flexibility:

```rust
// Accepts Vec<T>, [T; N], and partial slices &v[2..5], all via deref coercion
fn process(data: &[i32]) -> i32 { data.iter().sum() }
```

### 2.3 HashMap: the Single-Lookup Entry API

**The analogy:** checking `.contains_key()` then `.get()`/`.insert()` is like calling the hotel front desk twice, once to ask "does room 12 exist" and once to actually ask for the key. `.entry()` is a single call: "give me room 12, and if it doesn't exist yet, build it right now."

```rust
use std::collections::hash_map::Entry;

match map.entry(key) {
    Entry::Occupied(entry) => {
        let val = entry.get();
    }
    Entry::Vacant(entry) => {
        entry.insert(default_val);
    }
}

// The idiomatic one-liner form, used constantly in frequency counting:
*counts.entry(num).or_insert(0) += 1;
```

### 2.4 String to Vec Conversions & UTF-8 Memory Layout

| Target type | Cost | Unicode safe | Use case |
|---|---|---|---|
| `Vec<u8>` | O(1) via `.into_bytes()` | ❌ ASCII only | High-performance byte-level DSA |
| `Vec<char>` | O(N) allocation | ✅ Yes | O(1) random-index access on Unicode text |
| `Vec<&str>` | O(1) per slice | ✅ Yes | Word tokenization via `.split_whitespace()` |

```rust
// 1. Zero-cost owned byte vector (ASCII algorithms)
let bytes: Vec<u8> = owned_string.into_bytes();

// 2. Safe random character indexing on Unicode
let chars: Vec<char> = "hello 🦀".chars().collect();
let last_char = chars[chars.len() - 1];

// 3. Borrowed word views, no buffer copy
let words: Vec<&str> = text.split_whitespace().collect();
```

### 2.5 Iterating Without Fighting the Borrow Checker

| Approach | Borrow type | Ownership | Can mutate in place | Use case |
|---|---|---|---|---|
| `for i in 0..arr.len()` | Direct indexing | N/A | ✅ Yes (`arr[i] = ...`, `arr.swap(i, j)`) | Index-dependent logic, swapping |
| `.iter_mut()` | `&mut T` | Borrows mutably | ✅ Yes (`*item = ...`) | Sequential in-place transform |
| `.into_iter()` | `T` | Consumes / moves | ❌ No | Transforming ownership away |

---

## Part 3: Algorithm Pattern Library

This is the section to drill until your hands know it without your brain thinking about syntax.

### 3.1 In-Place Matrix Transform (Two-Pointer, O(1) Aux Space)

Flip a matrix row horizontally and invert every bit in one linear pass.

```rust
pub fn flip_and_invert_image_inplace(mut image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n = image.len();

    for i in 0..n {
        // Stop at (n+1)/2 to reach the middle without re-swapping back
        for j in 0..(n + 1) / 2 {
            let k = n - 1 - j;
            let temp = image[i][j] ^ 1;
            image[i][j] = image[i][k] ^ 1;
            image[i][k] = temp;
        }
    }
    image
}
```

The declarative twin, when immutability is preferred over raw speed:

```rust
pub fn flip_and_invert_image_functional(image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    image
        .into_iter()
        .map(|row| row.into_iter().rev().map(|b| b ^ 1).collect())
        .collect()
}
```

### 3.2 Sliding Window: Longest Substring Without Repeats

**The analogy:** imagine a rubber band stretched over the string. The right end always moves forward. The moment you touch a character you have already touched inside the band, the left end snaps forward to just past that character's last position, never backward.

```rust
use std::collections::HashMap;

/// Time: O(N), Space: O(K) where K is the unique character set size
fn length_of_longest_substring(input_text: &str) -> usize {
    let mut last_seen_index_map: HashMap<char, usize> = HashMap::new();
    let mut max_len = 0;
    let mut window_start = 0;

    for (window_end, current_char) in input_text.chars().enumerate() {
        if let Some(&prev_index) = last_seen_index_map.get(&current_char) {
            // .max() guarantees window_start never moves backward on a stale index
            window_start = window_start.max(prev_index + 1);
        }
        last_seen_index_map.insert(current_char, window_end);
        max_len = max_len.max(window_end - window_start + 1);
    }
    max_len
}

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_longest_substring() {
        assert_eq!(length_of_longest_substring("abcabcbb"), 3);
        assert_eq!(length_of_longest_substring("bbbbb"), 1);
        assert_eq!(length_of_longest_substring("pwwkew"), 3);
    }
}
```

### 3.3 One-Pass Running Min/Max Tracking

Avoid an O(N²) brute-force double loop by carrying state forward.

```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    let mut min_price = i32::MAX;
    let mut max_profit = 0;

    for price in prices {
        min_price = min_price.min(price);
        max_profit = max_profit.max(price - min_price);
    }
    max_profit
}
```

### 3.4 Floyd's Cycle Detection (Pointer Jumping)

**The analogy:** two runners on a circular track, one jogging (1 step) and one sprinting (2 steps). If the track has a loop, the sprinter eventually laps the jogger and they meet. That meeting point, combined with a reset lap, reveals exactly where the loop begins.

```rust
pub fn find_duplicate(nums: Vec<i32>) -> i32 {
    let mut slow = nums[0];
    let mut fast = nums[nums[0] as usize];

    // Phase 1: run until they collide somewhere inside the cycle
    while slow != fast {
        slow = nums[slow as usize];
        fast = nums[nums[fast as usize] as usize];
    }

    // Phase 2: reset slow to the start, advance both by 1 step at a time
    // Where they meet again is the cycle entrance, i.e. the duplicate
    slow = 0;
    while slow != fast {
        slow = nums[slow as usize];
        fast = nums[fast as usize];
    }
    slow
}
```

### 3.5 Prefix Sum / Pivot Index

**The concept:** find an index where `sum(left side) == sum(right side)`.

**Why binary search fails here:** with positive numbers only, moving the midpoint right predictably shrinks the right sum and grows the left sum, a monotonic relationship binary search depends on. Negative numbers destroy that monotonicity, moving right can just as easily decrease the left sum, causing the search to oscillate forever instead of converging.

**The trick:** maintain one running total instead of recomputing slice sums for every index (which would be O(N²)):

```
right_sum = total_sum - left_sum - current_element
```

```rust
pub fn find_even_index(arr: &[i32]) -> Option<usize> {
    let total_sum: i64 = arr.iter().map(|&x| x as i64).sum(); // i64 guards against overflow
    let mut left_sum: i64 = 0;

    for (i, &num) in arr.iter().enumerate() {
        let current = num as i64;
        let right_sum = total_sum - left_sum - current;

        if left_sum == right_sum {
            return Some(i); // first matching index wins
        }
        left_sum += current;
    }
    None
}
```

### 3.6 Two Sum: The Canonical HashMap Pattern

```rust
use std::collections::HashMap;

pub fn two_sum(numbers: &[i32], target: i32) -> (usize, usize) {
    let mut map: HashMap<i32, usize> = HashMap::with_capacity(numbers.len());

    for (index, &num) in numbers.iter().enumerate() {
        let complement = target - num;
        if let Some(&complement_index) = map.get(&complement) {
            return (complement_index, index);
        }
        map.insert(num, index);
    }
    // Use panic!/unreachable! only when the problem guarantees a solution exists
    panic!("No two sum solution found");
}
```

### 3.7 Sorting-Based Patterns

**Wave sort** (alternating peaks and valleys, O(N log N)):

```rust
pub fn wave_sort(arr: &mut [i32]) {
    arr.sort_unstable();
    // .saturating_sub(1) prevents usize underflow panics on empty slices
    for i in (0..arr.len().saturating_sub(1)).step_by(2) {
        arr.swap(i, i + 1);
    }
}
```

**Merge and deduplicate** (cheaper than `Vec -> HashSet -> Vec`):

```rust
fn merge_and_deduplicate(arr1: &[i32], arr2: &[i32]) -> Vec<i32> {
    let mut vec = Vec::with_capacity(arr1.len() + arr2.len()); // avoid re-allocations
    vec.extend_from_slice(arr1);
    vec.extend_from_slice(arr2);
    vec.sort_unstable(); // O(N log N)
    vec.dedup();         // removes consecutive duplicates, O(N)
    vec
}
```

| Approach | Heap allocations | Time | Notes |
|---|---|---|---|
| `sort_unstable()` + `dedup()` | 1 | O(N log N) | Lowest overhead, all in-place |
| `HashSet` conversion | 3 (`Vec`, `HashSet`, `Vec`) | O(N) with hashing | Higher overhead from SipHash + multiple allocations |

```rust
// Vec -> HashSet, O(N), consumes ownership
let set: HashSet<T> = vec.into_iter().collect();

// HashSet -> Vec, arbitrary order, must re-sort if order matters
let mut vec: Vec<T> = set.into_iter().collect();
vec.sort_unstable();
```

### 3.8 Search Patterns

```rust
// Forward search, O(N), on any comparable type
let wolf_pos = queue.iter().position(|&r| r == "wolf").unwrap();

// Backward search
let last_pos = queue.iter().rposition(|&r| r == "wolf").unwrap();

// Sorted arrays only: O(log N) starting index of a target
let idx = sorted_arr.partition_point(|&x| x < target);
```

Finding maximums safely:

```rust
let arr = [3, 1, 9, 4, 7];

// value only
if let Some(&max_val) = arr.iter().max() { /* 9 */ }

// value + index
if let Some((idx, &val)) = arr.iter().enumerate().max_by_key(|&(_, &v)| v) { /* idx=2, val=9 */ }

// floats need partial_cmp because NaN has no total order
let max_float = [3.5, 1.2, 9.8].iter().max_by(|a, b| a.partial_cmp(b).unwrap());
```

### 3.9 Vector Removal, Relocation & the "Move Zeroes" Trap

**The trap:** calling `.remove(i)` inside a `0..vec.len()` loop shifts every later element left by one, but your loop counter keeps marching forward, silently skipping the element that just slid into position `i`.

| Method | Time | Order preserved | Notes |
|---|---|---|---|
| `vec.remove(i)` | O(N) | ✅ Yes | Shifts everything after `i` left, panics if `i >= len` |
| `vec.swap_remove(i)` | O(1) | ❌ No | Swaps in the last element, panics if `i >= len` |
| `vec.retain(f)` | O(N) | ✅ Yes | Keeps elements where the predicate is true |

```rust
// Remove ALL occurrences of a value
vec.retain(|&x| x != target);

// Remove FIRST occurrence, order preserved
if let Some(idx) = vec.iter().position(|&x| x == target) {
    vec.remove(idx);
}

// Remove FIRST occurrence, fastest, order not preserved
if let Some(idx) = vec.iter().position(|&x| x == target) {
    vec.swap_remove(idx);
}
```

**Move Zeroes**, two ways:

```rust
// Two-pointer overwrite, O(N) time, O(1) extra space
pub fn move_zeros_in_place(arr: &[u8]) -> Vec<u8> {
    let mut vec1 = arr.to_vec();
    let mut write_ptr = 0;

    for i in 0..vec1.len() {
        if vec1[i] != 0 {
            vec1[write_ptr] = vec1[i];
            write_ptr += 1;
        }
    }
    while write_ptr < vec1.len() {
        vec1[write_ptr] = 0;
        write_ptr += 1;
    }
    vec1
}

// Functional filter + resize
pub fn move_zeros_functional(arr: &[u8]) -> Vec<u8> {
    let mut result: Vec<u8> = arr.iter().copied().filter(|&x| x != 0).collect();
    result.resize(arr.len(), 0);
    result
}
```

**Relocation cheat sheet:**

| Task | Pattern | Time | Order preserved |
|---|---|---|---|
| Move target to end, ordered | `let item = vec.remove(i); vec.push(item);` | O(N) | ✅ Yes |
| Move target to end, unordered | `let item = vec.swap_remove(i); vec.push(item);` | O(1) | ❌ No |
| Move target to end, in place | `vec[i..].rotate_left(1);` | O(N) | ✅ Yes |

### 3.10 Integer Square Root

```rust
// Rust 1.84+ only
let root = x.isqrt();

// Legacy toolchain / online judge fallback (Codewars, LeetCode)
let root = (x as f64).sqrt() as u32;

// Perfect square check
let is_perfect = root * root == x;
```

### 3.11 Local Playground: Square or Square Root

```rust
fn square_or_square_root(arr: &[u32]) -> Vec<u32> {
    arr.iter()
        .map(|&x| {
            let root = (x as f64).sqrt() as u32;
            if root * root == x { root } else { x * x }
        })
        .collect()
}
```

---

## Part 4: Compiler Error Fix-It Table

Read this table top to bottom the moment `cargo` throws red text at you mid-interview. Staying calm here is worth more than knowing another algorithm.

| Error / symptom | Root cause | Fix |
|---|---|---|
| `type i32 cannot be used to index a slice` | Loop variable typed `i32` from `0..n` where `n: i32` | Loop over `0..arr.len()` instead, or cast with `as usize` |
| `cannot borrow as mutable` on a parameter | Parameters are immutable bindings by default | Add `mut` to the parameter itself: `fn f(mut nums: Vec<i32>)` |
| `expected (), found String` (or similar) in an `if` | An `if` without `else` must evaluate to `()` | Use `return` as a guard clause, or add a matching `else` |
| `E0507: cannot move out of a shared reference` in `.retain()`/closures | `|&x|` tries to move a non-`Copy` value out of a reference | Use `|x|` and compare against references instead |
| `E0277` on `.sum()` | No implicit numeric casts; accumulator type mismatches iterator item type | Map and cast each item first, or keep types uniform |
| `E0282: type annotations needed` on `.sum() as i32` | Rust cannot infer `.sum()`'s intermediate type from a trailing cast | Use turbofish: `.sum::<usize>() as i32` |
| `E0599` on `.isqrt()` | Method requires Rust 1.84+, unavailable on the judge's toolchain | Fall back to `(x as f64).sqrt() as u32` |
| Panic: `range start index out of range` | Slicing `&s[1..]` on a slice of length 0 | Guard with `if s.len() > 1`, or use `.iter().skip(1)` |
| `expected Vec, found ()` after `.sort()` | `.sort()`/`.sort_unstable()` mutate in place and return `()` | Sort the mutable vector on its own line, then use it directly |
| Type mismatch comparing `x` to `.min()`/`.max()` result | `.min()` returns `Option<&T>`, not `T` | Pattern match: `if let Some(&min_val) = v.iter().min()` |
| Confusion around `vec.remove()` | It mutates in place **and** returns the removed element directly | No `?`, no `.unwrap()` needed; just use the returned value |
| `format!("{names[0]}...")` syntax error | Format string braces only accept simple identifiers | Pass indices as positional args: `format!("{}", names[0])` |
| Debug-quoted strings breaking assertions | `{:?}` used on a `String` shows escaped quotes | Use `{}` (Display) for clean text output |
| `**` used for exponent | Rust has no exponent operator | Use `x * x` or `x.pow(n)` |
| `vec![arr]` producing the wrong shape | `vec![x]` wraps `x` in a single-element vector, does not flatten | Use `.to_vec()`, `Vec::from(arr)`, or `.into()` |

---

## Part 5: Local Test Harness Boilerplate

Paste this into a scratch file to compile and sanity-check any solution locally before submitting.

```rust
struct Solution;

impl Solution {
    // Paste solution functions here
}

fn main() {
    let input = vec![
        vec![1, 1, 0],
        vec![1, 0, 1],
        vec![0, 0, 0],
    ];

    let result = Solution::flip_and_invert_image_functional(input);

    println!("Transformed Matrix Output:");
    for row in result {
        println!("{:?}", row);
    }
}
```

---

## Part 6: 5-Minute Pre-Interview Speed Read

**Type discipline:**
- Index with `usize`. Loop with `0..arr.len()`, never `0..n` where `n: i32`.
- Mark parameters `mut` to mutate in place. No implicit numeric casts, ever.
- No `**`. Use `x.pow(n)`.

**Reflex patterns, matched to problem smell:**

| If the problem smells like... | Reach for... |
|---|---|
| "no repeating characters", "longest/shortest substring" | Sliding window with `HashMap<char, usize>` |
| "find the duplicate", array values as pointers | Floyd's cycle detection |
| "two numbers that sum to target" | HashMap single-pass, `entry()` |
| "pivot / equal sums on both sides" | Prefix sum with one running total, cast to `i64` |
| "max profit", "running best so far" | One-pass min/max tracking, no nested loops |
| "remove/shift elements in place" | Two-pointer write index, never `.remove()` inside a growing/shrinking loop |
| "alternating pattern in array" | Sort, then `step_by(2)` swap |
| duplicate removal | `sort_unstable()` + `dedup()`, not `HashSet` round-trip |

**Panic-proofing checklist before you hit submit:**
- Any `&slice[n..]`? Guard against `len() == 0`.
- Any subtraction on `usize`? Use `.saturating_sub()`.
- Any sum that could overflow `i32`? Cast to `i64` first.
- Any `.min()`/`.max()` on an empty collection? Handle the `None` case.

**When the compiler yells `E0507`, `E0277`, or `E0282`:** breathe, then jump straight to Part 4.

---

## Part 7: Dated Revision Log

Keep appending new dated entries below in this same format whenever you learn something new. Never overwrite old entries, this log is your proof of progress.

### 📅 September 7, 2026
Covered: `format!` macro constraints, Display vs Debug, `if` without `else` typing, string/char to number conversions, slice-as-parameter idiom, `.position()`/`.rposition()`, wave sort with `.saturating_sub()`.

### 📅 September 8, 2026
Covered: `E0507` move-out-of-reference in closures, exponentiation syntax, `.isqrt()` vs `.sqrt()` fallback, iterator ownership table (`into_iter` vs `iter_mut` vs index loop).

### 📅 September 10, 2026
Covered: slice bound panics on empty/short slices, `.sort()` return type gotcha, safe max-finding patterns, `Vec` vs `HashSet` dedup performance, string-to-Vec UTF-8 layout, sliding window template.

### 📅 September 16, 2026
Consolidated all prior entries into this single structured revision vault. Added: Two Sum canonical pattern, array/vec conversion table, `HashMap::entry` deep dive, Floyd's cycle detection, prefix sum / pivot index with the negative-number binary-search trap explained, move-zeroes two-pointer pattern, vector relocation cheat sheet, and the full compiler error fix-it table.

<!-- Add your next entry below this line -->
