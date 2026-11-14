# ai-delegate-workflow

> 🇪🇬 النسخة العربي: [README.ar.md](README.ar.md)

A project-agnostic AI engineering workflow built on [delegate-skills](https://github.com/amElnagdy/delegate-skills).

**One orchestrator plans, decides, reviews and commits. Cheaper implementer CLIs write the code.**

It was extracted from a real monorepo (an e-commerce SaaS), but nothing here depends on that project. Copy the ideas and templates into any repo.

---

## 1. The problem

When one AI agent does everything in one session:

- an expensive model spends tokens on boilerplate (translations, clones of existing screens, simple tests);
- the same agent that wrote the code also reviews it;
- the context window fills up with file reads and chat history;
- "tests pass" is a claim, not evidence;
- nothing stops the agent from editing files it should never touch (schema, auth, config).

## 2. The idea

Split the roles:

| Role | Who | Does | Never does |
|---|---|---|---|
| **Orchestrator** | Your main agent session (e.g. Claude Code) | Plans, decides, writes briefs, reviews diffs, re-runs checks, updates docs, **commits** | Delegates protected work |
| **Implementer** | A separate coding-agent CLI (Codex, OpenCode, Cursor Agent, Aider, ...) | Edits a real working tree from a brief | Plans, decides, reviews, edits docs, commits |

The implementer's output is a **git diff**. The orchestrator reads it like a pull request.

## 3. Vocabulary

- **delegate-skills** — an open-source package of agent skills (`*-delegate`, `delegate-setup`). Each `*-delegate` skill wraps one implementer CLI behind a small Node script, `relay.mjs`.
- **Relay** — launches the implementer, waits, and writes a structured `result.json` (status, exit code, final report, touched files, session id). It **never commits**.
- **Lane** — a named binding of a kind of work to one implementer plus dials (model, effort). Examples: `feature`, `tests`, `ui`, `i18n`.
- **Fleet** — your set of lanes. Created once with `$delegate-setup`, which discovers installed CLIs, proposes lanes, and writes config only after you approve.
- **Brief** — a self-contained task description. The implementer has no access to the orchestrator's chat.
- **Delta brief** — a short follow-up sent to the same session (`--session <id>`) to fix review findings.
- **BASE** — the commit SHA recorded before dispatch, used to diff the run afterwards.
- **Protected paths** — files that are never delegated (schema/migrations, shared packages, auth, workflow config, docs).

## 4. The loop

```
Plan ─▶ Approve ─▶ Brief ─▶ Dispatch ─▶ result.json ─▶ Review diff ─▶ Verify ─▶ Commit
 main     you       main     relay        relay         main          main       main
```

1. **Plan** — the orchestrator writes a plan with a *delegation table* (subtask → files → lanes). You approve it.
2. **Brief** — one brief per subtask: goal, exact paths, reference sibling, acceptance checks, only the plan excerpt that agent needs.
3. **Dispatch** — clean `git status`, `BASE=$(git rev-parse HEAD)`, run the relay in the background, one run at a time.
4. **Read `result.json`** — status, touched files, final message. Its gate claims are *not* evidence.
5. **Review** — run the diff check against `$BASE`, read the full diff against the brief (scope creep, missing parts). Send fixes as a delta brief.
6. **Verify and land** — the orchestrator re-runs typecheck/lint/tests once on the final diff, syncs docs, and commits.

Details: [docs/02-the-loop.md](docs/02-the-loop.md)

## 5. Routing: who gets which work

| Work | 1st | 2nd | Last resort |
|---|---|---|---|
| i18n keys (wording already decided) | cheap lane | other lane | small Claude agent |
| Clone a named sibling (≤5 files, no new logic) | cheap lane | other lane | small Claude agent |
| Logic, hooks, services, tests, endpoints inside an approved contract | main lane | heavier lane | mid Claude agent |
| Broad read-only search | – | – | read-only explorer agent |
| Architecture, auth, payments, schema, security, final verification | – | – | **orchestrator only** |

Rules of thumb:

- **Rule 0** — if it's cheaper to do than to specify (≤3 files), the orchestrator does it. A cold agent re-reads files and you pay for the brief.
- **Rule 1** — high mistake cost (auth, tenant, payment, migrations, data integrity) → orchestrator.
- **Rule 2** — an open decision is made first by the orchestrator, then delegated.
- **Fallback** — move to the next column when the CLI is missing, quota is out, status ≠ ok, or the post-run check fails after one delta brief. Two failed runs on a subtask → next lane, then the orchestrator.

Details: [docs/03-routing.md](docs/03-routing.md)

## 6. Guardrails

1. **Protected paths** the orchestrator never assigns to a lane.
2. **Brief boundaries** — every brief says: no commit/push/install, no protected paths, run these gates.
3. **Diff check after every run** — a script given `BASE` that flags: HEAD moved (implementer committed), forbidden paths touched, style/strictness violations on added lines.
4. **Different actor reviews** — the author never reviews its own work. Review depth scales with complexity (low: inline; medium: orchestrator reads the full diff; high: separate reviewer agent).
5. **Ledger (advanced, optional)** — the plan's subtask→file→lane map is recorded; a hook blocks the orchestrator from writing lane-owned files itself; a final check compares file hashes to what each lane left.

Honest limits: read-only modes differ per CLI, and `touchedFiles` comes from `git status` (it can't see ignored files or writes outside the repo). The ledger detects drift; it is not security against a process running as the same OS user. **Review the diff, not the report.**

Details: [docs/04-guardrails.md](docs/04-guardrails.md)

## 7. Quick start

```bash
npx skills add amElnagdy/delegate-skills --skill delegate-setup
npx skills add amElnagdy/delegate-skills --skill codex-delegate
```

Requirements: Node 18+, git, each implementer CLI authenticated, an orchestrating agent that can run shell commands.

1. Ask your agent: `Use $delegate-setup to discover my installed implementer CLIs and create a fleet for feature, tests, and UI work.` and approve the proposed lanes.
2. Paste [templates/CLAUDE.md.snippet](templates/CLAUDE.md.snippet) into your repo's `CLAUDE.md` (or `AGENTS.md`).
3. Define your protected paths and a diff-check script.
4. Delegate: `Use $codex-delegate --lane feature to implement <task>, then review and commit it.`

Details: [docs/05-setup.md](docs/05-setup.md)

## 8. What's in this repo

| Path | What it is |
|---|---|
| `docs/01-concepts.md` | Orchestrator, implementer, lane, brief, relay |
| `docs/02-the-loop.md` | The six-step loop in detail |
| `docs/03-routing.md` | Routing table and rules |
| `docs/04-guardrails.md` | Protected paths, diff check, review tiers, ledger |
| `docs/05-setup.md` | Install and wire up |
| `templates/CLAUDE.md.snippet` | Drop-in orchestrator rules |
| `templates/delegation-table.md` | Plan section that assigns lanes |
| `templates/brief.md` | Brief for an implementer |
| `templates/review-checklist.md` | Pre-commit review checklist |

## 9. Credits

The delegation skills, relays and fleet setup are by [amElnagdy/delegate-skills](https://github.com/amElnagdy/delegate-skills). This repo only documents a workflow built around them.

License: MIT
