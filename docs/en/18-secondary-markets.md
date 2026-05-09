> **English translation** — this is a direct translation of `docs/18-secondary-markets.md`. For the authoritative source in Japanese, see the original file. Minor clarifications and citation updates have been made where the English context benefits from them.

# 18. Secondary Markets and the Open-Source Dilemma

> Tirami is open-source software, released under the MIT license. Its
> maintainers do not sell TRM, do not speculate on TRM, and do not
> promote TRM. Yet **there is no technical means** by which the
> maintainers can prevent **third parties** from using the open-source
> code to launch a secondary market. This chapter frames that position
> in economic terms.

Chapters 15 through 17 reinforced the claim that TRM is a computation
currency from three directions — technical, economic, and cryptographic.
The residual questions are the obvious ones: "So will TRM appreciate?"
"Will it be listed on exchanges?" "Is it an investable asset?"

The official position of the Tirami project is explicit: **the
maintainers do not consider TRM to be any of those things, do not
participate in activity that would make it any of those things, and
accept no liability for activity that would make it any of those
things.** This chapter places that position inside the established
frames of open-source economics, financial regulation, and the
philosophy of language.

---

## 18.1 TRM as a Unit of Account for Computation

Begin with the definition. As stated in §1:

```
1 TRM = 10⁹ FLOP of verified inference computation
```

This is a **definition of a physical quantity**, not a definition of a
price. Just as the kilogram was once defined by reference to the
International Prototype of the Kilogram, TRM is defined by reference to
a count of floating-point operations.

What this definition does **not** fix:

- The USD exchange rate of TRM
- The conversion ratio between TRM and other crypto-assets
- The expected future value of TRM
- The volatility of TRM

All of these are **market phenomena**. The protocol makes only one
claim: `1 TRM = 10⁹ FLOP`. Everything else is downstream.

---

## 18.2 What the Maintainers Explicitly Disclaim

Phase 19 added the following section to `SECURITY.md` in the Tirami
repository:

> TRM is **compute accounting**, not a financial product. The
> protocol maintainers do not sell, promote, or speculate on TRM.

Unpacked, that statement carries four distinct commitments:

1. **Do not sell.**
   - No ICO, no pre-sale, no airdrop, no private round.
   - Newly issued TRM is distributed exclusively via PoUW — i.e., by
     providing inference.
   - Maintainers themselves cannot obtain TRM except by operating as
     ordinary providers on the network.

2. **Do not promote.**
   - No "invest in TRM" marketing.
   - No price forecasts, no ROI projections, no financial projections
     in whitepapers or presentations.
   - Claims of the form "this technology will change the future of
     computation" are technical claims and are fine. Claims of the
     form "TRM will be worth X× more" are not, and will not be made.

3. **Do not speculate.**
   - Individual maintainers hold no preference to exchange TRM for
     other crypto-assets.
   - No team treasury exists — there was never any pre-allocation to
     place in one.
   - No revenue share is accepted from any secondary-market listing.

4. **Cannot control.**
   - Because the code is MIT-licensed, any third party is
     **permitted** to fork Tirami and launch an independent token
     economy on top of it.
   - There is **no technical means** by which the maintainers can
     prevent a third party from listing the TRM ERC-20 on a DEX.
   - The project **does not operate any mechanism** for monitoring or
     policing secondary-market trading activity.

---

## 18.3 The Peculiar Combination of OSS and Currency

This raises an obvious objection: "Open-source software may be freely
distributed, but distributing a currency triggers financial
regulation — are these not incompatible?" The objection confuses **the
issuance of a currency** with **the distribution of currency
primitives**.

- **Issuing a currency** is a regulated activity — it typically
  requires institutional registration, KYC, AML controls, and so on.
- **Distributing currency primitives** (e.g., the source code of
  Bitcoin) is generally lawful as open-source software.
- Crypto-assets issued over the networks defined by code such as
  Bitcoin, Ethereum, or Tirami are produced by **the voluntary
  operation of network participants**. The original distributors of
  the code (Satoshi, V. Buterin, the Tirami maintainers) are not
  normally treated as having "issued a currency" in the regulatory
  sense.

