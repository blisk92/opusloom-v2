# Base on Pi Durable, not a Pi Pocket fork

We evaluated (a) forking Pi Pocket, (b) basing on durable-pi, and (c) starting
fresh on `@earendil-works/pi-durable` directly. We chose **(c)**: OpusLoomV2 is
its own application built on the Pi Durable primitives directly, because the
Kanban/SDLC is the *center* of the system rather than an appendage to a chat
interface. We borrow selectively — Pi Pocket's `reload.ts` and commit
coordinator patterns; durable-pi's `jobs.ts` (durable tasks with phases as
checkpoints) and `streams.ts` (coalesced SSE) — but neither repo's structure
fits a ticket-driven workflow.

**Consequences.** The codebase doesn't inherit Pi Pocket's chat-first,
mobile-first surface or its per-person-collab accounting. durable-pi's
device-network scaffolding is a v2+ feature, not v1. We implement what we
need against Pi Durable's `defineDoc`, `defineTask`, `defineTool`,
`defineExtension`, and the conversation/task tree primitives.
