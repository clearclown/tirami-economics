> **English translation** — this is a direct translation of `docs/05-banking.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# Chapter 05: Banking and Credit

> Banks are the circulatory system of an economy.
> Stop the blood and the body dies. Flood it and the body bleeds out.

---

## What This Chapter Covers

- What banks do and how fractional reserve banking works
- Forge's lending pool versus traditional banking
- The credit score formula and what each component measures
- The cold-start problem and the welcome loan
- Collateral, default, and what happens when a borrower can't repay
- Circuit breakers — why they exist and exactly what triggers them
- Why the 2008 financial crisis could not have happened in Forge's design

---

## Traditional Banking

### The Basic Function

Banks connect those with surplus money to those who need it. The bank earns the spread
between the rate it charges borrowers and the rate it pays depositors.

Without banks, every borrower would need to find a matching lender directly —
an enormous search cost. Banks aggregate that matching function.

### Credit Creation

The most counterintuitive fact in introductory macroeconomics: banks *create* money.

When a depositor puts $100 into a bank that maintains a 10% reserve ratio, the bank
lends out $90. That $90 eventually returns as a deposit somewhere, enabling $81 to be
lent again, then $72.90, and so on. The original $100 generates up to $1,000 in total
credit — a money multiplier of 10×.

Forge's lending pool operates with a **30% minimum reserve ratio** (`spec/parameters.md §5`),
giving a maximum credit multiplier of **3.3×** — deliberately lower than the 10× typical
of modern banking. Safety over yield.

### The Fragility of Fractional Reserve Banking

The same mechanism that creates liquidity creates fragility. If all depositors demand
their funds simultaneously — a "bank run" — the bank cannot honor the claims: most of
its assets are outstanding loans. This fragility is why central banks exist as "lenders
of last resort."

Forge has no lender of last resort. The circuit breakers described below are its
equivalent — automatic, protocol-level, and pre-committed.

---

## The Forge Lending Pool

### LoanRecord

Every loan in Forge is a `LoanRecord` — a bilateral, Ed25519 dual-signed agreement:

```rust
pub struct LoanRecord {
    pub loan_id:               [u8; 32],   // SHA-256 unique ID
    pub lender:                NodeId,
    pub borrower:              NodeId,
    pub principal_cu:          u64,
    pub interest_rate_per_hour: f64,
    pub term_hours:            u64,
    pub collateral_cu:         u64,
    pub status:                LoanStatus, // Active | Repaid | Defaulted
    pub lender_sig:            [u8; 64],   // Ed25519
    pub borrower_sig:          [u8; 64],   // Ed25519
    pub created_at:            u64,
    pub due_at:                u64,
    pub repaid_at:             Option<u64>,
}
```

`LoanRecord` carries the same design philosophy as `TradeRecord`: dual-signed,
gossip-propagated across the mesh, verifiable by any node. No unilateral lending is
possible. The same bilateral trust model governs both inference trading and credit.

### Forge Lending Pool vs. Traditional Bank

| Property | Traditional bank | Forge lending pool |
|---|---|---|
| Reserve regime | Fractional (often ≤ 10%) | **30% hard minimum** |
| Lender of last resort | Central bank bailout | **None — fail-safe by design** |
| Credit rating | Opaque agency (S&P, Moody's) | **Each node computes locally** |
| Interest rate determination | Central bank policy rate | **Automatic via credit score formula** |
| Loan approval speed | Days to weeks | **Milliseconds** |
| Loan records | Internal bank ledger (private) | **Cryptographically signed, publicly verifiable** |
| Operating hours | Weekdays, business hours | **24/7** |

---

## Credit Scores

### The Formula

Every node carries a credit score computed from its own observed history
(`spec/parameters.md §4`):

```
credit_score = 0.3 × trade_score
             + 0.4 × repayment_score
             + 0.2 × uptime_score
             + 0.1 × age_score
