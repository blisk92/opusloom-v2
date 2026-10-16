# Mixed transition authority: agents, humans, and harness events

Tickets move between states via three authorities: the assigned agent
calling a transition tool, a human editing the Kanban, and the harness
itself reacting to events (a CI webhook, a PR being merged, a scheduled
time arriving, a deadline expiring).

**Why.** Pure agent-only (a) means humans can't intervene — defeats the
"watch it work" use case. Pure human-only (b) means autonomous picking is
impossible. Mixed authority matches the precedents: durable-pi's
`DeployTask` polls a committed `Deploys` doc for `approved/declined`; Pi
Pocket's `watches.ts` polls GitHub for CI status and updates the watch's
status; Pi Pocket's `schedules.ts` fires a message at a scheduled time.

**Consequences.** Every transition is a committed entry to the `TicketDoc`.
The audit log shows who/what moved which ticket, when, and why.
