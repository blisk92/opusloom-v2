# OpusLoomV2

A Pi Durable-based agent harness with a kanban-driven SDLC, named personas,
and a Playwright-based story-runner that walks user stories against a live UI.
Single-workspace v1; multi-workspace switcher is v2+.

## Roles

**Owner**: The human user. Talks to the orchestrator. Single-user in v1.

**Orchestrator**: An always-alive agent conversation owned by the harness. The
owner's design partner: creates tickets from intake, routes them to personas,
handles escalations. Has system + read tools. Cannot run code.

**Scheduler**: An always-alive durable task. Every ~30s, scans the TicketDoc
for runnable tickets and dispatches build agents. Per-workspace + per-persona
concurrency caps.

**Persona**: A long-lived agent conversation named after a domain role
(Frontend / Backend / QA / Infra). Owns its core conventions, accumulated
lessons, toolset, and compaction strategy.

**Build Agent**: The persona's working memory during a ticket's `InProgress`
phase. The persona's transcript grows across tickets. Per-ticket overlays
that are reset on retry are persisted to the ticket doc, not erased.

**Story-Runner**: A sub-conversation owned by a ticket's task, spawned only
during the ticket's `QA` phase. Walks user stories against the ticket's dev
server using Playwright. Reports pass/fail findings back to the parent
ticket task via committed entries.

**Subagent** (Pi Durable term): any agent conversation created as a child
of another. In OpusLoomV2, story-runners are subagents of their ticket task.

## Tickets

**Ticket**: The unit of work. Each ticket is a Pi Durable `defineTask`
with named phases; the SDLC graph is the phase machine. Tickets are durable —
a process crash resumes from the last checkpoint.

**Ticket State**: The current phase. Distinct from `kind`, `priority`, or
the ticket's `result`. The complete value set:

| State | Meaning |
|---|---|
| `Triage` | Entry; orchestrator intake conversation is active. |
| `Ready` | Non-blocking; gates pass; eligible for the scheduler. |
| `Todo` | Explicitly on the active board; eligible. |
| `Scheduled` | Has a future `scheduledAt`; not eligible until then. |
| `Blocked` | Open dependencies (`blockedBy` non-empty); not eligible. |
| `InProgress` | Build agent's owned conversation is open. |
| `devServerReady` | Per-ticket dev server healthy. |
| `QA` | Story-runner walking user stories; skipped if `userStories` is empty. |
| `UAT` | Human acceptance gate. |
| `Review` | PR open, awaiting merge. |
| `Complete` | Merged to default branch and post-merge checks pass. |
| `Failed` | Terminal of one run; recycled by scheduler after `recycleAfter` ms. |
| `Cancelled` | Terminal; deliberate stop. |

_Avoid_: `Running` (overloaded), `Merging` (transient).

**Ticket Doc** (`pocket.tickets`, session-scoped): The committed state of
every ticket in the workspace. Single source of truth that the scheduler
reads and the UI renders. The ticket's `defineTask` is the write model;
the doc is the read model that the Kanban projects.

**Ticket Field**: A persisted property on a ticket; conventions:

| Field | Type | Notes |
|---|---|---|
| `title` | string | Display title. |
| `description` | string | Freeform context. |
| `userStories` | `{given, when, then}[]` | Runnable contract for story-runner. |
| `assignee` | `string \| "system"` | Who owns this ticket's work. |
| `blockedBy` | `string[]` | Ticket IDs that must be `Complete` first. |
| `preconditions` | `{kind, met}[]` | External readiness checks. |
| `priority` | `"high" \| "normal" \| "low"` | Default `"normal"`. |
| `state` | `TicketState` | Mirror of the ticket task's current phase. |
| `kind` | `"code" \| "design"` | Default `"code"`. v2 hook. |
| `recursable` | `boolean` | Default `true`. |
| `branch`, `prUrl` | `string \| null` | Git metadata. |
| `labels` | `string[]` | Bookkeeping. |
| `attempts` | `number` | Default `5`. Per-ticket override. |
| `recycleAfter` | `number \| null` | ms; `null` = owner-only override. |
| `createdAt`, `updatedAt` | `number` | Bookkeeping. |

