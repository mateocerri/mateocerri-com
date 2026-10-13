---
title: "The binary follows the data"
date: 2026-10-13
description: "My agent lives between Windows and Linux, where every command must pick the right version of the same tool. A small shim answers that question once, so nothing trips at the boundary."
---

My agent lives in WSL and my files live in two worlds. Every shell call
carried an unspoken question: which `git`, the ELF one or the
`git.exe` one? Get it wrong and you pay in slow 9P file access, confused
line endings, or tools that half-work across the boundary. Worse, the
wrong answer is often silent. The command runs, the output looks right,
and the damage shows up three steps later.

[opencode-wsl-shim](https://github.com/mateocerri/opencode-wsl-shim)
answers the question once, with a rule instead of a lookup table:
**the binary follows the data**. Your files live in one world, so run that
world's native binary. Linux checkout under WSL, run the ELF. Windows
checkout, run the PE. Never mix toolchains across the boundary within a
single operation. A mixed pipeline is where corruption stops
being reproducible: half the tools normalizing line endings one way,
the other half the other way, and no deterministic rule to blame.

The uncrossed quadrants are trivial. The crossed ones needed a closer
look. A Windows binary called from WSL works through interop, but it
runs blind to inotify on `/mnt` drives and pays the 9P tax on every file
it touches. Fine for a quick query, wrong as a default. A Linux binary
called from Windows only exists behind `wsl.exe`, adding a VM boundary
crossing to every invocation plus the socket and path translation that
comes with it. The shim's answer to both is the same: translate toward
the world that owns the working directory, and when the expected binary
is missing, fall back to whichever one exists, but say so. The first
time per session that a rescue happens, the agent gets a one-line notice
appended to the result. Silent translation would just move the confusion
one layer down.

The map itself is deliberately boring: 102 tools, each admitted under
one rule. Same vendor, same flags, a real `.exe` on the other side.
Ten common names were rejected after probing: `npm`, `yarn`, `gradle`
and friends ship `.cmd` shims on Windows, and a `.cmd` is not an
executable. Feed one to the interop layer and bash tries to run it as
a shell script. Containers got their own category,
`NEVER_SHIM`: `docker` already owns a WSL integration, and translating
its calls would fight it. Version managers stay out on principle: they
belong to the shell's scope, not the project's, with `rustup` as the
lone exception that proves it.

One curiosity from the trenches: a stale shell hash once convinced me
`gh.exe` was shadowing the native `gh` long after brew had installed
it. The shim's rule would have picked correctly all along. The
confusion was in my shell, not in the rule. Rules beat memory.

## Recap

- Every cross-platform shell call asks which binary. Answer with a
  rule (the binary follows the data), not case-by-case luck.
- Crossed quadrants work but cost: no inotify on `/mnt`, 9P latency,
  VM crossings. Translate toward the world that owns the files.
- Fallbacks must be loud: one notice per binary per session keeps the
  agent's model of the world honest.
- Admission rule for the map: same vendor, same flags, real PE. `.cmd`
  wrappers and container CLIs need not apply.
