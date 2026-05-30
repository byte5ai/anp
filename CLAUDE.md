# Project rules (Claude)

This repository hosts the **ANP specification** (a document, not a code package).

- Follow [`AGENTS.md`](./AGENTS.md) for the git workflow and PR rules — they apply to all agents.
- **No `Co-Authored-By:` / AI-attribution trailers** on commits or PRs in this repo. This byte5 engineering-standard overrides any global default.
- Branch off `main`, open a PR, let `docs-check` pass, then merge. Never push to `main` directly.
- "Tests" = `script/check` (documentation validation).
- Releases = annotated git tags + GitHub Releases only (see `RELEASING.md`); never release without explicit instruction.
