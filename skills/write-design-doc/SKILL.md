---
name: write-design-doc
description: Write or revise self-contained, implementation-ready technical designs. Use when the deliverable is a design document or RFC for a codebase change, or when an existing design must be repaired for feasibility, missing decisions, or whole-system trade-offs.
---

# Write a Design Document

Treat a design document as an evidence-backed argument, not a transcript or a section-filling exercise. It must show why the chosen system change satisfies its goals and invariants, how it works, what it costs, and how it will be validated.

Do not implement the design unless the user separately asks for implementation. Do not use this skill to replace unsettled product requirements with technical assumptions.

## 1. Build the evidence map

Identify the primary readers and the decision the document must enable. Locate the repository's document conventions, domain glossary, context map, ADRs, prior designs, and applicable instructions before choosing a file or structure.

Read the relevant implementation, configuration, tests, callers, consumers, schemas, and operational paths. Reconstruct current control flow, data flow, state ownership, contracts, and failure behavior. Cite concrete files and symbols for claims about existing code; prefer stable symbol references over line-only references.

Maintain four distinct sets while researching:

- **Facts:** verified in code, tests, configuration, metrics, or authoritative documentation.
- **Requirements:** observable outcomes and constraints the design must satisfy.
- **Assumptions:** plausible inputs that still need confirmation.
- **Open decisions:** choices whose outcomes materially change the design.

Surface contradictions rather than choosing the convenient source silently. Resolve discoverable facts from the repository instead of asking the user.

**Gate:** Do not design until every material current-state claim has evidence, the affected system boundary is explicit, and remaining assumptions or open decisions are visible.

## 2. Define the design contract

State the problem independently of the proposed solution. Define measurable goals, explicit non-goals, invariants, compatibility commitments, and whole-system guardrails. Challenge solution language inherited from the request when it is not itself a real constraint.

Walk representative scenarios before choosing structure: normal operation, boundary inputs, partial failure, overload or concurrency where relevant, deployment and rollback, upgrade or migration, recovery, and interaction with adjacent paths. Read [design lenses](references/design-lenses.md) for the concerns actually implicated; do not copy unused lenses into the document.

If the desired user outcome or a blocking product decision is unresolved, ask for that decision before presenting a technical design as complete. Keep optional future work out of the current contract.

**Gate:** The design contract must be sufficient to reject a proposal that is incorrect, incomplete, over-scoped, or harmful to an adjacent path without relying on implementation taste.

## 3. Resolve the design

Derive the simplest coherent end state from the contract. Put each invariant and piece of state at its natural owner; keep interfaces small; avoid duplicate state, parallel implementations, fallback chains, and speculative extension points.

For each material, hard-to-reverse choice, compare the status quo and at least one credible alternative. Record why the selected option best satisfies the contract and where it is weaker. Do not manufacture alternatives for obvious local decisions.

At the point of need, use the existing canonical skills instead of restating their rules:

- Consult [`codebase-design`](../codebase-design/SKILL.md) when module depth, interface shape, or seam placement is material.
- Use [`domain-modeling`](../domain-modeling/SKILL.md) when the work explicitly includes changing domain language or recording an ADR-worthy decision. Otherwise mark the unresolved domain decision as a blocker rather than inventing private terminology in the document.
- Use [`grill-me`](../grill-me/SKILL.md) only when the user explicitly asks for an exhaustive design interview.

Build traceability for every goal and invariant:

```text
goal or invariant -> design mechanism -> owning module or interface -> validation
```

Account for every new state transition, source of truth, cross-system contract, failure mode, migration step, and obsolete path. If feasibility depends on an unproven API, data property, or performance assumption, resolve it with read-only evidence or identify a focused prototype or measurement as required validation; do not manufacture certainty.

If a material trade-off requires accepting a regression outside the contract, obtain user confirmation before treating it as decided.

**Gate:** Do not write an implementation-ready proposal until every goal and invariant traces to a mechanism and validation, every new state and contract has an owner, and no critical trade-off or feasibility question remains implicit.

## 4. Write for implementation and review

Honor the requested destination. If none is given, follow an established repository convention; if none exists, return the document in the response instead of inventing a documentation hierarchy.

Use only the sections the design needs, generally drawn from:

1. Summary and decision
2. Background and current behavior
3. Problem, goals, and non-goals
4. Constraints and invariants
5. Proposed design
6. Interfaces, ownership, and control or data flow
7. Failure handling and operational behavior
8. Compatibility, migration, rollout, and rollback
9. Whole-system effects and trade-offs
10. Validation strategy
11. Alternatives and unresolved questions

Lead with the problem, chosen design, and why. Use the project's domain language. Explain enough current behavior for readers without prior conversation context, while linking to code instead of reproducing it. Use diagrams or tables only when they materially clarify flow, ownership, lifecycle, traceability, or comparison.

Use an objective, documentation-oriented tone. Do not refer to private conversation context or the person who requested the document. Write decisions as decisions. Label assumptions and open questions explicitly; do not blur them with `may`, `might`, or multiple unchosen implementations. Keep rejected alternatives only when their rejection teaches a future reader something or prevents a likely wrong turn.

**Gate:** A fresh implementer must be able to explain what changes, where each responsibility lives, how important scenarios behave, why the design was chosen, and how success is proved without consulting the private conversation.

## 5. Run the contradiction pass

Re-read the document against the current code and the design contract. Check that:

- goals, non-goals, mechanisms, and validation agree;
- interfaces and ownership do not duplicate or leak responsibilities;
- common, boundary, failure, upgrade, rollback, and recovery scenarios are internally consistent where relevant;
- security, data integrity, compatibility, performance, capacity, observability, and operability effects are addressed when implicated;
- rollout steps preserve a valid system state and identify removal of superseded code, configuration, and tests;
- implementation does not depend on an unnamed decision, fictional API, or unsupported property of the current system.

Classify the result honestly:

- **Implementation-ready:** all critical decisions are resolved, feasibility is grounded in the codebase, and traceability is complete.
- **Draft:** one or more critical decisions, facts, approvals, prototypes, or measurements remain open and are named explicitly.

Do not label a document implementation-ready merely because all headings contain text.
