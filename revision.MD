# Rust Technical Interview Reference & Key Takeaways

A personal reference guide for writing idiomatic, high-performance, and memory-efficient Rust solutions for technical coding interviews (LeetCode/DSA).

---

## 🛠️ Fundamental Syntax & Types

### Primitive Types & Type Casting (`i32` vs. `usize`)z
* **Why:** Rust does not implicitly convert numbers. Array/vector indexing strictly requires `usize`, while DSA math often operates on `i32`.
* **Takeaway:** Always cast integer indices using the `as` keyword when accessing arrays (e.g., `nums[idx as usize]`).

### Iterators & Range Syntax (`0..nums.len()` vs. `0..n`)
* **Why:** Writing `0..n` (where `n: i32`) produces `i32` loop variables, triggering compilation errors when indexing slices.
* **Takeaway:** Prefer `0..nums.len()` because `.len()` naturally returns `usize`, keeping loop variables typed correctly for indexing without explicit casts.

### Ownership & Parameter Mutability (`mut nums: Vec<i32>`)
* **Why:** Function parameters in Rust are immutable bindings by default. In-place algorithms require explicit mutation privileges.
* **Takeaway:** Re-bind input arguments with `mut` in the function signature (e.g., `pub fn build_array(mut nums: Vec<i32>)`) to perform $O(1)$ auxiliary space modifications.

### Statements vs. Expressions (Semicolons `;`)
* **Why:** Omitting a semicolon turns a statement into a return expression.
* **Takeaway:** Use `;` to terminate side-effect statements (like `nums[i] /= n;`). Omit the semicolon on the final line of a block or function to return that value implicitly.

---

## 🚀 Memory Optimization & Data Structures

### Returning Arrays (`[T; N]`) vs. Vectors (`Vec<T>`)
* **Fixed-size Arrays (`[T; N]`):** Stack-allocated, fixed size known at compile time. Ideal for small, constant returns (e.g., coordinates, RGB values).
* **Vectors (`Vec<T>`):** Heap-allocated, dynamic size determined at runtime. Default choice for variable-length collection requirements.

### Single-Lookup Entry API (`HashMap::entry`)
* **Why:** Combining `contains_key()` with `.get()` or `.insert()` calculates the key's hash twice.
* **Takeaway:** Use `match map.entry(key)` to compute the hash location **exactly once**.
  ```rust
  use std::collections::hash_map::Entry;

  match map.entry(key) {
      Entry::Occupied(entry) => {
          let val = entry.get(); // Access existing value
      }
      Entry::Vacant(entry) => {
          entry.insert(default_val); // Insert missing value
      }
  }



```markdown
# Rust DSA Interview Cheat Sheet & Templates

Idiomatic, copy-pasteable Rust code templates for technical interview patterns, complete with line-by-line annotations explaining indexing safety, compiler constraints, and performance traits.

---

## 1. Matrix Transformations

### In-Place Symmetric Swap + Invert ($O(1)$ Auxiliary Space)
Reverses each matrix row and flips bits ($0 \leftrightarrow 1$) in a single pass without extra memory allocation.

```rust
pub fn flip_and_invert_image_inplace(mut image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n = image.len();

    for i in 0..n {
        // Stop at (n + 1) / 2 to reach the middle element.
        // Going beyond (n / 2) would re-swap elements back to original positions.
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

Clean, idiomatic Rust functional style using iterator combinators. Preferred when immutability is favored.

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

Tracks state in a single linear pass to avoid $O(N^2)$ nested comparison loops.

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

Uses array values as pointer offsets (`nums[i] as usize`) to detect graph cycles without modifying the array.

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

    // Phase 2: Reset slow to origin index 0; advance both pointers by 1 step.
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

## 5. Local Rust Playground Harness

Boilerplate wrapper to compile and test LeetCode solution logic locally in standard Rust execution environments.

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
Here is the summary of today's learning, dated **September 7, 2026**, formatted and annotated to match the clean, readable style of your revision guide. You can copy and paste this directly into your main `.md` file.

```markdown
## 📅 Revision Entry: September 7, 2026

---

### 1. 🛠️ Rust String Formatting & Compiler Constraints

#### Avoid In-Line Indexing in Format Strings (`format!`)
* **Why:** Rust's string interpolation only supports simple variable names inside braces `{name}`. Complex expressions like `names[0]` trigger compiler errors (`expected '}', found '['`).
* **Takeaway:** Pass array or slice indices as explicit positional arguments after the format string.
  ```rust
  // ❌ WRONG: Compiler error
  format!("{names[0]} likes this");

  // ✅ CORRECT:
  format!("{} likes this", names[0]);

```

#### Display `{}` vs. Debug `{:?}` Specifiers

* **Why:** Using `{:?}` on string types wraps output in literal escape quotes (`"\"Peter\" likes this"`), causing test assertion failures.
* **Takeaway:** Always use `{}` (Display trait) for clean user-facing text and `{:?}` (Debug trait) strictly for inspecting data structures.

#### Return Types in `if` Blocks Without `else`

* **Why:** An `if` statement without an `else` branch must evaluate to `()` (unit type). Returning a value like `format!(...)` inside it results in a type mismatch error (`expected (), found String`).
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

* **Why:** Functions taking `&Vec<T>` or fixed-size array references `&[T; N]` restrict caller flexibility.
* **Takeaway:** Accept slice references `&[T]` in function parameters to support vectors, fixed arrays, and partial array slices seamlessly via deref coercion.

#### Formatting Slices vs. Formatting Individual Elements

* **Why:** Passing a slice range like `&numbers[0..3]` into `format!("({})", ...)` prints the debug array representation `"[1, 2, 3]"` instead of raw digits.
* **Takeaway:** Either index elements individually (`numbers[0]`) or collect elements into a intermediate `String` before taking string slice ranges (`&s[0..3]`).

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


```markdown
## 📅 Revision Entry: September 8, 2026

---

### 1. 🛠️ Compiler Constraints & Move Semantics

#### Trait Constraints in Closures (`E0507: cannot move out of a shared reference`)
* **Why:** Pattern matching with `|&x|` in closures (like `vec.retain(...)`) tries to copy the value out of the reference. If type `T` does not implement `Copy` (e.g., `String`, `Vec`, or generic `T`), Rust blocks the move to prevent invalidating shared memory.
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

* **Why:** Rust does not support the `**` operator for exponents. Using `x ** x` causes a compilation error.
* **Takeaway:** Use basic multiplication (`x * x`) for squaring or `.pow(n)` for general integer exponentiation.

---

### 2. ⚡ Numeric & Square Root Patterns

#### Integer Square Root Compatibility (`.isqrt()` vs `.sqrt()`)

* **Method Availability:** `.isqrt()` is available in Rust **1.84+** on primitive integer types (`u32`, `i32`, `usize`).
* **Legacy / Online Judge Fallback:** Online competitive coding platforms on older toolchains will fail with `E0599` on `.isqrt()`. Use floating-point conversion as a cross-platform fallback:
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








