---
name: optimise-the-fuck-out-of-this
description: Find and fix performance bottlenecks through runtime evidence, hot-path analysis, structural redesign, and reproducible before/after profiling. Use when asked to aggressively optimise or optimize a codebase, remove sequential async work or contention, improve data layout or database access, or prove scalability under load. An audit or plan stays read-only; implementation requests carry through fixes and measurement.
license: MIT
metadata:
  author: paulbalaji
  version: "1.0.0"
---

# Optimise the fuck out of this

Do less work. Move less data. Coordinate less. Prove the improvement.

Scan the requested surface deeply, identify the actual limiting resources, and
implement the highest-value changes within scope. Prefer eliminating repeated
work and improving algorithms, query shapes, ownership, and layout before
tuning constants. Keep going through profiling, fixes, and verification when
implementation is requested. A list of suggestions alone does not finish it.

## Establish the target

Read repository instructions, identify the relevant entry points and callers,
and record the baseline revision and working-tree state. Preserve unrelated work.
Establish the workload, performance objective, and behavior that must survive:
output, ordering, consistency, freshness, authority, durability, cancellation,
retry, delivery, and public API contracts.

Infer reasonable objectives from the request and available evidence. Ask only
when a missing decision materially changes the work. An audit or plan remains
read-only. Editing code does not authorize production load generation, live
schema changes, deployment, or externally visible side effects.

## Find the expensive work

Use existing service logs, metrics, traces, profiles, query statistics, and
operational dashboards when accessible. Bound live reads by time, result count,
and query cost; record the service version, window, workload, and sampling.
Compare representative periods and account for traffic mix and deployment
changes. State access gaps and continue with local evidence.

Trace hot paths from entry to storage or external calls. Inspect CPU samples,
allocation and GC pressure, query plans, round trips, lock waits, queue depth,
pool saturation, payload sizes, and repeated computation as applicable.
Distinguish service time from time waiting for capacity. A source search finds
candidates; it cannot prove that they are hot, independent, or safe to change.

Keep a compact candidate ledger: location, evidence, cost mechanism, expected
gain, invariants, proposed experiment, and disposition. Rank by total workload
cost and user impact, including tail latency and resource use. Investigate
systemic causes across callers rather than polishing a cheap leaf function.

Read only the references relevant to the target:

- [Concurrency, ownership, and data layout](references/hot-paths.md) for async
  waterfalls, lock contention, repeated parsing, allocations, or memory layout.
- [Database access and scale testing](references/database.md) for projections,
  JSONB, query plans, schema changes, or relationship-aware load suites.
- [Runtime evidence and wide events](references/observability.md) when runtime
  signals are available or instrumentation needs improvement.
- [Measurement and reporting](references/measurement.md) before designing the
  baseline and comparison experiments.

## Default design bias

- Remove unnecessary sequential async work; bound genuinely independent work
  against the capacity of the whole system.
- Reduce shared mutable state. Consider ownership, sharding, or actors with
  backpressure before reaching for complicated synchronization.
- Select only the database fields the caller consumes. Keep frequently accessed,
  stable fields out of large JSONB documents when a typed column fits.
- Remove redundant parsing, compilation, allocation, copying, and serialization.
- Design around access patterns: contiguous storage, compact IDs, hot/cold
  separation, arrays, bitsets, and inline capacity where they pay off.
- Do not add caches as the performance fix. Inventory existing application
  caches and target them for removal by improving the underlying design.
  Remove each superseded cache and its invalidation machinery once equivalent
  behavior and acceptable load are proven. If removal would break freshness,
  deduplication, availability, or an external budget, explain the dependency and
  the redesign needed; never silently delete a correctness mechanism. Distinguish
  application memoization from durable state, idempotency records, and runtime
  or database-managed caches. Honor an explicit cache-removal request within
  those behavioral constraints and report any unresolved conflict.

These are strong starting hypotheses. Retain a simpler existing design when
the experiment disproves the proposed win. Lock-free code, actors, packed bits,
or a cache deletion do not count as success by themselves.

## Fix, measure, iterate

Capture the baseline before changing the implementation. Define the workload,
acceptance criteria, and experiment first. Reuse existing profiling and load
tools; add a reproducible harness when the missing experiment warrants it.

Implement coherent slices in impact order. For each slice, trace affected
callers, replace the old path, remove superseded machinery, and run focused
correctness checks plus before/after measurements. Exercise overload, failures,
cancellation, and boundary cases when the changed mechanism affects them.
Run repository-required checks for the affected scope.

Reject or revise changes that violate invariants, move the bottleneck into an
unacceptable resource, or trade better medians for unacceptable tails. Retain
neutral and regressive measurements. Re-profile after meaningful wins: the
next bottleneck may differ from the initial one.

Time-box expensive experiments according to the task budget. Stop unproductive
rewrites when evidence rejects the premise; do not invent a change to fill a
checklist. Clearly identify remaining measured bottlenecks and untested ideas.

## Deliver the proof

Lead with what improved and the measured effect. Include the important changes,
preserved behavior, baseline/candidate revisions, exact reproduction commands,
raw results, and a readable visualization of the comparisons. Show absolute
values, sample counts, uncertainty, throughput, resource cost, and tail behavior
where relevant. Distinguish measured results, extrapolation, and hypotheses.

State correctness checks, rejected changes, remaining bottlenecks, evidence
gaps, and rollout requirements. A local benchmark establishes local performance;
production claims require production evidence. Finish authorized repository
handoff work, and deploy or mutate live systems only within the user's scope.
