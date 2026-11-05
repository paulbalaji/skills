# Measurement and reporting

## Design a comparison that can reject the hypothesis

Choose metrics and acceptance criteria before seeing the candidate's results.
Measure the limiting resource and user-facing outcome: p50/p95/p99 latency,
throughput, CPU, allocations, RSS/GC, database calls/bytes, lock and queue waits,
or cost per completed operation as relevant. Include correctness, error rate,
timeouts, and peak in-flight work. Define acceptable tail/resource tradeoffs.

Capture baseline and candidate source revisions, dirty patches if any, lockfile
and build/artifact identity, pinned toolchain, optimization/build flags, hardware,
OS, database settings, fixture seed/size, request distribution, concurrency,
arrival rate, cache state, commands, and raw samples. Use comparable release
builds. Reuse one artifact directory and a machine-readable run manifest.

Keep workloads equivalent across versions. Include representative, worst-case,
empty/small controls, skew, ready and blocked consumers, and relevant failure
cases. Benchmark setup must not dominate the operation being measured. Test
both individual components and the affected end-to-end path where feasible.

## Repeat without fooling yourself

- Warm up runtime/JIT and steady-state resources; state what was excluded.
  Measure cold-start/cold-cache behavior separately when relevant. Removing an
  application cache requires a comparison that includes its previous hit/miss
  mix and the resulting downstream load.
- Alternate or randomize baseline/candidate order and use multiple independent
  runs. Keep unrelated load stable. Do not treat thousands of correlated samples
  inside one run as thousands of independent experiments.
- Choose run length/repetitions for observed variability and a meaningful effect
  size. Report absolute and relative differences with confidence intervals and
  the method, for example a bootstrap over independent runs. If noisy or
  underpowered, say inconclusive; do not keep sampling until a win appears.
- Tail estimates need sufficient observations. State sample counts and avoid
  unsupported p99 claims. Retain outliers unless a documented experiment failure
  invalidates the run; retain neutral and regressive cases.
- For load tests, sweep offered load and concurrency through saturation within
  resource bounds. Record offered and achieved throughput, queue delay, errors,
  and timed-out requests. Include scheduled-arrival latency when testing an SLO
  so a stalled generator cannot hide demand through coordinated omission.
- Separate injected I/O, mocks, microbenchmarks, local real-database tests,
  live reads, and production end-to-end observations. A synthetic speedup is not
  a production speedup. Projections must name their assumptions.

Statistical significance alone does not establish operational value. Report the
effect size and uncertainty; a tiny stable gain may not justify a complex rewrite.
Do not claim significance unless the experiment and analysis support it.

## Make the evidence inspectable

Deliver exact commands, the manifest, raw machine-readable samples, summary
tables, and a visualization generated from those samples. Prefer the project's
existing plotting tools; avoid adding a dashboard stack for a static comparison.

- Compare before/after latency and resource use with units and uncertainty.
- For scale tests, plot latency and throughput against actual dataset size or
  load; label logarithmic axes and the dimensions being scaled.
- Show failures, timeouts, budgets, and tail regressions. Distinguish missing
  measurements from zero values; do not connect unmeasured cells as observations.
- Define ratios: latency speedup = before / after; throughput gain = after /
  before. Always show absolute values alongside them.

Use a concise result ledger when several changes are involved:

| Change | Workload | Before | After | Uncertainty | Correctness | Decision |
| --- | --- | --- | --- | --- | --- | --- |

Link retained artifacts and name unrun checks. After authorized deployment,
compare representative traffic windows and verify version, error rate, tails,
resource consumption, queue/pool health, and the original bottleneck. Account
for traffic mix before attributing changes to the optimization. Keep local
performance acceptance separate from live rollout verification.
