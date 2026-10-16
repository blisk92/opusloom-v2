# Hybrid scheduler; non-blocking predicate; tick/priority/capacity defaults

Three related decisions on dispatch:

**Hybrid scheduler (R1-Q4).** The scheduler is an always-alive durable
task that wakes every tick, scans the `TicketDoc`, and dispatches build
agents. User-triggered work remains possible via the orchestrator
manually assigning a ticket, but the default is scheduler-driven.

**Non-blocking predicate (R2-Q11).** A ticket is runnable iff all five
gates pass: state ∈ {Ready, Todo, Scheduled-due}; every entry in
`blockedBy` is `Complete`; every entry in `preconditions` has `met: true`;
`assignee !== null`; active-agent count ≤ `maxConcurrentAgents`.

**Defaults (R3-Q5).** Tick = 30s (workspace-configurable, default 30).
Priority = explicit field with FIFO within (high / normal / low; default
normal). Capacity = per-persona cap (default 2) plus workspace total cap
(default 3). Story-runner is browser-heavy; the per-persona cap protects
resource ceilings. The workspace total is a backstop.

**Consequences.** Runnable tickets dispatch within ~30s of becoming
ready. The scheduler can crash and resume as a regular durable task.
Priority field gives users an explicit knob for "this is hot." Aging
priority (anti-starvation by time-since-creation) is v2+.
