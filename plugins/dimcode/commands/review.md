---
description: Run a DimCode code review against local git state
argument-hint: '[--wait|--background] [--base <ref>] [--scope auto|working-tree|branch]'
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(dimcode:*), Bash(git:*), AskUserQuestion
---

Run a DimCode review of the local git state.

Raw slash-command arguments:
`$ARGUMENTS`

Core constraint:
- This command is review-only.
- Do not fix issues, apply patches, or suggest that you are about to make changes.
- Your only job is to run the review and return DimCode's output verbatim to the user.

Scope rules:
- `--scope working-tree` (or no `--base`): review uncommitted work — staged, unstaged, and untracked files.
- `--base <ref>` or `--scope branch`: review the branch diff `<base>...HEAD` (default base: the repository's default branch).
- `--scope auto` or nothing: working tree if it is dirty, otherwise branch diff.

Execution mode rules:
- If the raw arguments include `--wait`, do not ask. Run the review in the foreground.
- If the raw arguments include `--background`, do not ask. Run the review in a Claude background task.
- Otherwise, estimate the review size before asking:
  - For working-tree review, inspect `git status --short --untracked-files=all`, `git diff --shortstat --cached`, and `git diff --shortstat`.
  - For base-branch review, use `git diff --shortstat <base>...HEAD`.
  - Treat untracked files as reviewable work even when `git diff --shortstat` is empty.
  - Only conclude there is nothing to review when the relevant status/diff is empty. When in doubt, run the review.
  - Recommend waiting only when the review is clearly tiny (roughly 1-2 files). In every other case, including unclear size, recommend background.
- Then use `AskUserQuestion` exactly once with two options, putting the recommended option first and suffixing its label with `(Recommended)`:
  - `Wait for results`
  - `Run in background`

Running the review:
- Write the review brief below to a temp file with the `Write` tool (dimcode only accepts stdin from a pipe — heredocs and `< file` redirects fail with `Error: stdin is empty`). Fill in the scope line from the rules above:

```
You are acting as a strict senior code reviewer. This is a READ-ONLY review: do not modify, create, or delete any files.

Review scope: <working tree (staged + unstaged + untracked) | diff of <base>...HEAD>.
Discover the changes yourself with git (git status, git diff, git diff <base>...HEAD) and read the surrounding code as needed.

Report:
- Findings ordered by severity (critical, major, minor), each with file path and line number, a one-sentence defect statement, and a concrete failure scenario.
- Only real issues: correctness bugs, security problems, data loss, broken edge cases. Skip style nits unless they change behavior.
- If there are no findings, say so explicitly and note any residual risk in one or two lines.
```

- Then, from the repository root, pipe it in:

```bash
cat /path/to/review-brief.txt | dimcode exec
```

Foreground flow:
- Run the command above and return its stdout verbatim, exactly as-is.
- Do not paraphrase, summarize, or add commentary before or after it.
- Do not fix any issues mentioned in the review output.

Background flow:
- Launch the same `cat ... | dimcode exec` command with `Bash` and `run_in_background: true`, description "DimCode review".
- Do not wait for completion in this turn.
- After launching, tell the user: "DimCode review started in the background. Check `/dimcode:status` for progress and `/dimcode:result` when it finishes."
