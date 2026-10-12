# Production evidence for post-merge reviews

When a change is already running, measure it instead of reasoning about it.
Everything here is read-only.

## Build a rollout timeline

From the orchestrator's history (pod or container image labels, deploy logs,
release records), list each build and when it started in each environment. Map
builds to commits or PRs. Every later comparison is per window between two
rollouts, and short windows are dominated by catch-up, so say so.

## Compare windows

For each window report median and p95, not single samples:

- CPU, memory working set and resident memory, threads, file descriptors,
  network. Note growth slopes within the current window (leaks show as steady
  slopes; caches show as steps that flatten).
- Request rates by dependency and method (RPC, database, HTTP) and their error
  rates. Check whether a claimed reduction is actually visible.
- Health signals the service exports (critical-error flags, lag, liveness), and
  which ones flap.

Attribute changes to the right build. An improvement can come from a different
PR than the one claiming it.

## Logs

Pull warnings and errors since the current build started, then aggregate
locally: count by normalized message and by tenant or domain, with one sample
line per bucket. Grep explicitly for the failure modes the code review predicted
(gap, rewind, halt, retry, incomplete) and report zero hits as evidence too.

## Data store state

Use a read-only credential, loaded into a variable and never printed. Bound
every query with indexed predicates and limits; production tables can be huge.
Useful checks: per-tenant progress and lag, rows that stopped updating, entities
that are configured but never initialized, and leftover state for removed
configuration.

## Report

Lead with the numbers that answer the user's question. Rank anomalies that look
like bugs, each with its evidence, and separate problems that predate the change
from ones that started with it. Mark every inference.

## Hygiene

Stop port-forwards and background collectors you started. Keep raw data in a
scratch directory and tell the user where it is so they can delete it.
