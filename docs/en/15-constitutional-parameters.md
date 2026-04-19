# 15. Constitutional Parameters — Immutable Economic Invariants

> In an economy without a central bank, what keeps the rule "total supply is
> capped at 2.1 billion TRM" alive? The answer: because governance cannot
> change it.

Before Phase 17, Tirami had a quiet hole in its design. A `ChangeParameter`
governance proposal could, in principle, modify **any** parameter. A faction
that accumulated a stake-weighted majority could, technically speaking,
rewrite `TOTAL_TRM_SUPPLY` from 2.1 billion to 21 billion.

For a protocol whose entire value proposition rests on credibility, that is
a fatal flaw.

Phase 18.1 (Constitution) closes it. This chapter explains why certain
parameters should be unreachable from governance, and compares the approach
with two older ideas from mainstream economics: written constitutions and
central-bank independence.

---

## 15.1 Bitcoin's 21,000,000

Bitcoin's total supply is **21 million BTC**. The line `MAX_MONEY = 21_000_000 * COIN`
in Satoshi's original code defines it.

Why 21 million?
- Satoshi never gave a logical justification.
- There is no intrinsic reason the number had to be 21 million.
- What matters is not the number itself but the empirical fact that **no one has
  managed to change it in fifteen years**.

Bitcoin's hard cap is held in place by **convention and political consensus**
among core developers. Technically, anyone can rewrite the `MAX_MONEY`
constant and recompile — but doing so requires every wallet, exchange, and
node from the past fifteen years to adopt the new version simultaneously.
A faction that refuses to follow splits off and becomes a **different coin**
(a fork). The social cost is enormous, and it is precisely that cost that
generated the trust: "21 M will not change."

---

## 15.2 The Fragility of the Bitcoin Model

The problem is that the guarantee is a **social contract**, not a technical
one.

- As of 2025, Bitcoin's full-node software is maintained by dozens of
  competing factions.
- If one day a faction proposes raising the cap from 21 M to 42 M under
  inflationary pressure, the implementation is **a single `if` branch** at
  the code level.
- The only thing standing in the way is human consensus that "the community
  has not decided to do this."

For a protocol whose founder is long gone, this is brittle. The 2016 Ethereum
DAO hard fork showed what happens when a majority of core developers decide
to rewrite history: the chain gets rewritten.

Tirami avoids this fragility by **enforcing immutability at the code level**.

---

## 15.3 Introducing Constitutional Parameters

Phase 18.1 adds two explicit lists to the Tirami codebase:

```rust
// crates/tirami-ledger/src/governance.rs
pub const MUTABLE_GOVERNANCE_PARAMETERS: &[&str] = &[
    "WELCOME_LOAN_AMOUNT",
    "MAX_LTV_RATIO",
    "MIN_RESERVE_RATIO",
    // ... 21 entries
];

pub const IMMUTABLE_CONSTITUTIONAL_PARAMETERS: &[&str] = &[
    "TOTAL_TRM_SUPPLY",           // 21,000,000,000
    "FLOPS_PER_CU",                // 10^9
    "SLASH_RATE_MINOR",            // 10%
    "SLASH_RATE_MAJOR",            // 30%
    "CANONICAL_BYTES_V2",          // canonical byte order for signatures
    "SIGNATURE_SCHEME_BASE",       // Ed25519
    "PROOF_POLICY_RATCHET",        // monotonically non-decreasing
    "WELCOME_LOAN_SUNSET_EPOCH",   // 2
    // ... 18 entries
];
```

`create_proposal` refuses to accept any proposal whose target parameter name
appears in `IMMUTABLE_CONSTITUTIONAL_PARAMETERS`, or whose name appears in
neither list (unknown parameters are rejected by default).

The consequences are concrete:

- You **cannot** raise `TOTAL_TRM_SUPPLY` from 21 B to 210 B through governance.
- You **cannot** change `FLOPS_PER_CU = 10⁹` (changing it would redefine the
  TRM unit itself).
- You can of course fork the software and rewrite the constant by hand — but
  **that would no longer be Tirami**.

This layers a code-level ratchet on top of Bitcoin's "social-contract"
defense.

---

## 15.4 Written Constitutions and Central-Bank Independence

The closest analogues in mainstream economics are **written constitutions**
and **central-bank independence**.

- The Federal Reserve, the Bank of Japan, the ECB, and their peers are
  formally independent of government, but in practice they remain exposed
  to political pressure (appointments of the chair, summons of the governor,
  and so on).
- The law establishing these institutions can itself be amended by
  legislation.
- And in **Venezuela, Zimbabwe, and Turkey**, regimes that seized operational
  control of monetary policy produced hyperinflation and currency collapse.

The lesson from human economics is blunt: the mechanism that protects the
currency is itself **something humans can change**, and therefore ultimately
fails to protect it.

Tirami's Constitution solves the problem technically:

- The code that *implements* the protection cannot be modified through
  governance.
- Modifying it requires a fork. Once forked, existing users and nodes remain
  on the original chain — the social cost is still there, only now it is
  preceded by a code-level rejection.

---

## 15.5 Drawing the Line Between Mutable and Immutable

Which parameters belong in the constitution, and which remain tunable? The
distinction is not obvious. Phase 18.1 applies the following principles:

