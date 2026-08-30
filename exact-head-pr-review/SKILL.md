---
name: exact-head-pr-review
description: Review, rereview, comment on, approve, or merge a GitHub pull request without target drift or stale-head evidence. Use when the task names a PR and requires current checks, comments, review threads, an exact-commit verdict, or an authorized GitHub write. Do not use for an unhosted local diff or ordinary implementation that does not require a PR verdict.
license: MIT
metadata:
  author: paulbalaji
  version: "1.0.0"
---

# Review a pull request at its exact head

Produce a verdict that names the repository, pull request, immutable head, and
evidence actually reviewed. Keep code analysis separate from GitHub mutation:
`review` is read-only unless the user also authorizes a comment, review, merge,
thread resolution, or other forge write.

## Pin the target

Before analysis, read repository-local instructions and build a target ledger:

- repository owner/name and pull-request number;
- base branch, base SHA, head branch, and `headRefOid`;
- true merge base and isolated worktree or checkout;
- current PR state, draft status, mergeability, and author;
- checks, reviews, conversation comments, inline comments, and review threads;
- requested final action and the exact API/CLI target for any authorized write.

Fetch the PR head explicitly. Do not infer the target from a similarly named
branch, local checkout, issue number, pasted review, or a previous turn.

## Reconcile live review state

Treat existing findings, bot output, resolved threads, and author replies as
leads rather than truth.

- Verify each finding against the current head and actual control flow.
- A resolved thread is not proof that code changed.
- Do not repeat a current, correctly addressed, or superseded finding.
- Distinguish code failures from infrastructure, cancellation, quota, flaky
  provider, or missing-environment evidence.
- Treat incomplete or unavailable independent analysis as a coverage limit, not
  validation.

If the head changes, refresh the ledger and review the new delta. Earlier tests,
threads, and approvals do not automatically apply to the new commit.

## Review the change

Start with the true merge-base diff, then trace affected callers and contracts.
Prioritize correctness, security and authority boundaries, data exposure,
compatibility, migrations, generated artifacts, performance regressions, and
missing negative cases. Use repository tools and focused tests before expanding
to broader checks.

Lead the report with findings ordered by severity. Attach findings to changed
lines when the forge supports it; keep cross-file or outside-diff concerns in
the review body. If there are no blocking findings, say so directly and name
remaining test or environment limits.

Use independent review when the change is broad, security-sensitive, difficult
to reproduce, or explicitly requests it. Ordinary narrow reviews do not need
delegation ceremony.

## Guard every GitHub write

Immediately before commenting, reviewing, resolving threads, approving, or
merging, refresh:

- repository and pull-request number;
- open/closed/merged state;
- exact `headRefOid` and base SHA;
- checks and mergeability;
- reviews, author replies, and unresolved threads; and
- the requested action.

Fail closed if the ledger, refreshed PR, endpoint, payload commit, or intended
action disagree. Submit one consolidated review rather than leaking drafts or
incremental partial verdicts. Match the user's requested review state and the
repository convention; never infer approval, `REQUEST_CHANGES`, thread
resolution, or merge authority from a request to inspect code.

Admin or branch-protection bypass is a separate explicit authorization. Even
when authorized, require the user's stated readiness conditions—normally green
exact-head checks and resolved material findings—rather than using bypass to
turn missing evidence into success.

## Read back the result

After a GitHub write, query the forge again and report the durable coordinates:

- review/comment ID and state;
- submitted commit SHA and inline-comment locations;
- current unresolved-thread state;
- merge commit and final PR state, when merged; and
- issue or queue state when the task includes them.

Do not call a drafted payload, queued command, CLI exit, or stale browser view a
completed review or merge. Finish by confirming the local checkout was not
silently redirected or left with unrelated changes.

## Stopping conditions

Stop the write and preserve the analysis when:

- the target or requested action is ambiguous;
- the PR moved and the new delta has not been reviewed;
- required checks or external reviews remain genuinely pending;
- the PR merged or closed before the requested write; or
- authorization does not cover the required review, bypass, or merge action.

Report the smallest human or external action needed, while continuing any safe
read-only or disjoint review work.
