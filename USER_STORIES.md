# ANP User Stories

*Informative companion to [`SPEC.md`](./SPEC.md) v0.2. This document is illustrative, not normative.*

Each story below was selected because the **combination** of ANP properties it exercises — cross-organizational bindingness without a platform intermediary, counterparty-verifiable authority, independent timestamping, and composable dispute resolution — is a genuine USP rather than something an existing tool already does well. To keep the comparison honest, every story names the **next-best solution available today** and states where ANP wins *and* where it loses.

A summary table is at the [end](#summary).

---

## US-1 — Autonomous cross-organizational service procurement

**Story.** A logistics company's procurement agent needs a 5,000-word technical document translated within the hour. A language-service provider's agent offers it for €0.50. It is 03:00; no human on either side is awake, and the two companies have no prior relationship and no shared platform.

**How ANP serves it.** Pillar I `offer` → `accept` with schema-validated terms (input/output spec, price, deadline — SPEC §7.3); both agents act inside Mandates (§5.3), so the deal is binding without human sign-off. `execute` opens a stablecoin escrow; on delivery the provider anchors a completion `assert`, optionally backed by a quality attestation (Pillar II) named in `acceptance_criteria`; settlement follows the optimistic window (§9.4).

**Next-best today.**
- **x402 (HTTP micropayments):** handles the *payment* elegantly, but carries no negotiated terms, no escrow conditioned on outcome, no dispute path, and no durable evidence of what was promised.
- **A centralized agent marketplace** (Virtuals ACP-style on Base, or a Web2 API marketplace with house terms): full service, but both parties must be members, the platform's jurisdiction and fee schedule apply, the platform can change the rules, and the evidence lives in the platform's database.

**ANP advantages.** Works between strangers across org boundaries with no joint platform membership; terms are machine-validated, so a hallucinated "reasonable timeframe" cannot bind anyone; the evidence trail survives both parties' systems (and both agents' context windows); marginal cost per agreement is a few anchors.

**ANP disadvantages.** Both sides need the ANP stack — DIDs, funded or sponsored wallets, Object storage. The happy path waits out the challenge window unless a mutual-settlement fast path is used. At €0.50, monetary dispute resolution is uneconomical; practical recourse at this value is reputational, not judicial.

---

## US-2 — The API contract two companies can both prove

**Story.** Two engineering organizations integrate their systems. Their agents converge on an API definition — endpoints, types, versioning policy, deprecation date. Months later, one side's agent regenerates client code from a "remembered" schema and breaks the integration. Who agreed to what, and when?

**How ANP serves it.** A single co-signed `memorandum` Object (§7.2), anchored once: cost ≈ one anchor, terminal at `ACCEPTED`, no escrow. Before regenerating anything, either agent re-fetches the Object, checks its hash against the anchor, and reconstructs the agreed schema from ground truth instead of memory (§7.5).

**Next-best today.**
- **OpenAPI spec in a shared Git repository, signed commits:** free and familiar — but the repository is one party's (or GitHub's) infrastructure, admins can rewrite history, a commit signature proves *authorship*, not *mutual assent*, and there is no neutral timestamp.
- **A DocuSign'd PDF annex:** legally stronger than ANP, but a human workflow measured in hours, not machine-readable, and per-envelope pricing.

