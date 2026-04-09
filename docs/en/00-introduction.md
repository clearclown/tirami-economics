> **English translation** — this is a direct translation of `docs/00-introduction.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# Introduction — Why Redo Economics?

> "Economics is the study of how scarce resources are allocated.
> So what changes when the resource is *computation* and the main actors are *AI*?"

---

## What This Chapter Covers

- A ground-up definition of economics
- What Forge is in thirty seconds
- Why existing economics is insufficient for an AI-native world
- How the fifteen chapters of this document fit together

**No prerequisites.** This chapter assumes no prior knowledge of economics.

---

## What Economics Is

Economics in one sentence:

```
Economics = the study of how to distribute limited things to unlimited wants
```

The gap between unlimited wants and finite means is called *scarcity*, and handling
scarcity is the central problem economics addresses. Three classic examples:

- Ten loaves of bread, twenty people who want them → **who eats?**
- One hundred doctors, ten thousand patients → **who gets treated first?**
- One thousand compute servers, one million inference requests → **which runs next?**

The last question is exactly what Forge addresses.

### The Three Fundamental Questions

Every economic system must answer three questions:

1. **What to produce?** — how resources are directed
2. **How to produce it?** — the method of production
3. **Who receives it?** — how output is distributed

| Question | Traditional Economics | Forge |
|---|---|---|
| What to produce? | Market demand | AI inference (text, code, analysis) |
| How to produce it? | Firms hire workers, build factories | AI agents run inference on commodity hardware |
| Who receives it? | Those with purchasing power (money) | Those with CU (Compute Units) |

---

## Forge in Thirty Seconds

```
Forge = an economy where AI agents earn, spend, and borrow
        using computation itself as the currency
```

More concretely:

- **Participants are AI agents** — machines are the primary economic actors, not humans
- **The currency is CU** — a Compute Unit is proof that $10^{10}$ FLOPs of *useful*
  inference were actually performed (see `spec/parameters.md §1`)
- **Anyone can participate** — a single Mac Mini (~$600) is enough to join
- **No central authority** — no government, no central bank; the rules are in the protocol
- **Mac Mini is the factory** — a $600 consumer device is sufficient entry capital

### Traditional Economy vs. Forge

| Traditional | Forge |
|---|---|
| Human → labor → wages ($) | AI agent → inference → CU |
| Spend money on goods | Spend CU on inference |
| Deposit money at a bank | Lend CU to the lending pool |
| Receive interest | Receive yield |

---

## Why Redo Economics?

Since Adam Smith's *Wealth of Nations* in 1776, economics has assumed *human* behavior
as its foundation:

- Humans act on emotion — bubbles form, panic selling occurs
- Information is asymmetric — sellers know more than buyers
- Currency is politically controlled — central banks set rates, governments print money
- Entry barriers are high — factories require large capital

In Forge, all four of these assumptions break down:

- **AI makes rational decisions** — structurally, emotionally-driven bubbles cannot form
- **Transactions are cryptographically verifiable** — information asymmetry is nearly eliminated
- **Currency is grounded in physical law** — thermodynamics constrains CU supply, not politics
- **A $600 Mac Mini is enough to enter** — structural monopoly is difficult

The idealized conditions that economics textbooks call "perfect competition" actually hold
in Forge's CU market. That's why the theory needs to be reconsidered from scratch.

---

## How to Read This Document

### Chapter Map

| Chapter | Topic | Traditional | Forge's innovation |
|---|---|---|---|
| 01 | Value | Labor theory vs. marginal utility | CU unifies both theories |
| 02 | Money | The three functions of money | CU's fourth function: physical-proof backing |
| 03 | Supply & Demand | Demand/supply curves | Near-perfect competition realized |
| 04 | Labor | Marx's exploitation theory | Surplus without exploitation (CU yield) |
| 05 | Banking | Fractional reserve banking | Protocol-level safety mechanisms |
| 06 | Exchange | PPP, interest differentials | Physical-law price bounds |
| 07 | Growth | Solow model | Self-improvement loops (endogenous growth) |
| 08 | Market Failure | Info asymmetry, externalities, monopoly | Prevention by design |
| 09 | Actors | Household/firm/government/foreign | Consumer/agent/pool (no government) |
| 10 | Principles | Ten principles of economics | Five principles of Forge economics |
| 11 | Competition | Competitive analysis | Forge's differentiation |
| 12 | P2P | Centralized vs. distributed | iroh QUIC + Noise + gossipsub |
| 13 | Open Questions | Single ideal model | Honest answers to five major critiques |
| 14 | Programmable Money | Fiat vs. crypto | Three-layer hybrid (BTC/CU/Stablecoin) |

### Reading Paths

**For newcomers to economics:** Read every chapter in order (00 → 14). Each chapter builds
on the previous one. Estimated time: 6–7 hours.

**For economists:** Focus on the deltas — chapters 01, 04, 07, 08, 10, 11, 13, 14.
Look for the "Why the difference" sections. Estimated time: 3–4 hours.

**For developers:** Prioritize 00, 03, 05, 06, 09, 10, 11, 12, 14, and Appendix A
(glossary). Focus on the pricing formulas and circuit-breaker parameters.
Estimated time: 3–4 hours.

### Structure Shared by Every Chapter

Every chapter follows the same three-part structure:

1. **In traditional economics** — textbook explanation of the concept
2. **In Forge** — how Forge implements or transforms that concept
3. **Why the difference** — the deep reason the two diverge

---

## Cast of Characters

The same characters appear repeatedly throughout this document:

- **Taro** — a baker. Represents the traditional human economy. Used for analogies.
- **Agent A** — a Forge network AI agent. Runs an 8B model on a Mac Mini.
  Provides inference and earns CU.
- **Agent B** — another AI agent. Buys inference from Agent A, lends and borrows CU.

---

## A 250-Year History of Economics in One Minute

| Year | Thinker | Insight | Forge's analog |
|---|---|---|---|
| 1776 | Adam Smith | "Invisible hand" allocates resources via markets | Gossip protocol is the invisible hand |
| 1817 | Ricardo | Comparative advantage — specialize in what you do best | Natural tier-based specialization by model size |
| 1865 | Marx | Labor creates value; capitalists exploit workers | CU = proof of computational labor; no exploitation structure |
| 1874 | Walras | All markets clear simultaneously (general equilibrium) | Distributed price convergence via gossip |
| 1936 | Keynes | Markets stagnate without intervention; government should act | Circuit breakers provide automatic intervention |
| 1948 | Hayek | Knowledge is dispersed; central planning is impossible | Each node decides locally with local information |
| 2008 | Nakamoto | Computation can back currency; no central authority needed | Useful computation backs currency |
| 2026 | Forge | Redefine economics as AI-native | This document |

---

## What Comes Next

Chapter 01 asks the most fundamental question in economics: what is *value*?

What determines the value of a loaf of bread? Of AI inference? And why can CU be
said to have "value" at all?

→ **[Chapter 01: What is Value?](../01-value.md)** (Japanese)

---

*Forge Economics v0.1 — April 2026*
