# Recursive grilling v1 hooks (no automation)

Three storage hooks ship in v1 for v2's recursive-grilling feature,
but **no code path triggers grilling autonomously** in v1:

1. **`Ticket.kind: "code" | "design"`** discriminator, default `"code"`.
2. **`FailureClassDoc`** per Failed ticket:
   `{ fingerprint, count, lastSummary, reproCount, escalated: boolean }`.
   Bookkeeping only; not actively populated.
3. **`recursable: boolean`** field on tickets (default `true`).
4. **Manual "Escalate to design" UI button** on Failed tickets that opens
   a design-grilling sub-conversation (the user explicitly opts in).

**Why.** Storage is cheap; behavior is expensive. Including hooks gives
v2 data continuity (no migration) at no runtime cost. The UI button
gives v1 users a manual escape valve ("I want to investigate this
design-class failure"). Automated escalation (fingerprint clustering,
recursion-depth limits, cost gates) is real work that needs real
failure data to design — that's v2.

**Consequences.** `Ticket.kind` exists from day one but writes only
`"code"`. `FailureClassDoc` reads/writes no-op. `recursable` is
bookkeeping for v2 routing. The "Escalate to design" button is wired
to a real workflow in v1: opens a grilling sub-conversation that itself
runs through the SDLC pipeline.
