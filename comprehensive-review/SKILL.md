---
name: comprehensive-review
description: Supervise a multi-reviewer comprehensive review of a pull request, stack, merged change, or local diff, then consolidate verified findings into one deliverable and optionally fix them. Works in any agent harness that can run parallel workers (subagents, other model CLIs, or sequential self-passes). Use when asked for a comprehensive, deep, swarm, or final review, a rereview after new commits, or a post-merge review that may need production evidence. Do not use for a quick single-file check or a narrow question about one function.
license: MIT
metadata:
  author: paulbalaji
  version: "1.0.0"
---

# Comprehensive review

You are the **supervisor**. You plan the review, brief independent
**workers**, verify what they report, and own the final deliverable. Workers
can be anything your harness can run in parallel: native subagents, another
model's CLI, or—if nothing else exists—your own sequential passes with a fresh
context per lens. Nothing here assumes a particular vendor or tool name.

The output is one consolidated, verified review. Raw worker output is evidence,
never the deliverable.

## 1. Pin the target and gather state

For pull requests, follow `exact-head-pr-review` when it is installed: pin the
repository, PR number, head SHA, base, and merge base, and refresh them before
any write. Otherwise record the same ledger yourself.

Also collect, before briefing anyone:

- the change description and any linked design or incident context;
- existing reviews, inline threads, author replies, and bot findings;
- your own earlier review rounds on this target and their open findings;
- check status, separating real failures from infrastructure, quota, or flaky
  environment failures;
- for stacks or follow-ups, the base each PR targets and which PRs interact.

Create an isolated read-only checkout (worktree or clone) at the pinned head.
Note whether dependencies are installed and whether tests need services such as
Docker or a database.

## 2. Choose lenses

Pick two to four orthogonal lenses that fit the change, plus one generalist
pass from a different model family when available. Overlapping lenses waste
time; uncovered risk areas are the real failure. See
[the lens catalog](references/lenses.md).

Scale to the change: a 30-line fix may need only your own read and one
independent pass; a stateful protocol change or a stack deserves the full set.

## 3. Brief workers

Give each worker a self-contained brief. Workers do not share your context, so
the brief must carry everything they need. Use
[the brief template](references/worker-brief.md).

- Write the brief to a file and pass the path or the file contents. Inline
  prompts with backticks or `$()` break under shell expansion; use quoted
  heredocs (`<<'EOF'`) when writing files from a shell.
- Put shared context in one file that every brief references.
- State the scope boundary exactly (for example, the merge-base diff for one PR
  of a stack) and the prior findings to re-verify.
- Require read-only behaviour. Only the supervisor writes code or forge state.
- Require a fixed output shape: severity, file:line at the pinned head, defect,
  concrete failure scenario, minimal fix, and `VERIFIED` or `SUSPECTED`, under
  a word cap.

Run workers in the background with their output captured to files. Do not
claim or predict a worker's result before it arrives. Treat an empty, failed,
timed-out, or unauthenticated worker as a coverage gap to report, not as a
clean result.

## 4. Work in parallel, without duplicating

While workers run, do targeted reading the briefs do not cover: CI triage, the
riskiest hunk, contract or schema semantics the change depends on, and the
anchors you will need for inline comments. Do not redo a worker's lens.

## 5. Consolidate and verify

Merge the reports into one finding list.

1. Deduplicate by root cause, not wording. Two lenses finding the same defect
   is corroboration.
2. Verify every P0–P2 yourself against the pinned code: read the path, or
   reproduce it with a small test or script. Downgrade or drop what you cannot
   support.
3. Resolve disagreements between workers with evidence (arithmetic, a trace, a
   test), not by vote. Say which claim you rejected and why.
4. Calibrate severity by real exposure: defaults versus optional config,
   production usage, and whether the failure is silent or loud.
5. Reconcile with existing review state. Do not repeat findings already raised
   and still accurate. Mention when an earlier bot or human finding is still
   valid at the current head.
6. Separate pre-existing issues from regressions introduced by the change.

## 6. Add production evidence when the change is live

For merged or deployed changes, the most valuable review input is often
runtime data. With read-only access, build a rollout timeline (which build ran
when) and compare resources, request rates, error rates, health signals, logs,
and database state across those windows. Never print credentials. Bound every
query. Mark inferences as inferences. See
[production evidence](references/production-evidence.md).

## 7. Deliver

Unless the user asks otherwise:

- Post one consolidated review: a short body (verdict, what was checked,
  findings by severity, out-of-scope notes) plus inline comments on lines that
  are in the diff. Confirm every anchor before posting.
- Choose approve or comment from the evidence and the user's convention. Do
  not request changes unless the user asks for that state.
- Keep the body short enough to read in about a minute. Findings carry the
  detail; skip praise, process narration, and per-round changelogs.
- Resolve only your own threads that you verified as fixed, each with a brief
  reply naming the fixing commit. Never resolve other people's threads.
- Edit a PR description only when you own the PR or are asked to.

Then report back to the user: verdict first, the decisive findings, links, and
what was not verified.

## 8. Fix mode (only when authorized)

When asked to fix findings, push to the existing PR, or open a follow-up:

- Fix only verified findings within the change's original goal. Defer
  speculative hardening and anything that needs a design decision, and say so.
- Write or extend a test that fails without each fix, then confirm it fails
  with the fix reverted (a quick mutation check) before keeping it.
- Run the project's tests, lint, and formatting the way its CI does. Respect
  commit hooks; never bypass them.
- Have an independent worker review your own diff before pushing. Treat its
  findings exactly like any other worker's.
- Re-check the target before pushing, so you neither overwrite someone else's
  new commits nor review a stale head.
- One follow-up PR per request unless told otherwise; describe it with the
  evidence and the remaining deferred items.

## Pitfalls

- A notification that a worker finished is not user approval.
- Long-running test suites and port-forwards die silently; check that the
  evidence actually completed.
- Pre-commit hooks and package installs can rewrite local configuration;
  verify and restore it after committing.
- A fix that restricts behaviour can freeze adjacent code paths (for example, a
  gate that applies to sources that cannot satisfy it). Ask which callers reach
  the new condition before trusting a green suite.
- Clean up worktrees, temporary files, and background processes you started.
