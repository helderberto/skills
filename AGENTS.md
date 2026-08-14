# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Gemini CLI, OpenCode, Copilot, etc.) when working with this repository.

## Repository Overview

`helderberto/agent-skills` is a personal SDLC toolbelt for AI coding agents. Skills are packaged Markdown instructions that extend agent capabilities across the full lifecycle — from spec to ship.

## Project Structure

```
skills/         → Core skills (one folder per skill, each with SKILL.md)
agents/         → Subagent definitions the skills fan out to (Claude Code only)
docs/           → Setup guides per agent (Cursor, Gemini CLI, OpenCode, Copilot)
.claude-plugin/ → Plugin manifest + marketplace entry (Claude Code)
```

## Integration Model

Different agents discover skills through different mechanisms. This repo supports all of them through one source of truth — the `skills/` directory.

| Agent       | Discovery mechanism                                           |
| ----------- | ------------------------------------------------------------- |
| Claude Code | Plugin marketplace (`/plugin install hb@helderberto-skills`)  |
| Gemini CLI  | `gemini skills install` from this repo                        |
| OpenCode    | Auto-routes via this `AGENTS.md` + the `skill` tool           |
| Cursor      | Copy skills into `.cursor/rules/` or reference via `.cursorrules` |

See [docs/](./docs/) for per-agent setup instructions.

## OpenCode Integration

OpenCode uses a **skill-driven execution model** — the agent reads this file plus the `skills/` directory and auto-routes user intent to the right skill.

### Core Rules

- If a task matches a skill, you MUST invoke it
- Skills live at `skills/<skill-name>/SKILL.md`
- Never implement directly when a skill applies
- Always follow the skill instructions exactly — do not partially apply them

### Intent → Skill Mapping

| User intent                             | Skill to invoke                                |
| --------------------------------------- | ---------------------------------------------- |
| New feature with unclear requirements   | `spec` → `plan` → `build`                      |
| Implement next phase of a plan          | `build`                                        |
| Verify work + check plan progress       | `test`                                         |
| Test-first development                  | `tdd`                                          |
| Implement using official docs           | `source-driven`                                |
| Research a question against sources     | `research`                                     |
| Bug, error, unexpected behavior         | `diagnose`                                     |
| Backfill tests / split large functions  | `fortify`                                      |
| Simplify code without changing behavior | `code-simplify`                                |
| Design or improve a module's interface  | `codebase-design`                              |
| Architectural friction in codebase      | `architecture-audit`                           |
| Prototype a design (terminal app or UI) | `prototype`                                    |
| Explain code or codebase area           | `explain-code`                                 |
| Code review on a PR                     | `code-review`                                  |
| Visual/HTML PR review (annotated diff)  | `visual-review`                                |
| Triage existing PR review comments (what to fix) | `triage-review`                        |
| Reply to PR review comment              | `pr-reply`                                     |
| Check accessibility                     | `a11y-audit`                                   |
| Check i18n coverage                     | `i18n`                                         |
| Audit bundle size / performance         | `perf-audit`                                   |
| Audit dependencies for CVEs             | `deps-audit`                                   |
| Check for sensitive data in repo        | `safe-repo`                                    |
| Run linter / formatter                  | `lint`                                         |
| Run the test suite                      | `testing`                                      |
| Validate code (lint + types + tests)    | `validate-code`                                |
| Write end-to-end tests (Cypress)        | `e2e`                                          |
| Commit changes (atomic, grouped by concern) | `commit`                                    |
| Commit and push                         | `ship`                                         |
| Create a pull request                   | `create-pull-request`                          |
| Record an architectural decision        | `create-adr`                                   |
| Pin down domain terminology / glossary  | `domain-modeling`                              |
| Set up pre-commit hooks                 | `setup-pre-commit`                             |
| Stress-test a plan or design            | `grill-me`                                     |
| Fix prose / typos in markdown           | `prose-fix`                                    |
| Restructure an article draft            | `revise`                                       |
| Hand off mid-session work to a fresh agent | `handoff`                                   |
| Author a new skill                      | `create-skill`                                 |

### Lifecycle Mapping (Implicit Flow)

When the user has a non-trivial task, follow this flow even without explicit commands:

- **SPEC** → `spec` (interview-driven), with `grill-me` to stress-test
- **PLAN** → `plan` (spec to vertical-slice phases), with `architecture-audit` or `codebase-design` as needed
- **BUILD** → `build` (one phase per invocation), driven by `tdd` and `source-driven`
- **TEST** → `test` (validate-code + changed-line coverage + plan vs codebase), with `diagnose` on failure
- **REVIEW** → `review` (orchestrates the audits below as parallel reviewers plus independent agent lenses; `code-review` for a single in-thread lens; `visual-review` for an annotated HTML diff report; `triage-review` to sort existing reviewer comments), covering `a11y-audit`, `i18n`, `perf-audit`, `deps-audit`, `safe-repo`, `harden`
- **SHIP** → `validate-code` → `ship` → `create-pull-request`

## Conventions

- Every skill lives in `skills/<name>/SKILL.md` (or `<name>/SKILL.md` if flat at root — see structure section)
- YAML frontmatter requires `name` and `description`
- Description starts with what the skill does, followed by trigger conditions ("Use when...")
- Optional `effort` frontmatter (`low`/`medium`/`high`/`xhigh`) overrides reasoning effort for the turn the skill fires — set it only when the task deviates from the `medium` default (`model:` is ignored for skills)
- Supporting files split into subfolders: `references/` (long-form docs loaded on demand), `scripts/` (deterministic helpers), `assets/` (templates/icons used in output). Split when `SKILL.md` exceeds ~150 lines
- Bump `.claude-plugin/plugin.json` `version` (semver) on every user-visible change to a skill so plugin consumers can track updates
- Create an annotated git tag `vX.Y.Z` matching the bumped version (`git tag -a vX.Y.Z -m "..."`) summarizing what changed since the previous tag, then `git push --tags`

## Boundaries

This is a documentation-only project. It contains no application code, no runtime dependencies, no build step. Skills are pure Markdown.
