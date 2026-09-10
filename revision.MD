

```markdown
# Rust Technical Interview Reference & Key Takeaways

A personal reference guide and cheat sheet for writing idiomatic, high-performance, and memory-efficient Rust solutions for technical coding interviews (LeetCode / Data Structures & Algorithms).

---

## 🛠️ Fundamental Syntax & Types

### Primitive Types & Type Casting (`i32` vs. `usize`)
* **Why:** Rust enforces strict type safety and does not implicitly convert numeric types. Array, vector, and slice indexing strictly requires `usize`, whereas algorithmic logic usually operates on signed integers like `i32`.
* **Takeaway:** Always explicitly cast integer indices using the `as` keyword when accessing memory or slices (e.g., `nums[idx as usize]`).

### Iterators & Range Syntax (`0..nums.len()` vs. `0..n`)
* **Why:** Writing `0..n` (where `n` is typed as `i32`) yields `i32` loop variables. Attempting to use an `i32` loop variable directly as a slice index triggers a compilation error (`type `i32` cannot be used to index a slice`).
* **Takeaway:** Prefer `0..nums.len()` for indexing loops because `.len()` naturally returns `usize`, keeping loop variables typed correctly without manual casting.

### Ownership & Parameter Mutability (`mut nums: Vec<i32>`)
* **Why:** Function parameters in Rust are immutable bindings by default. Performing in-place array modifications (e.g., two-pointer reversals or swaps) requires mutable access to the underlying buffer.
* **Takeaway:** Re-bind input arguments with `mut` directly inside the function signature (e.g., `pub fn build_array(mut nums: Vec<i32>) -> Vec<i32>`) to modify the buffer in-place using $O(1)$ auxiliary space.

### Statements vs. Expressions (Semicolons `;`)
* **Why:** Omitting a semicolon at the end of a block turns the final statement into a return expression.
* **Takeaway:** Use semicolons to terminate imperative side-effect operations (e.g., `nums[i] /= n;`). Omit the trailing semicolon on the final line of a block or function to return that value implicitly.

---

## 🚀 Memory Optimization & Data Structures

### Stack Arrays (`[T; N]`) vs. Heap Vectors (`Vec<T>`)
* **Fixed-size Arrays (`[T; N]`):** Allocated directly on the stack with a fixed capacity known at compile-time. Ideal for small, fixed returns (e.g., coordinates, directions, matrix cell bounds). Fixed arrays **do not** implement dynamic growth traits like `.extend()`.
* **Vectors (`Vec<T>`):** Dynamic, heap-allocated collections consisting of a pointer, length, and capacity. Default choice for variable-length algorithm outputs.

### Stack Array Concatenation without Allocation
To combine two fixed-size stack arrays without allocating heap memory, copy slice memory directly using `.copy_from_slice()`:
```rust
let arr1 = [1, 2, 3];
let arr2 = [4, 5];

let mut combined = [0; 5]; // Pre-allocate exact stack buffer
combined[..3].copy_from_slice(&arr1);
combined[3..].copy_from_slice(&arr2);

```

### Single-Lookup Entry API (`HashMap::entry`)

* **Why:** Combining `.contains_key()` followed by `.get()` or `.insert()` forces Rust to compute the hashing algorithm twice for the exact same key.
* **Takeaway:** Use `HashMap::entry` to locate the key bucket location in memory **exactly once**.

```rust
use std::collections::hash_map::Entry;

match map.entry(key) {
    Entry::Occupied(entry) => {
        let val = entry.get(); // Access existing reference
    }
    Entry::Vacant(entry) => {
        entry.insert(default_val); // Insert missing value directly
    }
}

```

---

# 📚 Algorithmic Patterns & Templates

Idiomatic, copy-pasteable Rust code templates for common technical interview patterns, complete with line-by-line annotations explaining indexing safety, compiler constraints, and performance traits.

---

## 1. Matrix Transformations

### In-Place Symmetric Swap + Invert ($O(1)$ Auxiliary Space)

Reverses each matrix row horizontally and flips bit values ($0 \leftrightarrow 1$) in a single linear pass without allocating extra memory.

```rust
pub fn flip_and_invert_image_inplace(mut image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n = image.len();

    for i in 0..n {
        // Stop at (n + 1) / 2 to reach the middle element.
        // Going past (n / 2) would re-swap elements back to original positions.
        for j in 0..(n + 1) / 2 {
            let k = n - 1 - j; // Calculate the opposite (symmetric) index in the row.

            // Flip both values using bitwise XOR (^ 1) and swap simultaneously.
            // Works for middle elements where j == k, as (x ^ 1) correctly flips the bit.
            let temp = image[i][j] ^ 1;
            image[i][j] = image[i][k] ^ 1;
            image[i][k] = temp;
        }
    }

    image
}

