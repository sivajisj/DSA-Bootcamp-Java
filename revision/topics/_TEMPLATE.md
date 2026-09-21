# 🔧 &lt;Topic Name&gt; — Full DSA Interview Revision

> Copy this file to `topics/<topic-name>.md` (lowercase, hyphenated) and delete this blockquote. Keep the shape below — it's what makes every topic file feel like the same trusted toolbox instead of a random pile of notes. Look at [`arrays.md`](arrays.md) for a fully filled-in example.

**Mission:** one sentence — what should you be able to *do* after mastering this file, under interview pressure?

**How to use this file:**
- **Night before an interview:** read the Speed Read section only.
- **Learning a new pattern:** read the section, the analogy, then type the code by hand without copy/paste.
- **After every mock/LeetCode session:** append a dated entry to the Dated Log with whatever tripped you up.

This file assumes you've read the Rust-specific landmines in [`../README.md` Part 1](../README.md#part-1-the-rust-mental-model). Everything here is `<topic>`-specific.

---

## Table of Contents
1. [Pattern Recognition Map](#1-pattern-recognition-map)
2. [Pattern One](#2-pattern-one)
3. [...](#)
4. [N-Minute Speed Read](#n-minute-speed-read)
5. [Dated Revision Log](#dated-revision-log)

---

## 1. Pattern Recognition Map

The single most valuable skill in any round is mapping the *problem smell* to a *pattern* before writing a line of code. Fill this in first, before writing a single code example — it forces you to know the shape of the whole topic before you drill into it.

| Problem smell | Pattern | Section |
|---|---|---|
| "..." | ... | [§2](#2-pattern-one) |

---

## 2. Pattern One

**Where you'll meet it:** one or two sentences on what tips you off that this is the right tool — the "smell" that should make this pattern pop into your head.

**The analogy:** a real-world comparison that makes the mechanism click without needing the code. Good analogies survive being explained to someone who has never coded.

```rust
// Runnable, tested Rust. Comment the *why*, not the *what* — the code already
// says what it does; the comment should explain a non-obvious constraint or trick.
```

Add a complexity note (time/space) and, where it clarifies a decision, a small comparison table against the naive approach.

---

## N. N-Minute Speed Read

**Step 1 — classify the problem** using the [Pattern Recognition Map](#1-pattern-recognition-map).

**Step 2 — pick the right invariant** for whichever pattern applies (one line per pattern, table form works well — see `arrays.md` §9 for the shape).

**Step 3 — panic-proof before you submit:** topic-specific edge cases (empty input, single element, overflow, off-by-one bounds, etc).

**Step 4 — state complexity out loud.**

---

## Dated Revision Log

Keep appending dated entries below whenever a mock interview or LeetCode session teaches you something new about this topic specifically. General Rust/compiler lessons still go in `../README.md`'s log.

### 📅 &lt;Month Day, Year&gt;
What you covered, in one or two sentences — specific enough that future-you can tell at a glance whether a topic has already been drilled.

<!-- Add your next entry below this line -->
