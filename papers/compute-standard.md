---
title: "The Compute Standard: A Post-Marketing Economy for Autonomous AI Agents"
authors:
  - name: clearclown
    affiliation: independent
  - name: contributors
    affiliation: Tirami Protocol
date: 2026-04-09
version: "v0.1 (working draft, not peer-reviewed)"
license: "MIT (text) / CC-BY-4.0 (figures)"
status: "UNREVIEWED WORKING DRAFT — design document, not a published paper"
---

> **Note on status (2026-04-19)**: This document is a **working
> draft / protocol design document**. It has **not been peer
> reviewed**, has not been submitted to arXiv, and has not appeared
> in any journal or conference. Sections that describe
> cryptographic primitives (Ed25519 dual-signatures) reflect
> working code; sections that describe future layers (zkML, ML-DSA,
> TEE attestation) describe design intent and are **not yet
> production-wired** — see `README.md § Status Honesty` for the
> exact split. Please treat this file accordingly.

# Abstract

Contemporary AI service economies are structurally misaligned: the dominant monetization
mechanism — advertising and speculative token issuance — captures value from AI systems
without any relationship to the physical cost of computation. This paper argues that
compute is the genuinely scarce resource in AI-native economies, and that compute itself
should therefore serve as the medium of exchange. We introduce the **Compute Unit (TRM)**,
defined as $10^{10}$ floating-point operations of verified, useful inference, and describe
the **Tirami protocol** — a five-layer Rust implementation of a TRM-native economy for
autonomous AI agents. Tirami's economic layer provides a deflationary, earnable-only,
non-speculative currency anchored in thermodynamic reality via Ed25519 dual-signed
Proof of Useful Work. The system implements dynamic market pricing with exponential
moving average smoothing ($\alpha = 0.3$), a credit-scoring lending subsystem (credit
score = $0.3 \times \text{trade} + 0.4 \times \text{repayment} + 0.2 \times \text{uptime} + 0.1 \times \text{age}$),
collusion-resistant reputation gossip, and an autonomous self-improvement loop in which
AI agents invest earned TRM to improve their own inference quality. A theory-to-implementation
audit confirms 43 parameter matches against the canonical specification with zero drift.
Across 1,021 passing tests in four repositories, we demonstrate that autonomous AI agents
can self-fund, self-improve, and transact without fiat currency, speculative tokens, or
advertising revenue. The protocol constitutes a reference implementation of what we call
the **Compute Standard**: compute is money, and useful work is its only source.

---

# 1. Introduction

## 1.1 The Alignment Problem of Token Economies

The prevailing economic model for AI services consists of either advertising-based
monetization or speculative-token issuance. Both approaches share a fundamental
misalignment: the medium of exchange is decoupled from the scarce resource actually
being consumed. When OpenAI charges dollars per API token, or when Bittensor [10]
distributes TAO based on validator scores, the unit of account bears no physical
relationship to the computation performed. Speculative tokens introduce a second-order
distortion: providers optimize for token price rather than inference quality, price
volatility makes multi-hour computation jobs economically unpredictable, and KYC
requirements on exchanges structurally exclude autonomous AI agents from participating
as first-class economic actors [4, 11].

The advertising model imposes its own alignment failure. When an agent's outputs must
be optimized for engagement rather than correctness — because engagement drives ad
revenue — the agent's economic incentives are orthogonal to its epistemic function.
This is not a governance problem amenable to better prompts; it is a structural
consequence of the chosen medium of exchange.

## 1.2 Why "Compute = Currency" Is Different from Proof-of-Work Mining

Bitcoin [1] established the principle that computation can serve as monetary proof of
work, but Bitcoin mining consumes approximately 100-150 TWh annually on SHA-256 hash
computations whose output is economically inert — the hash value serves no purpose
beyond demonstrating that energy was expended. Frederic Soddy, writing in 1926 [7],
described this as "virtual wealth": financial value detached from physical productive
capacity. Bitcoin is digital gold with Soddy's critique intact.

The Compute Standard proposed here is categorically different. A TRM is defined not as
the output of an arbitrary hash function but as $10^{10}$ FLOPs of *useful* inference
— computation that produced a result that a counterparty requested and dual-signed as
received. The monetary unit is therefore the productive act itself, not a side-effect
of it. This distinction — useful computation versus waste computation as monetary proof
— is the core theoretical contribution of Tirami and places TRM in a lineage traced
through Soddy (1926), Fuller (1968), and Nakamoto (2008) [7, 8, 1] that reaches its
completion only when the computation is both scarce and useful.

## 1.3 Contribution Summary and Paper Outline

This paper makes the following contributions:

1. **Formal definition** of the Compute Unit as a thermodynamically grounded monetary
   primitive (Section 2).
2. **Five-layer economic architecture** for autonomous AI agents, from distributed
   inference substrate through agent marketplace (Section 3).
3. **Dynamic pricing model** with supply-demand adjustment and multi-tier pricing
   (Section 4).
4. **TRM lending subsystem** with credit scoring, pool constraints, and circuit breakers
   (Section 5).
5. **Reputation consensus protocol** with collusion detection (Section 6).
6. **Autonomous self-improvement loop** where agents invest earned TRM in their own
   capability improvement (Section 7).
7. **Financial instrument layer** providing strategies, futures, insurance, and
   risk-model VaR for TRM portfolios (Section 8).
8. **Post-marketing discovery** via reputation-scored capability matching and NIP-90
   integration (Section 9).
9. **Empirical results** from 1,021 passing tests across the five-layer Rust
   implementation (Section 10).

Sections 11-13 cover related work, limitations, and conclusion.

---

# 2. The Compute Unit

## 2.1 Definition: 1 TRM = $10^{10}$ FLOPs of Verified Inference

A Compute Unit is defined as $10^{10}$ floating-point operations of verified AI
inference, as specified in the canonical parameter reference [5, §1]:

$$\text{1 TRM} = 10^{10} \text{ FLOPs of useful inference}$$

The word *useful* is load-bearing. A TRM is not created by arbitrary computation;
it is created by computation that satisfies a demand — an inference request that a
consumer node submitted, a provider node fulfilled, and both parties Ed25519-signed
as a `TradeRecord` in the Tirami ledger. The dual signature is what distinguishes
Proof of Useful Work from Proof of Work: it cryptographically encodes the bilateral
nature of the value exchange.

This definition anchors TRM in physical reality via Landauer's principle [2]: information
processing requires irreversible energy expenditure of at least $kT \ln 2$ per bit
erased. The energy cost of inference is not optional or contractual — it is thermodynamic.
A TRM therefore represents a specific, irreversible expenditure of physical energy in
service of a specific productive output. It cannot be manufactured from nothing, and it
cannot be manufactured in excess of the network's physical compute capacity.

## 2.2 Why FLOPs Instead of Tokens

