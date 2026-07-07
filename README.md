# dimcode-plugin-cc

Use [DimCode](https://dimcode.ai) from Claude Code: delegate coding tasks to the local `dimcode` CLI and run DimCode-powered code reviews — the same idea as OpenAI's [codex-plugin-cc](https://github.com/openai/codex-plugin-cc), rebuilt for dimcode.

Unlike the Codex plugin, there is no companion runtime or broker process: the dimcode CLI already provides one-shot execution (`dimcode exec`), session resume (`exec resume --last`), and session listing, so the plugin drives the CLI directly and uses Claude Code's own background tasks for long runs.

## Requirements

- `dimcode` CLI on PATH (tested with v0.2.21) and authenticated (`dimcode auth status` → `Authenticated`).
- The plugin always runs DimCode's configured default model (currently `arcship-5.5`); switch models with `dimcode model`, not through the plugin.

## Install

```bash
claude plugin marketplace add "/path/to/dimcode-plugin-cc"
claude plugin install dimcode@dimcode-cc
```

Then restart Claude Code (or start a new session) and run `/dimcode:setup` to verify.

## Commands

| Command | What it does |
|---|---|
| `/dimcode:rescue [task]` | Hand a debugging/implementation task to DimCode via the `dimcode-rescue` subagent. Flags: `--background`/`--wait`, `--resume`/`--fresh`. |
| `/dimcode:review` | Read-only DimCode code review of the working tree or a branch diff (`--base <ref>`, `--scope`, `--wait`/`--background`). |
| `/dimcode:status` | DimCode sessions in this repo + running background DimCode tasks. |
| `/dimcode:result [session-id]` | Final output of the last background DimCode run (or a session transcript by id). |
| `/dimcode:setup` | Check CLI presence, auth, and models. |
| `/dimcode:image <prompt>` | Generate an image with DimCode's image model (`--edit <path>` / `--enhance <path> --operation <op>` variants; `--out`/`--out-dir` to choose destination, defaults to the current directory). |

The `dimcode:dimcode-rescue` subagent is also used proactively by Claude when a task is worth a second pair of hands.

## dimcode CLI quirks the plugin works around (v0.2.21)

- **Sessions are cwd-scoped** — `dimcode session list` only shows sessions started in the current directory, so everything runs from the repo root.
- **Stale locks** — a finished `exec` can leave its session "held by another process"; the plugin recovers by resuming by explicit id, or by letting `session show` fork the locked session.
- **Fork-on-read** — `session show` and `session export` fork the session they touch, so the plugin avoids them outside lock recovery and transcript requests.
- **`exec --model` is broken** (dumps a raw error body) — the plugin never passes it.
- Prompts are passed on **stdin via heredoc** to avoid shell-quoting issues.

## Not implemented (add when needed)

- Stop-time review gate hooks (codex-plugin-cc's optional `Stop` hook) — add a `hooks/hooks.json` if you want every Claude turn gated behind a DimCode review.
- Session transfer (Claude → DimCode thread) — dimcode CLI has no session-import command to receive one.
- DimCode's remaining media commands (`video`/`speech`/`music`/`audio`/`asset`) — add alongside `/dimcode:image` if your provider gains those models.

## License

Apache-2.0. Command and skill prompt structure adapted from [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) (Apache-2.0); see NOTICE.