**Ticket Kind**: `"code"` (default) or `"design"`. Reserved for v2
recursive grilling; v1 marks every ticket as `"code"` and the field is
mostly bookkeeping for v2 hooks.

**Recursable**: A `boolean` ticket field indicating whether a failed ticket
may auto-recycle into another run. Default `true`. Setting to `false`
makes a ticket "owner-only" on failure.

**Feature** (optional): A grouping doc (`pocket.features`) for tickets
that came from a single intake request. v1 mostly doesn't need it (v1
promises one-ticket-per-intake-feature per R2-Q14); v2 may use it for
multi-ticket decomposition or recursive-grilling epics.

**Precondition**: A `{kind: string, met: boolean}` entry on a ticket.
External readiness check (e.g., `{"kind": "api-key", "met": true}`).
The scheduler requires `preconditions.every(p => p.met)` before dispatch.

**User Story**: A structured `{given, when, then}` triple on a ticket. The
story-runner walks each story in order; failures accumulate into one
report per QA pass.

**Failure Class**: A `{fingerprint, count, lastSummary, reproCount, escalated}`
record that v2 uses to cluster recurring failures for the recursive-grilling
feature. v1: the field exists on `FailedTicketDoc` but is not populated or
read actively.

## Worktrees and execution

**Worktree**: A per-ticket git worktree directory. Tickets get their own
branch + working directory; tools run inside the worktree, never against
the user's main checkout.

**Dev Server**: A per-ticket dev server started during the ticket task's
`devServerReady` phase. Stopped by the ticket task's `abort` handler
(bottom-up cleanup) or on task completion.

## Conversational structure

**Conversation Tree**: The hierarchy of agent conversations owned by the
harness. Top: orchestrator (always-alive). Children: personas (long-lived),
build agents (transcripts), story-runners (sub-conversations of ticket tasks).

**Consult**: A tool call to another persona for a knowledge question. The
consultant's reasoning runs inline in the asker's transcript, with the
consultant's `core` conventions + relevant lessons loaded for the consult.
Different from `request_persona_work`, which spawns a sub-task.

**Work Hand-off**: When a delegated task is too large for a consult, the
persona spawns `request_persona_work`, opening a sub-task assigned to the
target persona. The target returns a structured report.

## Compaction and memory

**Core Conventions**: A persona's stable, prompt-baked knowledge (domain,
voice, ground rules). Inline in the persona's `system_prompt_section`.
Per-persona — different roles need different framings.

**Lesson Doc**: A `{id, text, tags, references, ticketIds, writtenAt}`
entry on a persona's `ConventionDoc`. Tags drive retrieval
("ui:modal", "vue:composition-api", etc.). A lesson's `references` count
is incremented each time the persona calls it via `consultLessons(topic)`.

**Convention Doc** (`persona.conventions`, session-scoped): A doc holding a
persona's `core` string + `lessons[]`. Project can override.

**Recycling**: When a ticket hits `Failed`, the scheduler waits `recycleAfter`
ms, then transitions the ticket back to `Ready`/`Todo`. Default 1h. The
ticket's conversation is `reset()`-ed on recycle; old attempts stay in
storage (queryable via `search_history`). Per-ticket `recycleAfter: null`
opts the ticket out of auto-recycle.

## Routing

**Routing** (intake): The orchestrator's automatic assignment of an inbound
ticket to a persona. Rules first (cheap field match), LLM as fallback,
confidence surfaced in the Kanban card, user-overridable by drag/change.

## v2 deferred

**Recursive Grilling**: v2 feature — failed tickets cluster by fingerprint,
a classifier proposes "this is a design-class failure", and a design-grilling
session opens to write ADRs + fresh tickets. v1 ships only the storage hooks
(`FailureClassDoc`, `recursable`, manual "Escalate to design" UI button).

## Avoid

- **`Running`** — overloaded. Use `InProgress` for active ticket work.
- **`Merging`** — transient action, not a stable state. Use `Review`.
- **`Subagent`** in conversation logs — prefer the role name (`story-runner`).
- **`Agent`** alone — be specific (`build agent`, `story-runner`, `persona`).