### Belongs in the constitution

1. **Definitions that sit at the root of the economy**
   - `TOTAL_TRM_SUPPLY` — changing it rewrites every holder's relative share.
   - `FLOPS_PER_CU` — the unit in which TRM value is measured.
   - `HALVING_THRESHOLDS` — the long-run inflation trajectory.

2. **Lower bounds on attack-deterrent penalties**
   - `SLASH_RATE_MINOR`, `SLASH_RATE_MAJOR` — weakening these erases the cost
     of collusion and Sybil attacks.

3. **Cryptographic-primitive compatibility**
   - `CANONICAL_BYTES_V2` — the byte ordering used when hashing signature
     payloads. Changing it would invalidate every historical dual-signed
     trade.
   - `SIGNATURE_SCHEME_BASE` — Ed25519 cannot be removed.

4. **One-way gates**
   - `PROOF_POLICY_RATCHET` — promotion Optional → Recommended → Required is
     permitted; reversal is not.
   - `WELCOME_LOAN_SUNSET_EPOCH` — once the welcome loan is closed, it cannot
     be reopened.

### Remains tunable

1. **Operational knobs**
   - `MAX_LTV_RATIO` (3:1) — experience may argue for 2:1 or 4:1. Freezing
     this value would prevent the protocol from correcting under- or
     over-lending in response to live data.
   - `WELCOME_LOAN_AMOUNT` (1,000 TRM) — adjustable to the scale of the
     economy.

2. **Technical parameters**
   - `ANCHOR_INTERVAL_SECS` (3600) — how often anchors are written on-chain.
     The right cadence depends on current gas prices and is an operational
     judgment.
   - `AUDIT_CHALLENGE_INTERVAL_SECS` — the audit rate.

3. **Room for economic experimentation**
   - `MIN_PROVIDER_STAKE_TRM` (100) — the stake floor is tunable, but it is
     bounded below by the constitutional constant
     `MIN_PROVIDER_STAKE_CONSTITUTIONAL_FLOOR = 10`. This is a clean example
     of "mutable subject to a constitutional constraint."

---

## 15.6 Why We Protect Relative Structure, Not Absolute Values

An important observation: most of Tirami's constitutional parameters are not
**absolute numbers** — they are **relative structures**.

- `TOTAL_TRM_SUPPLY = 21 B` is an absolute value, but its meaning is the
  relational "cannot be exceeded."
- `SLASH_RATE_MINOR = 10%` is an absolute value, but the underlying invariant
  is the relational notion of "cost per violation."
- `HALVING_THRESHOLDS` is, at bottom, the *shape* of the halving curve — a
  relationship, not a number.

In contrast, the mutable parameters derive their meaning from the absolute
number itself:

- Welcome loan = 1,000 TRM — why 1,000? It depends on the scale of the
  economy.
- LTV = 3:1 — why 3? It is a trial-and-error operational value.

Put differently: a constitution forbids **impossible changes**; concrete
numbers inside that envelope may be updated. The same logic applies to the
U.S. Constitution and ordinary legislation. The Constitution specifies
relational invariants (e.g., "the number of representatives shall not exceed
one for every thirty thousand"), while the specific number of districts is
decided by state law.

---

## 15.7 Making Implementation Verifiable

A constitution only matters if **third parties can verify it**. In Tirami, an
auditor runs a single command and gets the full list of immutable parameters:

```bash
cargo run -p tirami-ledger --example dump-governance-lists
# → MUTABLE_GOVERNANCE_PARAMETERS: 21 entries
# → IMMUTABLE_CONSTITUTIONAL_PARAMETERS: 18 entries
```

This transparency turns the claim "the only path to amendment is a fork"
into something auditable. An auditor can:

- Read the two lists in `governance.rs`.
- Confirm via the twelve unit tests that `create_proposal` rejects any name
  appearing in `IMMUTABLE_*`.
- Inspect `git blame` to ensure no pull request quietly removed an entry
  from `IMMUTABLE_*`.

The net effect: immutability stops being merely a social contract and becomes
something a machine can check.

---

## 15.8 Economic Implications — Credible Neutrality

Vitalik Buterin has popularized the term **Credible Neutrality**: for a
protocol to be fair, it must be demonstrable that no one — *including its
developers* — can bend the rules to their advantage.

Tirami's Constitution implements Credible Neutrality directly:

- Developers cannot submit governance proposals unilaterally (submitting a
  proposal itself requires stake).
- Developers cannot change constitutional parameters (the governance path
  is closed by construction).
- The only option available to developers is to **fork**. Any fork is treated
  as a separate network; users and nodes decide which side to follow.

The overall posture combines Bitcoin's social contract, Ethereum's fork
culture, and Tirami's code-level checking.

---

## Next chapter

§16 takes a concrete case of a constitutional ratchet: **stake-required
mining**. It examines how Phase 18.2 introduced the rule "earning TRM
requires stake" incrementally, why legacy nodes (no stake) retained backward
compatibility, and how the protocol shifts weight toward the new generation
without breaking the old one.

References:

- Implementation: `crates/tirami-ledger/src/governance.rs`
- Design note: `tirami/docs/constitution.md` (English)
- Parameter table: `spec/parameters.md § 20`