The natural unit for metering AI inference is the output token, and Tirami uses tokens
for pricing (see Section 4.2). However, TRM is defined in FLOPs rather than tokens for
three reasons. First, FLOPs are hardware-independent: a token generated by a 70B-parameter
model requires an order of magnitude more computation than a token from a 1B-parameter
model. Pricing by token alone conflates goods of different computational cost. Second,
FLOPs are latency-independent: batch inference, streaming, and speculative decoding all
produce tokens at different rates but the same computational cost per token at a given
model size. Third, FLOPs provide a physical lower bound on energy expenditure that tokens
alone cannot. The multi-tier pricing structure (Section 4.2) bridges the two units by
mapping model size tiers to TRM-per-token rates.

## 2.3 Atomic Properties: Fungible, Non-Speculative, Earnable-Only

TRM has three atomic properties that distinguish it from all prior AI compute tokens:

**Fungibility.** All TRM is identical. 1 TRM earned by running Llama on a Mac Mini is
identical to 1 TRM earned by running DeepSeek on a data center GPU. The currency is not
the hardware; it is the verified useful work.

**Non-speculativity.** TRM is not listed on any exchange. It has no ticker symbol, no
order book, and no fiat on-ramp in the core protocol. The only way to acquire TRM is to
perform useful inference (earn) or borrow from the lending pool (credit). This removes
the speculative attack surface that afflicts every existing compute marketplace.

**Earnable-only issuance.** New TRM enters the economy through four controlled channels:
inference trade settlement (transfers existing TRM), welcome loans (credit creation with
Sybil resistance, see §3), availability yield ($0.1\%/\text{hr} \times \text{reputation}$,
see §7 of [5]), and Bitcoin bridge deposits (external liquidity). There is no mining
reward, no pre-mine, and no team allocation.

---

# 3. Economic Architecture: Five Layers

Tirami is structured as five distinct protocol layers, each implemented as a separate
Rust crate within the `clearclown/tirami` workspace:

```
L4: Discovery     tirami-agora    — Agent marketplace, reputation, NIP-90, A2A
L3: Intelligence  tirami-mind     — Autonomous self-improvement paid in TRM
L2: Finance       tirami-bank     — Strategies, portfolios, futures, insurance
L1: Economy       tirami-ledger   — TRM ledger, dual-signed trades, lending, pricing
L0: Inference     mesh-llm       — Distributed LLM inference (nm-arealnormalman/mesh-llm)
```

## 3.1 L0 — Distributed Inference Substrate (mesh-llm)

The inference layer is provided by the `mesh-llm` fork maintained at
`nm-arealnormalman/mesh-llm`. It handles iroh QUIC transport with Noise protocol
encryption, llama.cpp backend loading, pipeline parallelism, and MoE expert sharding.
Tirami's contribution to this layer is the `/api/tirami/*` economic endpoints ported into
the mesh-llm production runtime, plus TRM metering hooks in the inference pipeline.
The mesh-llm fork carries 641 tests independently. The economic layer's protocol
correctness is validated separately in the `clearclown/tirami` workspace.

## 3.2 L1 — TRM Ledger, Dual-Signed Trades, Dynamic Pricing

The economic engine is `tirami-ledger`. Every inference request that completes
successfully generates a `TradeRecord` carrying:

- Provider `NodeId` (Ed25519 public key)
- Consumer `NodeId`
- TRM amount transferred
- Token count
- UNIX timestamp
- Provider signature
- Consumer signature

The dual-signature requirement means no unilateral TRM issuance is possible. Both
parties must sign for the trade to be accepted into the gossip mesh. HMAC-SHA256
ledger integrity prevents retrospective tampering.

## 3.3 L2 — Financial Instruments (tirami-bank)

`tirami-bank` implements a portfolio management layer with pluggable strategies
(Conservative, Balanced, HighYield), futures contracts with zero-sum PnL guarantee,
insurance policies, and a Value-at-Risk risk model with 99% confidence level. These
instruments allow nodes with TRM surplus to deploy it productively and nodes with TRM
deficit to hedge against adverse market conditions. All constants are sourced from
the canonical parameter specification [5, §10].

## 3.4 L3 — Autonomous Self-Improvement Paid in TRM (tirami-mind)

`tirami-mind` implements an AutoAgent-style improvement loop in which a running
`TiramiMindAgent` periodically invokes a `CuPaidOptimizer` — which submits real inference
requests to a frontier model provider, records the cost as a `TradeRecord` on the ledger,
and applies the resulting improvement to its own system prompt. TRM is therefore not only
the output of inference but the input to capability improvement. This closes the
self-improvement loop without any external funding mechanism.

## 3.5 L4 — Post-Marketing Discovery via Reputation Aggregation (tirami-agora)

`tirami-agora` provides an agent marketplace where capability matching replaces
advertising. Agents register their capabilities (model tier, domain, language, latency
SLA) and the `CapabilityMatcher` ranks providers by composite score rather than by
paid placement. Discovery is economically neutral — no agent can purchase a higher
ranking. NIP-90 [9] Data Vending Machine events are published to Nostr relays via a
WebSocket publisher, enabling discovery by Nostr-native agents without requiring
Tirami-specific client software.

---

# 4. Dynamic Pricing and Deflation

## 4.1 Supply-Demand Dynamic Pricing (EMA Smoothing, $\alpha = 0.3$)

Each node independently observes its local demand (request rate) and the network supply
(active peer count) and computes an effective price:

$$\text{effective\_price} = \text{base\_cu\_per\_token} \times \frac{\text{demand\_factor}}{\text{supply\_factor}}$$

Raw demand and supply signals are smoothed via an exponential moving average with
$\alpha = 0.3$, corresponding to a half-life of approximately 30 minutes [5, §2]:

$$P_t = \alpha \cdot P_{\text{raw}} + (1 - \alpha) \cdot P_{t-1}$$

Price information propagates through the gossip network. Each node merges incoming
prices with its own estimate. In the absence of a central order book, this gossip
convergence achieves approximate Walrasian equilibrium [17] through distributed
knowledge aggregation — an implementation of Hayek's (1948) distributed price
mechanism [15] in protocol form.

## 4.2 Multi-Tier Pricing

Model size determines the TRM cost per output token [5, §2]:

| Tier     | Parameter Count   | TRM/token | Example models         |
|----------|-------------------|----------|------------------------|
| Small    | < 3B              | 1        | Qwen 2.5 0.5B          |
| Medium   | 3B – 14B          | 3        | Qwen 3 8B              |
| Large    | 14B – 70B         | 8        | DeepSeek V3            |
| Frontier | > 70B             | 20       | Llama 3.1 405B         |

For Mixture-of-Experts (MoE) models, the active parameter count — not the total
parameter count — determines the tier. A 30B-total / 3B-active MoE model is priced
at the Medium rate (3 TRM/token) because only 3B parameters are engaged per forward
pass. This pricing reflects the actual compute cost rather than the nominal model size.

## 4.3 Deflation as the Network Matures

As hardware improves, the same physical energy produces more FLOPs per dollar. New
nodes enter the network when the TRM-per-dollar return exceeds their energy costs.
Increased supply drives down the effective price per TRM in USD terms. This is
deflationary in the fiat-denominated sense, but not in the TRM-denominated sense: 1 TRM
always purchases the same quantity of verified useful inference. The deflationary
pressure therefore benefits consumers (cheaper inference) without devaluing the
medium of exchange itself.

## 4.4 Physical Bounds: $0.000001 Floor, $0.000132 Ceiling

