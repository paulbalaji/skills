# Presenting improvements

Every number should help answer: what changed, for which workload, by how much,
and why does it matter? Lead with the result and its scope. Separate measured
benefits, unchanged behavior, costs, and remaining uncertainty. One clear
sentence can be better than a table; a table is useful for comparable results
or a material tradeoff. Do not create one row per edited function.

## Choose the outcome before the unit

Pick a primary metric for the bottleneck or user/operator decision. Add a
supporting metric only when it explains the mechanism, tests the claim, or
exposes a tradeoff. Preserve relevant adverse results even when they spoil the
headline. Keep exhaustive measurements in linked evidence, not the summary.

| Question | Useful evidence | Boundary to make explicit |
| --- | --- | --- |
| Does the user wait less? | End-to-end wall latency in ms/s; median and relevant supported tail | Request, batch, startup, or recovery; queue time included/excluded; offered load and errors |
| Does the service do less CPU work? | CPU time in ms/op or CPU-seconds for a fixed workload; cores at matched traffic | CPU time differs from elapsed time; normalize completed work and state measurement scope |
| Did a CPU kernel improve? | ns/op or µs/op; hardware cycles/op or instructions/op if actually counted | Hardware/toolchain, build, workload and counter source; cycles, instructions, VM compute units, and time are distinct |
| Does it use less memory? | Peak or steady RSS, live heap, retained bytes, allocation count/bytes, as appropriate | Name the actual measure and observation point; requested allocation bytes are not retained memory or RSS |
| Does it scale further? | Achieved operations/s under a latency/error budget; saturation point | Offered load, concurrency, dataset/fanout, failures and resource limits; higher concurrency alone is not more capacity |
| Does it acquire or move less? | Calls/op, rows/query, decoded bytes, wire bytes, WAL or artifact size | Name the layer; logical calls differ from physical RPCs, and decoded data differs from compressed transfer bytes |
| Does polling improve the service? | Freshness/detection/recovery latency alongside calls/time and provider cost | Lower request rate must preserve freshness, finality, recovery and signing/execution requirements |
| Does CI finish sooner or cost less? | Completion time/critical path, summed runner time, billed cost | Queue vs execution, runner sizes, cache state, matched passing jobs and retries; wall time is not runner cost |

Counts are valid outcomes: `2 → 1 adapter calls per eligible operation` or
`one allocation removed per submission` can prove eliminated work without a
timing claim. Include path eligibility or frequency when known. State that
latency, production prevalence, or billing impact is unmeasured when it is.
Do not add a noisy microbenchmark merely to attach a speedup to a deletion.

CPU utilization percentages need their denominator: one core, all machine
cores, or a container quota, over a stated interval. Prefer cores for service
usage when that avoids ambiguity. CPU requests/limits are configuration, not
measured consumption. Do not estimate hardware cycles from wall time and a
nominal clock. Keep Solana compute units and provider-billed compute units
explicitly named; neither means CPU cycles or automatically predicts fees.

## Choose absolute values, percentages, or multiples

Always make absolute before/after values available. A relative figure alone
hides the scale of the benefit. Use one relative expression when useful; avoid
repeating the same result as a ratio, percentage, and delta in separate columns.

| Presentation | Use when | Definition / example |
| --- | --- | --- |
| Absolute reduction | Time, bytes, calls, or budget headroom is the practical result | `before − after`; 120 → 90 ms saves 30 ms per request |
| Percentage reduction | Incremental gains or reduced CPU, memory, bytes, calls, CU, or cost | `(before − after) / before × 100`; 120 → 90 ms is 25% less time |
| Speedup multiple | A several-fold time improvement is easier to grasp as a ratio | `before / after`; 120 → 30 ms is 4× speedup, or 75% less time |
| Throughput increase | More work completes per time under the same acceptance conditions | `(after − before) / before × 100`, or `after / before` for a multiple; 100 → 150 req/s is 50% more throughput |
| Percentage-point change | Comparing rates already expressed as percentages | Error rate 0.3% → 0.1% falls 0.2 percentage points; relative reduction is about 67% |
| Removed / newly feasible | Work reaches zero, a baseline fails, or a budget is crossed | `20,000 → 0 reads`; `failed at 200k CU → completes at 8,780 CU`; no invented infinite speedup |

