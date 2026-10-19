# Stack: Node 24 + TypeScript (backend), Vue 3 + Vite (frontend)

The implementation stack is locked as:

- **Runtime:** Node 24 LTS (latest LTS at build time; pinned via `.nvmrc`).
  TypeScript executed via `tsx` for dev, `tsc` for build.
- **Backend framework:** Express for HTTP (proven in Pi Pocket +
  durable-pi). No heavier framework; SSE streams are native `EventSource`.
- **Frontend framework:** Vue 3 (Composition API) + Vite. UI
  components render the Kanban, persona editor, ticket detail,
  conversation view.
- **State on the wire:** JSON over REST for commands; SSE for live
  updates (commit-coalesced; pattern from durable-pi's `streams.ts`).
- **Storage:** Pi Durable's `openNodeSqliteStorage` against a single
  `pocket.sqlite` per workspace.
- **Build / tooling:** `tsx` (dev), `tsc` (types), `vite` (frontend
  build), `eslint` + `prettier` (style). No ORM; no extra abstractions.
- **Testing:** Node's built-in test runner + a scripted LLM for
  harness tests (Pi Pocket's `test/serve.ts` pattern).

**Why.** Every choice is a default that has worked in the references
(Pi Pocket, durable-pi) and minimizes custom infrastructure. Solo dev,
"build quick" — every layer is a default that doesn't need justifying.
Bun was considered for runtime; Node 24 is the LTS Pi Durable is
tested against. React was considered for the frontend; Vue is what you
picked, and Vue 3 + Vite is the modern default.

**Consequences.** No Pi Durable or Express replacement. No TypeScript
language-server gotchas. Standard tooling means a fresh checkout is
`npm install && npm run build && npm start` to running. Hot reload via
Vite for the frontend; tsx watch for the backend.
