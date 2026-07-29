---
name: performance-engineering
description: Engineer and validate production performance changes. Use when implementing a deliberate optimization, reviewing a material hot-path change, or proving a latency, throughput, CPU, memory, RPC/IO, concurrency, cache, or scheduling improvement. For an unexplained performance regression, use diagnose first.
---

# Performance Engineering

Treat a performance improvement as a causal claim about observable system behavior. Prove the mechanism and account for cost transferred to complementary paths, resources, and lifecycle phases.

For an unexplained regression, use [`diagnose`](../diagnose/SKILL.md) until a reliable reproduction and confirmed cause exist, then resume here rather than duplicating its hypothesis loop.

For a review-only request, do not edit. Apply the evidence gates to the pinned artifact and treat an unsupported claim or unaccounted cost transfer as a finding.

## 1. Define the performance claim

Derive as much as possible from existing code, tests, benchmarks, metrics, profiles, and production documentation before asking the user for workload details.

Record only what is needed to test the claim:

- the operations and conditions under which the change should matter;
- the target metric and current baseline, or a falsifiable proxy when end-to-end measurement is infeasible;
- correctness invariants and complementary paths or resources that must not materially regress;
- the benchmark, profile, trace, counters, metrics, or logs that can distinguish success from noise.

Reuse an existing representative workload when possible. For a new benchmark, keep setup outside the timed region, verify the work is not optimized away, include warmup where state converges, run repetitions, and preserve distributions when averages can hide behavior. Alternate or randomize baseline and candidate runs when environmental drift is plausible.

If only a proxy such as RPC count, bytes transferred, allocations, instructions, or queue depth is available, label the end-to-end effect unverified; never invent a number.

**Gate:** The claim must have a target, comparison method, and validation for each plausible material cost transfer. Missing end-to-end evidence must be explicit, not silently approximated.

## 2. Build the causal cost model

Trace the critical path through callers, callees, shared resources, and background work. Use the relevant [performance lenses](references/performance-lenses.md) to locate where time or resources are consumed. Distinguish service time from queueing, useful work from coordination, and the origin of a cost from where it becomes visible. Estimate the removable portion before adding complexity.

State one falsifiable causal chain:

```text
Under <condition>, <mechanism> causes <cost>.
Changing <specific work or ownership> should move <target or proxy>
without materially regressing <complementary paths or resources>.
```

Instrument one variable at a time when evidence is ambiguous. Abandon a theory when its predicted observation does not occur.

**Gate:** Do not propose specialized production changes until evidence identifies a material cost and predicts an observable result. If no cause is established, report the evidence gap and next discriminating measurement.

## 3. Choose the simplest whole-system change

Prefer, in order:

1. delete unnecessary work;
2. reuse results already computed or fetched;
3. batch, deduplicate, or reshape dataflow;
4. improve the local algorithm, representation, or ownership;
5. add specialized machinery such as caching, concurrency, SIMD, sharding, or background work.

Account for where every removed cost goes. A write optimization that shifts work into reads, compaction, recovery, memory pressure, or operations is a cost transfer, not a free win. New state must have an owner, budget, invalidation or lifecycle rule, backpressure, failure behavior, and cleanup path.

Keep data-dependent concurrency explicitly bounded. Add indirection or a new seam only when it reduces total complexity or represents real variation. Fix missing invariants at their natural owner instead of hiding their cost behind a fallback.

If the best option causes a material regression outside the target, present the evidence, benefit, regression, and alternatives, then obtain user confirmation before implementation.

**Gate:** Proceed only with a coherent design whose expected gain follows from the causal model and whose material cost transfers are either acceptable or explicitly approved.

## 4. Validate the claim

Verify correctness first, then compare baseline and candidate under equivalent conditions. Use enough warmup and repetitions to characterize variance; inspect distributions and tail behavior when queueing or fan-out matters. A microbenchmark supports only a micro-level claim unless an end-to-end result connects it to system behavior.

Confirm the mechanism as well as the outcome: the expected RPCs, bytes, allocations, instructions, waits, or queueing must actually decrease. Recheck plausible cost transfers, including failure, overload, recovery, and complementary read/write paths. Keep the baseline recoverable through version control while testing; do not add a runtime fallback solely for comparison.

Classify the result precisely:

- **Demonstrated:** the target improves repeatably beyond noise, the causal mechanism is observed, and material complementary effects pass.
- **Directionally supported:** a proxy or mechanism improves, but representative end-to-end impact remains unmeasured.
- **Not demonstrated:** evidence is missing, the result is within noise, the mechanism did not change, or a material regression appears.

Retain a candidate introduced solely for performance only when the result is demonstrated, or when an improved proxy is itself an agreed target and no material complementary regression appears. In the latter case, keep its end-to-end effect explicitly unverified. When the result is not demonstrated, restore the baseline and remove the candidate unless it has an independently justified correctness, simplicity, or maintainability benefit, or the user explicitly approves retaining it after reviewing the evidence and risk.

After deciding which implementation remains, remove the rejected approach, temporary probes, and tests or configuration unique to it that no longer protect the current contract. Do not leave both implementations or a dormant compatibility path.

**Gate:** Report the tested conditions, causal evidence, before-and-after result, classification, complementary effects, residual risk, and final disposition. Never present an unqualified performance-only candidate as complete or upgrade a directional result into a demonstrated performance win.
