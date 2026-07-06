# Agent-Skill

A skill for AI Agents which makes use of TangleGuard to develop maintainable codebases.

It teaches the agent to query the codebase's real architecture (layers, packages, dependency graph) instead of guessing it from file paths — and to **preflight every import** with `check-import` before writing it, so dependencies that violate your rules or close a cycle never enter the codebase.

## Requirements

You have to have to TangleGuard CLI installed. The recommended way to install it is via [Homebrew](https://formulae.brew.sh/cask/tangleguard-cli):

```bash
brew install --cask tangleguard-cli
```

See the [documentation](https://tangleguard.com/apps/cli) for alternative installation methods.