```

### Functional Chain Transformation ($O(N^2)$ Space)

Clean, functional Rust style using iterator combinators. Preferred when immutability and declarative style are prioritized.

```rust
pub fn flip_and_invert_image_functional(image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    image
        .into_iter() // Consume matrix outer vector into an iterator of rows
        .map(|row| {
            row.into_iter()
                .rev()          // 1. Reverse inner row elements horizontally
                .map(|b| b ^ 1) // 2. Invert bits (0 -> 1, 1 -> 0) in single instruction
                .collect()      // Re-collect inverted elements into Vec<i32>
        })
        .collect() // Re-collect rows into Vec<Vec<i32>>
}

```

---

## 2. Dynamic Sliding & One-Pass Tracking

### Running Minimum / Maximum Tracking ($O(N)$ Time, $O(1)$ Space)

Tracks state variables in a single linear pass to avoid $O(N^2)$ nested comparison loops.

```rust
pub fn max_profit(prices: Vec<i32>) -> i32 {
    let mut min_price = i32::MAX; // Initialize to max boundary to handle first comparison
    let mut max_profit = 0;        // Minimum possible profit is 0 (no transaction)

    for price in prices {
        // Update running minimum purchase price seen so far
        min_price = min_price.min(price);

        // Update maximum profit achievable by selling at current price
        max_profit = max_profit.max(price - min_price);
    }

    max_profit
}

```

---

## 3. Floyd’s Cycle Detection (Pointer Jumping)

### Find the Duplicate Number ($O(N)$ Time, $O(1)$ Space)

Uses array values as pointer offsets (`nums[i] as usize`) to detect graph cycles without altering the input array.

```rust
pub fn find_duplicate(nums: Vec<i32>) -> i32 {
    // Cast index to usize because Rust slice indexing strictly requires usize.
    // Initialize slow (1 jump) and fast (2 jumps) pointers from origin index 0.
    let mut slow = nums[0];
    let mut fast = nums[nums[0] as usize];

    // Phase 1: Run until slow and fast pointers collide inside the cycle.
    while slow != fast {
        slow = nums[slow as usize];                           // 1 step forward
        fast = nums[nums[fast as usize] as usize];            // 2 steps forward
    }

    // Phase 2: Reset slow pointer to origin index 0; advance both pointers by 1 step.
    // The point where they meet again is the entrance to the cycle (the duplicate).
    slow = 0;
    while slow != fast {
        slow = nums[slow as usize]; // 1 step forward
        fast = nums[fast as usize]; // 1 step forward
    }

    slow // Cycle entrance value is the duplicate element
}

```

---

## 4. Single-Lookup HashMap Pattern

### Frequency Counting with the `Entry` API

Eliminates duplicate hash computations by computing key bucket locations exactly once per element.

```rust
use std::collections::HashMap;

pub fn count_frequencies(nums: Vec<i32>) -> HashMap<i32, usize> {
    let mut counts = HashMap::new();

    for num in nums {
        // .entry() locates the key once.
        // .or_insert(0) initializes missing keys to 0.
        // Dereference (*) operator modifies the stored counter directly in-place.
        *counts.entry(num).or_insert(0) += 1;
    }

    counts
}

```

---

## 5. Local Playground Test Harness

Boilerplate wrapper to compile and verify LeetCode solution logic locally in standard Rust environments.

```rust
struct Solution;

