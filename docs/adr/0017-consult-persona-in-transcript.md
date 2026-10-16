# `consult_persona` runs in the asker's transcript

When Dana (Frontend) needs a knowledge question from Sam (Backend), she
calls `consult_persona({ persona, question })`. The runtime injects the
consultant's `core` conventions + relevant `lessons` into a single model
turn under the consultant's LLM, runs it, and logs the consultant's
reasoning inline in Dana's transcript as a `consult:<persona>` entry.
Returns a structured report. Cost ≈ one model turn per consult.

**Why.** Knowledge queries are the common case (convention, error
meaning, design preference). In-transcript visibility gives the asker
auditability — future Dana knows *why* she chose what she did.
Sub-conversations (the alternative) hide reasoning behind a summary;
that's reserved for actual work delegation via `request_persona_work`.

**Consequences.** Latency ~2-5s per consult. If a persona needs to
delegate substantial work (not just a question), it uses
`request_persona_work` — separate tool, separate semantics, opens a
sub-task.
