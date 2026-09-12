# Bootstrap prompt

Copy-paste this into any chat AI to get a first ASCII draft. Works without the skill
installed — this is the technique reduced to one message.

---

````text
Before you generate any image, draw this as an ASCII diagram first.

Subject: [WHAT YOU WANT DIAGRAMMED]
The one thing a viewer must understand from it: [YOUR ONE SENTENCE]
Where it will end up: [slide deck / README / blog post / internal doc]

Rules for the ASCII draft:
- Use box-drawing characters (┌─┐│└┘├┤▼▶). Max 100 columns wide.
- 5 to 9 boxes. If it needs more than 12, tell me it should be two diagrams.
- Use real names, not placeholders like "Service A" or "Database".
- Label every arrow with what crosses it (payload, protocol, event name, or verb).
- Solid arrows for synchronous calls, dashed for async/events/queues.
- Draw trust and network boundaries as walls (╔═╗) and label the wall.
- Put my one sentence as a title line above the diagram.

Then, below the diagram, list separately:
1. Components you included that I did not mention, and why.
2. Components you deliberately left out, and why.
3. Anything you had to guess.

Output the ASCII in a fenced code block. Do not generate an image yet — wait until I
say the layout is correct.
````

---

## Follow-up messages

**Revise** (repeat as needed — this is where the savings are):

```text
Revisions: [what to change]
Re-output the complete ASCII block. Then re-run your own check for: missing auth,
cache, queue, retry/DLQ paths; unlabeled arrows; crossing arrows; placeholder names.
```

**Cut aggressively:**

```text
Delete each box in turn and tell me whether my one sentence still lands without it.
Remove every box that fails that test and show me the trimmed version.
```

**Approve and render** (only once the ASCII is right):

```text
The layout is correct. Now generate the image.

The ASCII block above is the AUTHORITATIVE layout — preserve every box, its relative
position, and every arrow direction. Add nothing, remove nothing, reorder nothing.

Every text string in the layout must appear in the image spelled exactly as written,
with no substitutions or abbreviations.

Style: [PICK ONE FROM references/image-prompts.md]

Do not add components, icons, or logos absent from the layout. Do not render the ASCII
characters themselves — render what they represent.
```

**Repair without a full regeneration:**

```text
Structure and style are right except [the specific region]. Edit only that region of
the image. Leave everything else pixel-identical.
```