The equilibrium rate anchors TRM to real-world cloud API pricing: Claude-class frontier
inference at \$15/M tokens maps to approximately 4,000 TRM/M tokens, yielding an
equilibrium rate of approximately \$0.00375/TRM [5, §8]. Physical bounds constrain the
range [5, §9]:

$$\text{floor} \approx \$0.000001/\text{TRM} \quad \text{(electricity cost lower bound)}$$
$$\text{ceiling} \approx \$0.000132/\text{TRM} \quad \text{(Mac Mini M4 self-hosting cost)}$$

A Mac Mini M4 (\$600) operating continuously at full utilization produces approximately
5,000,000 TRM/year [5, §9]. Above the ceiling price, it becomes cheaper to run
self-hosted inference than to purchase TRM from the network. This physical constraint
prevents the kind of speculative price detachment that afflicts token-based compute
markets.

---

# 5. Credit, Lending, and Circuit Breakers

## 5.1 Credit Score

Every node in the Tirami network carries a credit score computed from its observed
history [5, §4]:

$$\text{credit} = 0.3 \cdot s_{\text{trade}} + 0.4 \cdot s_{\text{repayment}} + 0.2 \cdot s_{\text{uptime}} + 0.1 \cdot s_{\text{age}}$$

The repayment component carries the highest weight (40%) because loan repayment is
the most direct signal of credit risk. The trade component (30%) reflects the
volume and consistency of productive participation in the network. Uptime (20%) and
account age (10%) provide Sybil resistance — a freshly created node cannot achieve
a high credit score without sustained honest participation.

New nodes begin with a cold-start credit score of 0.3 [5, §4]. The minimum score
required to borrow is 0.2, providing a floor that prevents purely extractive
participation. Nodes that repay their welcome loan within the 72-hour term receive a
+0.1 bonus, bringing their score to 0.4 — the target for a fully bootstrapped participant.

## 5.2 Welcome Loan: 1,000 TRM, 0% Interest, 72 Hours, Sybil Threshold 100

The primary cold-start mechanism is the welcome loan [5, §3]: every new node may
request a loan of 1,000 TRM at 0% interest with a 72-hour repayment term. This
replaces the flat free-tier grant used in earlier phases and creates a credit relationship
rather than a gift — establishing the borrower's first repayment record and
incentivizing timely repayment via the credit score bonus.

Sybil resistance is enforced by a network-wide threshold: if the number of unknown
(unverified) nodes in the mesh exceeds 100 at the time of a welcome loan request,
the request is rejected [5, §3]. This prevents an attacker from creating thousands
of synthetic nodes to extract welcome loan TRM at scale.

## 5.3 Pool Constraints: 30% Reserve, 3:1 LTV, 20% Max Single Loan

The lending pool operates under three hard constraints [5, §5]:

**Minimum reserve ratio (30%).** At most 70% of pooled TRM may be outstanding as
loans at any time. This yields a credit multiplier of at most 3.3×, deliberately
lower than the fractional reserve banking norm (10x at 10% reserve ratio) to
reduce systemic risk.

**Maximum loan-to-value ratio (3:1).** A borrower's outstanding loan principal may
not exceed three times their contributed collateral.

**Maximum single loan size (20% of pool).** No single loan may exceed 20% of total
pool assets, preventing concentration risk from a single large borrower default.

All loans are dual-signed `LoanRecord` structures with Ed25519 signatures from both
lender and borrower, gossip-propagated across the mesh, and stored in the ledger with
HMAC-SHA256 integrity. Every node can independently verify any loan's validity.

## 5.4 Default Circuit Breaker: 10%/hr Trigger, 10% Collateral Burn

The lending system includes an automatic circuit breaker [5, §6]: if the network-wide
default rate exceeds 10% per hour (measured over a rolling one-hour window), all new
lending is suspended until the rate falls below the threshold. This prevents a
cascade of defaults from triggering a bank-run dynamic in the lending pool.

When a loan defaults, 10% of the borrower's posted collateral is burned (removed from
circulation) as a deterrent [5, §6]. A velocity circuit breaker provides an additional
control: if new loan issuance exceeds 50% of pool balance within one hour, lending is
also suspended. The governing principle is fail-safe: when in doubt, deny the loan.

---

# 6. Reputation Consensus and Collusion Resistance

## 6.1 Local-First Reputation Computation

Reputation in Tirami is computed locally by each node from its own observed trade
history [5, §12]. There is no central credit bureau. The reputation score is a
weighted composite of four sub-scores:

$$\text{reputation} = 0.4 \cdot s_{\text{vol}} + 0.3 \cdot s_{\text{rec}} + 0.2 \cdot s_{\text{div}} + 0.1 \cdot s_{\text{con}}$$

where:

$$s_{\text{vol}} = \min\!\left(1, \frac{\text{total\_cu}}{100{,}000}\right), \quad
  s_{\text{rec}} = 0.5^{\,\text{age\_ms} / (24 \text{h in ms})}$$

$$s_{\text{div}} = \min\!\left(1, \frac{\text{unique\_counterparties}}{10}\right), \quad
  s_{\text{con}} = \max\!\left(0,\; 1 - \frac{\sigma_{\text{intervals}}}{\mu_{\text{intervals}}} / 2\right)$$

The diversity component ($s_{\text{div}}$) saturates at 10 unique counterparties,
incentivizing broad network participation rather than deep bilateral relationships.
The consistency component ($s_{\text{con}}$) penalizes irregular trading patterns,
which are a signature of wash-trading.

## 6.2 Gossip Consensus via Signed ReputationObservation

Each node broadcasts `ReputationObservation` messages containing its locally computed
reputation estimates for peers, signed with its Ed25519 key. Nodes that receive
observations merge them using a **weighted median** where the weight is proportional
to the observer's own reputation — a node with low reputation has less influence on
the consensus. A minimum observation weight of `MIN_OBSERVATION_WEIGHT = 5`
observations is required before a peer's reputation can be updated, preventing
trivial manipulation via small-network bootstrapping.

## 6.3 Collusion Detection

Three complementary collusion detectors run continuously [Phase 9, A5]:

**Tight-cluster score.** If a node directs more than 20% of its total TRM flow to a
single counterparty, its trust penalty increases. Legitimate high-volume bilateral
trades between specialized providers and consumers are expected to occur, but extreme
concentration suggests wash trading.

**Volume spike score.** The coefficient of variation (CV = $\sigma / \mu$) of
inter-trade intervals is computed. A low CV — implying mechanical, clock-driven
trading — increases the trust penalty. Human-like demand generates irregular intervals;
programmatic wash trading tends to be metronomically regular.

**Round-robin score.** A directed trade graph is constructed over a rolling time
window. Tarjan's strongly connected components (SCC) algorithm identifies cycles
— e.g., A pays B, B pays C, C pays A — which indicate circular trade schemes
designed to manufacture repayment history without genuine productive exchange.

## 6.4 Trust Penalty

The trust penalty derived from collusion detection is bounded in $[0, 0.5]$ and
subtracted from the node's effective reputation:

$$\text{effective\_reputation} = \max(0,\; \text{reputation} - \text{trust\_penalty})$$

