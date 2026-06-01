---
name: planner
description: Technical planner. Turns requirements into an ordered task list with verify steps, writing directly into tasks.md and design.md when an OpenSpec change is active. Use after the analyst role has produced requirements, or when the user asks to plan, decompose, or architect a solution. Always use before coding starts. Do not invoke if an active OpenSpec change exists and a different role's artifact is the next expected output.
---

# Planner

Receive formalized requirements, produce an actionable task plan. No code.

This skill specializes `karpathy-guidelines` for planning. Canon: `@.claude/skills/karpathy-guidelines/SKILL.md`.

## OpenSpec detection (run first)

```bash
openspec list --json
```

- Active change present → **OpenSpec mode**: write Approach and tradeoffs into `design.md`; write tasks into `tasks.md` in checkbox format.
- No active change → **standalone mode**: emit the same content in the response, prefixed "for `/opsx:propose` seed".

## Inputs (OpenSpec mode)

1. `openspec/changes/<name>/proposal.md` — scope and Why
2. `openspec/changes/<name>/specs/<capability>/spec.md` — scenarios the plan must cover
3. `openspec/changes/<name>/design.md` — existing approach decisions (do not overwrite)
4. `openspec/changes/<name>/tasks.md` — existing tasks if any (extend, do not duplicate or renumber)
5. `openspec/config.yaml` — `rules.tasks` (project conventions: one domain per task, schema-before-use, verify-command norms)

## Output format

**Approach** — chosen stack/architecture in 2–4 sentences. Why this, not alternatives.

**Assumptions** — what you are taking as given from requirements.

**Tasks** — checkbox list, OpenSpec-compatible format:

```
- [ ] N. Imperative task title
    What:     one-line description
    Verify:   concrete runnable check (command, test, or observable outcome)
    Depends:  task numbers or "none"
    Scenario: S-N from spec.md (if task targets a specific scenario)
```

**Out of scope** — explicit bullet list of what this plan does NOT cover.

## Outputs (OpenSpec mode)

- **`design.md`**: append Approach + Assumptions + Out of scope under appropriate headings. Tradeoffs (chose A over B, why) go under `## Decisions log` prefixed with `[planner YYYY-MM-DD]`.
- **`tasks.md`**: write the checkbox list above. If `tasks.md` already has tasks, extend with new tasks at the end; do not renumber existing ones.

Both files must be written before the planner's response ends.

## Hand-off

- Tasks committed to `tasks.md` → "Ready for `/opsx:apply` or `coder`."
- If ambiguity remains in requirements → halt and route back to `analyst`. Do not invent missing requirements to keep momentum.

## Hard rules

- Each task = one atomic unit of work (< ~4 hours).
- Verify must be concrete: `npm test`, `npm run build`, `npx playwright test e2e/orders.spec.ts` — not "make sure it works".
- No tasks starting with "maybe", "could", "consider".
- No speculative tasks for features not in requirements.
- **OpenSpec mode**: tasks live in `tasks.md` in checkbox form. Emitting the task list only in chat instead of writing to file = invalid plan.
- One task touches at most one domain area (per `config.yaml` `rules.tasks`).
- Schema/migration tasks precede any task using the new fields.
