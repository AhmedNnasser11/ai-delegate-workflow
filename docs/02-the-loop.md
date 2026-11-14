# The delegate loop

1. **Plan.** Orchestrator writes a plan with a delegation table (see templates). You approve it.
2. **Brief.** One brief per subtask: goal, exact paths, acceptance checks, only the plan steps that agent executes. Never paste the whole plan.
3. **Dispatch.** Clean `git status`, record `BASE=$(git rev-parse HEAD)`, run the relay in the background, one run at a time.
4. **Read result.json.** Status, touchedFiles, final message. Treat its gate claims as claims, not evidence.
5. **Review the diff.** Run a diff check against `$BASE` (moved HEAD? forbidden paths? style/strictness breaks?). Read the full diff against the brief for scope creep and missing parts. Send fixes back as a *delta brief* using `--session <id>`.
6. **Verify and land.** Orchestrator re-runs typecheck/lint/tests once on the final diff, syncs docs, and commits.

Failure rule: two failed runs on the same subtask → next lane, then the orchestrator itself.
