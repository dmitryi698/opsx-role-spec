# claude-skills

Karpathy-derived role pipeline for AI coding tools, integrated with OpenSpec workflow.

Roles: `analyst` · `planner` · `coder` · `reviewer` · `tester` · `karpathy-guidelines`

## Connect to a project

```bash
git submodule add https://github.com/YOUR_USERNAME/claude-skills .claude/pipeline
```

Then add one line to each tool's instruction file:

**Claude Code** — `CLAUDE.md`:
```
@.claude/pipeline/CLAUDE-SKILLS.md
```

**Gemini CLI** — `GEMINI.md`:
```
@.claude/pipeline/GEMINI.md
```

**OpenAI Codex CLI** — `AGENTS.md`:
```
@.claude/pipeline/AGENTS.md
```

## Update

```bash
git submodule update --remote .claude/pipeline
```

## Workflow guide

See [WORKFLOW.md](./WORKFLOW.md) for step-by-step usage with OpenSpec.
