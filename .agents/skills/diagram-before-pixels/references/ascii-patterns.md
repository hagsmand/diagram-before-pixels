# ASCII topology patterns

Pick by the shape of the flow, not by preference. Each pattern lists what it is for,
the failure it prevents, and a rendered example you can adapt.

## Character set

```
Boxes      ┌ ─ ┐ │ └ ┘ ├ ┤ ┬ ┴ ┼
Heavy      ┏ ━ ┓ ┃ ┗ ┛          (emphasis / the subject of the diagram)
Dashed     ┄ ┈ ╌                 (async, optional, planned)
Double     ╔ ═ ╗ ║ ╚ ╝           (trust or network boundary)
Arrows     ▶ ◀ ▲ ▼  ──▶  ◀──  ──▶◀──  (last one = bidirectional)
ASCII-only + - | > v ^ =         (when Unicode will not render)
```

Convention used throughout — keep it consistent within one diagram:

| Meaning | Notation |
|---|---|
| Synchronous call | solid line, `──▶` |
| Asynchronous / event / queue | dashed line, `┄┄▶` |
| Retry / redelivery | `──▶` with label `retry ×3` |
| Bidirectional | `◀──▶` |
| Boundary crossing | line passes through a `║` or `═` wall |
| The box the diagram is *about* | heavy border `┏━┓` |

---

## 1. Linear pipeline

**Use when** data moves through ordered stages with no branching. ETL, build
pipelines, request middleware chains, media processing.

**Prevents** the classic mistake of drawing a pipeline as a hub-and-spoke, which hides
ordering — the one thing a pipeline diagram exists to communicate.

```
┌──────────┐  raw CSV   ┌──────────┐  typed rows  ┌──────────┐  upserts  ┌──────────┐
│  s3-drop │───────────▶│ validate │─────────────▶│ enrich   │──────────▶│ orders_pg│
└──────────┘            └────┬─────┘              └──────────┘           └──────────┘
                             │ schema fail
                             ▼
                        ┌──────────┐
                        │  dlq-s3  │
                        └──────────┘
```

Note the error branch dropping downward. A pipeline diagram with no failure path is
almost always incomplete.

---

## 2. Layered stack

**Use when** the story is about separation of concerns or dependency direction.
Clean architecture, network layers, platform tiers.

**Prevents** ambiguity about which way dependencies point — label the gaps.

```
┌─────────────────────────────────────────────────────┐
│  web (Next.js)          │  mobile (Expo)            │  presentation
└─────────────────────────┴───────────────────────────┘
                      │ tRPC over HTTPS
                      ▼
┌─────────────────────────────────────────────────────┐
│  use-cases: PlaceOrder · RefundOrder · Reprice      │  application
└─────────────────────────────────────────────────────┘
                      │ repository interfaces
                      ▼
┌─────────────────────────────────────────────────────┐
│  entities: Order · Money · Customer   (no I/O)      │  domain
└─────────────────────────────────────────────────────┘
                      ▲
                      │ implements
┌─────────────────────────────────────────────────────┐
│  drizzle · stripe-sdk · resend       (adapters)     │  infrastructure
└─────────────────────────────────────────────────────┘
```

The upward `implements` arrow from infrastructure is the whole point of a clean-
architecture diagram. Omit it and the diagram says nothing a bullet list could not.

---

## 3. Hub and spoke

**Use when** one component mediates many. API gateway, message broker, orchestrator.

**Prevents** the reader assuming spokes talk to each other. If they do, this is the
wrong pattern — use the bounded mesh.

```
                    ┌───────────┐
                    │  auth-svc │
                    └─────▲─────┘
                          │ verify JWT
                          │
┌──────────┐  HTTPS  ┏━━━━┷━━━━━━┓  gRPC   ┌──────────────┐
│  clients │────────▶┃  gateway  ┃────────▶│  orders-svc  │
└──────────┘         ┗━━━┯━━━━┯━━┛         └──────────────┘
                         │    │ gRPC
              rate limit │    └───────────▶┌──────────────┐
                         ▼                 │ catalog-svc  │
                   ┌───────────┐           └──────────────┘
                   │  redis    │
                   └───────────┘
```

---

## 4. Request/response sequence

**Use when** *ordering in time* is the message. Auth handshakes, payment flows,
distributed transactions, retry semantics.

**Prevents** the reader having to infer order from arrow positions on a box diagram —
which they will get wrong.

```
 browser        gateway        auth-svc        orders-svc        stripe
    │              │              │               │                │
    │─ POST /pay ─▶│              │               │                │
    │              │─ verify ────▶│               │                │
    │              │◀─ sub, scope │               │                │
    │              │─ create ─────────────────────▶│                │
    │              │              │               │─ charge ──────▶│
    │              │              │               │◀─ intent id ───│
    │              │◀─ 201 order, pending ────────│                │
    │◀─ 202 + poll │              │               │                │
    │              │              │               │                │
    │              │              │  ┄ webhook: payment_intent.succeeded ┄
    │              │              │               │◀┄┄┄┄┄┄┄┄┄┄┄┄┄┄│
    │              │              │               │─ mark paid ──▶ orders_pg
```

Numbers are optional; vertical position already encodes order. Add them only if the
final image will be discussed step by step.

---

## 5. State machine

**Use when** an entity has a lifecycle with legal and illegal transitions. Order
status, job status, subscription state, approval workflows.

