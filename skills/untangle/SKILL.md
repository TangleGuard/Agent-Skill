---
name: untangle
description: >-
  Break a circular dependency or an architecture rule violation through a
  guided refactor: find the cycles, pick the cheapest edges to cut using
  TangleGuard's ranked cut list and import-level evidence, apply the refactor,
  and verify the tangle is gone. Use when the user asks to untangle, break,
  fix, or remove a circular dependency, cycle, tangle, or dependency rule
  violation — or after `validate`/`check-circles` reported one that should now
  be fixed.
---

# Untangle: Break a Cycle with Evidence, Not Guesswork

Requires `tangleguard-cli` on the PATH (install: `brew install --cask
tangleguard-cli`, see https://tangleguard.com/apps/cli). All commands take
`-l <language>` and optionally `-p <path>`.

## Workflow

### 1. Find the tangle

```bash
tangleguard-cli -q -l <language> check-circles
```

Simple cycles (2 modules) list the edges directly. Tangles (3+ mutually
dependent modules) come with a **ranked cut list** — the fewest edges to
remove to dissolve the tangle — with the exact import statements behind each
cut (`file:line` + statement) already inlined. That cut list is the
refactoring plan; do not invent your own.

If the user pointed at a specific cycle or pair of modules, focus on that one.
Otherwise fix the worst tangle first and confirm before continuing to others.

### 2. Weigh the cuts

For each candidate edge, read the evidence — the imported items and the
statements behind it (already in the cut list; for edges outside a tangle run
`explain-dependency --from <a> --to <b>`). Cheap cuts first:

- **Type-only or single-item imports** — cheap: move the type/interface to a
  lower layer both sides may depend on.
- **Shared helpers imported by both sides** — extract them into a common
  module below both.
- **Broad runtime imports** — expensive: invert the dependency (callback,
  trait/interface, event) or move the dependent code to where the dependency
  points.

### 3. Refactor — with preflight

Apply the cut. Before writing any **new** import the refactor introduces,
preflight it so the fix doesn't create the next violation:

```bash
tangleguard-cli -q -l <language> check-import --from <source> --to <target>
```

`DENIED` means pick a different target — typically one layer lower.

### 4. Verify

```bash
tangleguard-cli -q -l <language> check-circles   # the cycle is gone
tangleguard-cli -q -l <language> validate        # and nothing new broke
```

Both clean → done. Report what was cut, where the code moved, and why that
edge was the cheapest.
