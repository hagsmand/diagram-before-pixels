# Phase 2 review checklist

Run all 10 against the current ASCII draft. Any failure means fix and re-run — do not
advance to image generation. Each pass takes seconds; that is the trade this skill is
built on.

Order matters: 1–3 can delete whole boxes, so run them before polishing anything.

This checklist is for the **structural** branch (nodes, edges, boundaries). For the
illustrative branch, use the lighter "Illustrative variant" checklist at the bottom of
this file instead.

---

## 1. Message

**Check:** Can you state, in one sentence, what the viewer concludes after three
seconds?

**Fix:** If not, you are drawing an inventory, not a diagram. Go back to Phase 0 and
get the sentence from the user. Then put it as a title line above the ASCII.

**Symptom of failure:** the diagram has no visual focus — every box looks equally
important.

---

## 2. Completeness

**Check:** Walk the real system — code, infra config, the user's description — and
confirm every participant that affects the message is present.

Commonly omitted, in rough order of frequency:

- **Auth** — who validates the token, and where
- **Cache** — Redis/CDN layer that changes the latency story entirely
- **Queue / broker** — and whether delivery is at-least-once
- **Retry and DLQ** — where failures go; a flow with no failure path is half a diagram
- **Rate limiter / quota**
- **Observability** — only if the message is about operability; otherwise it is noise
- **Migration or dual-write path** — for any diagram of a system mid-transition
- **The human** — approval steps, on-call, manual gates

**Fix:** add the box, then immediately run check 3 on it.

---

## 3. Necessity

**Check:** Delete each box in turn. Does the sentence from check 1 still land?

**Fix:** If it lands, the box stays deleted. Say out loud what you cut and why —
"dropped the CDN, it doesn't affect the auth story." Silent cuts look like oversights.

This is the highest-yield check in the list and the cheapest place in the whole
process to run it. In an image, cutting a box costs a regeneration.

---

## 4. Direction

**Check:** Every arrow points the way the *initiative* flows, not the way data
happens to move. `orders-svc ──▶ orders_pg` even though rows come back. Sync and
async are visually distinct (solid vs dashed).

**Fix:** Flip arrowheads. Convert event/queue edges to dashed. If an edge is truly
bidirectional and that matters, use `◀──▶`; if it does not matter, pick the
initiating direction.

**Watch for:** dependency-inversion arrows in layered diagrams pointing the wrong way
— the single most common error in clean-architecture diagrams.

---

## 5. Edge labels

**Check:** Every arrow has a label saying what crosses it — payload, protocol, event
name, or the verb.

**Fix:** Label it. `POST /orders`, `order.paid`, `SQL`, `verify JWT`, `retry ×3`.

Unlabeled arrows are the number one cause of "beautiful but wrong." The image model
will happily render a clean unlabeled arrow, and the reader will invent a meaning for
it — usually not yours.

---

## 6. Boundaries

**Check:** Are network, trust, and ownership boundaries drawn?

- Network: VPC, subnet, cluster, region
- Trust: public / authenticated / internal-only
- Ownership: which team or repo owns each box

**Fix:** Wrap groups in `╔═╗` walls, label the wall itself. For ownership, annotate to
the right rather than adding another nesting level.

**Skip only if** the diagram genuinely has one trust zone — and confirm that rather
than assuming it.

---

## 7. Adjacency

**Check:** Are related nodes neighbors? Are there zero crossing arrows?

**Fix:** A crossing arrow is a layout bug, not a routing problem. Move the boxes so
the crossing disappears. If it cannot disappear, the topology pattern is wrong — go
back to `ascii-patterns.md` and pick another, or split into two diagrams.

---

## 8. Naming

**Check:** Every label is a real name from the real system. Zero `Service A`, `DB`,
`Frontend`, `Microservice`, `External API`.

**Fix:** Look the names up — `workspaceSymbol`, the infra config, `package.json`
names, the actual queue and table names. Placeholder names are a signal that nobody
has verified what the box is.

---

## 9. Reading order

**Check:** One consistent axis — left→right or top→down, not both. Entry point is
where the reader's eye starts (top-left for LTR audiences). Terminal states and sinks
are at the end of the axis.

**Fix:** Rotate the whole diagram rather than reordering individual boxes.

---

## 10. Text budget

**Check:** List every string that must appear in the final image, verbatim. Count
them.

**Fix:** If the list exceeds ~15 strings, the image model will start misspelling and
dropping them. Reduce labels, or plan for the SVG/HTML overlay fallback in
`image-prompts.md`.

This list is a required input to the Phase 3 prompt — write it out now, not later.

---

## Exit gate

All ten pass →

1. Freeze the ASCII. No further edits without re-running the checklist.
2. Ask where the diagram is going. Docs or wiki → ship the ASCII or convert to
   Mermaid and stop; there is no reason to spend quota.
3. Presentation-grade output needed → Phase 3.

---

## Illustrative variant

For the illustrative branch's composition sketch. Five checks, not ten — there are no
edges, boundaries, or dependency directions to verify.

1. **Message** — can you state what the viewer feels or recognizes after three
   seconds? If not, go back and get the one sentence before drawing more.
2. **Completeness** — does every region that affects that sentence appear? A dragon
   sketch that's all head and no scene context is missing the story.
3. **Necessity** — remove each region in turn. If the sentence still lands without it,
   cut it. Same highest-yield check as the structural list.
4. **Weight and placement** — does box size match intended visual dominance, and does
   grid position match intended frame position (top/mid/bottom, left/center/right)? A
   "background" element drawn as big as the subject will render as big as the subject.
5. **Text budget** — same as structural check 10: list every string (if any) that must
   appear in the image, verbatim. Most illustrative requests have zero — that's fine,
   note it explicitly rather than leaving it unconsidered.

Exit gate: all five pass → freeze the sketch, go to Phase 3 with the "Illustrative
handoff" variant in `image-prompts.md`.
