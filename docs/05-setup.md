# Setup

```bash
npx skills add amElnagdy/delegate-skills --skill delegate-setup
npx skills add amElnagdy/delegate-skills --skill codex-delegate
npx skills add amElnagdy/delegate-skills --skill opencode-delegate
```
Requirements: Node 18+, git, each implementer CLI authenticated, an orchestrating agent that can run shell commands.

1. Run `$delegate-setup` — approve the proposed lanes (global or per-repo).
2. Add `templates/CLAUDE.md.snippet` to your repo's `CLAUDE.md`.
3. Add your protected-path list and diff-check script.
4. Dispatch: `Use $codex-delegate --lane feature to implement ...`, then review and commit.
