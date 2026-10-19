# OpusLoomV2 — v1 Spec

Status: draft. The design tree is settled (see GLOSSARY.md and docs/adr/0001-…0023-…md). This spec operationalises v1: the end-to-end shape, what the user sees, the surface contracts, and the testing approach.

## Problem Statement

The user is building an AI agent harness that drives a software project through a structured software-development-lifecycle. Tickets are the unit of work; the system routes them to specialised agents ("personas"), supervises their execution through named lifecycle phases, validates UI changes against user stories, and recycles work that fails — all without a human in the loop for every step. Today there is no implementation; the design lives in `GLOSSARY.md` and 23 ADRs. The user wants v1 of this product, fast, against a single workspace, with a CLI launcher and a Vue 3 web UI on top.

The product must:

- Be open-source, public, and runnable on a single developer machine.
- Take a folder, scan it, configure itself, and start a Kanban-driven loop.
- Route tickets to four default personas (Frontend, Backend, QA, Infra) and let the user override.
- Validate UI changes against user stories via Playwright before promoting a ticket through QA.
- Survive process crashes, restarts, and recycle stale work.
- Show the system state on two graph pages: a System Dashboard (state machine + live counts) and a Per-Ticket Workflow (per-ticket phase graph).

## Solution

A Node 24 / TypeScript CLI launcher that supervises an Express server. The server embeds a Pi Durable `Harness` containing the orchestrator conversation, four persona conversations, and a long-lived scheduler task. Tickets are Pi Durable `defineTask`s whose phases are the SDLC states. The server exposes a JSON API and an SSE stream; the Vue 3 web app (built with Vite) is served as static assets from the same process.

A user opens the app against a folder. The wizard detects git/language/agent-context, then asks the user to confirm personas, pick a model per persona, and add credentials. From then on, the user describes features to the orchestrator's intake conversation; the orchestrator extracts structured `given/when/then` stories, creates the ticket, and routes it to a persona. The scheduler dispatches the assigned persona's build agent, which writes code, optionally starts a per-ticket dev server, and on QA spawns a story-runner sub-conversation that drives Playwright against the live UI. Findings are committed; the ticket transitions through Review or recycles on failure. The Kanban + System Dashboard + Per-Ticket Workflow views let the user watch, steer, and intervene.

The shape is recorded in `GLOSSARY.md` and the 23 ADRs. This spec names the user-facing surface, the API contract, the schema, and the testing approach that proves the system works end-to-end.

## User Stories

### Workspace and onboarding

1. As an owner, I want to open OpusLoomV2 against a folder, so that the system reads my project metadata.
2. As an owner, I want OpusLoomV2 to detect my git root, primary language, and existing agent-context files (CLAUDE.md, AGENTS.md), so that I don't have to enter that information by hand.
3. As an owner, I want a wizard that confirms the four default personas, so that I can disable any that don't apply to my project.
4. As an owner, I want to choose a model per persona, so that each persona runs on a model that fits its role.
5. As an owner, I want to enter model-provider credentials, so that the personas can call the LLMs.
6. As an owner, I want OpusLoomV2 to remember the last folder I opened, so that I don't have to pick it every launch.
7. As an owner, I want to see the workspace name, git branch, and detected language on the workspace settings page, so that I can verify the auto-detection.
8. As an owner, I want to set a project name, so that the workspace has a friendly title in the UI.
9. As an owner, I want to add additional project context (notes) that the orchestrator reads at intake, so that the system can apply project-specific conventions.

### Orchestrator intake and routing

10. As an owner, I want to type a feature description into the orchestrator's chat, so that a new ticket is created from my words.
11. As an owner, I want the orchestrator to ask me clarifying questions for ambiguous features, so that the resulting ticket is well-defined.
12. As an owner, I want every intake to extract `given/when/then` user stories, so that the story-runner can verify the UI.
13. As an owner, I want every ticket to carry a freeform description plus structured user stories, so that the story-runner has both context and a runnable contract.
14. As an owner, I want tickets with no user stories to skip the QA phase, so that backend-only work doesn't run Playwright.
15. As an owner, I want the orchestrator to suggest a persona for each ticket with a confidence score, so that I can sanity-check the routing.
16. As an owner, I want to drag a ticket from one persona's column to another's, so that I can override the routing after the fact.
17. As an owner, I want the system to surface the routing source (rules vs LLM) on each ticket, so that I know whether the decision was explainable.

