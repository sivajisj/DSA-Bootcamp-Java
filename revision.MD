# 🦀 Rust DSA Interview Revision Vault

**This file has moved and grown up.** Everything that used to live here now lives in **[`revision/README.md`](revision/README.md)**, plus a new `revision/topics/` folder holding one deep-dive file per data structure or algorithm family — starting with a heavily expanded array guide.

👉 **Start here:** [`revision/README.md`](revision/README.md)
👉 **Arrays deep dive:** [`revision/topics/arrays.md`](revision/topics/arrays.md)

## Why the split?

The old single-file vault worked fine while it only covered a handful of patterns, but "one giant markdown file" doesn't scale as more topics (strings, trees, graphs, DP, ...) get added — it turns into an unsearchable wall of text. The new shape:

```
revision/
├── README.md              ← general Rust mental model, compiler fix-it table,
│                             test harness boilerplate, pre-interview speed read,
│                             and the general dated revision log
└── topics/
    ├── _TEMPLATE.md        ← copy this to start a new topic file
    ├── arrays.md           ← ✅ live: two pointers, sliding window, prefix sum,
    │                             Kadane's, binary search, in-place tricks, etc.
    └── (strings.md, trees.md, graphs.md, ... added the same way as they're drilled)
```

Nothing was deleted — every pattern, analogy, and dated log entry from the old `revision.MD` is preserved, just reorganized so array-specific material sits together in one focused, easy-to-drill file instead of being mixed in with general Rust gotchas. See [`revision/README.md` Part 8](revision/README.md#part-8-topic-files-deep-dives) for how to add the next topic yourself.
