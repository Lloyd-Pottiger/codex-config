---
name: resolving-merge-conflicts
description: Resolve conflicts during an active Git merge, rebase, cherry-pick, or revert without losing either side's intent or unrelated work. Use when one of those operations is in progress with unmerged paths, or the user asks to resolve and finish that active operation; do not use for stash-apply conflicts or to initiate an integration operation.
---

# Resolving Merge Conflicts

Resolve conflicts from evidence, not markers alone. Preserve compatible intent from both sides, choose between incompatible intent only from the operation's goal and authoritative requirements, and leave unrelated work untouched.

## 1. Pin the Conflict State

Read the applicable repository instructions, then inspect:

```shell
git status
git diff --name-only --diff-filter=U
git ls-files -u
```

Identify whether Git is merging, rebasing, cherry-picking, or reverting from repository state. Record every unmerged path, conflict type, current operation commit when applicable, and staged or unstaged paths outside the conflict set. Do not start another merge-like operation on top of the current one.

Never abort, reset, restore, or discard a side unless the user explicitly requests that destructive action. If unrelated staged work may be absorbed by the final commit and its ownership cannot be established, pause before continuing the operation.

**Completion criterion:** the active operation, complete unmerged path set, conflict types, and unrelated work are known.

## 2. Reconstruct Both Intents

For each path, inspect the available index stages and the surrounding file:

```shell
git show :1:<path>  # merge base, when present
git show :2:<path>  # stage 2
git show :3:<path>  # stage 3
```

Stages may be absent for add/delete and rename conflicts. During a rebase, avoid reasoning from the labels “ours” and “theirs”; their usual conversational meaning is easy to reverse. Compare the stage contents, the commit being replayed, the target history, and the merge base directly.

Read the commits, blame/history, tests, docs, issue/spec references, and callers needed to explain why each side changed. Classify content, add/add, modify/delete, rename, directory/file, submodule, generated-file, and lockfile conflicts before editing. Resolve generated artifacts from their sources and regenerate them when the repository provides a deterministic command.

**Completion criterion:** both intents and the required final behavior are evidenced for every conflict; any material ambiguity is explicit.

## 3. Resolve Coherently

Edit the whole affected seam, not only the marked lines. Preserve both changes when they are compatible. When they conflict, follow the operation's stated goal and authoritative spec; do not invent a third behavior to avoid choosing. Update nearby tests or callers when the coherent merged behavior requires it.

Do not blanket-apply `--ours` or `--theirs`. Use whole-side selection only when evidence shows the other side is entirely superseded for that path. Resolve delete/rename and mode changes intentionally with path-specific Git commands.

If incompatible business intent remains after inspection, resolve other independent paths but leave the ambiguous path unmerged and ask the user for the missing decision.

**Completion criterion:** every edited path represents one coherent behavior, with no intent decided from marker position or side labels alone.

## 4. Stage and Verify Each Resolution

Stage only paths whose resolutions were inspected:

```shell
git add -- <resolved-paths>
git rm -- <intentionally-deleted-paths>
```

Never use `git add .`, `git add -A`, or a blanket commit command that can capture unrelated work. After staging, require all of the following:

```shell
git diff --name-only --diff-filter=U
git diff --cached --check
```

Search resolved text files for conflict-marker lines, inspect the staged diff against the operation's expected result, and run the smallest relevant tests for the merged behavior. Then run the repository's required typecheck, test, format, generation, or validation commands in its documented order. Fix only failures caused by the resolution; report pre-existing or unrelated failures separately.

Some checks mutate files. After every formatter, generator, fixer, or manual correction:

1. inspect staged and unstaged diffs;
2. path-spec stage only intended resolution changes;
3. re-inspect the complete cached diff; and
4. rerun every check invalidated by the new content.

Repeat until no intended resolution change remains unstaged. Do not continue with a check failure caused by the resolution. Document pre-existing or unrelated failures separately, and continue only when they do not invalidate the resolved behavior or a repository-required gate.

**Completion criterion:** no unmerged index entries or intended unstaged resolution changes remain, the complete staged diff contains only the intended operation and resolution, no conflict markers remain in resolved text, and all resolution-affected checks and repository-required gates pass.

## 5. Finish the Existing Operation

Unless the user limited the request to preparing resolutions, finish the operation with its normal continuation command. For a rebase or multi-commit cherry-pick/revert, a continuation may expose another conflict; return to step 1 and repeat until the sequencer finishes.

Do not rewrite commit messages unnecessarily, bypass hooks, force-push, or begin a different integration operation. If continuation fails, inspect the new repository state before acting. When blocked, preserve the valid resolutions and report the exact blocker and continuation command rather than aborting.

Finally inspect `git status`, the resulting history, and the final diff or commit. Re-run checks invalidated by later conflict rounds.

**Completion criterion:** the requested Git operation is finished or deliberately left at one documented user decision, unrelated work remains intact, and final verification results are reported.
