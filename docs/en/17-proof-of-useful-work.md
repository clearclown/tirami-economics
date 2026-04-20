# 17. Proof of Useful Work — Transitioning to zkML

> ⚠️ **Implementation status (as of 2026-04-19):**
> The zkML verification discussed in this chapter is **not loaded in production**.
> The `tirami-zkml-bench` crate currently ships only a `MockBackend` (for
> shape-testing the harness; cryptographically invalid), and production
> integration of the ezkl / risc0 backends is **scheduled for Phase 20+** and
> not yet implemented. `tirami-ledger::zk::policy_allows_trade()` is likewise
> not called from any live production code path.
>
> This chapter therefore describes **future design intent**, not behaviour
> that ships today. Tirami's current defences against lazy providers are the
> three layers already live — signed trades (dual signature), audit
> challenge, and the collusion detector (summarised in §17.8) — with zkML
> planned as a fourth layer in Phase 20+.

> Bitcoin's PoW bought trust by burning wasted computation.
> Tirami's PoUW tries to *generate* trust from useful computation.
> For that to work, **the correctness of the computation must be
> cryptographically provable**. zkML is the cryptographic primitive
> that makes it possible.

The stake-required mining scheme discussed in §16 is a "violations are
costly" mechanism. But costs are only meaningful if violations can be
detected in the first place. From Phase 17 onwards Tirami exposes three
layers of violation detection:

1. **Collusion detector** — statistical detection of suspicious trade
   patterns.
2. **Audit challenge** — random re-execution to confirm reproducibility
   of results.
3. **Proof of inference** (Phase 18.3+) — a cryptographic proof, via
   zkML, that the computation actually happened as claimed.

This chapter concerns the third and **most important** layer: zkML.

---

## 17.1 The Lazy Provider Attack

Start from a concrete attack scenario. A Tirami provider operates under
the following contract:

- A consumer sends a `chat completion` request (prompt + max_tokens).
- The provider runs a large model (e.g. Llama 70B) and returns tokens.
- The provider claims compute equivalent to the 70B model and is paid
  in TRM accordingly.

Enter the **Lazy Provider**:

```
Malicious provider X:
  - Advertises that it is serving Llama 70B.
  - Actually generates replies using Llama 7B (~10× cheaper).
  - The user cannot easily tell the difference (7B output is still
    grammatical).
  - Collects TRM as if 70B had been used.
```

The collusion detector and audit challenge help against this attack,
but imperfectly:

- They are not perfect — statistical detection has false negatives.
- They are slow — audits lag the trade in time.
- They are expensive — re-execution burns provider-side compute.

The **complete solution** is to cryptographically prove that the output
really was produced by the claimed model. That is what zkML is for.

---

## 17.2 The Basic Idea Behind zkML

zkML (zero-knowledge machine learning) works roughly like this:

1. The provider holds the model weights (a specific commit of
   Llama 70B, say).
2. When producing output `O` for prompt `P`, the provider compiles the
   statement "**using model M, input P produces output O**" into a
   zero-knowledge proof π.
3. The consumer verifies π. It does not need to know the weights of M.
4. If π verifies, it is cryptographically guaranteed that the
   computation really was performed with M.

This renders the lazy provider attack infeasible:

- A 7B reply cannot be accompanied by a π that claims 70B was used.
- Producing a π equivalent to 70B requires actually running 70B.
- Computation cost cannot be forged.

---

## 17.3 Reality Check — zkML Is Heavy

zkML has one crippling practical problem: **proof generation is 10× to
1000× slower than inference itself**.

| Backend | Prove time (1 M-param model) | Verify time | Proof size |
|---|---|---|---|
| ezkl (as of 2025) | 10–60 s | 100 ms | a few KB |
| risc0 | 60 s to several minutes | tens of ms | ~1 KB |
| halo2 custom | requires model-specific tuning | a few ms | ~500 B |

In other words, **requiring a zkML proof on every trade is not viable**.
Making a user wait a full minute for proof generation on every 70B
request would destroy the user experience.

Tirami therefore takes the pragmatic compromise of a **probabilistic
proof strategy**, in the same spirit as Filecoin's Proof-of-Spacetime:

- Demand a zkML proof not on every trade, but on a randomly selected
  subset (e.g. 1%).
- Because providers cannot predict when a proof will be required, they
  are forced to compute honestly at all times (a single proof failure
  triggers slashing).
- The average latency overhead is kept to roughly 1%.

The `ProofPolicy` state machine in Phase 18.3 is the concrete
implementation of this strategy.

---

## 17.4 The Four States of ProofPolicy

```rust
// crates/tirami-ledger/src/zk.rs
pub enum ProofPolicy {
    Disabled,      // proofs are ignored entirely (up to Phase 17)
    Optional,      // reputation bonus if a proof is attached (Phase 19)
    Recommended,   // no proof ⇒ audit-tier penalty
    Required,      // no proof ⇒ rejected by the ledger (constitutional)
}
```

The economic meaning of each state:

### Disabled (before Phase 17)
- Proofs, if produced, are never consulted.
- Lazy providers are deterred only by the collusion detector and audits.
- There is no "cryptographically 100% correct" guarantee.

### Optional (default from Phase 19 onwards)
- Trades with a proof attached receive a reputation bonus.
- Trades without a proof are still valid.
- Providers attach proofs at their discretion.
- This is the **transitional regime while zkML infrastructure matures**.

### Recommended (Phase 20+ target)
- Trades without a proof have their audit tier pinned to `Unverified`.
- `Unverified` is audited 100% of the time, so omitting the proof
  effectively doubles the compute cost (the provider computes once; an
  auditing node recomputes).
- Providers have an economic incentive to attach proofs and lift their
  tier.

