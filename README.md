# OpusLoomV2

A fresh Pi Durable-based agent harness.

## Status

Early. No code yet — design first. See [GLOSSARY.md](./GLOSSARY.md) for canonical terms and [docs/adr/](./docs/adr/) for architectural decision records.

## Design settled so far

See `docs/adr/` for the full set. Top-level shape:

- **Base:** fresh on `@earendil-works/pi-durable` (no fork)
- **Tickets:** each ticket is a `defineTask`; SDLC states are its phases
- **Authority:** mixed (agent + humans + harness events)
- **Autonomy:** hybrid scheduler, scheduler-default
- **Roles:** orchestrator (always-alive) + 4 personas (Frontend, Backend, QA, Infra)
- **Routing:** auto-route with user override
- **Failure handling:** inline retry with `attempts` cap (default 5), auto-recycle on Failed
- **Story-runner:** Playwright walks user stories as a sub-conversation owned by the ticket task
- **Intake:** always via orchestrator
- **Cross-persona tickets:** don't decompose; assigned persona orchestrates via `consult_persoana`

## Working agreements

- One question per turn in design discussions.
- All Round-1 / Round-2 decisions live in `docs/adr/0001-…` etc.
- Round-3 questions still open: persona conventions doc, `consult_persoana` semantics, scheduler internals, onboarding, cost/budget per ticket.
