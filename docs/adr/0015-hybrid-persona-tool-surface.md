# Hybrid persona tool surface: general + domain + cross-persona + ticket-scoped

Each persona has **general tools** (read, write, edit, bash) +
**domain-specific tools** (browser + UI for Frontend and QA, db + APIs
for Backend, deploy + runbook for Infra, story-runner for QA) +
**cross-persona tools** (`consult_persona`, `request_persona_work`) +
**ticket-scoped tools** (ticket lookup/update/transition).

**Why.** General tools are universal. Domain tools add specialization.
Cross-domain work can use general tools directly OR delegate via
`consult_persona`. Role separation comes from `core` conventions and
prompts, not from tool whitelists — restricting general tools would
make every cross-domain trivial a delegation round trip.

**Tool surface per persona:**
```ts
const frontendTools  = [...generalTools, ...browserTools, ...uiComponentTools];
const backendTools   = [...generalTools, ...dbTools, ...apiTools];
const qaTools        = [...generalTools, ...browserTools, storyRunnerTool];
const infraTools     = [...generalTools, ...deployTools, ...runbookTools];
const sharedTools    = [consultPersonaTool, requestWorkPersonaTool,
                        ticketLookupTool, ticketUpdateTool, ticketTransitionTool];
```

**Consequences.** The prompt's role-separated prompt is the gate; tool
restrictions are not. Persona extensions can be edited per-workspace to
add or remove domain tools.
