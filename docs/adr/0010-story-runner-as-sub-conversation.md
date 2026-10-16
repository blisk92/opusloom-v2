# Story-runner runs in a sub-conversation owned by the ticket task

When a ticket enters `QA` phase, the ticket task spawns a child
conversation with `ownership: { kind: "task", taskId: ticket.id }`,
configured with `browser.*` plus `read` only, on a (preferred) cheaper
vision-capable model. The build agent's conversation pauses waiting on
the subagent's settlement. The subagent returns a structured report;
the parent ticket transitions accordingly.

**Why.** Role separation enforced structurally: build agent has full
coding tools; story-runner has only browser + read. Different models per
role (build agent = expensive code-fluent; story-runner = cheap
vision-capable). Failure isolation — a story-runner crash doesn't lose
build agent state. The conversation tree matches Pi Durable's existing
patterns (durable-pi's `Ops.openIncident`, Pi Pocket's `subagent` tool).

**Consequences.** Build agent only sees the *report* of the story-runner,
not its inline reasoning. That's intended: the build agent acts on
findings, not on debug trails. Story-runner transcripts persist in
storage as audit, not as live context.
