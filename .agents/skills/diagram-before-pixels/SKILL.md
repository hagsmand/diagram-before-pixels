---
name: diagram-before-pixels
description: >-
  Converge on an image's structure in cheap ASCII before spending any
  image-generation quota, then render it once. Covers two distinct cases:
  structural diagrams (architecture, system, flowchart, sequence, state
  machine, infographic) drafted as nodes/edges/boundaries, and illustrative or
  creative image requests (a dragon, a character, a scene, a product shot)
  drafted as a composition sketch — a single frame with labeled regions
  showing where each visual element sits. Use whenever an image model (Nano
  Banana, Gemini, GPT Image, DALL-E, Midjourney, Imagen, Stable Diffusion)
  will produce the final artwork, whichever case applies — do not force a
  creative subject into boxes-and-arrows. Also use when image generations keep
  coming back structurally or compositionally wrong, when quota is limited, or
  when the result must match a real system or a specific composition exactly.
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

## Step 0 — Classify: structural or illustrative?

Two different jobs share this skill. Tell them apart before drawing anything.

- **Structural** — the viewer must understand relationships: architecture, flowcharts,
  sequence diagrams, state machines, org charts, infra topology. There are real nodes
  that call, own, or depend on each other. Go to **Phase 0** below.
- **Illustrative** — the viewer must see a subject: a dragon, a character, a product
  shot, a landscape, a portrait. There is no call graph — there's a subject with parts,
  occupying regions of a frame. Skip Phase 0's node/edge/boundary extraction and go
  straight to **Illustrative composition** below.

Tell them apart by asking: do any two elements *call, send to, or depend on* each
other? "Draw a dragon breathing fire over a village" has a subject and a scene, not a
call graph — illustrative. "Show how the checkout service calls Stripe" has actors
exchanging messages — structural. A dragon's wing is not a "component" that an arrow
points at; forcing it into boxes-and-arrows is the mirror image of the mistake this
skill exists to prevent — don't trade a wrong picture for a wrong diagram. When the
request is genuinely ambiguous, ask the user which one they mean rather than guessing.

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

## Illustrative composition (for the illustrative branch of Step 0)

Same reasoning as Phase 0–3, lighter weight, different vocabulary: no nodes and edges,
just a subject and the frame it occupies.

1. **The one sentence.** What must the viewer feel or recognize after three seconds —
   "a dragon mid-roar, coiled to strike" is a sentence; "a dragon" is not.
2. **Elements.** Named parts of the subject and scene — `head`, `wings`, `tail`, `fire
   breath`, `mountain background`, `foreground rock` — not `Component A`.
3. **Frame regions.** Where each element sits (top/mid/bottom × left/center/right) and
   its relative weight — dominant, secondary, or background — plus the frame's aspect
   ratio (square, 16:9, portrait), which is part of the spec, not an afterthought.
4. **Composition sketch.** One frame divided into labeled regions — a grid, not a node
   graph. Plain ASCII rules (`+ - |`), not box-drawing Unicode: this is a wireframe of
   the *frame*, and using the structural diagram's own character set would blur the
   two into looking like the same kind of artifact. Region name in caps, one-line
   description beneath it:

   ```
   +--------------------------------------------------+
   |             STORMY SKY + FULL MOON               |
   |        Clouds, stars, distant flying dragons     |
   +---------------------------+----------------------+
   |                           |                      |
   |      DRAGON WINGS         |   MOUNTAIN PEAKS     |
   |                           |   + ruined castle    |
   |     +---------------------+----------------------+
   |     |                                            |
   |     |            MAIN DRAGON                     |
   |     |       Head, glowing eyes, scales           |
   |     |                                            |
   +-----+----------------------------+---------------+
   |          DRAGON BODY             |  FIRE BREATH  |
   |     Standing on rocky cliff      |  toward valley|
   +----------------------------------+---------------+
   |       FOREGROUND: rocks, warrior, glowing lava   |
   +--------------------------------------------------+
   ```

   Region sizes are a rough proportion of screen real estate, not exact math — bigger
   box means more visual weight, that's the whole signal.

Run the same review loop as Phase 2: show the sketch, revise it, and run the
highest-yield check — remove each labeled region in turn and ask whether the one
sentence still lands. Cut what doesn't. Then hand off per Phase 3 and the "Illustrative
handoff" variant in `references/image-prompts.md`, swapping "preserve every box and
arrow" for "preserve every labeled region's position and relative size." The same
negative constraints apply: no elements beyond the sketch, no resized or repositioned
regions, no invented parts.

## When to skip the image model entirely

This applies to the structural branch — an illustrative request (a dragon, a scene, a
portrait) has no non-pixel end state; the composition sketch is scaffolding, not the
deliverable. For structural diagrams, be honest about this rather than pushing every
diagram to pixels:

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
| `references/ascii-patterns.md` | Phase 1 — choosing and drawing a topology (or pattern 10 for illustrative) |
| `references/review-checklist.md` | Phase 2 — every review round (structural or illustrative variant) |
| `references/image-prompts.md` | Phase 3/4 — building the handoff prompt, picking a style, per-model quirks |
| `assets/bootstrap-prompt.md` | The user wants a copy-paste prompt to get an ASCII draft out of any chat AI |
| `assets/templates/` | Phase 1 — starting skeleton per topology, or `composition.txt` for the illustrative branch |
