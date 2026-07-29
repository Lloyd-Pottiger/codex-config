# Codex Profile

This repository is laid out like a Codex home directory. It tracks only the
portable configuration that should be shared:

- `AGENTS.md`: global working instructions.
- `agents/`: sub-agent definitions.
- `skills/`: Codex skills.

Local runtime state, credentials, logs, history, SQLite databases, sessions, and
machine-specific config are intentionally ignored.

## What Problems This Profile Tries To Solve

This profile is tuned for production-facing coding work, especially infrastructure
and backend changes where "mostly works" is not good enough.

It is opinionated about a few recurring Codex failure modes:

- Patch-first changes: Codex often preserves the current shape and translates
  each new requirement or correction into one more branch, wrapper, flag,
  fallback, rule, or exception. This profile treats non-trivial feedback as
  evidence about the desired outcome, re-derives the simplest coherent
  end-state, and only then minimizes diff size.
- Unbounded refactors in the wrong direction: Codex can either avoid needed
  restructuring and keep stacking patches, or use a small task as an excuse for
  a broad cleanup. This profile tries to hold a middle line: allow refactoring
  within the affected design seam when the local structure is wrong, but avoid
  unrelated cleanup outside that seam.
- Half-removed old approaches: when an implementation starts with approach A and
  later switches to approach B, Codex often leaves dead helpers, comments,
  branches, imports, or state behind. This profile tells workers to remove the
  superseded local implementation in the same change.
- Tests that follow implementation history instead of current behavior: Codex
  tends to keep tests that only proved a temporary reuse path, old wiring, or an
  intermediate TDD step. This profile treats tests as protection for the current
  contract and asks agents to delete or rewrite stale tests.
- Review that only checks correctness bugs: the bundled `codex-review` skill was
  tightened to also look for material implementation concerns such as
  over-abstraction, duplicate state, unnecessary API surface, and avoidable
  common-case cost.
- Actions that trust conversation memory over repository state: Codex may write
  commit messages, summaries, reviews, or cleanup decisions that describe only
  the changes from the current session rather than the full observable diff.
  This profile tells agents to rediscover scope from repository state before any
  action that summarizes, commits, reviews, validates, or cleans up changes.
- Comments and docs that record patch history: this profile prefers comments
  that explain invariants, rationale, interfaces, and non-obvious behavior,
  rather than narrating what was tried or intentionally not done.
- Hot-path drift in infra code: the global instructions require performance
  effects to be considered during implementation and review. The bundled
  `performance-engineering` skill provides deeper causal analysis and
  validation for deliberate optimizations and material performance claims.

In short, this profile is trying to make Codex behave less like a patching tool
and more like a careful engineer optimizing for the final code shape.

## Install or Update

Install or update `${CODEX_HOME:-$HOME/.codex}` with one command:

```bash
curl -fsSL https://raw.githubusercontent.com/Lloyd-Pottiger/codex-profile/main/install.sh | sh
```

The installer adds missing files and updates existing profile files in place.
Files that exist only in the destination are kept.
It also registers installed `agents/*.toml` files in `config.toml`; existing
agent registration sections are preserved.

From a local checkout, run:

```bash
./install.sh
```

To install into a custom Codex home:

```bash
CODEX_HOME=/path/to/.codex ./install.sh
```

## Repository Layout

```text
.
├── AGENTS.md
├── install.sh
├── agents/
├── skills/
├── LICENSE
└── README.md
```

## Sources

Some skills are adapted from:

- https://github.com/obra/superpowers
- https://github.com/mattpocock/skills

Recommended companion projects:

- CodexPotter: https://github.com/breezewish/CodexPotter/tree/v2

Agents are adapted from:

- https://github.com/VoltAgent/awesome-codex-subagents

## License

This repository is MIT licensed; see `LICENSE`.
