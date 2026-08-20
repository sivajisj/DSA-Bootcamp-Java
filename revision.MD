### Key Rust Takeaways

* **String Ownership:** Returning owned data created dynamically inside a function (like `"something".to_string()`) requires changing the return type to `String`. Alternatively, return string literals with `'static` references (`&'static str`).
* **What `usize` is:** `usize` is an unsigned integer type whose size dynamically matches your computer's architecture (32-bit or 64-bit).
* **Why `usize` is required:** Rust strictly enforces using `usize` for all array and vector indexing because memory addresses and array lengths depend on pointer size.
* **How to fix indexing errors:** Convert integer types like `i32` to `usize` using the `as` keyword (e.g., `nums[idx as usize]`).

Primitive Types & Type Casting (i32 vs. usize)

Why: You hit repeated compiler errors because Rust does not implicitly convert integers. Array indexing strictly requires usize, while algorithm math often works in i32.

Focus: How as usize works and type requirements for slice operations.

Iterators & Range Syntax (0..nums.len() vs. 0..n)

Why: The expression 0..n (where n is i32) produces i32 loop variables, breaking array access. Using 0..nums.len() naturally infers usize.

Focus: Type inference in Ranges and using .len() directly in loops.

Ownership and Parameter Mutability (mut nums: Vec<i32>)

Why: In-place algorithms require modifying input arguments. By default, function arguments in Rust are immutable bindings.

Focus: Re-binding function parameters with mut and understanding when values are moved vs. borrowed.

Rust Statements vs. Expressions (Semicolons ;)

Why: You hit a syntax error on nums[i] /= n due to a missing semicolon inside the loop body.

Focus: The difference between returning an expression value and terminating a statement with ;.

Idiomatic Iteration (Functional Style - Optional)

Why: Once comfortable with indexed for loops, Rust developers often write in-place updates using mutable iterators to avoid index operations entirely.

Focus: for val in nums.iter_mut() and method chaining.
