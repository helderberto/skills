---
name: lint
effort: low
description: Run linting and formatting checks. Use when user asks to "run linter", "/lint", "check linting", "fix lint errors", or requests code linting/formatting. Don't use for the full format/type/test pass (use /validate-code) or for installing commit-time hooks (use /setup-pre-commit).
---

# Linting

Run the project's linter/formatter — detect it, don't assume a toolchain.

## Detection

Prefer a task the project already defines, then fall back to the ecosystem tool:

| Ecosystem | Check | Fix |
|---|---|---|
| Node | `package.json` `lint` (eslint/biome) | `lint:fix` |
| Python | `ruff check` / `flake8` | `ruff check --fix` |
| Go | `golangci-lint run` / `gofmt -l` | `gofmt -w` |
| Rust | `cargo clippy` / `cargo fmt --check` | `cargo fmt` |
| else | `Makefile` / CI lint target | — |

## Workflow

1. Detect the linter (project task first, then ecosystem default)
2. Run the check command
3. For fixes: run the fix variant (only when requested)
4. Report `file:line` references for all errors

## Rules

- Prefer the project's defined task over a raw tool call
- Never invent a linter the project doesn't use

## Error Handling

- If no linter or task found → report which config/task is expected and stop
- If the linter exits with parse errors → report each file-level parse error separately with `file:line`
- If the linter config is missing → report which config is expected and stop
