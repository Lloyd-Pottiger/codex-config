---
name: code-review
description: Review a stable branch, commit, PR, or working-tree change with an independent reviewer subagent in a fresh context, against both repository standards and the originating spec. Use when explicitly invoked, when asked for a deep or second-pass review, or as the final review phase after substantial implementation.
---

# Code Review

Dispatch a dedicated reviewer subagent and wait for its result. The fresh reviewer owns evidence discovery and review; the main agent verifies and triages its result only after the reviewer returns. The delegation exists for the fresh context: the reviewer sees only the review brief, not the conversation that produced the code.

## Fresh Context Rule

- Dispatch exactly one reviewer per review scope and let it finish before triage.
- Do not run a separate manual review while the reviewer is still running.
- Do not inspect the diff in parallel as a substitute for waiting.
- Do not produce findings, a final answer, or a review summary until the reviewer returns.
- While waiting, send brief progress updates only when the user asks for status.

## Fit

Use this skill when:

- The user explicitly invokes this skill or asks for a deep, independent review.
- The change is non-trivial, high-risk, production-critical, or performance-sensitive.
- A substantial implementation is complete and needs a final independent review pass.

Avoid this skill when:

- The request is a small or simple review that can be handled by direct inspection.
- Code is still changing rapidly and the review result would immediately go stale.
- The user needs an interactive design discussion rather than a finished-code review.

## Workflow

1. Pin the review artifact.

Derive the exact scope from the user request and repository state. Use the narrowest prompt that identifies one stable artifact:

```text
Review current code changes (staged, unstaged, and untracked files)
Review code changes against the base branch <branch>
Review code changes introduced by commit <sha> (<title>)
Review code changes for commit range <sha1>..<sha2>
Review <specific path/module/feature> for <specific risk or behavior>
```

Resolve every named ref before dispatching. For a branch or tag, review against its merge-base with `HEAD`; for an explicit commit range, preserve the requested endpoints. For current changes, include staged, unstaged, and untracked files. Stop before dispatching if the ref is invalid, the selected diff is empty, or the scope remains ambiguous after inspecting the repository.

2. Pass known evidence sources.

Include any user-supplied issue, PRD, spec, acceptance criteria, or standards paths in the review request. Do not invent a spec when none is known; the reviewer will discover repository-backed sources and report when no spec is available.

3. Dispatch the reviewer subagent.

Read `references/review-brief.md` in this skill's directory. Dispatch the review to a subagent — the host's reviewer-type agent when one is available, otherwise a general-purpose subagent. The task prompt is the full brief followed by a `## Review request` section holding the pinned review prompt and any known evidence sources. Give the reviewer nothing beyond that prompt: the fresh context is the point.

4. Wait for the reviewer to return.

A foreground dispatch blocks until the reviewer finishes. Do only non-conflicting work while waiting, and do not treat a long review as stuck; a production-critical review of a large change can take many minutes.

5. Triage the result before answering the user.

- Verify each reported finding and implementation concern against the diff and surrounding code.
- Drop clear false positives and speculative redesign suggestions instead of forwarding them blindly.
- Preserve the reviewer's priority labels for findings when they are defensible; keep implementation concerns separate unless the evidence clearly supports upgrading one into a finding.
- Preserve explicit Spec and Standards coverage, but do not let either axis mask correctness, security, performance, compatibility, or operational findings.
- If the review reports no actionable findings, say that directly and include any material implementation concerns or residual validation gaps it identified.

## Review Standard

The delegated reviewer is instructed to perform a production-critical review:

- Verify the pinned artifact and determine the exact change set before judging it.
- Discover the originating spec from the user request, commit or PR references, repository specs or PRDs, tests, and related docs. If no authoritative spec exists, say so rather than inferring requirements.
- Discover repository standards from applicable `AGENTS.md`, contribution guides, coding standards, and engineering docs.
- Review the **Spec** axis for missing or partial requirements, unrequested scope, and implementations that contradict the cited requirement.
- Review the **Standards** axis for documented-rule violations. Treat general code smells only as judgment calls; repository standards override them, and tooling-enforced style is not a review finding.
- Explain or internally account for non-obvious control flow, data flow, state changes, concurrency, and failure modes.
- Evaluate negative impacts across correctness, security, robustness, compatibility, CPU, memory, IO/RPC behavior, logging cost, and maintainability.
- Evaluate implementation shape: whether abstractions pay for themselves, whether a simpler local design exists, and whether the diff adds avoidable common-case cost. Prefer `delete > reuse > merge > abstract` when judging alternatives.
- Report only discrete, actionable issues or material implementation concerns that the author would likely fix.
- Prefer no findings over speculative findings.

## Implementation Concerns

- Keep implementation concerns separate from priority-labeled findings unless they rise to a concrete correctness, performance, security, or operational problem.
- Report them only when the change introduced or materially worsened unnecessary abstraction, duplication, state, API surface, or hot-path overhead, and a simpler or cheaper local alternative is clear.
- Do not forward subjective rewrites or "maybe cleaner" redesign ideas.

## Priority Labels

- `[P0]`: Drop everything to fix. Blocking release, operations, or major usage.
- `[P1]`: Urgent. Should be addressed in the next cycle.
- `[P2]`: Normal. Should be fixed eventually.
- `[P3]`: Low. Nice to have.

## Constraints

- Run this at most once for a given review scope unless the code changed substantially after the run.
- Do not use this as a lint replacement.
- Do not ask the delegated reviewer to edit files.
- If the dispatch fails or the reviewer returns without completing the review, report the failure and the visible output; do not fabricate review results.
