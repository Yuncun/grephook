---
description: Set up ast-checker in the current project
---

You are setting up ast-checker for the user's project. Follow these steps in order. Be concise — at each step, just state what you're doing and the result.

## Step 1: Find the project root (and refuse if not in a project)

Try `git rev-parse --show-toplevel` first. If that errors, look for any of these project markers in the current directory: `pyproject.toml`, `package.json`, `Cargo.toml`, `go.mod`, `pom.xml`, `Gemfile`, `composer.json`, `build.gradle`, `build.gradle.kts`, `sgconfig.yml`.

**If none of those exist, abort with this message:**

> No project detected in `<cwd>`. ast-checker scaffolds files (`sgconfig.yml`, `.ast-grep/`) into a project root — running this in a workspace root or random directory would pollute it. Either:
> - `cd` into a project directory and re-run, OR
> - if this IS your project root, create a marker first (e.g. `git init`, `npm init -y`, etc.) so the tool knows it's a project.

Stop. Don't create any files.

If a project root was found (via git or marker), continue.

## Step 2: Verify ast-grep is installed

Run `command -v ast-grep`. If not installed:
- On Mac: tell the user to run `brew install ast-grep`
- On Linux/other: suggest `cargo install ast-grep --locked` or check their package manager
- Stop here. Tell the user to install ast-grep then re-run `/harness init`.

If installed, run `ast-grep --version` to confirm and report the version.

## Step 3: Create sgconfig.yml at project root if missing

Check if `<project-root>/sgconfig.yml` exists. If not, create it with:

```yaml
ruleDirs:
  - .ast-grep
```

If it exists, leave it alone (the user may have customized it).

## Step 4: Create .ast-grep/ directory if missing

Run `mkdir -p <project-root>/.ast-grep`.

## Step 5: Add a starter rule if .ast-grep/ is empty

If `.ast-grep/` has no `*.yml` files, copy the starter template:

```bash
cp "${CLAUDE_PLUGIN_ROOT}/templates/no-direct-field-mutation.yml" <project-root>/.ast-grep/
```

Then read the file and surface the placeholders to the user — explicitly call out:
- The `files:` glob (currently `**/your_app/routes.py`) — they MUST change this to a real path in their project
- The pattern (`$X.prompt = $Y`) — they should change `prompt` to a field they want to protect

Tell them: "This is a TEMPLATE. It won't catch anything until you adapt it. Edit `.ast-grep/no-direct-field-mutation.yml` to match your project, then run `ast-grep scan` to verify it doesn't fire on existing code (zero false positives)."

## Step 6: Verify the setup

Run `cd <project-root> && ast-grep scan` and report the result. If the starter rule's placeholders haven't been adapted, scan should report no findings (because the file glob doesn't match anything).

## Step 7: Print a summary

Print:
- ✅ ast-grep version: ...
- ✅ sgconfig.yml: created/exists
- ✅ .ast-grep/: N rule(s) loaded
- Next step: edit `.ast-grep/no-direct-field-mutation.yml`, adapt to your project, save. Then edit any file in your project — the hook will fire.

Tell the user they can run `/harness status` anytime to check the setup, or `/harness add-rule` to browse and add more rules from the catalog.
