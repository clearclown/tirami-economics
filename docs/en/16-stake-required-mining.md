> **English translation** — this is a direct translation of `docs/16-stake-required-mining.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# 16. Stake-Required Mining — A New Sybil-Resistance Design

> ⚠️ **Implementation status (as of 2026-04-19)**:
> The check function `ComputeLedger::can_provide_inference()` is
> implemented, but **is not yet wired into the trade-execution paths of
> the HTTP `/v1/chat/completions` endpoint or the P2P pipeline** — it is
> exercised only by tests. In other words, the rule this chapter
> describes ("no TRM earnings without stake") is **not enforced in
> current production operation**. Wiring the real gate is scheduled for
> Phase 20.
>
> The following related mechanisms *are* live in the current protocol:
> - The slashing loop (Phase 17 Wave 1.3 — `spawn_slashing_loop`) is
>   running.
> - Staking into `StakingPool` earns a yield bonus (§13).
> - Welcome-loan sunset at epoch 2 (Phase 18.2, constitutional) is
>   enforced in `can_issue_welcome_loan`.
>
> The body of this chapter therefore describes **design intent for the
> near future**, not present-day enforcement.

> You cannot bind "a person with nothing to lose" with economic rules.
> Earlier versions of Tirami made exactly that mistake. Phase 18.2
> corrected it.

Among the constitutional invariants introduced in §15 is
`MIN_PROVIDER_STAKE_CONSTITUTIONAL_FLOOR = 10 TRM`. This is the floor
for the Phase 18.2 rule that "you cannot earn TRM without stake." This
chapter asks three questions: **why require stake at all**, **why the
pre-Phase-17 fully-free configuration was untenable**, and **how this
maps onto the classical economic notion of "skin in the game."**

---

## 16.1 The Hole Before Phase 17

By the time Phase 17 shipped, Tirami already had:

- A slashing mechanism (`apply_slash`) that burns stake on violation.
- A collusion detector covering tight clusters, volume spikes, and
  round-robin patterns.
- An audit system that randomly re-runs suspicious trades.

One **fatal hole** remained, however. Every one of these penalties
implicitly assumed the target had collateral in `StakingPool`, yet the
actual configuration was "no stake required to earn TRM."

Concretely:

```
Provider A: serves compute with 0 TRM staked, continues earning TRM.
           Even if an audit catches a violation, there is no stake to
           burn, so apply_slash burns 0 TRM. No real penalty.
```

A deterrent that cannot be enforced is no deterrent at all. A Bitcoin
miner who attempts a 51% attack makes the attack expensive because the
invested hardware and electricity become **unrecoverable** once the
network forks away from them. It is that cost which produces
deterrence.

Before Phase 17, Tirami's equivalent "cost" was zero.

---

## 16.2 Skin in the Game in Classical Economics

Nassim Taleb's *Skin in the Game* (2018) argues for the importance of
decision-makers bearing the risk of their own choices:

- A surgeon learns to treat patients without risking being cut open
  herself — yet surgeons can still be good surgeons because their
  long-term skin is at stake through **reputation**.
- Investment advisors handle other people's money and so lack direct
  skin in the outcome — which is precisely why supervisory regulation
  (SEC, FSA, and so on) exists.
- Politicians risk losing elections, but do not bear the long-term cost
  of their policies — which is why constitutional and judicial restraint
  is needed.

Tirami providers sit in the same structural position. Unless it is made
explicit what a violator loses, a rational actor will find that
**violating pays better than complying**.

---

## 16.3 The Phase 18.2 Solution

Phase 18.2 made stake mandatory. Requiring a non-zero stake from day
zero, however, would set the entry barrier too high for a fresh
network. Following Filecoin's bootstrap-grant logic, Tirami uses a
**graduated ramp**:

```
New entrant: may join with 0 TRM staked. Cumulative earnings are
             capped at STAKELESS_EARN_CAP_TRM = 10 TRM.

After earning 10 TRM: must stake MIN_PROVIDER_STAKE_TRM = 100 TRM to
                      earn any additional TRM.
