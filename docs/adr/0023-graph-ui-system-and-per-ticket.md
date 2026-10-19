# Graph UI: System Dashboard + Per-Ticket Workflow, on two pages

The Vue 3 web app exposes two graph views, both backed by the same SDLC
state machine but rendering at different scopes:

**Page 1 — System Dashboard (`/system`).** Operational view: nodes are
ticket states, edges are valid transitions. Each node shows the count
of tickets currently in that state, live via SSE. Failed / Blocked
states are visually distinct (color or icon). Clicking a node opens
the Kanban filtered to that state. Replaces the traditional "ops
dashboard" with a structural view of the system.

**Page 2 — Per-Ticket Workflow (in `/tickets/:id`).** Each ticket's
detail page mounts a graph panel showing the ticket's `defineTask`
phases. Each phase is a node; edges are the SDLC flow. The current
phase is highlighted; past phases show "done" markers with their
checkpoint summaries; future phases are dimmed. Sibling panel: per-
phase conversation snippets, attempts counter, recycle history.

**Why.** The state machine is the architectural shape of the system.
Operators and users need two views of it: "what's the system doing?"
(aggregated, by state) and "where is this ticket?" (focused, by
phase). Two pages, two URLs, two data fetches. The shared topology
keeps the mental model consistent.

**Library: `vue-flow`.** Vue 3-native, MIT-licensed, mature. Provides
nodes/edges/handles API that maps cleanly to both views. Same
primitives render the state machine on page 1 and the per-ticket
phase graph on page 2 — only the data and the node renderer differ.

**Alternatives considered.**
- **D3.js** — more flexible, more code. Rejected for v1 (overkill for
  two views; brings 80kb+ for a small surface).
- **Hand-rolled SVG** — most control, most work. Rejected for v1
  (no time-to-market win over vue-flow).

**Consequences.** Adds vue-flow as a runtime dep (~30kb gzipped). The
state-machine topology lives in a single TS file (`web/src/sdlc.ts`)
that both views import — no duplication. Page 1 is essentially a
static rendering of the topology with live counts; page 2 hydrates
the topology with per-ticket data from Pi Durable's task graph.
