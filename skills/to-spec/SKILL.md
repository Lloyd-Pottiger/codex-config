---
name: to-spec
description: Synthesize the current conversation and codebase into a concise product or feature specification, then publish it to an explicitly selected project tracker.
---

# To Spec

Turn the current conversation and verified codebase context into a spec (also called a PRD). Synthesize settled information rather than restarting discovery. Ask only when an unresolved requirement would materially change the spec or the publication destination cannot be determined safely.

Resolve the destination from the user's request, repository documentation or configuration, and available tracker integrations. Publish only when one destination is explicit or unambiguous. Otherwise return the draft in the response and ask where it should be published; do not invent a tracker, local directory, or label vocabulary.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Identify the observable seams at which the feature can be accepted and tested. Prefer an existing public seam that proves the behavior without coupling tests to implementation details. Add a new seam only when the contract cannot be verified cleanly through the existing design.

3. Write the spec using the template below. Publish it to the resolved destination. Apply an existing `ready-for-agent` label only when that is the project's established convention; do not create repository workflow policy as a side effect of publishing a spec.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A complete, non-redundant numbered list of user stories. Each user story should be in the format:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

Cover the distinct actors, outcomes, boundary conditions, and failure behavior needed to define the contract. Do not multiply stories that express the same behavior.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- The observable acceptance seams and critical failure cases
- Which contracts need unit, integration, or end-to-end coverage
- Relevant testing patterns already present in the codebase

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Assumptions, unresolved questions, or evidence that materially affects implementation.

</spec-template>
