# Don't decompose cross-persona features; assigned persona orchestrates

When a feature touches multiple personas, we do **not** decompose into
sub-tickets. One ticket stays the unit of work in the Kanban; the
assigned persona orchestrates the work directly, using `consult_persona`
(knowledge queries) or `request_persona_work` (sub-task delegation) to
pull in specialists.

**Why.** Single-persona tickets should stay single (no ceremony for
trivial work). Full-stack features benefit from decomposition in some
shapes, but **decomposition creates a tracking burden** (the user now
follows a Kanban card per persona, not per feature). The orchestrator's
role is to make the call, not to multiply tickets. The persona's
responsibility is the whole feature.

**Trade-offs.** Cross-persona delegation via tool calls is more complex
than sub-tickets (the persona decides when to consult vs do-it-itself).
Feature-level progress is harder to read ("which sub-task is this at?"
becomes "which conversation transcript is this at?"). For v1, the
Kanban shows individual tickets; a feature-level grouping doc is v2
(this proposal is *not* the epic/feature pattern).

**Consequences.** The Kanban stays clean (one card per feature). Each
persona's transcript accumulates cross-persona context as work happens.
When boundaries need explicit visibility, the persona's `core` says
"escalate to Backend via `consult_persona` for schema decisions."
