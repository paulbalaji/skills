# Repository De-Slopping: An Evidence-First Operating Guide

De-slopping is the disciplined removal of accidental complexity, dead paths,
duplicated concepts, misleading configuration, weak contracts, and low-signal
automation. The goal is not to make a repository smaller at any cost. The goal
is to make its behavior easier to prove, change, review, and operate.

This guide is intentionally language-, framework-, and company-agnostic. It is
designed for maintainers and coding agents working in mature repositories where
security boundaries, production behavior, and existing user work must survive
cleanup.

The examples use Git and pull-request terminology. Where a repository uses a
different version-control or review system, apply the equivalent concepts:
immutable revision, isolated checkout, change request, independent review, and
landed-revision readback.

Match actions to the user's authorization. A request for a plan, approach,
review, audit, or issue remains read-only. Repository edits, forge writes,
merges, deployments, and cleanup deletions require explicit implementation
authorization. Permission to edit or merge code never implies permission to
mutate production.

## The standard

A de-slopping change is complete only when it:

1. reproduces or disproves the problem before editing;
2. identifies the current consumer, threat, or demonstrated failure behind
   every retained or introduced concept;
3. deletes the superseded path instead of hiding it behind another layer;
4. preserves security, authority, data exposure, and production invariants;
5. adds or reuses the narrowest mechanical guard when a material regression can
   recur and existing enforcement does not catch it;
6. includes a negative fixture when a guard is added or changed;
7. measures additions, deletions, net change, and any relevant runtime cost;
8. passes focused and affected-system verification on the exact reviewed head;
9. merges only green code whose base has not drifted; and
10. verifies the required post-merge or production outcome, rather than treating
    a merge as proof of live behavior.

If the evidence is missing, the cleanup is not done.

## What de-slopping is not

Do not substitute any of these for engineering judgment:

- a generic "AI slop" classifier;
- a repository-wide clone detector;
- an arbitrary file-size or line-count cap;
- a rule requiring every code change to modify a test file;
- whole-repository mutation testing;
- automatic deletion of static-analysis findings without runtime reachability
  proof;
- global warning suppression or weakened lint, type, or security rules;
- a plugin framework built around one current caller;
- compatibility aliases, shadow paths, adapters, or re-exports with no named
  external migration need and deletion date; or
- a broad architecture rewrite smuggled into a cleanup issue.

These mechanisms are attractive because they are easy to count. They usually
measure proxies, create compliance theater, and miss the actual boundary.

## 1. Establish authority and invariants

Before touching code, write down:

- the repository and exact target branch;
- repository-local instructions and current-state documentation;
- the product behavior that must remain true;
- security and trust boundaries;
- public API or data-projection allowlists;
- production and deployment constraints;
- unrelated local work that must be preserved; and
- actions that require human or external authority.

Treat historical plans as history unless the repository explicitly marks them
as current. Treat the current code, schema, configuration, and live system as
authoritative evidence, not assumptions.

## 2. Pin the baseline

Fetch the target branch and record its immutable commit. Confirm:

- the primary checkout is clean;
- `HEAD` and the intended remote base are understood;
- every auxiliary worktree is inventoried;
- open pull requests and review threads are current;
- generated artifacts are distinguishable from authored files; and
- the audit commit named in an old issue has not drifted.

Do not review or merge a moving target. Re-pin the base and head after every
rebase, force-push, or upstream merge.

## 3. Build an evidence ledger

Create a concise ledger before implementation:

| Slice | Premise | Consumer/threat | Depends on | Evidence |
| --- | --- | --- | --- | --- |
| Legacy parser | No callers | None | Contract cleanup | Negative fixture |

For each candidate, answer:

- Is it reachable from a production, build, migration, operational, or public
  entry point?
- Is it intentionally retained for a named external consumer?
- For public APIs, what generated clients, external consumers, representative
  traffic evidence, and owner/product evidence exist?
- Does another slice touch the same contract, schema, or presentation seam?
- What observation would prove the issue premise false?
- What exact command, test, rendered artifact, or live readback proves success?

This ledger determines ordering. Parallelize only genuinely disjoint write sets.
Serialize changes that share contracts, presentation types, schemas, generated
files, or migration state.

