# Tickets carry freeform description plus structured user stories; empty stories skip QA

Each ticket has both a freeform `description: string` and a structured
`userStories: {given, when, then}[]`. Tickets with `userStories.length === 0`
skip the QA-with-story-runner phase entirely (`InProgress → Review`).
Tickets with stories go through full QA-with-story-runner.

**Why.** The orchestrator naturally extracts both during intake ("what
should happen when the user..." pulls a story; "what's the broader
context?" pulls the description). Two questions; two fields. Freeform
context informs story-runner interpretation (a vague `given: "logged-in
user"` is ambiguous without description context). Backend tickets don't
have UI-level stories — for those, skipping QA is honest.

**Consequences.** Story-runner reads both fields. A backend ticket goes
straight from build done to PR review, no Playwright round trip. The
`(iii) synthesized-from-diff` option is rejected as magical and brittle
for v1.
