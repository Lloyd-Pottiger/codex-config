# Performance Lenses

Select only the lenses implicated by the performance claim or hot path. Use them to locate a causal mechanism or cost transfer; do not mechanically report every item.

## RPC and IO

- Count requests, round trips, keys, rows, bytes, scans, seeks, flushes, and synchronization points per operation.
- Look for N+1 behavior, missing batching or deduplication, fetching unused data, loose scan bounds, and repeated reads.
- Check whether batching improves throughput by increasing latency, memory, or failure blast radius.

## CPU and memory

- Look for avoidable parsing, encoding, hashing, conversion, branching, indirection, and invariant work inside loops.
- Count allocations, retained bytes, large copies, temporary buffers, cache misses, and pointer chasing where material.
- Check whether a lower CPU cost moves the bottleneck to memory, IO, locks, or downstream services.

## Concurrency, queues, and locks

- Separate service time from queueing time and locate the saturation point.
- Check fan-out bounds, scheduling overhead, cancellation, fairness, backpressure, and overload collapse.
- Check lock scope, acquisition frequency, ordering, contention, shared cache lines, and locks held across suspension or blocking work.
- Treat more concurrency as a load multiplier unless the downstream capacity and tail effect are measured.

## Caches, storage, and background work

- Identify the source of truth, capacity budget, admission policy, invalidation, eviction, warmup, and recovery behavior.
- Account for write amplification, compaction, replication, checkpointing, index maintenance, and cache coherence.
- Include retry and repair work, partial failure, stale data, startup, shutdown, and backlog drain costs.

## Whole-system cost transfer

- Read versus write latency and throughput
- Foreground latency versus background work
- Steady state versus startup, recovery, rebuild, and migration
- Median versus tail latency
- Throughput versus fairness and overload behavior
- CPU versus memory, network, storage, and external service load
- Single-request speed versus batching delay and failure blast radius
- Local win versus fleet cost, observability cost, and operational complexity

## Evidence integrity

- Does the workload or proxy represent the operation, data shape, scale, concurrency, and state relevant to the claim?
- Are setup, teardown, fixture generation, and unrelated IO outside the measured region?
- Are warmup, duration, repetitions, run ordering, machine state, and configuration controlled well enough to compare results?
- Are errors and failed operations counted separately rather than making latency look artificially good?
- Are distributions, tail latency, throughput, and saturation considered where averages can hide behavior?
- Is the claimed improvement larger than run-to-run variance and practically meaningful?