impl Solution {
    // Paste solution functions inside this implementation block
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

## 📅 Revision Entry: September 7, 2026

### 1. 🛠️ Rust String Formatting & Compiler Constraints

#### Avoid In-Line Indexing in Format Strings (`format!`)

* **Why:** Rust's string interpolation macro only supports simple variable identifiers inside braces (`{variable}`). Expressions containing slice bounds or index brackets like `names[0]` trigger compiler syntax errors (`expected '}', found '['`).
* **Takeaway:** Pass array or slice indices as explicit positional arguments after the format string.
```rust
// ❌ WRONG: Syntax error
format!("{names[0]} likes this");

// ✅ CORRECT:
format!("{} likes this", names[0]);

```



#### Display `{}` vs. Debug `{:?}` Specifiers

* **Why:** Using `{:?}` on string types formats them using their Debug representation, which includes literal escape quotes (`"\"Peter\" likes this"`), causing test assertions to fail.
* **Takeaway:** Always use `{}` (Display trait) for clean user-facing text and `{:?}` (Debug trait) strictly for inspecting data structures.

#### Return Types in `if` Blocks Without `else`

* **Why:** An `if` statement without a matching `else` branch must evaluate to `()` (unit type). Returning a value like `format!(...)` directly inside an `if`-only block causes a compiler type mismatch error (`expected (), found String`).
* **Takeaway:** Either add an explicit `return` keyword for guard clauses or provide a matching `else` block to return expressions implicitly.
```rust
// Explicit return guard clause:
if n <= 0 {
    return format!("no one likes this");
}

```



---

### 2. ⚡ Array & String Manipulation Patterns

#### Converting Strings & Characters to Numbers

* **`&str` to Number:** Use `.parse::<T>()`, which returns a `Result<T, ParseIntError>` for runtime string validation.
* **`char` to Number:** Use `.to_digit(radix)` (e.g., `.to_digit(10)` for base-10), which returns an `Option<u32>`.

#### Array Slices (`&[T]`) as Function Parameters

* **Why:** Functions accepting concrete `&Vec<T>` or fixed-size array references `&[T; N]` restrict caller flexibility.
* **Takeaway:** Accept slice references `&[T]` in function signatures to support vectors, fixed arrays, and partial array slices seamlessly via deref coercion.

#### Formatting Slices vs. Formatting Individual Elements

* **Why:** Passing a slice range like `&numbers[0..3]` into `format!("({})", ...)` prints the debug array representation `"[1, 2, 3]"` instead of contiguous digits.
* **Takeaway:** Either index elements individually (`numbers[0]`) or collect elements into an intermediate `String` before taking string slice ranges (`&s[0..3]`).

---

### 3. 🧩 Search & Sorting Patterns

#### Finding Indices in Non-Integer Slices (`.position()` & `.rposition()`)

* **Why:** Searching non-integer collections (like `&[&str]` or `Vec<String>`) requires matching references using closures.
* **Takeaway:** Use `.iter().position(|&x| x == "target")` for forward $O(N)$ searches, or `.iter().rposition(...)` / `.iter().rev().position(...)` to search backward from the end. Returns `Option<usize>`.
```rust
let wolf_pos = queue.iter().position(|&r| r == "wolf").unwrap();

```



#### Wave Sorting ($O(N \log N)$)

* **Pattern:** Rearranges an array into an alternating pattern ($a_1 \ge a_2 \le a_3 \ge a_4$).
* **Strategy:** Sort ascending using `.sort_unstable()`, then iterate in steps of 2 using `.step_by(2)` and swap adjacent pairs in-place using `.swap(i, i + 1)`.
* **Underflow Prevention:** Use `.saturating_sub(1)` when computing bounds like `arr.len().saturating_sub(1)` to prevent `usize` underflow panics on empty slices (`[]`).
```rust
pub fn wave_sort(arr: &mut [i32]) {
    arr.sort_unstable();
    for i in (0..arr.len().saturating_sub(1)).step_by(2) {
        arr.swap(i, i + 1);
    }
}

```



---

## 📅 Revision Entry: September 8, 2026

### 1. 🛠️ Compiler Constraints & Move Semantics

#### Trait Constraints in Closures (`E0507: cannot move out of a shared reference`)

* **Why:** Destructuring with `|&x|` in closure parameters (like `vec.retain(...)`) attempts to copy the underlying value out of the reference. If type `T` does not implement `Copy` (e.g., `String`, `Vec`, or non-`Copy` generics), Rust blocks the move to prevent invalidating shared memory.
* **Takeaway:** Receive the item as a reference `|x|` and compare against references or dereference for comparison.
```rust
// ❌ WRONG: Fails for non-Copy types (E0507)
a.retain(|&x| x != b[i]);

// ✅ CORRECT: Works for all types implementing PartialEq
a.retain(|x| x != &b[i]);

// ✅ IDIOMATIC (Array Difference):
a.retain(|item| !b.contains(item));

```



#### Exponentiation Syntax

* **Why:** Rust does not support the `**` operator for exponentiation. Writing `x ** x` causes a compilation error.
* **Takeaway:** Use basic multiplication (`x * x`) for squaring or `.pow(n)` for general integer exponentiation.

---

### 2. ⚡ Numeric & Square Root Patterns

#### Integer Square Root Compatibility (`.isqrt()` vs `.sqrt()`)

* **Method Availability:** `.isqrt()` is available natively in Rust **1.84+** on primitive integer types (`u32`, `i32`, `usize`).
* **Legacy / Online Judge Fallback:** Online competitive coding platforms on older toolchains fail with `E0599` on `.isqrt()`. Use floating-point conversion as a cross-platform fallback:
```rust
// Rust 1.84+ only:
let root = x.isqrt();

// Legacy platform fallback (Codewars, LeetCode):
let root = (x as f64).sqrt() as u32;

// Perfect Square Check:
let is_perfect = root * root == x;

```



---

### 3. 🧩 In-Place Mutation vs. Iterator Ownership

#### Selection Guide for Iterating Arrays/Slices

| Iterator / Approach | Borrow Type | Ownership | Can Mutate In-Place? | Primary Use Case |
| --- | --- | --- | --- | --- |
| `for i in 0..arr.len()` | Direct Indexing | N/A | **Yes** (`arr[i] = ...`) | Index-dependent logic & swapping (`arr.swap(i, j)`) |
| `.iter_mut()` | `&mut T` | Borrows mutably | **Yes** (`*item = ...`) | Sequential in-place value transformation |
| `.into_iter()` | `T` | **Consumes / Moves** | ❌ No | Transforming/consuming collection ownership |

---

### 📝 Practical Reference Code: Square or Square Root

```rust
// Modifies or maps numbers based on whether they are perfect squares
fn square_or_square_root(arr: &[u32]) -> Vec<u32> {
    arr.iter()
        .map(|&x| {
            let root = (x as f64).sqrt() as u32;
            if root * root == x {
                root
            } else {
                x * x
            }
        })
        .collect()
}

```

---

## 📅 Revision Entry: September 10, 2026

### 1. 🛠️ Slice Bounds, In-Place Sorting & Maximum Element Patterns

#### Bound Panics on Empty / Short Slices

* **Why:** Range slicing `&slice[1..]` panics at runtime if `slice.len() == 0` with `range start index 1 out of range for slice of length 0`.
* **Takeaway:** Avoid raw slice indexing on inputs with uncertain bounds. Use length guards or `.skip(n)` iterator methods.
```rust
// ❌ PANIC: Crashes if vector/slice is empty or length 1
rev2.extend_from_slice(&rev1[1..]);

// ✅ SAFE: Manual guard
if rev1.len() > 1 {
    rev2.extend_from_slice(&rev1[1..]);
}

// ✅ IDIOMATIC & SAFE: Iterator skip handles short/empty collections automatically
rev2.extend(rev1.iter().skip(1).copied());

```



#### In-Place Sorting Return Types

* **Why:** Calling `.sort()` or `.sort_unstable()` mutates the underlying collection in-place and returns the unit type `()`.
* **Takeaway:** Assigning `let x = arr.sort()` yields `()` (E0277). Always call sort on a mutable collection and access the collection directly.
```rust
let mut arr = [3, 2, 1];
arr.sort_unstable(); // Mutates arr in-place
println!("{:?}", arr); // [1, 2, 3]

```



#### Finding Maximum Elements

```rust
let arr = [3, 1, 9, 4, 7];

// 1. Get max value reference (returns Option<&T>)
if let Some(&max_val) = arr.iter().max() {
    println!("Max value: {}", max_val); // 9
}

// 2. Get max value along with its index
if let Some((idx, &val)) = arr.iter().enumerate().max_by_key(|&(_, &v)| v) {
    println!("Max {} at index {}", val, idx); // Max 9 at index 2
}

// 3. Max float (handles partial ordering because NaN lacks total Ord)
let max_float = [3.5, 1.2, 9.8].iter().max_by(|a, b| a.partial_cmp(b).unwrap());

```

---

### 2. 🚀 Performance Optimization: Deduplication & Collections

#### Merge & Deduplicate Benchmarks

Instead of converting `Vec -> HashSet -> Vec` to remove duplicates, use `Vec` + `sort_unstable()` + `dedup()`.

```rust
fn merge_and_deduplicate(arr1: &[i32], arr2: &[i32]) -> Vec<i32> {
    // Allocate vector with pre-calculated total capacity to prevent re-allocations
    let mut vec = Vec::with_capacity(arr1.len() + arr2.len());
    
    vec.extend_from_slice(arr1);
    vec.extend_from_slice(arr2);

    vec.sort_unstable(); // Sort in-place fast (O(N log N))
    vec.dedup();         // Remove consecutive duplicates in-place (O(N))

    vec
}

```

#### Performance Comparison Matrix

| Approach | Heap Allocations | Time Complexity | Memory Overhead / Downsides |
| --- | --- | --- | --- |
| **`sort_unstable() + dedup()`** | **1 (`Vec` total)** | $O(N \log N)$ | **Lowest (In-place sorting & deduplication)** |
| **`HashSet` Conversion** | 3 (`Vec`, `HashSet`, `Vec`) | $O(N)$ with hashing | High (SIPHash computation + multiple allocations) |

#### `Vec` <-> `HashSet` Conversion Cheat Sheet

```rust
// Vec -> HashSet (Consumes ownership, O(N))
let set: HashSet<T> = vec.into_iter().collect();

// HashSet -> Vec (Consumes ownership, arbitrary order)
let mut vec: Vec<T> = set.into_iter().collect();

// Note: HashSet -> Vec yields elements in non-deterministic order.
// Always run .sort_unstable() afterward if order is required.
vec.sort_unstable();

```

---

### 3. 🔤 String to `Vec` Conversions & UTF-8 Memory Layout

| Target Type | Time / Memory Cost | Unicode Safe? | Typical Use Case |
| --- | --- | --- | --- |
| **`Vec<u8>`** | $O(1)$ (via `.into_bytes()`) | ❌ ASCII only | High-performance DSA, ASCII string algorithms |
| **`Vec<char>`** | $O(N)$ allocation | **✅ Yes** | Random index lookup ($O(1)$ access) for UTF-8 |
| **`Vec<&str>`** | $O(1)$ per slice | **✅ Yes** | Word tokenization (`.split_whitespace()`) |

#### Idiomatic Code Examples

```rust
// 1. Zero-Cost Owned Byte Vector (ASCII algorithms)
// Transfers ownership of internal heap allocation without copying data
let bytes: Vec<u8> = owned_string.into_bytes();

// 2. Safe Random Character Indexing (Unicode)
// Converts UTF-8 string to UTF-32 char array for O(1) random indexing
let chars: Vec<char> = "hello 🦀".chars().collect();
let last_char = chars[chars.len() - 1]; // '🦀'

// 3. String Slices (No string buffer copy)
// Yields borrowed &str views into existing buffer
let words: Vec<&str> = text.split_whitespace().collect();

```

---

### 🧩 Sliding Window DSA Pattern Reference

Sliding window pattern using explicit, self-documenting variable names and single-pass hash map tracking for character index lookups.

```rust
use std::collections::HashMap;

/// Finds the length of the longest substring without repeating characters.
/// 
/// Time Complexity: O(N) - Single pass through the string.
/// Space Complexity: O(K) - Where K is the size of the unique character set.
fn length_of_longest_substring(input_text: &str) -> usize {
    // Tracks the most recent index where each character was encountered in the string.
    let mut last_seen_index_map: HashMap<char, usize> = HashMap::new();
    
    let mut max_substring_length = 0; // Stores the length of the longest valid substring found.
    let mut window_start = 0;        // The starting boundary (left index) of current sliding window.

    // Iterate through string character by character, maintaining ending boundary (right index).
    for (window_end, current_char) in input_text.chars().enumerate() {
        
        // If character was seen previously, retrieve its last known position.
        if let Some(&previous_char_index) = last_seen_index_map.get(&current_char) {
            // Move window start past previous occurrence of duplicate character.
            // Using .max() ensures window_start never moves backward for stale indices.
            window_start = window_start.max(previous_char_index + 1);
        }

        // Record or update current character's latest index position.
        last_seen_index_map.insert(current_char, window_end);

        // Calculate current window size (window_end - window_start + 1) and update maximum length.
        max_substring_length = max_substring_length.max(window_end - window_start + 1);
    }

    max_substring_length
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

Here is the clean, consolidated Markdown content covering all the key Rust concepts, interview patterns, and compiler error fixes we worked through today. You can append this directly to your **`revision.md`** file.

---

```markdown
## Daily Technical Revision

### 1. Two Sum Pattern ($O(N)$ Time, $O(N)$ Space)
* **Single-Pass HashMap:** Use `HashMap::with_capacity(nums.len())` to pre-allocate memory and eliminate reallocation overhead during iteration.
* **Return Signatures & Divergence:** If a function guarantees a solution exists but signature dictates returning non-`Option` (e.g., `(usize, usize)`), use `panic!("...")` or `unreachable!()` for non-matching fallbacks.

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

    panic!("No two sum solution found");
}

