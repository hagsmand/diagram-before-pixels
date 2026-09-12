# Phase 3 — ASCII to image handoff

The frozen ASCII is the spec. The prompt's only job is to say "render this exactly"
and describe style. Everything else is a liability.

## The template

Fill in the four bracketed sections. Keep the section order — the layout block should
come before the style spec so the model reads structure as the primary constraint.

````text
Render this as a single technical diagram image.

The ASCII layout below is the AUTHORITATIVE specification. Preserve every box, each
box's relative position, every connector, and every arrow direction exactly as shown.
Do not add, remove, merge, or reorder anything.

```
[PASTE THE FROZEN ASCII BLOCK HERE — VERBATIM, INCLUDING THE TITLE LINE]
```

Text that must appear in the image, spelled exactly as written, with no
substitutions, translations, abbreviations, or paraphrasing:
- "[label 1]"
- "[label 2]"
- "[every remaining label and arrow annotation]"

Style: [PASTE ONE STYLE PRESET BELOW, OR THE HOUSE STYLE]

Do not:
- add any component, icon, box, or arrow that is not in the ASCII layout
- reposition or reorder boxes
- invent, shorten, or reword any label
- add logos or product icons implying a technology not named in the layout
- add decorative background elements, gradients, people, or hardware imagery
- render the ASCII characters themselves — render what they represent
````

That last negative constraint matters more than it looks. Without it, models
periodically produce a photograph of monospace text on a screen instead of a diagram.

## Illustrative handoff (composition sketch, not diagram)

For the illustrative branch (SKILL.md Step 0), the ASCII is a frame-region grid, not a
node graph, and the negative-constraint list changes shape to match. Do not reuse the
diagram template above — it tells the model to preserve boxes and arrows, which are
meaningless in an illustration.

````text
Render this as a single illustration.

The ASCII composition sketch below is the AUTHORITATIVE framing. Each region's name,
its position in the grid, and its relative size describe where that element sits in
the final image and how much visual weight it carries. Do not render the ASCII rule
characters themselves — they mark regions, not objects.

```
[PASTE THE FROZEN COMPOSITION SKETCH HERE — VERBATIM, INCLUDING THE TITLE LINE]
```

Style: [PASTE ONE STYLE PRESET BELOW, OR THE HOUSE STYLE]

Do not:
- add any element, character, or background detail not named in a region
- move an element into a different region, or swap two regions' relative sizes
- invent additional subjects not implied by the named regions
- render grid lines, borders, or any trace of the ASCII sketch's rule characters
````

The same Phase 4 diagnosis table applies: a wrong element in the wrong place is a
sketch problem (fix the grid, regenerate), a right layout that looks wrong is a style
problem (swap the preset, resend the sketch byte-identical).

## Style presets

Copy one verbatim into the `Style:` line.

**Clean technical / blueprint**
> Flat 2D vector technical diagram. Thin uniform 2px strokes, rectangular boxes with
> 4px rounded corners, generous white space, white background. Single accent color for
> arrows, neutral dark gray for box outlines and text. Sans-serif labels, consistent
> size. No shadows, no gradients, no 3D. Reads like documentation, not marketing.

**Flat vector isometric**
> Isometric 2.5D illustration, 30-degree axis, flat fills with no gradients. Muted
> three-color palette plus white. Boxes as extruded slabs of equal height, connectors
> as flat ribbons following the isometric grid. Labels rendered flat and horizontal
> (not skewed to the isometric plane) so they stay legible.

**Hand-drawn whiteboard**
> Hand-drawn whiteboard sketch aesthetic. Slightly irregular ink strokes, marker
> texture, off-white paper background. Two marker colors: dark for structure, one
> accent for flow arrows. Handwritten-style but fully legible labels. Casual but not
> messy — every label readable.

**Dark dashboard**
> Dark UI aesthetic. Near-black background (#0f1115), boxes as subtly lighter panels
> with 1px cool-gray borders, one saturated accent for active flow arrows and a muted
> desaturated tone for secondary edges. Light gray text. Thin monospace labels. Even,
> low-contrast lighting. No glow, no neon.

**Print-ready**
> Monochrome line art suitable for grayscale print at 300 DPI. Pure black strokes on
> white, no fills, no halftones. Differentiate edge types by stroke pattern (solid,
> dashed, dotted) rather than by color. Serif labels at a size legible when the image
> is reproduced at 8cm wide.

## Model notes

Capabilities shift release to release — treat these as starting assumptions and adjust
from what you actually observe.

| Model family | Layout adherence | In-image text | Notes |
|---|---|---|---|
| Gemini image models (incl. Nano Banana) | Strong | Strong | Best default for this workflow. Supports conversational iterative edits, so Phase 4 targeted repair works well. |
| GPT Image / DALL·E | Good | Good | Follows explicit negative constraints reliably. Verbatim-text list matters here. |
| Imagen | Good | Moderate | Strong aesthetics; verify every label. |
| Midjourney | Weak for prescribed layout | Weak | Optimizes for aesthetics over instruction-following. Use the overlay fallback. |
| Stable Diffusion / Flux | Weak without control | Weak–moderate | Only viable with ControlNet-style conditioning; otherwise use the overlay fallback. |

## Overlay fallback (weak in-image text)

When the model cannot render text reliably, split the job:

1. Ask the model for **the visual frame only** — boxes, connectors, style, background.
   State explicitly: "leave all label areas blank, render no text of any kind."
2. Add labels in a layer you control — SVG `<text>`, an HTML/CSS overlay, or a
   Figma/slide text layer on top of the exported image.

You get the style you wanted and text that is correct, searchable, and editable
without regenerating. For diagrams with more than ~15 strings this is often the better
path even with a strong text model.

## Skipping pixels: ASCII to Mermaid

For docs and anything version-controlled, this is usually the right end state. The
mapping from the frozen ASCII is mechanical:

| ASCII pattern | Mermaid |
|---|---|
| Linear pipeline | `flowchart LR` |
| Layered stack | `flowchart TD` with `subgraph` per layer |
| Hub and spoke | `flowchart LR`, hub declared first |
| Request/response sequence | `sequenceDiagram` |
| State machine | `stateDiagram-v2` |
| Tree | `flowchart TD` or a fenced tree left as-is |
| Bounded mesh | `flowchart` with nested `subgraph` per boundary |
| Swimlane | `sequenceDiagram` with `box`, or a Markdown table |

Carry the edge labels across (`A -- "order.paid" --> B`) and keep dashed edges dashed
(`-.->`). Zero quota, diffable in a PR, and it will not go stale silently the way an
embedded PNG does.

## Phase 4 diagnosis

Before spending a second generation, identify which layer failed:

- **Structure wrong** — a box is missing, an arrow reversed, grouping off. Either the
  ASCII was wrong (fix it) or the model ignored it (strengthen the authoritative-layout
  instruction, reduce competing style prose). Regenerate.
- **Style wrong, structure right** — swap the style preset. Resend the ASCII
  **byte-identical**; do not retype or reflow it.
- **One region wrong** — use image edit / inpaint scoped to that region. Do not
  regenerate the whole image.
- **Text garbled throughout** — switch to the overlay fallback. Further prompt tuning
  will not fix a model-level limitation.

Never respond to a bad generation by writing a longer prose description. That is the
prose-first failure mode, re-entered through the back door.
