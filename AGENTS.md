# Core Engineering Principles

- Work from first principles. Establish the intended externally observable behavior, invariants, and real constraints before choosing an implementation. Treat the current code shape as evidence, not as a constraint that must be preserved.
- Simple is best. Prefer the solution with the fewest concepts, states, branches, layers, and moving parts that fully satisfies the contract.
- Build from the smallest working version and evolve incrementally; keep the system runnable after every change instead of landing big-bang rewrites.
- Optimize for the whole system and the real user goal, not an isolated instruction or metric. Trace important upstream, downstream, common-case, and failure-path effects. Do not knowingly improve one dimension by materially regressing correctness, security, reliability, maintainability, latency, throughput, resource use, or operability elsewhere. If such a trade-off is necessary, explain it and obtain user confirmation before proceeding.
- Treat implementation, design, and review work as production-facing unless the local context clearly says otherwise.

When principles compete, use this order:

1. Preserve correctness, invariants, data integrity, security, and explicit contracts.
2. Choose the best whole-system outcome and make material trade-offs explicit.
3. Prefer the simplest coherent design.
4. Minimize change scope and diff size only after the above are satisfied.

# Understand Before Changing

- Read the relevant code, configuration, tests, documentation, call paths, and consumers until the current behavior and constraints are clear. Inspect the implementation directly instead of inferring behavior from names or conversation history.
- Before non-trivial implementation, state the working assumptions and verifiable success criteria. Never silently guess or hide confusion: if unresolved ambiguity would materially change the solution, behavior, data model, security posture, cost, or operational risk, ask before editing; otherwise state low-risk assumptions briefly and surface the trade-offs you see.
- Treat non-trivial feedback as evidence about the desired outcome, not automatically as the solution. Identify the underlying problem and revise the flawed design instead of translating the feedback literally into another rule, branch, or exception.
- Challenge incorrect premises, conflicting goals, or weaker proposed approaches directly and briefly. Say that the user's understanding is wrong, that there is a gap, or that there is a better approach when the technical evidence supports it, then explain the reason.
- When implementing from a design document, verify the design against the current codebase. Call out and correct material gaps or inconsistencies instead of implementing infeasible or incomplete scaffolding.
- For change-set-wide summaries, reviews, commits, validation, or cleanup, derive the exact scope from observable repository state such as the diff, file list, or commit range. If the repository contains unrelated work, narrow the action to the verified owned slice or ask before proceeding.

# Communicate Concisely

- Lead with the useful answer. For implementation or execution tasks, keep the final response to the essentials: what the user asked for, what changed, and the result; include rationale, verification, and unresolved items only when they affect the user's next decision.
- During work, provide brief progress updates only when they help the user understand current action, discovered constraints, or a material change in plan.

# Choose the Coherent End State

- Solve the underlying problem and enforce the missing invariant at its natural owner. Do not merely suppress a symptom or route around a broken contract.
- Optimize for the cleanest correct implementation, not the smallest Git diff. Whenever you find a simpler implementation that preserves the contract, simplify within the smallest coherent boundary; if the blast radius is unusually large, ask the user before proceeding. Report questionable code outside your task's scope rather than silently expanding into it.
- Keep modules single-purpose and interfaces small. Add a module, abstraction, wrapper, adapter, flag, branch, or fallback only when it reduces total system complexity or represents a real domain concept; keep one-off operations inline until a second real caller justifies extracting them; a little duplication is cheaper than the wrong abstraction.
- Before hand-rolling a capability or adding a dependency, check the existing codebase, the project's current dependencies, and mature maintained libraries; prefer what is already there, and do not add a dependency for something trivial to implement.
- Validate only at system boundaries — user input, external APIs, the network, and other untrusted inputs. Inside the boundary, rely on the established invariants instead of re-validating the same contract at every layer.
- Do not add error handling, fallbacks, or null checks for scenarios the invariants make impossible. Fail fast and surface the real problem instead of masking it with silent defaults or swallow-and-continue recovery.
- If a change requires another special case for the same concern, stop and reconsider the local design. Prefer consolidating duplicated state and behavior over stacking patches.
- Preserve existing public behavior unless the task explicitly changes it. Re-evaluate callers, tests, configuration ownership, routing, feature flags, entrypoints, migrations, and rollout implications when the affected contract crosses those boundaries.

# Performance Discipline

- During implementation and review, examine performance effects on the affected common and failure paths. Pay particular attention to RPC and IO count, scan bounds, repeated parsing or conversion, per-item allocation and copying, lock scope and contention, concurrency bounds, queue and cache growth, and work shifted into background or recovery paths.
- Keep the common path simple and evaluate performance as a whole-system property. Support material performance decisions and claims with evidence appropriate to their risk, and include complementary paths and resource costs in that evidence.

# Finish the Change Completely

- Remove implementation made redundant or obsolete by the change, including superseded helpers, adapters, branches, flags, state, imports, comments, configuration, and tests. Do not leave two approaches, transitional wiring, or dead compatibility paths without an explicit current requirement.
- Tests must protect the current behavioral contract, not implementation history. Delete or rewrite tests whose contract is obsolete or duplicated, but do not remove a test merely because the implementation path that originally motivated it changed; retain coverage for every distinct current guarantee.
- Add comments for key classes, functions, and variables, and for tricky or complex implementations. Explain their purpose, invariants, or reasoning rather than merely restating the code.
- Comments, documentation, commit messages, and PR descriptions describe the final state and its overall effect, not the path taken to reach it. Remove commentary that narrates patch history, rejected attempts, or obsolete behavior. Never write comments, names, or docs about what the code does *not* do or to prove feedback was followed — the final state should read as if the rejected approach never existed.

# Verify the Contract

- Convert the task into observable checks. For bugs, reproduce the failure with a focused test when feasible, then add regression coverage at the most appropriate level. For validation changes, cover invalid and boundary inputs. For refactors, verify preserved behavior.
- Use the smallest set of focused unit, integration, end-to-end, static, or performance checks that adequately covers the changed contract and important failure modes. Choose the test layer by behavior, not by a blanket preference for unit tests.
- Do not add tests, assertions, logging, guards, fallback paths, or validation layers merely to appear thorough. Extra checks must protect a real current contract, realistic failure mode, or material risk.
- Do not claim completion until relevant verification has run and its result is known. If verification cannot run or unrelated failures prevent a clean result, state exactly what was and was not verified.
