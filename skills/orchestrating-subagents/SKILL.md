---
name: orchestrating-subagents
description: Orchestrate subagents as an artifact DAG. Use only when the user explicitly authorizes delegation for read-only exploration or review fan-out, isolated parallel implementation, mixed multi-stage work, or dependency-ordered subtasks.
---

# Orchestrating Subagents

Treat orchestration as an **artifact DAG**: subagents execute bounded nodes; accepted outputs become versioned artifacts; dependency edges decide which nodes are ready. The main agent owns the objective, DAG, cross-node decisions, integration, and final verification.

## 1. Confirm Delegation Value

Delegate only with explicit user authorization and at least one concrete benefit:

- **context isolation**: local detail can stay out of the main agent's context;
- **parallel latency**: independent nodes can reduce critical-path time; or
- **independent perspective**: fresh context can improve exploration, design, or review.

Do not spawn agents merely because a task is large. Keep a node local when prompting, supervising, and integrating it would cost more than doing it locally. Treat a user-specified agent count as a capacity limit unless the user explicitly requires that many agents.

**Completion criterion:** every delegated node has a named benefit; otherwise keep it local.

## 2. Build the Artifact DAG

Inspect enough repository state to identify the real boundaries before dispatching. Define each node with:

- `ID` and `role`;
- `mode`: `read` or `write`;
- `depends_on`;
- exact input artifacts and versions, such as paths, design revision, or base/head commits;
- responsibility and, for writers, exclusive ownership;
- a checkable completion criterion; and
- expected output artifacts.

Use responsibility boundaries, not file lists alone. Separate writers only when their interface assumptions and verification can remain independent. Add an edge when one node needs another's result; dependencies make nodes serial, not the whole DAG.

Keep requirement interpretation, cross-node trade-offs, conflict resolution, integration acceptance, and the final user response with the main agent. Use a dedicated organizer only when a brief local inspection cannot resolve the DAG, writer ownership, or integration order.

For established DAG shapes, read [references/patterns.md](references/patterns.md) only for the matching branch.

**Completion criterion:** every required output is reachable from the DAG, every dependency is explicit, and every concurrent writer has independent responsibility.

## 3. Prepare Stable Inputs and Context

Dispatch a node only when all dependencies are accepted and every input artifact is stable at the version named in its prompt. Give each subagent the smallest sufficient context:

- subagents start with a fresh context and see only the task prompt — pass user constraints and task-local facts explicitly;
- point to raw code, commits, tests, docs, or prior artifacts instead of retelling exploration history; and
- resume the same subagent instance for follow-ups that genuinely depend on its prior context instead of spawning a duplicate.

Write prompts from [references/prompt-contracts.md](references/prompt-contracts.md). When the host provides specialized subagent types, use them for focused reading, bounded implementation, defect review, or test and rollout risk as appropriate. Otherwise encode the role directly in the prompt.

For two or more concurrent writers, invoke the `using-git-worktrees` skill and prepare one branch/worktree per writer from the same accepted base commit. Give every writer its absolute worktree path and branch; require all work to occur there. Never let concurrent agents write to the same checkout or branch.

**Completion criterion:** every ready node has stable, sufficient inputs; every writer has an isolated workspace and explicit responsibility.

## 4. Dispatch the Ready Set

Compute the **ready set**: pending nodes whose dependencies are accepted and whose inputs and workspaces are ready. Dispatch independent ready nodes without waiting between launches, up to the available capacity. Prioritize critical-path nodes, then use spare capacity for sidecar exploration or review.

Do not duplicate a running node locally or edit a writer's owned responsibility. Concurrent read-only nodes may inspect the same artifact when their questions or review angles are distinct. Concurrent writers require both worktree isolation and independent logical ownership; worktree isolation alone does not make coupled changes safe.

**Completion criterion:** every dispatched node was ready at launch, and no concurrent node pair has conflicting write or decision ownership.

## 5. Accept Outputs, Then Advance

Wait for actual completion instead of polling frequently. A wait timeout means only that work is still running. While waiting, do only work that cannot conflict with running nodes.

Validate each returned artifact against its node contract and repository state. Do not accept a success claim without inspecting the relevant evidence, diff, commit, or verification result. If an output is incomplete, use a follow-up with the same agent to close the specific gap when possible; avoid spawning duplicate work.

Mark a node `accepted` only after validation. Then recompute the ready set and repeat dispatch. If a prerequisite changes, version the new artifact and mark running or completed descendants that consumed the old version as `stale`; stop, redo, or re-review them as appropriate.

**Completion criterion:** every accepted node satisfies its contract, and no downstream result relies on a superseded artifact.

## 6. Integrate Through Gates

Integrate accepted writer commits in dependency order. Before accepting each commit, inspect its exact diff and verification. After integration, resolve conflicts and incompatible assumptions in the integrated checkout; do not push that responsibility back to workers whose local slices were individually correct.

Run slice review against the slice artifact and holistic review against the integrated artifact. Any modification after review creates a new artifact version: invalidate affected findings and rerun the review or verification whose evidence no longer applies.

Verify the terminal integrated artifact locally. Subagent checks and per-branch tests do not replace integrated verification.

**Completion criterion:** all required nodes are accepted, accepted commits are integrated, findings are fixed or explicitly resolved, affected checks were rerun after changes, and terminal verification is known.

## Final Report

Report the integrated result, accepted writer commits and branches when used, material review findings or residual risks, and final verification commands and results. Derive this scope from the final repository state and DAG ledger, not conversation memory.
