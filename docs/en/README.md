# Tirami Economics — English translations

This directory contains English-language versions of the most important
theory chapters. The canonical source is the Japanese files in the parent
directory; these translations are maintained on a best-effort basis and
may lag behind by one version.

For a fully English academic treatment, see `papers/compute-standard.md`
(v0.1 preprint) and `papers/forge-v0.3-deployment.md` (empirical report).

---

## ⚠️ Before you read

- **TRM is not a financial product.** It is a compute accounting unit (1 TRM = 10⁹ FLOP).
- The maintainers **do not sell, promote, speculate on, or participate in any third-party secondary market for** TRM.
- Because this is MIT OSS, we cannot technically prevent third parties from bridging, listing, or creating derivatives of TRM **without the maintainers' knowledge or involvement**. Anyone who chooses to interact with such secondary markets accepts all resulting risks themselves.
- There is **no ICO, pre-sale, airdrop, private round, or team treasury**. No mechanism exists for any developer to hold TRM except by ordinary mining as a regular provider.
- Details: [tirami/SECURITY.md § Secondary Markets](https://github.com/clearclown/tirami/blob/main/SECURITY.md#secondary-markets--third-party-tokenization) / [`docs/18-secondary-markets.md`](../18-secondary-markets.md).

---

## Status Honesty (2026-04-19)

The Tirami implementation ([clearclown/tirami](https://github.com/clearclown/tirami)) has reached Phase 19. Here is exactly **what is running** and **what is still design-stage**:

✅ **Functional today** (1,192 Rust unit tests + 15 Solidity tests pass):
- HTTP chat inference + dual-signed TRM trade over iroh P2P forward
- Replay protection via 128-bit nonce (`execute_signed_trade`)
- Collusion detector + slashing loop (running continuously in production)
- Welcome loan + constitutional sunset at epoch 2
- Governance proposals (21 mutable parameters, 18 constitutionally immutable)
- Staking pool + referral + ledger persistence

🟡 **Scaffolded (spec and types exist, but not production-wired)**:
- zkML proof-of-inference: only `MockBackend` (shape test, cryptographically invalid). ezkl / risc0 integration slated for Phase 20+ (see [§17](17-proof-of-useful-work.md)).
- Stake-required mining gate: `can_provide_inference` is implemented but not yet enforced in the HTTP/P2P trade execution path (see [§16](16-stake-required-mining.md)).
- ML-DSA post-quantum hybrid signatures: implementation exists, default off (blocked by iroh dependency collision).
- TEE attestation (Apple Secure Enclave / NVIDIA H100 CC): scaffold only.
- Daemon-mode worker gossip recv loop (peer auto-discovery, issue #88).

❌ **Not done**:
- External security audit (Phase 17 Wave 3.3 gate — candidates: Trail of Bits, Zellic, Open Zeppelin, Least Authority).
- Base L2 mainnet deploy (`make deploy-base-mainnet` refuses to run without `AUDIT_CLEARANCE=yes` + `MULTISIG_OWNER` + an interactive prompt; gated until audit completes).
- Live bug bounty (PGP key is documented as placeholder, program not active).
- 10-node 7-day stress test, Base Sepolia 30-day stable operation.

The economic theory in this repository (`docs/00–18.md` + `spec/*`) presumes the status above. In particular, §16–§17 are written as *future design intent*, not current production behavior — each chapter states this explicitly at the top.

---

## Papers PDF (v0.1 working draft)

Academic PDF renderings of the economic specification (`papers/compute-standard.md`) are available for local review:

| Purpose | Artifact | Notes |
|---------|----------|-------|
| Local review | [`papers/build/compute-standard.pdf`](../../papers/build/compute-standard.pdf) | Polished math typesetting (STIX Two Math) for local reading |
| arXiv submission | [`papers/build/compute-standard.arxiv.pdf`](../../papers/build/compute-standard.arxiv.pdf) | Latin Modern + arXiv-compatible, tarball ready to upload ([`.tar.gz`](../../papers/build/compute-standard.arxiv.tar.gz)) |

Build pipeline (pandoc + xelatex): see [`papers/README.md`](../../papers/README.md).

---

## Table of contents

Existing English translations (chapters 00, 02, 03, 05, 14):

- [00 — Introduction](00-introduction.md)
- [02 — Money](02-money.md)
- [03 — Supply and demand](03-supply-demand.md)
- [05 — Banking](05-banking.md)
- [14 — Programmable money](14-programmable-money.md)

New chapters added in this pass (Phase 17-19 reflection):

- [15 — Constitutional parameters](15-constitutional-parameters.md)
- [16 — Stake-required mining](16-stake-required-mining.md)
- [17 — Proof of useful work](17-proof-of-useful-work.md)
- [18 — Secondary markets](18-secondary-markets.md)

Chapters 01, 04, 06-13 remain in Japanese only. Contributions welcome.
