# Development Workflow

Practical guide for using **opsx-role-spec** with the OpenSpec workflow.

For: developers working in an AI coding assistant (Claude Code, Gemini CLI, OpenAI Codex CLI). Not for the AI — the AI reads the SKILL.md files directly.

## Table of Contents

- [Tool Map](#tool-map)
- [Scenario 1: New Feature — Idea to Archive](#scenario-1-new-feature--idea-to-archive)
- [Scenario 2: Small Fix or Bug](#scenario-2-small-fix-or-bug)
- [Scenario 3: Just Thinking](#scenario-3-just-thinking)
- [Quick Reference](#quick-reference)
- [Antipatterns](#antipatterns)
- [First Run](#first-run)

---

## Tool Map

| Stage | Command | What it does |
|-------|---------|--------------|
| Think before starting | `/opsx:explore` | Thinking partner — diagrams, questions, no code written |
| Formalize requirements | `/analyst <description>` | Brownfield read of `openspec/specs/`, outputs `[ADDED]/[MODIFIED]/[REMOVED]` deltas |
| Create change with all artifacts | `/opsx:propose <name>` | Generates `proposal.md` + `specs/` + `design.md` + `tasks.md` |
| Refine plan / decompose | `/planner` | Writes to `design.md` + `tasks.md` in checkbox format |
| Implement ONE task | `/coder` or `/opsx:apply` | Reads scenarios, writes code, runs Verify, ticks checkbox |
| Review code against contract | `/reviewer` | Maps WHEN/THEN → diff, writes to Decisions log |
| Verify behavior | `/tester` | Test cases seeded from WHEN/THEN, runs unit + e2e |
| Finalize | `/opsx:archive <name>` | Promotes deltas into `openspec/specs/`, moves change to archive |

---

## Scenario 1: New Feature — Idea to Archive

End-to-end example: "add order cancellation with status `CANCELLED` and a cancellation reason."

### Step 0 — (optional) Think out loud

If the idea is still fuzzy:

```
/opsx:explore I want to add order cancellation, not sure where to store the reason
```

The AI will sketch a status state machine, read existing specs, and ask clarifying questions. No code is written.

**Output:** scoped understanding in your head; sometimes drafts in an existing `proposal.md` if a change is already active.

### Step 1 — Formalize requirements

```
/analyst add ability to cancel an order with status CANCELLED and a cancellation reason
```

The AI will:
- read `openspec/specs/orders/spec.md` to avoid duplicating existing requirements
- read `openspec/config.yaml` for project conventions (multi-tenancy, RBAC, validation)
- output requirements as `[ADDED] / [MODIFIED] / [REMOVED]` with citations to existing spec lines
- hand off: "Ready for `/opsx:propose order-cancellation`"

**Output:** a requirements list and the name for the upcoming change.

### Step 2 — Create the change with all artifacts

```
/opsx:propose order-cancellation
```

The AI will:
- create `openspec/changes/order-cancellation/`
- generate `proposal.md`, `specs/orders/spec-delta.md` with WHEN/THEN scenarios, `design.md`, `tasks.md`
- show a coverage check (what is covered by scenarios, what is not)

**Output:** full artifact set on disk. From this point the change is active and negative triggers in the skills engage.

### Step 3 — (optional) Refine the plan

If `tasks.md` from `/opsx:propose` is too coarse or missing `Verify:` steps:

```
/planner refine the plan for order-cancellation
```

The AI rewrites `tasks.md` in checkbox format with `What/Verify/Depends/Scenario` and adds tradeoffs to `design.md` → `## Decisions log`.

Usually skipped — `/opsx:propose` already produces a workable plan. Come here only if the decomposition is off.

### Step 4 — Implement tasks

Two options:

**Option A — autonomous via OpenSpec:**

```
/opsx:apply order-cancellation
```

Walks `tasks.md` top to bottom: reads scenarios → implements → runs Verify → checks `- [x]`. Stops on blockers and ambiguities.

**Option B — task by task manually:**

```
/coder implement task 1 from order-cancellation
```

After task 1 — mandatory review (step 5) before task 2. Coder does not start task 2 on its own.

**When to use which:**
- `/opsx:apply` — for linear features without surprises, simple CRUDs
- `/coder` one by one — when each task is non-trivial and you want review/test after each

### Step 5 — Review after each task

```
/reviewer
```

The AI will:
- read `git diff` against the change's base
- read `specs/orders/spec-delta.md` for scenarios
- build the mandatory **Contract coverage** table (every WHEN/THEN → line of code)
- write non-obvious decisions to `design.md` → `## Decisions log` prefixed `[review YYYY-MM-DD]`
- output `APPROVE` / `REQUEST CHANGES` / `APPROVE WITH COMMENTS`

**On `REQUEST CHANGES`** — go back to coder:

```
/coder fix R-3, R-5
```

Coder addresses those IDs only, does not expand scope. Then `/reviewer` again.

**On `APPROVE`** or `APPROVE WITH COMMENTS` → go to tester.

### Step 6 — Test after approve

```
/tester
```

The AI will:
- seed test cases from WHEN/THEN scenarios in `spec.md` (one table row per scenario, with ID `S-N`)
- run unit tests for logic, e2e tests for UI flows
- output a table with PASS/FAIL/manual per scenario
- `PASS` → "Ready for `/opsx:archive order-cancellation`"
- `FAIL` → "Fix bug 2 — re-enter coder"

**On FAIL:** `/coder fix bug 2`, then `/tester` again. Do not go back to analyst — test failures are about code, not requirements.

### Step 7 — Archive

When tester gives PASS and all checkboxes in `tasks.md` are `[x]`:

```
/opsx:archive order-cancellation
```

The AI will:
- verify all artifacts are complete
- propose syncing deltas from `changes/order-cancellation/specs/` into the canonical `openspec/specs/orders/spec.md` (accept — this is spec promotion)
- move the change to `openspec/changes/archive/YYYY-MM-DD-order-cancellation/`

From this point, the next `/analyst` reads `specs/orders/spec.md` with cancellation already in the baseline.

---

## Scenario 2: Small Fix or Bug

For small changes (bug, typo, constant update) — the short path:

```
/opsx:propose fix-order-totals-rounding
# AI asks a couple of questions and creates artifacts

/opsx:apply fix-order-totals-rounding
# single task, implemented automatically

/reviewer
# usually APPROVE with one minor issue

/tester
# one or two test rows from the spec

/opsx:archive fix-order-totals-rounding
```

That's **5 commands** for a bugfix — the minimum that preserves contract discipline.

### Without OpenSpec entirely

Acceptable only for:
- typos in string literals
- constant updates that don't change behavior
- style / formatting with no logic change

```
/coder fix rounding in src/lib/api/orders.ts line 234
```

Coder in standalone mode skips spec reads, makes the change, runs Verify. **No review/test/archive — at your own risk.** Any functional change should go through OpenSpec.

---

## Scenario 3: Just Thinking

```
/opsx:explore
```

Describe your idea freely. Multiple iterations are fine. When it crystallizes — move to `/analyst` or go straight to `/opsx:propose`.

In explore mode the AI **does not write code**, but can create OpenSpec artifacts (proposal, design, specs) if you ask.

---

## Quick Reference

```
Idea is fuzzy              → /opsx:explore
Idea is clear, no change   → /analyst → /opsx:propose <name>
Change exists, bad plan    → /planner
Change exists, ready to code → /opsx:apply  OR  /coder
After each task            → /reviewer → /tester
All [x] and PASS           → /opsx:archive
REQUEST CHANGES            → /coder fix R-N
FAIL                       → /coder fix bug N
```

### Re-entry rules

| Situation | Re-enter at | NOT at |
|-----------|-------------|--------|
| `REQUEST CHANGES` from reviewer | `/coder fix R-N` | analyst, planner |
| `FAIL` from tester | `/coder fix bug N` | analyst, planner |
| Requirements turned out wrong | `/analyst` + edit `proposal.md` | coder |
| Architecture turned out wrong | `/planner` + edit `design.md` | coder |

Rule: feedback on code → re-enter at coder. Feedback on requirements/architecture → re-enter at the appropriate upstream role with an explicit artifact edit.

---

## Antipatterns

1. **Don't call `/coder` directly with an active change but no `tasks.md`** — it won't know which task to do. Run `/opsx:propose` first.
2. **Don't skip `/reviewer` between tasks** — this breaks the Decisions log and lets blockers through. Even on simple tasks, reviewer returns APPROVE in 30 seconds.
3. **Don't re-run `/analyst` after FAIL** — re-entry is `/coder`, not `/analyst`. Going back to analyst restarts the entire pipeline from scratch.
4. **Don't edit `openspec/specs/` by hand** — specs are promoted only via `/opsx:archive`. Manual edits leave the next `/analyst` reading inconsistent state.
5. **Don't ignore the `Verify run` in coder** — it actually runs your test/build commands. If it fails, the task is not done; don't tick `[x]` until it passes.
6. **Don't write to `design.md` Decisions log manually** — reviewer and coder write there as they work. Manual entries can confuse the format and get overwritten.
7. **Don't invoke two roles at once** — each role has a negative trigger: "Do not invoke if an active OpenSpec change exists and a different role's artifact is the next expected output." Respect it: invoke only the role whose artifact is next.

---

## First Run

To verify the full pipeline works in your project — pick a small real task from your backlog and walk it through all 7 steps of Scenario 1. Examples:

- "add a `notes` field to the order"
- "add a date filter to the client list"
- "add an `EMERGENCY` type to order priorities"

This takes ~30 minutes and exercises every joint: brownfield read, propose, scenarios, checkbox flip, Decisions log, archive. Afterwards you'll see exactly where the pipeline pinches for you.

---

## Where Decisions Live

| Decision type | File |
|--------------|------|
| Per-change tradeoffs, invariants, constraints | `openspec/changes/<name>/design.md` → `## Decisions log` |
| Project-wide tech conventions (stack, structure, naming) | `openspec/config.yaml` |
| Pipeline-meta rules (re-entry, hand-off, rule scope) | `karpathy-guidelines/SKILL.md` |
| Canonical capability scenarios | `openspec/specs/<capability>/spec.md` |
| What was done / task progress | `openspec/changes/<name>/tasks.md` checkboxes |

No other location is canonical. If a decision doesn't fit one of the above — surface it for discussion, don't create new files.
