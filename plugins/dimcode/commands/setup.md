---
description: Check whether the local dimcode CLI is ready to use from Claude Code
argument-hint: ''
allowed-tools: Bash(dimcode:*), Bash(command:*)
---

Check the local dimcode installation, in order:

```bash
command -v dimcode
dimcode version
dimcode auth status
dimcode model list
```

Report the results as a short checklist:

- **CLI**: found on PATH or not (with the path). If missing, tell the user to install the DimCode app + CLI from its official source — do not guess at an install command.
- **Auth**: authenticated or not. If not, tell the user to run `! dimcode auth login` (the `!` prefix runs it interactively in this session).
- **Models**: list the available models. Note that the plugin always uses the configured default model; switching is done outside the plugin with `dimcode model`.

If everything passes, say the plugin is ready and list the available commands: `/dimcode:rescue`, `/dimcode:review`, `/dimcode:status`, `/dimcode:result`.
