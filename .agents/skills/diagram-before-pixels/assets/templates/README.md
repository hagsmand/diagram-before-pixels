# Templates

Blank scaffolds for Phase 1. Copy the one whose shape matches the flow, then fill it
in top to bottom in the order each file specifies.

| File | Use when |
|---|---|
| `pipeline.txt` | Ordered stages, no branching. ETL, build steps, middleware chains. |
| `layered.txt` | Separation of concerns or dependency direction. |
| `hub-spoke.txt` | One component mediates many. Gateway, broker, orchestrator. |
| `sequence.txt` | Ordering *in time* is the message. Handshakes, payment flows. |
| `state-machine.txt` | An entity's lifecycle with legal and illegal transitions. |
| `tree.txt` | Containment or ownership. Repo layout, org chart, config precedence. |
| `swimlane.txt` | Who does what, when. Handoffs, release processes, runbooks. |
| `quadrant.txt` | Positioning options against two axes. |
| `bounded-mesh.txt` | Many-to-many *and* boundaries matter. Security, VPC topology. |

Worked examples of each, with the failure mode each pattern prevents, are in
`../../references/ascii-patterns.md`.
