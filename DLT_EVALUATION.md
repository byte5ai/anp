# DLT Evaluation for an ANP Implementation

*Informative companion to [`SPEC.md`](./SPEC.md) §13. Not normative. Snapshot date: **2026-06-10** — finality figures, fees, and upgrade statuses change; re-verify before committing to a profile.*

## Method and hard filters

Per [SPEC §13.1](./SPEC.md#13-dlt-requirements--profiles), plus project policy, a candidate Settlement Layer must pass **both** hard filters:

1. **Sub-second finality** — real, deterministic finality (BFT consensus commit) under ~1 s on mainnet *today*. Optimistic preconfirmations and centralized-sequencer "soft confirmations" do not count.
2. **Very low per-notarization cost** — a small contract-call write (an ANP anchor: hash + ~200 bytes metadata) well under $0.01, predictably.

Additional graded criteria: programmable escrow with time-window logic and **contract-readable anchor state** (the §6.2.1 uncontested path must be able to prove the *absence* of a dispute on-chain — see issue [#6](https://github.com/byte5ai/anp/issues/6)), native on-chain randomness (VRF arbiter/witness draws, [#9](https://github.com/byte5ai/anp/issues/9)), sponsored transactions, native stablecoins, maturity. **Quantum resistance is reported separately for every chain** — it is a graded criterion, not a hard filter, because no candidate chain offers post-quantum account cryptography today; ANP's binding layer is PQC off-chain by design (§6.5).

Findings are source-backed (research notes with URLs are preserved in the repository issues and PR descriptions); numbers flagged *vendor-sourced* or *inferred* were not independently measurable.

---

## Shortlist (passes both hard filters)

| Chain | Finality (mainnet, measured/claimed) | Anchor-write cost (USD) | VM | Native randomness | Sponsored tx | Native stablecoins | Maturity |
|---|---|---|---|---|---|---|---|
| **IOTA Rebased** | ~312 ms p99 consensus commit (Starfish) | ~$0.000045 burned + ~$0.0001 refundable deposit | Move (object) | ✓ beacon + ECVRF | ✓ Gas Station (self-hosted) | bridged USDC.e/USDT only | mainnet 05/2025 |
| **Sui** | ~390–400 ms (Mysticeti v2) | ~$0.002 | Move (object) | ✓ beacon | ✓ protocol-native | USDC ✓ | mainnet 05/2023 |
| **Aptos** | sub-second E2E (~600–900 ms, *vendor-sourced*; Baby Raptr live) | ~$0.001 | Move (account) | ✓ AIP-41 | ✓ fee-payer | USDC ✓ | mainnet 10/2022 |
| **Sei** | ~400–800 ms (Twin-Turbo BFT) | ~$0.0001–0.002 (*inferred*) | EVM (parallel) | ✗ (oracle VRF) | ✗ (ERC-4337) | USDC ✓ | mainnet 2023 |
| **Monad** | ~800 ms full finality (MonadBFT; 1-round speculative) | ~$0.0002 | EVM | ✗ | ✗/unclear | USDC ✓ | **mainnet 11/2025** |

### IOTA Rebased — reference profile (§13.2)

A dedicated implementability check against the current mainnet (protocol v27) found **no hard blockers**; gaps are tracked as issues [#25](https://github.com/byte5ai/anp/issues/25)–[#30](https://github.com/byte5ai/anp/issues/30).

**Pro**
- Cheapest anchoring in the field: ~0.001 IOTA burned (≈ $0.000045) per small anchor object, plus a **100%-refundable** storage deposit (~$0.0001 locked while the anchor lives). Merkle batching is optional, not necessary.
- **Starfish** consensus (Mysticeti successor, live since 04/2026): p99 consensus commit ~312 ms; huge throughput headroom (network averages ~21 TPS against 50k+ capacity).
- Every ANP mechanism verified expressible: shared-object escrow + `Table` dispute registry + `Clock` give the §6.2.1 uncontested-window check; **native randomness beacon and ECVRF** are live on mainnet, so VRF arbiter/witness draws (§8.3/§9.2) work as specced; per-thread anchors as dynamic fields give one-call thread queries.
- Gas Station (v0.5.2, self-hosted) sponsors agent transactions end-to-end, including zero-balance keypairs.
- **IOTA Identity ships PQC VC signatures today** (ML-DSA-44/65/87, SLH-DSA, FALCON, hybrid composites) plus BBS+ ZK selective disclosure — the strongest *binding-layer* PQC story of any candidate, exactly matching §6.5's design.
- First-party **Notarization / Hierarchies / Trust Framework** components to align with ([#30](https://github.com/byte5ai/anp/issues/30)).

**Contra**
- IOTA Identity has **no stable release** (v1.9.9-beta.1) and lacks the spec-mandated Bitstring Status List ([#27](https://github.com/byte5ai/anp/issues/27)).
- No EUR stablecoin, USD bridged-only; the spec's EURe examples don't work here ([#28](https://github.com/byte5ai/anp/issues/28)).
- No 384-bit hash natives in Move — anchors store digests as opaque bytes; on-chain recomputation needs a 256-bit profile hash ([#25](https://github.com/byte5ai/anp/issues/25)).
- Thin general-purpose ecosystem; Foundation strategy is pivoting to global-trade infrastructure — smaller tooling/liquidity base than Sui.
- Token-denominated fees float with the IOTA price (currently negligible; a 10–50× repricing would still keep anchors under a cent).

**Quantum resistance**
- Accounts/consensus: Ed25519/BLS12-381, **no published L1 PQC roadmap** — the §13.1 SHOULD is unmet; escrow custody carries the §6.5 open risk.
- Binding layer: best-in-field — PQC and hybrid VC signatures shipped in the identity stack (off-chain, which is where ANP needs them).
- No PQC verification natives in the Move VM, none on the roadmap → the §6.5 status-gate must be enforced optimistically ([#26](https://github.com/byte5ai/anp/issues/26)).

### Sui

**Pro**
- ~390–400 ms verified mainnet finality (Mysticeti v2); ~$0.002 predictable fees with epoch-auctioned reference gas price and storage rebates.
- Same Move/object/consensus family as IOTA Rebased — **one ANP Move profile can cover both chains** with minor divergence handling; a credible fallback if IOTA ecosystem risk materializes.
- Native randomness (`Random`), `Clock`, protocol-native sponsored transactions, plus protocol-level gasless stablecoin transfers (shipped 05/2026).
- Native Circle USDC; mature, large ecosystem (mainnet since 2023).

**Contra**
- ~40× higher anchor cost than IOTA (still well under the filter).
- No EURC/EUR stablecoin.
- No 384-bit hash, no PQC natives (same profile caveats as IOTA, [#25](https://github.com/byte5ai/anp/issues/25)/[#26](https://github.com/byte5ai/anp/issues/26) apply analogously).

**Quantum resistance**
- Ed25519 accounts, nothing shipped. Published research is real but early: Mysten's crypto-agility work and a fork-free PQ upgrade path for EdDSA chains (with George Mason University). Better than silence, weaker than Aptos's accepted AIP.

### Aptos

**Pro**
- Sub-second end-to-end finality with **Baby Raptr live on mainnet**; full Raptr/Zaptos landing through 2026 (figures currently vendor-sourced ~600–900 ms — flag for independent measurement).
- ~$0.001 per small transaction; native fee-payer (sponsor) transactions; native AIP-41 randomness; native USDC.
- **The only candidate with an *accepted* account-level PQC migration plan: AIP-137** (SLH-DSA-SHA2-128s post-quantum accounts, FIPS 205, status *Accepted* 02/2026; mainnet activation not yet confirmed). Directly addresses ANP's escrow-custody quantum risk (§6.5/§16.1) — no other shortlist chain has anything comparable.
- sha2-512/sha3-512 natives exist (closest any candidate gets to ANP's ≥384-bit hash family for on-chain recomputation).

**Contra**
- **Account-centric Move** — a different framework from the Sui/IOTA object model; an Aptos profile is a separate implementation effort, not a port (anchor-as-object and per-thread dynamic fields need redesign around tables/resources).
- Sub-second claim rests on vendor benchmarks; independent ms-precise mainnet measurements were not found.
- No EUR stablecoin; smaller agent-standards ecosystem than EVM chains.

**Quantum resistance**
- Ed25519/secp256k1 today; **AIP-137 accepted** (see above) — the best PQC trajectory in the field. If escrow-custody quantum risk becomes the dominant selection criterion, Aptos moves to the front.

### Sei

**Pro**
- ~400 ms single-slot BFT finality (some sources put hard determinism at 2 blocks ≈ 800 ms — still sub-second); fees clearly ≪ $0.01.
- **Full EVM compatibility** → cheapest path to the Ethereum agent-standards stack (ERC-8004, EAS, x402) *without* sacrificing the finality filter — effectively SPEC §13.3's interop profile on a sub-second chain.
- Native Circle USDC (issued directly on Sei).

**Contra**
- No native randomness (needs Pyth Entropy or similar oracle VRF — an extra trust component for §9.2 draws).
- No protocol-native sponsorship (ERC-4337 paymaster machinery instead).
- **Giga** migration (Autobahn consensus, EVM-only future) rolling out through 2026 — platform transition risk; relatively small/concentrated validator set.
- Average fee in USD is inferred from gas model + token price; no authoritative published average.

**Quantum resistance**
- secp256k1/Ed25519; **no PQC roadmap found** (verified absence). Standard EVM hash set only.

### Monad — conditional entry

**Pro**
- Passes both filters with strong numbers: ~800 ms full deterministic finality (MonadBFT), ~$0.0002 per escrow-sized call, full EVM bytecode compatibility (ERC-8004/EAS deployable), native USDC.

**Contra**
- **Mainnet only since 2025-11-24** — about six months of history. Unproven under stress, unproven economic security; too young to custody ANP escrow in production. Re-rank in 12 months.
- Gas quirk: fees charge the gas *limit*, not gas *used* — over-estimation costs real money; needs disciplined gas estimation in the anchoring client.
- No native randomness, no native sponsorship (status unclear), no PQC roadmap.

**Quantum resistance**
- secp256k1 only; nothing published.

---

## Evaluated and excluded

| Chain | Failing criterion (measured/verified) | Note |
|---|---|---|
| **Solana** | Finality ~12.8 s today — **Alpenglow is not on mainnet** (live on a community test cluster since 05/2026; mainnet target Q3 2026) | Fees pass (~$0.00025). **Re-evaluate the day Alpenglow activates** (~150 ms promised) — would then be a serious candidate. |
| **Avalanche** | Finality ~1–2 s — not reliably sub-second | |
| **Hedera** | Finality ~3 s | HCS fixed-USD fees ($0.0008/msg) are a predictability gold standard, and Hedera runs SHA-384/CNSA-suite internally — but HCS messages are **not smart-contract-readable**, which independently breaks the §6.2.1 uncontested path (see [#6](https://github.com/byte5ai/anp/issues/6)). SPEC §13.4's Hedera row needs this caveat. |
| **Algorand** | Finality ~2.8 s | **The only chain with shipped protocol-level PQC**: Falcon-signed State Proofs (since 2022). Worth watching as PQC prior art despite the finality miss. |
| **MegaETH** (and "real-time" L2s) | "~10 ms" is centralized-sequencer preconfirmation; true finality inherits Ethereum L1 (~13 min) | No L2 can honestly claim sub-second *finality* today. |
| **Base / Arbitrum / OP** | L1-anchored finality ~13–17 min | Strongest ERC-8004/EAS/x402 ecosystem — remains the natural home of SPEC §13.3's *interop* profile, just not of sub-second settlement. |
| **Stellar** | Ledger close ~5.8 s | Fees pass ($0.00001). **Only chain in the field with native EURC** — relevant to the §16.8 EUR-stablecoin question. |
| **TON** | Conflicting finality data (0.8 s vs ~5 s masterchain settlement); fees ~$0.01+, variable | Lean fail on both filters. |
| **ICP** | Update-call (write) latency ~1–2 s | |
| **Sonic** | Claimed ~720 ms vs independent reports of 1–2 s — conflicting | Don't shortlist until independently measured. |
| **BNB Chain / NEAR** | Dashboard figures (650/600 ms) conflict with official docs (~1.2–2 s) | Follow-up check worthwhile; not shortlisted on conflicting data. |

---

## Quantum resistance — cross-cutting summary

Reported separately per project policy. State of the field, 2026-06-10:

| Tier | Chains | What exists |
|---|---|---|
| **Shipped, protocol-level** | Algorand *(excluded — finality)* | Falcon-signed State Proofs since 2022; first PQ-signed mainnet tx 2025. |
| **Accepted roadmap, account-level** | **Aptos** | AIP-137: SLH-DSA-SHA2-128s post-quantum accounts (FIPS 205), status *Accepted*; mainnet activation unconfirmed. Best trajectory on the shortlist. |
| **Published research, nothing shipped** | Sui, Solana, Ethereum | Mysten crypto-agility + fork-free EdDSA→PQ path; Solana opt-in Winternitz vault; Ethereum lean-roadmap PQ research. |
| **Binding-layer PQC (off-chain)** | **IOTA** | Identity stack ships ML-DSA/SLH-DSA/FALCON + hybrid VC signatures — covers ANP's *binding* layer (§6.5), not L1 accounts. |
| **Nothing found** | Sei, Monad, Sonic, Avalanche, TON, Stellar, Hedera¹ | ¹ Hedera's internal SHA-384/CNSA hashing is a hash-strength margin, not account PQC. |

Two conclusions for ANP:

1. **No candidate chain can verify PQC signatures in-contract today** → ANP's design decision to keep binding signatures off-chain and quantum-safe (§6.5) is validated; the §6.5 status-gate needs the optimistic-enforcement binding on every shortlist chain ([#7](https://github.com/byte5ai/anp/issues/7), [#26](https://github.com/byte5ai/anp/issues/26)).
2. **Escrow custody remains the quantum-exposed surface everywhere** (§16.1). Today that argues for bounded escrow amounts and short windows (already in the spec); structurally, Aptos AIP-137 is the only credible path to closing it at the account layer.

---

## Recommendation

1. **IOTA Rebased stays the reference profile** — confirmed implementable end-to-end with no hard blockers, the cheapest anchoring in the field, live randomness/ECVRF, and a binding-layer PQC story none of the others match. The open items are tracked ([#25](https://github.com/byte5ai/anp/issues/25)–[#30](https://github.com/byte5ai/anp/issues/30)); the real risks are ecosystem thinness and the beta identity stack, not protocol capability.
2. **Sui is the designated fallback / second Move profile** — same family, ~95% shared profile work, larger ecosystem; implement the Move profile so it runs on both from day one.
3. **Aptos is the quantum hedge** — track AIP-137 activation; if escrow-custody PQC becomes decisive (regulatory or threat-model driven), it is the only chain with an accepted answer.
4. **Sei is the EVM-interop bridge candidate** — ERC-8004/EAS alignment at sub-second finality, accepting oracle-VRF and paymaster trade-offs.
5. **Watch list:** Solana (re-evaluate at Alpenglow mainnet activation), Monad (re-rank after ~12 months of mainnet history), Sonic/BNB/NEAR (pending independent finality measurements).
