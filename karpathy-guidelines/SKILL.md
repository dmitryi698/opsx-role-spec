---
name: karpathy-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code to avoid overcomplication, make surgical changes, surface assumptions, and define verifiable success criteria.
license: MIT
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

## Pipeline Conventions (OpenSpec mode)

These conventions specialize the four principles above for this project's role pipeline. They are not part of the canonical Karpathy mindset — they describe how the `analyst` / `planner` / `coder` / `reviewer` / `tester` skills wire together when an OpenSpec change is active. The 5 role skills point here as canon; on conflict, this section wins.

### Hand-off chain

```
analyst → /opsx:propose → planner → /opsx:apply | coder
       → reviewer → tester → /opsx:archive
```

Each role's output explicitly names the next role or command, so the chain is followable without re-reading every skill file.

### Re-entry rules

- `REQUEST CHANGES` (reviewer) or `FAIL` (tester) → re-enter at **coder** with concrete IDs (`fix R-3, R-5`, `address bug 2`).
- Never restart the pipeline from `analyst` or `planner` on review/test feedback — that feedback addresses code, not requirements.
- If feedback reveals that the requirements themselves are wrong, pause and prompt the user; do not silently re-derive.

### Decision log (three layers, no others)

- **Per-change decisions** (tradeoffs, invariants, constraints surfaced during work) → `openspec/changes/<name>/design.md` under a `## Decisions log` heading. Reviewer writes there before issuing the Verdict; coder may append `[coder YYYY-MM-DD]` entries when a non-obvious choice is made during implementation.
- **Project-wide conventions** (tech stack, file structure, naming rules, RBAC, multi-tenancy) → `openspec/config.yaml` (`context:` and `rules:`).
- **Pipeline conventions** (this section) → here in `karpathy-guidelines/SKILL.md`.
- No other location is canonical. A decision that doesn't fit one of the three above has no home — surface it for the user to decide where it belongs.

### OpenSpec detection preamble

Every role's first step:

```bash
openspec list --json
```

- Active change present → **OpenSpec mode**: read `proposal.md`, `specs/<capability>/spec.md`, `design.md`, `tasks.md` for the change before producing output; write back to the appropriate artifact.
- No active change → **standalone mode**: skill's original behaviour, no OpenSpec reads/writes.

### Negative trigger pattern

Each role's `description` field ends with:

> Do not invoke if an active OpenSpec change exists and a different role's artifact is the next expected output.

This prevents trigger collision when multiple roles match a single user message — only the role whose artifact is next will load.

### Rule scope notes

- `any` type as blocker — **TypeScript projects only**. This project is TS5 strict, so the rule applies. When porting these skills elsewhere, gate accordingly.
- UI strings and copy in Russian — **this project only**. Documented in `openspec/config.yaml` `specs:` rules; not a Karpathy principle.
- Verify commands — concrete project commands (`npm test`, `npm run build`, `npm run test:e2e`) belong in `openspec/config.yaml` and `tasks.md`, not here. This file stays project-agnostic at the principle level.
