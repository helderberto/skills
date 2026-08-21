---
name: grill-me
description: A relentless interview to sharpen a plan or design. Use when the user wants to stress-test a plan, pressure-test assumptions, or says "grill me" or "poke holes in this". Don't use for code review (use /code-review) or gathering initial requirements (use /spec).
---

# Grill Me

Interview the user relentlessly until you reach a shared understanding. Map the plan as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1 — <question title>**: <question body, may be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2 — <question title>**: <question body>

➡️ <your recommended answer>
```

Each answered round reshapes the tree: settled decisions push the frontier outward and unblock the questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

**Facts are your job; decisions are the user's.** If a frontier question can be answered from the codebase or environment, dispatch a subagent to find it — never ask the user for anything you could look up yourself. Don't block the round on a running lookup: only the questions downstream of it wait; ask the rest of the frontier now.

Done when the frontier is empty: every branch visited, nothing left silently assumed. Do not act until the user explicitly confirms the shared understanding — "whatever you think is best" or "you decide" is delegation, not confirmation; put the specific decision back to them.
