### Key Rust Takeaways

* **String Ownership:** Returning owned data created dynamically inside a function (like `"something".to_string()`) requires changing the return type to `String`. Alternatively, return string literals with `'static` references (`&'static str`).
* **What `usize` is:** `usize` is an unsigned integer type whose size dynamically matches your computer's architecture (32-bit or 64-bit).
* **Why `usize` is required:** Rust strictly enforces using `usize` for all array and vector indexing because memory addresses and array lengths depend on pointer size.
* **How to fix indexing errors:** Convert integer types like `i32` to `usize` using the `as` keyword (e.g., `nums[idx as usize]`).
