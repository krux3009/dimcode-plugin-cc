---
name: dimcode-cli-runtime
description: Internal helper contract for driving the local dimcode CLI from Claude Code
user-invocable: false
---

# DimCode CLI Runtime

Use this skill only inside the `dimcode:dimcode-rescue` subagent and the `dimcode` plugin commands.

## Core invocation

Always run dimcode from the repository root (dimcode sessions are scoped to the working directory — `dimcode session list` only shows sessions started in the current cwd).

Fresh task — pass the prompt on stdin via a quoted heredoc. Never inline the task text as a shell argument; heredoc avoids all quoting and multiline problems:

```bash
dimcode exec <<'DIMCODE_TASK'
<task text here>
DIMCODE_TASK
```

Resume the most recent session in this repository:

```bash
dimcode exec resume --last <<'DIMCODE_TASK'
<follow-up text here>
DIMCODE_TASK
```

Resume a specific session:

```bash
dimcode exec resume <session-id> <<'DIMCODE_TASK'
<follow-up text here>
DIMCODE_TASK
```

## Stale-lock recovery (known dimcode quirk, v0.2.21)

A finished `dimcode exec` run can leave its session locked. `exec resume` then fails with:

```
Error: Session <id> is held by another process: cli-<pid>@<host>
```

Recovery sequence when resume fails with that error:

1. Run `dimcode session list` (read-only, safe). Sessions are listed newest-first as `<id>\t<status>\t<title>`.
2. `dimcode exec resume <newest-id>` with the same heredoc prompt.
3. If that id is also locked, run `dimcode session show <locked-id>` — this forks the locked session into a new unlocked one and prints `Session: <fork-id>`. Then `dimcode exec resume <fork-id>`. The fork keeps the full conversation context.

## Hard rules

- Do NOT pass `--model` to `dimcode exec`. It is broken in dimcode 0.2.21 (dumps a raw error body). The model is whatever `dimcode model` has configured as default (currently `dimcode-api-oauth/arcship-5.5`). If the user wants a different model, tell them to change it with `dimcode model` themselves.
- Do NOT run `dimcode session export` or `dimcode session show` casually — both FORK the session they touch (fork-on-read), polluting the session list and shifting what `--last` points to. Use them only in the lock-recovery sequence above or when the user explicitly asks for a transcript.
- `--image <path>` and `--trace[=/abs/path]` are the only known-safe `exec` flags; pass them through only when the user asked for them.
- dimcode output streams the model's reasoning before the final answer. Return it as-is; presentation rules live in the `dimcode-result-handling` skill.
- One task = one `exec` invocation, plus at most the lock-recovery fallback. No other Bash activity.
