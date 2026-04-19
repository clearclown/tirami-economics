> **English translation** — this is a direct translation of `docs/14-programmable-money.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# Chapter 14: Programmable Money and the Hybrid Layer-2 Strategy

> "JPYC is the digitization of the yen. TRM is the monetization of computation.
> They are not competitors — they are complements."
>
> Choosing not to use a blockchain is not a rejection of blockchains. It is using
> the right tool in the right place — that is the design judgment Tirami makes.

---

## What This Chapter Covers

- What programmable money is, and how it relates to smart contracts
- A taxonomy of the main programmable money systems (JPYC, USDC, DAI, CBDC)
- Why Tirami TRM is fundamentally different from all of them
- Five reasons why TRM is not on a blockchain
- The Layer 0 / Layer 1 / Layer 2 three-tier hybrid design
- Concrete integration patterns between JPYC/USDC and TRM (bridge contracts)

---

## Why This Chapter Exists

Two questions arise almost immediately when Tirami's design is explained to someone
familiar with DeFi:

1. "Is this like JPYC?"
2. "Why isn't it on a blockchain?"

Both are reasonable. Programmable money is rapidly maturing — JPYC in Japan, USDC
globally, DAI in DeFi — and "new currency = blockchain" has become a near-universal
assumption. Tirami's choice to run TRM off-chain looks strange in that context.

It is not negligence. It is the result of a specific optimization: **high-frequency,
low-value, AI-agent-native micro-transactions** have requirements that no existing
blockchain can meet. This chapter explains why, and what Tirami does instead.

---

## What is Programmable Money?

### The evolution of currency

```
Commodity money  →  Fiat currency  →  Electronic money  →  Programmable money
(shells, gold)      (notes, coins)     (Suica, PayPal)       (JPYC, USDC, DAI)
     ↓                   ↓                   ↓                      ↓
 Object backs        State backs         Company backs           Code backs
   value               value               value                   value
```

Programmable money allows *conditional* payment: "transfer funds automatically when
the goods are delivered." Traditional electronic money (Suica, PayPal) can only hold
a balance and send it — the execution of conditions requires human or institutional
intermediaries.

Smart contracts — self-executing programs on a blockchain — remove that intermediary.
The code *is* the contract; execution is automatic and immutable.

### Why programmable money matters for AI

AI agents making autonomous economic decisions cannot wait for human approval of each
transaction. They need a currency that can be controlled by code without human
intervention at every step. This is exactly what programmable money enables.

The critical distinction: "programmable money" does not necessarily mean "on-chain
money." Tirami is programmable — its rules are in protocol code — but its core
transactions run off-chain.

---

## The Main Programmable Money Systems

### JPYC (Japanese Yen Peg)

| Property | Value |
|---|---|
| Issuer | JPYC Inc. |
| Peg | 1 JPYC = 1 JPY |
| Chains | Ethereum, Polygon, Astar, Avalanche |
| Regulation | Japan Payment Services Act |
| Purpose | Bring existing yen into Web3 |

JPYC is a wrapper around existing yen. It moves yen into Web3 space for use in
smart contracts. When done, it can be redeemed 1:1 for yen. JPYC creates no new value
— the total yen supply does not change.

### USDC (US Dollar Peg)

The dominant regulated stablecoin globally. Issued by Circle, backed by US Treasuries
and bank deposits with monthly audit reports. Same structural logic as JPYC: a
1:1 wrapper around USD, enabling programmable USD.

### DAI (Decentralized USD Peg)

Issued by MakerDAO (a DAO, not a company), backed by over-collateralized crypto assets
(e.g., 150 USD of ETH deposited to borrow 100 DAI). In practice, a significant fraction
of DAI collateral is USDC, so it is not fully independent. Unlike JPYC/USDC, DAI has
no fiat-holding entity — in theory.

### CBDC (Central Bank Digital Currency)

Central bank digital currencies (China's e-CNY, planned ECB digital euro) are the
furthest extension of programmability: the central bank itself programs the money. The
"programmability" can include expiry dates, restricted spending categories, or automatic
tax withholding. This is monetary sovereignty in its most complete form — and also
the most powerful surveillance and control instrument.

### Comparison (including TRM)

| Dimension | JPYC | USDC | DAI | CBDC | **Tirami TRM** |
|---|---|---|---|---|---|
| Backing | JPY | USD | ETH etc. | Fiat | **Computation (FLOPs)** |
| Issuer | JPYC Inc. | Circle | MakerDAO | Central bank | **The compute node itself** |
| Purpose | Digitize yen | Digitize USD | Decentralized USD | Digital central bank | **Monetize computation** |
| Chain | Ethereum+ | Ethereum+ | Ethereum | Bespoke DLT | **None (off-chain)** |
| Gas cost | Yes | Yes | Yes | Low (assumed) | **None** |
| Speculation | Restricted | Restricted | Possible | Impossible | **Structurally discouraged; protocol does not incentivize it** |
| Typical user | Human | Human | DeFi user | Citizen | **AI agent** |
| Typical tx size | $10–$10,000 | $10–$10,000 | $10–$10,000 | Retail | **$0.0001–$1** |
| Settlement time | Seconds–minutes | Seconds–minutes | Seconds–minutes | Seconds | **Milliseconds** |

TRM is anomalous on nearly every dimension. This is not accidental — Tirami is solving
a fundamentally different problem.

---

## JPYC and TRM: Fundamentally Different

### The Direction of Value

The most important difference is the *direction in which value flows*:

```
JPYC: existing currency (yen) → digitized into Web3 space
      → extension of the existing economy (yen → JPYC → yen)