```

---

### 2. Array & Vector Operations

#### A. Array to `Vec` Conversions

| Method | Usage | Ownership Behavior |
| --- | --- | --- |
| `arr.to_vec()` | Preferred for slices/arrays | Clones/copies data into new `Vec` |
| `Vec::from(arr)` | Trait implementation | Consumes array or copies primitives |
| `arr.into()` | Type inference conversion | Consumes array into `Vec` |

> ⚠️ **Warning:** Avoid using `vec![arr]`. It creates a `Vec<[T; N]>` (vector of fixed arrays) rather than a flat `Vec<T>`.

#### B. Finding Minimum Elements

* **Value Only:** `slice.iter().min()` returns `Option<&T>`. Match using `if let Some(&val)` to unwrap safely.
* **Index + Value:** `slice.iter().enumerate().min_by_key(|&(_, &v)| v)` returns `Option<(usize, &T)>`.
* **Float Handling:** `f32`/`f64` do not implement `Ord` due to `NaN`. Use `.min_by(|a, b| a.partial_cmp(b).unwrap())`.

#### C. Finding Element Indices (`.position()`)

* Use `slice.iter().position(|&x| x == target)` to find the index of the **first occurrence** ($O(N)$ time).
* For **sorted arrays**, use `.partition_point(|&x| x < target)` to get the starting index of a target in $O(\log N)$ time.

---

### 3. Vector Element Removal Patterns

| Method | Time Complexity | Order Preserved? | Behavior / Notes |
| --- | --- | --- | --- |
| `vec.remove(i)` | $O(N)$ | **Yes** | Shifts all subsequent elements left. Panics if `i >= len`. |
| `vec.swap_remove(i)` | $O(1)$ | ❌ No | Replaces target with last element. Panics if `i >= len`. |
| `vec.retain(f)` | $O(N)$ | **Yes** | Keeps elements where predicate evaluates to `true`. |

#### Common Implementation Patterns

```rust
// 1. Remove ALL occurrences of value
vec.retain(|&x| x != target);

