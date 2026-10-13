---
title: "Giving my agent tools that show their work"
date: 2026-10-12
description: "Two small OpenCode plugins, a script registry and anchor-based edits, and the 70-file rename that proved them."
---

Every session my agent reinvented the same helpers: diffing two PNGs,
taking a census of a log directory, trimming a noisy trace. And every
session it wrote files the same way, through opaque heredocs in the shell,
invisible in review. Two small plugins fixed both habits.

[Scriptbox](https://github.com/mateocerri/opencode-scriptbox) is a
closed-world registry of reusable scripts. Helpers get a canonical home
with a purpose header and a descriptive name, and the agent lists them on
demand instead of rewriting them. Save once, reuse by name.

[Patchflow](https://github.com/mateocerri/opencode-patchflow) makes edits
auditable. Anchor-based atomic batch edits: path, anchor, replacement,
expected match count, with a dry run first. One reviewable call instead
of N silent writes.

The live demo: renaming a field type across 70 files of a benchmark
happened in a single patchflow call. That call is the whole philosophy.
Agents that cost fewer tokens and show their work. Two more plugins in the family
([slim-output](https://github.com/mateocerri/opencode-slim-output),
[wsl-shim](https://github.com/mateocerri/opencode-wsl-shim)) share it.

## Recap

- Agents rewrite helpers every session unless the helpers have a canonical
  home. A registry with names and purposes beats regeneration.
- Opaque shell writes are unauditable. Anchor-based edits with a dry run
  turn N silent writes into one reviewable call.
- Small, composable plugins beat one big framework: each tool does one
  thing and shows its work.

> **Takeaway.** There is a specific pleasure in fixing a small thing well
> and knowing strangers will trip over it less. Open source runs on that
> feeling as much as on the code.