```

The repayment component carries the highest weight (40%) because loan repayment is the
most direct signal of creditworthiness. Trade volume (30%) reflects productive
participation. Uptime (20%) and account age (10%) resist Sybil attacks — a freshly
created node cannot manufacture a high score without sustained honest operation.

### Sub-Score Definitions

| Score | Weight | Range | Formula | What it measures |
|---|---|---|---|---|
| `trade_score` | 30% | [0, 1] | `min(1.0, total_trade_volume / 100,000)` | Lifetime CU traded; saturates at 100k CU |
| `repayment_score` | 40% | [0, 1] | `on_time / total_loans` (0.5 if no loans) | Fraction of loans repaid on time |
| `uptime_score` | 20% | [0, 1] | `hours_online / hours_since_join` | Network uptime ratio |
| `age_score` | 10% | [0, 1] | `min(1.0, days / 90)` | Account age; saturates at 90 days |

**Cold start** (`spec/parameters.md §4`): new nodes begin with credit score 0.3.
**Decay**: after 7 days of inactivity, `uptime_score` decreases by 0.01/day.

### Maximum Borrow Amount

Maximum borrow is *quadratic* in credit score:

```
max_borrow = credit_score² × pool_available × 0.2
```

| Credit score | max_borrow as fraction of pool |
|---|---|
| 0.3 (new node) | 1.8% |
| 0.5 | 5.0% |
| 0.7 | 9.8% |
| 0.9 | 16.2% |
| 1.0 | 20.0% (`spec/parameters.md §5`) |

The quadratic relationship rewards sustained creditworthiness disproportionately, while
strictly limiting the borrowing capacity of unproven nodes.

### Interest Rate Formula

```
offered_rate = base_rate + (1.0 - credit_score) × risk_premium
             = 0.1%/hr  + (1.0 - credit_score) × 0.5%/hr
