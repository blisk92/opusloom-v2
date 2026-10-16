# Auto-recycle failed tickets after a delay; reset conversation on recycle; per-ticket overrides

When a ticket hits `attempts` (default 5), it auto-transitions to `Failed`
(non-sticky). The scheduler waits `recycleAfter` ms (default 1h,
configurable per-ticket), then transitions the ticket back to `Ready`/
`Todo` for re-dispatch. On recycle, the ticket's conversation is `reset()`-
ed so the next attempt starts fresh; old attempts remain in storage
(queryable via `search_history`).

**Why.** "All tasks are critical to the project" was the user's stated
principle. Sticky `Failed` violates that. Auto-recycle + delay respects
it without thrashing (the delay is long enough that codebase state
probably evolved before retry). Reset on recycle gives the next cycle a
clean shot rather than inheriting a transcript full of broken-fix
history. Per-ticket overrides (`recycleAfter: number | null`,
`resetOnRecycle: boolean`) absorb the long tail.

**Between attempts and on `Failed`: transcript untouched.** Symmetry with
recycle: keeping the transcript throughout lets human override see the
full audit trail and lets subsequent agents query historical reasoning.
Reset is the deliberate action, on recycle.

**Consequences.** Failed tickets never silently disappear. Storage
grows with abandoned attempts (acceptable — queryable). A ticket with
`recycleAfter: null` is owner-only on failure; humans must re-queue.
