# Per-ticket dev environment: each ticket owns a dev server

The ticket task's `devServerReady` phase starts a dev server (`npm run
dev`, `cargo run`, etc.) in the ticket's worktree, isolated port,
isolated state. The server is polled until healthy, recorded on a
`DevServerDoc` (`{url, port, pid}`), then passed to the story-runner.
The ticket task's `abort` handler shuts it down (bottom-up cleanup).

**Why.** Isolation — one ticket's hot reload doesn't clobber another's
test target. Speed — local cold-start in 5–30s vs CI's minutes.
Determinism — clean state per ticket; no drift from sibling tickets.
Lifecycle is a phase; no separate process manager needed.

**Trade-offs.** Per-ticket cold-start latency (~10–30s before QA can
begin); port management trivial via `findFreePort()`. State isolation
is a v1 simplification (each ticket gets its own dev DB or snapshot —
complex real-world state is a v2 problem).

**Consequences.** For non-UI tickets (empty `userStories`), the
`devServerReady` phase is bypassed entirely. No dev server, no cost.
