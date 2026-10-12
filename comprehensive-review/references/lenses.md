# Lens catalog

Pick lenses that are orthogonal for the change at hand. Each lens names the
questions a worker must answer, so two workers do not end up reading the same
code for the same reason.

## Correctness and state

- What are the state transitions, and what does each return value or error
  cause upstream (retry, drop, halt, publish)?
- Can data be skipped, duplicated, reordered, or confirmed before it is proven?
- Restart, reorg/rollback, retry, and partial-failure behaviour.
- Races between notification, read, and write; stale caches across cycles.
- Does anything loop forever without surfacing, or halt when it should retry?

## Security and authority

- Who can call each entry point; what input is attacker-controlled?
- Authentication, replay, signature or message binding, privilege boundaries.
- Denial of service: unbounded queues, per-request fan-out, slow consumers that
  stall shared work, expensive unauthenticated queries.
- Secrets in code, logs, fixtures, or recorded data.

## Resources and performance

- Calls per cycle or per request (RPC, database, network), and how they scale
  with tenants, chains, clients, or input size.
- Hot loops, O(n²) work, blocking calls on async runtimes, unbounded memory.
- Query plans on production-sized tables; index usage; keyset versus offset.
- Whether a claimed optimisation is visible in real metrics.

## Compatibility and consumers

- Who consumes the changed interface (other services, agents, UIs, SDK users)?
- Wire and schema compatibility: unknown fields, new message types, strict
  deserializers, changed identifiers.
- Migrations, rollout order, mixed-version operation, rollback safety.
- Changeset or version-bump accuracy for published packages.

## Domain semantics

- The external system the change models (contracts, protocols, chain RPC
  behaviour, payment rails): does the code match its real behaviour, including
  rounding, legacy versions, sentinel values, and error encodings?
- Read the source of the external contract or protocol rather than trusting
  names.

## Tests, simplicity, and repository rules

- Would each new test fail without the change? Do mocks diverge from the real
  dependency in ways that hide bugs?
- What could be deleted rather than hardened? Is new machinery defending
  complexity the change itself introduced?
- Repository conventions: type-safety rules, error-handling rules, formatting.

## Production evidence (post-merge only)

- Rollout timeline, resource and error deltas per build, health signals, logs,
  and data-store state. See [production evidence](production-evidence.md).

## Generalist pass

One worker from a different model family, given the whole change and a brief
list of risk areas, catches what the lens split misses. Treat its findings like
any other worker's.
