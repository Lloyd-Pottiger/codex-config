---
name: codex-review
description: Review a stable branch, commit, PR, or working-tree change with an independent Codex CLI against both repository standards and the originating spec. Use when explicitly invoked as $codex-review, when asked for a deep or second-pass review, or as the final review phase after substantial implementation; after starting the script, wait for it to exit before triage or a final response.
---

# Codex Review

Launch a separate Codex reviewer through `scripts/review.js` and wait for it to complete. The fresh reviewer owns evidence discovery and review; the main agent verifies and triages its result only after the script exits.

## Synchronous Execution Rule

`scripts/review.js` is a foreground process: it runs to completion and then exits. After starting it, the active task is waiting for that exit.

- Do not run a separate manual review while the script is still running.
- Do not inspect the diff in parallel as a substitute for waiting.
- Do not produce findings, a final answer, or a review summary until the script exits.
- If your harness exposes the running command as a pollable task, check it roughly every 5 minutes for new progress output — but treat each check as a status peek, not permission to start other review work.
- While waiting, send only brief progress updates to the user when new output appears or the user asks for status.
- If the user asks for status, report whether the script is still running, then keep waiting unless the user explicitly tells you to stop.

## Fit

Use this skill when:

- The user explicitly invokes `$codex-review` or asks for a deep review.
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

Resolve every named ref before starting. For a branch or tag, review against its merge-base with `HEAD`; for an explicit commit range, preserve the requested endpoints. For current changes, include staged, unstaged, and untracked files. Stop before launching the reviewer if the ref is invalid, the selected diff is empty, or the scope remains ambiguous after inspecting the repository.

2. Pass known evidence sources.

Include any user-supplied issue, PRD, spec, acceptance criteria, or standards paths in the review prompt. Do not invent a spec when none is known; the independent reviewer will discover repository-backed sources and report when no spec is available.

3. Run the review script in the foreground.

```shell
node <skill-directory>/scripts/review.js --cwd "<project directory>" "<review prompt>"
```

`scripts/review.js` lives inside this skill directory, not inside the project being reviewed. The review may take a long time; let it run until it exits. Do not append `&`, do not detach it, and do not move on to another review path while it is active.

4. Monitor progress.

The script buffers progress and emits the final Markdown when it exits. If the harness hands back a pollable handle for the running command, peek at it about every 5 minutes for new buffered output; otherwise simply wait for the process to exit. If a check returns no new output, keep waiting quietly. Do not treat the review as stuck until there has been no progress for more than 30 minutes.

5. After the script exits, triage the result before answering the user.

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
- Do not call the script "background" work; it is the foreground review task until it exits.
- If the script exits unsuccessfully, report the failure and the visible output; do not fabricate review results.
