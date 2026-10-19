# OpusLoomV2

A Pi Durable-based agent harness with a kanban-driven SDLC, named
personas, and a Playwright-based story-runner for validating user
stories against a live UI.

## Shape

- **Tickets** are Pi Durable `defineTask`s; their phases are the SDLC
  states (Triage → Ready → Todo → InProgress → devServerReady → QA →
  UAT → Review → Complete, plus `Failed` / `Cancelled` / `Blocked`).
- **Orchestrator** is an always-alive agent conversation with system
  + read tools. Owns ticket intake, routing, and escalations.
- **Four personas** (Frontend, Backend, QA, Infra) are long-lived
  agent conversations with their own conventions docs and compaction
  strategies. Each is a specialist lead; cross-persona work is via
  `consult_persona` and `request_persona_work`.
- **Story-Runner** walks user stories against a per-ticket dev server
  during QA; sub-conversation owned by the ticket task.
- **Hybrid scheduler** (always-alive) dispatches build agents on a
  30s tick. Per-persona concurrency cap (default 2) + workspace total
  (default 3). Inline retry on QA fail with `attempts: 5` cap; auto-
  recycle on `Failed` with conversation reset.

## Stack

- **Runtime:** Node 24 + TypeScript (executed via `tsx`)
- **Backend:** Express + Pi Durable 1.0.x
- **Frontend:** Vue 3 (Composition API) + Vite
- **Storage:** SQLite via Pi Durable's `openNodeSqliteStorage`
- **Live updates:** Server-Sent Events (SSE) with commit coalescing
- **Testing:** Node's built-in test runner + scripted LLM

## Deployment

CLI launcher (`bin/opusloom.js`) supervises a Node server that serves
both the JSON API and the Vue 3 web app. One process, one port. Pi
Pocket's pattern.

## Status

Early. Design tree walked; code not started. See:

- [GLOSSARY.md](./GLOSSARY.md) — canonical vocabulary
- [docs/adr/](./docs/adr/) — 22 architectural decision records (one
  per significant design decision, sequenced)

## Working agreements

- `npm install` to set up.
- `npm run dev` to start the launcher + watch backend + Vite dev
  server.
- `npm test` to run the test suite.
- Source layout follows the Pi Pocket convention:
  - `bin/` — entry point
  - `src/launcher/` — CLI / supervisor
  - `src/server/` — Express + Pi Durable harness
  - `src/server/extensions/` — orchestrator + persona extensions
  - `web/` — Vue 3 frontend
  - `docs/adr/` — decision records
  - `GLOSSARY.md` — vocabulary
