# ast-grep rule catalog

Architecture rules that catch common LLM shortcuts. Each rule is in its own file with metadata explaining when to use it. Copy the ones you want into your project's `.ast-grep/` directory.

| Rule | Catches | Language | Added |
|---|---|---|---|
| [no-direct-field-mutation](no-direct-field-mutation.yml) | LLM mutating an entity field outside the service layer | Python | 2026-04-25 |
| [handler-defer-persist](handler-defer-persist.yml) | Persistence calls inside handlers that should defer to accept/commit | Python | 2026-04-25 |
| [routes-must-use-service](routes-must-use-service.yml) | Route handlers calling backend methods directly instead of via service layer | Python | 2026-04-25 |

## How to use

1. Pick the rules that match patterns in your project.

2. Copy them into your project's `.ast-grep/` directory:

   ```bash
   mkdir -p .ast-grep
   cp <plugin-path>/templates/no-direct-field-mutation.yml .ast-grep/
   ```

3. Make sure your project has an `sgconfig.yml` at its root pointing at the rule directory:

   ```yaml
   ruleDirs:
     - .ast-grep
   ```

4. **Adapt the rule.** Every rule has placeholders (e.g. `**/your_app/routes.py`, field names, function-name regex). Edit them to match your project.

5. **Verify zero false positives** before committing. Run `ast-grep scan` against your existing code — if it fires, either the rule is wrong or your code already has the violation. Fix one before adding the rule.

6. The post-edit hook (from this plugin) will pick up the new rules automatically — it scans whatever is in `.ast-grep/`.

## Easier: use the slash command

Instead of copying files manually:

```
/harness add-rule              # lists all available templates
/harness add-rule <name>       # copies the named template into your project's .ast-grep/
```

## Adding your own rules

Same pattern: one YAML file per rule, metadata header at the top explaining when to use it. Each rule should pass on existing code (zero false positives) before you commit it. ast-grep supports 30+ languages via tree-sitter — if you have rules in non-Python languages, the catalog welcomes them.

## Why per-file?

Each file is a self-contained unit you can copy or skip. The metadata header tells you what each rule does and when it's appropriate, so you can browse the catalog without reading the YAML. Adding more rules over time doesn't bloat any single file.
