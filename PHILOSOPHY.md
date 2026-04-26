# Philosophy

## The framework

Borrowing Martin Fowler's [harness engineering](https://martinfowler.com/articles/harness-engineering.html): every coding-agent system has **guides** (feedforward — informs the agent before it acts) and **sensors** (feedback — observes after the agent acts).

This plugin is a sensor. After every Claude edit, it runs ast-grep on the file. If a rule fires, the hook returns JSON `decision: block` and Claude self-corrects in the same turn.

## Why a sensor (not a guide)

CLAUDE.md is advisory. Skills are advisory. Even Karpathy's LLM Wiki, which dramatically improves context, can be ignored mid-turn. None of these are deterministic — Claude may or may not honor them.

The post-edit hook is one of the few mechanisms in Claude Code that's **guaranteed to be seen by the model.** When the hook returns a `decision: block`, Claude has no choice — the block is in its context, the next turn must address it.

That makes hooks the right place for hard constraints. CLAUDE.md is the right place for guidance. Use both, but don't conflate them.

## Why ast-grep specifically

We want to catch when Claude writes a specific code shape — not when it writes "bad" code in general. That's a structural problem, not a semantic one.

Three options for catching code shapes:

| Tool | Speed | Determinism | Fits "ban this code shape"? |
|---|---|---|---|
| Text/regex | Fast | Brittle | No — misses formatting variations, matches strings/comments |
| LLM judge | Slow ($) | Variable | Yes but expensive per check |
| AST tools (ast-grep, Semgrep) | Fast | Deterministic | **Yes — designed for this** |

ast-grep specifically:
- Multi-language (30+ via tree-sitter — Python, JS, Go, Rust, Java, Kotlin, Ruby, etc.)
- Patterns look like the language (`pattern: $X.foo($$$)`)
- Single binary, fast, free
- Active focus on AI-generated rules ([their team blogs about it](https://ast-grep.github.io/blog/ast-grep-agent.html))

Semgrep would also work but is positioned as SAST (security). ast-grep's positioning matches the use case better.

## Why one plugin, one job

Splitting per-checker is intentional. Each checker has different install conventions, language coverage, and conflict potential. Bundling them creates a "dispatcher" that grows over time and tries to support every ecosystem — fragile and harder to maintain.

Better: each plugin owns one checker. Plugins coexist via Claude Code's hook merging — they all fire on the same edit, each independently. No coordination needed.

The future companion plugins (`ruff-checker`, `eslint-checker`, etc.) follow the same pattern: minimal hook, single tool, no dispatcher. All listed under the same `yuncun` marketplace.

## What this is NOT

- Not a linter. It runs ast-grep; you bring the rules.
- Not a replacement for code review. Linters catch mechanical violations. Architectural judgment ("you bandaided this") still needs a human or LLM judge.
- Not a silver bullet. Bad rules (false positives) train the LLM to suppress them. Test every rule on existing code before adding it — zero false positives is the bar.
