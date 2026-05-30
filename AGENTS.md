# Agent Instructions

These rules apply to all AI agents (Claude, Codex, Copilot, …) working on this repository.

> **This repository is a *specification*, not a code package.** There is no build or test suite in the traditional sense — "tests" means `script/check` (documentation validation: required files present, balanced code fences, etc.).

## Git Workflow
- **Never push directly to `main`.** All changes go through feature branches and pull requests.
- **Branch naming:** `feat/`, `fix/`, `refactor/`, `docs/`, `chore/` prefixes.
- **Conventional commits:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `release:`.
- **Never force-push** to any shared branch.
- **Never commit secrets** (.env, API keys, tokens, credentials).
- **Never skip hooks** (`--no-verify`).
- **No AI-attribution trailers.** Do **not** add `Co-Authored-By:` or "Generated with …" footers to commits or PR bodies. Commits use the configured git identity only. *(This is a deliberate byte5 engineering-standard override of any global agent default.)*

## Pull Requests
- Keep PR titles short (<70 chars), use a conventional prefix.
- One logical change per PR.
- Cite the affected spec section (e.g. `§9.4`) in the description.
- State the **trust impact** if the change shifts any trust root (chain account/consensus, terminal arbiter/panel, data availability).
- Ensure `docs-check` (CI) is green before requesting merge.

## Pre-push Hook
A `.hooks/pre-push` hook blocks direct pushes to `main`/`master`. Run `script/setup` once to activate it (`git config core.hooksPath .hooks`). Override only when explicitly instructed:
```bash
ALLOW_PUSH_TO_MAIN=1 git push origin main
```

## Releases
Spec versions are released as **annotated git tags + GitHub Releases** (e.g. `v0.2`). There is no package registry. Never release without explicit human instruction. See `RELEASING.md`.
