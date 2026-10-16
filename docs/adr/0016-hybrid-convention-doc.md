# Hybrid convention doc: inline core + lazy-loaded lessons

A persona's knowledge lives in two places. **Core** — the persona's
stable, prompt-baked knowledge (domain, voice, ground rules) — is
inline in the persona's `system_prompt_section`, auto-loaded every turn.
**Lessons** — discrete, dated, tagged notes (`{id, text, tags,
references, ticketIds, writtenAt}`) — live in a `persona.conventions`
doc (`ConventionDoc`); the persona calls `consultLessons(topic)` to
retrieve by tag when relevant, and `saveLesson(text, tags)` to persist.

**Why.** Core conventions are non-negotiable identity — burying them in
tool calls means the persona drifts. Lessons accumulate forever; context
is bounded. Promotion-by-reference-count is v2 (referenced 3+ times →
auto-promote to core); v1 has no automatic extraction.

**Lesson writes are explicit.** The persona calls `saveLesson` when it
judges something worth remembering. The orchestrator validates and
writes. No background extraction in v1.

**Consequences.** Lesson writes produce committed `ConventionDoc` entries
(queryable in `search_history`). A persona's `core` is in the prompt,
auditable; lessons are in storage, also auditable. Multi-project
overrides apply (`ConventionDoc.projectId` field).
