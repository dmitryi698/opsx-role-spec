# opsx-role-spec

Role pipeline for AI coding assistants — analyst, planner, coder, reviewer, tester — integrated with the OpenSpec spec-driven workflow.

Derived from [Andrej Karpathy's LLM coding guidelines](https://x.com/karpathy/status/2015883857489522876). Each role is a focused SKILL.md that knows when to hand off to the next role, what artifacts to read and write, and how to stay in sync with the OpenSpec change lifecycle.

## Roles

| Role | Invoked as | Does |
|------|-----------|------|
| `karpathy-guidelines` | `/karpathy-guidelines` | Canonical mindset: think before coding, simplicity first, surgical changes, goal-driven execution. Pipeline conventions live here. |
| `analyst` | `/analyst` | Formalizes requirements as brownfield deltas (`[ADDED]`/`[MODIFIED]`/`[REMOVED]`) against existing specs. No code. |
| `planner` | `/planner` | Turns requirements into a checkbox task list with `Verify:` steps. Writes to `design.md` + `tasks.md`. |
| `coder` | `/coder` | Implements exactly one task. Runs the `Verify:` step until it passes, then flips the checkbox. |
| `reviewer` | `/reviewer` | Maps every WHEN/THEN scenario to the diff. Writes non-obvious decisions to `design.md`. |
| `tester` | `/tester` | Runs test cases seeded from WHEN/THEN scenarios. Reports PASS/FAIL/manual per scenario. |

## Hand-off chain

```
analyst → /opsx:propose → planner → /opsx:apply | coder
       → reviewer → tester → /opsx:archive
```

- `REQUEST CHANGES` or `FAIL` → re-enter at **coder**, never at analyst.
- Requirements turned out wrong → re-enter at **analyst**, update `proposal.md`.

## Connect to a project

```bash
git submodule add https://github.com/dmitryi698/opsx-role-spec.git .claude/pipeline
```

Add one line to each tool's instruction file:

```
# Claude Code — CLAUDE.md
@.claude/pipeline/CLAUDE-SKILLS.md

# Gemini CLI — GEMINI.md
@.claude/pipeline/GEMINI.md

# OpenAI Codex CLI — AGENTS.md
@.claude/pipeline/AGENTS.md
```

## Update

```bash
git submodule update --remote .claude/pipeline
```

## Workflow guide

See [WORKFLOW.md](./WORKFLOW.md) for a step-by-step walkthrough: new feature in 7 steps, bugfix in 5 commands, re-entry rules, antipatterns.
