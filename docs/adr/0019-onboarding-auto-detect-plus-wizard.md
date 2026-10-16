# Onboarding: auto-detect + wizard, single workspace per process

**Onboarding is a wizard**, not zero-config: OpusLoomV2 opens by scanning
the workspace folder (git root, primary language(s), existing
agent-context files like `CLAUDE.md`/`AGENTS.md`/`README.md`), then
opens a wizard: project name → confirm personas → model per persona →
credentials → confirm. The wizard writes `workspace.toml`, persona
`agents/*.md` files, and `.opusloom/` config.

**Why.** Zero-config (a) is forgiving for power users but risky for
new users who don't know what's being configured. A wizard surfaces the
decisions auto-detect can't make (which personas, which model,
credentials) without forcing a TUI. Web UI for both onboarding and
runtime keeps one ergonomic model.

**Single workspace per OpusLoomV2 process.** Multi-workspace in v1 is
running multiple OpusLoomV2 instances against different folders (already
isolated at the OS level). A unified workspace switcher is a v2 add —
substantial engineering (registry, switcher UI, cross-workspace
search) and we don't yet know what users want from it.

**Consequences.** Onboarding is per-workspace; the workspace pick is
the first action when opening OpusLoomV2. The `workspace.toml`
remembers the last workspace for next launch. The four default personas
ship as templates and can be edited per-workspace.
