# Design Lenses

Select the lenses implicated by the proposed change. Use them to discover decisions and contradictions; do not copy these headings or add empty “not applicable” sections to the design document.

## State and data

- What is the source of truth, who owns it, and which representations are derived?
- What invariants, state transitions, cardinalities, retention rules, and consistency guarantees apply?
- How are schema changes, existing data, backfills, mixed versions, and rollback handled?
- Are operations idempotent where retries or replay can occur?

## Interfaces and dependencies

- What must each caller know: types, invariants, ordering, error modes, configuration, and performance characteristics?
- Which dependencies are in-process, locally substitutable, remotely owned, or truly external?
- What versioning, compatibility, rate, quota, timeout, and ownership contracts cross each seam?

## Concurrency and distributed behavior

- What can race, reorder, duplicate, disappear, or arrive late?
- What are the lock, transaction, atomicity, isolation, and cancellation rules?
- How does the system behave under partition, partial success, retry, failover, and duplicate execution?
- Where are concurrency, queue, and retry budgets enforced?

## Failure and recovery

- How is each failure detected, contained, surfaced, retried, repaired, or declared terminal?
- Can recovery resume safely, and what happens to in-flight or partially migrated work?
- Does a fallback preserve the contract, or merely hide a broken invariant while adding another mode?

## Performance and capacity

- What are the critical read, write, background, recovery, and control paths?
- Which latency, throughput, CPU, memory, network, storage, or external-service budgets constrain the design?
- Does improving one path transfer material cost to another path or lifecycle phase?
- How does behavior change at saturation, during bursts, and at projected scale?

## Security and privacy

- Where are trust boundaries, authentication, authorization, secrets, and sensitive data?
- How are validation, tenancy isolation, auditability, retention, deletion, and abuse controls enforced?
- Does the design expand access, persistence, or attack surface?

## Operations and delivery

- How is the change configured, observed, deployed, staged, rolled back, and removed?
- What signals distinguish success, degradation, overload, and partial failure?
- Can old and new versions coexist safely, and what is the point of no return?
- Who owns ongoing capacity, cache, queue, retry, migration, and incident response behavior?

## Verification

- Which observable test proves each goal and invariant at the highest useful seam?
- Which failure, concurrency, migration, compatibility, and performance claims need integration, end-to-end, stress, fault-injection, or benchmark evidence?
- What evidence must exist before rollout, and what production signals determine progression or rollback?
