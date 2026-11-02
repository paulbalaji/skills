# Finding and running local agent CLIs as workers

Other model families make the best independent workers. This guide covers
finding the CLIs installed on the machine, checking they work, and running them
headless and read-only. Flags drift between releases: confirm each one with the
tool's `--help` before relying on it.

## 1. Discover what is installed

Check PATH first, then inspect likely install directories only for missing
workers. Avoid dumping configuration or entire home-directory listings.

```bash
for c in claude codex kimi muse opencode cursor-agent gemini qwen aider goose amp crush; do
  type -a "$c" 2>/dev/null
done
```

**Do not assume aliases and functions are loaded in the worker shell.**
`type -a` shows what the current shell can resolve. Check the user's shell
definition when a known alias is missing, without sourcing unknown startup
commands. Resolve each worker to an absolute binary path and identify the flags
an alias adds. For example, a `glm` alias may select a Codex provider profile.

Model routing may live in the CLI's config rather than a separate binary.
Profile layouts vary by version; inspect `--help` and list only profile names
and non-secret routing metadata. Do not dump configs or auth files. Preserve an
explicitly requested model; different aliases may still use the same model.

## 2. Check that each worker can actually run

Run a cheap smoke prompt with a short timeout, or the tool's auth-status
command (for example `codex login status`). Confirm invocation, flags, working
directory, and sandbox errors before diagnosing an authentication problem.
Common failures:

- expired or revoked login (HTTP 401, "refresh token revoked"): the user must
  log in again; record the gap and continue without that worker;
- missing API key or provider config;
- a sandbox that blocks the reads or test commands the brief needs;
- some models return an empty final message; treat that as a failed run.

Never paste credentials into a brief or a command line visible in logs.

## 3. Invoke headless and read-only

Each worker gets the brief file, the checkout as its working directory, extra
read-only directories it needs, and an output file. Prefer the tool's
read-only or plan mode, and confirm what it actually restricts. Plan mode, an
agent name, and omitting an auto-approval flag are not filesystem or forge-write
sandboxes. Restrict tools/connectors as needed; a read-only filesystem sandbox
does not itself block remote API mutations. Do not reuse aliases or profiles
that bypass approvals or sandboxes.

| CLI | Headless invocation (verify with `--help`) | Controls and limits to verify | Output capture |
|---|---|---|---|
| `codex` | `codex exec -C <checkout> - < brief.txt` | `-s read-only`; `-p <profile>` selects routing. `--add-dir` grants additional writable directories, not read-only access | `-o <file>` writes the final message |
| `claude` | `claude -p "$(cat brief.txt)"` run from `<checkout>` | `--permission-mode plan`; narrow available tools with `--tools` where supported; check inherited MCP tools | `--output-format text`, redirect stdout |
| `kimi` | `kimi -p "$(cat brief.txt)"` from `<checkout>` | verify whether `--plan` exists and enforces the needed restrictions | `--output-format text`, redirect stdout |
| `muse` | `muse exec --prompt-file brief.txt --workspace <checkout>` | verify a read-only permission profile with `muse sandbox` | `--json` or stdout |
| `opencode` | `opencode run "$(cat brief.txt)" -m <provider/model>` | inspect permissions of the selected `--agent`; its name alone proves nothing | `--format json` or stdout |
| `cursor-agent` | `cursor-agent -p "$(cat brief.txt)" --model <model>` | verify supported tool restrictions; omitting `--force` is insufficient isolation | `--output-format text`, redirect stdout |

Notes:

- Prefer stdin or a prompt-file option for long briefs. If a CLI only takes a
  prompt argument, `"$(cat brief.txt)"` passes its contents as one argument;
  backticks and `$()` inside that file are not recursively executed. Avoid
  `eval`, unquoted substitution, or embedding the brief in shell source. Quoted
  heredocs (`<<'EOF'`) write literal brief files safely.
- Some workers need write access only for build output (for example a Rust
  `target/` directory or a test database container). Give them a scratch copy
  or a dedicated build directory instead of loosening the sandbox on the shared
  checkout.

## 4. Supervise workers and collect final results

Prefer bounded foreground calls when the harness kills children on parent
exit. Background execution requires a session the supervisor can keep alive
and poll. `nohup` alone does not guarantee survival in an agent harness.

```bash
mkdir -p "$REVIEW_SCRATCH"
"$REVIEW_CODEX_BIN" exec -s read-only -C "$REVIEW_CHECKOUT" \
  -o "$REVIEW_SCRATCH/codex.md" - \
  < "$REVIEW_SCRATCH/brief-generalist.txt" \
  > "$REVIEW_SCRATCH/codex.log" 2>&1 &
review_worker_pid=$!
# Keep this supervisor session alive; enforce a deadline and collect exit status.
```

- One output file and one log per worker; the log is where auth and sandbox
  errors show up.
- Poll completion with a bounded timeout, then collect the exit status and
  final report. Non-empty logs or partial output do not establish completion.
- Track worker name, model, reviewed SHA, process/session ID, deadline, and final
  report path. A completed generalist report cannot stand in for missing lenses.
- Running several workers against the same account can hit rate limits; stagger
  them or cap concurrency.
- When a worker exits without output, read its log, record the cause, and
  decide whether to retry once or proceed without it. Do not reconstruct what it
  "would have said".

## 5. Clean up

Kill workers you started that are still running once you have what you need,
and remove disposable scratch files. Preserve final reports and raw evidence
needed for the handoff, naming their location; never stop unrelated sessions.
