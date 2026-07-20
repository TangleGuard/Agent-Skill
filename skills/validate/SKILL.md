---
name: validate
description: >-
  Architecture guardrail: preflight a change BEFORE making it — whether an
  import/dependency is allowed by the rules, whether it would create a
  circular dependency, where a piece of code belongs — and validate the
  workspace after a change for rule violations and cycles. Use BEFORE adding
  a dependency between modules, moving code between layers, or any structural
  change, and AFTER such a change to confirm the architecture is still clean.
---

# Validate: Check Before You Change It

TangleGuard enforces the architecture declared in `tangleguard.json` (layers
and dependency rules) and detects circular dependencies. Use it in both
directions: **preflight** before writing a change so violations never enter
the codebase, and **validate** after a change to confirm nothing broke.

The binary is `tangleguard-cli`. If it is not on the PATH, ask the user to
install it (`brew install --cask tangleguard-cli`, other options at
https://tangleguard.com/apps/cli). Commands need the language (`-l`) and
optionally a path (`-p`, defaults to the current directory); the config-only
checks (`placement`, `check-dependency`) read `tangleguard.json` without
scanning and need no `-l`. Supported languages include `rust`, `go`,
`typescript` / `javascript`, `python`, `kotlin`, `java`, `csharp`, `php`, and
`dart`.

## Preflight (before the change)

The default preflight is **`check-import`** — one combined verdict covering
both the dependency rules and circular dependencies:

```bash
tangleguard-cli -q -l <language> [-p <path>] check-import --from <source> --to <target>
```

The first line is `ALLOWED` or `DENIED`; a denial names the violated rule
and/or the exact cycle the edge would close. `--from`/`--to` accept file
paths relative to the workspace root (the files you are editing) or
`pkg::module` names. **If it returns DENIED, do not add the edge — choose a
design the architecture allows.**

Single-signal variants when you only need one answer. A `<node>` is a
workspace path like `apps::web` or a layer name like `Apps`:

```bash
# Where does a node belong, and what may it wire to? (config-only, no -l)
tangleguard-cli [-p <path>] placement --node <node>

# Is a dependency edge allowed by the rules? (config-only, no -l)
tangleguard-cli [-p <path>] check-dependency --from <node> --to <node>

# Would adding a dependency create a circular dependency? (scans, needs -l)
tangleguard-cli -l <language> [-p <path>] would-create-cycle --from <node> --to <node>
```

- **`placement`** lists the node's layer plus every layer/path it may depend
  on, and every one that may depend on it — each annotated with the rule that
  permits it. Ask this when deciding where new code belongs, instead of
  probing edges one at a time.
- **`check-dependency`** reports allowed or denied for a single edge; a
  denial also lists the allowed alternatives from the source.
- **`would-create-cycle`** reports whether the edge closes a loop and prints
  the existing path it would close.

Add `--ci` to `check-import` / `check-dependency` / `would-create-cycle` to
exit non-zero on a denial (useful as a commit/CI guardrail).

## Validate (after the change)

```bash
tangleguard-cli -q -l <language> [-p <path>] validate         # rule violations + cycles
tangleguard-cli -q -l <language> [-p <path>] validate-rules   # rule violations only
tangleguard-cli -q -l <language> [-p <path>] check-circles    # circular dependencies only
```

Violations come back with `file:line` evidence — open those locations and fix
the code. Never edit `tangleguard.json` to make `validate` pass; rule changes
are a human decision. If `check-circles` reports a tangle, the untangle skill
walks through breaking it.

## A Typical Flow

1. About to change an existing block → run `context --node <file-or-name>`
   (architecture skill) for its rules, actual edges, and blast radius.
2. Deciding where new code belongs → ask `placement`.
3. About to add or change an import → run `check-import` with the two files;
   on DENIED pick a different design instead of adding the edge.
4. Make the change.
5. Run `validate` and fix anything it reports at the given `file:line`.
