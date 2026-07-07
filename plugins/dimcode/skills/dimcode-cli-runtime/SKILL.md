---
name: dimcode-cli-runtime
description: Internal helper contract for driving the local dimcode CLI from Claude Code
user-invocable: false
---

# DimCode CLI Runtime

Use this skill only inside the `dimcode:dimcode-rescue` subagent and the `dimcode` plugin commands.

## Core invocation

Always run dimcode from the repository root (dimcode sessions are scoped to the working directory — `dimcode session list` only shows sessions started in the current cwd).

Fresh task — pass the prompt on stdin **through a pipe**. dimcode only reads stdin when it is a pipe: heredocs (`<<'EOF'`) and file redirects (`< file`) both fail with `Error: stdin is empty`, because the shell presents those as file descriptors, not pipes. This applies in foreground and background alike.

For a short single-line prompt:

```bash
printf '%s' 'task text here' | dimcode exec
```

For anything multiline or containing quotes, write the prompt to a temp file with the Write tool first, then pipe it:

```bash
cat /path/to/prompt.txt | dimcode exec
```

This `Write + cat |` form is also the only reliable way to run dimcode with `run_in_background: true`.

Resume the most recent session in this repository:

```bash
cat /path/to/prompt.txt | dimcode exec resume --last
```

Resume a specific session:

```bash
cat /path/to/prompt.txt | dimcode exec resume <session-id>
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
