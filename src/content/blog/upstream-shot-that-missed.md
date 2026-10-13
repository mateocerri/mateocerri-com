---
title: "The bug was a missing .exe"
date: 2026-10-12
description: "I went upstream to kache with an RFC and a fix. Both landed through the maintainer, under his name. That still counts."
---

I did not go looking for an open-source project to contribute to. I was
trying to give my coding agent IDE capabilities on a Windows repo.
opencode V2 had dropped LSP support, so I studied the options for
agent-grade code intelligence. Rust-analyzer was out. With 8 GB of RAM
I could not run VS Code plus rust-analyzer together on a big project,
and Zed, still green, uses rust-analyzer under the hood anyway. That led
me toward a lightweight LSP in the Glancer style (under 100 MB at rest).
From there I hit the real bottleneck: slow cargo and a giant `target/`
directory, with `cargo clean` as the all-or-nothing answer
that forces a full rebuild. [Kache](https://github.com/kunobi-ninja/kache),
a zero-copy content-addressed build cache for Cargo, already solved the
dedup and caching half. Duplicating that as my own tool made no sense.
Contributing where users and infra already exist beats maintaining a
project from zero. The gap was precise cleanup with a verifiable
criterion, plus a machine-readable signal for light tools. So I wrote
the RFC instead of forking.

The contribution started big: an RFC proposing exact liveness for
per-artifact garbage collection. Cargo's own freshness rules in place
of the `atime` heuristic, which lies under `noatime` or `relatime`.
Plus rebuild events consumable by external tools. My acceptance
criterion was blunt: after cleaning, the next `cargo check` must be a
no-op. If it rebuilds, the tool was wrong. Two days later the
maintainer implemented it himself as
[PR #1485](https://github.com/kunobi-ninja/kache/pull/1485). It merged
the same day, and my issue closed with zero comments. Adopted,
not answered.

I wanted to contribute to the Rust ecosystem somehow, and with the first
idea already built, I moved on to the other gap I had found. On a stock
Windows install, `kache cargo`
fails before doing anything:

```
error: finding Cargo behind the Kache shim
```

The resolver walks `PATH` looking for a file literally named `cargo`.
But Rust toolchains on Windows only ship `cargo.exe`. No `cargo`, no
build. The only workaround was setting `KACHE_REAL_CARGO` by hand,
which took me a while to even find.

My fix was narrow on purpose: try the bare name first, then `name.exe`
on Windows, Unix behavior untouched. Only `.exe`. A `.bat` wrapper
must never be picked as the real compiler. Tests for the candidate
order, empty and duplicate extensions, case-insensitive dedup, and a
wiring test pinning that off Windows no suffix is tried at all.
Developed on Debian under WSL2, then built and verified on Windows
itself: `kache.exe cargo -- --version` printed the toolchain version
with no env var set, where before it failed.

That pull request ([#1537](https://github.com/kunobi-ninja/kache/pull/1537))
did not merge directly either. The maintainer adopted the commit into
his own [PR #1539](https://github.com/kunobi-ninja/kache/pull/1539) with
extra tests, merged it, and closed mine as superseded, keeping my
co-author credit in the squash.

Neither thread merged under my name, and both landed. I read a foreign
Rust codebase end to end. I learned to package a proposal as a concrete
problem with scope and an acceptance test, which carried more weight
than the idea alone (the maintainers were already working that area).
I learned how small an upstream diff should be. That is the real
contribution story. The motive was never the badge. It
was killing `cargo clean`.

## Recap

- I went upstream for a selfish reason: slow builds on a thin machine. The
  best open-source motives usually are.
- Packaging the RFC as problem plus scope plus acceptance test mattered
  more than the idea. The maintainers were already in that area and built
  it in two days.
- The small fix taught more than the big proposal: one Windows-only edge,
  a narrow diff, tests for the ordering. Small is what merges.

> **Takeaway.** Neither thread merged under my name, and both landed. What
> matters is that the software is better now. That part counts.
