# Agent Profile

This repository tracks the portable configuration shared between coding agents:

- `AGENTS.md`: global working instructions.
- `skills/`: agent skills.

The profile installs to `~/.agents`.
`AGENTS.md` is additionally installed into the Codex home when one exists.

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
- Speculative compatibility machinery: Codex can add generations, fences,
  version fields, or dual read/write paths even when no consumer can be left
  behind by the change. This profile treats such mechanisms as justified only
  by present constraints — released artifacts, persisted data, external users,
  rolling deployments — and changes the contract directly when every consumer
  moves atomically with it.
- Imitating neighboring code without checking its rationale: Codex often
  copies a mechanism or pattern from a nearby module without verifying that
  the constraint that justified it exists in the new case, or was sound at
  all — so a copied mistake spreads. This profile treats existing code as
  evidence, not endorsement.
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
- Uniform diligence regardless of stakes: Codex tends to give a trivial change
  the same verification depth, test volume, defensive checks, and process (such
  as TDD) as a risky one, producing ceremony instead of confidence. This
  profile scales effort with the risk and subtlety of the contract and
  measures diligence by the fitness of the result, not the volume of
  artifacts produced.
- Review that only checks correctness bugs: the bundled `code-review` skill was
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

Install or update `${AGENTS_HOME:-$HOME/.agents}` with one command:

```bash
curl -fsSL https://raw.githubusercontent.com/Lloyd-Pottiger/agent-profile/main/install.sh | sh
```

The installer adds missing files and updates existing profile files in place.
Files that exist only in the destination are kept. When a Codex home exists
(`${CODEX_HOME:-$HOME/.codex}`), `AGENTS.md` and the agents are also installed
there — the agents converted to TOML and registered in its `config.toml`.
Skills need no Codex copy: Codex reads `~/.agents/skills` directly.

From a local checkout, run:

```bash
./install.sh
```

To install into a custom location:

```bash
AGENTS_HOME=/path/to/.agents ./install.sh
```

## Repository Layout

```text
.
├── AGENTS.md
├── install.sh
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

## License

This repository is MIT licensed; see `LICENSE`.
