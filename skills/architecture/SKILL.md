---
name: architecture
description: >-
  Inspect a codebase's real architecture — layers, packages, lines of code,
  the package dependency graph, and the evidence behind any dependency (the
  exact imports, file:line) — by querying the TangleGuard CLI instead of
  reading files and guessing. Use when orienting in an unfamiliar codebase,
  when asked how the code is organized or what depends on what, and before
  any architectural or design discussion.
---

# Architecture: Query the Structure, Don't Guess It

TangleGuard is a static analysis tool (native Rust, extremely fast) that
extracts a codebase's structure deterministically. If you only need the
structure, ask TangleGuard instead of reading source files: it is faster,
cheaper in tokens, and grounded in facts rather than inference from file
paths.

The binary is `tangleguard-cli`. If it is not on the PATH, ask the user to
install it (`brew install --cask tangleguard-cli`, other options at
https://tangleguard.com/apps/cli) — do not fall back to guessing the
architecture. Commands need the language (`-l`) and optionally a path (`-p`,
defaults to the current directory). Output is markdown, ready to drop into
your reasoning; add `--format json` to parse it programmatically. Supported
languages include `rust`, `go`, `typescript` / `javascript`, `python`,
`kotlin`, `java`, `csharp`, `php`, and `dart`.

## Get the big picture

```bash
tangleguard-cli -q -l <language> [-p <path>] architecture
```

Prints a compact summary: the layers, the allowed dependency rules, the
packages (with their resolved layer and lines of code), and the package-level
dependency graph. Run this first — one call instead of reading dozens of
files.

## Explain a dependency (evidence behind an arrow)

```bash
tangleguard-cli -q -l <language> [-p <path>] explain-dependency --from <source> --to <target>
```

The architecture summary shows _that_ a dependency exists; this shows _why_:
every reference behind it — imported items, source `file:line`, and the exact
import statement. The statements reveal whether a dependency is a single
type-only import (cheap to move) or a broad runtime one. `--from`/`--to`
accept `pkg::module` prefixes (the package segment may be omitted) or file
paths; `--to` also accepts an external package name to list every usage of a
third-party dependency.