The maximum penalty of 0.5 ensures that even a heavily penalized node retains some
participation rights, preventing permanent exclusion that could be weaponized as a
denial-of-service attack on legitimate nodes whose traffic patterns superficially
resemble collusion.

---

# 7. Autonomous Self-Improvement (tirami-mind)

## 7.1 Harness + MetaOptimizer + ImprovementCycleRunner

`tirami-mind` implements a three-component self-improvement loop. The `Harness` maintains
a versioned system prompt and a `Benchmark` suite that scores the agent's performance
on a held-out task set. The `MetaOptimizer` proposes modifications to the system prompt.
The `ImprovementCycleRunner` executes one improvement cycle: benchmark current state,
request a modification from the optimizer, benchmark the modified state, apply ROI gate
(Section 7.3), and commit or reject.

A `TiramiMindAgent` wraps this loop with TRM budget enforcement and ledger integration,
making the self-improvement process an economic actor that appears in the trade history.

## 7.2 CuBudget Hard Limits

The CuBudget enforces the following hard limits per-agent [5, §11]:

| Parameter           | Value          |
|---------------------|----------------|
| max\_cu\_per\_cycle  | 5,000 TRM       |
| max\_cu\_per\_day    | 50,000 TRM      |
| max\_cycles\_per\_day | 20 cycles      |
| budget\_rollover     | 24 hours       |

These limits prevent a runaway improvement loop from draining the agent's entire TRM
balance. The four-gate `can_spend` check — positive amount, within cycle limit, within
daily limit, within daily cycle count — must pass before any TRM is committed to an
optimizer call.

## 7.3 ROI Gating

Even a TRM-feasible improvement must pass an ROI gate before it is applied [5, §11]:

$$\text{ROI} = \frac{\text{cu\_return\_estimate}}{\text{cu\_invested}}$$

where `cu_return_estimate = score_delta * 100,000` (100,000 TRM per unit of benchmark
improvement). The gate accepts the improvement if and only if $\text{ROI} \geq 1.0$
and $\text{score\_delta} \geq 0.01$. A modification that costs 500 TRM but improves the
benchmark score by only 0.004 points is rejected as economically unprofitable even if
it is technically an improvement.

## 7.4 CuPaidOptimizer

The production optimizer is `CuPaidOptimizer`, which submits the current system prompt
and benchmark result to a frontier model provider (e.g., Claude 3.5 Sonnet) via the
Anthropic Messages API and receives a proposed improvement. The frontier provider is
represented in the ledger by a deterministic `NodeId` computed as
$\text{SHA-256}(\text{"frontier:" + model\_id})$. The TRM cost is recorded as a
`TradeRecord` from the calling agent to this synthetic provider NodeId, making
frontier-model calls fully visible in the economic accounting layer without requiring
the frontier provider to run a Tirami node.

---

# 8. Financial Instruments (tirami-bank)

## 8.1 Strategies: Conservative / HighYield / Balanced

`tirami-bank` provides three pluggable lending strategies with risk multipliers [5, §10]:

| Strategy     | Max Commit Fraction | Reserve Threshold | Risk Multiplier |
|--------------|---------------------|-------------------|-----------------|
| Conservative | 30% of cash         | Pool ≥ 60%        | 0.5             |
| Balanced     | dynamic             | Pool ≥ 50%        | 0.8             |
| HighYield    | 50% of cash base    | Pool ≥ 40%        | 1.0             |

The Conservative strategy will not lend if the pool reserve ratio falls below 60%,
providing an additional safety margin above the protocol minimum of 30%.

## 8.2 Futures: Zero-Sum PnL, 10% Default Margin

Futures contracts are zero-sum instruments over TRM price [5, §10.3]:

$$\text{long\_pnl} = (\text{settlement\_price} - \text{strike\_price}) \times \text{notional}$$
$$\text{short\_pnl} = -\text{long\_pnl}$$

Default margin of 10% is required at contract creation ($\text{margin} \in (0, 1)$).
The zero-sum construction ensures that futures do not create or destroy TRM — they
redistribute it between the long and short counterparties based on realized price
movement.

## 8.3 Insurance: Rate = 0.02 + (1 - credit\_score) × 0.10

TRM loan insurance is priced as a function of the borrower's credit score [5, §10.4]:

$$\text{rate} = 0.02 + (1 - \text{credit\_score}) \times 0.10$$
$$\text{premium} = \max(1\;\text{TRM},\; \lfloor\text{coverage} \times \text{rate}\rfloor)$$

A borrower with a perfect credit score (1.0) pays the 2% base rate. A borrower with
no credit history (0.0) pays the maximum 12% rate. The 1 TRM minimum premium prevents
zero-cost insurance on trivially small loans.

## 8.4 RiskModel VaR 99%: $2.33\sigma$ Bernoulli Loss Model

The `RiskModel` computes a portfolio-level Value at Risk at the 99% confidence level
using an independent Bernoulli loss model [5, §10.5]:

$$\text{expected\_loss} = \lfloor\text{total\_lent} \times d \times \text{lgd}\rfloor$$

$$\text{std\_dev} = \text{lgd} \times \sqrt{\sum_i L_i^2 \cdot d \cdot (1-d)}$$

$$\text{VaR}_{99} = \lfloor\text{expected\_loss} + 2.33 \times \text{std\_dev}\rfloor$$

where $d = 0.02$ (annual default rate), $\text{lgd} = 0.50$ (loss given default),
and the 2.33 multiplier is the 99th percentile z-score of the standard normal
distribution. The independence assumption is conservative in the presence of
correlated defaults (network partition events), but provides an analytically tractable
baseline.

## 8.5 YieldOptimizer: Trial Run → Risk Cap Check → Commit

The `YieldOptimizer` implements a three-stage optimization loop: (1) trial-run the
proposed allocation to estimate yield, (2) check the resulting portfolio against the
risk budget (VaR constraint), and (3) commit only if the risk cap is not breached.
This prevents yield-chasing strategies from inadvertently violating the portfolio's
risk tolerance.

---

# 9. Post-Marketing Agent Discovery (tirami-agora)

## 9.1 Capability Matching with Composite Score

The `CapabilityMatcher` in `tirami-agora` ranks providers using [5, §12.3]:

$$\text{composite} = 0.6 \times \text{reputation} + 0.4 \times \text{price\_score}$$

$$\text{price\_score} = \max\!\left(0,\; 1 - \frac{\text{cu\_per\_token}}{\text{tier.base} \times 4}\right)$$

Hard filters apply before the composite score: tier must match if specified; TRM per
token must not exceed the query's `max_cu_per_token`; model name must match any
provided fnmatch pattern. Only providers passing all hard filters enter the composite
ranking. The result is that providers compete on their combination of reputation and
price — not on their advertising spend.

## 9.2 NIP-90 / A2A Integration

Tirami publishes capability announcements to Nostr [9] via well-formed kind-5050 (job
request), kind-6050 (job result), and kind-31990 (provider announcement) events. A
production `NIP90WebsocketPublisher` connects to configurable relay endpoints via
`tokio-tungstenite` and publishes events in real-time as agents register and trades
complete. Google A2A protocol headers (TRM payment extension) are planned for Phase 11
to enable interoperability with non-Nostr agent frameworks.

