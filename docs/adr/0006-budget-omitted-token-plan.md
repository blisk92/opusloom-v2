# No per-ticket budget; the attempt cap is the only cost guard

We considered a soft+hard cap (warn at 80% of $5, hard stop at $5,
per-ticket override). We chose to omit the feature for v1: the user is on
a token plan (predetermined token budget, not metered dollars), so a
runaway ticket burning dollars is not a real risk.

The `attempts: 5` cap plus the recycle delay is the only guardrail. This
bounds time, not money — and on a token plan, *time-bounded* also covers
*token-bounded* because the conversation grows linearly with turns
(which are time-bounded).

**Consequences.** If we ever switch to a metered plan, a per-ticket token
cap (e.g., 1M tokens backstop) becomes a v2 add. Recording `usage` per
ticket is still useful for reports; we expose it but don't hard-stop on
it.

**Re-open this ADR if:** model costs or token-plan structure change, or
we see patterns of runaway tickets in usage reports.
