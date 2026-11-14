# Routing: who gets which work

Walk the row left to right. Move to the next column when the CLI is missing, quota is out, `result.json` status is not ok, or the post-run check fails after one delta brief.

| Work | 1st | 2nd | Last |
|---|---|---|---|
| Translations / i18n keys (wording decided) | cheap lane | other lane | small Claude agent |
| Copy a named sibling (≤5 files, no new logic) | cheap lane | other lane | small Claude agent |
| Logic, hooks, services, tests, endpoints inside an approved contract | main lane | heavier lane | mid Claude agent |
| Broad read-only search | — | — | read-only explorer agent |
| Architecture, auth, payments, schema, security, final verification | — | — | **orchestrator only** |

Rule 0: if it's cheaper to do than to specify (≤3 files, shorter to write than to brief), the orchestrator does it. A cold subagent re-reads files and you pay for the brief.
Rule 1: high mistake cost (auth, tenant, payment, migrations, data integrity) → orchestrator.
Rule 2: an open decision is made by the orchestrator first, then delegated.
