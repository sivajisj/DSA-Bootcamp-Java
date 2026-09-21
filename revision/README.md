# 🦀 Rust DSA Interview Revision Vault

**Mission:** crack high paying Rust backend / systems interviews by never losing marks to compiler friction. Every pattern here is built so you can recall it under pressure, not just recognize it.

Think of this vault as two layers, like a toolbox with a top tray and drawers underneath. This top-level file (`README.md`) is the **top tray** — the general Rust muscle memory that applies no matter what kind of problem you're solving: how the type system bites you, how to fix the compiler's red text fast, and your pre-interview checklist. The **drawers** are `topics/*.md` — one deep-dive file per data structure or algorithm family (arrays, strings, trees, ...), each written to be genuinely fun to read, not just correct.

**How to use this vault:**
- **Night before an interview:** read [Part 6](#part-6-5-minute-pre-interview-speed-read) here, then the speed-read section of whatever topic you're weakest in.
- **While learning a new pattern:** open the matching file in `topics/`, read the full section, the analogy, and type out the code once by hand.
- **After every mock interview or LeetCode session:** append a new `Revision Entry` at the bottom of whichever file the lesson belongs to — general Rust gotcha → here; pattern-specific insight → the topic file.

---

## Table of Contents
1. [The Rust Mental Model](#part-1-the-rust-mental-model)
2. [Data Structure Playbook](#part-2-data-structure-playbook)
3. [Algorithm Pattern Library](#part-3-algorithm-pattern-library)
4. [Compiler Error Fix-It Table](#part-4-compiler-error-fix-it-table)
5. [Local Test Harness Boilerplate](#part-5-local-test-harness-boilerplate)
6. [5-Minute Pre-Interview Speed Read](#part-6-5-minute-pre-interview-speed-read)
7. [Dated Revision Log](#part-7-dated-revision-log)
8. [Topic Files (Deep Dives)](#part-8-topic-files-deep-dives)

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

> **Arrays have graduated!** The full array/vector/matrix playbook — two pointers, sliding window, prefix sums, Kadane's, binary search, in-place tricks, and more — now lives in its own deep-dive: **[`topics/arrays.md`](topics/arrays.md)**. What stays here is the general-purpose Rust data-structure mechanics that isn't specific to any one topic.

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

> **Heads up:** the array-flavored entries that used to live here (matrix transforms, sliding window, Kadane-style running min/max, Floyd's cycle detection, prefix sum, Two Sum, sorting patterns, search patterns, and the Move Zeroes trap) have all moved into **[`topics/arrays.md`](topics/arrays.md)**, expanded with a lot more coverage. This section keeps the material that isn't array-specific.

### 3.1 Integer Square Root

```rust
// Rust 1.84+ only
let root = x.isqrt();

// Legacy toolchain / online judge fallback (Codewars, LeetCode)
let root = (x as f64).sqrt() as u32;

// Perfect square check
let is_perfect = root * root == x;
```

### 3.2 Local Playground: Square or Square Root

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

**Reflex patterns, matched to problem smell** (full detail lives in each topic's own speed-read section):

| If the problem smells like... | Reach for... | Full detail |
|---|---|---|
| "pair/triplet sums to X", sorted array | Two pointers | [`topics/arrays.md` §2](topics/arrays.md#2-two-pointers) |
| "no repeating characters", "longest/shortest substring/subarray" | Sliding window with `HashMap` | [`topics/arrays.md` §3](topics/arrays.md#3-sliding-window) |
| "subarray sum equals K", repeated range-sum queries | Prefix sum | [`topics/arrays.md` §4](topics/arrays.md#4-prefix-sum--difference-array) |
| "max profit", "max subarray sum", "running best so far" | Kadane's / one-pass min-max tracking | [`topics/arrays.md` §5](topics/arrays.md#5-kadanes-algorithm-max-subarray-family) |
| Sorted or rotated-sorted array, need O(log N) | Binary search variants | [`topics/arrays.md` §6](topics/arrays.md#6-binary-search-on-arrays) |
| "find the duplicate", array values as pointers | Floyd's cycle detection | [`topics/arrays.md` §8.2](topics/arrays.md#82-cyclic-sort-values-in-range-1n-find-missingduplicate) |
| "pivot / equal sums on both sides" | Prefix sum with one running total, cast to `i64` | [`topics/arrays.md` §4](topics/arrays.md#4-prefix-sum--difference-array) |
| "remove/shift/rotate elements in place" | Two-pointer write index or reversal trick | [`topics/arrays.md` §8](topics/arrays.md#8-in-place-rearrangement-tricks) |
| "alternating pattern", "sort 0s 1s 2s" | Sort + `step_by(2)`, or Dutch National Flag | [`topics/arrays.md` §7](topics/arrays.md#7-sorting-based-patterns) |
| duplicate removal | `sort_unstable()` + `dedup()`, not `HashSet` round-trip | [`topics/arrays.md` §7](topics/arrays.md#7-sorting-based-patterns) |

**Panic-proofing checklist before you hit submit:**
- Any `&slice[n..]`? Guard against `len() == 0`.
- Any subtraction on `usize`? Use `.saturating_sub()`.
- Any sum that could overflow `i32`? Cast to `i64` first.
- Any `.min()`/`.max()` on an empty collection? Handle the `None` case.

**When the compiler yells `E0507`, `E0277`, or `E0282`:** breathe, then jump straight to Part 4.

---

## Part 7: Dated Revision Log

Keep appending new dated entries below in this same format whenever you learn something new that's general to Rust (not tied to one topic — topic-specific lessons go in that topic's own log). Never overwrite old entries, this log is your proof of progress.

### 📅 September 7, 2026
Covered: `format!` macro constraints, Display vs Debug, `if` without `else` typing, string/char to number conversions, slice-as-parameter idiom, `.position()`/`.rposition()`, wave sort with `.saturating_sub()`.

### 📅 September 8, 2026
Covered: `E0507` move-out-of-reference in closures, exponentiation syntax, `.isqrt()` vs `.sqrt()` fallback, iterator ownership table (`into_iter` vs `iter_mut` vs index loop).

### 📅 September 10, 2026
Covered: slice bound panics on empty/short slices, `.sort()` return type gotcha, safe max-finding patterns, `Vec` vs `HashSet` dedup performance, string-to-Vec UTF-8 layout, sliding window template.

### 📅 September 16, 2026
Consolidated all prior entries into this single structured revision vault. Added: Two Sum canonical pattern, array/vec conversion table, `HashMap::entry` deep dive, Floyd's cycle detection, prefix sum / pivot index with the negative-number binary-search trap explained, move-zeroes two-pointer pattern, vector relocation cheat sheet, and the full compiler error fix-it table.

### 📅 September 21, 2026
Restructured the vault: split into this general-purpose `README.md` plus a `topics/` folder of deep-dive files, starting with a heavily expanded **[`topics/arrays.md`](topics/arrays.md)** (two pointers, sliding window, prefix sum, Kadane's, binary search, Dutch National Flag, cyclic sort, matrix tricks, and more). Root-level `revision.MD` now points here. See [Part 8](#part-8-topic-files-deep-dives) for how to add the next topic.

---

## Part 8: Topic Files (Deep Dives)

Each data structure or algorithm family gets its own file under `topics/`, written like a mini-course: a pattern recognition map at the top, an analogy for every trick, runnable Rust, and its own dated log at the bottom.

| Topic | File | Status |
|---|---|---|
| Arrays & Matrices | [`topics/arrays.md`](topics/arrays.md) | ✅ Live |
| Strings | `topics/strings.md` | 🔲 Not started |
| Linked Lists | `topics/linked-lists.md` | 🔲 Not started |
| Stacks & Queues | `topics/stacks-queues.md` | 🔲 Not started |
| Hashing | `topics/hashing.md` | 🔲 Not started |
| Trees | `topics/trees.md` | 🔲 Not started |
| Heaps / Priority Queues | `topics/heaps.md` | 🔲 Not started |
| Graphs | `topics/graphs.md` | 🔲 Not started |
| Dynamic Programming | `topics/dynamic-programming.md` | 🔲 Not started |
| Bitwise Tricks | `topics/bitwise.md` | 🔲 Not started |
| Greedy & Intervals | `topics/greedy-intervals.md` | 🔲 Not started |

### How new topics get added

1. Copy [`topics/_TEMPLATE.md`](topics/_TEMPLATE.md) to `topics/<topic-name>.md`.
2. Fill in a pattern recognition map, then one section per pattern with an analogy + working, tested Rust code.
3. End it with its own "5-Minute Speed Read" and "Dated Revision Log" sections, matching the shape of `arrays.md`.
4. Flip the topic's row in the table above from 🔲 to ✅ and link it.
5. If the topic overlaps another (e.g. a graph problem that's really a hashing problem), cross-link instead of duplicating the code — see how `arrays.md` links back to this file for Two Sum and Floyd's cycle detection instead of repeating them.
