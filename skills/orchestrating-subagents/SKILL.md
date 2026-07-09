---
name: orchestrating-subagents
description: Orchestrate serial or parallel Codex subagents when the user explicitly authorizes delegation, subagents, multi-agent work, concurrent agent execution, or review fan-out.
---

# Orchestrating Subagents

Use this skill only when the user explicitly authorizes subagents, delegation, multi-agent work, concurrent agent execution, or review fan-out. Do not treat task complexity alone as permission to spawn agents.

## Parallel Gate

- Start with a brief local inspection.
- If delegation is not useful, stay local.
- If delegation is useful, ask one question first: can the work run safely in parallel?
- Treat work as parallel-safe only when there are at least 2 genuinely independent scopes, ownership does not overlap, each scope can be verified independently, and each prompt can be written without another subagent's result.
- If that check passes, use parallel subagents. If it fails, use serial subagents.
- When the user gives an agent count, treat it as the number of spawned task agents. The main agent and any organizer do not count.
- Do not duplicate delegated work locally or edit a worker's owned files while it is running.

## Organizer Gate

- Plan locally by default. Use `agent_type: "agent-organizer"` only if local inspection still leaves one of these unresolved:
- worker ownership is unclear;
- it is unclear whether the work can run in parallel safely;
- the stage graph or dependency order is unclear; or
- there are 3 or more writer agents and the integration order is still unclear.
- If the user specified `n`, tell the organizer to plan for `n` spawned task agents when safe and to explain any smaller lineup.
- Do not use an organizer just because the task is non-trivial, has review fan-out later, or uses more than one agent.

## Spawning

Choose agents by boundary, not by habit:
- Use `agent_type: "explorer"` for independent codebase questions.
- Use `agent_type: "worker"` for implementation slices with clear ownership.
- Use `agent_type: "reviewer"` only after there is a stable artifact to review.
- Use `agent_type: "qa-expert"` for test strategy, acceptance coverage, or release-risk review.
- For `serial-subagents`, wait for each stage to finish before prompting the dependent stage. Common pipelines: `explorer -> worker`, `worker -> reviewer`, `qa-expert -> worker -> qa-expert`.
- Each worker prompt must say the agent is not alone in the codebase, must not revert unrelated edits or edits by other agents, and must include `Ownership`, `Task`, `Constraints`, and `Return format`.

## Waiting

- Codex subagents can be slow. Prefer patience over supervision.
- Do not poll frequently just to check progress.
- Preferred wait: `wait_agent` on all outstanding targets with a `10-20 min` timeout.
- A `wait_agent` timeout is not a failure. It means the agent has not reached a final state yet.
- Do not treat one or more short waits or timeouts as evidence that a subagent is hung.
- Default global wait budget: `60 min`, unless the user specifies a different budget.
- If a wait times out before the global budget, report brief status if useful, continue non-overlapping work, then wait again.
- Do not infer agent failure from one or more timeouts.
- If the global budget is reached, report completed agents, outstanding agents, partial results, and ask whether to continue waiting, close slow agents, or take over.

## Parallel-Only Rules

- For 2 or more concurrent agents, use one `multi_tool_use.parallel` call that wraps the `spawn_agent` calls.
- For 2 or more concurrent writer agents, use `$using-git-worktrees` isolation. This skill owns the `when`; follow `$using-git-worktrees` for setup details.
- Give each concurrent writer one worktree branch and exclusive ownership. If ownership overlap is unavoidable, serialize instead.
- Require commits when feasible; otherwise require an exact changed-file list and a clean diff summary.

## Required Agent Output

Workers must return: `Scope`, `Result`, `Changed files`, `Verification`, and `Risks/Follow-ups`. Worktree-isolated writers must also return `Worktree`, `Branch`, and `Commit(s)` when commits were created.
Explorers and reviewers must return: `Scope`, `Findings`, `Evidence`, and `Risks/Follow-ups`.
`qa-expert` agents must return: `Scope`, `Assessment`, `Evidence`, and `Risks/Follow-ups`.
Reject vague output. If an answer lacks evidence, changed-file scope, or verification status, treat it as incomplete and resolve the gap before relying on it.

## Integration

- Do not finish, summarize, or start review fan-out until every required prior-stage agent reaches a final state, unless the user interrupts or changes the plan.
- Read each result and inspect the changed files or evidence that matters.
- Resolve conflicts and incompatible assumptions before moving to the next stage.
- For concurrent writers, inspect each worker branch before picking accepted commits into the target branch or integration worktree.
- Do not manually recreate large worker changes locally unless the worker could not produce commits and the diff has been inspected.
- Do not trust an agent success report by itself. Inspect the work, then verify the integrated result locally.

## Review Fan-Out

After implementation work is integrated, launch review agents only if the user requested review fan-out or the workflow clearly needs a post-work review stage.
- Use the user-specified review-stage count. If unspecified, default to 3 agents.
- Review-stage agents are read-only unless the user explicitly authorizes them to edit.
- Spawn 2 or more review-stage agents with `multi_tool_use.parallel`.
- Use `agent_type: "reviewer"` for defect-review angles and `agent_type: "qa-expert"` for tests, verification, or rollout angles.
- Give every review-stage agent the same stable artifact or diff, plus a distinct angle.
- Default lineup: `reviewer` for correctness and requirement coverage; `reviewer` for simplicity, maintainability, performance, and concurrency risk; `qa-expert` for tests, verification, rollout, and operational risk.
- Wait with the same protocol. Integrate findings by severity and evidence. Fix real issues, reject false positives explicitly, and re-run verification after changes.

## Final Report

Keep the final report short, but include:
- the integrated outputs or accepted changes;
- writer worktrees or branches used and which commits were picked, when concurrent writers were used;
- key review findings or unresolved risks, when review ran; and
- final verification commands and results.