## 4. Reproduce before editing

Every slice starts with evidence of the current failure or waste:

- trace entry-point reachability;
- search all callers and dynamic-loading boundaries;
- run the failing behavior or construct the smallest representative fixture;
- inspect generated and rendered output rather than source text alone;
- compare configured state with live state when operations are involved; and
- distinguish provider, CI, or environment noise from a code defect.

If the premise is false, close or rewrite the issue with the evidence. Do not
manufacture a code change to justify the ticket.

## 5. Prefer hard cutovers and deletion

When a replacement is authorized and all callers are controlled:

1. cut callers over;
2. remove the old implementation;
3. remove its types, flags, configuration, tests, docs, and exports;
4. update generated artifacts; and
5. prove the old path cannot return.

Add a compatibility layer only when there is a concrete external migration.
Document its owner, consumer, deadline, and deletion condition. "Might be useful"
is not a migration requirement.

Reuse stable domain concepts already present in the repository. A concept is
generic because several current consumers share the same semantics—not because
it has hooks, registries, factories, or extension points.

## 6. Protect trust boundaries

Cleanup must not silently broaden capability. Review changes for:

- authentication and authorization;
- sign/approve versus execute distinctions;
- server-bound payloads and revalidation;
- mutation and broadcast authority;
- public versus internal data projection;
- secret handling and build-time exposure;
- database and migration invariants;
- idempotency, retries, and terminal-state evidence; and
- default-deny or allowlist behavior.

Security controls belong at real trust boundaries and must block a named threat.
Do not duplicate execution-only controls into lower-capability read or approval
flows without evidence. Do not weaken an allowlist merely to simplify types.

For cross-language or generated response contracts, explicitly compare field
allowlists, aliases, omitted versus nullable values, error envelopes, dates,
decimals, large integers, unknown-field behavior, and public-safe projection
boundaries.

## 7. Add or reuse proportional regression protection

Choose the guard that matches the demonstrated regression:

| Regression | Appropriate guard |
| --- | --- |
| Dead production surface | Entry-point-aware reachability check |
| Import cycle | Authored runtime graph cycle check |
| Duplicate response contract | Type/schema derivation or contract parity test |
| Unknown configuration key | Strict schema parse with rendered-config fixture |
| Fragile deployment topology | Structured manifest assertions |
| Unexpected warning | Scoped test-console failure with explicit allowlist |
| Baseline laundering | Merge-base-relative fingerprint ratchet |
| Missing critical test evidence | Changed executable line/branch coverage |
| Retired environment returns | Active-surface source/config guard |

Every slice needs durable proof, but not every slice needs a new guard. An
existing compiler, build, route manifest, schema validator, or reachability
check may already prove the simpler state. A one-off deletion can be complete
with reproduced reachability evidence and existing compile/build/test coverage.
Add or modify a guard only when the failure can materially recur and current
enforcement cannot catch it. Record why existing proof is sufficient when no new
guard is warranted.

Every new or modified guard needs a negative fixture. The fixture must introduce
the forbidden condition and prove that the canonical local/CI command used to
build, start, validate, or render production inputs rejects it. Never execute a
fixture through live production mutation. A test that only asserts the happy
path does not prove a new gate works.

Good guardrails are:

- scoped to authored or active surfaces;
- structured rather than substring-based when structure exists;
- deterministic and bounded in CI;
- explicit about generated, historical, vendor, and test-support exclusions;
- difficult to bypass accidentally; and
- cheap enough that maintainers will keep them enabled.

Warning allowlists require a precise category, source, and message plus a
reason, owner, and removal condition. Do not turn temporary upstream noise into
permanent global suppression.

Apply the same discipline to static-analysis baselines, ignored entry points,
generated-code exclusions, and dependency allowlists: name the consumer or
tooling limitation that makes the exclusion necessary and define when it should
be reviewed or removed.

## 8. Perform the overengineering deletion pass

Before finalizing, list every concept introduced by the change:

| Introduced concept | Current consumer or named threat | Keep/delete |
| --- | --- | --- |
| Strict config parser | Runtime startup and rendered deployment checks | Keep |
| Generic plugin registry | One parser | Delete |