### Required (Phase 21+ target)
- Trades without a proof are rejected outright in
  `execute_signed_trade`.
- A zkML proof is **mandatory**.
- Once this state is reached, lazy providers are cryptographically
  impossible.

---

## 17.5 A Monotonic Ratchet — No Rollbacks

Transitions of `ProofPolicy` are constitutionally constrained:

```rust
pub fn try_ratchet_proof_policy(
    current: ProofPolicy,
    proposed: ProofPolicy,
) -> Result<ProofPolicy, ProofPolicyRatchetError> {
    if (proposed as u8) >= (current as u8) {
        Ok(proposed)
    } else {
        Err(ProofPolicyRatchetError::Downgrade)
    }
}
```

Governance can advance `Disabled → Optional → Recommended → Required`,
but never reverse direction.

Why this constraint matters:

- If a proposal like "we ran in `Required` for a year, but it was
  inconvenient, so let's go back to `Optional`" could pass, it would
  betray every user who transacted safely during the `Required` window.
- It is the same category of break as letting Bitcoin's
  `MAX_MONEY = 21 M` be lifted to 42 M years after launch.
- **Once `Required` is reached, it is `Required` forever** — this is
  the strongest possible commitment to users.

---

## 17.6 Economic Analysis — Resolving Information Asymmetry

Michael Spence's signalling theory (1973 Nobel) analysed informational
asymmetries in markets:

- Employers do not know applicants' abilities a priori.
- Applicants signal ability via a costly proxy (education) and let
  employers infer quality.
- If the cost of the signal correlates with ability, the market becomes
  more efficient.

The lazy-provider problem in Tirami is fundamentally the same
informational asymmetry:

- The consumer does not know what the provider actually computed.
- The provider knows exactly what it did.
- The cost of the asymmetry is borne by the consumer (who receives
  low-quality replies).

The lesson from signalling theory:

- The signal (the proof) must be unforgeable in terms of its cost.
- An unforgeable signal reliably reveals the sender's type.

A zkML proof is, by cryptographic design, unforgeable — making it a
near-perfect signal.

---

## 17.7 Comparison With Other PoUW Projects

Tirami's zkML strategy is unusual in the industry. For comparison:

| Project | Correctness verification | Lazy-provider deterrent | Uses zkML |
|---|---|---|---|
| **Bittensor** | None (vulnerable to validator-weight-consumption attack) | Weak (reputation only) | No |
| **io.net** | None (off-chain trust) | Weak | No |
| **Gensyn** | Optimistic proof-of-learning | Moderate (challenge-response) | Partial |
| **Worldcoin** | zk-SNARK-of-humanity | — | Bespoke |
| **Filecoin** | PoSt + probabilistic challenge | Strong | No (storage-specific) |
| **Tirami (target)** | Probabilistic zkML + collusion + audit | **Strong** | **Planned** |

Where Tirami differs:

- It upgrades the "pure trust" model of Bittensor and io.net with
  cryptographic proof.
- It transplants Filecoin's probabilistic-challenge model to the **LLM
  inference** domain.
- It provides a stronger guarantee than Gensyn's optimistic approach
  (no fraud proofs required).

---

## 17.8 The State of Play in 2026

As of April 2026, zkML sits roughly here:

- **ezkl** (ONNX-native) is approaching practicality for LLM inference,
  though 70B-class models are still heavy.
- **risc0** is a general-purpose zkVM that can run Rust code while
  producing a proof, but it is not ML-optimised.
- **halo2** custom circuits are the most efficient option but require
  per-model engineering.

The `tirami-zkml-bench` crate currently looks like this:

- `MockBackend` (always available, cryptographically invalid) — for
  shape-testing the harness.
- `EzklBackend`, `Risc0Backend`, `Halo2Backend` — feature-gated
  scaffolds. Real backend integration lands progressively from Phase
  20 onward.

Practically: `ProofPolicy = Optional` already works, but the only
attachable proofs today come from `MockBackend` and are unsuitable for
production. Real zkML proofs wait for the ezkl integration in Phase 20.

---

## 17.9 Macroeconomic Implications

Consider the economic effects once `ProofPolicy` reaches `Required`:

1. **Higher barrier to entry for providers.**
   - zkML proof generation demands additional GPU time.
   - Small providers (e.g. a MacBook Air) may stop at `Optional`.
   - Providers able to serve `Required` tend to have larger
     infrastructure.

2. **Price stratification.**
   - Providers split into `Required`, `Optional`, and `No proof` tiers.
   - During `Recommended` / `Required` regimes, proof-carrying trades
     command a premium (via the audit-tier-driven reputation gain).

3. **Sybil attacks get harder.**
   - A swarm of 1,000 fake proof-less providers offering cheap service
     is simply rejected by the ledger under `Required`.
   - A Sybil cluster that wants to emit proofs must actually compute —
     pushing the market closer to one that prices raw compute fairly.

4. **Credible neutrality is strengthened.**
   - Under `Required`, any third party can cryptographically verify
     which provider served which request.
   - Even the maintainers cannot install a back door that rescues
     proof-less trades (the constitutional ratchet forbids it).

This is the long-term shape Tirami is aiming at. The 2026 `Optional`
regime is the first step towards it.

---

## Next chapter

§18 takes the paradoxical counterpart to this chapter: the position
that **TRM is not a financial product**, stated as an official
manifesto. It examines the Tirami response to questions like "but what
if a third party creates a market anyway?" and "who carries the risk of
secondary markets?"

References:
- Implementation: `crates/tirami-ledger/src/zk.rs`
- Benchmark crate: `crates/tirami-zkml-bench`
- Design note: `tirami/docs/zkml-strategy.md`
- Parameters: `spec/parameters.md § 24`