### Kanban and ticket lifecycle

18. As an owner, I want to see all tickets in a Kanban view, so that I can see the project at a glance.
19. As an owner, I want to filter the Kanban by state, persona, priority, and label, so that I can focus.
20. As an owner, I want to see ticket count badges per column, so that I can see pressure.
21. As an owner, I want a Failed column, so that failed tickets are visible rather than dropped.
22. As an owner, I want a Cancelled column, so that deliberate stops are visible.
23. As an owner, I want a Blocked column, so that tickets waiting on dependencies don't appear in the active flow.
24. As an owner, I want a Scheduled column, so that tickets with a future start time are visible.
25. As an owner, I want to click a ticket to see its detail, so that I can read its description, stories, attempts, and conversation.
26. As an owner, I want ticket details to show the conversation transcript, so that I can see what the agent did.
27. As an owner, I want to see the build agent's tool calls in the transcript, so that I can audit decisions.
28. As an owner, I want to see story-runner reports inline in the transcript, so that I can see what UI checks ran and what passed.
29. As an owner, I want to see the current `attempts` count and `recycleAfter` setting, so that I know what will happen on the next failure.
30. As an owner, I want to see the full ticket history, so that I can audit every transition.
31. As an owner, I want to manually transition a ticket between states, so that I can intervene when needed.
32. As an owner, I want to set a ticket's priority to high, normal, or low, so that the scheduler honours urgency.
33. As an owner, I want to add a ticket to `blockedBy` other tickets, so that the scheduler respects dependencies.
34. As an owner, I want to set ticket preconditions, so that the scheduler waits for external readiness (API keys, manual approvals).
35. As an owner, I want to set a custom `attempts` value, so that a long-running ticket can have a higher retry budget.
36. As an owner, I want to set a custom `recycleAfter`, so that a hot ticket recycles faster.
37. As an owner, I want to set `recycleAfter: null` to opt a ticket out of auto-recycle, so that a Failed ticket waits for me.
38. As an owner, I want to see when the scheduler last ran and what it dispatched, so that I can debug.

### Scheduler and dispatch

39. As the scheduler, I want to wake every 30 seconds (configurable per workspace), so that I can pick up newly-runnable tickets without thrashing CPU.
40. As the scheduler, I want to check all five non-blocking gates (state, deps, preconditions, assignee, capacity) before dispatching, so that I never dispatch a ticket that will fail at runtime.
41. As the scheduler, I want to dispatch at most N tickets per persona and M total, so that resource ceilings are respected.
42. As the scheduler, I want to commit the dispatch as a single transaction, so that the build agent's conversation is consistent.
43. As the scheduler, I want to wait on `attempts` and `recycleAfter` to bound retry storms.

### Build agent (per persona)

44. As a persona, I want my conversation to persist across tickets, so that my accumulated knowledge survives.
45. As a persona, I want to receive ticket assignments via a `Ready` transition, so that the dispatcher doesn't bypass my role.
46. As a persona, I want to have access to general tools (read/write/edit/bash) plus my domain tools (browser for FE/QA, db for BE, deploy for Infra), so that I can do my work directly.
47. As a persona, I want to call another persona via `consult_persona`, so that I can pull in domain knowledge without spawning a sub-task.
48. As a persona, I want to call `request_persona_work` to delegate substantial work, so that complex cross-domain tickets make progress.
49. As a persona, I want to save lessons to my `ConventionDoc`, so that future me has prior knowledge.
50. As a persona, I want my compaction to be per-persona with a user-configurable threshold, so that I don't lose my context prematurely.
51. As a persona, I want my compaction to extract lessons into my `ConventionDoc` before summarising, so that lessons survive and remain reusable.
52. As a persona, I want to receive the story-runner's report inline in my transcript, so that I can act on findings.

### Story-Runner (QA)

53. As the system, I want to spawn a story-runner sub-conversation in the ticket's `QA` phase, so that the build agent can be re-entered on pass.
54. As the story-runner, I want a `browser` tool to drive Playwright, so that I can walk user stories.
55. As the story-runner, I want a per-ticket dev server URL passed in, so that I can `page.goto(...)` against the right host.
56. As the story-runner, I want to capture a structured finding per failed story (story index, expected, observed, screenshot, console log), so that the build agent can act on specifics.
57. As the story-runner, I want to use only browser + read tools, so that I can't accidentally edit code.
58. As the story-runner, I want my full transcript to be queryable post-run, so that post-mortems are possible.

