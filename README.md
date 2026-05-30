# ANP — Agent Notarization & Negotiation Protocol

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Status: Draft](https://img.shields.io/badge/status-draft%20concept-orange.svg)](#status)
[![Spec: v0.2](https://img.shields.io/badge/spec-v0.2-informational.svg)](./concept-paper.md)

**An open, DLT-neutral trust layer for agent-to-agent agreements, notarization, and dispute resolution.**

AI agents increasingly act on behalf of people and organizations — and, increasingly, on each other. The moment two agents commit to something without a human mediating every step, a gap appears that today's agent protocols don't fill: there is no way to make a commitment **binding, private, verifiable, and enforceable** across organizational and trust boundaries.

ANP fills exactly that gap.

> **📄 The full specification lives in [`concept-paper.md`](./concept-paper.md).** This README is the short version.

---

## The three pillars

ANP is one protocol with three composable pillars on a shared digital-identity foundation:

| Pillar | What it does | Examples |
|---|---|---|
| **① Contract** | Two or more parties form a structured, tamper-evident agreement | A purchase, a price, an order, an API definition, a deadline, an agreed wording |
| **② Notarize** | A credible third party attests to a verifiable fact | An audit, an event, a measurement, a state, an explicitly requested confirmation |
| **③ Resolve** | Parties agree on a neutral arbiter, submit evidence, and have a binding ruling enforced | Quality disputes, non-delivery, partial performance |

The need appears wherever an **unambiguous determination is in the interest of at least two parties** — and because the per-determination cost is meant to be negligible, that spans everything from a factory purchase down to "we agree this is the API contract."

## Why this is hard (and why DLT)

Agents can't trust each other by default:

- **Non-determinism** — the same prompt can yield different outputs.
- **Context volatility** — after a restart or memory flush, an agent may have *no recollection* of what it agreed to.
- **No persistent identity, no enforcement** — an agent is a process, not a person.

The failure modes ANP defends against are concrete: **hallucinated terms, lost memories, and manipulated databases.** A distributed ledger provides an external, tamper-evident anchor that no context-window reset or compromised database can alter.

**The thesis: AI makes DLT usable; DLT makes agents trustworthy.** Agents absorb the wallet/key/fee friction that blocks human DLT adoption; the ledger gives agents the external source of truth and programmable enforcement they lack.

## How it works — one artifact, four guarantees

Every binding action produces an **ANP Object** (a signed, schema-valid, VC-shaped artifact) that is:

1. **Schema-valid** → no ambiguous obligations (defends against hallucinated/loose terms).
2. **Signed** with an agile, post-quantum-capable suite → authenticity & non-repudiation.
3. **Hash-linked** to its predecessor → a tamper-evident history.
4. **Anchored** on-chain by its **hash + status only** → an external source of truth, while the payload stays **off-chain** (privacy / GDPR-compatible).

The three pillars are three *shapes* of this same spine plus a state machine and a settlement hook.

## Design principles

- **Machine-first** — structured data, not natural language. Ambiguity is a bug.
- **Anchor, don't publish** — the ledger holds commitments, not content.
- **DLT-neutral** — built only on cross-platform standards (**W3C DID + VC 2.0**) and abstract ledger capabilities (anchoring, settlement). Chain-specific standards are optional interop profiles.
- **Authority by mandate, not mandatory human approval** — human-in-the-loop is a *configurable threshold*, not a fixed step (Human → Agent → Agent → Human).
- **Post-quantum-ready at the binding layer** — signature suites are agile and may be post-quantum today.
- **Not a token, not a marketplace, not a product** — open infrastructure.

## Where ANP sits in the stack

```
Application      Agent platforms, marketplaces, vertical apps
─────────────────────────────────────────────────────────────
ANP (Trust)      Identity + Contract · Notarize · Resolve   ← this protocol
─────────────────────────────────────────────────────────────
Primitives       W3C DID · W3C VC 2.0 · Anchor(hash) · Settlement/Escrow
─────────────────────────────────────────────────────────────
DLT              IOTA Rebased (reference) | EVM/Base | Hedera | …
```

ANP **reuses** identity (DID/VC), settlement (native chain), and attestation patterns; **aligns with — but does not depend on** — A2A (communication), AP2 (payment mandates), x402 (payment rail), ERC-8004 (identity/reputation), and EAS (attestation); and **contributes** the chain-neutral, multi-party, general-purpose layer for binding agreements + notarization + dispute resolution that the rest of the stack leaves empty.

## Reference DLT

The spec is chain-agnostic. The **reference profile** targets **IOTA Rebased** (Move VM, sub-second finality, near-zero / sponsored fees). Hard selection criteria for any chain: sub-second finality, a credible post-quantum path at the protocol layer, ultra-low fees, programmable escrow, programmatic wallets. Native EUR/USD stablecoins are a valued nice-to-have. An EVM interoperability profile (ERC-8004 / EAS / x402) is available but optional.

## Status

**v0.2 — Draft / Concept.** This is a design specification, stable enough to implement against, with unresolved questions explicitly marked ([§16 Open Questions](./concept-paper.md#16-open-questions)). It is **not** a final standard. Trust posture is stated plainly: ANP *reduces and makes explicit* trust; it does not claim to eliminate it.

Roadmap: concept (now) → proof-of-concept per pillar → spec v1.0 → community & ecosystem. See [§17](./concept-paper.md#17-roadmap).

## Contributing

ANP is open infrastructure and feedback is welcome — see [`CONTRIBUTING.md`](./CONTRIBUTING.md). Open an issue to discuss a design question, or a pull request to propose a change to the spec. By participating you agree to the [Code of Conduct](./CODE_OF_CONDUCT.md).

## License

[MIT](./LICENSE) © 2026 byte5.ai

---

*Authored by byte5 ([byte5.ai](https://byte5.ai)). Contact: `cw@byte5.de`.*
