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
# Rust DSA Cheat Sheet & Code Templates

Quick-reference code snippets and standard templates for common interview algorithmic patterns in Rust.

---

## 1. Matrix Transformations

### In-Place Symmetric Swap + Invert ($O(1)$ Space)
Used for reversing rows in-place and flipping bits simultaneously.
```rust
pub fn flip_and_invert_image_inplace(mut image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n = image.len();
    for i in 0..n {
        // Bound loop to middle to prevent double-swapping
        for j in 0..(n + 1) / 2 {
            let k = n - 1 - j; // Opposite index
            let temp = image[i][j] ^ 1;
            image[i][j] = image[i][k] ^ 1;
            image[i][k] = temp;
        }
    }
    image
}

```

### Functional Chain Transformation ($O(N^2)$ Space)

Idiomatic, functional row reflection and bit flip using iterators.

```rust
pub fn flip_and_invert_image_functional(image: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    image
        .into_iter()
        .map(|row| {
            row.into_iter()
                .rev()          // Reverse horizontal row
                .map(|b| b ^ 1) // Invert bit (0 -> 1, 1 -> 0)
                .collect()
        })
        .collect()
}

```

---

## 2. Dynamic Sliding & One-Pass Tracking

### Running Minimum / Maximum (Best Time to Buy & Sell Stock)

Single-pass $O(N)$ tracking without explicit window state.

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

---

## 3. Floyd’s Cycle Detection (Pointer Jumping)

### Find the Duplicate Number ($O(N)$ Time, $O(1)$ Space)

Treating array values as graph pointers (`nums[x as usize]`).

```rust
pub fn find_duplicate(nums: Vec<i32>) -> i32 {
    // Phase 1: Initialize slow (1 jump) and fast (2 jumps)
    let mut slow = nums[0];
    let mut fast = nums[nums[0] as usize];

    // Phase 1 Loop: Advance until pointers collide in the loop
    while slow != fast {
        slow = nums[slow as usize];
        fast = nums[nums[fast as usize] as usize];
    }

    // Phase 2: Reset slow to origin; step both by 1 jump to find intersection
    slow = 0;
    while slow != fast {
        slow = nums[slow as usize];
        fast = nums[fast as usize];
    }

    slow // Intersection point is the duplicate value
}

```

---

## 4. Single-Lookup HashMap Pattern

### Frequency Counting with `Entry` API

Prevents double hashing overhead during insertion and updates.

```rust
use std::collections::HashMap;

pub fn count_frequencies(nums: Vec<i32>) -> HashMap<i32, usize> {
    let mut counts = HashMap::new();
    for num in nums {
        *counts.entry(num).or_insert(0) += 1;
    }
    counts
}

```

---

## 5. Standard Playground Runner Template

Boilerplate structure to test LeetCode solution functions locally in Rust Playground.

```rust
struct Solution;

impl Solution {
    // Paste solution method here
}

fn main() {
    let input = vec![
        vec![1, 1, 0],
        vec![1, 0, 1],
        vec![0, 0, 0],
    ];

    let result = Solution::flip_and_invert_image_functional(input);
    
    println!("Output:");
    for row in result {
        println!("{:?}", row);
    }
}

```

```

```
