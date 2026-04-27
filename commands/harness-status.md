---
description: Show grephook status for the current project
---

You are showing a status report for grephook. Be concise — gather the data and present a clean table.

## Gather

1. **Project root:** Run `git rev-parse --show-toplevel` in the current directory. Fall back to cwd if not a git repo.

2. **ast-grep installation:**
   - Run `command -v ast-grep` to find the binary path
   - If found, run `ast-grep --version` for the version
   - If not found, mark as missing

3. **Config file:** Check if `<project-root>/sgconfig.yml` exists. Note its contents.

4. **Rules directory:** Check if `<project-root>/.ast-grep/` exists. List `*.yml` files in it. For each, extract the `id:` field with `grep -E "^id:" <file>` to get the rule ID.

5. **Last scan result:** Run `cd <project-root> && ast-grep scan` (capture exit code and a brief summary of any findings).

## Present

Format as a table:

| Field | Status |
|---|---|
| Project root | `<path>` |
| ast-grep | ✅ vX.Y.Z at `<path>` ── or ── ❌ not installed |
| sgconfig.yml | ✅ present ── or ── ❌ missing (run `/harness init`) |
| Rules loaded | N rules: `<id1>`, `<id2>`, ... ── or ── ❌ no rules in `.ast-grep/` |
| Current scan | ✅ clean ── or ── ❌ N findings |

If anything is missing, suggest the next action:
- ast-grep missing → `brew install ast-grep`
- sgconfig.yml missing → `/harness init`
- No rules → `/harness add-rule` to browse catalog
- Findings → list them briefly so the user knows what to fix
