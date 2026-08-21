---
name: plan
effort: high
description: Turn a spec into a multi-phase implementation plan using tracer-bullet vertical slices. Use after /spec when a spec exists at `.specs/specs/<slug>.md`, or when the user asks to break work into phases or slices. Don't use without a spec, or for single-file changes with obvious scope.
argument-hint: "[slug]"
---

# Plan

Break a spec into phased vertical slices (tracer bullets).

**Interactive prompts**: present options as a numbered list and wait for the user's choice.

Output: `.specs/plans/<slug>.md`.

## Input

The argument (if provided) is: $ARGUMENTS

Use argument as `<slug>`. If empty, list specs as numbered options and wait for the user's choice.

## Workflow

### 1. Read the spec

Read `.specs/specs/<slug>.md`. If missing, list specs as numbered options and wait for the user's choice.

If `.specs/plans/<slug>.md` exists, present options and wait:

1. Overwrite existing (Recommended)
2. Pick a new name

### 2. Explore the codebase

Map architecture, patterns, integration points. Skip if codebase context exists from prior step.

**Research protocol**: codebase first, then docs. Unverifiable claims → flag as uncertain, never fabricate.

### 3. Identify durable architectural decisions

Before slicing, extract decisions that hold across all phases:

- Route structures / URL patterns
- Database schema shape
- Key data models and definitions
- Auth/authorization approach
- Third-party service boundaries

### 4. Draft vertical slices

Each phase: thin vertical slice through all layers (schema → service → API → UI → tests). Demoable alone.

**Deriving tasks from the spec:**

| Spec Section      | Becomes                                          |
| ----------------- | ------------------------------------------------ |
| New Modules       | Implement module with interface                  |
| Schema Changes    | Migration + validation                           |
| API Contracts     | Route returning shape                            |
| Navigation        | Wire component to route                          |
| User Stories      | Verify coverage; add task if missing             |
| Testing Decisions | Tests land in the phase where their module lands |
| Out of Scope      | Never create tasks for these                     |

**Within each slice, order by dependency:** schema → service → API → UI → tests. Happy paths before edge cases.

**Phase naming:** use a goal phrase answering "what can we demo when this is done?" (e.g., "Phase 1 — Revenue visible end-to-end"), not a layer name. An "and" in a phase title is a sign it's two phases.

**Blocking edges:** each phase declares which phases must complete before it can start. Default to a linear chain (each phase blocked by the previous); declare independent edges only when phases genuinely don't gate each other — two independent phases mean two `/build` invocations can work the frontier in parallel.

**Done when:** checkbox list of atomic, verifiable conditions. Each must name a test file/name, a shell command, or a file+content to verify. No prose-only conditions. Test: "Can an agent verify by reading files, running a command, or checking a test?"

**Layer-by-layer exception:** if complex schema changes underpin all modules and no story stands alone, build data foundation first, then slice vertically.

**Wide-refactor exception:** a mechanical change (rename a column, retype a shared symbol) whose blast radius fans across the codebase can't land as a vertical slice — a single edit breaks every call site at once. Sequence it as **expand → migrate → contract**: expand adds the new form beside the old (nothing breaks); call sites migrate in batches sized by blast radius (per package/directory), each batch its own phase blocked by the expand, CI green throughout because the old form still exists; contract deletes the old form in a final phase blocked by every migrate batch.

**Phase count thresholds:**

- 1 module touched → 2–3 phases max
- 2–3 modules touched → 3–5 phases max
- 4+ modules or 6+ phases → stop and present options:
  1. Split the spec (Recommended)
  2. Continue anyway

Count "modules touched" by scanning the spec's New Modules and Schema Changes sections.

Assign an agent tag to tasks where appropriate:

- `[skill:diagnose]` — tracing a bug or unexpected runtime behavior
- `[agent:test-auditor]` — writing or reviewing tests
- `[skill:code-review]` — reviewing API surfaces, interfaces, or public contracts

### 5. Quiz the user

Present breakdown (title, user stories covered, done-when per phase). Present options and wait:

1. Looks good, proceed (Recommended)
2. Merge some phases
3. Split a phase

Iterate until approved.

### 6. Save plan

Save to `.specs/plans/<slug>.md` (create dir if missing).

```markdown
# Plan: <Feature Name>
```

Use this structure for the plan body:

```markdown
## Architectural Decisions

Durable decisions that apply across all phases:

- **Key decision**: ...

---

## Phase 1 — <Goal>

**User stories**: <list from spec>

**Blocked by**: None — can start immediately <!-- later phases: "Phase 1" etc. -->

### What to build

Concise description of this vertical slice — end-to-end behavior, not layer-by-layer.

### Done when

- [ ] Atomic, testable condition
- [ ] Another testable condition

---

<!-- Repeat for each phase -->

## Out of Scope

Carried forward from spec verbatim.

## Open Questions

Gaps found in the spec needing resolution. Blank if none.
```

Print one line per phase: `Phase N — <title> (<condition summary>)`. Present options and wait:

1. Run `/build <slug>` (Recommended)
2. Run `/test <slug>`
3. Done for now

## Execution guidance

To implement this plan phase by phase, run `/build <slug>`. It handles branch creation, implementation, feedback loops, checkbox marking, and commits — one phase per invocation.

## Rules

- Phases derive from spec user stories — never invented
- Each phase must be demoable end-to-end on its own
- "Done when" must be a checkbox list of testable conditions, not prose
- **Safety valve**: if a phase has >5 "Done when" items, stop and split it into smaller phases before continuing
- Never modify the source spec content
- Carry spec's Out of Scope forward verbatim