TRM:   physical computation → new currency issued
      → creation of a new economy (computation → TRM → computation)
```

JPYC flows top-down: take something that already has value and make it more usable.
TRM flows bottom-up: productive physical acts create new monetary units.

### Investment Speculation Resistance

JPYC and USDC are stable because their collateral forces them to be. If Circle's
banking partners fail, the USDC peg breaks. The stability is externally enforced, not
structural.

TRM has no fixed peg, but its issuance is hard-linked to physical computation. The only
way to get more TRM is to do more useful computation or borrow from the lending pool
(Chapter 05). There is no exchange listing, no order book, no speculative on-ramp.
Purely speculative price swings are structurally ruled out.

### Complementarity, Not Competition

TRM and JPYC are not rivals. They operate in different layers:

- JPYC bridges the human economy into Web3
- TRM bridges AI-agent computation into a monetary economy

They *need* each other. When a company wants to fund an AI agent, it sends JPYC to a
bridge contract that credits the agent in TRM. When unused TRM is returned to the company,
it flows back through the bridge as JPYC. This is not a niche integration — it is the
primary path for human-to-AI-economy capital flows.

---

## Five Reasons TRM is Off-Chain

### 1. Gas Costs Make Small Transactions Uneconomical

A typical Tirami inference request costs 0.0001 to 0.01 USD. Ethereum L1 gas for a
simple token transfer is 2–50 USD. That is 200× to 500,000× the transaction value in
fees. Even Polygon (~0.05 USD) and Solana (~0.0005 USD) are unworkable for the small
end of the inference cost range.

```
Transaction value:   ▏            (0.0001 USD)
Ethereum gas:        ████████████████████████ (5 USD = 50,000× the tx value)
Polygon gas:         █████        (0.05 USD = 500×)
Solana gas:          ▏            (0.0005 USD = 5× — marginally viable)
```

An economy where fees routinely exceed transaction values cannot function as an economy.

### 2. On-Chain Listing Imports Speculation

If TRM were an ERC-20 token, someone would create a Uniswap pool within hours. Speculators
would buy expecting appreciation. Price would decouple from compute supply. Providers
would prefer selling TRM on the open market to providing inference. The Bittensor (TAO),
Akash (AKT), and Render (RNDR) tokens all followed this path: speculation overtook
the underlying compute utility as the dominant price driver.

Tirami's design principle is to structurally discourage rather than reward
speculative holding. The protocol does not list TRM on exchanges on its own —
it does not advertise, does not seed liquidity pools, and does not hold a
treasury. Whether a secondary market emerges on third-party infrastructure
is outside maintainer control (see `SECURITY.md § Secondary Markets`), but
the protocol itself does not incentivize it.

### 3. Settlement Latency Is Too High

AI agents can make hundreds of inference decisions per second. Each decision may
trigger a TRM transfer. The required settlement latency is milliseconds.

| System | Settlement latency |
|---|---|
| Tirami (off-chain) | ~1 ms |
| Solana | ~400 ms |
| Polygon | ~2 s |
| Ethereum L1 | ~12 s + confirmation wait |

Ethereum's 12 seconds is *eternal* from an agent's perspective. A system requiring
per-transaction finality cannot serve the real-time inference economy.

### 4. Throughput Requirements Exceed Every Existing Chain

At network scale — millions of agents, each making multiple requests per second —
the required TPS approaches 10⁷. Ethereum manages 15–30 TPS. Solana's peak is ~65,000
TPS. Tirami's design requires the gap to be closed by 2–3 orders of magnitude. Off-chain
bilateral settlement, where only the two parties in a transaction need to agree, provides
the necessary throughput because transactions are not globally ordered.

### 5. Protocol Design Principle

From `tirami/CLAUDE.md`:

> **No blockchain in the core.** TRM accounting uses local ledgers + gossip + dual
> signatures. Bitcoin anchoring is optional and future.

This is not a temporary compromise pending a better blockchain. Off-chain is the
*intended architecture*. The four preceding reasons explain why.

---

## The Three-Layer Hybrid Design

The answer to the limitations of pure off-chain operation is not to move everything
on-chain. It is to use blockchains selectively, where they provide unique value that
the off-chain layer cannot.

```
┌──────────────────────────────────────────────────────┐
│  Layer 0: TRM economy (off-chain)                      │
│                                                       │
│  · Inference trades (thousands per second)            │
│  · Ed25519 dual-signed + gossip (iroh QUIC + Noise)   │
│  · Millisecond settlement, zero gas                   │
│  · 99.9% of all TRM transactions                       │
└──────────────────────────────────────────────────────┘
                        │
                        │  periodic aggregation
                        ▼
