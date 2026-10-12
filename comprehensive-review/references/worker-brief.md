# Worker brief template

Write one shared context file and one brief per worker. Keep briefs
self-contained: a worker has none of the supervisor's conversation.

## Shared context file

```text
Target: <owner/repo> PR #<n> "<title>" by <author>, head <sha>, base <branch/sha>.
Checkout (read-only): <path>. Scope: `git -C <path> diff <merge-base>...HEAD`
  [only this PR's delta for stacks: `git diff <parent-branch>...HEAD`].
Description: <path to saved PR body>.
Existing review state: <path to saved threads/bot findings>; verify, don't repeat.
Prior findings to re-verify: <list, with ids or file:line>.
Related code outside the diff: <paths> (consumers, contracts, schemas).
Repository rules: <path to AGENTS.md / CLAUDE.md / CONTRIBUTING>.
Test notes: <how to run focused tests; required services; deps installed?>.
```

## Per-worker brief

```text
Read <shared context file> first.

Do NOT modify files in the checkout. You may run read-only commands and tests
that write only build output. Scratch files go under <scratch dir>.

Lens: <one lens from the catalog>. Answer specifically:
1. <question>
2. <question>
3. <question>

Output, terse, at most <N> words:
- Findings ordered P0, P1, P2, P3, nit. Each: file:line at head <sha>,
  one-sentence defect, concrete failure scenario (inputs/state -> wrong result),
  minimal fix, and VERIFIED (traced or reproduced) or SUSPECTED.
- A status line for each prior finding listed in the context: fixed, partially
  fixed, still open, or not applicable, with evidence.
- What you could not check and why.
Drop anything you cannot support with code or data.
```

## Running workers

- Native subagents: one per brief, in parallel, each told to read its brief.
- Other model CLIs: run in the background, sandboxed read-only, with output
  captured to a file. See [local worker CLIs](worker-clis.md) for discovery and
  per-tool flags.
- No parallel workers available: run each lens yourself in sequence, starting
  each from the brief alone rather than from your accumulated notes.

Give every worker a bounded runtime. When one fails to authenticate, returns
nothing, or times out, record the gap and continue; do not invent its result.