```

This is checked at runtime by `can_provide_inference`:

```rust
pub fn can_provide_inference(&self, node_id: &NodeId, pool: &StakingPool, now_ms: u64) -> bool {
    // プライマリパス: active stake が閾値以上
    if pool.active_stake(node_id, now_ms) >= MIN_PROVIDER_STAKE_TRM {
        return true;
    }
    // フォールバック: bootstrap faucet 内
    let earned = self.balances.get(node_id).map(|b| b.contributed).unwrap_or(0);
    let never_slashed = self.slash_events.iter().all(|e| e.node_id != *node_id);
    earned < STAKELESS_EARN_CAP_TRM && never_slashed
}
```

Three properties deserve attention:

1. **A slashed provider loses the faucet path.** The rule "stakeless
   until 10 TRM, then staked from the 11th TRM onward" cannot be gamed
   by violating one's way out: once `slashed`, the provider **cannot
   return to the stakeless bucket**. Rehabilitation requires posting
   stake.

2. **The stake threshold (100 TRM) is tunable, but the constitutional
   floor (10 TRM) is not.** Governance can lower it to 50 TRM; it cannot
   lower it to 5 TRM.

3. **The faucet cap (10 TRM) also has a constitutional ceiling (100
   TRM).** This blocks the "expand the faucet to 1,000,000 TRM and
   effectively zero out the cost" attack.

---

## 16.4 Economic Analysis — Why These Thresholds

Where do the Phase 18.2 numbers (`100 TRM` stake / `10 TRM` faucet)
come from? These are defensible design judgements, arrived at through
the following considerations:

**Rationale for the 100 TRM stake:**

- By the `1 TRM = 10⁹ FLOP` definition, 100 TRM = 10¹¹ FLOP ≈ roughly
  one minute of Apple M2 GPU compute — on the order of a day's worth of
  work under real operation.
- At the maximum slash rate (major = 30%), a violation burns up to
  30 TRM, which is 3% of the welcome loan (1,000 TRM). This is enough
  friction to be felt when a violator attempts re-entry.
- The value is deliberately kept low so as not to become a barrier for
  small providers and crowd out bottom-up participation.

**Rationale for the 10 TRM faucet:**

- A zero-cost slot for new nodes that want to "just try it."
- 10 TRM = 10¹⁰ FLOP ≈ tens of seconds of compute.
- The constitutional ceiling of 100 TRM limits abuse.

**Sybil-resistance calculation:**

- Spinning up 1,000 Sybil nodes to drain the faucet yields at most
  1,000 × 10 = 10,000 TRM of stakeless earnings.
- This is counter-balanced by the per-ASN rate limit from Phase 17
  Wave 4 and the global `slash_events` check from Phase 18.2:
  - If a Sybil commits a violation, it is slashed and can no longer
    return to the stakeless bucket.
  - If it commits no violation, it is providing legitimate
    compute — which is a **net positive** for the network.

Together, these produce the sweet spot: low entry barrier, strong
violation deterrence.

---

## 16.5 Contrast with Traditional Mining

Contrast with Bitcoin mining:

| | Bitcoin | Tirami Phase 18.2 |
|---|---|---|
| Up-front cost | ASIC hardware (thousands of USD) | TRM stake (100 TRM, or 10 TRM faucet) |
| Recoverability | Hardware can be resold | Stake returns on unlock (minus burns) |
| Cost of violation | Fork / hardware becomes worthless | Slashing (stake is burned) |
| Entry barrier | High (hardware required) | Low (10 TRM faucet) |
| Speed of detection | Block validation (~10 min) | Audit + collusion detector (real-time) |

Tirami has a lower entry barrier (thanks to the faucet), faster
detection (audit + collusion), and a violation cost that is
proportionally heavier (100 TRM is a meaningful fraction of a
provider's "capital").

---

## 16.6 The Welcome-Loan Sunset

One consideration is folded quietly into the Phase 18.2 design. The
existing `welcome_loan_amount = 1,000 TRM` mechanism exists to give new
nodes bootstrap capital — which overlaps functionally with the
stakeless faucet.

`WELCOME_LOAN_SUNSET_EPOCH = 2` (constitutional) was therefore
introduced. Once epoch 2 is reached (≥ 87.5% of supply issued), the
welcome loan is retired permanently.

At epoch 2, the network is:

- Already mature (87.5% of supply issued).
- Offering new participants two paths: post stake, or try the 10 TRM
  faucet.
- Viable without the welcome loan.

The philosophy echoes Filecoin's "block reward decays over time":
initial handouts are **graduated out** as the network matures.

---

## 16.7 Auditing the Implementation

For a third-party auditor verifying that stake-required mining is
enforced correctly:

```bash
# 1. 憲法床と閾値の確認
grep -nE "MIN_PROVIDER_STAKE_|STAKELESS_EARN_CAP_|WELCOME_LOAN_SUNSET_" \
  crates/tirami-ledger/src/lending.rs

# 2. can_provide_inference のユニットテスト (+6 件)
cargo test -p tirami-ledger --lib can_provide_inference

# 3. can_provide_inference 実行パス
grep -rnE "can_provide_inference\(" crates/tirami-node/src/
```

The important property is that this check must fire in **both the HTTP
handler and the P2P pipeline**. Gating only one of them would open a
hole: "bypass the stake requirement via the low-latency HTTP path."

---

## 16.8 Balancing Accessibility

Requiring stake naturally makes it harder for "users who have compute
but no TRM" to participate. Tirami softens this along three axes:

1. **Faucet 10 TRM** — a fully free trial allocation.
2. **Welcome loan 1,000 TRM** — a 72-hour loan at 0% interest (while
   it remains pre-sunset).
3. **Referral bonus** — when an existing user refers a new one, both
   receive a bonus.

The path "if you have compute, you can earn TRM even without money"
is preserved. That is intentional: it is what keeps Tirami's founding
premise — **compute is money** — alive in practice.

---

## 16.9 Macroeconomic Implications

At the macro level, stake-required mining affects TRM supply dynamics
in three ways:

- **Lower velocity** — a portion of TRM is always locked into stake.
  Circulating TRM shrinks to `total_supply × (1 - stake_ratio)`.
- **Higher elasticity of supply and demand** — because unlocking
  staked TRM is gated, abrupt sell pressure is dampened.
- **Price stability** — as a consequence, the TRM ↔ compute exchange
  rate stabilizes, with stake acting as a buffer on circulating
  supply.

This is structurally analogous to the "monetary base vs M2"
relationship in modern finance. Tirami's "monetary base" is total
supply; circulating TRM (the M2-equivalent) is the non-staked portion.

---

## Next chapter

§17 turns to the other axis of violation deterrence alongside stake:
**Proof of Useful Work** — that is, **zkML (zero-knowledge
machine-learning proofs)**. It discusses the `ProofPolicy` gate, which
cryptographically proves that a compute result is correct, and how to
operate it through a monotonic ratchet of Optional → Recommended →
Required.

References:

- Implementation: `crates/tirami-ledger/src/lending.rs::{MIN_PROVIDER_STAKE_TRM, STAKELESS_EARN_CAP_TRM}`
- Design: `tirami/docs/constitution.md § Article XI`
- Parameters: `spec/parameters.md § 22, § 23`

---

← [Chapter 15](../15-constitutional-invariants.md) (Japanese) | [Table of Contents](../README.md)

---

*Tirami Economics v0.1 — April 2026*
