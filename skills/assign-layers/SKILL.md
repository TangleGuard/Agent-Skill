---
name: assign-layers
description: >-
  Map a codebase's components onto a target architecture: the user picks the
  architecture (a template like layered/hexagonal/clean, or their own layers),
  the agent assigns each component to a layer by its ROLE, and the resulting
  rule violations become the migration backlog toward that architecture. Use
  when the user wants to adopt or migrate to an architecture ("make this a
  clean architecture", "assign my components to layers", "how far is my code
  from a layered architecture"), or after applying a template that left
  components unassigned.
---

# Assign Layers: Map Components onto a Target Architecture

Requires `tangleguard-cli` on the PATH (install: `brew install --cask
tangleguard-cli`, see https://tangleguard.com/apps/cli). Commands take
`-l <language>` and optionally `-p <path>`.

## The core principle: assign by role, not by fit

Assign each component to the layer where it **belongs by what it is** — a
controller belongs in the presentation/adapter layer, persistence code in
infrastructure — regardless of what its dependencies currently look like.
The current structure is what the user wants to *change*; it gets no vote.

Consequently, **violations after the assignment are the deliverable, not a
failure**: each one marks a place where the code contradicts the target
architecture, with `file:line` evidence. Never adjust an assignment just to
make `validate` pass — that would silently redefine the target back to the
status quo. Moving a component to a different layer afterwards is a design
decision for the user.

## Workflow

### 1. Establish the target layers

```bash
tangleguard-cli [-p <path>] read-config
```

If the config already declares layers (e.g. the user applied a template in
the desktop app), use them. If not, ask the user which architecture they
want and seed the config with one of the canonical templates below (same
names as the desktop app's templates) — or with layers the user describes.

- **Layered** — `Presentation` (UI, views, controllers) → `Application`
  (use cases, domain logic, services) → `Infrastructure` (DB, external
  services). Rules: Presentation→Application, Application→Application,
  Application→Infrastructure.
- **Hexagonal** — `Adapters` (UI, DB, APIs) → `Application` (use cases,
  ports) → `Domain` (pure business logic). Rules: Adapters→Application,
  Adapters→Domain, Application→Application, Application→Domain,
  Domain→Domain.
- **Clean** — `Frameworks & Drivers` → `Interface Adapters` → `Use Cases` →
  `Entities`. Rules: each layer → the next inner one only.

### 2. List the components

```bash
tangleguard-cli -q -l <language> [-p <path>] architecture
```

Packages with `layer: null` (JSON) or no layer (markdown) need assignment.
For a single-package workspace add `--depth 1` and assign the package's
top-level modules instead — layer workspace paths are then two segments,
`[package, module]`.

### 3. Determine each component's role

Names and paths first; that settles most components. When a name is opaque,
gather evidence before guessing:

```bash
tangleguard-cli -q -l <language> context --node <component>
```

and read a few of its files. A component whose role is genuinely unclear
stays **unassigned** — say so instead of guessing.

### 4. Propose — never write without approval

Present the full assignment as a table: component → layer, with a one-line
reason each, plus the unassigned remainder. Wait for the user to confirm or
adjust. Rule and layer changes are always the user's decision.

### 5. Apply

Merge the assignments into the config — take the exact JSON from
`read-config`, add each component's workspace path to its layer's
`workspace_paths` (leave rules, layout, and every other field untouched),
and write it back:

```bash
tangleguard-cli [-p <path>] write-config --config '<full merged JSON>'
```

### 6. Produce the migration backlog

```bash
tangleguard-cli -q -l <language> [-p <path>] validate
```

Report the violations grouped by edge, framed as the gap between the code
and the chosen architecture — expected and actionable, not an error. Each
violation's `file:line` is a concrete refactoring site; work through them
with the untangle skill's cut-weighing approach and preflight every new
import with `check-import`. Suggest reviewing the assignment visually in the
TangleGuard desktop app, where layers and members can be adjusted by hand.
