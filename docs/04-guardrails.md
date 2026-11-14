# Guardrails

1. **Protected paths** — a list the orchestrator never gives to a lane (schema/migrations, shared packages, auth, workflow config, docs/status).
2. **Brief boundaries** — every brief says: don't commit/push/install, don't touch protected paths, run these gates.
3. **Diff check after every run** — a script that, given `BASE`, flags: HEAD moved (implementer committed), forbidden paths touched, style/strictness violations on added lines.
4. **Different actor reviews** — author never reviews own work. Review depth scales with complexity tier (low: inline, medium: orchestrator reads full diff, high: separate reviewer agent).
5. **Ledger (optional, advanced)** — plan lists subtasks → files → lanes. A hook blocks the orchestrator from writing lane-owned files itself, and a final check compares file hashes to what each lane left. Note: this is a guard against drift, not security against a process running as the same OS user.

Honest limits: relays' read-only modes vary per CLI; `touchedFiles` is `git status`-based and can't see ignored files or writes outside the repo. Review the diff, not the report.