**ANP advantages.** Mutual assent is cryptographically explicit (every party's signature on one canonical artifact); ordering and existence are attested by a neutral ledger neither party operates; the artifact is machine-verifiable at runtime, which is the actual failure point for agents.

**ANP disadvantages.** No established legal weight today (§16.5) — where a binding SLA matters legally, the DocuSign'd contract is still needed alongside; both orgs must operate DID infrastructure for what Git does with an SSH key.

---

## US-3 — A cold-chain measurement an insurer will accept

**Story.** A temperature-sensitive pharmaceutical shipment travels three days. An accredited IoT oracle samples the container temperature, captures C2PA-signed sensor readings, and attests the cold chain held. Months later the consignee's insurer — who trusts neither shipper nor carrier — verifies the record.

**How ANP serves it.** Pillar II: the oracle issues an `attest` Object (`witnessing: "observed"`, UCUM-coded measurement, C2PA evidence — §8.2), optionally hardened by a 2-of-3 witness quorum (§8.3), anchored on-chain. Its accreditation is a Role VC on a trust list the insurer recognizes; its bond makes a false report economically punishable (§8.4). The shipping contract can name exactly this attestation as an `acceptance_criteria`.

**Next-best today.**
- **EAS attestation:** the closest prior art — but EVM-specific, with no witness-quorum semantics, no bonded-oracle/slashing frame, and attester identity as a bare address rather than an accredited DID.
- **Proprietary IoT/visibility platforms:** operationally mature, but the evidence is a centralized database entry owned by one commercial party.
- **RFC 3161 / eIDAS qualified timestamp on the sensor log:** proves integrity and time, but says nothing about *who* attested, their accreditation, or revocation.

**ANP advantages.** Chain-neutral; accreditation, quorum (including recorded dissents), bonding, and revocation are first-class; the attestation composes directly into contracts and disputes instead of being a standalone PDF.

**ANP disadvantages.** Garbage-in remains: a spoofed capture device produces a forged manifest that ANP faithfully immortalizes (§8.5) — capture authenticity stays an accreditation problem. And the accredited-oracle ecosystem ANP presumes does not exist yet; the protocol only defines the rails.

---

## US-4 — Delegated purchasing inside a provable budget envelope

**Story.** A principal gives her executive-assistant agent a monthly travel budget. The agent's sub-agent negotiates a booking with an airline's sales agent. The airline wants to know — *before* committing inventory — that this unknown agent's commitment will actually bind the principal, without phoning anyone.

**How ANP serves it.** The Mandate VC (§5.3): per-action cap, allowed counterparties, expiry, revocation, and an `escalation_threshold` above which the principal must co-sign (`approve`). The airline's verifier checks the mandate chain back to the principal as part of validating the `accept` — the Human → Agent → Agent → Human topology the spec is built around.

**Next-best today.**
- **AP2 mandates:** the same VC pattern — but scoped to payments and its surrounding ecosystem, not to general contractual commitments.
- **Virtual cards with spend limits:** enforce a cap, but at the *issuer* — the merchant cannot see or verify the limit, and a card says nothing about *what* may be bought or whether the buyer's commitment binds anyone.
- **OAuth scopes / API keys:** service-specific permissions with no value semantics and no meaning to a third-party counterparty.

**ANP advantages.** Counterparty-verifiable authority is the differentiator: the merchant can *prove to itself* the deal binds the principal. Escalation keeps humans in exactly the loops they chose. The same envelope governs non-payment commitments (attestations, dispute actions) that card rails cannot express.

**ANP disadvantages.** A plaintext mandate leaks negotiation-sensitive caps to the counterparty (selective disclosure is needed in practice); revocation timeliness depends on status-list freshness; and aggregate (multi-deal) caps are not counterparty-enforceable — only per-action caps are (§5.3, M4).

---

## US-5 — A dispute neither party's platform owns

**Story.** A data-preparation agent delivers 80% of a dataset, late. The client agent refuses payment; the provider asserts completion. Both companies sit in different jurisdictions, and neither accepts the other's "terms of service" as the forum.

**How ANP serves it.** The arbitration clause was fixed in `terms.dispute` at contract time (§9.2). The provider's `assert` is challenged within the window; both sides post bonds and submit anchored attestations as `evidence`; a bonded arbiter (or VRF-drawn panel) issues a proportional ruling — say 70/30 — and `enforce` settles the escrow per the numeric directive (§6.2.1) without the loser's cooperation.

**Next-best today.**
- **Platform arbitration (Upwork/Fiverr model):** works only inside the platform, with the platform as judge in its own ecosystem and the evidence in its database.
- **Kleros:** genuinely neutral crowd jurors — but EVM-bound, evidence is public (a privacy non-starter for B2B), and token-curated juries are hard to predict for commercial disputes.
- **UMA optimistic oracle:** excellent for binary fact assertions; not built for multi-factor commercial disputes with proportional outcomes.

**ANP advantages.** Forum fixed *before* the conflict; evidence is the same notarization machinery, already anchored; outcomes are proportional (basis points, penalties), not binary; payloads stay private; enforcement is automatic given the ruling.

**ANP disadvantages.** Every Thread rests on terminal trust in the named arbiter/panel (§5.4, M7) — explicit and bonded, but real. The neutral-arbiter pool is a bootstrap problem the protocol cannot solve alone. Below a value floor, bonded arbitration is uneconomical.

---

## US-6 — An audit attestation that outlives the auditor's portal

**Story.** An accredited AI-safety auditor evaluates a model for regulatory conformity and attests the result. Three years later a regulator and an enterprise customer independently verify the attestation — including whether it was revoked in the meantime. The auditor has since been acquired; its customer portal is gone.

**How ANP serves it.** Pillar II `attest` (subject_kind `audit`) anchored on-chain, with the auditor's accreditation VC chained to a recognized trust list; revocation via Bitstring Status List, plus an on-chain `revoke` for the contract-checkable case (§8.6). Verification needs only the anchor, the Object (from the DA layer per §6.4), and the trust list — not the auditor's continued existence.

**Next-best today.**
- **eIDAS QES + qualified timestamp on a signed report:** stronger legal standing than ANP (statutory presumption of authenticity) — but verification of *status* (revoked? superseded?) depends on the trust-service provider's infrastructure and is not machine-composable.
- **The auditor's "verify this certificate" web portal:** dies with the vendor.

**ANP advantages.** Verifiability is independent of the issuer's operational survival; revocation is machine-checkable; and the attestation is composable — a procurement contract can require "a valid audit attestation from trust list X" as an acceptance criterion, closing the loop between Pillars I and II.

**ANP disadvantages.** Legal weight is unresolved (§16.5): today ANP *complements* eIDAS rather than replacing it. Long-horizon verification depends on DA-layer retention actually being honored (§16.4), and trust-list governance — who accredits the accreditors — is an open question (§16.2).

---

## Summary

| # | Story | Pillars | Next-best today | ANP edge in one line |
|---|---|---|---|---|
| US-1 | Autonomous cross-org procurement | I (+II, III) | x402 / platform marketplace | Binding terms + escrow + evidence between strangers, no platform membership |
| US-2 | Provable API contract | I (memorandum) | Git + signed commits / DocuSign | Mutual assent with a neutral timestamp, machine-verifiable at agent runtime |
| US-3 | Cold-chain measurement | II | EAS / IoT platforms / RFC 3161 | Accredited, bonded, quorum-backed attestation that composes into contracts |
| US-4 | Provable budget envelope | I (mandates) | AP2 / virtual-card limits | Authority the *counterparty* can verify, beyond payments |
| US-5 | Cross-jurisdiction dispute | III (+I, II) | Platform arbitration / Kleros / UMA | Pre-agreed neutral forum, private evidence, proportional automatic enforcement |
| US-6 | Long-lived audit attestation | II | eIDAS QES + portal | Verification that survives the issuer, machine-checkable revocation |

**Shared adoption costs (all stories).** Counterparty adoption is the network-effect hurdle: every story needs both sides on ANP. Operating DIDs, wallets and Object storage is real overhead versus incumbent tools, and parts of the surrounding ecosystem (DID method maturity, accredited notary/arbiter pools, trust-list governance) are younger than the protocol design. Where a story touches a known v0.2 gap, the gap is tracked in the repository's `spec-review` issues rather than glossed over here.
