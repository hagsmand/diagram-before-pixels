# diagram-before-pixels

**An Agent Skill that stops your coding agent from burning image-generation quota on
structurally wrong diagrams.**

Structure is cheap in text and expensive in pixels. So settle the structure in ASCII —
where a revision costs seconds and no quota — and only then spend one image generation
rendering a layout that is already correct.

The technique in one line: **10 ASCII revisions and 1 image generation, instead of 10
image generations.**

## Why

Ask an image model to draw a system from a prose description and it has to invent the
structure. It guesses. You regenerate. You write a longer description. It guesses
differently. Five to fifteen generations later you have something close enough.

An ASCII draft removes the guessing. It also makes review nearly instant — you can see
in two seconds that the cache is missing, that an arrow points the wrong way, or that
three boxes can be cut. Do that ten times for less than the cost of one image.

Once the ASCII is right, the image model stops being an architect and becomes a
renderer. That is the job it is actually good at.

```
prose-first          ▶ 5-15 generations   (model inventing structure each time)
ascii-first          ▶ 1-2 generations    (structure already settled)
                       + 5-10 text edits  (seconds each, no quota)
```

## What the skill does

Five phases, enforced in order:

| Phase | What happens |
|---|---|
| **0 — Extract** | Nodes, edges, boundaries, and the one sentence the viewer must conclude. Real names, never `Service A`. |
| **1 — Draft** | Pick a topology matching the *shape of the flow*. Nine patterns and blank scaffolds included. |
| **2 — Review** | A 10-point checklist run until it passes. Expect 5–10 rounds. This is where the savings are. |
| **3 — Freeze** | Build the handoff prompt: verbatim ASCII as authoritative layout + exact text list + style spec + negative constraints. |
| **4 — Render** | One generation. If it's wrong, diagnose *which layer* failed — never re-prompt from prose. |

It also tells you when **not** to spend quota at all: for docs, READMEs, and anything
version-controlled, ASCII or Mermaid is the better end state — diffable, reviewable in
a PR, and it doesn't go stale silently the way an embedded PNG does. Image generation
is for presentation-grade illustration.

## Install

Works with Claude Code, OpenAI Codex CLI, Cursor, OpenCode, and other agents that read
the [Agent Skills](https://agentskills.io/specification) format.

**Any agent** — installs into every agent you have:

```bash
npx skills add hagsmand/diagram-before-pixels
```

**Claude Code, as a plugin:**

```
/plugin marketplace add hagsmand/diagram-before-pixels
/plugin install diagram-before-pixels@diagram-before-pixels
```

**Manual** — copy into whichever path your agent scans:

```bash
git clone https://github.com/hagsmand/diagram-before-pixels
cd diagram-before-pixels

# Codex CLI / Cursor / OpenCode — native path
cp -r .agents/skills/diagram-before-pixels ~/.agents/skills/

# Claude Code
cp -r .agents/skills/diagram-before-pixels ~/.claude/skills/
```

Native skill directories, for reference:

| Agent | Reads |
|---|---|
| OpenAI Codex CLI | `.agents/skills/` · `~/.agents/skills/` |
| Cursor | `.agents/skills/` · `.cursor/skills/` (+ legacy `.claude/skills/`, `.codex/skills/`) |
| OpenCode | `.opencode/skills/` · `.claude/skills/` · `.agents/skills/` |
| Claude Code | `.claude/skills/` · plugins |

This repo keeps one canonical copy at `.agents/skills/diagram-before-pixels/` — no
symlinks, no duplicated content to drift. The plugin manifests point Claude Code at
that same path.

## Use it without installing anything

`.agents/skills/diagram-before-pixels/assets/bootstrap-prompt.md` is the technique
reduced to one copy-paste message. Works in any chat AI.

## Layout

```
.agents/skills/diagram-before-pixels/
├── SKILL.md                      the five-phase workflow
├── agents/openai.yaml            Codex display sidecar (ignored elsewhere)
├── references/
│   ├── ascii-patterns.md         9 topologies, worked examples, layout hygiene
│   ├── review-checklist.md       the 10-point Phase 2 gate
│   └── image-prompts.md          handoff template, style presets, per-model notes
└── assets/
    ├── bootstrap-prompt.md       standalone copy-paste version
    └── templates/                blank scaffold per topology
```

`references/` and `assets/` load on demand, so the resident context cost is just
`SKILL.md`.

## Contributing

Validate the skill before opening a PR:

```bash
npx -y @agentskills/skills-ref validate .agents/skills/diagram-before-pixels
```

## License

MIT — see [LICENSE](LICENSE).
