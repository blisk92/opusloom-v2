# Per-persona two-stage compaction; auto-compaction percentage set at persona level

Each persona compacts **on its own schedule** with **its own
extraction directives**. The compaction runs in two stages: stage 1
summarizes the oldest portion of the conversation; stage 2 extracts
durable lessons into `ConventionDoc.lessons[]` and stable rules into
`ConventionDoc.core` (replacing it), then the conversation truncates.

**Why.** Different roles need different memory. The orchestrator
preserves decisions and unresolved asks; Dana preserves UI patterns;
Riley preserves recurring failure modes; Casey preserves deploy
gotchas. One universal summarization loses role-specificity. Two-stage
extraction means lessons survive *and* are reusable across tickets
(via `consultLessons`); without extraction, lessons live only in
compressed form, churning each compaction.

**Compaction threshold is per-persona, configurable in the UI.**
Different models have different context behaviors; the threshold
matches the model's needs. No project-level override in v1 — the user
noted models are persona-scoped, so compaction follows the model.

**Compaction UI:** each persona's editor page has a "compact when X%
full" slider (50–95%, default 80%). Maps to Pi Durable's
`reserveTokens` and `backgroundTokens`. Mapping is mechanical:
`reserveTokens = tokens * (1 - percent/100)`,
`backgroundTokens = reserveTokens - tokens*0.10`.

**Consequences.** Each persona has its own `compaction_extraction`
directive (a 5-line prompt). The LLM call that does extraction is the
same role's model. Persistent docs (`ConventionDoc`) grow over time;
conversation transcripts shrink. Tickets can survive context overflow
without losing decisions.
