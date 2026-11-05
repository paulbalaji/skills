# Concurrency, ownership, and data layout

## Async waterfalls

Search for serial awaits, awaited loops, serialized RPC/DB reads, and helpers
that hide sequential fanout. Trace data dependencies and caller invariants.
Classify each sequence as dependent, intentionally ordered, capacity-limited,
or unnecessarily serial. Preserve transaction snapshots, locks, leases,
read-after-write checks, nonce allocation, ordering, and observable side effects.

For a small fixed independent set, consider `Promise.all` or `tokio::join!`.
For variable fanout, reuse a bounded mapper, semaphore, scheduler, stream, or
worker pool. Budget aggregate concurrency across requests, replicas, retries,
and background workers against DB pools, provider limits, memory, and deadlines.
An unbounded `Promise.all(items.map(...))` is not a general loop replacement.

Specify failure behavior. `Promise.all` rejection does not cancel sibling work;
abort and settle it when required before retrying or releasing ownership.
Choose all-settled or fail-fast semantics deliberately. Tokio `join!` multiplexes
futures on one task, waits for every branch, and does not provide CPU parallelism;
`try_join!` returns on an error, so review cancellation safety of dropped futures.
Dropping a spawned task's handle can leave the task running; explicitly manage
abort and completion where retry, shutdown, or ownership requires it.
Keep blocking/CPU work off the async executor using the project's existing
bounded execution facilities. Preserve result order and error contracts.

Test normal and partial failure, cancellation, slow consumers, saturation, and
retry overlap. Benchmark both ready and blocked paths; concurrency that merely
increases queue waits or overloads a downstream service is not a win.

Semantics: [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all),
[Tokio join](https://docs.rs/tokio/latest/tokio/macro.join.html),
[Tokio try_join](https://docs.rs/tokio/latest/tokio/macro.try_join.html).
Verify behavior against the repository's runtime and dependency versions.

## Shared state and contention

Measure lock wait time, hold time, contention, and ownership transitions.
First reduce critical sections, remove I/O under locks where invariants permit,
batch updates, or eliminate shared ownership. Consider partitioning by a stable
key, with shard skew, multi-key atomicity, and lock ordering accounted for.

Use an actor when exclusive ownership and explicit messages simplify the
hot path. Bound mailbox capacity and in-flight work. Define admission,
backpressure, ordering, acknowledgments, cancellation, shutdown/drain behavior,
and slow-consumer handling. Never replace a mutex with an unbounded queue.
Measure scheduler hops and head-of-line blocking: a single actor can serialize
the whole workload and regress uncontended performance.

Use lock-free structures only for demonstrated contention and a clear progress
requirement. Prefer established primitives. Review memory ordering, ABA,
reclamation, fairness, and false sharing when relevant. Measure uncontended,
contended, and skewed workloads. A lock-free rewrite can consume more CPU.

## Repeated parsing and temporary work

- Trace repeated `new URL()` calls. Parse once at a stable ownership boundary,
  pass required fields, and reuse immutable results. Preserve validation,
  relative-base resolution, normalization, encoding, and credential handling.
  Mutable URL objects must not leak shared state between callers. Do not replace
  standards-based parsing with ad hoc string splits.
- Hoist invariant regex construction out of functions and loops; use the
  language's existing lazy/static facility where initialization requires it.
  Dynamic patterns need their actual inputs. In JavaScript, shared `g`/`y`
  regexes mutate `lastIndex`; preserve per-call behavior or use a stateless
  pattern when equivalent. Test repeated and interleaved calls. See
  [RegExp state](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test).
- Remove repeated decoding, intermediate collections, cloning, string building,
  and serialization when profiles show they matter. Pass views or borrow when
  lifetimes and ownership permit; avoid retaining a large buffer for a tiny view.

## Data-oriented design

Start with the operations performed together, their frequency, working-set size,
and access order. Optimize locality and bytes touched per operation.

- Replace hash tables with flat arrays or dense tables when key density and
  lookup semantics fit. Include memory usage and sparse-key cases in comparison.
- Replace pointer-rich graphs with compact indices into stable storage when
  beneficial. Define deletion, compaction, invalidation, and stale-handle behavior;
  use generations where slot reuse requires them. Preserve external identity.
- Compare structure of arrays against array of structures for actual field
  access. Separate hot and cold fields; measure iteration and update workloads.
  Column-wise scans and whole-record processing may favor different layouts.
- Consider packing common flags or tags into spare index bits only after
  proving index bounds, signedness, sentinel values, and overflow behavior.
  Centralize encoding/decoding and verify round trips and boundaries. Account
  for JavaScript's numeric and bitwise limits when applicable.
- Replace heap-backed short lists with an existing inline-capacity collection
  where supported. Choose capacity from observed distributions; measure spills,
  object/stack growth, moves, and recursion. Inline storage is not necessarily
  stack storage; JavaScript has no general stack-allocated array equivalent.
- Use a bitset for dense boolean state when compactness or bulk operations help.
  Check sparse alternatives, iteration cost, tail bits, concurrent updates, and
  false sharing. Do not reinterpret externally serialized arrays without a
  deliberate format change.

Benchmark realistic sizes, sparsity, skew, and mutation patterns. Check allocation
count, bytes, cache misses when available, latency, and throughput. Do not force
low-level layouts into a runtime where they add conversions and allocations.
