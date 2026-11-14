# ai-delegate-workflow

A project-agnostic AI engineering workflow built on [delegate-skills](https://github.com/amElnagdy/delegate-skills):
**one orchestrator plans, decides, reviews and commits. Cheaper implementer CLIs write the code.**

> Extracted from a real monorepo (an e-commerce SaaS). Nothing here depends on that project.

## The idea in one line

Plan → approve → **delegate** (by lane) → review the diff yourself → verify → commit.

The implementer never commits. The orchestrator never trusts the implementer's own "tests pass" claim.

## Why delegate?

| Problem with one agent doing everything | What delegation changes |
|---|---|
| Expensive model writes boilerplate | Mechanical work goes to cheap/free CLIs (Codex, OpenCode, ...) |
| Author reviews own code | Review is always a different actor than the author |
| Context bloat | Implementer gets a short, self-contained **brief**, not the chat |
| "Trust me, it works" | You re-run the gates on the final diff |
| Agent touches things it shouldn't | Protected paths + a forbidden-path diff check |

## Contents

- [docs/01-concepts.md](docs/01-concepts.md) — orchestrator, implementer, lane, brief, relay
- [docs/02-the-loop.md](docs/02-the-loop.md) — the 6-step delegate loop
- [docs/03-routing.md](docs/03-routing.md) — who gets which kind of work
- [docs/04-guardrails.md](docs/04-guardrails.md) — protected paths, diff check, ledger idea
- [docs/05-setup.md](docs/05-setup.md) — install delegate-skills and wire it into a repo
- [templates/CLAUDE.md.snippet](templates/CLAUDE.md.snippet) — drop-in orchestrator rules
- [templates/delegation-table.md](templates/delegation-table.md) — plan section that assigns lanes
- [templates/brief.md](templates/brief.md) — brief for an implementer
- [templates/review-checklist.md](templates/review-checklist.md) — what to check before landing

## Quick start

```bash
npx skills add amElnagdy/delegate-skills
```
Then ask your orchestrating agent:
```
Use $delegate-setup to discover my installed implementer CLIs and create a fleet for feature, tests, and UI work.
```
Copy `templates/CLAUDE.md.snippet` into your `CLAUDE.md` and start planning.

License: MIT
