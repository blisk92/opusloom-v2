# Ship four personas by default: Frontend, Backend, QA, Infra

The default persona set is **four**: `Frontend` (Dana), `Backend`
(Sam), `QA` (Riley), `Infra` (Casey). Each is a long-lived conversation
owned by the orchestrator with its own conventions doc, toolset, and
compaction policy. The story-runner role is subsumed into Riley — story-
running is a persistent persona with accumulated test heuristics, not an
ephemeral agent.

**Why.** Specialization has visible benefits. Frontend work keeps past
UI-decision memory; Backend keeps schema/library decisions; Infra keeps
deploy/runbook context; QA keeps test patterns and recurring failure
modes. The persistent-memory argument is *stronger* when personas are
specialized than when they're generalist. QA's persistence pays: Riley
accumulates "story patterns that consistently broke" across tickets.

**Consequences.** Four conversations to manage (one per persona), four
compaction policies, four model specs. Substantial overhead, but matches
the four natural divisions in the SDLC. Adding more personas
(multi-user, project-keyed, etc.) is v2+.
