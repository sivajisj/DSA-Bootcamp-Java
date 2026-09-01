# Rust Technical Interview Reference & Key Takeaways

A personal reference guide for writing idiomatic, high-performance, and memory-efficient Rust solutions for technical coding interviews (LeetCode/DSA).

---

## 🛠️ Fundamental Syntax & Types

### Primitive Types & Type Casting (`i32` vs. `usize`)
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
