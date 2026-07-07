---
name: dimcode-result-handling
description: Internal guidance for presenting DimCode output back to the user
user-invocable: false
---

# DimCode Result Handling

When a dimcode run returns output:

- Present DimCode's final answer verbatim. If the stream includes a reasoning preamble before the answer, you may trim the preamble, but never alter the answer itself.
- For review output, present findings first, ordered by severity. Use file paths and line numbers exactly as DimCode reports them.
- Preserve evidence boundaries: if DimCode marked something as an inference, uncertainty, or open question, keep that distinction.
- If there are no findings, say so explicitly and keep the residual-risk note brief.
- If DimCode made edits, say so explicitly and list the touched files when the output names them.
- Do not turn a failed or incomplete DimCode run into a Claude-side implementation attempt. Report the failure and stop.
- If DimCode was never successfully invoked, do not generate a substitute answer at all.
- CRITICAL: After presenting review findings, STOP. Do not make any code changes. Do not fix any issues. You MUST explicitly ask the user which issues, if any, they want fixed before touching a single file. Auto-applying fixes from a review is strictly forbidden, even if the fix is obvious.
- If the run failed, include the most actionable stderr lines (e.g. the stale-lock error) and stop there instead of guessing.
- If dimcode is missing or unauthenticated, direct the user to `/dimcode:setup` and do not improvise alternate auth flows.