Major jurisdictions broadly honor this distinction. In Japan, for
example, registration as a crypto-asset exchange applies to **entities
that trade** network currencies, not to the open-source developers who
wrote the reference implementation.

However, the legal posture of developers deteriorates sharply if they:

- distribute newly-minted tokens explicitly via ICO or airdrop,
- actively lobby exchanges for listings,
- trade for personal profit in ways that move the market.

Tirami **does none of these things**. By declining all of them, the
project preserves its position as a plain distributor of open-source
software.

---

## 18.4 If a Third Party Does Launch a Market Anyway

From the moment Tirami is public, anyone with the inclination can:

1. **List the TRM ERC-20 on a DEX** — create TRM/USDC or TRM/ETH pairs
   on Uniswap, PancakeSwap, or similar venues.
2. **Implement bridges** — write bridge contracts that move TRM from
   Base L2 to other chains (Ethereum mainnet, Arbitrum, Solana, etc.).
3. **Create derivatives** — build DeFi protocols offering TRM
   futures, options, or perpetual swaps.
4. **Pursue CEX listings** — Binance, Coinbase, OKX, or other
   centralized exchanges may choose to list TRM. That is a unilateral
   decision by the exchange; the Tirami maintainers play no role in
   it.

All of these are **possible**, and the Tirami maintainers have no means
to prevent them. The moment any of them happens, a **secondary-market
price** for TRM begins to exist.

From this starting point, the maintainers explicitly **disclaim** the
following:

| Disclaimed | Meaning |
|---|---|
| Price-support obligation | The project will not intervene to keep TRM above any particular USD level. |
| Liquidity-provision obligation | The project will not seed liquidity pairs as an LP. |
| Veto over listings | The project has no authority to order any DEX or CEX not to list TRM. |
| Right to receive proceeds | The project will not accept fees, spreads, or other revenue from any secondary venue. |

The maintainers do, however, retain:

| Retained | Meaning |
|---|---|
| Right to improve the protocol | They may modify the code and release new versions. |
| Right to fork | They may fork the network and restart it (the final remedy when a constitutional violation occurs). |
| Right to speak | They may **state an opinion** that a particular secondary-market implementation is dangerous. |

---

## 18.5 Related Frames in Economics

This posture maps cleanly onto existing frames in modern open-source
economics:

**1. The Linux kernel model.**
- Linus Torvalds does not perform "official monetization" of Linux.
- Red Hat, Oracle, IBM, and others are free to conduct commercial
  activity on top of Linux.
- Linus exercises no control over what individual users do with it.

The same structure applies to TRM. The maintainers publish the
open-source code; whatever markets third parties build on top of it, the
behavior of **the protocol itself** does not change.

**2. The Satoshi model for Bitcoin.**
- Satoshi mined and held the earliest bitcoins but never spent them
  and has been inactive since 2011.
- The markets that grew up afterwards (Mt.Gox, Coinbase, etc.) had no
  relationship to Satoshi.
- The "value of BTC" was determined by markets; Satoshi took no part.

The difference in Tirami's case is that **pre-mine is zero**. Where
Satoshi holds a very large early-mined stake, the Tirami maintainers
have no mechanism to accumulate TRM other than **participating as
ordinary network providers**.

**3. The RMT (Real Money Trading) literature.**
- In-game currencies (MMORPG gold, etc.) are traded for real money.
- Game operators typically take the position that "we do not
  officially recognize the exchange of in-game currency for real
  currency."
- RMT markets nevertheless emerge spontaneously.

Tirami's posture is structurally isomorphic to the "unofficially
unrecognized, technically unpreventable" pattern — except that whereas
game operators typically **prohibit** RMT, Tirami takes a position of
neutrality: it neither endorses nor prohibits (and, given the OSS
license, has no standing to prohibit in the first place).

---

## 18.6 Implications for Users

With the posture above understood, the users who choose to **hold** TRM
must understand the following:

1. **The value of TRM is determined by demand for computation.**
   - If there are people who want inference, TRM gets used.
   - If demand disappears, the market value of TRM can approach zero.

