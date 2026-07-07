---
description: Delegate investigation, an explicit fix request, or follow-up rescue work to the DimCode rescue subagent
argument-hint: "[--background|--wait] [--resume|--fresh] [what DimCode should investigate, solve, or continue]"
allowed-tools: Bash(dimcode:*), AskUserQuestion, Agent
---

Invoke the `dimcode:dimcode-rescue` subagent via the `Agent` tool (`subagent_type: "dimcode:dimcode-rescue"`), forwarding the raw user request as the prompt.
`dimcode:dimcode-rescue` is a subagent, not a skill — do not call `Skill(dimcode:dimcode-rescue)` (no such skill) or `Skill(dimcode:rescue)` (that re-enters this command and hangs the session).
The final user-visible response must be DimCode's output verbatim.

Raw user request:
$ARGUMENTS

Execution mode:

- If the request includes `--background`, run the `dimcode:dimcode-rescue` subagent in the background.
- If the request includes `--wait`, run the subagent in the foreground.
- If neither flag is present: default to foreground for a small, clearly bounded request; prefer background when the task looks complicated, open-ended, multi-step, or likely to keep DimCode running for a long time.
- `--background` and `--wait` are execution flags for Claude Code. Do not forward them to the subagent as part of the task text.
- If the request includes `--resume` or `--fresh`, do not ask whether to continue. The user already chose. Leave the flag in the forwarded request — the subagent handles that routing.
- Otherwise, check for a resumable DimCode thread in this repository by running `dimcode session list` from the repository root (read-only, safe).
- If it lists at least one session AND the request reads like a follow-up ("continue", "keep going", "apply the top fix", "dig deeper"), use `AskUserQuestion` exactly once with these two choices, recommended first:
  - `Continue current DimCode thread (Recommended)`
  - `Start a new DimCode thread`
- If the user chooses continue, add `--resume` before routing to the subagent. If they choose new, add `--fresh`.
- If `dimcode session list` shows no sessions, or the request is clearly a fresh task, do not ask. Route normally.

Operating rules:

- The subagent is a thin forwarder only. Return its output verbatim to the user.
- Do not paraphrase, summarize, rewrite, or add commentary before or after it.
- If `dimcode` is not on PATH or reports it is unauthenticated, stop and tell the user to run `/dimcode:setup`.
- If the user did not supply a request, ask what DimCode should investigate or fix.