// 2. Remove FIRST occurrence by value (Order Preserved)
if let Some(idx) = vec.iter().position(|&x| x == target) {
    vec.remove(idx);
}

// 3. Remove FIRST occurrence by value (Fast, Unordered)
if let Some(idx) = vec.iter().position(|&x| x == target) {
    vec.swap_remove(idx);
}

```

---

### 4. Common Compiler Fixes & Gotchas

#### A. Type Mismatch: `u32` vs `Option<&u32>`

* **Problem:** Comparing an iterator reference `&x` directly with `.min()` result (`Option<&u32>`).
* **Fix:** Pattern match or unwrap the option before comparing values.
```rust
if let Some(&min_num) = vec.iter().min() {
    if let Some(index) = vec.iter().position(|&x| x == min_num) { ... }
}

```



#### B. `Vec::remove()` Return Type & Mutability

* **Mutability:** `.remove()` mutates the vector in-place. The vector variable must be declared `let mut vec = ...`.
* **Return Value:** `vec.remove(index)` returns the removed **element `T` directly**, not an `Option` or `Result`.
* **No `?` or `.unwrap()`:** Do not apply `?` or `.unwrap()` to `vec.remove()` or the returned vector itself.

#### C. Pattern: Remove Smallest Element from Slice

```rust
fn remove_smallest(numbers: &[u32]) -> Vec<u32> {
    let mut vec1 = numbers.to_vec();

    if let Some(&min_num) = vec1.iter().min() {
        if let Some(index) = vec1.iter().position(|&x| x == min_num) {
            vec1.remove(index);
        }
    }

    vec1
}

```


