# Contributing to ANP

ANP (Agent Notarization & Negotiation Protocol) is an open specification. It is currently a **draft concept** (v0.2), which means the most valuable contributions right now are *design feedback* — challenging assumptions, finding gaps, and pressure-testing the protocol against real use cases.

## Setup

This is a documentation repository — no dependencies to install. Run the setup script once to activate the git hooks (which block accidental direct pushes to `main`):

```bash
script/setup
```

## How to contribute

- **Discuss a design question** → open a [GitHub Issue](../../issues). Describe the scenario, the part of the spec it touches (cite the section, e.g. `§9.4`), and what you think is wrong, missing, or ambiguous.
- **Propose a change to the spec** → open a Pull Request against [`concept-paper.md`](./concept-paper.md). Keep each PR focused on one coherent change and explain the *why* in the description.
- **Report a real-world use case** → open an Issue labelled `use-case`. Concrete scenarios sharpen the spec more than abstract debate.

## What makes a good spec contribution

- **Be concrete.** Cite the section. Prefer "in §7.4 the N-party fork case is undefined because …" over "the contracting flow is unclear."
- **Respect the design goals** ([§3](./concept-paper.md#3-design-goals--non-goals)): machine-first, anchor-don't-publish, DLT-neutral, identity-rooted, post-quantum-ready at the binding layer, privacy-preserving. A change that breaks one of these needs to justify the trade-off.
- **State the trust impact.** ANP is honest about residual trust roots (chain account/consensus, terminal arbiter/panel, data availability). If a change shifts trust, say so.
- **Keep normative language precise.** Use RFC 2119 keywords (MUST / SHOULD / MAY) the way the spec does.

## Scope

This repository is the **protocol specification**. Reference implementations, SDKs, and tooling will live in separate repositories and are coordinated via Issues here.

## Decision process

While the spec is a draft, changes are merged by the maintainers (byte5) after discussion. As the project matures we intend to move to a more formal, RFC-style process with versioned releases and a public changelog. Open an Issue if you'd like to help shape that process.

## Code of Conduct

Participation is governed by the [Code of Conduct](./CODE_OF_CONDUCT.md). Be constructive and assume good faith.

## License

By contributing, you agree that your contributions are licensed under the [MIT License](./LICENSE).