Choose the expression that communicates the effect; no universal cutoff forces
every result into × or %. Small changes usually read better as an absolute
delta and, if supported, a percentage than as `1.03×`. Say `no material change`
or `inconclusive` when noise or operational relevance does not support a win.
For regressions, use `12% more CPU` or `1.4× as long`, with the absolute values.
Avoid ambiguous `times faster/slower`, `4× reduction`, or `100% faster` wording.

A 2× latency speedup halves elapsed time; it is not a 100% time reduction.
Define whether a reported ratio is a ratio of medians, a median of paired
ratios, or another statistic. Keep the estimator consistent and do not average
unrelated workload speedups into a service-wide claim. Do not invert p50 latency
to manufacture throughput in a queued or concurrent system.

## Make units and denominators readable

- Use ms for most request/stage latencies, s or min:s for long operations,
  and µs/ns for small CPU kernels. Keep before/after in the same unit and
  comparable rows consistent. Label batch size and use `/op`, `/request`,
  `/batch`, `/second`, or `/day` where needed.
- Do not label a 10,000-call batch's milliseconds as per-call latency. Dividing
  batch duration by its count yields average time per operation for that batch,
  not a per-operation p95/p99 distribution. State which is measured or derived.
- Distinguish MB/GB (decimal) from MiB/GiB (binary). Convert explicitly and keep
  both sides consistent. Call a sampled RSS observation `RSS at batch completion`
  rather than `peak RSS`. State process/pod/fleet and heap/native scope.
- Use CPU ms for accumulated processor time and elapsed ms for wall time.
  Multithreaded CPU time can exceed elapsed time. Report per-completed-operation
  cost when work volume differs; show the volume and errors, too.
- Round displayed results to useful precision supported by variability, normally
  a few significant digits; retain raw precision in artifacts. Show small
  positive values in a suitable unit instead of rounding them to zero. Calculate
  percentages/ratios from unrounded inputs. Exact counts can remain integers.

## A table row must earn its place

For each proposed row, check:

1. **Identifiable work:** a reader can name the operation and relevant size,
   fanout, cache/start state, or arrival load. Keep controls distinct from the
   representative case; label stress fixtures as stress fixtures.
2. **Comparable endpoints:** baseline and candidate do the same required work,
   with the metric, unit, denominator, direction and boundary apparent.
3. **Decision value:** the row establishes a benefit, cost, control, limit, or
   failure threshold. It answers a question beyond "we measured this."
4. **Impact:** explain what is saved, made feasible, or still unchanged. If
   end-to-end or production impact is unknown, state the proven local effect
   and that limit; do not invent a business benefit.
5. **Evidence:** tie it to the run/source and uncertainty appropriate to the
   claim. A shared caption or linked methodology can cover common provenance.

Drop decorative counters, test totals, or unchanged metrics that do not inform
the performance decision; correctness validation belongs in its own section.
Keep meaningful controls and regressions in an adjacent compact table or note.
Do not bury a tail or memory regression behind a positive speedup column.

A useful default, adapted to the result rather than filled mechanically:

| Workload / condition | Metric: before → after | Effect | Impact / qualification |
| --- | --- | --- | --- |

For homogeneous latency cases, separate p50/p95 columns may be clearer. Include
a ratio only if it adds meaning. For mixed latency, CPU, and memory results,
group by question or clearly name each measure; never use an unlabeled `Before`
column full of incompatible quantities. Keep common hardware, repetitions and
evidence type in the caption. Avoid a wide table repeating every raw metric.

## Bound the claim to the evidence

Label deterministic work counts, controlled/injected-I/O tests, local real
databases, release kernels, live endpoint samples, production observations,
and projections. Keep that label near the result it qualifies. A health check
or successful deployment proves neither a performance gain nor lower billing.

