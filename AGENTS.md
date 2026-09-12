# AGENTS.md

This repository **is** an Agent Skill. There is no application to build, run, or test.

The skill lives at `.agents/skills/diagram-before-pixels/SKILL.md`. If you were asked to
make a diagram, read that file and follow it.

## If you are editing this repo

- `SKILL.md` frontmatter may contain **only** these keys: `name`, `description`,
  `license`, `compatibility`, `metadata`, `allowed-tools`. Anything else is rejected by
  Anthropic's packaging tooling and by `skills-ref validate`. No `when_to_use`, no
  `argument-hint`, no `model`.
- `name` must stay equal to the parent directory name (`diagram-before-pixels`).
- `description` must stay under 1024 characters and must state both *what it does* and
  *when to use it* — it is the only text an agent sees when deciding to load the skill.
- Keep `SKILL.md` under 500 lines. Detail goes in `references/`, which loads on demand.
- One canonical copy only, at `.agents/skills/diagram-before-pixels/`. Do not add a
  duplicate under `.claude/skills/` or `skills/` — the plugin manifests in
  `.claude-plugin/` point Claude Code at the canonical path.
- If you add or rename a reference or template, update the file table at the bottom of
  `SKILL.md` and `assets/templates/README.md`.

## Verify

```bash
npx -y @agentskills/skills-ref validate .agents/skills/diagram-before-pixels
jq . .claude-plugin/marketplace.json .claude-plugin/plugin.json
wc -l .agents/skills/diagram-before-pixels/SKILL.md   # must be < 500
```

Spec: <https://agentskills.io/specification>
