---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. While grilling, sharpen domain terminology, update CONTEXT.md inline as terms crystallise, and offer ADRs sparingly when a real trade-off is locked in. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions **one at a time**, waiting for feedback on each question before continuing. If a question can be answered by exploring the codebase, explore the codebase instead of asking.

Don't ask silly questions — make sure you fully understand the context first:

- Check the current project state: files, docs, recent commits.
- Explore the current structure and codebase to understand the context.
- Read existing domain documentation: `CONTEXT.md`, `CONTEXT-MAP.md`, and any ADRs under `docs/adr/`. The grilling works against the language and decisions already recorded there.

## Domain awareness

Before grilling, read the project's existing domain documentation: `CONTEXT.md` (or `CONTEXT-MAP.md` in a multi-context repo) and any ADRs under `docs/adr/`. The grilling works against the language and decisions already recorded there. The context/ADR file layout and formats live in the [`domain-modeling`](../domain-modeling) skill — see [CONTEXT-FORMAT.md](../domain-modeling/CONTEXT-FORMAT.md) and [ADR-FORMAT.md](../domain-modeling/ADR-FORMAT.md). Create files lazily, only when you have something to write.

## During the session

Run the [`domain-modeling`](../domain-modeling) discipline throughout the interview: challenge terms that conflict with the existing glossary, sharpen fuzzy language into precise canonical terms, stress-test relationships with concrete edge-case scenarios, and cross-reference the code when the user states how something works. Update `CONTEXT.md` inline the moment a term resolves, and offer an ADR sparingly — only when a decision is hard to reverse, surprising without context, and the result of a real trade-off. The full rules for each of those moves live in the `domain-modeling` skill.

Keep the grilling cadence from the top of this skill: one question at a time, and capture `CONTEXT.md` terms inline as they resolve — never batched.
