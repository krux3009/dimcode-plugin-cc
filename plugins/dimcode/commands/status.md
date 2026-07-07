---
description: Show DimCode sessions for this repository and any running background DimCode tasks
argument-hint: ''
disable-model-invocation: true
allowed-tools: Bash(dimcode:*)
---

!`dimcode session list`

Above is the output of `dimcode session list` for this repository (dimcode sessions are scoped to the working directory; newest first, format `id  status  title`).

- Render it as a compact Markdown table with columns ID, Status, Title. If it says no sessions were found, say so in one line.
- Titles containing "(fork #N)" are automatic forks dimcode creates when a locked session is inspected — same conversation content, safe to resume.
- Then check this Claude session for running or recently finished background DimCode tasks (background Bash tasks whose command invokes `dimcode`). List each with its task/shell ID and state. If one is running, remind the user they can fetch its output later with `/dimcode:result`.
- Do not include extra prose beyond the table and the background-task list.