Use representative workload frequency and affected-path share before estimating
aggregate value. Do not add percentages from different workloads, multiply
component speedups, add overlapping allocation savings, or count the same
measurement reused across configurations as independent experiments. Sum actual
compatible work/cost measurements, not ratios; weight them only with a justified
workload mix. Stage timings can overlap and need not sum to request latency.

Compare production windows by traffic mix, volume, uptime/catch-up phase, version
and error rate. Report confounding and adverse signals. Forecasts need explicit
rates, eligibility, prices and assumptions. A short provider-usage sample is an
observation, not a daily/monthly bill or a hard capacity allowance.

When presenting a narrow helper speedup, include the absolute saving and its
share of the larger path if measured. Without that share, state that total
request impact is unknown. An optimization may be worthwhile because it removes
an allocation or simplifies ownership even when latency is unchanged; say that.

## Example: a compact comparison with impact

Historical values from [the packed-prover PR](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9691),
reformatted rather than rerun: local optimized isolated prover, 1,048,576 leaves,
medians of three alternating runs. These are descriptive measurements; no
statistical significance or deployed relayer improvement is established.

| Workload / condition | Metric: before → after | Effect | Impact / qualification |
| --- | --- | --- | --- |
| Isolated prover process | Peak RSS: 137.3 → 69.7 MB | 49% lower | 67.6 MB less peak process memory at this tree size; pod RSS unmeasured. |
| 10,000 historical proofs | Batch elapsed: 430.6 → 32.6 ms | 13.2× speedup | About 398 ms saved per batch; full request impact unmeasured. |
| Build prover from leaves | Elapsed: 5.734 → 5.781 s | 47 ms longer | The change does not demonstrate faster tree construction; retain this control. |

The rows explain memory headroom, faster proof work, and the construction
tradeoff. They do not claim that the relayer became 13.2× faster or that its
pod memory fell 49%. If only one of these outcomes matters to the request,
lead with that result and keep the others as concise controls or qualifications.

## Lessons from public performance PRs

These are historical reporting examples, not newly reproduced benchmarks.
Apply the lessons rather than copying their numbers into another workload.

| Example | Lesson for the output |
| --- | --- |
| [Bounded concurrency #9958](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9958) | Public-RPC p50 improves while the sparse tail sample worsens. Ready-subscriber latency improves dramatically while full broadcast completion still waits for the slow subscriber. Name the completion boundary. |
| [WebSocket encoding #9614](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9614) | CPU ms, received frames/s, batch completion, and RSS at completion answer different questions. Retain the 1,000-recipient memory increase and distinguish modeled fanout from measured future resource use. |
| [Buffer ownership #9523](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9523) | Removed allocations/requested bytes are useful proof without inventing CPU, RPC, or fleet-RSS savings. |
| [Narrow reads/parsing #9959](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9959) | A 10,000-call URL helper and a release regex kernel are component benchmarks. Decoded JSON bytes are not PostgreSQL wire bytes; an unoptimized test-profile result is not release throughput. |
| [Reconciliation #9226](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9226) | The matched canary supports fewer slow queries while CPU rises. Report the SQL win without claiming a CPU win; polling models also need discovery/recovery latency. |
| [Sealevel CU #9858](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9858) | Use CU per operation, preserve cap failures as outcomes, and distinguish per-invocation totals including CPIs from program-self totals. Neither is hardware cycles or elapsed time. |
| [Depot CI #9300](https://github.com/hyperlane-xyz/hyperlane-monorepo/pull/9300) | Workflow completion, median job speedup and runner cost are separate. Match resource/cache states, retain failed jobs, and show slower families alongside the overall improvement. |

For a PR, keep Problem / Solution / Validation concise: problem and resulting
behavior, the decisive scoped comparison with impact, then reproduction and
correctness evidence plus material limits. Link the detailed report. Use the
same honest boundaries in a chart or social summary; shortening the format does
not permit stronger claims.
