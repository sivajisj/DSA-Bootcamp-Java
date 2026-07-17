# Rust Readiness Review of This Assignment Set

A review of how well these assignments prepare you to solve DSA problems **in Rust**. All existing files (01–17) and their links are kept as-is; files 18–25 fill the gaps identified below.

## Rating

| Scope | Rating | Notes |
|---|---|---|
| Original set (01–17) | **6 / 10** | Strong foundations: binary search, sorting, recursion/backtracking, linked lists, stacks/queues, trees. But missing the topics that dominate interviews: hashing, graphs, DP, heaps, sliding window. |
| With new files (18–25) | **9 / 10** | Covers everything needed for LeetCode-style interviews. Remaining 1 point: language-agnostic lists can't teach Rust idioms — that comes from actually solving them in Rust. |

## What was missing (now added)

| File | Topic | Why it matters |
|---|---|---|
| 18 | Hashing (HashMap / HashSet) | The single most-used technique in interviews; was completely absent. |
| 19 | Two pointers, sliding window, prefix sums | Core array/string patterns; previously only implicit in a few problems. |
| 20 | Heaps / priority queue | Top-K, scheduling, median-of-stream patterns. |
| 21 | Greedy + intervals | Merge intervals, jump game, activity selection. |
| 22 | Graphs (BFS/DFS, topo sort, Dijkstra, DSU, MST) | Biggest gap — zero dedicated graph coverage before. |
| 23 | Dynamic programming | Second-biggest gap; only touched indirectly via recursion. |
| 24 | Tries + KMP/Rabin-Karp | Prefix trees and pattern matching. |
| 25 | Monotonic stack, segment tree, BIT | Advanced but frequently asked at top companies. |

Already well covered by 01–17 (no changes needed): binary search (excellent), sorting, recursion & backtracking (N-Queens, Sudoku, combination sums), bit manipulation, math, linked lists, stacks/queues, trees.

## Rust-specific notes

Most LeetCode problems accept Rust submissions. The problem lists here are language-agnostic, so they work fine — but know these Rust realities:

### Standard library maps directly to these topics
- `Vec<T>` — arrays/lists; `VecDeque<T>` — queue/deque; `HashMap`/`HashSet` — hashing; `BTreeMap`/`BTreeSet` — ordered maps; `BinaryHeap<T>` — **max**-heap (wrap in `std::cmp::Reverse` for a min-heap).
- Slices: `windows()`, `chunks()`, `sort_unstable()`, `binary_search()` are built in.

### Where Rust is harder than Java (practice these early)
- **Linked lists (file 15):** LeetCode uses `Option<Box<ListNode>>`. Ownership makes pointer juggling tricky — learn `take()`, `as_mut()`, and `while let` patterns. Recommended reading: *Learning Rust With Entirely Too Many Linked Lists*.
- **Trees (file 17):** LeetCode uses `Rc<RefCell<TreeNode>>`. Practice `borrow()`/`borrow_mut()` and cloning `Rc` handles before attempting medium tree problems.
- **Graphs (file 22):** skip pointer-based node structs entirely — use index-based adjacency lists (`Vec<Vec<usize>>`). This is the idiomatic Rust approach and avoids borrow-checker fights.

### Where Rust differs in small ways
- **OOP (file 14):** Rust has no inheritance — translate the concepts to structs, enums, traits, and trait objects.
- **Math/bitwise (files 11–12):** integer overflow panics in debug builds; use `i64`, `checked_*`, or `wrapping_*` methods deliberately.
- **Strings (file 08):** `String` is UTF-8, not indexable by position — convert with `.chars().collect::<Vec<char>>()` or work with `.as_bytes()` for ASCII problems.
- **DP (file 23):** 2D tables are just `vec![vec![0; cols]; rows]`.

### Suggested order for a Rust learner
1. 01–05 basics → solve in Rust to learn ownership on simple problems
2. 18–19 (hashing, two pointers/sliding window) → learn the std collections
3. 06–07, 10 (search, sort, recursion)
4. 15–17 (linked list, stack/queue, trees) → the ownership deep end
5. 20–22 (heaps, greedy, graphs)
6. 23 (DP) → 24–25 (tries, advanced)
