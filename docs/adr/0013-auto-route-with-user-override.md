# Auto-route tickets to personas with user override at intake

Routing at end of orchestrator intake: rules first (exact field match
on `kind`/`labels` — confidence 1.0 short-circuits), LLM fallback
otherwise. The suggested persona + confidence is surfaced in the Kanban
card; the owner can drag-reassign or accept.

**Why.** Most tickets are unambiguous after a 5-question intake
conversation. Auto-routing saves user effort on the common case; user
override preserves authority on the edge case. Rules-then-LLM beats
LLM-only (transparency, cost) and rules-only (handles ambiguity).

**Consequences.** Auto-routing accuracy improves with usage data; the
heuristic can retrain without rewriting routing logic. Low-confidence
routings surface in the UI for confirmation before dispatch.
