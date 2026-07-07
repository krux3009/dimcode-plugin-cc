---
description: Generate, edit, or enhance an image with DimCode's image model
argument-hint: '[--edit <path>|--enhance <path> --operation <op>] [--out <path>|--out-dir <dir>] <prompt>'
allowed-tools: Bash(dimcode:*), Read
---

Run a DimCode image job from the raw request below.

Raw request:
`$ARGUMENTS`

Mode selection:
- Default: generate — `dimcode image generate --prompt <prompt> (--out <path>|--out-dir <dir>)`
- `--edit <path>` present: edit — `dimcode image edit --prompt <prompt> --image <path> (--out ...)`
- `--enhance <path>` present: enhance — `dimcode image enhance --image <path> --operation <operation> (--out ...)`. Enhance takes no prompt; it requires `--operation`, and valid operation names are not documented — pass the user's operation through verbatim, and if dimcode rejects it, show dimcode's usage/error output so the user can pick a valid one.

Output path:
- If the user gave `--out` or `--out-dir`, pass it through unchanged.
- Otherwise default to `--out-dir .` (dimcode picks the filename). One of the two is mandatory — never call `dimcode image` without it.

Prompt handling:
- Everything in the request that is not a recognized flag is the prompt text. Pass it as a single quoted `--prompt` argument.
- Do not rewrite or "improve" the user's prompt.

Model selection:
- First try without `--provider`/`--model` (uses dimcode's configured modality default).
- If dimcode fails with `missing_selection: No default provider/model configured for image.generate`, run `dimcode model list`, pick the image-capable model (name contains "image", e.g. `dimcode-api-oauth/gpt-image-2`), and retry once with explicit `--provider <provider> --model <model>` split from that entry at the `/`.
- If no image model exists, stop and tell the user their connected dimcode provider has no image model.
- After a successful explicit-flag retry, mention once that the user can make it the default with `dimcode modality set image.generate --provider <provider> --model <model>` (note the dotted capability name — plain `image` is rejected).

After the run:
- Report the saved file path.
- Display the result by `Read`ing the image file.
- If the job failed, show dimcode's error output as-is; if it suggests auth problems, point the user to `/dimcode:setup`.
