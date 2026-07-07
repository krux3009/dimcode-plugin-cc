---
description: Show the final output of a finished background DimCode run in this session
argument-hint: '[session-id]'
disable-model-invocation: true
allowed-tools: Bash(dimcode:*), BashOutput
---

Fetch and present the result of the most recent DimCode run.

Primary path (no argument given):
- Look for background DimCode tasks in this Claude session (background Bash tasks whose command invokes `dimcode`).
- Retrieve the stdout of the most recent one with `BashOutput` and present DimCode's final answer verbatim, per the `dimcode-result-handling` skill. If it is still running, say so and show what has streamed so far.
- If there is no background DimCode task in this session, say so and point at the fallback below only if the user gave a session id.

Fallback path (explicit session id argument):
- Run `dimcode session export <session-id>` from the repository root and present the last assistant message from the exported JSON, plus the token usage from `metadata.usage`.
- Warn the user in one line that `session export` forks the session in dimcode (a duplicate "(fork #N)" entry will appear in `/dimcode:status`) — this is a dimcode quirk, not data loss.

Do not summarize or condense DimCode's findings. Preserve file paths, line numbers, verdicts, and next steps exactly as reported. Do not fix anything mentioned in the output.
