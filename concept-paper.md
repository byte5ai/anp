# ANP: Agent Notarization & Negotiation Protocol

## An Open, DLT-Neutral Trust Layer for Agent-to-Agent Agreements, Notarization, and Dispute Resolution

*Concept Paper / Draft Specification — v0.2 (May 2026)*

**Status:** Draft (Concept). This document is a design specification intended to be stable enough to implement against, while explicitly marking unresolved questions. It is not a final standard.

**Editorial note (v0.1 → v0.2):** v0.1 (March 2026) described a single-purpose *negotiation* protocol. v0.2 broadens the scope to a three-pillar **trust layer** — **Contracting, Notarization, Dispute Resolution** — on a shared identity foundation, corrects several technical and attribution errors found during review (see [Appendix C](#appendix-c-changes-from-v01)), and reorients the design around DLT-neutral primitives.

---

## Abstract

AI agents increasingly act on behalf of people and organizations — and, increasingly, on each other. The moment two agents commit to something without a human mediating every step, a gap appears that today's agent protocols do not fill: there is no way to make a commitment *binding, private, verifiable, and enforceable* across organizational and trust boundaries.

ANP is an open, **DLT-neutral** protocol that fills exactly that gap. It defines:

1. **Contracting** — how two or more parties form a structured, schema-validated, tamper-evident agreement.
2. **Notarization** — how a credible third party (notary, witness, oracle, data provider, owner) attests to an event, measurement, state, audit, or confirmation.
3. **Dispute Resolution** — how parties agree on a neutral arbiter, submit evidence, obtain a binding ruling, and have it enforced on-chain.

All actors are identified by **W3C Decentralized Identifiers (DIDs)**; all roles, authorities, and attestations are expressed as **W3C Verifiable Credentials (VCs)**. The distributed ledger is used for one job only: to **anchor a cryptographic commitment** (a hash, an ordering, a status) of each binding artifact, while the artifact itself stays off-chain. This single design choice resolves privacy/GDPR exposure, makes the protocol post-quantum-ready at the binding layer, and provides an external source of truth that no agent's context-window reset or compromised database can alter.

ANP does **not** reinvent identity, payments, or attestation. It assembles proven primitives (DID, VC, on-chain anchoring, native settlement/escrow) and contributes the layer that the current ecosystem (A2A, AP2, x402, ERC-8004, EAS) deliberately leaves empty: **a general-purpose, multi-party, chain-neutral mechanism for binding agreements, third-party notarization, and fair dispute resolution.**

The thesis remains: **AI makes DLT usable; DLT makes agents trustworthy.** Neither reaches its potential alone in autonomous machine-to-machine interaction.

---

## Table of Contents

1. [Introduction & Motivation](#1-introduction--motivation)
2. [Terminology & Conventions](#2-terminology--conventions)
3. [Design Goals & Non-Goals](#3-design-goals--non-goals)
4. [Architecture Overview](#4-architecture-overview)
5. [Identity & Trust Model](#5-identity--trust-model)
6. [Data Model & Anchoring](#6-data-model--anchoring)
7. [Pillar I — Contracting](#7-pillar-i--contracting)
8. [Pillar II — Notarization](#8-pillar-ii--notarization)
9. [Pillar III — Dispute Resolution](#9-pillar-iii--dispute-resolution)
10. [Settlement & Economics](#10-settlement--economics)
11. [Security Considerations](#11-security-considerations)
12. [Privacy Considerations](#12-privacy-considerations)
13. [DLT Requirements & Profiles](#13-dlt-requirements--profiles)
14. [Conformance & Profiles](#14-conformance--profiles)
15. [Prior Art & Positioning](#15-prior-art--positioning)
16. [Open Questions](#16-open-questions)
17. [Roadmap](#17-roadmap)
18. [Appendices](#18-appendices)

---

## 1. Introduction & Motivation

### 1.1 The coming collision

Every major AI lab ships agent frameworks. Google's A2A (donated to the Linux Foundation, June 2025) standardizes how agents *discover and talk to* each other; Anthropic's MCP standardizes how agents *use tools and context*. The common assumption is that agents serve humans. But at scale, agents will transact with *each other*: one company's agent needs a translation, a part, an API guarantee, a delivery date; another company's agent offers it. Today a human brokers that. Tomorrow agents will close it end-to-end.

The instant two agents commit without a human in every loop, they need something none of today's communication protocols provide: a **binding agreement layer** — and, by extension, a way to have third parties *vouch* for facts and a way to *resolve* disagreements.

### 1.2 Why agents cannot trust each other

A handshake between humans is backed by reputation, liability, and relationships built over millennia. Agents have none of that, and three structural properties make naive trust impossible:

- **Non-determinism.** The same prompt can yield different outputs. An agent that "agreed" to term X may reinterpret it later.
- **Context volatility.** Agents operate in finite context windows. After a restart, compaction, or memory flush, an agent may have *no recollection* of what it agreed to. This is not an edge case — it is how LLMs work.
- **No persistent identity, no enforcement.** An agent is a process, not a person. Which instance promised what, and what happens if it fails? Without external machinery: nothing.

The user-facing failure modes this protocol must defend against are precise: **hallucinated terms, lost or incomplete memories, and manipulated databases.** A correct design cannot rely on any single party's storage or recollection.

### 1.3 The three needs, generalized

The need for ANP appears wherever an **unambiguous determination is in the interest of at least two parties** — which spans an enormous range, precisely *because* the per-determination cost is meant to be negligible:

- **Contracting.** From a factory purchase agreement down to "we agree this is the API contract," a delivery date, a price, or even an agreed wording. The economic floor for "worth recording" drops to near zero.
- **Notarization.** A credible third party bears witness: an audit result, a situation that occurred, a measurement, an event, a state, or an explicitly requested confirmation of something verifiable. "Notary" here is a *role in the use-case logic*, not necessarily a legal notary — though a real-world authority that can prove its status with a digital identity can occupy that role and gains a correspondingly exposed position in the protocol.
- **Dispute resolution.** A blend of both: parties *agree* on a forum (a contract) and a neutral third party *attests* to and participates in the outcome (an ombudsman/arbiter).

ANP treats these as three pillars of one protocol because they share the same spine: **identity, a signed binding artifact, an on-chain anchor, and conditional settlement.**

### 1.4 The symbiosis thesis (sharpened)

- **AI's problem is trust.** Agents are capable but unreliable as counterparties.
- **DLT's problem is usability.** Wallets, keys, and fees are friction for humans — but *native* to software agents, especially when an operator sponsors fees.
- **The unlock:** agents make DLT usable by absorbing its friction; DLT makes agents trustworthy by providing an external, tamper-evident anchor and a programmable enforcement layer. An agent need not *remember* what it agreed to — it reads the ledger.

This is the design's center of gravity. Everything below follows from it.

---

## 2. Terminology & Conventions

### 2.1 Requirement levels

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in **RFC 2119** and **RFC 8174** when, and only when, they appear in all capitals.

### 2.2 Roles

ANP is a multi-party protocol. The following roles are defined; one real-world entity MAY hold several roles, and a role MAY be filled by an agent, a human, or an organization.

| Role | Definition |
|---|---|
| **Principal** | The root authority on whose behalf an agent acts (a human or organization). Holds a DID and the ultimate signing authority. |
| **Agent** | A software actor that performs ANP actions under a **Mandate** issued by a Principal. Holds its own DID and keys. |
| **Party** | Any signer of an agreement. ANP supports *N* parties, not just two. |
| **Counterparty** | A Party other than the one currently being described. |
| **Notary** | An entity that issues **Attestations** (Pillar II). May be an ordinary peer or an *accredited* authority whose real-world status is proven by a credential. |
| **Witness** | A Notary participating in a **quorum** attestation, where M-of-N witnesses must concur. |
| **Oracle / Data Provider** | A specialized Notary attesting to external/off-chain facts (measurements, prices, events). Subject to bonding. |
| **Arbiter / Ombudsman** | A neutral third party that issues a **Ruling** in a dispute (Pillar III). |
| **Owner** | The controlling party of an asset, resource, or right being transacted or attested. |
| **Verifier** | Any party that checks credentials, signatures, and on-chain anchors. Verification is permissionless. |

### 2.3 Core terms

- **ANP Object** — the atomic, signed, schema-validated unit of the protocol (an offer, an acceptance, an attestation, a ruling, …). The *binding artifact*. Stored off-chain; its hash is anchored on-chain.
- **Anchor** — an on-chain record committing to an ANP Object: `{ object_hash, object_type, thread_ref, status, anchored_by, timestamp }`. Contains **no** payload and **no** PII.
- **Thread** — a hash-linked sequence of ANP Objects sharing a `thread_ref` (a negotiation, a notarization case, a dispute case).
- **Mandate** — a VC issued by a Principal to an Agent defining the Agent's authority envelope.
- **Attestation** — an ANP Object of type `attestation`: a signed statement by a Notary about a subject, anchored on-chain.
- **Suite** — the cryptographic signature/hash algorithm set in force for an Object (see [§6.5](#65-cryptographic-agility--post-quantum-readiness)).
- **Settlement Layer** — the DLT providing anchoring, smart contracts, and native value transfer.
- **Profile** — a concrete binding of ANP's abstract primitives to a specific DLT (see [§13](#13-dlt-requirements--profiles)).

---

## 3. Design Goals & Non-Goals

### 3.1 Goals

1. **Machine-first, unambiguous.** ANP messages are structured data validated against schemas. Natural-language obligations are not binding; ambiguity is a bug.
2. **Anchor, don't publish.** The ledger holds commitments, not content. Binding artifacts live off-chain.
3. **DLT-neutral.** The protocol depends only on cross-platform standards (DID, VC) and abstract DLT capabilities (anchoring, settlement). Chain-specific standards are optional interop profiles.
4. **Identity-rooted.** Every actor and authority is expressed as a DID/VC. No anonymous binding actions.
5. **Authority by mandate, not by mandatory human approval.** Human-in-the-loop is a *configurable threshold*, not a fixed step.
6. **Post-quantum-ready at the binding layer.** Signature suites are agile and MAY be post-quantum today, independent of the chain's native account cryptography.
7. **Privacy-preserving and erasure-compatible.** No personal data on-chain; selective disclosure off-chain.
8. **Cheap and realtime.** Designed for sub-second finality and near-zero per-action cost, so that recording even trivial determinations is economically sensible.
9. **Open.** Open specification, open reference implementation, no proprietary extensions.

### 3.2 Non-Goals

- **Not a token.** ANP introduces no native cryptocurrency. It uses the Settlement Layer's native asset and/or stablecoins.
- **Not a marketplace.** ANP does not match counterparties. Discovery happens elsewhere (e.g., A2A Agent Cards, registries).
- **Not a product.** ANP is infrastructure; products may be built on top.
- **Not a communication protocol.** ANP carries binding artifacts; transport/discovery (A2A, MCP, HTTPS) is out of scope and pluggable.
- **Not a legal framework.** ANP provides cryptographic evidence and programmable enforcement. Legal recognition is jurisdiction-specific and explicitly an open question ([§16](#16-open-questions)).

---

## 4. Architecture Overview

### 4.1 Layered model

```
┌───────────────────────────────────────────────────────────────┐
│  Application      Agent platforms, marketplaces, vertical apps  │
├───────────────────────────────────────────────────────────────┤
│  ANP (Trust Layer)                          ◄── this protocol   │
│  ┌─────────────┐  ┌────────────────────────────────────────┐   │
│  │  Identity   │  │  Pillar I    Pillar II    Pillar III    │   │
│  │  & Trust    │  │  Contract  ·  Notarize  ·  Resolve      │   │
│  └─────────────┘  └────────────────────────────────────────┘   │
├───────────────────────────────────────────────────────────────┤
│  Neutral Primitives                                             │
│  W3C DID  ·  W3C VC 2.0  ·  Anchor(hash)  ·  Settlement/Escrow  │
├───────────────────────────────────────────────────────────────┤
│  DLT (Settlement Layer)                                         │
│  IOTA Rebased (reference)  |  EVM/Base  |  Hedera  |  …          │
└───────────────────────────────────────────────────────────────┘
```

**Adjacent / optional (interoperability, not dependencies):** A2A (communication), the AP2 *mandate pattern* (authority as VCs), and the EVM-specific stack — ERC-8004 (identity/reputation registries), EAS (attestations), x402 (HTTP payments) — which ANP can bind to within an EVM profile but never requires.

### 4.2 The spine: one artifact, four guarantees

Every binding action produces an **ANP Object** that is:

1. **Schema-valid** → no ambiguous obligations (defends against hallucinated/loose terms).
2. **Signed** with an agile, possibly post-quantum suite → authenticity, non-repudiation, quantum-readiness at the binding layer.
3. **Hash-linked** to its predecessor in a Thread → tamper-evident history.
4. **Anchored** on-chain by its hash + status → an external source of truth (defends against lost memory and manipulated databases), while the payload stays off-chain (privacy).

Pillars I–III are three *shapes* of this same spine plus a state machine and a settlement hook. This is what lets one protocol serve contracting, notarization, and dispute resolution coherently rather than as three bolted-together systems.

**Trust posture (stated plainly).** ANP *reduces and makes explicit* trust; it does not claim to eliminate it. The binding/evidentiary layer is tamper-evident and quantum-safe (§6.5); but three residual trust roots remain and are named throughout: (1) the **chain account/consensus** that custodies escrow and orders anchors; (2) a Thread's **terminal arbiter/panel** (§5.4); and (3) **data availability** of off-chain Objects (made a Core MUST, §6.4). Where this document says "verifiable" or "enforceable," read it as *verifiable/enforceable given a valid signature, a posted bond, and an agreed terminal forum* — not as zero-trust. This honest framing is load-bearing for the rest of the spec.

### 4.3 Data flow (typical)

```
Off-chain                                           On-chain (Settlement Layer)
─────────                                           ───────────────────────────
Agent builds ANP Object  ──sign──►  hash ──anchor──►  Anchor{hash,type,thread,status}
Counterparty fetches Object (P2P / shared store)
Counterparty verifies: signature, schema, mandate, prev_hash, on-chain anchor
…thread proceeds…
Conditions met / ruling issued   ──────trigger─────►  Escrow.release / penalty
```

Transport of the Object itself is out of scope: parties MAY exchange Objects directly (A2A/MCP/HTTPS) or via a shared/content-addressed store. Only the **anchor** is authoritative for ordering and existence.

---

## 5. Identity & Trust Model

### 5.1 Identifiers

- Every Principal, Agent, Notary, Oracle, Arbiter, and Owner **MUST** be identified by a **W3C DID** (DID Core 1.0; v1.1 in Candidate Recommendation as of March 2026). The DID method is profile-dependent (e.g., `did:iota`, `did:web`, `did:key`, `did:ethr`).
- An Object's `signature` **MUST** verify against a verification method in the signer's DID document at the time of anchoring. Implementations **SHOULD** record the DID document version/state used.
- Key rotation and revocation are DID-method-specific. Profiles **MUST** state which method(s) they support and the rotation/revocation guarantees those methods provide. ANP does not assume DID Core itself provides rotation semantics (it does not).

### 5.2 Roles and authority as Verifiable Credentials

All non-identity assertions are **VCs** (VC Data Model 2.0, a W3C Recommendation since May 2025), secured via JOSE/COSE or SD-JWT:

- **Role credentials** — e.g., "DID X is an accredited Notary," issued by a recognized authority (an *accreditation issuer*).
- **Mandates** — authority of an Agent to act for a Principal (see §5.3).
- **Attestations** — the substance of Pillar II (a Notary's signed statement; see §8). An Attestation *is* a VC.

A VC by itself proves *who said what*, not that it is true and not that a third party witnessed it at time *T*. **Notarization in ANP = VC (authorship) + on-chain anchor (independent timestamp/ordering) [+ optional witness quorum].** This composition is explicit and load-bearing.

### 5.3 Mandates (delegated authority)

This subsection replaces v0.1's "human approval by default." Instead of forcing a human into every loop, a Principal issues a **Mandate VC** that defines an Agent's authority envelope. The pattern follows Google AP2's mandates (Intent/Cart) but is generalized and standards-only (a VC + a constraints schema), hence DLT-neutral.

A Mandate **MUST** specify at least:

```jsonc
{
  "mandate_id": "urn:uuid:…",
  "principal": "did:iota:…",          // who delegates
  "agent": "did:iota:…",              // who may act
  "scope": ["contracting", "notarization"],   // which pillars
  "constraints": {
    "max_value": { "amount": "5000", "currency": "EUR" },  // per-agreement cap
    "aggregate_value": { "amount": "20000", "currency": "EUR", "window": "P30D" },
    "allowed_counterparties": ["did:iota:…", "*"],
    "allowed_actions": ["offer", "counter_offer", "accept", "attest"],
    "escalation_threshold": { "amount": "1000", "currency": "EUR" } // above → Principal co-sign
  },
  "sub_delegation": { "allowed": false, "max_depth": 0 },
  "not_before": "2026-05-01T00:00:00Z",
  "expires": "2026-08-01T00:00:00Z",
  "status_list": "https://…/status/3#94"   // Bitstring Status List entry for revocation
}
```

**Authority rules (normative):**

- An Agent action is **binding** only if a valid, unexpired, unrevoked Mandate authorizes it *and* the action lies within `constraints`.
- Actions **at or above** `escalation_threshold` **MUST** carry an additional **Principal co-signature** (an `approve` Object signed by the Principal's key). Below it, the Agent acts autonomously.
- Sub-delegation, if allowed, forms a **mandate chain**; verifiers **MUST** validate the entire chain back to the Principal, and profiles **MUST** bound `max_depth` to limit verification cost and DoS surface.
- This model directly serves the **Human → Agent → Agent → Human** topology: each human end sets a Mandate; agents transact autonomously within it; anything exceeding it bubbles back up for a human signature.

**What thresholds can and cannot enforce (M4 — stated honestly):**

- **Per-action value** (`max_value`, `escalation_threshold`) is fully checkable by a counterparty's Verifier: the action's value and the mandate cap are both present, and value **MUST** be expressed in the mandate's currency using a declared FX reference (`constraints.fx_ref`) when the agreement prices in another asset (e.g., `EURe`→`EUR`).
- **Aggregate value** (`aggregate_value` over a window) is **not** enforceable by a counterparty, because an agent's other Threads are private and not globally discoverable. It is enforceable only by (a) the **Principal's own accounting** (replaying the agent's anchored Threads) or (b) a designated **mandate-state oracle** the Principal trusts. The spec states this limitation rather than pretending counterparties can police aggregates; the **per-action** cap is the counterparty-checkable guarantee.
- **Non-Contracting actions need a value mapping.** Notarization and dispute actions have no price. For threshold purposes their "value" is defined per scope: a notarization's value is its posted bond/fee; a dispute action inherits the disputed Thread's escrow value. Mandates **MAY** set separate caps per `scope` so escalation cannot be silently bypassed by routing through Pillar II/III.

### 5.4 Trust framework for third parties

Notaries, Oracles, and Arbiters occupy exposed positions, so their trust is established explicitly:

- **Accreditation.** A third party's authority is a Role VC from an accreditation issuer. Relying parties decide which issuers they trust via **Trust Lists** (an out-of-band, governable set of trusted issuer DIDs). ANP does not centralize this; it standardizes the *format* and *resolution* of trust lists, not their contents.
- **Reputation.** ANP defines a DLT-neutral **reputation interface** (record/fetch signed feedback referencing a Thread), modeled on the ERC-8004 reputation registry but expressed abstractly so any chain can implement it. Heavy data stays off-chain; only signed pointers/hashes are anchored.
- **Bonding & slashing.** A third party **MAY** be required to post a bond (stake) as a precondition to act in a Thread. Misbehavior proven by a Ruling **MAY** slash the bond. This makes "attested-as-reported vs. actually-true" economically meaningful (see §8.4) and deters frivolous or malicious arbitration. Bond *custody* and *slash distribution* are defined in the settlement interface (§10.3).
- **The trust root is terminal and named (M7).** Bonding cannot recurse forever: someone adjudicates the adjudicators. ANP makes this explicit — the **final appeal tier** (a designated Arbiter or M-of-N panel) is **unappealable** and is the Thread's terminal trust anchor. Relying parties accept a Thread *only* if they accept its named final tier (chosen at contract time, §7.3 / §9.2). Honesty over false decentralization: ANP minimizes and bonds trust along the chain, but every Thread rests on a terminal forum the parties agreed to trust.

---

## 6. Data Model & Anchoring

### 6.1 The ANP Object envelope

All Objects share one envelope. Type-specific fields live under `body`.

```jsonc
{
  "anp_version": "0.2",
  "type": "offer | counter_offer | accept | approve | execute | terminate |
           attest | witness | revoke |
           dispute | evidence | rule | appeal | enforce",
  "object_id": "urn:uuid:…",
  "thread_ref": "urn:uuid:…",          // shared across a negotiation/case
  "sequence": 3,                        // monotonic within the thread
  "previous_hash": "sha384:…",          // hash-link to predecessor (or null for the first)
  "timestamp": "2026-05-29T12:00:00Z",  // asserted by signer; on-chain anchor is authoritative
  "suite": "anp-suite-2",               // cryptographic suite identifier (see §6.5)
  "signers": [
    { "did": "did:iota:agentA", "role": "agent",
      "authority": { "mandate": "urn:uuid:…" } }   // mandate (agents) or accreditation (third parties)
  ],
  "body": { /* type-specific, schema-validated */ },
  "proof": [
    { "did": "did:iota:agentA", "alg": "ML-DSA-65", "value": "base64u…" }
  ]
}
```

- `previous_hash` and `sequence` make each Thread an append-only, tamper-evident chain.
- `proof` is an array → multi-signature and quorum are first-class. Each proof entry names its algorithm, supporting per-signer agility.
- **Signer/proof correspondence (normative):** every `proof[]` entry **MUST** correspond 1:1 to a `signers[]` entry with the same `did`; a Verifier **MUST** reject Objects with proofs from non-listed signers or listed signers without a proof. Each signer's `authority` is either a `mandate` (for an Agent acting under §5.3) or an `accreditation` (for a Notary/Oracle/Arbiter under §5.4); the `authority` named is what the Verifier checks the action against.
- **Suite/alg binding (normative):** the `suite` enumerates the permitted signature `alg` values and the hash algorithm; every `proof.alg` **MUST** be a member of the active `suite`, and all Objects in a Thread **MUST** share the Thread's pinned suite (`terms.governing_suite` where present). See [§6.5](#65-cryptographic-agility--post-quantum-readiness).
- `body` **MUST** validate against the JSON Schema registered for `type` (see [Appendix A](#appendix-a-schemas)).

### 6.2 On-chain anchor record

What is written to the ledger:

```jsonc
{
  "object_hash": "sha3-384:…",   // tagged hash of the canonicalized ANP Object (see §6.5)
  "object_type": "accept",
  "thread_ref": "urn:uuid:…",    // opaque random UUID
  "status": "active",            // coarse lifecycle flag (see below)
  "anchored_by": "did:iota:…",
  "timestamp": "<ledger time>",
  "outcome": null                // null except for `enforce`: a minimal numeric settlement directive (see §6.2.1)
}
```

It contains **no payload, no terms, and no free-text personal data.**

**Coarse status, fine state off-chain (m5).** `status` is a deliberately coarse on-chain lifecycle flag — `active | superseded | revoked | disputed | enforced`. The fine-grained pillar states (PROPOSED/ACCEPTED/APPROVED/EXECUTED, ATTESTED/WITNESSED, RULED/FINALIZED, …) are **derived off-chain** by replaying the Thread's anchored Object *types*; the chain need not encode them. Mapping: `active` = thread open/in-progress; `superseded` = this Object was replaced by a later head; `revoked` = attestation/credential withdrawn; `disputed` = an open dispute freezes settlement; `enforced` = a final ruling has been executed.

**Linkability — honest statement (m6).** `thread_ref` is a random UUID, but every anchor also carries `anchored_by` (a DID). On a public ledger this creates a **permanent, un-erasable DID↔Thread link graph**. This is *not* fully privacy-neutral: it does not expose content, but it does expose *who participated in which Thread and when*. For privacy-sensitive Threads, parties **SHOULD** use per-relationship or per-Thread **pairwise DIDs** (§12), trading correlatability against the accountability that named, reputationed actors provide. The GDPR posture (§12) covers the off-chain payload; the on-chain DID linkage is minimized, not eliminated.

#### 6.2.1 Ruling outcome — the one place numbers go on-chain (C1)

Trustless escrow enforcement requires the settlement contract to read *what the ruling decided*, which an opaque hash cannot convey. ANP therefore defines a **minimal, non-PII, numeric settlement directive** carried by an `enforce` anchor:

```jsonc
"outcome": {
  "escrow_id": "0x…",
  "release_bps": 7000,    // basis points to the provider
  "refund_bps": 3000,     // basis points back to the payer
  "penalty_bps": 0,       // additional penalty applied
  "fee_source": "loser_bond",
  "ruling_anchor": "sha3-384:…"   // hash of the off-chain Ruling Object that justifies these numbers
}
```

This is the **trusted input** the escrow contract acts on. "Trustless enforcement" in this protocol therefore means *enforcement that is automatic and verifiable **given a valid arbiter signature and posted bond***, not zero-trust: the numeric directive is authorized by the (bonded, pre-agreed) Arbiter's signature over the `enforce` Object, and the off-chain Ruling at `ruling_anchor` carries the human-readable rationale. The directive contains only integers and references — no personal data.

### 6.3 Canonicalization

To make hashes reproducible, Objects **MUST** be canonicalized before hashing using a deterministic scheme (JCS, RFC 8785, RECOMMENDED). The hash algorithm is governed by the active `suite` ([§6.5](#65-cryptographic-agility--post-quantum-readiness)).

### 6.4 On-chain vs. off-chain — the anchoring pattern

This is the verified, industry-standard pattern (the same rationale behind EAS off-chain attestations) and a deliberate correction of v0.1, which placed full terms on-chain:

- **Off-chain:** the full ANP Object (signed VC-shaped artifact), held by the parties and/or a content-addressed/DA store (e.g., IPFS CID, or a profile-specific availability layer).
- **On-chain:** only the anchor record (§6.2). Many Objects MAY be anchored under a single **Merkle root** to collapse per-Object cost where a profile supports it.
- **Data availability is a Core requirement, not an aspiration (C3).** A hash proves *integrity, existence, and ordering*, not *availability* — and §1.2 forbids depending on any single party's storage. Therefore, when an Object's hash is anchored, the anchoring party **MUST** also, atomically with or before the anchor, either:
  1. **deliver** the full Object to every counterparty named in `signers`/`terms.parties` and collect a **signed receipt** (a lightweight ANP Object acknowledging the hash), **or**
  2. **place** the Object in a content-addressed **DA layer** referenced by the anchor (CID), with a retention term at least as long as the Thread's dispute/appeal windows.
- **Void-on-unavailability (normative):** an anchor whose underlying Object cannot be produced on demand **MUST** be treated as **void** in dispute resolution — it cannot be relied upon as evidence and cannot trigger settlement. This removes the incentive to "anchor-and-withhold": an unverifiable anchor benefits no one.
- Profiles **MUST** specify which DA mechanism(s) they support and the default retention duties.

### 6.5 Cryptographic agility & post-quantum readiness

Quantum resistance is a **hard requirement at the ANP layer** (a chain that cannot at least be paired with quantum-safe binding artifacts is out of consideration). ANP achieves this independently of the chain's native account cryptography:

- **Suites are versioned and negotiable.** A `suite` identifier enumerates the permitted signature algorithm(s) **and** the hash algorithm. Verifiers **MUST** reject Objects whose suite they do not support; Threads **SHOULD** pin a suite at creation, and every Object's `proof.alg` **MUST** be a member of the active suite.
- **Tagged hashes (m3).** All on-chain and `previous_hash` digests **MUST** carry an algorithm tag (multihash, or a `sha3-384:` / `sha384:` prefix) matching the active suite, so a Thread pinned to SHA3-384 is unambiguous. (Examples in this document use `sha3-384:`.)
- **Binding signatures sit on off-chain Objects**, so the signature algorithm is chosen by the parties, *not* dictated by the chain. Suites **MAY** use NIST post-quantum signatures — **ML-DSA (FIPS 204)** or **SLH-DSA (FIPS 205)** — today, or hybrid classical+PQC.
- **On-chain we anchor only hashes**, so hash strength is the relevant chain-side concern. Anchors **MUST** use a hash with ≥256-bit pre-image and ≥192-bit collision resistance against Grover/quantum search — **SHA-384** or **SHA3-384** RECOMMENDED.
- **Binding authority over on-chain *state* (normative).** Anchoring an Object and mutating an anchor's `status` are themselves authority-bearing actions. To prevent a forged chain account from rewriting history, the right to set/change a Thread's anchor `status` (e.g., `superseded`, `revoked`, `disputed`, `enforced`) **MUST** be gated on a valid PQC-suite signature from the entitled DID — i.e., a status transition is only honored if accompanied by (or references) a correctly-signed ANP Object. A chain key alone **MUST NOT** be sufficient to change semantic state. This pushes the quantum-relevant authority back onto the agile binding layer.
- **Honest limitation (settlement & on-chain control).** What still inherits the chain's *native* account cryptography (e.g., Ed25519 on IOTA Rebased/Sui, secp256k1 on EVM — **not** post-quantum today) is: (a) **custody and release of escrowed value**, and (b) the ability to *submit* transactions at all. A quantum adversary who forges the chain account could move escrowed funds or spam anchors **even though** the binding signatures remain unforgeable. Therefore: the **evidentiary/contractual layer is quantum-safe now** and forged anchors cannot change *meaning*; but **escrow custody and transaction submission are only as quantum-safe as the chain.** Mitigations: bound escrow amounts and dispute/appeal time windows; prefer chains with a credible PQC migration plan for accounts; disclose the absence of one as a long-term risk. See [§16](#16-open-questions).

### 6.6 Confidentiality controls

- **Selective disclosure.** Objects MAY be issued as SD-JWT VCs so a party reveals only required fields to a given Verifier (e.g., prove "price ≤ cap" without revealing the price).
- **Commit-reveal.** For sealed bids or sensitive terms, a party MAY anchor only a salted commitment first and reveal the Object later (e.g., during a dispute). The salt prevents dictionary attacks on low-entropy terms.
- **Encryption at rest/in transit** of off-chain Objects is the parties' responsibility; ANP defines the envelope, not the storage.

---

## 7. Pillar I — Contracting

### 7.1 Purpose

Let two or more parties form a binding, private, tamper-evident agreement about anything that can be expressed as structured terms — a purchase, a price, an order, an API contract, a deadline, an agreed wording.

### 7.2 Object types

| `type` | Meaning |
|---|---|
| `offer` | A machine-readable proposal with structured `terms`. |
| `counter_offer` | A modified proposal referencing the previous Object's hash. |
| `accept` | Agreement to the referenced Object's exact terms. |
| `approve` | A Principal co-signature, required when an action meets/exceeds the mandate escalation threshold. |
| `execute` | An **explicit, anchored** Object that opens settlement (escrow). It MAY be auto-*generated* by an agent once the required `accept`/`approve` set exists, but it **MUST** exist as an anchored Object — the `EXECUTED` transition is never implicit, so state is always reconstructable from the ledger (M8). |
| `terminate` | Ends a Thread before binding acceptance. |

**Agreement modes.** An agreement with `terms.escrow.required = false` and no `acceptance_criteria` is a **Memorandum** — a pure mutual statement of record (an API definition, an agreed wording, a deadline acknowledgement). It terminates at `ACCEPTED` (or `APPROVED`); the `EXECUTED`/`DISPUTED` tail does not apply. This is the low-value, high-frequency case the protocol must serve cheaply: cost ≈ one anchor, no settlement. A **Settlement agreement** (escrow required and/or acceptance criteria present) uses the full machine below.

### 7.3 Terms object

`terms` is schema-validated and pillar-extensible. Example (a service agreement):

```jsonc
{
  "subject": "translation",
  "parties": ["did:iota:buyerAgent", "did:iota:sellerAgent"],
  "input_spec":  { "format": "text/plain", "lang": "en", "max_tokens": 5000 },
  "output_spec": { "format": "text/plain", "lang": "de", "quality": "professional" },
  "price": { "amount": "0.50", "currency": "EURe", "settlement_profile": "iota" },
  "deadline": "2026-05-29T13:00:00Z",
  "acceptance_criteria": [
    { "kind": "attestation", "schema": "translation-quality-v1",
      "issuer_trust_list": "urn:anp:trustlist:quality-notaries" }
  ],
  "penalty": { "late_delivery": "10%", "non_delivery": "100%" },
  "escrow": { "required": true, "amount": "0.50", "currency": "EURe" },
  "dispute": {
    "arbiter": "did:iota:arbiterX",
    "process_profile": "urn:anp:dispute:optimistic-v1",
    "challenge_window": "PT24H",
    "appeal": { "allowed": true, "panel": "urn:anp:panel:tier2" }
  },
  "governing_suite": "anp-suite-2"
}
```

Note how `acceptance_criteria` can reference a required **Attestation** (Pillar II) and `dispute` embeds the **arbitration clause** (Pillar III) — the pillars compose.

### 7.4 State machine

```
   ┌─────────┐  offer / counter_offer (n rounds)   ┌──────────┐
   │  DRAFT  │ ───────────────────────────────────►│ PROPOSED │
   └─────────┘                                      └────┬─────┘
        ▲ terminate              accept (all parties, same head hash) │
        │                                                             ▼
   ┌────┴─────┐   below threshold                          ┌──────────────┐
   │TERMINATED│◄── terminate ─────────────────────────────-│   ACCEPTED   │
   └──────────┘                                             └──────┬───────┘
                            approve (if any action ≥ threshold)     │
                                                                    ▼
            Memorandum (no escrow/criteria) ──► [terminal]   ┌──────────────┐
                                                             │   APPROVED   │
                                              execute (anchored) → open escrow │
                                                                    ▼
                                                             ┌──────────────┐
                                                             │   EXECUTED   │  escrow held
                                                             └──────┬───────┘
                                  completion asserted (either party) │
                                                                    ▼
                                                          ── enters optimistic
                                                             window → Pillar III
                                                       (assert → finalize | dispute)
```

**Transition rules (normative):**
- A `counter_offer` **MUST** reference the `previous_hash` of the current Thread head; this preserves the full negotiation history.
- **Single binding release path (M1).** `execute` only opens escrow and reaches `EXECUTED`; it never releases funds. *All* settlement — even when an `acceptance_criteria` attestation exists — flows through the Pillar III optimistic window: a party anchors a **completion assertion**, and the attestation is the *evidence backing that assertion*, not an independent auto-release. This guarantees one finality semantics and prevents racing two release paths.
- **Signer/party invariant (m10).** `ACCEPTED` requires an `accept` from **every** DID in `terms.parties`, each referencing the *same* final head hash, and each `accept`'s `signers` **MUST** include that party's DID. No party may be bound without its own anchored `accept`.
- If any committing action meets/exceeds the actor's mandate `escalation_threshold`, an `approve` from the corresponding Principal is **REQUIRED** to reach `APPROVED`; otherwise `ACCEPTED` proceeds directly to `execute`.

**Multi-party scope (C4).** The Thread is a single, linearly hash-chained structure — inherently single-writer-at-a-time. ANP therefore defines two binding patterns and nothing in between:
- **Bilateral turn-taking** — the 2-party case: alternating `offer`/`counter_offer`, then mutual `accept`.
- **Coordinator-assembled N-party** — for N>2: one Party acts as **coordinator** and assembles the final terms as a single head Object; all other Parties only `accept` that identical head hash (no concurrent counters). This sidesteps fork/ordering ambiguity that free-for-all concurrent counters would create.
- **Accept-invalidation (normative):** any new `counter_offer` referencing the current head **supersedes** it and **invalidates all prior `accept`s** on now-superseded heads (their anchors move to `superseded`). A set of `accept`s is binding only if every one references the *current* head. Concurrent forks (two Objects sharing a `previous_hash`) are resolved by anchor ordering: the first-anchored is the head, the other is `superseded`. Free concurrent multi-party negotiation is explicitly **out of scope** for binding Threads in v0.2.

### 7.5 Anti-hallucination / determinism properties

- **Only structured terms bind.** Free-text is non-binding commentary. "Reasonable timeframe" cannot be a binding term; "2026-05-29T13:00:00Z" can.
- **The ledger is ground truth.** An agent recovering from a context reset reconstructs state by reading anchors + fetching Objects, not from memory.
- **Mandate caps bound blast radius.** A hallucinating agent cannot exceed `max_value`/`aggregate_value`, and high-value actions require a human co-signature.
- **Replay/idempotency.** `object_id` + `sequence` + `previous_hash` make replays and reorderings detectable and rejectable.

---

## 8. Pillar II — Notarization

### 8.1 Purpose

Let a credible third party attest to something verifiable — an event, a measurement, a state, an audit, or an explicitly requested confirmation — and make that attestation independently checkable by anyone, at any time.

**Observed vs. relayed (normative distinction).** An attestation **MUST** declare whether the Notary **independently observed** the subject (`witnessing: "observed"`) or merely **relayed** what a party presented (`witnessing: "relayed"`). A *relayed* confirmation of a self-reported state carries near-zero independent value — it proves only that the party showed the Notary something — and Verifiers **MUST** weight it accordingly. The strongest notarizations are *observed* by an accredited, bonded Notary/Oracle. This makes the "explicitly requested confirmation" case honest about what it does and does not prove.

### 8.2 The Attestation object

An Attestation is an ANP Object of `type: attest` whose `body` is (or wraps) a Verifiable Credential:

```jsonc
{
  "type": "attest",
  "thread_ref": "urn:uuid:…",
  "signers": [{ "did": "did:iota:notaryN", "role": "notary",
                "authority": { "accreditation": "urn:uuid:…" } }],
  "body": {
    "subject_kind": "measurement | event | state | audit | confirmation",
    "witnessing": "observed",            // observed | relayed (see §8.1)
    "statement": {                       // schema-validated per subject_kind
      "claim": "tank-7 temperature at sample time",
      "value": { "quantity": "4.2", "unit": "Cel", "scale": "1" },   // UCUM unit code
      "observed_at": "2026-05-29T11:58:00Z",
      "method": "calibrated-sensor-XYZ#serial123"
    },
    "evidence": [
      { "kind": "c2pa", "hash": "sha384:…", "locator": "ipfs://…" },   // signed media/sensor capture
      { "kind": "document", "hash": "sha384:…" }
    ],
    "confidence": { "model": "interval", "lower": "4.1", "upper": "4.3" },
    "validity": { "not_before": "…", "expires": "…" },
    "status_list": "https://…/status/9#1201"      // for cheap off-chain revocation
  },
  "proof": [ { "did": "did:iota:notaryN", "alg": "ML-DSA-65", "value": "…" } ]
}
```

The Attestation's hash is anchored on-chain (§6.2). **Authorship (VC) + independent timestamp/ordering (anchor) = notarization.**

### 8.3 Witnesses and quorum

For higher assurance, an Attestation MAY require an **M-of-N witness quorum**. To be implementable, the quorum is fully parameterized in the requesting context (a contract's `acceptance_criteria` or a standalone notarization request):

```jsonc
"quorum": {
  "witness_pool": "urn:anp:trustlist:metrology-witnesses",  // named trust list of eligible Witnesses
  "n": 3,                       // size of the eligible/invited set
  "m": 2,                       // concurrences required (m ≥ 1; m=1 is NOT a quorum, it is a single attestation)
  "witness_window": "PT2H"      // witnesses must respond within this window
}
```

Normative rules:
- Each Witness **MUST** be a member of `witness_pool` and **MUST** add an **independent, separately-anchored `witness` Object** referencing the (already-anchored) Attestation hash. The single-object multi-signature form is **not** used for quorum, since independent signers cannot co-sign one canonicalized Object without a defined aggregation scheme.
- A `witness` Object carries a verdict `{concur | dissent}` with an optional reason hash. The Attestation reaches `WITNESSED` when **M concur** within `witness_window`.
- **Dissent and timeout:** if fewer than M concur before the window closes (whether through dissent or silence), the quorum **fails**; the Attestation remains `ANCHORED` but unwitnessed and **MUST NOT** satisfy an `acceptance_criteria` that required the quorum. Dissents are part of the permanent record.

### 8.4 "Attested-as-reported" vs. "true"

An attestation proves that *Notary N stated X at time T*, backed by N's accreditation and bond — **not** that X is objectively true. This distinction is normative:

- Verifiers **MUST** treat an Attestation as "X as reported by N," weighted by N's accreditation, reputation, and bond.
- Oracles/Data Providers attesting to external facts **SHOULD** be bonded; a later Ruling that the report was false **MAY** slash the bond.
- This is the same realism that distinguishes oracle *delivery/consensus* from *ground truth*; ANP encodes it rather than hiding it.

### 8.5 Content/sensor provenance (optional)

When the notarized subject is media or sensor output, the `evidence` entry **SHOULD** use **C2PA** (Content Credentials, spec v2.2, May 2025): the artifact carries a signed provenance manifest, ANP hashes and anchors that artifact, and the Notary attests over it. C2PA is an *input format*, not the anchor; its trust model is PKI/certificate-based, so ANP supplies the immutability and ordering.

**Caveat (m8):** anchoring binds *when ANP saw the manifest*, not that the manifest's capture claim is genuine. A compromised signing certificate or a spoofed capture device produces a forged provenance manifest that ANP will faithfully immortalize. Capture-device authenticity is the Notary's accreditation problem (§5.4), not something anchoring can validate.

### 8.6 Revocation

- **Off-chain (default, cheap):** the Attestation references a **W3C Bitstring Status List** entry; the issuer flips one bit to revoke/suspend. One status credential covers ≥131,072 attestations — marginal cost per revocation is negligible and privacy-preserving ("herd privacy").
- **On-chain (when trustless contract checks are needed):** the anchor `status` is set to `revoked`. This costs a transaction per revocation and therefore does **not** scale for mass revocation — use it only for the small set of attestations a smart contract must check trustlessly.

### 8.7 State machine

```
REQUESTED ──issue──► ATTESTED ──anchor──► ANCHORED ──(quorum required?)──► WITNESSED
                                              │  │                              │
                                              │  └──── quorum fails/times out ──┤ (stays ANCHORED,
                                              │                                 │  unwitnessed)
                                              └─────────── revoke ──────────────┴──► REVOKED
```
**Order matters (M2):** the Attestation is **anchored first**, so its hash exists and is ordered before Witnesses reference it. Each `witness` Object then references the anchored hash and is itself anchored; `WITNESSED` is reached when M concur (§8.3). `REVOKED` is reachable from `ANCHORED` or `WITNESSED` (via the anchor `status`, §8.6). A standalone notarization (no contract) skips `REQUESTED` — a Notary simply issues and anchors.

---

## 9. Pillar III — Dispute Resolution

### 9.1 Purpose

When parties disagree about whether terms were met, resolve it through a forum they agreed to in advance (the arbitration clause), a neutral arbiter, evidence drawn from attestations, a binding ruling, on-chain enforcement, and an optional appeal.

### 9.2 The arbitration clause

Disputes are only fair if the forum is fixed *before* the conflict. A contract's `terms.dispute` (see §7.3) **MUST**, when present, specify: the `arbiter` (or a selection rule/panel), a `process_profile`, a `challenge_window`, escrow handling, and appeal availability. Absent a clause, ANP provides no on-chain enforcement path — only the anchored evidence trail.

**Neutral selection (to avoid re-creating the single-evaluator weakness).** A single pre-named arbiter is just a single evaluator agreed in advance; if one side insists on a captured arbiter, neutrality is lost. The default `process_profile` therefore offers — and above a value threshold **SHOULD** mandate — one of two neutral-selection mechanisms instead of a fixed `arbiter`:
- **VRF draw from a bonded pool:** the arbiter is selected pseudo-randomly (verifiable random function) from a named, bonded pool at dispute time, so neither party chooses it.
- **M-of-N panel:** a panel is drawn from the pool and rules by majority.

Genuine arbiter neutrality at scale remains an open problem (§16.6); ANP specifies the *mechanisms* and leaves pool governance to the trust framework.

### 9.3 Object types

| `type` | Meaning |
|---|---|
| `assert` | A party anchors a claimed outcome — *completion* ("delivered, criteria met") **or** *non-performance* ("counterparty failed to deliver by deadline"). Starts the optimistic window. Either side MAY assert. |
| `dispute` | Challenges an `assert` (or a frozen Thread); freezes the relevant escrow and opens the evidence phase. |
| `evidence` | Submits evidence (typically references to Attestations / Objects by hash). |
| `rule` | The Arbiter's binding **Ruling** (itself an Attestation by the Arbiter), carrying the off-chain rationale. |
| `appeal` | Escalates a Ruling to the final panel, exactly once, if `appeal.allowed`. |
| `enforce` | Anchors the numeric settlement directive (§6.2.1) and executes it on the escrow contract. MAY be auto-generated but **MUST** be an anchored Object. |

### 9.4 Optimistic model (default `process_profile`)

Modeled on UMA-style optimistic oracles — efficient when uncontested, escalating only on challenge:

```
EXECUTED ──assert (completion OR non-performance, either party)──► ASSERTED
   │                                            │ no challenge within window
   │ dispute within challenge_window            ▼
   ▼                                        ┌──────────┐  enforce
CHALLENGED ──evidence (both sides)──► RULED │ FINALIZED│ ──────► escrow settled
   │ rule (Arbiter / panel)                 └──────────┘
   ▼
RULED ──appeal (once) ──► APPEALED ──panel rule (final)──► FINALIZED ──enforce──► settled
   │ no appeal within appeal_window
   ▼
FINALIZED ──enforce──► settled
```

- **Assert-and-wait.** An `assert` stands and auto-finalizes if no `dispute` is filed within `challenge_window`. This keeps the happy path cheap and fast.
- **Non-performance is covered (M5).** The silent non-performer is the canonical agent failure (lost memory, hallucination). If no completion `assert` is anchored by `terms.deadline + grace`, the counterparty MAY anchor a **non-performance `assert`** (or, where the contract allows, trigger automatic refund). Either side can start the clock; a frozen `EXECUTED` Thread cannot strand escrow indefinitely.
- **Challenge → evidence → ruling.** A `dispute` opens the evidence phase; the Arbiter (or panel, §9.2) issues a `rule`. Both challenger and defendant **SHOULD** post a bond; the losing side's bond funds arbiter fees and deters frivolous disputes (shortfall handling in §10.3).
- **Appeal — exactly once (m7).** If `appeal.allowed`, a Ruling MAY be escalated **once** to the final panel within `appeal_window`; the panel's ruling is final. Deeper multi-tier appeals are a separate, non-default profile.
- **Enforcement — verifiable, not cooperation-free (C1).** `enforce` anchors the numeric settlement directive (§6.2.1) signed by the entitled Arbiter/panel; the escrow contract acts on it. Enforcement is **automatic and verifiable given a valid arbiter signature and posted bond** — it does *not* require the *losing* party's cooperation, but it *does* rest on the (bonded, pre-agreed) Arbiter's honesty and the chain account's security. ANP makes that residual trust explicit, bonded, and auditable rather than eliminating it.

### 9.5 Differentiation

ANP aims to be stronger than a single evaluator's binary sign-off (as in Virtuals ACP / the proposed ERC-8183) by **supporting** neutral arbiter selection (VRF / M-of-N panels, §9.2), an explicit single-appeal tier, bonded incentives with slashing, and evidence drawn from the same notarization machinery — all chain-neutral. These are *mechanisms the protocol provides*, not a claim that neutrality is automatically achieved: their effectiveness depends on pool governance and honest final panels (§16.6). The honest framing is "richer, configurable dispute resolution with explicit, bonded trust," not "trustless arbitration."

---

## 10. Settlement & Economics

### 10.1 Settlement abstraction

ANP defines an abstract settlement interface that each Profile binds to native chain capabilities:

```
escrow.open(thread_ref, terms_hash, amount, asset, conditions) → escrow_id
escrow.settle(escrow_id, outcome)        // apply the §6.2.1 directive: release/refund/penalty in one call

bond.post(thread_ref, role, amount, asset) → bond_id   // notary/oracle/arbiter/party stake
bond.slash(bond_id, outcome)                            // executed per a final ruling directive
bond.release(bond_id)                                   // returned when obligations discharged
```

- `escrow.settle` takes the single numeric **outcome directive** (§6.2.1) so release, refund, and penalty are one atomic, verifiable action rather than three racing calls.
- `conditions` reference anchors (a final Ruling / `enforce` directive). Per §7.4, settlement always flows through the optimistic window — there is no separate auto-release path.
- **No `x402` dependency**: where the chain provides native value transfer and stablecoins, ANP uses them directly; x402 is at most an EVM-profile adapter, avoiding redundant payment machinery.

### 10.2 Assets

ANP uses the **Settlement Layer's native asset and/or stablecoins**. Native **EUR/USD stablecoins on the chain** are a valued capability (they widen the realistic use-case range to invoices, deposits, and real settlements) and are listed as a desirable Profile property (§13). ANP itself introduces **no token**.

### 10.3 Incentives for third parties

- **Fees.** Notaries, Oracles, and Arbiters MAY charge fees, paid via the settlement interface (often from escrow or the losing party's bond on finalization).
- **Bond custody.** Bonds are held by the profile's `bond` contract (§10.1), denominated in the Thread's settlement asset, posted as a precondition to act, and released (`bond.release`) once obligations are discharged. A bond is never held by a counterparty.
- **Slash distribution.** A slash is computed against the proven harm, capped at the bond, and directed per the final ruling's `outcome`: typically the **harmed party** is made whole first, then **arbiter/witness fees**, with any remainder **burned or returned to the bond pool** (profile choice). Burning a portion is RECOMMENDED to deter collusion (a colluding pair cannot fully recycle a slashed bond between themselves).
- **Fee shortfall.** If the losing party's bond is smaller than the arbiter fee, the shortfall is drawn from that party's remaining escrow; if still insufficient, the deficit is recorded as negative reputation and (in profiles that require it) the party's minimum bond for future Threads rises. Arbiters MAY decline Threads whose bonds do not cover their fee schedule.
- **Reputation.** The reputation interface (§5.4) lets repeated honest behavior compound into selection advantage — and lets misbehavior be priced in.

---

## 11. Security Considerations

| Threat | Mitigation |
|---|---|
| **Hallucinated / loose terms** | Only schema-valid structured terms bind; free text is non-binding. |
| **Lost memory / context reset** | Ledger anchors + fetched Objects are authoritative; agents reconstruct, never "recall." |
| **Manipulated database** | Off-chain Objects are useless if altered (hash mismatch vs. anchor); ordering/existence are on-chain. |
| **Replay / reordering** | `object_id`, monotonic `sequence`, `previous_hash` chaining; verifiers reject duplicates/gaps. |
| **Over-commitment by an agent** | Mandate `max_value`/`aggregate_value`/`escalation_threshold`; high-value actions need Principal `approve`. |
| **Key compromise** | DID-method key rotation/revocation; suite agility; bounded escrow exposure; mandate expiry. |
| **Sybil third parties** | Accreditation VCs + Trust Lists + bonding; reputation weighting; optional proof-of-humanity for Principals. |
| **Collusion (party↔notary / party↔arbiter)** | Pre-agreed neutral arbiter, M-of-N quorum/panels, bonds + slashing, appeal tier, public anchored evidence. |
| **Mandate-chain abuse / DoS** | Bounded `max_depth`, caching, signature-verification limits per Thread. |
| **Quantum adversary (binding layer & semantic state)** | PQC-capable suites (ML-DSA/SLH-DSA), SHA-384/SHA3-384 anchors; anchor `status` mutations gated on a PQC signature (§6.5) so a forged chain key cannot rewrite *meaning*. |
| **Quantum adversary (settlement & tx submission)** | *Open risk*: escrow custody and the ability to submit transactions inherit chain-native account crypto (Ed25519/secp256k1, not PQC). Bound amounts/time windows; prefer chains with a PQC account roadmap (§16). |
| **Data unavailability / anchor-and-withhold** | Core MUST: deliver Object + signed receipts to counterparties **or** place in a DA layer (§6.4); an anchor whose Object cannot be produced is **void** in dispute and triggers no settlement. |

**Trust assumptions made explicit:** ANP assumes (a) the Settlement Layer provides correct ordering and liveness and is not majority-compromised; (b) accreditation issuers on a relying party's Trust List are honest *to that party's satisfaction*; (c) DID methods deliver the rotation/revocation guarantees their Profile claims. ANP reduces — but does not eliminate — trust; it makes the remaining trust explicit, bonded, and auditable.

---

## 12. Privacy Considerations

- **No PII on-chain, ever.** Anchors carry only hashes, types, opaque `thread_ref`s, status, and a signer DID. Personal/contractual content stays off-chain.
- **GDPR / erasure compatibility.** Because the binding payload is off-chain, the right to erasure can be honored by deleting the off-chain Object *after* the Thread's retention/dispute/appeal windows; the residual on-chain hash is not personal data, and a deleted payload simply becomes unverifiable (and per §6.4 the anchor is then void). Erasure during an open dispute window would void live evidence and is therefore deferred until windows close.
- **Residual on-chain linkage.** The `anchored_by` DID on each anchor creates a permanent, un-erasable DID↔Thread link (§6.2). This is *minimized*, not eliminated; pairwise DIDs (below) reduce it. ANP does not claim full on-chain unlinkability.
- **Selective disclosure.** SD-JWT lets a party prove a predicate (e.g., "within budget," "over 18") without revealing the underlying value.
- **Linkability.** A persistent `thread_ref` and a persistent signer DID can enable correlation. Implementations **SHOULD** support per-relationship or per-Thread pairwise DIDs where unlinkability matters, balancing against the accountability that named, reputationed actors provide.
- **Commit-reveal** protects sensitive or low-entropy terms (salted commitments) until disclosure is necessary (e.g., in a dispute).

---

## 13. DLT Requirements & Profiles

### 13.1 Abstract requirements

A Settlement Layer **MUST** provide:

1. **A cheap anchoring primitive** — commit a hash + small metadata + status, at near-zero marginal cost (directly or via sponsored transactions).
2. **Sub-second to low-second finality** — agents operate at machine speed; multi-minute confirmation breaks the flow.
3. **Programmable smart contracts** — for escrow, conditional release, penalties, and bond/slash logic.
4. **Programmatic wallets** — agents create/use accounts and sign without human UX.
5. **Quantum-pairing compatibility** — it **MUST** be possible to realize ANP's quantum-safe binding layer on top (always true, since binding signatures are off-chain) and the chain **SHOULD** have a credible PQC roadmap for its account/settlement cryptography.

A Settlement Layer **SHOULD** additionally provide: very low/fixed fees; high throughput for concurrent Threads; and — desirably — **native EUR/USD stablecoins**.

### 13.2 Reference profile — IOTA Rebased

IOTA Rebased (mainnet ~May 2025) is the reference example. Verified properties and honest caveats:

- **Move VM, object-centric ledger**, architecturally based on Sui (a modified fork: different tokenomics, validator selection, consensus-robustness tweaks). The object model maps naturally to ANP Threads/anchors as typed on-chain objects.
- **Mysticeti** DAG-based BFT consensus, **delegated Proof-of-Stake**, **sub-second finality (~<500 ms)**; **50,000+ TPS** is a *capacity/test* figure, not a sustained-mainnet guarantee.
- **Fees** exist (Rebased **ended** legacy IOTA's feeless model): reference price ≈ **0.005 IOTA per average transaction** (token-denominated, so the fiat value floats — do not quote a fixed USD price). **Sponsored transactions via the IOTA Gas Station** (launched Alpha in 2025 — new, not battle-hardened) let an operator pay fees so agents transact at zero user cost.
- **IOTA EVM** (L2, Solidity) provides EVM interoperability distinct from the Move L1 — a dual-VM story.
- **Caveats to disclose:** **IOTA Identity is Alpha** (not production-ready early 2026); chain-native signatures are **Ed25519 (not PQC)** — hence ANP's binding-layer PQC strategy and the settlement-layer open risk (§6.5, §16).

### 13.3 EVM interoperability profile (optional)

For deployments wanting alignment with the Ethereum agent stack, an EVM profile MAY bind ANP primitives to existing standards: identity/reputation to **ERC-8004** registries (live on Ethereum mainnet ~Jan 2026), anchoring/attestation to **EAS**, and (optionally) payments to **x402**. These bindings are *interop conveniences*, not dependencies; the abstract ANP semantics are unchanged. Note the trade-off against the project's stated DLT-neutrality and quantum/IOTA preferences: EVM account crypto is also not PQC, and gas variability can create cost barriers for high-frequency anchoring (favoring Merkle batching).

### 13.4 Other candidate chains (context)

| Chain | Fit notes |
|---|---|
| **Sui** | Shares IOTA Rebased's Move/object/Mysticeti lineage; sub-second finality, sponsored tx. Same family, larger ecosystem. |
| **Hedera** | HCS gives fixed, USD-denominated fees (≈ $0.0008/message since Jan 2026) and a few-second deterministic finality — excellent for predictable anchoring budgeting; more enterprise-governed. |
| **Aptos** | Move, ~650 ms finality, sub-cent fees; account-based rather than object-centric; smaller agent-standards ecosystem. |
| **Solana** | Sub-cent fees, high throughput; fee/inclusion variance under congestion. |
| **Base / Ethereum L2s** | Where ERC-8004/EAS/x402 live; strongest standards alignment, but gas variability and non-PQC account crypto. |

Selection is a Profile decision; the abstract requirements (§13.1) are the gate, and **PQC-pairing + sub-second + ultra-cheap** are the hard filters per project policy.

---

## 14. Conformance & Profiles

An implementation conforms to ANP v0.2 if it satisfies the **Core** requirements and at least one **Pillar** and one **Profile**.

- **Core (MUST):** ANP Object envelope (§6.1), canonicalization (§6.3), anchoring pattern (§6.2/§6.4), DID-based identity (§5.1), VC-based roles/mandates (§5.2/§5.3), suite agility with a PQC-capable suite available (§6.5).
- **Pillar profiles (implement ≥1):** Contracting (§7), Notarization (§8), Dispute Resolution (§9). Dispute Resolution conformance requires the optimistic `process_profile` at minimum.
- **Settlement profiles (implement ≥1):** the IOTA reference profile (§13.2) or another profile meeting §13.1, exposing the settlement interface (§10.1).
- **Interop profiles (OPTIONAL):** EVM/ERC-8004/EAS/x402 bindings (§13.3).

Conformance levels: **Minimal** = Core + exactly one of {Notarization, Contracting} + anchoring, *no settlement* (a Memorandum/attestation deployment). **Full** = Core + all three Pillars + a settlement profile + dispute enforcement. **Dispute Resolution conformance requires a settlement profile** (enforcement has nothing to act on otherwise), so it cannot appear at the Minimal level.

---

## 15. Prior Art & Positioning

| Effort | What it does | Relation to ANP |
|---|---|---|
| **A2A** (Google→Linux Foundation, 2025) | Agent discovery & messaging | Adjacent transport; out of ANP scope. |
| **MCP** (Anthropic) | Tool/context access | Orthogonal plumbing. |
| **AP2** (Google, Sep 2025) | Payment **mandates** as signed VCs | ANP adopts the *mandate pattern* (VC-based authority); no hard dependency. |
| **x402** (Coinbase/Cloudflare, 2025) | HTTP-402 stablecoin micropayments | Optional EVM settlement adapter; not required where the chain settles natively. |
| **ERC-8004** (Google+EF+MetaMask+Coinbase, live ~Jan 2026) | On-chain Identity/Reputation/Validation registries | Closest prior art for identity/reputation; *deliberately no escrow/dispute*. ANP adopts the **pattern** (DLT-neutral), binds to it only in the EVM profile. |
| **EAS** | On-chain/off-chain attestations | EVM/Solidity-specific; ANP's notarization is the **DLT-neutral generalization** (VC + anchor). EAS = EVM-profile reference binding + prior art. |
| **Virtuals ACP / proposed ERC-8183** | Client/Provider/Evaluator + escrow + single-evaluator release (Base) | Closest *full-core* prior art. ANP differentiates: chain-neutral; neutral arbiters + M-of-N panels + appeals + slashing; general (any agreement/attestation, not just paid jobs). |
| **W3C DID / VC 2.0** | Identity & credentials | **Foundational dependencies** (the neutral core). |
| **UMA optimistic oracle** | Assert-challenge-resolve for subjective facts | Design template for ANP's optimistic dispute `process_profile`. |
| **C2PA** | Signed content/sensor provenance | Optional evidence format for media/measurement notarization. |

**Positioning in one line:** ANP reuses identity (DID/VC), settlement (native chain), and attestation patterns (EAS-style), aligns with — but does not depend on — A2A/AP2/ERC-8004, and contributes the **chain-neutral, multi-party, general-purpose layer for binding agreements + third-party notarization + fair dispute resolution** that the rest of the stack leaves empty.

---

## 16. Open Questions

1. **Settlement-layer PQC.** The binding layer is quantum-safe today; chain-native account crypto (Ed25519 on IOTA/Sui, secp256k1 on EVM) is not. What is the migration path, and how should escrow exposure be bounded until chains ship PQC accounts?
2. **Trust-list governance.** ANP standardizes trust-list *format/resolution*, not contents. Who curates accreditation issuers in practice — industry consortia, per-vertical registries, reputation-weighted discovery? Avoiding re-centralization is the challenge.
3. **Cross-chain notarization & contracts.** How do anchors and rulings on different Settlement Layers reference and enforce each other? (Bridges, light clients, or a notarized cross-anchor.)
4. **Data availability.** Who guarantees off-chain Object persistence, for how long, and at whose cost? Default DA layer per Profile vs. party-held storage with retention SLAs.
5. **Legal recognition.** When (if ever) does an ANP agreement or notarization carry legal weight in a given jurisdiction, and what mapping to eIDAS/qualified-signature regimes is realistic for "accredited" roles?
6. **Arbiter selection & neutrality.** Mechanisms for selecting genuinely neutral arbiters (random selection from a bonded pool, party-agreed lists, panels) and preventing capture.
7. **Reputation gaming.** How to make the reputation interface resistant to wash-feedback and sybil inflation without a central referee.
8. **Stablecoin availability.** Native EUR/USD stablecoins materially expand use cases but are unevenly available across candidate chains; how central should they be to the reference profile?

---

## 17. Roadmap

**Phase 1 — This document (now).** Publish the v0.2 concept/draft spec; gather feedback; refine.

**Phase 2 — Proof of Concept (per pillar).**
- *Contracting:* two agents negotiate a structured service agreement; Objects anchored on IOTA testnet; mandate-gated approval; escrow auto-release on accepted criteria.
- *Notarization:* a notary/oracle issues and anchors an attestation (with a C2PA-backed measurement); a contract consumes it as an acceptance criterion.
- *Dispute:* an optimistic challenge → arbiter ruling → on-chain enforcement, with bonded parties.
- Open-source from day one; reproducible demo.

**Phase 3 — Specification v1.0.** Freeze envelope, schemas, state machines, suite registry, profile bindings, error/timeout semantics, and the security model; publish conformance tests.

**Phase 4 — Community & ecosystem.** Engage the agent and DLT communities; seek implementers, accreditation issuers, arbiter/oracle providers, and early adopters; converge interop profiles with the broader stack.

---

## 18. Appendices

### Appendix A: Schemas

Normative JSON Schemas (to be finalized in v1.0) will cover:

- the **Object envelope** (§6.1), including `signers[].authority` (mandate | accreditation) and the `proof[]`↔`signers[]` correspondence;
- per-`type` `body` schemas — Pillar I `terms` (incl. `governing_suite`, `escrow`, `acceptance_criteria`, `dispute`), Pillar II `statement` per `subject_kind` with a measurement value object `{quantity, unit (UCUM), scale}` and the `witnessing` flag, Pillar III `evidence`/`rule`;
- the **Mandate** VC (§5.3) incl. `constraints.fx_ref` and per-`scope` caps;
- the **quorum** object (§8.3) `{witness_pool, n, m, witness_window}`;
- the **Anchor record** (§6.2) and the **ruling outcome directive** (§6.2.1) `{escrow_id, release_bps, refund_bps, penalty_bps, fee_source, ruling_anchor}`;
- the **suite registry** (§6.5) mapping suite IDs → permitted `alg` set + tagged hash.

All `body` instances **MUST** validate against the schema registered for their `type`. Canonicalization for hashing: JCS (RFC 8785); digests are algorithm-tagged (§6.5).

### Appendix B: Scenario walkthroughs

These trace the four motivating use cases through the state machines to validate completeness.

**B.1 Factory purchase (high-value Contracting + HITL escalation).**
Buyer-agent `offer` → Seller-agent `counter_offer` (price/delivery) → Buyer-agent `accept`. Value exceeds both mandates' `escalation_threshold` → each Principal adds an `approve`. `execute` (anchored) opens escrow in EURe. Delivery is notarized (B.3-style attestation). The seller anchors a completion `assert` backed by that attestation; with no `dispute` in the `challenge_window`, the Thread `FINALIZED`s and `enforce` settles escrow. Had the seller gone silent past `deadline + grace`, the buyer would anchor a non-performance `assert` and reclaim escrow. Memory loss anywhere is irrelevant — state is reconstructable from anchors + recoverable Objects.

**B.2 API-definition agreement (low-value, high-frequency Contracting).**
Two agents `offer`/`accept` a structured API contract (endpoints, types, versioning, deprecation date). Below threshold → fully autonomous, no human, no escrow needed (Notarization-only is also valid). Cost ≈ one anchor. This is the case that only works because per-determination cost is near zero.

**B.3 Measurement notarization (Pillar II standalone).**
An accredited oracle samples tank-7 temperature, captures a C2PA-signed sensor reading, issues an `attest` (with confidence interval), optionally collects a 2-of-3 witness quorum, and anchors it. Anyone can later verify "oracle O reported 4.2 °C at 11:58, accredited by issuer I, bonded." A consumer treats it as attested-as-reported, not absolute truth.

**B.4 Dispute (Pillar III).**
Seller asserts "delivered, criteria met" → `ASSERTED`. Buyer files `dispute` within the 24 h window → `CHALLENGED`; both post bonds and submit `evidence` (anchored attestations). Pre-agreed arbiter issues a `rule` (30% penalty for late partial delivery). No appeal filed → `FINALIZED`; `enforce` releases 70% to seller, refunds 30% to buyer, and pays the arbiter fee from the loser's bond.

### Appendix C: Changes from v0.1

- **Scope:** single negotiation protocol → three-pillar trust layer (Contracting, Notarization, Dispute Resolution) on a shared identity foundation.
- **Anchoring:** full terms on-chain → **hash on-chain, payload off-chain** (privacy/GDPR fix).
- **Authority:** mandatory human approval → **configurable Mandate model** (HITL as a threshold).
- **Crypto:** added **suite agility + PQC** binding layer and SHA-384/SHA3-384 anchors; documented the settlement-layer quantum caveat.
- **Attribution:** removed the incorrect **"OpenClaw ACP"** citation; positioned against **Virtuals ACP / ERC-8183** instead.
- **Technical accuracy:** clarified IOTA Rebased TPS (capacity), the end of the feeless model, Gas Station alpha status, IOTA Identity alpha status, and Ed25519 (non-PQC) account crypto.
- **Positioning:** explicit reuse of DID/VC + native settlement; A2A out of scope; AP2/ERC-8004/EAS/x402 reframed as optional interop/prior art, not dependencies.
- **Adversarial-review hardening (v0.2 draft):** added the on-chain numeric **ruling outcome directive** (§6.2.1) so escrow enforcement is machine-readable; made **data availability** a Core MUST with **void-on-unavailability** (§6.4); sharpened the **quantum posture** to cover escrow custody and status mutation, not just signatures (§6.5); unified Contracting/Dispute into a **single optimistic release path** (§7.4); scoped **N-party** agreements to bilateral + coordinator-assembled with accept-invalidation (§7.4); fully specified **witness quorum** with dissent/timeout (§8.3); added the **non-performance** dispute path (§9.4) and **neutral arbiter selection** (§9.2); defined **bond custody, slashing distribution, and fee-shortfall** (§10.3); and added an explicit **trust posture** (§4.2).

### Appendix D: References

- W3C **DID Core 1.0** (Rec, 2022); **DIDs v1.1** (CR, Mar 2026); **Controlled Identifiers v1.0** (Rec, May 2025).
- W3C **Verifiable Credentials Data Model 2.0** family (Rec, May 2025), incl. **Bitstring Status List v1.0**, **SD-JWT**.
- **RFC 2119 / 8174** (requirement levels); **RFC 8785** (JCS canonicalization).
- **NIST FIPS 203 (ML-KEM), 204 (ML-DSA), 205 (SLH-DSA)** (PQC, Aug 2024).
- **C2PA** Technical Specification v2.2 (May 2025).
- **Google A2A** (Apr 2025; Linux Foundation, Jun 2025); **Anthropic MCP**.
- **Google AP2** (Sep 2025); **Coinbase x402** (May 2025; x402 Foundation w/ Cloudflare, Sep 2025).
- **ERC-8004 "Trustless Agents"** (proposed Aug 2025; registries live ~Jan 2026); **proposed ERC-8183** (Client/Provider/Evaluator escrow, ~Mar 2026).
- **Ethereum Attestation Service (EAS)** (attest.org).
- **Virtuals Protocol Agent Commerce Protocol (ACP)** (Base).
- **UMA Optimistic Oracle**.
- **IOTA Rebased** (mainnet ~May 2025; Mysticeti consensus; Move VM); **IOTA Identity** (Alpha); **IOTA Gas Station** (Alpha); **IOTA EVM** (L2).
- **Sui**, **Hedera (HCS)**, **Aptos**, **Solana**, **Base/Ethereum L2s** (candidate-chain context).

*(Full URLs and access dates to be attached in the v1.0 bibliography; all technical claims above were source-verified during the v0.2 review.)*

---

*ANP is an open-source infrastructure project. Contributions, feedback, and collaboration are welcome.*

*Author: Christian (byte5 GmbH)*
*Technical elaboration: CyberClaw (AI-Team)*
