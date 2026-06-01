# Releasing

ANP is a **specification**, not a software package — so "releasing" means publishing a versioned snapshot of the spec, not shipping to a package registry (no PyPI/npm).

## Versioning

- The spec version lives in two places that **must agree**:
  - `SPEC.md` front matter (`v0.X`)
  - `README.md` status badge (`spec-v0.X`)
- Use `vMAJOR.MINOR` (e.g. `v0.2`). Pre-release drafts may use `v0.3-draft`.

## Process (only on explicit human instruction)

1. Open a `release/vX.Y` branch.
2. Bump the version in `SPEC.md` and `README.md`; update the changelog section / `Appendix C` (changes).
3. Ensure `docs-check` passes.
4. Open a PR, get it reviewed, merge to `main`.
5. Tag the merge commit:
   ```bash
   git tag -a vX.Y -m "ANP spec vX.Y"
   ALLOW_PUSH_TO_MAIN=1 git push origin vX.Y   # tags only; never force-push main
   ```
6. Create the GitHub Release from the tag with notes summarizing the changes.

## Rules

- **Never release without explicit instruction.** A GitHub Release is a release.
- Collect changes, review thoroughly, release **once** — no rapid-fire releases.
- Never force-push `main`. Tags are pushed explicitly and are immutable once published.
