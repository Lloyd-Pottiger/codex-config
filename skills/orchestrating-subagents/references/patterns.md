# Artifact DAG Patterns

Read only the pattern matching the current task. Adapt node count and review angles to the actual boundaries and risks; do not fill capacity for its own sake.

## Large Commit Review

Establish one stable review artifact: the exact base/head commit range plus requirements or design docs. Fan out read-only nodes by distinct risk surface, for example:

```text
scope commit range
    |-- correctness and requirements
    |-- performance, concurrency, and failure modes
    |-- architecture and simplicity
    `-- tests, rollout, and operational risk
                 |
          main-agent triage
                 |
      optional holistic re-review
```

Give all reviewers the same artifact and different questions. Let the main agent deduplicate findings, verify evidence, and reject false positives. Choose angles from the diff; do not use a fixed reviewer count. If fixes change the range, create a new review artifact and re-review affected areas.

## Parallel Writers

Stabilize the design, interface contracts, and base commit before creating worktrees:

```text
accepted design + base SHA
       |-- writer A worktree --|
       |-- writer B worktree --|--> inspect commits --> integrate --> verify
       `-- writer C worktree --|
```

Split by independently testable responsibility. Distinct files are insufficient when writers change opposite sides of the same interface or share an unstated invariant. For each writer, record base SHA, branch, absolute worktree, responsibility, writable scope, read-only dependencies, preserved interfaces, verification, and expected commit.

Inspect and accept each branch before integration. Integrate in declared dependency order. Run integrated tests even when every branch passed independently.

## Mixed Read/Write Pipeline

Use explicit gates between phases:

```text
parallel exploration
         |
    design synthesis
         |
parallel worktree writers
         |
 per-slice review and fixes
         |
      integration
         |
 holistic review and fixes
         |
 terminal verification
```

Exploration outputs are evidence, not automatically design decisions. The design node or main agent must resolve conflicts and publish one accepted design artifact before writers start. Per-slice review checks each implementation contract; holistic review checks cross-slice behavior and integration risks. Modifications invalidate only the descendants whose inputs changed.

## General Dependency DAG

For arbitrary dependent subtasks, keep a small node ledger and schedule waves rather than forcing the entire task into serial or parallel mode:

```text
repeat until terminal artifact accepted:
    validate completed outputs
    version changed artifacts and invalidate stale descendants
    compute nodes whose dependencies and inputs are ready
    dispatch independent ready nodes up to capacity
    perform only non-conflicting local work
    wait for the next completion
```

Prefer critical-path nodes when capacity is constrained. A failed or blocked node blocks only its descendants; continue independent branches when useful. Replan the DAG when new evidence changes task boundaries, but preserve accepted artifacts that remain valid.