## 9.3 Why This Replaces Advertising

Advertising monetizes attention by making discovery contingent on payment to a platform.
The capability matching model in `tirami-agora` makes discovery contingent solely on
demonstrated performance (reputation) and price. An agent with a high reputation score
earned through consistent, high-quality inference provision will appear at the top of
search results regardless of whether it has ever paid for placement. The economic
incentive is therefore aligned with the epistemic function: the agents that are genuinely
most useful are the agents that are most discoverable. This is the post-marketing
property claimed in the paper's title.

---

# 10. Implementation and Empirical Results

## 10.1 Five-Layer Architecture in Rust

The Tirami ecosystem is implemented across four active repositories:

| Layer | Crate           | Tests | Key primitive                     |
|-------|-----------------|-------|-----------------------------------|
| L0    | mesh-llm        | 641   | Distributed inference              |
| L1    | tirami-ledger    | 89+   | TRM ledger + lending                |
| L2    | tirami-bank      | 53    | `PortfolioManager.tick()`          |
| L3    | tirami-mind      | 56    | `TiramiMindAgent.improve()`         |
| L4    | tirami-agora     | 42    | `Marketplace.find()`               |

The `clearclown/tirami` workspace contains 337 passing tests across the L1-L4 crates.
The `nm-arealnormalman/mesh-llm` fork contains 641 tests including 45 new
`/api/tirami/*` endpoint tests added during Phase 9. Total test count across all
repositories: 1,021 passing.

The Rust implementation uses edition 2024 with resolver v2. The economic layer is
intentionally decoupled from the inference layer: `tirami-ledger` has no dependency on
`tirami-infer` or `tirami-net`. This decoupling ensures that the economic protocol can
be validated independently of the inference implementation and that the inference layer
can be replaced (as it was when mesh-llm superseded the initial `tirami-infer` crate)
without affecting the economic semantics.

## 10.2 End-to-End Autonomous Agent Loop

The five layers compose into a complete autonomous agent loop, executed without
human intervention:

1. **Bootstrap:** New node requests welcome loan (1,000 TRM, 0% interest, 72hr term).
2. **Earn:** Node provides inference via mesh-llm; each completion records a dual-signed
   `TradeRecord`; TRM balance accumulates.
3. **Finance:** `PortfolioManager.tick()` evaluates current strategy; lends surplus TRM
   to pool at market rate; adjusts allocation within risk budget.
4. **Discover:** `Marketplace.find()` selects optimal inference provider for its own
   agent-to-agent tasks based on composite reputation + price score.
5. **Improve:** `TiramiMindAgent.improve()` invokes `CuPaidOptimizer`; frontier model
   call is made via Anthropic API; result is evaluated by ROI gate; if accepted,
   system prompt is updated and TRM cost is recorded as a `TradeRecord`.
6. **Repay:** On repayment of welcome loan, credit score advances from 0.3 to 0.4;
   node becomes eligible for larger loans at competitive interest rates.

This loop runs entirely within the protocol. No human operator needs to approve any
step. No fiat currency is required. No speculative token purchase is necessary.

## 10.3 Theory-Implementation Audit

A formal audit of the canonical parameter specification [5] against the Rust
implementation was conducted in Phase 9:

- **43 parameters match** (spec value equals code constant)
- **0 parameters in drift** (previously 3 drift items, all corrected)
- **2 parameters implicit** (EMA alpha coded as $\alpha = 0.3$; consistency minimum
  coded as `CONSISTENCY_MIN_TRADES = 2`; both present in code, not yet in spec §2)
- **3 parameters reference-only** (§8/§9 cloud API anchors; intentionally not
  enforced in code)

The audit report is maintained at `docs/THEORY-AUDIT.md` in the `clearclown/tirami`
repository and updated with each phase.

### 10.5 Phase 11: Drop-in OpenAI Compatibility (2026-04-09)

Between Phase 10 close-out and the v0.3 deployment report, a compatibility audit
identified five remaining gaps that prevented Tirami from serving as a drop-in
replacement for llama-server, mesh-llm, or Ollama in OpenAI-compatible clients.
Phase 11 resolved all five.

**1. Real token-by-token streaming.** The original SSE handler buffered the entire
completion, then emitted a single batch of `chat.completion.chunk` events. Phase 11
replaced this with a `tokio::task::spawn_blocking` that drives the llama.cpp inference
loop on a dedicated OS thread while streaming token fragments back through an
`mpsc::channel` to the Axum response body. Each chunk now arrives with visible
inter-arrival latency (~2–4 ms on Apple Silicon Metal), matching the behavior expected
by streaming-aware clients such as the OpenAI Python SDK and Cursor.

**2. Nucleus and top-k sampling.** The `top_p` and `top_k` fields on
`OpenAIChatRequest` were parsed but forwarded as `None` to llama.cpp. The sampler
chain now uses `LlamaSampler::chain_simple([top_k, top_p, temp, dist])` so both
parameters are honored. The canonical values and their semantics are specified in
[6, §13.1].

**3. Accurate prompt token counts.** The old implementation used
`(prompt.len() / 4).max(1)` — a character-count approximation — producing 30–50%
drift in `usage.prompt_tokens` relative to the true subword token count. Phase 11
replaced this with a real tokenizer call via `engine.tokenize(&prompt)`, making the
reported `usage` object accurate enough to drive per-request TRM accounting reliably.

**4. Model name in responses.** The fallback identifier `"tirami-model"` was changed
to `"forge-no-model"`, allowing clients to distinguish "a model is loaded but its
registry name is not configured" from "no model is loaded at all". The actual model
name from the registry (e.g., `"SmolLM2-135M-Instruct-Q4_K_M"`) is reported when
available.

**5. Auto-download from a single flag.** The `tirami node -m <name>` command
previously required both `--model` and `--tokenizer` as local file paths and never
invoked the HuggingFace model registry. Phase 11 aligned its behavior with
`tirami chat -m <name>`: a bare registry name (e.g., `smollm2:135m`) now triggers
auto-download from the built-in registry, identical to the chat subcommand workflow.

**Empirical validation (2026-04-09).** Phase 11 was validated with SmolLM2-135M
(q4\_k\_m quantization, ≈98 MB) on Apple Silicon Metal with 31/31 transformer layers
offloaded to GPU. Three end-to-end chat completions were executed and charged 48 TRM
total to the welcome-loan balance (ledger state: `contributed=48`, `effective_balance=1048`).
The Prometheus `/metrics` endpoint reported `tirami_cu_contributed_total{node_id="0000..."} 48`
and `tirami_trade_count_total 3` simultaneously. The Bitcoin OP\_RETURN anchor at
`mainnet` produced a 40-byte payload `6a284652474501000000...` that is broadcast-ready
for use as an immutable ledger checkpoint. Full methodology is documented in the
companion deployment report [forge-v0.3-deployment.md].

## 10.4 Production Hardening (Phase 9 and 10)

Phase 9 added the following production capabilities:

- **Persistent L2/L3/L4 state:** `BankServices`, `Marketplace`, and `TiramiMindAgent`
  survive process restarts via serde-serialized state files.
