# Concepts

- **Orchestrator** — the main agent session (e.g. Claude Code). Plans, decides, reviews, verifies, commits.
- **Implementer** — a separate coding-agent CLI (Codex, OpenCode, Cursor Agent, Aider, ...) that edits a real working tree. The diff is the deliverable.
- **Lane** — a named binding of a *kind of work* to one implementer plus dials (model, effort). Examples: `feature`, `tests`, `ui`, `i18n`.
- **Fleet** — your set of lanes. Created once with `$delegate-setup`, which discovers installed CLIs and writes config only after you approve.
- **Brief** — self-contained task text. The implementer has no orchestrator chat history.
- **Relay** — the small Node script (`relay.mjs`) that launches the implementer and writes a structured `result.json` (status, exitCode, final report, touchedFiles, session id). It never commits.
