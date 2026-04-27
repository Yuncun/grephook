---
description: Browse the ast-grep rule catalog (no args) or install a rule from it (with name arg)
---

The user invoked `/harness rules`. The argument (if any) is a rule name to install.

## If no argument given — list the catalog

List available templates from `${CLAUDE_PLUGIN_ROOT}/templates/`:

1. Run `ls "${CLAUDE_PLUGIN_ROOT}/templates/"*.yml` to get the file list.
2. For each file, read the metadata header (lines starting with `#`) and extract:
   - The "Catches:" line (after the colon)
   - The `language:` field from the rule body
3. Print a table (use the actual extracted Catches text per file, not these examples):

| Rule | Catches | Language |
|---|---|---|
| no-direct-field-mutation | LLM mutating an entity field outside the service layer | Python |
| handler-defer-persist | Persistence calls inside handlers that should defer to accept/commit | Python |
| routes-must-use-service | Route handlers calling backend methods directly | Python |

Then print these next steps:

> To install one from the catalog: `/harness rules <name>`
>
> To write a CUSTOM rule not in this catalog (e.g., language-specific or project-specific patterns), just ask in natural language — e.g., "add an ast-grep rule to ban console.log in components." The `writing-ast-grep-rules` skill handles pattern, format, and testing.

## If a rule name argument is given — install it

1. Verify `${CLAUDE_PLUGIN_ROOT}/templates/<name>.yml` exists. If not, list available templates and stop with "rule not found, here's the catalog."

2. Find the project root: `git rev-parse --show-toplevel` (fall back to cwd).

3. Ensure `<project-root>/.ast-grep/` exists: `mkdir -p <project-root>/.ast-grep`.

4. Check if `<project-root>/.ast-grep/<name>.yml` already exists. If yes, ask the user whether to overwrite (don't proceed without confirmation).

5. Copy the template:
   ```bash
   cp "${CLAUDE_PLUGIN_ROOT}/templates/<name>.yml" "<project-root>/.ast-grep/"
   ```

6. Read the copied file. Surface the placeholders the user MUST adapt — explicitly call out:
   - The `files:` glob (which paths in their project should match)
   - The pattern (variable names, field names, function patterns)
   - The constraints (regex patterns)
   - Any `# Adapt:` instructions in the metadata header

7. Tell the user to:
   - Edit `.ast-grep/<name>.yml` to adapt the placeholders to their project
   - Run `ast-grep scan` from project root to verify zero false positives on existing code
   - Once it passes on existing code, the rule is live — the post-edit hook picks it up automatically