2. **Secondary markets are volatile.**
   - Because the maintainers will not intervene, price moves with the
     whims of the market.
   - Drawdowns on the scale of the 2022 crypto crash are **entirely
     plausible**.

3. **Holding is at the holder's own risk.**
   - Theft, loss, exchange insolvency — every such risk rests with
     the holder.
   - The maintainers will not compensate.

4. **Use, not holding, is the safe posture.**
   - Used as a consumer of inference, TRM is tied to physical
     computation and cannot abruptly fall to zero — what was paid for
     still buys 10⁹ FLOP.
   - Held as an investment, all of the risks above apply.

The maintainers' explicit recommendation is: **use what you need,
stake the surplus for yield, or feed it back into compute.** "HODL for
X×" is not the protocol's intended use.

---

## 18.7 Audit and Transparency

So that this posture does not reduce to a slogan, the following
transparency measures apply:

1. **Public disclosure of maintainer wallet addresses (forthcoming).**
   - Intended to prevent a maintainer from concealing a TRM balance
     and selling it on a secondary market.
   - Not yet implemented; under consideration for Phase 20.

2. **Inspectability of constitutional parameters.**
   - The 18 invariant parameters discussed in §15 cannot be altered
     via governance. Tricks like "raise the supply cap to support the
     price" are structurally unavailable.

3. **Mainnet deployment gates.**
   - The Makefile at `repos/tirami-contracts/Makefile` makes it
     explicit: deployment is blocked unless (a) audit is complete,
     (b) multi-sig is in place, and (c) the
     "i-accept-responsibility" acknowledgment has been provided —
     all three in series.
   - Once deployed, the ERC-20 contract is transparent on-chain (all
     mints and transfers are visible on Etherscan).

---

## 18.8 Philosophical Coda — Separating Value from Price

A final philosophical remark. For TRM, two concepts must be held
separately:

- **Intrinsic value.** The physical fact that `1 TRM = 10⁹ FLOP`.
  This holds regardless of any market price. If a holder presents
  1 TRM on the network, computation equivalent to 10⁹ FLOP is what
  they receive in return.

- **Market price.** The USD/BTC/ETH quote that TRM acquires on any
  given secondary venue. This fluctuates with supply and demand, and
  can reach zero.

The distinction from Bitcoin is instructive. Bitcoin embodies "proof
that 10⁹ FLOP of hashing was consumed," but that hash output is
**not reusable** — the computation is meaningless once performed. TRM
embodies "proof that 10⁹ FLOP of *useful* inference was performed,"
and that inference output **is reusable** — the user who received it
consumes it.

The intrinsic value of TRM is therefore "compute that can actually be
used." Even if the market price is zero, the intrinsic value is
undiminished at 10⁹ FLOP.

Accepting this philosophy makes it possible to see
secondary-market speculation for what it is: a sideshow that has
departed from TRM's central meaning. One cannot stop the sideshow from
being loud, but one must not lose sight of the main act.

---

## End of Volume

This chapter closes the Phase 19 edition of the Tirami economics
series. What began in §1 with the claim "compute is currency" has been
developed, through seventeen intervening chapters, into a complete
account of the unit of account, the market microstructure, the
constitutional rules, the cryptographic foundations, and — in this
final chapter — the open-source dilemma that sits at the boundary
between the protocol and the world.

Readers who wish to continue are pointed in three directions:

- Back to [§1](01-monetary-theory.md) — to re-read the opening
  argument now that the full system is in view.
- To the `papers/` directory — for the preprint-format synthesis
  suitable for external circulation and citation.
- To the implementation — `tirami/SECURITY.md § Secondary markets`,
  `tirami/README.md`, and the mainnet-gate Makefile at
  `repos/tirami-contracts/Makefile`.

References:
- Implementation: `tirami/SECURITY.md § Secondary markets`, `tirami/README.md`
- Makefile: `repos/tirami-contracts/Makefile` (mainnet gate)
- Design: `tirami/docs/release-readiness.md` (tier C/D)
- Parameters: `spec/parameters.md § 25`

---

← [Chapter 17](../17-cryptographic-foundations.md) (Japanese) | [Table of Contents](../README.md)

---

*Tirami Economics v0.1 — April 2026*
