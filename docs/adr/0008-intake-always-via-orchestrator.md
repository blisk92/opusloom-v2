# All intake is via the orchestrator

Every ticket enters the system through the orchestrator's intake
conversation. The user describes a feature in natural language;
orchestrator asks clarifying questions as needed and creates the ticket
in `Triage`. A direct UI form is rejected for v1.

**Why.** The story-runner needs structured `given/when/then` stories on
tickets (per R2-Q8). The orchestrator pulls them in conversation
naturally; a form forces the user to know the structure upfront, and most
won't. Continuation of design-partner role throughout the system —
intake is the design front door. Light conversation for simple tickets
(one-line "fix the bug" still creates a one-line ticket).

**Consequences.** Every intake pays one orchestrator turn (a model call).
Long-lived orchestrator with aggressive compaction bounds this cost.
Other intake sources (GitHub issues, scheduled templates) are v2.
