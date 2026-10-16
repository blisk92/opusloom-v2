# A ticket is a Pi Durable `defineTask`

Three candidate shapes: a flat record (`TicketDoc` only), a ticket
mapping 1:1 to a Pi Durable *conversation*, or a ticket mapping 1:1 to a
Pi Durable *task*. We chose the third: each ticket is a `defineTask` with
its SDLC states as named phases; the ticket's working conversation is
owned by the task.

**Why.** Pi Durable's `defineTask` already provides phases, checkpoints,
ownership, abort cascades, sleep / signal-via-doc waits, and an
input / output / result lifecycle. The SDLC graph *is* a state machine with
phases — exactly what `defineTask` was built for. A flat record loses the
durability and the audit trail; a conversation-as-ticket loses the explicit
phase model.

**Consequences.** The ticket's checkpoint graph records every phase
transition (Triage → Ready → Todo → InProgress → devServerReady → QA → UAT →
Review → Complete, plus `Blocked ↔ Ready`, plus `Failed` and `Cancelled`).
The full list is in [GLOSSARY.md](../../GLOSSARY.md). The `TicketDoc` is the
*read model* the Kanban renders; the task graph is the *write model* that
drives transitions.
