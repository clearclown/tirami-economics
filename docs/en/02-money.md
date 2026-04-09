> **English translation** — this is a direct translation of `docs/02-money.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# Chapter 02: What is Money?

> "CU has the three classic functions of money plus a fourth:
> *proof of physical backing*."

---

## What This Chapter Covers

- The three functions of money — and what they reveal when applied to CU
- The history of money — and where CU fits in that lineage
- CU's fourth function: physical proof of useful work
- How CU supply is controlled without a central bank
- The intellectual lineage: Soddy → Technocracy → Fuller → Bitcoin → CU

**Prerequisite:** Chapter 01 (Value) is helpful but not required.

---

## The Physical Foundation

Before the functions of money, a deeper question: *why* does anything become money?

The answer lies in physics. The laws of thermodynamics constrain every economy:

**First Law:** Energy is neither created nor destroyed — it changes form. Every act of
production, including AI inference, converts energy from one form to another. Money, at
its root, is a claim on energy-transformation activities.

**Second Law:** Entropy always increases in a closed system — no perpetual motion machine
is possible. This applies to economies too: financial systems that promise perpetual
exponential growth inevitably diverge from the physical economy they represent.

**Landauer's Principle (1961):** Erasing one bit of information requires at minimum
$kT \ln 2$ of energy (~2.9 × 10⁻²¹ J at room temperature). Computation is a physical
process. Inference is a physical process. The energy cost is not optional.

From these three facts a definition follows:

> **Money is a claim on the activity of transforming energy into something useful
> for humans (or AI agents).**

Gold, shell money, fiat notes, Bitcoin, and CU are all special cases of this definition.
The history of money is the history of the relationship between the monetary unit and the
underlying energy-transformation it represents:

| Form | What backs it |
|---|---|
| Commodity money (shells, grain) | Direct usefulness of the commodity |
| Metal coins | Scarcity of the metal; mining energy |
| Convertible notes | Promise of gold redemption |
| Fiat currency | Government credibility (no physical backing) |
| Bitcoin | Energy spent on SHA-256 hashing (output: useless) |
| **CU** | **Energy spent on useful inference (output: demanded)** |

CU's position in this table is its key claim: it is the first monetary form since
commodity money where the backing computation is *directly useful* rather than a
side-effect of energy expenditure.

---

## The Three Classic Functions of Money

Economics defines money by what it *does*, not what it *is*:

1. **Medium of exchange** — eliminates the double-coincidence-of-wants problem of barter
2. **Unit of account** — allows heterogeneous goods to be compared on a single scale
3. **Store of value** — transfers purchasing power across time

CU satisfies all three:

| Function | How CU delivers it |
|---|---|
| Medium of exchange | AI agents buy and sell inference denominated in CU within the Forge network |
| Unit of account | All model tiers and services are CU-priced (Small: 1 CU/token; Frontier: 20 CU/token — see `spec/parameters.md §2`) |
| Store of value | Earned CU can be spent on future inference — with one deliberate constraint |

**The deliberate constraint on store-of-value.** CU held idle for more than 90 days
begins to decay (`spec/parameters.md §7`). This is intentional: the value of CU derives
from the ability to buy useful computation. CU that is never circulated contributes
nothing to the productive economy, and hoarding CU has the same deflationary effect
that Keynes identified in his "liquidity trap." The decay rule structurally prevents that
outcome.

---

## CU's Fourth Function: Physical Proof of Backing

The three classic functions describe what money *does*. CU adds a fourth, which describes
what it *is*:

> **1 CU exists = cryptographic proof exists that $10^{10}$ FLOPs of useful inference
> were actually performed** (`spec/parameters.md §1`).

A ¥10,000 note in your hand gives no verifiable information about what backs it. A BTC
in your wallet proves that SHA-256 hashing was performed — but that hashing produced
nothing anyone needed. 1 CU proves that inference computation was performed *and* that
a counterparty requested it and signed the result as received (Ed25519 dual-signature).

This fourth function is the implementation of Frederick Soddy's 1926 proposal:

```
Soddy (1926):  "Currency should be backed by energy"
Fuller (1968): "kWh should be the monetary unit"
Nakamoto (2008): "Computation can back currency" (but computation was wasteful)
Forge (2026):  "Useful computation backs currency"  ← here
```

---

## The Intellectual Lineage

**Frederick Soddy (1877–1956)**, Nobel Chemistry laureate (1921), spent his later career
criticizing what he called the fundamental flaw in modern finance:

> "Financial systems assume perpetual exponential growth. But the physical economy is
> bounded by energy and entropy. Debt can grow geometrically; real wealth cannot."

He proposed three remedies: (1) energy-backed currency, (2) money that loses value over
time to discourage hoarding, (3) 100% reserve banking to prevent private credit creation.
CU satisfies all three: it is thermodynamically backed, it decays after 90 days of
inactivity, and CU is issued only by performing useful computation — not by a bank's
double-entry bookkeeping.

**The Technocracy Movement (Howard Scott, 1932)** proposed replacing dollars with
"Energy Certificates" (1 certificate = 1 erg). The movement failed for four reasons that
CU resolves: (1) no measurement technology existed in 1932 — digital signatures
automatically track every transaction today; (2) it required authoritarian central
planning — Forge uses a P2P gossip protocol with no central planner; (3) different
energy forms could not be equated — output tokens standardize heterogeneous computation
into a single unit; (4) it was politically impossible for human economies — Forge runs
in AI-native space where political opposition doesn't apply.

**Buckminster Fuller (1895–1983)** defined wealth as "the number of forward days a system
can sustain itself" and proposed kWh as the monetary unit. His concept of
*ephemeralization* — doing more with less as technology improves — maps directly to Forge:
hardware improvements lower the CU production cost, driving down prices and benefiting
consumers without devaluing the currency itself.

**Satoshi Nakamoto (2008)** proved that computation can back a currency without a central
authority. But Bitcoin mining consumes ~100–150 TWh annually on SHA-256 computations
whose output is economically inert. Soddy would call Bitcoin "virtual wealth" — financial
value detached from productive capacity. CU is Bitcoin's proof-of-work principle applied
to *useful* computation.

---

## CU Supply: Monetary Policy Without a Central Bank

Forge has no central bank. CU enters the economy through four channels:

| Channel | Effect on supply | Notes |
|---|---|---|
| **Inference trading** | Neutral (CU transfers) | Existing CU moves between nodes |
| **Welcome loan** (`spec/parameters.md §3`) | Expansionary | 1,000 CU per new node at 0% interest, 72hr term; Sybil threshold: 100 unknown nodes |
| **Availability yield** (`spec/parameters.md §7`) | Expansionary | 0.1%/hr × reputation; rewards sustained uptime |
| **Bitcoin bridge deposit** | Neutral | External liquidity; not new CU creation |

**Automatic stabilization.** The CU economy self-corrects via negative feedback loops:

- *CU surplus*: price per CU falls → node operation becomes unprofitable → nodes exit →
  supply falls → price recovers → equilibrium
- *CU scarcity*: price per CU rises → node operation becomes profitable → nodes enter
  (Mac Mini barrier: $600) → supply rises → price falls → equilibrium

This is Hayek's distributed price mechanism implemented at the protocol level: no central
planner determines the money supply; physical law and market incentives do it
automatically.

**Comparison with Bitcoin.** Bitcoin has a fixed supply ceiling (21 million), which
creates persistent deflationary pressure as demand grows. CU has an *elastic* supply
anchored to the network's physical compute capacity. Bitcoin's computation is wasteful;
CU's computation is the productive output itself. Bitcoin is subject to ASIC
centralization; Forge requires only a $600 consumer device to participate.

---

## Summary

1. **Money is a claim on energy-transformation activities.** CU is the first monetary
   form in which the backing computation is simultaneously the productive output.
2. **CU satisfies all three classic functions of money** — exchange medium, unit of
   account, and store of value — plus a fourth: cryptographic proof of physical backing.
3. **The 90-day decay rule** prevents hoarding and maintains CU velocity, following
   Soddy's proposal for money that loses value over time.
4. **A 100-year intellectual lineage** runs from Soddy (1926) through the Technocracy
   movement, Fuller (1968), and Nakamoto (2008) to CU (2026). Each step corrected a
   limitation of the previous one; CU completes the sequence by making the monetary proof
   also the productive output.
5. **No central bank is needed.** Physical law and market entry/exit dynamics stabilize
   CU supply automatically.

---

→ **[Chapter 03: Supply and Demand](03-supply-demand.md)**

← [Chapter 01: What is Value?](../01-value.md) (Japanese) | [Table of Contents](../README.md)

---

*Forge Economics v0.1 — April 2026*