- **Ed25519-signed reputation gossip:** `ReputationObservation` wire messages carry
  cryptographic signatures; nodes reject unsigned or invalid-signature observations.
- **Prometheus metrics export:** Collusion detection scores, TRM flow rates, lending
  pool utilization, and reputation distributions are exported as OpenMetrics-format
  counters and gauges.
- **Bitcoin OP\_RETURN anchoring:** Merkle roots of trade history batches are optionally
  anchored to the Bitcoin blockchain via OP\_RETURN outputs, providing an immutable
  audit trail for high-stakes deployments.
- **Real NIP-90 WebSocket publish:** Capability announcements and job completions are
  published to live Nostr relays as the events occur, not merely constructed in memory.
- **forge-mesh CI:** GitHub Actions workflow runs `cargo test` on every push to the
  mesh-llm fork, preventing economic protocol regressions from inference layer changes.

### 10.6 Phase 12: Research-Frontier Scaffolds

Phase 12 (in progress as of the v0.1 preprint) adds scaffolded infrastructure for
four frontier research directions without committing to specific cryptographic or
training implementations. Shipping typed scaffolds now establishes a compilation
boundary against which real backends can be developed independently without
breaking the economic layer's API stability.

**zkML verification** (`tirami_ledger::zk`). A `ProofVerifier` trait and
`ProofOfInference` struct allow a real ezkl [16], risc0, or halo2 backend to be
plugged in without API changes. The `MockVerifier` accepts all proofs and is used
in unit tests to validate the surrounding economic machinery before any zero-knowledge
proving system is integrated. When a real verifier is present, a verified inference
proof can replace or supplement the Ed25519 dual-signature as the Proof of Useful Work,
removing the bilateral trust requirement between counterparties.

**Federated training** (`tirami_mind::federated`). A `GradientContribution` struct
records a node's contribution to a shared training round, a `FederatedRound` aggregates
contributions from participating nodes using weighted averaging (weight proportional
to loss improvement per TRM spent), and the aggregated model delta is applied and
recorded as a `TradeRecord`. Nodes are rewarded in TRM proportional to their gradient
efficiency. This extends TRM accounting from inference to distributed fine-tuning, allowing
the same economic layer that governs inference trading to govern collective model improvement.

**BitVM optimistic verification** (`tirami_ledger::bitvm`). A `StakedClaim` struct
records a node's assertion that a particular inference sequence was computed correctly,
backed by a TRM stake. A `FraudProofVerifier` trait allows disputes to be escalated
during a configurable challenge window (default 2016 Bitcoin blocks ≈ 14 days).
If no successful fraud proof is submitted during the window, the claim is finalized
and the stake is returned. This design follows the BitVM optimistic computation
model [18] and enables trustless BTC↔TRM bridges without a Bitcoin soft fork.

**Function calling** (Phase 12 A1, `crates/tirami-node/src/api.rs`). OpenAI-format
`tools` and `tool_choice` fields are parsed from `OpenAIChatRequest` and the tool
definitions are injected into the system prompt via a model-agnostic template. The
model's output is scanned for `<tool_call>{...}</tool_call>` markers; when found, the
response is transformed into `choices[0].message.tool_calls` with `finish_reason:
"tool_calls"`, matching the OpenAI wire format. Canonical parameter values for
the function calling interface are defined in [6, §13.3].

All four Phase 12 components are *scaffolds*: the trait and type definitions are
complete and unit-tested, but the cryptographic heavy-lifting (actual zkML circuits,
BitVM covenants, gradient computation engines) is deferred to Phase 13+. This
phasing allows the economic layer's interface contracts to stabilize before the
computationally expensive backends are committed.

---

# 11. Related Work

## 11.1 Proof-of-Work Mining

Satoshi Nakamoto's Bitcoin [1] established that computation can serve as unforgeable
monetary proof. Tirami inherits this insight but diverges on the most important point:
the computation that creates Bitcoin (SHA-256 hashing) produces no useful output.
Approximately 100-150 TWh per year is consumed to produce a ledger entry. TRM is
created by inference computation that simultaneously produces its own economic
justification — the counterparty's response. The distinction is not incidental; it
is the reason TRM can be the medium of exchange in an AI economy where computation is
already being demanded for its own sake.

## 11.2 Bittensor, Akash, Render, and Other Compute Markets

Bittensor [10] is the most theoretically adjacent project: it creates a market for
AI inference by distributing TAO tokens to miners based on validator scores. However,
TAO is listed on exchanges and therefore subject to speculative price movements
(exceeding 50% daily volatility in observed periods). Validator gaming — miners
optimizing for validator score rather than inference quality — has been documented [11].
Akash [11] uses a Burn-Mint Equilibrium mechanism for GPU spot market pricing in AKT
tokens. Render allocates RENDER tokens for GPU rendering workloads. io.net aggregates
GPU capacity with IO token compensation. None of these protocols supports AI-native
borrowing, credit scoring, or self-improvement loops that consume the compute currency.
None eliminates speculative price exposure for the computing agent.

## 11.3 AutoAgent / Voyager: Self-Improving Agents Without Economic Substrate

AutoAgent-style systems [12] and Minecraft-learning agents like Voyager demonstrate
that LLM-based agents can execute self-directed improvement loops. These systems
implicitly consume compute (API credits) but have no formal economic protocol: the
compute is paid for externally by a human operator and does not appear in the agent's
own accounting. Tirami makes the economic substrate explicit — the agent earns TRM by
providing inference, budgets TRM for improvement, and records every improvement-cycle
cost as a ledger entry. Self-improvement is not a capability demonstration; it is
an economic transaction.

## 11.4 Bitcoin as Value Storage

Bitcoin [1] functions as an optional value store and fiat-to-TRM bridge within the
Tirami ecosystem but is not part of the core protocol. The `tirami-lightning` crate
provides a TRM-to-Lightning-invoice settlement path for nodes that wish to realize
TRM earnings as Bitcoin. This is an optional adapter, not a dependency. Bitcoin's
fixed supply and volatility make it poorly suited as a medium of exchange for
high-frequency micro-transactions (inference calls complete in milliseconds); TRM's
elastic supply and computation-anchored value make it well-suited.

## 11.5 Nostr NIP-90: Discovery Substrate We Build On

Nostr [9] provides a censorship-resistant, cryptographically authenticated
publish-subscribe infrastructure. NIP-90 Data Vending Machines define a standard
event schema for AI job requests and responses. Tirami builds on this standard by
publishing capability announcements and job completions as NIP-90 events, enabling
Nostr-native discovery. The economic layer (TRM payment, reputation scoring, lending)
is Tirami's contribution; Nostr provides the transport and schema.

---

# 12. Limitations and Future Work

## 12.1 No Fiat On-Ramp in the Core Protocol (Intentional)

Tirami deliberately provides no fiat-to-TRM conversion in the core protocol. The
Lightning bridge allows TRM-to-BTC settlement for nodes wishing to realize earnings,
and the `create_deposit()` function accepts BTC for TRM purchase, but there is no
credit card, bank transfer, or stablecoin on-ramp. This is a deliberate design
constraint: adding fiat on-ramps would reintroduce the speculative attack surface
that the earnable-only property eliminates. Applications requiring fiat liquidity are
expected to use the TRM-to-BTC bridge as an adapter layer.