For each helper, layer, type, service, flag, table, route, metric, and workflow:

- name its current consumer or named threat;
- show why an existing seam cannot serve it;
- delete duplicate or speculative machinery; and
- record material functionality deliberately not built.

This pass is not cosmetic. It is where a merely tidy refactor becomes a durable
simplification.

## 9. Account for line economics

Measure gross additions, gross deletions, and net lines. Explain where remaining
lines go—for example:

- production logic;
- negative fixtures and regression tests;
- generated lockfiles;
- CI enforcement;
- documentation and operational evidence; or
- deliberately retained compatibility code.

Net negative is not automatically good, and net positive is not automatically
bad. A 20-line deletion backed by a 100-line precise regression fixture may be
excellent. A 500-line framework that deletes 30 lines is probably not.

Measure relevant costs before changing architecture to satisfy a budget:

- bundle or image size;
- test wall time and peak memory;
- startup or request latency;
- rendered manifest count;
- database query shape; and
- CI concurrency and retention.

Recalibrate brittle budgets narrowly when measured resource use remains
acceptable. Do not evade them with exclusions.

## 10. Verify continuously and proportionally

Use an expanding verification ladder:

1. the negative fixture, when a guard was added or changed;
2. focused unit or contract tests;
3. affected package tests;
4. lint, format, and type checks;
5. build and bundle checks;
6. schema, migration, config, or manifest checks;
7. affected backend/frontend/integration suites; and
8. the repository's required full checks.

Record exact commands and honest outcomes. A skipped, flaky, environment-limited,
or interrupted test is unverified—not passed. Generated changes must be reviewed,
not blindly committed.

## 11. Review the exact head independently

For a broad or high-risk change, an independent reviewer should inspect the
immutable pull-request head for:

- correctness and edge cases;
- security-boundary or authority changes;
- public contract and compatibility changes;
- caller blast radius;
- unnecessary abstractions and dead code;
- missing negative cases;
- CI permissions and dependency provenance; and
- mismatch between the pull-request narrative and the actual diff.

Refresh the base, head, checks, reviews, and unresolved threads immediately
before merge. Review evidence from an earlier SHA is stale.

For a narrow low-risk deletion, use the repository's normal required review.
Do not add independent-review ceremony merely because this guide was invoked.

## 12. Use small, auditable pull requests

A de-slopping pull request should contain:

- the repository's title convention, or a Conventional Commit title when no
  convention exists;
- the issue it closes;
- before/after evidence;
- scope and preserved invariants;
- deletion and overengineering summary;
- guardrail and negative-fixture proof;
- gross additions, deletions, and net change;
- exact verification commands and results;
- residual risk; and
- rollout applicability.

Merge only a green exact head. After merge, read back the merge commit and issue
state. Reconcile the parent checklist explicitly.

## 13. Separate merge evidence from production evidence

Do not deploy cleanup automatically. Deploy only when the change affects live
behavior or runtime configuration and its acceptance criteria require rollout.

When rollout is required, verify the immutable merged artifact and the named live
outcome:

- image tag and digest;
- release revision;
- desired/updated/available/ready replicas;
- container readiness and restart counts;
- migrations and one-off jobs;
- health endpoints;
- the specific fixed behavior;
- negative routes, refusals, or alerts; and
- logs or metrics for the original failure signature.

Define rollback criteria before rollout. For irreversible schema, data, or
configuration transitions, define and test a forward-recovery path rather than
claiming rollback is available.

A green CI run or merged commit is not production evidence. A dry run, shadow
database, or target governance proposal is not completed live state.

## 14. Clean up the cleanup

After merge:

- remove clean temporary worktrees;
- prune local branches whose pull requests merged;
- preserve unrelated or unpublished work in named stashes;
- remove regenerable build artifacts and temporary evidence files;
- keep the primary checkout clean and equal to the intended remote branch; and
- report reclaimed disk space when cleanup was material.

Never destroy a worktree before inspecting tracked changes, untracked files,
ignored files, stashes, branch/PR state, and active process working directories.

