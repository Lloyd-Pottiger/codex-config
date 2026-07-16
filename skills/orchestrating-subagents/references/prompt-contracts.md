# Subagent Prompt Contracts

Use the universal contract for every node, then append the role-specific return contract. Keep prompts self-contained and task-local.

## Universal Contract

```text
Role
  The responsibility and read/write mode for this node.

Objective
  One concrete result this node must produce.

Inputs
  Exact artifact paths, commit SHAs/ranges, accepted design revision, and
  assumptions supplied by completed dependencies.

Ownership
  Responsibility boundary. For writers: absolute worktree, branch, writable
  scope, read-only dependencies, and interfaces/invariants to preserve.

Constraints
  Applicable repository instructions, user constraints, forbidden actions,
  and coordination rules. State that other work may exist and unrelated edits
  must not be reverted.

Completion criterion
  A checkable condition for accepting the node, including required evidence or
  verification.

Return format
  The role-specific fields below.
```

Do not tell an agent to “explore broadly,” “implement the feature,” or “review everything.” If its result cannot be accepted or rejected against the prompt alone, narrow the node or sharpen the completion criterion.

## Explorer

Return:

- `Scope inspected`
- `Direct answer`
- `Evidence` with paths and line references when possible
- `Unknowns or risks`

Stop when the named question is answered. Do not edit or drift into general review.

## Designer or Synthesizer

Return:

- `Inputs reconciled`
- `Design or decision`
- `Interfaces and invariants`
- `Trade-offs`
- `Unresolved blockers`

Resolve conflicting exploration evidence explicitly. Produce one versioned design artifact that downstream prompts can reference; do not leave critical choices implicit.

## Writer

Return:

- `Scope implemented`
- `Result`
- `Changed files`
- `Verification` with commands and results
- `Risks or follow-ups`
- `Worktree`, `branch`, and `commit SHA`

Require the writer to stay in the assigned worktree, commit the completed slice when feasible, and report blockers rather than expanding ownership. A changed-file list without repository evidence is not an integration artifact.

## Reviewer

Return:

- `Artifact reviewed`
- `Review angle`
- `Findings` ordered by severity
- `Evidence` for each finding
- `Smallest fix or mitigation`
- `Residual risk`

Keep reviewers read-only unless the DAG explicitly gives a later fix node write ownership. Give parallel reviewers distinct angles and the same exact artifact version.

## QA Specialist

Return:

- `Artifact and behavior boundary`
- `Critical risks and acceptance criteria`
- `Validation performed or proposed`
- `Evidence and gaps`
- `Release gate and residual risk`

Map each critical risk to a validation path. Do not turn a low-risk bounded change into an exhaustive release plan without evidence that it needs one.