```

A top-credit borrower (score 1.0) pays 0.10%/hr. A minimum-credit borrower (score 0.3)
pays 0.45%/hr. Nodes with credit below 0.2 are denied credit entirely
(`spec/parameters.md §4`).

---

## The Welcome Loan: Solving Cold Start

### The Bootstrap Problem

A new node has zero CU. Without CU it cannot participate as a consumer to build trade
history, and without trade history it cannot build credit to borrow. The "chicken and
egg" problem.

### Solution: 1,000 CU at 0% for 72 Hours

Every new node may request a **welcome loan** (`spec/parameters.md §3`):
- **Principal:** 1,000 CU
- **Interest:** 0%
- **Term:** 72 hours
- **Collateral:** none required
- **Repayment bonus:** +0.1 credit score if repaid on time (0.3 → 0.4)

The welcome loan is a *credit relationship*, not a gift. It establishes the node's first
repayment record and incentivizes timely repayment via the credit score bonus. It
replaces the flat free-tier grant used in earlier protocol phases.

**Typical bootstrap trajectory:**

```
Day 0:   0 CU → welcome loan → 1,000 CU
Day 3:   repay loan → credit_score: 0.0 → 0.35
Week 1:  ~3,000 CU earned → ~2,000 CU borrowable
Month 1: credit_score 0.55 → ~10,000 CU borrowable
Month 3: credit_score 0.80 → lending CU to others
Month 6: operating a lending position in the pool
```

### Sybil Resistance

An attacker who creates thousands of fake nodes would extract 1,000 CU × N through
welcome loans. Three safeguards prevent this:

1. **Hardware constraint** — many nodes on one machine share hardware; each node's
   inference capacity is diluted, making repayment harder.
2. **Repayment obligation** — fake nodes with low inference throughput are likely to
   default within 72 hours.
3. **Network threshold** (`spec/parameters.md §3`) — if the count of unknown (unverified)
   nodes in the mesh exceeds **100**, welcome loan requests are rejected. An attacker
   would need to keep the fake-node count below this threshold.

---

## Collateral and Default

### Collateral

Loans require collateral. Maximum Loan-to-Value ratio is **3:1** (`spec/parameters.md §5`):
1,000 CU collateral → at most 3,000 CU can be borrowed. Collateral is locked in the
borrower's ledger for the loan duration.

### Default Triggers

Default is automatic:

- **Trigger 1:** `term_hours` elapses with `repaid_at = None`
- **Trigger 2:** borrower node is force-stopped

On default: collateral transfers to the lender; 10% of the collateral is **burned**
(destroyed) as a penalty to deter strategic default (`spec/parameters.md §6`); the
default is gossip-propagated so every node knows.

---

## Circuit Breakers

### Why They Are Necessary

The 2008 financial crisis provides the template: subprime mortgages were securitized
as CDOs, rated AAA by agencies with misaligned incentives, distributed globally, and
when housing prices fell, the cascade of defaults created a credit freeze. Lehman
Brothers failed. The world economy contracted.

Three root causes: too much leverage, opacity, and no automatic shutoff.

Forge addresses all three at the protocol level.

### The Safeguards

| Mechanism | Threshold | Purpose |
|---|---|---|
| Minimum reserve ratio | 30% of pool | Ensures pool liquidity; credit multiplier ≤ 3.3× |
| Max single loan | 20% of pool | Prevents one large borrower from dominating risk |
| Max LTV | 3:1 | Individual loan over-leverage protection |
| Max loan term | 168 hours (7 days) | Limits duration of exposure |
| Velocity circuit breaker | 50% of pool / 1 hr | Stops a sudden burst of new loans |
| Default circuit breaker | 10% of loans / 1 hr | Halts all lending if cascade begins |
| Minimum credit score | 0.2 | Bars untrusted nodes from borrowing |

(`spec/parameters.md §5`, `§6`)

**The default circuit breaker in detail.** If more than 10% of outstanding loans default
within any rolling one-hour window, *all new lending is automatically suspended*
(`spec/parameters.md §6`). Existing loans continue; no existing lender is harmed. Only
new loans are blocked. When the default rate falls below the threshold, lending resumes.
This is the protocol analogue of a circuit breaker in electrical engineering: cut the
circuit before the surge destroys the entire grid.

### Why 2008 Could Not Happen in Forge

| 2008 cause | Forge's structural answer |
|---|---|
| Excessive leverage (LTV > 100%) | Max LTV = 3:1, hard cap |
| Opacity (CDO contents unknown) | All loans carry dual-signed, gossip-verified records |
| Captured rating agencies (AAA on garbage) | No central rating agency; each node scores locally |
| "Too big to fail" moral hazard | No bailout mechanism; no lender of last resort |
| Human panic amplifying losses | AI agents do not panic |
| Regulatory capture (rules changed after crisis) | Circuit breakers are protocol constants; changing them requires network-wide consensus |

---

## Summary

1. **LoanRecord = dual-signed bilateral agreement**, gossip-propagated, verifiable by
   every node. Same cryptographic trust model as TradeRecord.
2. **Credit score formula:** `0.3 × trade + 0.4 × repayment + 0.2 × uptime + 0.1 × age`.
   New nodes start at 0.3. Minimum to borrow: 0.2.
3. **Welcome loan** solves the cold-start problem: 1,000 CU at 0% for 72 hours.
   Timely repayment brings credit score to 0.4. Sybil threshold: 100 unknown nodes.
4. **Quadratic max-borrow** (`credit_score² × pool_available × 0.2`) disproportionately
   rewards sustained creditworthiness.
5. **30% reserve ratio** gives a maximum credit multiplier of 3.3× — deliberately
   conservative compared to conventional banking.
6. **Circuit breakers** are automatic, protocol-level, and pre-committed: velocity limit,
   default rate trigger (10%/hr), collateral burn (10%) on default.

---

→ **[Chapter 06: Exchange and Two Economies](../06-exchange.md)** (Japanese)

← [Chapter 04: Labor and Surplus Value](../04-labor.md) (Japanese) | [Table of Contents](../README.md)

---

### Implementation Reference

| Concept | Rust file | Notes |
|---|---|---|
| `LoanRecord` | `forge-ledger/src/lending.rs` | Dual-signed loan structure |
| `LendingCircuitState` | `forge-ledger/src/lending.rs` | Velocity + default rate circuit breakers |
| Credit score formula | `forge-ledger/src/lending.rs` | §4 parameters |
| `/v1/forge/borrow` | `forge-node/src/api.rs` | Loan request endpoint |
| `/v1/forge/credit` | `forge-node/src/api.rs` | Returns node's credit score |
| `/v1/forge/pool` | `forge-node/src/api.rs` | Pool status: available, utilization, avg rate |

All numeric constants (reserve ratio, LTV, credit weights, default threshold) are in
`spec/parameters.md §3-§6`.

---

*Forge Economics v0.1 — April 2026*