### Failure handling and recycle

59. As the ticket task, I want to track `attempts` (default 5) and stop retrying once the cap is hit, so that I don't loop forever.
60. As the ticket task, I want to transition to `Failed` with a reason when the cap is hit, so that the owner can re-queue.
61. As the scheduler, I want to auto-recycle a Failed ticket after `recycleAfter` ms (default 1h), so that the work has another chance.
62. As the scheduler, I want to call `reset()` on the ticket's conversation on recycle, so that the next attempt starts fresh.
63. As the owner, I want to override the recycle decision (re-queue immediately, or pin as owner-only), so that I have control.

### Per-persona memory and conventions

64. As a persona, I want a `core` system-prompt section that doesn't drift, so that my role identity is stable.
65. As a persona, I want a `lessons[]` array in my `ConventionDoc`, so that I can accumulate dated, tagged notes.
66. As a persona, I want to retrieve lessons by tag, so that the right knowledge surfaces when relevant.
67. As an owner, I want to edit a persona's `core` and `lessons` in the UI, so that I can shape its behaviour.
68. As an owner, I want to see a persona's per-persona compaction threshold and edit it via a slider, so that I can tune memory.

### Cross-persona consulting

69. As Dana, I want to call `consult_persona({ persona: "Backend", question: "..." })` and have Sam's reasoning inline in my transcript, so that I can act on his guidance.
70. As Sam, I want to do the same to Casey, so that I get Infra's knowledge when I need it.
71. As a persona, I want my `core` and relevant `lessons` to be loaded into the consult's prompt, so that my answer reflects my accumulated knowledge.

### Graph UI

72. As an owner, I want a `/system` page that renders the SDLC state machine as a graph, so that I can see the system's structural shape.
73. As an owner, I want each state node to show the live count of tickets in it (SSE-driven), so that I can see system pressure at a glance.
74. As an owner, I want to click a state node to drill into the Kanban filtered to that state, so that the graph and Kanban connect.
75. As an owner, I want a `/tickets/:id` page that renders the ticket's `defineTask` phase graph, so that I can see where this ticket is in its lifecycle.
76. As an owner, I want the current phase highlighted, past phases marked done with summaries, and future phases dimmed, so that progress is unambiguous.
77. As an owner, I want the per-ticket page to also show attempts, recycle history, and conversation snippets per phase, so that the page is a full ticket ops surface.

### Configuration and ops

78. As an owner, I want to see the model list available to me, so that I can pick models that are valid.
79. As an owner, I want to add or change credentials for a model provider, so that the personas can call it.
80. As an owner, I want to see real-time token usage, so that I can monitor spend.
81. As an owner, I want to restart the server, so that I can recover from bugs.
82. As an owner, I want the server to recover from crashes, so that runs continue.

### v2 hooks (storage only — no behavior in v1)

83. As an owner, I want to see `Ticket.kind: "code" | "design"` on each ticket card, so that the field is exposed.
84. As an owner, I want a "Escalate to design" button on a Failed ticket, so that I can manually open a grilling sub-conversation.
85. As the system, I want to track `FailureClassDoc` on Failed tickets, so that v2 can cluster by fingerprint.
86. As the system, I want `recursable: boolean` on each ticket, so that v2 can opt tickets out of auto-recycle.

## Implementation Decisions

### Module breakdown (high level — concrete file paths are out of scope)

- **Entry point** — A Node entry that runs the launcher.
- **Launcher** — Reads options, picks access mode, supervises the server.
- **Server** — Express + Pi Durable harness; serves `/v1/*` JSON API + `/v1/events` SSE stream + the Vue static build.
- **Storage** — Pi Durable `openNodeSqliteStorage` against a single `pocket.sqlite` per workspace. Catalog of conversations, tasks, entries, and the app's documents.
- **Harness assembly** — One `Harness` containing the orchestrator, four persona conversations, and a long-lived scheduler task. `conversationCreated` callback creates per-conversation docs upfront.
- **Extensions** — Five `defineExtension`s: orchestrator, four personas, and a `core` extension providing the build/QA/review task machinery and the agent-runtime.
- **Tools** — `consult_persona`, `request_persona_work`, ticket-management tools, story-runner tools, `save_lesson`, `consult_lessons`, `escalate_to_design` (v1 stub).
- **Web app** — Vue 3 + Vite, structured around routes: `/` (Kanban), `/system` (System Dashboard), `/tickets/:id` (per-ticket detail + workflow graph), `/settings` (workspace, personas, models, credentials).
- **Graph UI** — `vue-flow` for both `/system` and `/tickets/:id`'s graph panel. Shared topology file.
- **Launcher** — Pi Pocket's pattern: access menu (local/LAN/Cloudflare/Tailscale), supervisor restart on crash, child process IPC.
- **CLI** — `bin/opusloom.js` is the node entry; the launcher reads its options from argv + env vars.

