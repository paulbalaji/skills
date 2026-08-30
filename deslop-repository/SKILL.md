---
name: deslop-repository
description: Reduce accidental repository complexity through evidence-led deletion, hard cutovers, narrow negative guardrails, exact-head review, and rollout verification. Use when asked to de-slop or remove dead, duplicate, legacy, or overengineered paths and make cleanup durable in an existing codebase. Do not use for ordinary feature development, cosmetic formatting, or a small routine refactor.
license: MIT
metadata:
  author: paulbalaji
  version: "1.0.0"
---

# De-slop a repository

Make the repository easier to prove, change, review, and operate. Reduce ways
to express the same state without trading away current behavior, security
boundaries, public contracts, or unrelated user work.

Match actions to the user's authorization. If asked for a plan, approach,
review, audit, or issue text, remain read-only. Repository edits, forge writes,
merges, deployments, and cleanup deletions require explicit implementation
authorization. Never infer live-production mutation authority from permission
to edit or merge code.

For a multi-slice program, issue-writing task, or unfamiliar repository, read
[the operating guide](references/operating-guide.md) before planning. For a
narrow cleanup, use the workflow below and read the relevant guide section when
the task involves CI policy, security boundaries, pull requests, deployment, or
worktree removal.

## Establish the evidence boundary

Before editing:

1. Read repository-local instructions and current-state documents completely.
2. Fetch the target branch and record its exact immutable commit.
3. Confirm checkout status and preserve unrelated edits, worktrees, and stashes.
4. Trace current entry points, callers, dynamic loading, generated surfaces,
   public contracts, schemas, configuration, and trust boundaries.
5. Reproduce or disprove the reported waste or failure.
6. Identify the exact command, artifact, or live readback that will prove the
   desired outcome.

For a suspected public API deletion, also inventory generated clients and
external consumers, inspect a representative traffic window when available,
and obtain the repository's required owner/product evidence. Static reachability
alone cannot prove an externally invoked route is dead.

Treat current code, rendered artifacts, and live state as evidence. Treat old
plans and audit SHAs as potentially stale until refreshed. Apply Git, worktree,
and pull-request instructions only when the repository uses those mechanisms;
otherwise use the equivalent version-control and change-review concepts.

## Build a slice ledger

For work spanning multiple changes, record for each slice:

- premise to reproduce;
- current consumer, named threat, or demonstrated failure;
- dependencies and overlapping write sets;
- behavior and boundaries that must remain true;
- negative fixture and mechanical guard;
- acceptance evidence; and
- base or production drift risk.

Parallelize only disjoint write sets. Serialize slices sharing contracts,
presentation types, schemas, generated files, or migration state. Re-evaluate
the order when current evidence invalidates the initial plan.

## Implement deletion-first

- Prefer hard cutover and deletion when callers are controlled.
- Remove the superseded code, flags, types, exports, configuration, tests, and
  documentation in the same slice.
- Reuse a stable existing concept before adding another service, layer, route,
  table, registry, or framework.
- Add compatibility only for a named external consumer. Record its owner,
  deadline, and deletion condition.
- Fail fast on invalid configuration and unexpected state. Do not use defensive
  fallbacks that make incorrect state look valid.
- Preserve authority distinctions, allowlists, server-bound payloads, secret
  handling, idempotency, and terminal-state evidence.

If the premise is false, close or rewrite the task with evidence. Do not invent
a code change to justify it.

## Add or prove the regression-shaped guard

Choose the narrowest guard that matches the demonstrated failure, such as:

- entry-point-aware reachability for dead production surface;
- authored runtime import-cycle detection;
- strict schema parsing for unknown configuration;
- schema/type derivation for duplicate contracts;
- structured manifest assertions for deployment topology;
- scoped warning failure with an explicit allowlist;
- merge-base-relative fingerprints for baseline laundering;
- changed executable line/branch evidence for critical modules; or
- an active-surface scan for a retired environment or command.

Every slice needs durable mechanical proof, but it need not create a new tool.
Reuse an existing compiler, build, route manifest, schema check, or reachability
gate when it directly catches the regression. Add a new guard only when current
enforcement cannot.

Add a negative fixture that introduces the forbidden condition and prove the
canonical local/CI command used to build, start, validate, or render production
inputs rejects it. Never execute a fixture through live production mutation. A
happy-path test does not prove a gate. Keep the guard structured, deterministic,
bounded, and explicit about generated, historical, vendor, and test-support
exclusions.

Do not introduce arbitrary file-size limits, broad clone gates, mandatory
test-file-change rules, whole-repository mutation testing, generic AI-slop
classifiers, or global warning suppression.

## Perform the overengineering pass

Before finalizing, list every introduced helper, type, layer, service, flag,
route, table, metric, and workflow. For each one:

1. name its current consumer or threat;
2. show why an existing seam cannot serve it;
3. remove duplicate or speculative machinery; and
4. record material functionality deliberately not built.

Automatically reported dead code is a lead, not deletion proof. Establish
runtime, build, migration, operational, and dynamic-loading reachability first.

When consolidating cross-language or generated response contracts, explicitly
compare field allowlists, aliases, omitted versus nullable values, error
envelopes, dates, decimals, large integers, unknown-field behavior, and
public-safe projection boundaries.

## Account for the change

Measure gross additions, gross deletions, and net lines. Explain where retained
lines go: production logic, negative fixtures, generated lockfiles, CI
enforcement, documentation, or a concrete migration bridge.

Measure relevant runtime and CI costs before redesigning to satisfy a budget.
Use actual bundle/image size, test wall time and memory, latency, query shape,
or rendered resource count. Do not evade a budget with broad exclusions.

## Verify and review

Use an expanding verification ladder:

1. negative fixture;
2. focused unit or contract tests;
3. affected package tests;
4. lint, format, and type checks;
5. build, bundle, schema, config, and manifest checks;
6. affected integration or end-to-end suites; and
7. repository-required full checks.

Record exact commands and honest outcomes. Skipped, flaky, interrupted, or
environment-limited checks remain unverified.

Use a small pull request following the repository's title convention (prefer a
Conventional Commit title when none exists). Include before/after evidence,
preserved invariants, deletions, guardrail proof, line economics, exact commands,
residual risk, and rollout applicability.

Independently review the immutable head for correctness, security and public
contract changes, caller blast radius, unnecessary abstractions, dead code,
test gaps, CI permissions, dependency provenance, and narrative/diff mismatch.
Refresh the base, head, checks, reviews, and unresolved threads immediately
before merge. Merge only a green exact head, then read back the merge and issue
state.

## Distinguish merged from live

Deploy only when live behavior or runtime configuration changed and acceptance
requires rollout. When deployment is required, verify the immutable artifact,
release revision, readiness and restarts, migrations, health endpoints, the
named fixed behavior, negative/refusal paths, and the original failure signature
in logs or metrics. Define rollback criteria beforehand. For irreversible
migrations or configuration transitions, define a forward-recovery path instead
of pretending rollback is available.

A merge, green CI run, dry run, shadow database, or pending governance target is
not production completion.

## Clean up safely

Remove temporary worktrees, branches, and generated artifacts after merge, but
first inspect tracked changes, untracked and ignored files, stashes, branch/PR
state, and active process working directories. Preserve unpublished work
recoverably. Finish with the primary checkout clean and equal to its intended
remote branch.
