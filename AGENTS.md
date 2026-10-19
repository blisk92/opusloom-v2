# Agent instructions for OpusLoomV2

This is a Pi Durable-based agent harness with a kanban-driven SDLC, named personas, and a Playwright-based story-runner for validating user stories against a live UI. Solo dev project; "build quick" cadence.

## Reading order for newcomers

1. `README.md` — project shape, stack, deployment, status.
2. `GLOSSARY.md` — canonical vocabulary (Ticket, Persona, Story-Runner, etc.).
3. `docs/specs/v1-opusloomv2.md` — the v1 user surface, contracts, testing seam, out-of-scope.
4. `docs/adr/0001-…0023-…md` — the architectural decision log, sequenced.
5. `docs/agents/` — agent conventions (this file plus its siblings).

## Agent skills

### Issue tracker

GitHub Issues via the `gh` CLI. The repo isn't pushed yet; the convention is recorded for when it is. See `docs/agents/issue-tracker.md`.

### Triage labels

Default `mattpocock/skills` vocabulary — `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context. `GLOSSARY.md` at the root, `docs/adr/` for decisions, `docs/specs/` for v1+ specs. See `docs/agents/domain.md`.

## Working agreements for OpusLoomV2

- Use the GLOSSARY vocabulary. "Ticket" not "issue", "Persona" not "agent role", "Story-Runner" not "QA bot". The `_Avoid_` lists in `GLOSSARY.md` are binding.
- When proposing code, check `docs/adr/` first. If a new architectural choice is needed, write the ADR before the code.
- The design tree is settled; v1 is a build sprint. "Build quick" — feature requests that aren't user stories in `docs/specs/v1-opusloomv2.md` go to v2.
- A single test seam: `test/serve.ts`-style end-to-end. New test code lives under `test/`.
- TypeScript erasable syntax only (no enums, no parameter properties, no namespaces). Imports use `.ts` extensions.
- Code style: 4 spaces, 100 columns (matches `pi-pocket/.prettierrc.json`); braces on every `if`/loop; blank line after declarations and before `return`/`throw`.
