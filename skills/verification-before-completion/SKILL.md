---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

For regression tests, verify the test can actually fail: write it → run (pass) → revert the fix → run (must fail) → restore → run (pass). A test that never went red proves nothing.

## Stop Signals

If you catch yourself thinking any of these, stop and run the verification:

| The thought | The reality |
|---|---|
| "Should work now" / "I'm confident" / "seems fine" | Confidence isn't evidence — run it |
| "Just this once" / "I'm tired, skip it" | No exceptions; exhaustion isn't one |
| "Linter passed" | Linter isn't compiler or test run |
| "Agent reported success" | Verify independently against the diff |
| "Partial check is enough" | Partial proves nothing |
| "Different wording, so the rule doesn't apply" | Spirit over letter |
| About to commit/push/PR, or say "Done!" / "Great!" / "Perfect!" | No completion claim without fresh evidence |

## Why This Matters

Claiming completion without fresh evidence is dishonesty dressed up as efficiency, and it breaks trust quickly. The costs are concrete and recurring:

- Undefined functions or missing wiring shipped as "done" — crashes and broken behavior in production.
- Incomplete requirements shipped — features that look finished but aren't.
- False completion forces the user to verify, redirect, and rework, wasting the exact time the shortcut claimed to save.
- Once trust erodes, every later claim is suspect — even when it is backed by evidence.

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