**Prevents** shipping a diagram that shows states but not the *events* that cause
transitions — which is the part people actually need.

```
                    ┌─────────┐
        create ────▶│ pending │
                    └────┬────┘
             payment ok  │  │  timeout 15m
                  ┌──────┘  └──────┐
                  ▼                ▼
             ┌────────┐       ┌──────────┐
             │  paid  │       │ expired  │◀── terminal
             └───┬────┘       └──────────┘
      ship  │    │  refund request
            ▼    └──────────────┐
      ┌──────────┐              ▼
      │ shipped  │        ┌──────────┐
      └────┬─────┘        │ refunded │◀── terminal
   deliver │              └──────────┘
           ▼
      ┌───────────┐
      │ delivered │◀── terminal
      └───────────┘
```

Mark terminal states explicitly. Unreachable or un-exitable states are the bugs this
diagram is best at exposing — and you find them for free in ASCII.

---

## 6. Tree / hierarchy

**Use when** containment or ownership is the message. Repo layout, org structure,
config precedence, DOM.

```
monorepo/
├── apps/
│   ├── web/            Next.js 15 · app router
│   └── admin/          Vite SPA
├── packages/
│   ├── ui/             shared components  ◀── consumed by both apps
│   ├── db/             drizzle schema + migrations
│   └── config/         eslint · tsconfig · tailwind presets
└── infra/
    └── terraform/      vpc · rds · ecs
```

Annotate to the right. A bare tree carries no more information than `ls -R`; the
annotations are the diagram.

---

## 7. Swimlane (actor × phase)

**Use when** the message is *who does what, when*. Handoffs, on-call runbooks,
release processes, CI/CD ownership.

**Prevents** handoff ambiguity — the reason most process diagrams get redrawn.

```
            │ author        │ CI                │ reviewer      │ release
────────────┼───────────────┼───────────────────┼───────────────┼──────────────
open PR     │ ██ push       │                   │               │
            │               │                   │               │
checks      │               │ ██ lint·test·build│               │
            │               │    (12 min)       │               │
            │               │                   │               │
review      │               │                   │ ██ approve    │
            │               │                   │    or request │
            │               │                   │               │
merge       │ ██ squash     │                   │               │
            │               │                   │               │
deploy      │               │ ██ build image    │               │ ██ canary 5%
            │               │                   │               │ ██ full rollout
```

---

## 8. Quadrant / matrix

**Use when** you are positioning options against two axes. Build-vs-buy, effort-vs-
impact, risk registers.

```
        high impact
             ▲
   ┌─────────┼─────────┐
   │ DO NEXT │ DO NOW  │
   │         │         │
   │ · SSO   │ · fix   │
   │   rollout│   n+1  │
   │         │ · cache │
   │─────────┼─────────│──▶ low effort
   │ SKIP    │ FILL-IN │
   │         │         │
   │ · custom│ · dark  │
   │   CRM   │   mode  │
   └─────────┼─────────┘
             ▼
        low impact
```

Label the axes at both ends. A quadrant with one-ended axes is read backwards about
half the time.

---

## 9. Bounded mesh

**Use when** components talk to several others *and* boundaries matter. Security
reviews, VPC topology, trust-zone diagrams.

**Prevents** the single worst omission in system diagrams: not showing where the
trust boundary is.

```
╔═══════════════ public internet ═══════════════╗
║  ┌──────────┐         ┌──────────────┐        ║
║  │ browser  │         │ stripe       │        ║
║  └────┬─────┘         └──────┬───────┘        ║
╚═══════│══════════════════════│════════════════╝
        │ TLS 1.3              │ signed webhook
╔═══════▼══════════════════════▼════════════════╗
║             vpc-prod  (10.0.0.0/16)           ║
║  ┌──────────────┐        ┌─────────────────┐  ║
║  │   alb        │───────▶│ webhook-worker  │  ║
║  └──────┬───────┘        └────────┬────────┘  ║
║         │ HTTP                    │           ║
║  ╔══════▼═════════ private subnet ▼════════╗  ║
║  ║  ┏━━━━━━━━━━━┓        ┌──────────────┐  ║  ║
║  ║  ┃checkout-  ┃───────▶│  orders_pg   │  ║  ║
║  ║  ┃   api     ┃  SQL   └──────────────┘  ║  ║
║  ║  ┗━━━━┯━━━━━━┛                          ║  ║
║  ║       │ ┄┄▶ ┌──────────────┐            ║  ║
║  ║       └────▶│  redis       │            ║  ║
║  ║   cache     └──────────────┘            ║  ║
║  ╚═════════════════════════════════════════╝  ║
╚═══════════════════════════════════════════════╝
```

---

## Layout hygiene

Applies to every pattern:

- **One axis.** Left→right or top→down, never both in one diagram.
- **No crossing arrows.** A crossing means two related nodes are not adjacent. Move
  the nodes; do not route around.
- **Align box edges** into columns. Ragged edges read as noise and the image model
  will reproduce the raggedness.
- **Equal-width boxes** in the same tier. Pad labels with spaces.
- **Labels above or beside the line**, never inside a box that is not a node.
- **Legend only if you used non-obvious notation.** Three lines maximum, bottom-left.
- **Title line above the diagram** stating the one sentence from Phase 0. It travels
  with the ASCII into the image prompt and keeps the model anchored.
