# Orchestrator has system tools plus read tools; no write or bash

The orchestrator is the always-alive project conductor. Its toolset is
**system** (ticket CRUD, kanban view, ADR read/write, escalation
trigger) **plus read** (`read`, `grep`, `search_history`).

**Why.** The orchestrator must look at build-agent transcripts and
story-runner reports to make routing / escalation decisions — `(a)
system only` would fly blind. But the orchestrator *cannot edit code*;
running `bash`/`write`/`edit` would blur the role boundary (defeats
separation-of-concerns). Read tools are cheap and replay-safe.

**Tiered mode (d) — per-turn role flip — is v2** if we ever want an
"investigate mode" that briefly unlocks write tools.

**Consequences.** The orchestrator's role boundaries are structurally
enforced (no `bash`/`write`/`edit` in the tool whitelist), not just
prompt-enforced. Storage size stays small (read-heavy, not write-heavy).
The orchestrator's compaction can be aggressive (Q-R3-3) without losing
role separation; its `core` says "you don't fix code".