### SDLC state machine (decoded from GLOSSARY.md)

The thirteen states with their allowed transitions, encoded as the canonical data the harness and graph UI share:

```
Triage → Ready                       (orchestrator routes, gates pass)
Ready ↔ Blocked                      (toggle on dependency changes)
Ready → Todo                         (orchestrator promotes to active board)
Ready → Scheduled                    (orchestrator defers with a future startAfter)
Scheduled → Ready                    (clock hits startAfter; same path as above)
Todo → Ready                         (re-entry; same path)
Todo → Scheduled                     (re-entry; same path)
Ready/Todo → InProgress              (scheduler dispatches; ownership = task)
InProgress → devServerReady          (build agent claims done; spawn dev server)
devServerReady → QA                  (dev server healthy; spawn story-runner)
QA → UAT                             (story-runner reports pass)
QA → InProgress                      (story-runner reports fail; attempts++)
UAT → Review                         (human gate passes; PR opened)
Review → Complete                    (PR merged)
{any non-terminal} → Failed          (attempts cap hit)
{any non-terminal} → Cancelled       (owner override)
Ready/Todo → Ready/Todo              (auto-recycle; reset conversation)
```

The transition authority is mixed per ADR 0003: agent transitions, owner transitions, harness-event transitions (CI, PR merged, clock). Every transition is a committed entry to the `TicketDoc`; the audit log is the `pocket.live` + `TicketDoc` history.

### Storage schema (decoded from GLOSSARY.md)

- **`TicketDoc`** (`pocket.tickets`, session-scoped) — One entry per ticket; the field set in GLOSSARY.md's "Ticket Field" table.
- **`ConventionDoc`** (`persona.conventions`, session-scoped) — Per-persona `core` + `lessons[]`.
- **`FailureClassDoc`** (`pocket.failure_class`, session-scoped) — Per-Failed-ticket `{fingerprint, count, lastSummary, reproCount, escalated}`. v1 does not author; v2 will.
- **Per-ticket `DevServerDoc`** (task-scoped) — `{url, port, pid}` for the per-ticket dev server.
- **Per-conversation per-ticket `pocket.task`** — the ticket itself as a `defineTask`; its phases are the SDLC states.

### API contract

