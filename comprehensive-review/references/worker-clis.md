# Finding and running local agent CLIs as workers

Other model families make the best independent workers. This guide covers
finding the CLIs installed on the machine, checking they work, and running them
headless and read-only. Flags drift between releases: confirm each one with the
tool's `--help` before relying on it.

## 1. Discover what is installed

Search the usual install locations rather than assuming:

```bash
for c in claude codex kimi muse opencode cursor-agent gemini qwen aider goose amp crush; do
  type -a "$c" 2>/dev/null
done
ls ~/.local/bin ~/.*/bin 2>/dev/null   # per-tool install dirs, e.g. ~/.kimi-code/bin
ls ~/.nvm/versions/node/*/bin 2>/dev/null   # npm-installed CLIs under nvm
```

**Aliases and shell functions do not exist in the non-interactive shells most
harnesses spawn.** `type -a` shows whether a name is an alias, a function, or a
binary. Resolve each worker to an absolute binary path, and spell out any flags
an alias would have added. For example, a `glm` alias that expands to
`codex --profile glm ...` becomes the codex binary plus `-p glm`.

Model routing often lives in the CLI's own config rather than in a separate
binary. Codex profiles are sections or files under `~/.codex/` (for example
`glm.config.toml`), selected with `-p <profile>`. List them before choosing.

## 2. Check that each worker can actually run

Run a cheap smoke prompt with a short timeout, or the tool's auth-status
command (for example `codex login status`). Common failures:

- expired or revoked login (HTTP 401, "refresh token revoked"): the user must
  log in again; record the gap and continue without that worker;
- missing API key or provider config;
- a sandbox that blocks the reads or test commands the brief needs;
- some models return an empty final message; treat that as a failed run.

Never paste credentials into a brief or a command line visible in logs.

## 3. Invoke headless and read-only

Each worker gets the brief file, the checkout as its working directory, extra
read-only directories it needs, and an output file. Prefer the tool's
read-only or plan mode. Do not reuse aliases or profiles that bypass approvals
or sandboxes.

| CLI | Headless invocation (verify with `--help`) | Read-only control | Output capture |
|---|---|---|---|
| `codex` | `codex exec "$(cat brief.txt)" -C <checkout> --skip-git-repo-check` | `-s read-only`; `-p <profile>` selects a model/provider profile; `--add-dir <dir>` | `-o <file>` writes the final message |
| `claude` | `claude -p "$(cat brief.txt)"` run from `<checkout>` | `--permission-mode plan`; `--add-dir <dir>`; `--allowedTools` to narrow | `--output-format text`, redirect stdout |
| `kimi` | `kimi -p "$(cat brief.txt)"` from `<checkout>` | `--plan`; `--add-dir <dir>`; `-m <model>` | `--output-format text`, redirect stdout |
| `muse` | `muse exec --prompt-file brief.txt --workspace <checkout>` | a read-only `--permission-profile` if configured; check `muse sandbox` | `--json` or stdout |
| `opencode` | `opencode run "$(cat brief.txt)" -m <provider/model>` | a read-only `--agent` (for example `plan`) if defined | `--format json` or stdout |
| `cursor-agent` | `cursor-agent -p "$(cat brief.txt)" --model <model>` | omit `--force` so commands are not auto-approved | `--output-format text`, redirect stdout |

Notes:

- Pass long briefs from a file. If a CLI only takes the prompt as an argument,
  use `"$(cat brief.txt)"` and keep the file free of characters the shell could
  re-expand. Quoted heredocs (`<<'EOF'`) write such files safely.
- Some workers need write access only for build output (for example a Rust
  `target/` directory or a test database container). Give them a scratch copy
  or a dedicated build directory instead of loosening the sandbox on the shared
  checkout.

## 4. Run in the background and collect

```bash
mkdir -p "$SCRATCH"
nohup "$CODEX_BIN" exec -s read-only -C "$CHECKOUT" --skip-git-repo-check \
  -o "$SCRATCH/codex.md" "$(cat "$SCRATCH/brief-generalist.txt")" \
  > "$SCRATCH/codex.log" 2>&1 &
```

- One output file and one log per worker; the log is where auth and sandbox
  errors show up.
- Wait on "output file is non-empty or the process has exited", with a bounded
  timeout, rather than sleeping a fixed time.
- Running several workers against the same account can hit rate limits; stagger
  them or cap concurrency.
- When a worker exits without output, read its log, record the cause, and
  decide whether to retry once or proceed without it. Do not reconstruct what it
  "would have said".

## 5. Clean up

Kill workers you started that are still running once you have what you need,
and remove scratch files unless the user wants them.
