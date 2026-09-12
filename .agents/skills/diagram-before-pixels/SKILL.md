---
name: diagram-before-pixels
description: >-
  Converge on a diagram's structure in cheap ASCII before spending any
  image-generation quota, then render it once. Use when asked to create or
  revise an architecture diagram, system diagram, flowchart, sequence diagram,
  infographic, or technical illustration — especially when an image model
  (Nano Banana, Gemini, GPT Image, DALL-E, Midjourney, Imagen, Stable
  Diffusion) will produce the final artwork. Also use when image generations
  keep coming back structurally wrong, when diagram credits or quota are
  limited, or when a diagram must match a real system exactly.
license: MIT
---

# Diagram before pixels

Structure is cheap in text and expensive in pixels. Settle the structure in ASCII,
where a revision costs seconds and no quota, then spend exactly one image
generation rendering a layout that is already correct.

A pretty diagram of the wrong system is still a wrong diagram. The image model's job
is to **render**, not to architect. You do the architecture in monospace first.

## The rule

Never send a prose-only description to an image model when structure matters. Send
the finalized ASCII layout as the authoritative spec, with prose only for style.

## Phase 0 — Extract the spec before drawing anything

Do not draw yet. Write these four things out first.

1. **The one sentence.** What must the viewer conclude after three seconds of
   looking? Write it literally. If you cannot write it, stop and ask the user —
   every diagram that skips this step comes out pretty and wrong.
2. **Nodes.** Real names, pulled from the codebase, infra config, or the user's own
   words. `checkout-api`, `orders_pg`, `stripe-webhook-worker`. Never `Service A`,
   never `Database`. Placeholder names hide the fact that nobody has checked what
   the boxes actually are.
3. **Edges.** For each one: source, target, direction, synchronous or asynchronous,
   and *what flows across it* (payload, protocol, event name).
4. **Boundaries.** Network (VPC, subnet), trust (public vs authenticated vs
   internal), and ownership (which team or repo owns which box). Boundaries are the
   most frequently omitted element and usually the most load-bearing.

When exploring an unfamiliar codebase to fill this in, prefer LSP navigation
(`workspaceSymbol`, `findReferences`, `incomingCalls`) over grep — you want the real
call graph, not string matches.

## Phase 1 — ASCII draft

Read `references/ascii-patterns.md` and pick the topology that matches the **shape of
the flow**, not the one you like drawing. Linear pipeline, layered stack,
hub-and-spoke, request/response sequence, state machine, tree, swimlane, quadrant,
or bounded mesh.

Start from the matching skeleton in `assets/templates/` rather than a blank buffer.

Constraints:

- Box-drawing characters (`┌ ─ ┐ │ └ ┘ ├ ┤ ┬ ┴ ┼ ▼ ▲ ◀ ▶`). ASCII fallback (`+ - |
  > v`) only when the target medium cannot render Unicode.
- Width ≤ 100 columns. Wider does not survive a chat window, a PR description, or a
  16:9 slide.
- **5–9 boxes.** Fewer than 5 usually means the diagram is not earning its place.
  More than 12 means you are drawing two diagrams at once — split them and say so.
- Every arrow carries a label.

Emit the draft in a fenced code block. Show it to the user. Do not describe it in
prose afterward; if it needs a paragraph of explanation, the diagram is not done.

## Phase 2 — Review loop (this is where the savings are)

Run every check in `references/review-checklist.md` against the draft. Fix, re-render
the whole block, run the checklist again.

**Do not advance to Phase 3 while any check fails.**

Expect 5–10 rounds. Each round is seconds and effectively free — this is the entire
point of the technique. Ten cheap revisions here replace ten expensive
regenerations later.

Two things to do explicitly in this loop:

- **Show each revision.** The user reviewing a text block in two seconds is the
  mechanism that makes this work. Do not batch ten silent revisions and present only
  the last one — you will have optimized against your own guess instead of theirs.
- **Name what you cut and why.** "Dropped the CDN box — it doesn't affect the auth
  story." Cuts are as valuable as additions and are invisible unless stated.

The highest-yield check, run it first: delete each box in turn and ask whether the
one sentence from Phase 0 still lands. If it does, the box stays deleted.

## Phase 3 — Freeze and hand off

Only now bring in the image model. Build the prompt per
`references/image-prompts.md`. It has five parts, all required:

1. An instruction that the ASCII block below is **authoritative layout** — preserve
   every box, its relative position, and every arrow direction.
2. The frozen ASCII, verbatim, inside a fenced block. Do not paraphrase it into
   prose. Do not re-type it.
3. A verbatim list of every text string that must appear in the image, spelled
   exactly. Image models substitute and misspell in-image text constantly; an
   explicit list is the only reliable defense.
4. The style spec — one of the presets in `references/image-prompts.md`, or the
   user's house style.
5. Negative constraints: do not add components absent from the ASCII, do not reorder
   boxes, do not invent labels, do not add icons implying a technology that is not
   listed.

## Phase 4 — One generation, then targeted repair

Generate once. If the result is wrong, diagnose **which layer** failed before
spending another credit:

| What is wrong | What it means | What to do |
|---|---|---|
| Structure — missing box, wrong arrow, wrong grouping | The ASCII was wrong, or the model ignored it | Fix the ASCII, regenerate. Never patch structure with prose. |
| Style only — structure correct, looks wrong | Style spec too vague | Adjust style spec, resend the ASCII **byte-identical** |
| One region — single label garbled, one icon off | Local defect | Use the model's image-edit / inpaint path, not a full regeneration |
| Text garbled throughout | Model has weak in-image text rendering | Switch to the overlay fallback in `references/image-prompts.md` |

Re-prompting from prose after a bad generation is the failure mode this whole skill
exists to prevent. If you find yourself writing a longer description, stop and go
back to the ASCII.

## When to skip the image model entirely

Be honest about this rather than pushing every diagram to pixels:

- **Docs, READMEs, wikis, PR descriptions, code comments** — ship the ASCII itself,
  or convert it to Mermaid. Both are diffable, reviewable in a PR, version
  controlled, and cost zero quota. An image in a README goes stale silently.
- **Anything that will change again next sprint** — Mermaid or SVG. Regenerating an
  image on every change is exactly the quota drain this skill is about.
- **Slides, landing pages, social posts, conference talks, external decks** — this is
  what image generation is actually for. Presentation-grade illustration earns its
  cost; a system diagram in an internal doc usually does not.

If the user has not said where the diagram is going, ask. The answer changes whether
Phase 3 should happen at all.

## Why this saves quota

Prose-first, the image model is inventing structure from an ambiguous description,
so it guesses — and typically needs 5–15 regenerations before the guess matches
intent. ASCII-first, structure is already settled and agreed, so 1–2 generations is
normal.

The revisions do not disappear. They move from the expensive medium to the cheap one.
Ten ASCII edits plus one generation, instead of ten generations.

## Reference files

Load these on demand, not up front:

| File | Read it when |
|---|---|
| `references/ascii-patterns.md` | Phase 1 — choosing and drawing a topology |
| `references/review-checklist.md` | Phase 2 — every review round |
| `references/image-prompts.md` | Phase 3/4 — building the handoff prompt, picking a style, per-model quirks |
| `assets/bootstrap-prompt.md` | The user wants a copy-paste prompt to get an ASCII draft out of any chat AI |
| `assets/templates/` | Phase 1 — starting skeleton per topology |
