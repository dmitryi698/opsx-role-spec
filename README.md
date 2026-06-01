# claude-skills

Karpathy-derived role pipeline for Claude Code, integrated with OpenSpec workflow.

Roles: `analyst` · `planner` · `coder` · `reviewer` · `tester` · `karpathy-guidelines`

## Connect to a project

```bash
git submodule add https://github.com/YOUR_USERNAME/claude-skills .claude/pipeline
```

Then add one line to your `CLAUDE.md`:

```
@.claude/pipeline/CLAUDE-SKILLS.md
```

That's it. All roles become available as `/analyst`, `/planner`, `/coder`, `/reviewer`, `/tester`.

## Update

```bash
git submodule update --remote .claude/pipeline
```

## Workflow guide

See [WORKFLOW.md](./WORKFLOW.md) for step-by-step usage with OpenSpec.
