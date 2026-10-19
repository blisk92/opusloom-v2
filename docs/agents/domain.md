# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- **`GLOSSARY.md`** at the repo root. OpusLoomV2 is a single-context repo (no `GLOSSARY-MAP.md`), so this is the one canonical vocabulary file. The terms there (Ticket, Persona, Story-Runner, ConventionDoc, FailureClassDoc, etc.) are the project's own; respect them.
- **`docs/adr/`**: read ADRs that touch the area you're about to work in. OpusLoomV2's ADRs are sequenced `0001-…` to `0023-…` covering the design tree, deployment, stack, and graph UI. Each ADR cross-references the next; reading in order gives the full design rationale.
- **`docs/specs/`**: project-level specs. Currently `v1-opusloomv2.md` defines the v1 surface; per-area specs will land here as work begins.
- **`docs/agents/`**: per-agent conventions (this file and its siblings).

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/domain-modeling` skill (reached via `/grill-with-docs` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.

## File structure

Single-context repo:

```
/
├── GLOSSARY.md
├── docs/
│   ├── adr/
│   │   ├── 0001-base-on-pi-durable.md
│   │   └── 0023-graph-ui-system-and-per-ticket.md
│   ├── agents/
│   │   ├── issue-tracker.md
│   │   ├── triage-labels.md
│   │   └── domain.md
│   └── specs/
│       └── v1-opusloomv2.md
└── src/
```

Multi-context repo (presence of `GLOSSARY-MAP.md` at the root):

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `GLOSSARY.md`. Don't drift to synonyms the glossary explicitly avoids. The glossary's `_Avoid_` lists are the canonical "do not use these" entries.

If the concept you need isn't in the glossary yet, that's a signal: either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders), but worth reopening because…_

The ADRs in `docs/adr/` are the project's *decision log*; superseding one is itself an ADR-worthy action.