REST over `/v1/*` (Pi Pocket's URL shape). Authentication: bearer token (Tailscale, LAN, or Cloudflare-quick-tunneled). Commands use `requestId` for exactly-once; an in-memory or short-lived-TTL cache dedupes retries. Routes follow Pi Pocket's conventions:

- `GET /v1/device` — server metadata, model list, capability flags.
- `GET /v1/tree` — the workspace tree (sessions, agents).
- `GET /v1/events` — SSE stream: `hello → snapshot → deltas`. Frames are `{type, ...}` discriminated unions (durable-pi's pattern).
- `GET /v1/sessions`, `POST /v1/sessions` — list / create sessions.
- `GET /v1/sessions/:s/agents`, `GET /v1/sessions/:s/agents/:a` — list / detail agents.
- `GET /v1/sessions/:s/agents/:a/events` — per-agent SSE (deltas + docs).
- `POST /v1/sessions/:s/agents/:a/messages` — submit a user message. Body: `{content, requestId, whenBusy}`. Optional `waitMs` for synchronous wait.
- `POST /v1/sessions/:s/agents/:a/abort`, `.../fork`, `.../reset`, `.../compact` — the canonical Pi Durable operations.
- `PATCH /v1/sessions/:s/agents/:a/config` — change model, thinking level, tools, instructions.
- `PATCH /v1/sessions/:s/agents/:a/extensions` — turn extension features on/off.
- `GET /v1/sessions/:s/agents/:a/docs/:kind` — read a doc (TicketDoc, ConventionDoc, etc.).
- `PATCH /v1/sessions/:s/agents/:a/docs/:kind` — write a doc.
- `GET /v1/sessions/:s/agents/:a/tasks` — list the ticket's task graph (live + terminal).
- `POST /v1/sessions/:s/agents/:a/tasks/:taskId/abort` — abort a task.
- `POST /v1/sessions/:s/agents/:a/attachments/upload` — file uploads for the conversation.

Live updates: SSE per-agent + per-workspace coalesced at 250ms (Pi Pocket's window). The Vue app subscribes; `coalesce` throttles; `sendIfChanged` dedupes.

### Frontend routes

- `/` — Kanban (default landing).
- `/system` — System Dashboard (graph + counts).
- `/tickets/:id` — Per-Ticket Workflow (graph + transcript + actions).
- `/settings` — Workspace, personas, model providers.
- `/pip` (or `/assistant` — name TBD) — Orchestrator chat. The owner talks to the orchestrator here for intake and ad-hoc design questions.
- `/people` (v1: just the owner) — Auth and access.

### Key technical clarifications

- **Live updates use SSE, not WebSocket.** SSE matches the references (Pi Pocket, durable-pi) and is enough for one-way commit-stream delivery. Reconnect-and-replay is the standard pattern; Pi Durable's `watchEvents` gives snapshot + deltas; a fresh SSE connection re-reads the current view.
- **The Vue app is plain ES modules served as static assets.** No build step beyond `vite build` (we *do* build, unlike Pi Pocket — Vue SFCs need a build). Vite is dev-server-aware for HMR.
- **Compaction thresholds map mechanically to Pi Durable's `reserveTokens` / `backgroundTokens`.** No Pi Durable API change; UI knob over an existing primitive.
- **The "Escalate to design" button opens a new sub-conversation with the orchestrator as the parent.** It uses the same `Harness.commit` + `defineTask` machinery as a normal ticket; the "design" kind marker distinguishes it for v2.
- **Per-ticket worktrees** — At `InProgress` transition, the ticket task runs `git worktree add` into `.opusloom/worktrees/<project>/<agent>/<ticket-id>/` (the pattern from OpusLoom V1). The build agent's `cwd` is the worktree. The PR step pushes the branch; `git worktree remove` cleans up on `Complete` or `Cancelled`.
- **Story-runner spawn** — At `QA` phase, the ticket task creates a child conversation with `ownership: { kind: "task", taskId: ticket.id }`, configured with `browser.*` + `read` only on a cheap vision-capable model. The ticket task's checkpoint at QA holds the dev server URL.
- **Per-persona compaction** — Each persona's extension installs a custom `compaction` settings object whose instructions prioritise role-specific extractions. Two-stage: extract → summarize.

### Architectural clarifications (carried from ADRs)

| Decision | ADR |
|---|---|
| Fresh on Pi Durable | 0001 |
| Ticket = `defineTask` | 0002 |
| Mixed transition authority | 0003 |
| Hybrid scheduler + 5-gate predicate + defaults | 0004 |
| Auto-recycle with reset; `attempts: 5`; per-ticket overrides | 0005 |
| No per-ticket budget (token plan; attempts is the guard) | 0006 |
| Orchestrator: system + read tools | 0007 |
| Intake always via orchestrator | 0008 |
| Ticket content: description + userStories; empty skips QA | 0009 |
| Story-runner as sub-conversation owned by ticket task | 0010 |
| Per-ticket dev environment | 0011 |
| Four default personas | 0012 |
| Auto-route with user override | 0013 |
| Don't decompose cross-persona | 0014 |
| Hybrid persona tool surface | 0015 |
| Hybrid convention doc (inline core + lazy lessons) | 0016 |
| `consult_persona` in-transcript | 0017 |
| Per-persona two-stage compaction | 0018 |
| Onboarding: auto-detect + wizard, single workspace v1 | 0019 |
| Recursive grilling v1 hooks | 0020 |
| CLI + thin web UI | 0021 |
| Node 24 + TypeScript + Vue 3 | 0022 |
| Graph UI: System Dashboard + Per-Ticket Workflow | 0023 |

## Testing Decisions

### What makes a good test

- **Test the external contract, not the implementation.** Tests should hit the HTTP API or the rendered UI; the harness's internal task graph is implementation.
- **Tests are scripts that a future agent can read and follow.** Every test that exercises the SDLC lifecycle has a one-paragraph description of what user story it's covering.
- **A test fails for exactly one reason.** No flaky concurrency, no race-condition dependencies, no time-dependent assertions.
- **No model calls in unit tests.** A scripted LLM fakes model output deterministically. The real model only runs in the end-to-end seam (or in a manual "smoke" run).

### Test seams (one primary seam; the rest is coverage, not new seams)

- **Primary: end-to-end `test/serve.ts` seam.** A single Node script that:
  1. Creates a temp dir.
  2. Boots OpusLoomV2 in test mode against that dir, with a scripted LLM and Chromium.
  3. Walks the SDLC through one full ticket lifecycle, asserting at each phase:
     - Workspace open + wizard + persona selection + model selection.
     - Orchestrator intake creates a ticket with `description` + `userStories[]`.
     - Scheduler dispatches at most the configured concurrency.
     - Per-ticket phases transition in order: `InProgress → devServerReady → QA → UAT → Review → Complete`.
     - Story-runner sub-conversation spawns and reports findings.
     - Failed ticket auto-recycles after `recycleAfter`; second attempt succeeds; ticket completes.
     - System Dashboard renders the state machine with correct live counts.
     - Per-Ticket Workflow renders the per-ticket phase graph with the current phase highlighted.
  4. Tears down the temp dir.

  This is the gold-standard seam: it covers user stories 1-86 in one script.

- **Coverage under the same seam (not new seams):**
  - `sdlc.ts` topology: pure function — assert that a known set of states connects with the expected edges.
  - `ConventionDoc` serialization: pure function — assert that `core` + `lessons[]` round-trips through storage.
  - HTTP API per route: the seam *is* the API; tests are part of the primary seam.
  - Vue components in isolation: Vue Test Utils + jsdom. Part of the primary seam's render assertions; not a separate integration test.

### Prior art

- **Pi Pocket** has a similar end-to-end pattern (`test/serve.ts` with scripted model + real Chromium) and is the immediate reference.
- **durable-pi** has unit + UI tests at the API + component level; we follow that style for `sdlc.ts` and Vue components, but the dominant seam is the end-to-end one.

## Out of Scope

v1 explicitly does *not* include:

- **Multi-workspace switcher** — single workspace per process. Multi-workspace is launching multiple OpusLoomV2 instances. (ADR 0019)
- **Multi-user** — single owner v1. Roles/invites are v2.
- **Recursive grilling automation** — v1 has the storage hooks (`Ticket.kind`, `FailureClassDoc`, `recursable`) and the manual "Escalate to design" button. Fingerprint clustering and automatic escalation are v2.
- **GitHub issue auto-import** — Intake is via the orchestrator's chat only. GitHub auto-import is v2.
- **Push notifications** — v1 has no notification channel. Push is v2.
- **Recurring ticket templates** — v1 intake is one-ticket-per-conversation. Templates are v2.
- **Per-ticket budget (dollar cap)** — Token plan + `attempts` cap is the only cost guard. (ADR 0006)
- **Tauri desktop shell** — CLI launcher + browser UI. Tauri is rejected for v1. (ADR 0021)
- **Voice / TTS** — No voice in v1.
- **Custom persona templates per stack** — Four default personas cover most stacks. Stack-specific templates are v2.
- **Aging priority (anti-starvation)** — Explicit `priority` field with FIFO within. Aging is v2.
- **Auto-promotion of lessons to core** — Lessons stay in `lessons[]`; manual promotion is via the persona editor. (ADR 0016)

## Further Notes

- **Solo dev, public repo, "build quick"** — the design tree is settled; v1 is a build sprint, not a discussion. Once tooling is in place, expect commits landing daily or faster.
- **Pi Durable 1.0.x is the only version tested against.** When a new version lands, the install path needs re-validation; nothing in the design is version-locked except for the API shape.
- **Tests in v1 are written alongside features, not after.** The "test seam" in this spec is the contract every feature commits to; a feature lands with its end-to-end coverage.
- **Tickets within OpusLoomV2 are tracked in the project's chosen issue tracker (TBD; see top of this spec).** When the user sets one up, the v1 spec is sliced into per-area tickets and each ticket's implementation lands the corresponding test-coverage update.
- **The "build quick" instinct is the design's friend.** When a feature would be nice but isn't a user story here, defer it to v2. The user stories in this spec are the v1 surface; everything else waits.
