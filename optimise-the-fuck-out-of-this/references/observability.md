# Runtime evidence and wide events

Read [Logging Sucks by Boris Tane](https://loggingsucks.com/) when applying this
part of the skill. If unavailable, disclose that and use the summary below.

## Apply the article's model

Use one structured completion event per request or job per service, enriched
through its lifecycle. Include correlation IDs, deployment/version, outcome,
duration, relevant domain context, and downstream work. Emit on failure as well
as success. Join events across services using consistent identifiers. Traces
and wide events complement each other; merely adopting a telemetry SDK does
not supply useful context.

Choose retention after the outcome is known: keep errors, slow operations, and
explicit diagnostic cohorts; sample ordinary successes to control volume.
Record the sampling policy so analysis can account for it. These practices are
adapted from the article; the constraints below apply to this optimization work.

## Instrument the questions you need to answer

Reuse the existing logger, request context, middleware, and telemetry stack.
Cover in-scope entry points with a shared lifecycle integration where possible,
rather than scattering per-step logs or creating a new framework. For long-lived
streams and workers, define useful bounded operation/interval boundaries;
do not wait indefinitely for a process to end before producing evidence.

Capture query/call counts, bytes, queue/pool/lock waits, retries, batch sizes,
and stage timings when they explain the cost mechanism. Prefer monotonic timing.
Derive outcomes from actual status/domain results, including handled errors.
Retain audit, security, startup, and crash diagnostics that completion events
cannot replace. Logging failures must not alter business outcomes.

Bound event size and serialization work. Redact credentials and sensitive
payloads, and follow existing retention/access rules. Keep high-cardinality
identifiers in logs/traces where supported; do not turn them into unbounded
metric labels. Do not add database/RPC calls just to enrich a log event.

Verify completion behavior for success, handled and unhandled error, timeout,
and cancellation within the runtime's guarantees. Check isolation between
concurrent requests, correlation propagation, sampling, redaction, and logging
overhead. Sampled completion events cannot reconstruct unbiased request rates
or latency distributions without suitable weighting; use unsampled aggregate
metrics for acceptance when available.

Apply this consistently across the requested surface when logging work is
in scope. Otherwise add only instrumentation needed to resolve a performance
question; do not turn a focused optimization into an unrelated logging migration.