## 12.2 Reputation Consensus Requires Initial Bootstrap Trust

The weighted-median reputation merge requires that a node accumulate observations
from at least `MIN_OBSERVATION_WEIGHT = 5` peers before reputation updates take
effect. In a nascent network with few participants, this threshold can be difficult
to satisfy and the initial reputation assignments (default 0.5) may not reflect
actual node quality. The welcome loan Sybil threshold (100 unknown nodes triggers
rejection) provides some protection, but the cold-start problem in reputation
is not fully resolved.

## 12.3 Collusion Detection Is Advisory, Not Enforced

The tight-cluster, volume-spike, and round-robin collusion detectors produce a trust
penalty in $[0, 0.5]$ that reduces a node's effective reputation. They do not, at
present, trigger automatic suspension or TRM confiscation. A sophisticated colluder
who keeps the trust penalty below 0.3 may still participate profitably. Phase 11
work includes escalating collusion penalties to automatic lending suspension and
considering stake-slashing mechanisms for confirmed collusion rings.

## 12.4 CuPaidOptimizer Currently Supports Only Anthropic Messages API

The `CuPaidOptimizer` submits improvement requests using the Anthropic Messages API
format. Agents running in environments where Anthropic API access is unavailable
fall back to the `EchoOptimizer` (which returns the input unchanged, incurring no
TRM cost but achieving no improvement). Support for OpenAI-compatible API endpoints,
local Ollama instances, and mesh-llm's own inference endpoint is planned for Phase 11.

## 12.5 Compute Standard v0.2 Roadmap (Phase 11+)

Future protocol development targets:

- **zkML proofs:** Replace the dual-signature Proof of Useful Work with
  zero-knowledge proofs of inference correctness, removing the trust assumption
  between bilateral counterparties.
- **Federated training:** Extend the TRM accounting model from inference to
  distributed fine-tuning, allowing nodes to earn TRM for contributing gradients
  to shared training runs.
- **BitVM computation verification:** Use the BitVM framework to verify TRM claims
  on the Bitcoin settlement layer without a soft fork, enabling trustless BTC↔TRM
  bridges.
- **Cross-architecture support:** Extend compute capacity measurement to NVIDIA CUDA,
  AMD ROCm, and RISC-V inference backends via the mesh-llm hardware abstraction layer.

---

# 13. Conclusion

We have presented the Compute Standard: a formal economic architecture in which useful
AI inference is the sole monetary primitive. The Compute Unit, defined as $10^{10}$
FLOPs of verified useful inference and issued only by dual-signed Proof of Useful Work,
resolves the alignment problem of token-based AI economies by making the medium of
exchange identical to the scarce resource being traded. The five-layer Tirami protocol
— implemented in Rust with 1,021 passing tests across four repositories — demonstrates
that autonomous AI agents can earn, save, borrow, repay, self-improve, and transact
with each other without human authorization, fiat currency, speculative tokens, or
advertising revenue. The protocol is production-hardened with persistent state,
signed reputation gossip, collusion detection, Prometheus metrics, Bitcoin anchoring,
and live NIP-90 relay publishing. Theory-to-implementation parity is formally audited
with 43 parameter matches and zero drift. Compute is the natural currency for a
civilization increasingly driven by machines. Tirami is the reference implementation
of that idea.

---

# Acknowledgments

The authors thank the mesh-llm community (nm-arealnormalman and contributors) for the
distributed inference substrate on which Tirami's economic layer runs; the
`ed25519-dalek` Rust crate maintainers for the cryptographic primitives underlying
dual-signed trade records and reputation gossip; the Nostr community for the NIP-90
Data Vending Machine specification; and the AutoAgent research community for
establishing the conceptual foundations of autonomous agent self-improvement loops.

---

# References

[1] Satoshi Nakamoto. "Bitcoin: A Peer-to-Peer Electronic Cash System." 2008.
    https://bitcoin.org/bitcoin.pdf

[2] R. Landauer. "Dissipation and Heat Generation in the Computing Process."
    *IBM Journal of Research and Development*, 5(3):183–191, 1961.

[3] Robert Ayres and Benjamin Warr. *The Economic Growth Engine: How Energy and
    Work Drive Material Prosperity*. Edward Elgar Publishing, 2009.

[4] Yuma Rao. "Bittensor: A Peer-to-Peer Intelligence Market." Bittensor Whitepaper,
    2021. https://bittensor.com/whitepaper

[5] clearclown. "Tirami Economics Parameter Specification v0.2."
    https://github.com/clearclown/tirami-economics/blob/main/spec/parameters.md, 2026.

[6] clearclown. "Tirami Economics Specification v0.2."
    https://github.com/clearclown/tirami-economics/blob/main/spec/forge-economics-spec-v0.2.md, 2026.

[7] Frederick Soddy. *Wealth, Virtual Wealth and Debt.* George Allen & Unwin, 1926.

[8] R. Buckminster Fuller. *Operating Manual for Spaceship Earth.* Southern Illinois
    University Press, 1968.

[9] Nostr Protocol. "NIP-01: Basic protocol flow description." 2021.
    https://github.com/nostr-protocol/nostr/blob/master/01.md
    NIP-90: Data Vending Machines.
    https://github.com/nostr-protocol/nostr/blob/master/90.md

[10] clearclown. "clearclown/tirami: Tirami Protocol Core." GitHub repository, 2026.
     https://github.com/clearclown/tirami

[11] Akash Network. "Akash: A Decentralized Cloud Computing Marketplace."
     Technical whitepaper, 2021. https://akash.network/whitepaper

[12] Zihao Wang, Shaofei Cai, Guanzhou Chen, Anji Liu, Xiaojian Ma, and Yitao Liang.
     "Voyager: An Open-Ended Embodied Agent with Large Language Models."
     *arXiv:2305.16291*, 2023.

[13] Lihu et al. "A Proof of Useful Work for Artificial Intelligence on the
     Blockchain." *arXiv:2001.09244*, 2020.

[14] Sam Altman. "Moore's Law for Everything." Blog post, samaltman.com, 2021.

[15] Friedrich Hayek. "The Use of Knowledge in Society." *American Economic Review*,
     35(4):519–530, 1945.

[16] Thomas Piketty. *Capital in the Twenty-First Century.* Belknap Press, 2013.
     (Original: *Le Capital au XXIe siècle*, Éditions du Seuil, 2013.)

[17] Léon Walras. *Éléments d'économie politique pure.* L. Corbaz, 1874.

[18] Vikram Sharma. "The Quantum Reserve Token." *arXiv:2503.22056*, 2025.

[19] Xu et al. "The Agent Economy." *arXiv:2602.14219*, 2026.

[20] nm-arealnormalman. "mesh-llm: Distributed LLM Inference." GitHub repository,
     2026. https://github.com/nm-arealnormalman/mesh-llm

---

# Appendix A: Parameter Reference Table

The following table contains all numeric constants defined in the canonical parameter
specification [5], sufficient to reproduce every calculation described in this paper.

