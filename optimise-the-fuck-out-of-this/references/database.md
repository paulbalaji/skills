# Database access and scale testing

## Fetch and touch less

Trace ORM/query-builder calls through generated SQL and downstream consumers.
Find `SELECT *`, full-entity hydration, oversized relations, and JSONB documents
loaded for a few fields. Project required columns at the query boundary and
update types/callers. Include keys needed for joins, authorization, pagination,
ordering, and downstream behavior. Check ORM hidden loads and hooks.

Find N+1 access, repeated round trips, unbounded result sets, deep offsets,
large sorts, unnecessary counts, and work performed in application code that
the database can avoid. Prefer set-based queries or bounded batching when
semantics fit. Do not hide an inefficient query behind a cache or merely enlarge
the connection pool. Inspect actual plans and measurements before adding indexes;
include write amplification, index size, and maintenance in the tradeoff.

## Stop treating large JSONB as a record store

Inventory hot-path reads, filters, joins, grouping, and ordering on individual
JSONB keys. For a stable frequently used key, prefer a typed real column and
appropriate constraints/indexes; keep genuinely variable payloads in JSONB.
Measure bytes returned, decoding cost, storage/TOAST work, plans, and write cost.
An expression index may help a predicate but does not eliminate full-document
transfer or application decoding. Compare options against the actual access path.

For a column promotion, define the source of truth and preserve missing versus
JSON-null versus SQL-null semantics, types, invalid values, defaults, and updates.
Cover every writer, backfill historical rows in bounded batches, validate
equivalence, cut reads over, and remove transitional duplication when safe.
Account for migration locking, table rewrites, concurrent old/new versions,
and forward recovery. Prepare migration code locally; live application needs
the user's rollout authorization. Do not create permanent dual-write drift.

## Build the relationship-aware load suite

When database scalability is in scope, deliver an executable repository-local
suite using its real schema, migrations, indexes, and application queries.
Reuse the repository's test/container tooling. Include seeded generation,
setup, query execution, result capture, comparison, plotting, and cleanup.
Keep it opt-in when too expensive for ordinary CI.

Use a disposable database with an explicitly identified target and fail-closed
target checks. Never infer permission to generate load against production from
read access. Cap rows, bytes, concurrency, connections, and runtime; provide a
dry-run size estimate and a way to stop. Do not copy sensitive production data
into fixtures or commit it with results.

Construct baseline, 10x, 100x, and 1000x scenarios from real relationships:

- Preserve foreign keys, cardinalities, uniqueness, tenant isolation, lifecycle
  states, time ranges, and query selectivity. Include skew, large tenants,
  fanout, sparse cases, and realistic payload/JSONB sizes.
- Vary total population and per-parent fanout separately, then combine meaningful
  extremes. Define exactly which dimension each multiplier scales. Multiplying
  every table equally can conceal the expensive joins; multiplying each fanout
  can explode row counts. Show estimated and actual sizes.
- Use the application's real query paths, parameter distributions, pagination,
  transactions, read/write mix, and concurrent clients. Validate returned data
  so a broken query cannot appear faster by doing less required work.
- Pin database version/configuration and application/pool settings. Refresh
  planner statistics after loading fixtures. Capture plans with realistic bind
  values and prepared-statement behavior where relevant.

For PostgreSQL, capture `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` where appropriate
in the isolated environment. ANALYZE executes the statement; mutation and
external trigger effects need isolation even when a transaction is rolled back.
Collect normal query latency separately from instrumented plan runs. Inspect
estimate/actual row mismatches, scan and join amplification, spills, I/O, locks,
and pool waits. See [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html).

Use [the measurement protocol](measurement.md) for repeated baseline/candidate
runs on equivalent fixtures. Set per-query/transaction deadlines and record
timeouts, errors, saturation, and resource limits as results. Never drop failed
queries from latency summaries without reporting the failures alongside them.

Profile the baseline, fix the demonstrated bottleneck, and re-run the same suite.
Show latency and throughput versus scale, uncertainty, and the first observed
failure or budget breach. Distinguish observed cliffs from extrapolated ones.
If 1000x exceeds the experiment budget, report the unrun cell and size estimate;
keep the suite reproducible for that scale rather than inventing measurements.