## A reusable issue template

```markdown
## Outcome

State the simpler end state and the user/operator benefit.

## Reproduced evidence

Show the current failure, duplication, reachability, or drift.

## Scope

List exact surfaces and write sets.

## Preserved invariants

List behavior, contracts, security boundaries, and data allowlists.

## Non-goals

Name tempting adjacent rewrites and speculative machinery that are excluded.

## Guardrail

Name the existing enforcement that proves the simpler state. If recurrence
warrants a new or changed guard, name the canonical local/CI command and the
negative fixture it must reject. Never run the fixture through live production.

## Acceptance evidence

List exact tests, builds, rendered artifacts, reviews, and live readbacks.

## Rollout

State whether deployment is required and why.
```

## A reusable pull-request checklist

```markdown
- [ ] Premise reproduced before editing
- [ ] Current consumers and trust boundaries traced
- [ ] Superseded paths, flags, types, tests, and docs deleted
- [ ] Public/API/security invariants preserved
- [ ] Narrow guardrail added, or existing enforcement proved sufficient
- [ ] New/changed guard has a negative fixture, or marked not applicable
- [ ] Overengineering deletion pass recorded
- [ ] Deliberately unbuilt functionality recorded
- [ ] Gross additions, deletions, and net lines explained
- [ ] Focused and affected-system checks passed
- [ ] Exact PR head independently reviewed when risk/scope warrants it
- [ ] Exact-head CI green and base unchanged
- [ ] Merge commit and issue state read back
- [ ] Required production behavior verified, or deployment marked inapplicable
- [ ] Temporary worktrees and artifacts cleaned
```

## A copy-paste prompt for coding agents

```text
De-slop this repository through small, deletion-first, evidence-backed slices.

Match actions to my authorization. If I ask for a plan, approach, review, audit,
or issue text, remain read-only. Do not edit, write to a forge, merge, deploy, or
delete cleanup artifacts unless I explicitly authorize that implementation step.

First read all repository instructions and current-state docs, fetch the target
branch, confirm the exact clean head, and inventory relevant issues, callers,
write sets, dependencies, security boundaries, public contracts, and drift.

For each slice:
- reproduce or disprove the premise before editing;
- prefer hard cutover and deletion over adapters, aliases, compatibility
  re-exports, shadow paths, or new frameworks;
- preserve behavior, authority boundaries, public DTO allowlists, secrets, and
  unrelated user work;
- add or reuse a narrow programmatic guardrail only when the failure can
  materially recur and existing enforcement cannot catch it; otherwise record
  why the existing compiler/build/test proof is sufficient;
- when adding or changing a guard, add a negative fixture proving the canonical
  local/CI production-input check catches it, never a live production mutation;
- list every introduced concept and its current consumer or named threat;
- delete speculative or duplicate machinery and record what was deliberately
  not built;
- measure gross additions, deletions, net lines, and relevant runtime/CI cost;
- run focused checks continuously, then every affected test/type/lint/format/
  build/schema/config/manifest contract;
- create a small pull request following the repository's title convention, with
  before/after evidence, deletion summary, guardrail proof, residual risk, and
  exact commands;
- independently review broad, high-risk, security-boundary, public-contract,
  CI-policy, or deployment changes; use normal repository review for narrow
  low-risk deletions;
- merge only green exact-head changes and read back the merge and issue state;
- deploy only when live behavior or runtime configuration changed and acceptance
  explicitly requires it, then verify the named live behavior; and
- clean up temporary worktrees and artifacts.

Do not add generic plugin systems, arbitrary size/clone/test-file gates, a broad
AI-slop classifier, whole-repository mutation testing, automatic deletion of
static-analysis findings, global warning suppression, or unrelated architecture
rewrites.

Continue safe disjoint work while CI runs. Stop only for a genuine authority,
credential, protected-branch, external-review, or materially ambiguous product/
security blocker, and report the smallest human action required.
```

## Closing principle

The best cleanup leaves less code to understand, fewer ways to express the same
state, stronger proof at the actual failure boundary, and a smaller operational
story. Delete boldly—but only after the evidence tells you exactly what is safe.