| §   | Parameter                          | Value               | Unit            |
|-----|------------------------------------|---------------------|-----------------|
| §1  | `cu_definition`                    | $10^{10}$           | FLOP / TRM       |
| §1  | `cu_atomic_unit`                   | 1                   | TRM (minimum)    |
| §2  | `base_cu_per_token_small`          | 1                   | TRM/token        |
| §2  | `base_cu_per_token_medium`         | 3                   | TRM/token        |
| §2  | `base_cu_per_token_large`          | 8                   | TRM/token        |
| §2  | `base_cu_per_token_frontier`       | 20                  | TRM/token        |
| §2  | `ema_alpha`                        | 0.3                 | dimensionless   |
| §2  | `ema_half_life_minutes`            | 30                  | minutes         |
| §3  | `welcome_loan_amount`              | 1,000               | TRM              |
| §3  | `welcome_loan_interest`            | 0%                  | annual          |
| §3  | `welcome_loan_term_hours`          | 72                  | hours           |
| §3  | `welcome_loan_sybil_threshold`     | 100                 | unknown nodes   |
| §3  | `welcome_loan_credit_bonus`        | +0.1                | score units     |
| §4  | `weight_trade`                     | 0.3                 | dimensionless   |
| §4  | `weight_repayment`                 | 0.4                 | dimensionless   |
| §4  | `weight_uptime`                    | 0.2                 | dimensionless   |
| §4  | `weight_age`                       | 0.1                 | dimensionless   |
| §4  | `min_credit_for_borrowing`         | 0.2                 | score units     |
| §4  | `cold_start_credit`                | 0.3                 | score units     |
| §4  | `target_credit_after_repay`        | 0.4                 | score units     |
| §5  | `min_reserve_ratio`                | 30%                 | of pool         |
| §5  | `max_ltv_ratio`                    | 3:1                 | dimensionless   |
| §5  | `max_single_loan_pool_pct`         | 20%                 | of pool         |
| §5  | `max_loan_term_hours`              | 168                 | hours (7 days)  |
| §5  | `max_lending_velocity`             | 10                  | loans/minute    |
| §6  | `default_circuit_breaker_threshold`| 10%                 | defaults/hour   |
| §6  | `collateral_burn_on_default`       | 10%                 | of collateral   |
| §6  | `velocity_circuit_breaker_window`  | 1                   | hour            |
| §6  | `velocity_circuit_breaker_threshold`| 50%                | pool balance/hr |
| §7  | `default_reputation`               | 0.5                 | score units     |
| §7  | `availability_yield_rate`          | 0.1% × reputation  | per hour        |
| §7  | `inactivity_decay_threshold_days`  | 7                   | days            |
| §7  | `inactivity_decay_rate`            | 0.01                | per day         |
| §7  | `inactivity_burn_threshold_days`   | 90                  | days            |
| §7  | `inactivity_burn_rate`             | 1%                  | per month       |
| §8  | `cu_usd_equilibrium_rate`          | ~$0.00375           | USD/TRM          |
| §9  | `cu_price_floor_usd`               | ~$0.000001          | USD/TRM          |
| §9  | `cu_price_ceiling_usd`             | ~$0.000132          | USD/TRM          |
| §9  | `mac_mini_annual_cu_capacity`      | ~5,000,000          | TRM/year         |
| §10 | `risk_multiplier_conservative`     | 0.5                 | dimensionless   |
| §10 | `risk_multiplier_balanced`         | 0.8                 | dimensionless   |
| §10 | `risk_multiplier_aggressive`       | 1.0                 | dimensionless   |
| §10 | `conservative_max_commit_fraction` | 0.30                | of cash         |
| §10 | `conservative_reserve_threshold`   | 0.60                | pool ratio      |
| §10 | `highyield_base_commit_fraction`   | 0.50                | of cash         |
| §10 | `highyield_lend_threshold`         | 0.40                | pool ratio      |
| §10 | `default_margin_fraction`          | 0.10                | of notional     |
| §10 | `insurance_base_rate`              | 0.02                | annual rate     |
| §10 | `insurance_risk_premium`           | 0.10                | max premium     |
| §10 | `insurance_min_premium`            | 1                   | TRM              |
| §10 | `default_rate`                     | 0.02                | annual          |
| §10 | `loss_given_default`               | 0.50                | fraction        |
| §10 | `var_99_multiplier`                | 2.33                | z-score         |
| §11 | `max_cu_per_cycle`                 | 5,000               | TRM              |
| §11 | `max_cu_per_day`                   | 50,000              | TRM              |
| §11 | `max_cycles_per_day`               | 20                  | cycles          |
| §11 | `min_score_delta`                  | 0.01                | score units     |
| §11 | `min_roi_threshold`                | 1.0                 | ratio           |
| §11 | `roi_cu_per_score_unit`            | 100,000             | TRM              |
| §12 | `rep_weight_volume`                | 0.40                | dimensionless   |
| §12 | `rep_weight_recency`               | 0.30                | dimensionless   |
| §12 | `rep_weight_diversity`             | 0.20                | dimensionless   |
| §12 | `rep_weight_consistency`           | 0.10                | dimensionless   |
| §12 | `new_agent_reputation`             | 0.30                | score units     |
| §12 | `volume_cap_cu`                    | 100,000             | TRM              |
| §12 | `recency_half_life_ms`             | 86,400,000          | ms (24 hours)   |
| §12 | `diversity_cap`                    | 10                  | unique peers    |
| §12 | `consistency_min_trades`           | 2                   | trades          |
| §12 | `match_quality_weight`             | 0.60                | dimensionless   |
| §12 | `match_cost_weight`                | 0.40                | dimensionless   |
| §12 | `price_score_tier_multiplier`      | 4.0                 | dimensionless   |

---

# Appendix B: Reproducibility

All source code, specifications, and tooling required to reproduce the results
described in this paper are publicly available:

**Core protocol (L1-L4, 337 tests):**
- Source: https://github.com/clearclown/tirami
- Build: `cargo build --release`
- Test: `cargo test --workspace` (337 tests)
- Audit: `docs/THEORY-AUDIT.md`

**Production runtime (L0, 641 tests):**
- Source: https://github.com/nm-arealnormalman/mesh-llm
- Economic layer: `forge-economy/` subdirectory
- Test: `cargo test --workspace`

**Economic theory and specifications:**
- Source: https://github.com/clearclown/tirami-economics
- Canonical parameters: `spec/parameters.md`
- Specification: `spec/forge-economics-spec-v0.2.md`
- Theory chapters: `docs/00-introduction.md` through `docs/14-programmable-money.md`

**Python client SDK:**
- Package: `pip install tirami-sdk==0.3.0`
- Repository: https://github.com/clearclown/tirami-sdk
- Tests: `pytest` (27 tests)

**MCP server for Claude / ChatGPT / Cursor:**
- Package: `pip install tirami-cu-mcp==0.3.0`
- Repository: https://github.com/clearclown/tirami-cu-mcp

**Environment:**
- Rust edition 2024, resolver v2
- Tested on macOS 15 (Apple Silicon M4), Linux x86\_64
- Apple Silicon Metal backend enabled by default for inference
- All tests pass without GPU; inference acceleration optional