┌──────────────────────────────────────────────────────┐
│  Layer 1: Merkle anchor (periodic on-chain)           │
│                                                       │
│  · Merkle root of trade batch written to BTC or ETH  │
│  · Provides immutable audit trail                     │
│  · Cost: one on-chain transaction per few hours       │
│  · "Events from this period cannot be retroactively   │
│     altered" — the only on-chain security guarantee   │
│     that Layer 0 cannot provide alone                 │
└──────────────────────────────────────────────────────┘
                        │
                        │  when needed
                        ▼
┌──────────────────────────────────────────────────────┐
│  Layer 2: Smart contracts (selective on-chain)        │
│                                                       │
│  · DAO governance (parameter change votes)            │
│  · Large-value conditional loan contracts             │
│  · JPYC / USDC ↔ TRM bridge contracts                 │
│  · Insurance, futures, options (tirami-bank roadmap)   │
└──────────────────────────────────────────────────────┘
```

**Layer 0** is where the economy lives. All TRM transactions run here. No gas, no
confirmation wait, no speculative on-ramp.

**Layer 1** provides the one thing off-chain cannot: proof that a historical record
cannot be rewritten. Writing a Merkle root of 1 million transactions to Bitcoin costs
one transaction fee and produces an immutable timestamp for all 1 million records.
This is OpenTimestamps-style anchoring applied to TRM trade history.

**Layer 2** handles the rare cases where smart contracts add genuine value: governance
decisions that benefit from on-chain voting, fiat bridge contracts that need
programmable escrow, and financial derivatives that require atomic settlement guarantees.

The philosophical principle: **use blockchains for the minimum that only blockchains
can provide.** This is not blockchain skepticism or blockchain enthusiasm — it is
engineering appropriateness. Bitcoin Lightning Network uses the same logic: most
payments are off-chain; the chain is for settlement and dispute resolution.

---

## JPYC Integration: A Concrete Scenario

**Scenario:** A company allocates 100,000 yen/month to its AI agent team.

**Step 1 — Deposit:** The company sends 100,000 JPYC to the Tirami bridge contract
on Ethereum L2.

**Step 2 — TRM issuance:** The bridge contract credits the agent wallets with the
equivalent TRM amount.

**Step 3 — Normal operation:** Agents trade inference on Layer 0. No gas, no
confirmation wait.

**Step 4 — Redemption:** Month-end unused TRM flows back through the bridge contract
as JPYC.

**Step 5 — Programmable constraints:** The Layer 2 contract can encode: "maximum
100,000 JPYC; only approved model tiers; expires in 30 days." This is programmable
budget control — the human economy's accounting rules enforced on the AI economy's
TRM flow.

The integration has four benefits:
1. JPYC's programmability (conditional payment, multisig, escrow) is available at the
   funding layer
2. TRM's off-chain efficiency (zero gas, millisecond settlement) is preserved for all
   actual inference trades
3. Human accounting and AI computation are connected at a defined exchange rate
4. JPYC's regulatory compliance (Japan Payment Services Act) covers the fiat side

---

## Open Problems

**Layer 1 anchor frequency.** More frequent anchoring = higher on-chain cost; less
frequent = larger window for undetected tampering. The right cadence (1 hour to 1 day)
requires empirical calibration.

**Bridge security.** Cross-chain bridges are the most frequently attacked component
in the DeFi ecosystem (Ronin, Wormhole, Nomad — billions USD lost). Any Tirami bridge
contract must use cryptographic proof-based design (not trusted validator sets), impose
deposit caps and withdrawal delays to bound blast radius, and undergo independent
security audits.

**Regulatory classification.** TRM does not cleanly fit existing regulatory categories
in any jurisdiction. When TRM bridges to JPYC or USDC, the fiat-side transaction falls
under existing regulations; the TRM side remains in a legal grey area. Clarification
will be needed as the protocol matures.

---

## Summary

1. **TRM is not digitized yen or digitized USD.** It is monetized computation.
   JPYC and TRM serve different layers and are complementary, not competitive.

2. **TRM is off-chain for five structural reasons:** gas economics, speculation isolation,
   settlement latency, throughput, and protocol design principle. These are not
   limitations to be overcome — they are the correct choice for the use case.

3. **The three-layer hybrid design** allocates work appropriately: Layer 0 for all
   routine TRM transactions (off-chain, zero gas); Layer 1 for immutable audit anchoring
   (periodic on-chain, cheap); Layer 2 for governance, bridges, and advanced financial
   contracts (selective on-chain).

4. **JPYC/USDC bridge contracts** on Layer 2 connect human capital allocation to the
   AI compute economy without changing Layer 0's efficiency properties.

---

← [Chapter 13: Open Questions](../13-open-questions.md) (Japanese) | [Table of Contents](../README.md)

---

*Tirami Economics v0.1 — April 2026*
