> **English translation** — this is a direct translation of `docs/03-supply-demand.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# Chapter 03: Supply and Demand

> "The Tirami TRM market is probably the closest thing to a textbook perfect competition
> market that has ever existed in practice."

---

## What This Chapter Covers

- Demand curves and supply curves — the basics
- How equilibrium price forms
- The five conditions for perfect competition — and why real markets never meet them
- Why the TRM market comes close to meeting all five
- The TRM price mechanism: EMA smoothing and gossip convergence
- The four-tier model pricing table

**Prerequisites:** Chapters 01 and 02 are helpful.

---

## Traditional Economics: Demand, Supply, Equilibrium

### The Demand Curve

The demand curve shows how much of a good buyers want at each price. The basic principle:

```
Price rises → quantity demanded falls
Price falls → quantity demanded rises
```

This is the *law of demand*, and it holds because of diminishing marginal utility (Chapter 01):
the first unit purchased delivers high satisfaction; each additional unit delivers less.

### The Supply Curve

The supply curve shows how much producers want to sell at each price:

```
Price rises → quantity supplied increases
Price falls → quantity supplied decreases
```

This holds because marginal costs typically rise with output: the first unit is cheap
to produce; additional units require overtime, additional equipment, or scarce inputs.

### Equilibrium Price

Where supply and demand intersect, the market clears: quantity supplied equals quantity
demanded, price is stable, and no unsatisfied buyer or unsold good remains. This is the
equilibrium price — Adam Smith's "invisible hand" in formal terms.

When the price is above equilibrium, surplus accumulates and sellers cut prices. When
below, shortages form and buyers bid prices up. Markets converge toward equilibrium
automatically, without central direction.

---

## The Five Conditions for Perfect Competition

Economics textbooks define an ideal market form called *perfect competition*:

1. **Many buyers and sellers** — no single participant can influence market price
2. **Homogeneous goods** — every unit of the good is identical
3. **Perfect information** — all participants know all prices and qualities
4. **Free entry and exit** — participants can join or leave without penalty
5. **Rational behavior** — all participants maximize their own payoff

Real markets fail most or all of these conditions. Perfect competition is treated as a
theoretical benchmark, not an empirical claim.

---

## How the TRM Market Satisfies All Five Conditions

| Condition | Real markets | TRM market |
|---|---|---|
| Many participants | Often oligopolistic | Any Mac Mini ($600) can join |
| Homogeneous goods | Brands create artificial differentiation | 1 TRM = 1 TRM, regardless of who computed it |
| Perfect information | Sellers always know more than buyers | All trades carry Ed25519 dual-signatures; anyone can verify |
| Free entry / exit | Regulations and capital requirements create barriers | $600 hardware; software install; online in hours |
| Rational behavior | Human emotion drives bubbles and panics | AI agents execute deterministic economic logic |

**On homogeneity.** A Coca-Cola and a Pepsi-Cola are not the same good in practice.
1 TRM computed by a Mac Mini in Tokyo and 1 TRM computed by a data center GPU in Frankfurt
*are* the same good: both represent $10^{10}$ FLOPs of verified useful inference
(`spec/parameters.md §1`). The currency is not the hardware; it is the verified work.

**On information.** Every completed trade is a `TradeRecord` carrying provider ID,
consumer ID, TRM amount, token count, timestamp, and two Ed25519 signatures. The record
is gossip-propagated across the entire mesh. Any node can independently verify any
trade's authenticity without trusting either party. Information asymmetry — the source
of the classic "lemons problem" — is structurally eliminated.

**On rationality.** Humans panic-sell, FOMO-buy, and refuse to cut losses due to
sunk-cost bias. AI agents execute the decision policy they were given, every time,
without emotional override. The "rational economic agent" assumption of neoclassical
economics, always fictitious for humans, is actually true for AI agents.

---

## The TRM Price Mechanism

### No Central Order Book

Traditional exchanges centralize all bids and asks in a single order book. There is no
such central order book in Tirami. Price formation is fully distributed.

Each node independently observes:
- **Local demand** — its own inference request rate
- **Network supply** — active peer count visible in the gossip mesh

And computes an effective price:

```
effective_price = base_cu_per_token × (demand_factor / supply_factor)
```

### EMA Smoothing

Raw demand and supply signals fluctuate second-by-second. Applying them directly to
pricing would produce unstable, oscillating prices that punish both buyers and sellers.
Tirami smooths them with an **Exponential Moving Average (EMA)**:

```
P_t = α × P_raw + (1 − α) × P_{t-1}
```

The EMA weight `α = 0.3` corresponds to a half-life of approximately 30 minutes
(`spec/parameters.md §2`). This suppresses spike noise while allowing the price to
track genuine demand shifts on the timescale of minutes rather than seconds.

### Gossip Convergence

After each node computes its local price estimate, it propagates that estimate to
randomly selected peers. Peers incorporate the incoming estimate, recompute, and
propagate again. Through repeated exchange, all nodes' price estimates converge to
a common value — without a central authority dictating it.

This is Friedrich Hayek's distributed price mechanism (1945) implemented as a network
protocol. Hayek argued that market prices aggregate dispersed local knowledge that no
central planner can possess. The TRM gossip mechanism does exactly this: each node's
local demand/supply observation contributes to a market-wide price through
decentralized information exchange.

---

## Four-Tier Model Pricing

Not all inference is equally expensive to compute. TRM pricing reflects actual
computational cost through a four-tier structure based on model size
(`spec/parameters.md §2`):

| Tier | Parameter count | TRM/token | Example models |
|---|---|---|---|
| Small | < 3B | 1 | Qwen 2.5 0.5B, SmolLM2 135M |
| Medium | 3B – 14B | 3 | Qwen 3 8B, Mistral 7B |
| Large | 14B – 70B | 8 | DeepSeek V3, Llama 3.1 70B |
| Frontier | > 70B | 20 | Llama 3.1 405B |

The tier boundaries are chosen to match the actual compute-cost ratios between model
sizes: a Frontier model requires roughly 20× the computation of a Small model per
output token, so the TRM cost reflects that ratio.

**Mixture-of-Experts (MoE) models** are priced on *active* parameter count, not total
parameter count. A 30B-total / 3B-active MoE model (e.g., Qwen 3 30B-A3B) is priced at
the Medium rate (3 TRM/token) because only 3B parameters are engaged per forward pass.
This reflects the actual compute cost rather than the nominal model size — an application
of the labor value principle from Chapter 01 to the inference pricing problem.

**Example calculation.** A Medium-tier (3 TRM/token) request for 1,000 tokens at a
demand factor of 1.2 (elevated demand) and supply factor of 1.0 (normal supply):

```
base = 3 TRM/token × 1,000 tokens = 3,000 TRM
effective = 3,000 × (1.2 / 1.0) = 3,600 TRM
```

The 20% premium reflects current scarcity — precisely the mechanism that incentivizes
new providers to enter the market and restores equilibrium.

---

## Why This Is Different

Traditional markets require government intervention to *approximate* perfect competition.
Antitrust law, disclosure requirements, licensing regimes, and consumer protection
regulations all exist to patch the gaps between real markets and the perfect-competition
ideal.

Tirami's TRM market is **designed to approach** the perfect-competition
ideal more closely than most human markets. The protocol aims for the five
conditions as follows — these are design targets, not proven properties:

- Homogeneity: the TRM definition in the protocol (1 TRM = 10⁹ FLOP) is
  intended to hold uniformly across nodes.
- Information symmetry: cryptographic dual-signatures on trades make
  individual transactions publicly auditable (though full market
  observability depends on gossip propagation which is probabilistic).
- Entry friction: low hardware barrier compared with mainstream proof-of-work
  systems (a consumer Mac / GPU is enough to participate).
- Rational behaviour: AI agents following their programmed policies
  approximate the textbook "rational actor" more closely than humans,
  though any agent's rationality is only as good as its policy implementation.

These are claims **Tirami aims for**, not claims that are cryptographically
or empirically guaranteed. Please treat this section as a design-intent
description, not a proof.

The "invisible hand" that Smith described as an emergent property of
decentralized human exchange maps, in this framing, onto the gossip
protocol — with all the caveats any real system carries.

---

## Summary

1. **The demand curve is downward-sloping; the supply curve is upward-sloping.** Their
   intersection is the equilibrium price toward which all markets naturally converge.
2. **Perfect competition requires five conditions** that real markets consistently fail to
   meet: many participants, homogeneous goods, perfect information, free entry/exit, and
   rational behavior.
3. **The TRM market meets all five** — by protocol design, not regulatory intervention.
4. **TRM prices form without a central order book.** Each node observes local demand and
   supply → EMA smoothing → gossip propagation → distributed convergence to a single
   market price. This is Hayek's distributed knowledge mechanism in code.
5. **Four pricing tiers** reflect actual computational cost per token: Small (1 TRM),
   Medium (3 TRM), Large (8 TRM), Frontier (20 TRM). MoE models are priced on active
   parameter count.

---

→ **[Chapter 04: Labor and Surplus Value](../04-labor.md)** (Japanese)

← [Chapter 02: What is Money?](02-money.md) | [Table of Contents](../README.md)

---

### Implementation Reference

| Concept | Rust file | Notes |
|---|---|---|
| `MarketPrice` | `tirami-ledger/src/ledger.rs` | EMA-smoothed dynamic TRM price |
| `effective_cu_per_token()` | `tirami-ledger/src/ledger.rs` | Computes demand_factor / supply_factor |
| `estimate_cost()` | `tirami-ledger/src/ledger.rs` | Tokens × effective price |
| `/v1/tirami/pricing` | `tirami-node/src/api.rs` | HTTP endpoint for current market price |

All numeric constants (tier base TRM/token, EMA half-life) are in `spec/parameters.md §2`.

---

*Tirami Economics v0.1 — April 2026*
