# helderberto/agent-skills

[![Test Plugin Installation](https://github.com/helderberto/agent-skills/actions/workflows/test-plugin-install.yml/badge.svg)](https://github.com/helderberto/agent-skills/actions/workflows/test-plugin-install.yml)

**Personal SDLC toolbelt for AI coding agents — from spec to ship.**

A collection of skills that encode the workflows, quality gates, and engineering practices I use day-to-day. Pure Markdown, zero runtime deps, installable as a [Claude Code](https://claude.com/claude-code) plugin or copied into any agent that reads instruction files.

```
  SPEC            PLAN            BUILD            TEST             REVIEW           SHIP
 ┌──────┐       ┌──────┐        ┌──────┐         ┌──────┐         ┌──────┐         ┌──────┐
 │ Idea │ ────▶ │ Spec │ ─────▶ │ Code │ ──────▶ │ Test │ ──────▶ │  QA  │ ──────▶ │  Go  │
 │Refine│       │Slices│        │ Impl │         │Verify│         │ Gate │         │ Live │
 └──────┘       └──────┘        └──────┘         └──────┘         └──────┘         └──────┘
 /hb:spec       /hb:plan        /hb:build        /hb:test         /hb:review       /hb:ship
```

Each phase has a dedicated **workflow skill** that orchestrates the smaller toolbelt skills underneath it. Type the spine in order, or let it chain — every skill auto-routes by description. Only the outward-facing steps that push work out of your hands (`ship`, `create-pull-request`) are gated against auto-triggering, so you always pull that trigger yourself.

---

## Quick Start

<details open>
<summary><b>Claude Code (recommended)</b></summary>

Install via the marketplace:

```
/plugin marketplace add helderberto/agent-skills
/plugin install hb@helderberto-skills
```

After install, skills are available as `/hb:<skill-name>` — e.g. `/hb:spec`, `/hb:tdd`, `/hb:ship`. Most skills also auto-trigger from natural language ("review this PR", "check accessibility", etc.) based on their description.

</details>

<details>
<summary><b>Gemini CLI</b></summary>

Install as native skills:

```bash
gemini skills install https://github.com/helderberto/agent-skills.git --path skills
```

Skills are auto-discovered and routed by description. See [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md).

</details>

<details>
<summary><b>OpenCode</b></summary>

Clone and point OpenCode at the workspace — `AGENTS.md` plus the `skills/` directory drive auto-routing:

```bash
git clone https://github.com/helderberto/agent-skills.git
```

Open the project in OpenCode. The `.opencode/skills` symlink and root `AGENTS.md` are already wired. See [docs/opencode-setup.md](docs/opencode-setup.md).

</details>

<details>
<summary><b>Cursor</b></summary>

Clone the repo, then copy individual skills into `.cursor/rules/`:

```bash
git clone https://github.com/helderberto/agent-skills.git
cp agent-skills/skills/tdd/SKILL.md .cursor/rules/tdd.md
cp agent-skills/skills/code-review/SKILL.md .cursor/rules/code-review.md
```

See [docs/cursor-setup.md](docs/cursor-setup.md) for the recommended starter set.

</details>

---

<details>
<summary><b>Workflow example</b> — full SDLC walkthrough</summary>

A non-trivial feature flows through all six phases. Each workflow skill is one invocation:

```
You: /hb:spec add dark mode support
AI:  Interviews, scans the codebase, writes .specs/specs/dark-mode.md.
     Run /hb:plan dark-mode next?

You: /hb:plan dark-mode
AI:  Breaks the spec into phased vertical slices.
     Writes .specs/plans/dark-mode.md.

You: /hb:build dark-mode
AI:  Implements next incomplete phase. TDD loop, lint, type-check.
     Marks checkboxes in the plan. Offers a commit.

You: /hb:test dark-mode
AI:  Runs validation (lint, types, tests) + coverage, then verifies
     plan checkboxes against the codebase. Reports progress + blockers.

You: /hb:review
AI:  Detects what changed, runs relevant audits in order
     (code-review, a11y-audit, safe-repo, perf-audit, deps-audit, ...).
     Consolidates findings into Critical / Important / Suggestion.

You: /hb:ship
AI:  Pre-launch gate (validate-code + safe-repo --diff).
     Atomic commits, push current branch.
     /hb:ship --fast skips the gate (hotfix only).
```

For quick standalone tasks, you don't need the workflow — just describe what you want and the relevant skill triggers ("write tests for X", "audit deps", "create an ADR for Y").

</details>

---

## Skills

Skills come in two modes. **User-invoked** ones never auto-trigger (`disable-model-invocation: true`) — the outward-facing, irreversible actions you must pull the trigger on yourself (`ship`, `create-pull-request`), plus `teach`, which only ever starts by hand. Everything else is **model-invoked**: it auto-routes by description and stays callable explicitly as `/hb:<name>`. Model-invoked descriptions carry the trigger and anti-trigger clauses routing depends on; user-invoked ones keep a single what-it-does sentence, since trigger phrases are dead weight when nothing auto-routes.

Skills also carry an **effort** hint: mechanical ones (`lint`, `commit`, `prose-fix`) run at low reasoning effort, heavy ones (`architecture-audit`, `harden`, `diagnose`) at high or xhigh, and the rest at the implicit medium default. The override lasts only for the turn the skill fires — so complexity matches the task without you touching `/effort`.

### SDLC workflow

The six-phase spine. Type each to advance, or let one phase chain into the next:

| Skill | Phase | What it does |
|-------|-------|--------------|
| [`spec`](skills/spec/SKILL.md) | SPEC | Interview + codebase scan → structured spec in `.specs/specs/<slug>.md` |
| [`plan`](skills/plan/SKILL.md) | PLAN | Turn spec into multi-phase implementation plan (tracer-bullet vertical slices) |
| [`build`](skills/build/SKILL.md) | BUILD | Implement next incomplete phase of a plan with feedback loops |
| [`test`](skills/test/SKILL.md) | TEST | Validate (lint/types/tests) + coverage, and verify plan checkboxes against codebase |
| [`review`](skills/review/SKILL.md) | REVIEW | Fan out parallel reviewers (scope-detected audits + independent agent lenses), consolidate into one verdict — **optional QA pass**, not a ship gate |
| [`ship`](skills/ship/SKILL.md) | SHIP | Pre-launch gate (validate-code + safe-repo) + atomic commits + push (`--fast` to skip gate) · **user-invoked** |

### On-demand tools

Focused capabilities the agent applies automatically based on the task (all callable explicitly too). Expand a group to browse.

<details>
<summary><b>Build &amp; test</b></summary>

| Skill | What it does |
|-------|--------------|
| [`tdd`](skills/tdd/SKILL.md) | Red → green → refactor loop for any new logic |
| [`source-driven`](skills/source-driven/SKILL.md) | Implement using official docs for exact dependency versions |
| [`fortify`](skills/fortify/SKILL.md) | Split large functions, add edge-case coverage, backfill missing tests |
| [`code-simplify`](skills/code-simplify/SKILL.md) | Reduce complexity without changing behavior — clarity over cleverness (Chesterton's Fence) |
| [`e2e`](skills/e2e/SKILL.md) | Write end-to-end tests for user flows using Cypress |

</details>

<details>
<summary><b>Verify</b></summary>

| Skill | What it does |
|-------|--------------|
| [`validate-code`](skills/validate-code/SKILL.md) | Auto-fix lint, verify types, run tests |
| [`lint`](skills/lint/SKILL.md) | Run linting and formatting checks |
| [`diagnose`](skills/diagnose/SKILL.md) | Disciplined diagnosis loop for hard bugs and perf regressions |
| [`visual-validate`](skills/visual-validate/SKILL.md) | Browser-driven UI validation via Chrome DevTools or Playwright MCP |

</details>

<details>
<summary><b>Review &amp; audit</b></summary>

| Skill | What it does |
|-------|--------------|
| [`code-review`](skills/code-review/SKILL.md) | PR review against an approval standard — correctness, security, performance, a Fowler smell baseline; Critical→Nit severities |
| [`visual-review`](skills/visual-review/SKILL.md) | Render a PR diff as an annotated HTML page — each hunk linked to a design/simplification principle with a suggested rewrite |
| [`triage-review`](skills/triage-review/SKILL.md) | Triage existing PR review comments (Copilot + human), verify against code, classify Address/Skip/Optional/Discuss |
| [`a11y-audit`](skills/a11y-audit/SKILL.md) | Accessibility compliance audit (WCAG) |
| [`i18n`](skills/i18n/SKILL.md) | Find hardcoded strings, check translation coverage |
| [`perf-audit`](skills/perf-audit/SKILL.md) | Frontend bundle size and performance audit |
| [`deps-audit`](skills/deps-audit/SKILL.md) | Check dependencies for vulnerabilities (npm/pnpm/yarn/pip/go/cargo) |
| [`safe-repo`](skills/safe-repo/SKILL.md) | Sensitive data scan; `--diff` mode for in-flight changes |
| [`harden`](skills/harden/SKILL.md) | Proactive security hardening at trust boundaries (OWASP-style) |

</details>

<details>
<summary><b>Git &amp; release</b></summary>

| Skill | What it does |
|-------|--------------|
| [`commit`](skills/commit/SKILL.md) | Group unstaged changes into atomic commits by concern (repository style) |
| [`create-adr`](skills/create-adr/SKILL.md) | Record a 1–3 sentence Architecture Decision Record |
| [`create-pull-request`](skills/create-pull-request/SKILL.md) | Open a GitHub PR with structured body · **user-invoked** |

</details>

<details>
<summary><b>Design &amp; discovery</b></summary>

| Skill | What it does |
|-------|--------------|
| [`codebase-design`](skills/codebase-design/SKILL.md) | Shared deep-module vocabulary for designing or improving an interface |
| [`frontend-ui-engineering`](skills/frontend-ui-engineering/SKILL.md) | Front-load UI construction decisions — prop API, state placement, required states, a11y by construction |
| [`architecture-audit`](skills/architecture-audit/SKILL.md) | Surface architectural friction, propose refactors toward deep modules as RFCs |
| [`domain-modeling`](skills/domain-modeling/SKILL.md) | Build and sharpen the project's ubiquitous language and glossary |
| [`research`](skills/research/SKILL.md) | Investigate a question against primary sources; capture cited findings as Markdown |
| [`prototype`](skills/prototype/SKILL.md) | Build a throwaway prototype — terminal app or toggleable UI variations — to flesh out a design |
| [`grill-me`](skills/grill-me/SKILL.md) | Stress-test a plan or design — interview in rounds over the design-tree frontier, each question with a recommended answer |

</details>

<details>
<summary><b>Session, meta &amp; writing</b></summary>

| Skill | What it does |
|-------|--------------|
| [`handoff`](skills/handoff/SKILL.md) | Compact the current conversation into a handoff doc for a fresh agent |
| [`teach`](skills/teach/SKILL.md) | Stateful teaching workspace — lessons, references, learning records tied to a mission · **user-invoked** |
| [`explain-code`](skills/explain-code/SKILL.md) | Explain code with visual diagrams and analogies |
| [`create-skill`](skills/create-skill/SKILL.md) | Author a new skill with proper structure |
| [`setup-pre-commit`](skills/setup-pre-commit/SKILL.md) | Configure Husky + lint-staged for commit-time gates |
| [`prose-fix`](skills/prose-fix/SKILL.md) | Fix typos, dashes, formatting in markdown |
| [`revise`](skills/revise/SKILL.md) | Structurally edit and improve article drafts |

</details>

---

## Agents

Subagents the skills fan out to for independent, read-only perspectives (e.g. `/hb:review` spawns lenses, `/hb:plan` tags steps). Bundled with the plugin, so installing `hb` ships them too. **Claude Code only** — other agents consume `skills/` and ignore this directory.

| Agent | Purpose |
|-------|---------|
| [`security-auditor`](agents/security-auditor.md) | Threat modeling, vulnerability assessment |
| [`test-auditor`](agents/test-auditor.md) | Test effectiveness beyond coverage |
| [`frontend-architect`](agents/frontend-architect.md) | Component design, deep modules, structural health |
| [`parity-check`](agents/parity-check.md) | Audit code migrations for missing functionality |
| [`git-detective`](agents/git-detective.md) | Investigate git history and trace changes |
| [`learner`](agents/learner.md) | Capture insights into CLAUDE.md |

---

## Structure

```
agent-skills/
├── .claude-plugin/      Plugin manifest + marketplace entry (Claude Code)
├── .opencode/skills →   Symlink to skills/ for OpenCode discovery
├── skills/              one folder per skill, each with SKILL.md
├── agents/              subagent definitions (Claude Code only)
├── docs/                Skill anatomy + per-agent setup guides
├── AGENTS.md            Intent → skill mapping (drives OpenCode auto-routing)
├── CONTRIBUTING.md      How to contribute new skills or improvements
├── LICENSE              MIT
└── README.md
```

### Artifact convention

Workflow skills write structured artifacts to `.specs/`:

- `.specs/specs/<slug>.md` — specs from `/hb:spec`
- `.specs/plans/<slug>.md` — phased plans from `/hb:plan`

The `.specs/` directory is local-first. Add it to `.gitignore` if you prefer specs as scratch space, or commit it if you want specs as versioned project documentation.

---

## Contributing

PRs welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for workflow, conventions, and where to start.

---

## License

[MIT](LICENSE) © Helder Burato Berto
