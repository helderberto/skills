---
name: tdd
effort: high
description: Test-driven development. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

See [tests.md](references/tests.md) for examples and [mocking.md](references/mocking.md) for mocking guidelines.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Anti-Pattern: Tautological Tests

The assertion recomputes the expected value the same way the code does — `expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand via the same logic, a constant asserted equal to itself. The test passes by construction and can never disagree with the code.

**Expected values must come from an independent source of truth**: a known-good literal, a worked example, the spec. See [tests.md](references/tests.md) for an example.

## Workflow

### 0. Discover the stack

The TDD cycle is universal; the commands are not. Before the first test, discover how _this_ repository tests and use its commands for every RED and GREEN:

- Build system: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Makefile`
- Checked-in wrappers first: `./gradlew`, `./mvnw`, `make test`, repo scripts — never a globally installed tool the repo doesn't use
- The focused-single-test command vs the full-suite command — run focused during the loop, full suite once before completion
- Where tests live, how they're named, what patterns neighboring tests follow
- CI workflows show the commands that actually gate merges

Never assume `npm test`.

### 1. Planning

When exploring the codebase, read `CONTEXT.md` (if it exists) so that test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Before writing any test, write down the seams under test and confirm them with the user:

Ask: "What's the public interface, and which seams should we test?"

**No test is written at an unconfirmed seam.**

Before writing any code:

- [ ] Confirm the seams under test with the user
- [ ] Confirm with user which behaviors to test (prioritize)
- [ ] Identify opportunities for deep modules (small interface, deep implementation) — run the `/codebase-design` skill for the vocabulary and the testability checks
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get user approval on the plan

**You can't test everything.** Agreeing the seams up front is how testing effort lands on critical paths and complex logic, not every possible edge case.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

This is your tracer bullet - proves the path works end-to-end.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### 4. Refactor

After all tests pass, look for [refactor candidates](references/refactoring.md):

- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

**Never refactor while RED.** Get to GREEN first.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```
