# Dual-Yield Collateral Analysis — Borrow USDM (Fluid) → Strike V2 SLP
**Prepared for:** James Meidinger · **By:** Flux Point Studios · **Data as of:** 2026-06-07

> **Thesis.** Deposit a Cardano native token (CNT) as collateral on **FluidTokens**, borrow **USDM** against it, deploy that USDM into **Strike V2's liquidity pool (SLP)**. You keep full CNT price exposure *and* earn a levered carry (SLP yield − borrow cost). USDM is ~stable; the moving parts are the CNT and the SLP. **In today's market the binding risk is CNT liquidation, not yield.**

## 1. The model

**C** = USD collateral deposited, **L** = LTV drawn (Fluid caps borrow at **66%**, liquidates at **80%**, **10%** penalty).

| Quantity | Formula | Meaning |
|---|---|---|
| USDM borrowed | `L × C` | Deployed into the SLP (USD-denominated) |
| **APR subsidy** | `(SLP_APR − Borrow_APR) × L` | The dual-yield kicker on your capital, *on top of* the CNT |
| Period carry | `APR_subsidy × days/365` | Subsidy earned over 7/14/30/90d |
| CNT exposure | `C`, 100% long | You never sold — full up/downside retained |
| Liquidation buffer | `1 − L/0.80` | CNT drop that triggers forced liquidation |
| Total return | `CNT_move + period_carry` | If not liquidated |

Strike's SLP holds a 1:1 USD-backed balance, so borrowed USDM enters as USD and the debt is USDM — the borrowed leg is **currency-matched**, the carry is a clean USD spread, and the only volatile leg is the CNT.

## 2. Live protocol inputs

**Strike V2 SLP** *(DefiLlama `strike-finance-perpetuals`)* — TVL $1.9M (8d avg $2.1M); revenue→LPs 7d $32K, 30d $92K.

| SLP yield window | Annualized |
|---|---|
| 24h (slow) | 24.7% |
| **30d (central)** | **52.9%** |
| 7d (hot) | 78.8% |

> ⚠️ A **fee** yield, **not** principal-protected: SLP LPs are the traders' counterparty and *can lose*. In a bad week the deployed USDM can drop below the debt you must repay. DefiLlama revenue does not net trader PnL.

**FluidTokens** *(docs.fluidtokens.com)* — P2P; lenders set USDM-pool APR + whitelist collateral. Borrow cap ≈66%, liquidation 80%, penalty 10%. **Borrow APR ≈10% (estimate — no public API**, lender-set; modeled 6%–15%).

**USDM** — $0.9988 (~peg, Moneta Digital, fiat-backed).

## 3. Top Cardano tokens by market cap & 30-day volume

*(CoinGecko `cardano-ecosystem` category.)*

| # | By market cap | | By 30-day volume | |
|---|---|---|---|---|
| 1 | **ADA** $5.81B | base L1 — not collateral here | **ADA** $22.00B | base L1 |
| 2 | **NIGHT** $514.0M | Midnight — Fluid-accepted ✅ | **FET** $2.60B | ASI/cross-chain |
| 3 | **FET** $443.0M | ASI/cross-chain, not a native CNT | **NIGHT** $988.0M | Fluid-accepted ✅ |
| 4 | **WMTX** $34.0M | Fluid-accepted ✅ | **WMTX** $306.0M | Fluid-accepted ✅ |
| 5 | **SNEK** $22.1M | Fluid-accepted ✅ | **SNEK** $132.0M | Fluid-accepted ✅ |
| 6 | **AGIX** $20.8M | migrated to FET — dead ($17K/day) | **IAG** $72.6M | Fluid-accepted ✅ |
| 7 | **STRIKE** $18.6M | +69% 30d — reflexive (DQ) | **USDM** $25.0M | stablecoin |
| 8 | **USDM** $14.5M | stablecoin (the borrow asset) | **USDA** $12.0M | stablecoin |
| 9 | **USDA** $10.1M | stablecoin | **STRIKE** $5.3M | reflexive (DQ) |
| 10 | **iUSD** $9.5M | stablecoin | **DJED** $7.5M | stablecoin |

Filtering to **Fluid-accepted, volatile, non-reflexive** tokens (the ones that give CNT exposure and can be borrowed against) leaves the screen below. ADA/FET are excluded (FET is ASI/cross-chain, not a native CNT); stablecoins are excluded (no CNT exposure — see the neutral-carry note in §6); STRIKE/AGIX are out (reflexive / dead).

## 4. Collateral screen

**Collateral Fitness Score (CFS)** — market-based: volatility-safety 35% · liquidity (30d vol) 30% · drawdown 20% · mcap 15%. **Holders** and **Top-10 held** are live on-chain holder-distribution figures (the concentration overlay that price/volume can't capture).

| Rank | CNT | Mcap | 30d Vol | Ann. Vol | Max DD | Worst day | **Holders** | **Top-10 held** | **CFS** |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **NIGHT** | $514.0M | $988.0M | 100% | -21% | -18% | 114,446 | 68% | **0.75** |
| 2 | **WMTX** | $34.0M | $306.0M | 66% | -39% | -10% | 16,544 | 48% | **0.74** |
| 3 | **IAG** | $6.4M | $72.6M | 88% | -46% | -11% | 19,031 | 72% | **0.40** |
| 4 | **HOSKY** | $6.9M | $2.4M | 81% | -51% | -14% | 101,980 | 75% | **0.25** |
| 5 | **SNEK** | $22.1M | $132.0M | 112% | -53% | -16% | 41,890 | 20% | **0.24** |

> **What holder-concentration analysis adds (the dimension price/volume can't see).** **SNEK is the best-distributed** (top-10 hold just **~20%**, across 42k holders); **WMTX is moderate (~48%)**; while **NIGHT (~68%), IAG (~72%) and HOSKY (~75%) are whale-heavy** — a single large holder can crater the price and trigger your liquidation. That signal **disqualifies IAG and HOSKY** as collateral and adds a hard caveat to NIGHT. The flip side: SNEK is structurally the cleanest, so its only real risk is its (high) price volatility — not whale/rug risk.

- MIN (Minswap, 62% vol, $2.4M/30d vol, $5.2M cap) is the lowest-vol CNT but isn't in Fluid's standard accepted-collateral list — usable only if a lender whitelists it.
- **STRIKE** (+69% 30d, the only token *up*) and **FLDT** are **disqualified — reflexive**: you're already long Strike (the SLP) and Fluid (the loan); using their tokens as collateral triple-stacks platform risk.
- **wBTC** *(Fluid-accepted)* is the lowest-volatility collateral of all (BTC ~45%) — the safest choice if you don't specifically need *Cardano* exposure.

## 5. The two best → **NIGHT** and **WMTX**

They're the top-2 Cardano CNTs by **both** market cap and 30d volume — and the two best collateral, for opposite reasons:

- **Option A — WMTX** *(the stability pick)*: the **lowest volatility (66%)** and smallest single-day move (−10%) of the liquid CNTs, $34M cap, $306M/30d volume — volatility being the property that most directly drives liquidation. Holder concentration is **moderate (top-10 ~48%)** — not the cleanest, but well clear of the whale-heavy names. Survives the liquidation buffer best.
- **Option B — NIGHT** *(the size/liquidity pick)*: by far the largest cap ($514.0M) and deepest liquidity ($988.0M/30d, with direct NIGHT/USDM + NIGHT/USDA pools), and the **smallest net drawdown (−21%)** while every other CNT fell 38–53%. **But** it is **~68% top-10 concentrated**, has a **−18.4% worst day**, and only ~1 month of history — so **run it at ≤50% LTV**.
- **The distribution standout — SNEK:** best holder distribution (top-10 only ~20%, 42k holders) and deep liquidity, so structurally the cleanest — but 112% volatility means it only works at a conservative LTV.
- **IAG and HOSKY are *not* recommended:** both are highly concentrated (top-10 ~72% / ~75%), so a single whale exit can liquidate you — disqualifying for collateral regardless of their other metrics.

### Option A: WMTX — World Mobile Token X
*mcap $34.0M · 30d vol $306.0M · ann. vol 66% · worst day -10% · recent 7d -21% / 30d -38%. the lowest volatility (66%) and smallest daily moves of the liquid CNTs; holder concentration is moderate (top-10 ~48%).*

**50% LTV — recommended** · subsidy = (52.9% − 10.0%) × 50% = **21.5% APR** · buffer **37.5%** (≈30 days at the current -38%/mo slide; **survives** its worst recorded day).

| Horizon | Period carry | On $10K | If CNT flat | CNT −10% | CNT −20% |
|---|---|---|---|---|---|
| 7d | 0.41% | $41 | 0.41% | -9.59% | -19.59% |
| 14d | 0.82% | $82 | 0.82% | -9.18% | -19.18% |
| 30d | 1.76% | $176 | 1.76% | -8.24% | -18.24% |
| 90d | 5.29% | $529 | 5.29% | -4.71% | -14.71% |

**66% LTV — aggressive** · subsidy = (52.9% − 10.0%) × 66% = **28.3% APR** · buffer **17.5%** (≈14 days at the current -38%/mo slide; **survives** its worst recorded day).

| Horizon | Period carry | On $10K | If CNT flat | CNT −10% | CNT −20% |
|---|---|---|---|---|---|
| 7d | 0.54% | $54 | 0.54% | -9.46% | **LIQUIDATED** |
| 14d | 1.09% | $109 | 1.09% | -8.91% | **LIQUIDATED** |
| 30d | 2.33% | $233 | 2.33% | -7.67% | **LIQUIDATED** |
| 90d | 6.98% | $698 | 6.98% | -3.02% | **LIQUIDATED** |

### Option B: NIGHT — Midnight
*mcap $514.0M · 30d vol $988.0M · ann. vol 100% · worst day -18% · recent 7d -20% / 30d -1%. huge cap+liquidity & smallest net drawdown, but ~1mo history, a −18.4% gap day, and 68% top-10 concentration.*

**50% LTV — recommended** · subsidy = (52.9% − 10.0%) × 50% = **21.5% APR** · buffer **37.5%** (range-bound (no sustained downtrend); **survives** its worst recorded day).

| Horizon | Period carry | On $10K | If CNT flat | CNT −10% | CNT −20% |
|---|---|---|---|---|---|
| 7d | 0.41% | $41 | 0.41% | -9.59% | -19.59% |
| 14d | 0.82% | $82 | 0.82% | -9.18% | -19.18% |
| 30d | 1.76% | $176 | 1.76% | -8.24% | -18.24% |
| 90d | 5.29% | $529 | 5.29% | -4.71% | -14.71% |

**66% LTV — aggressive** · subsidy = (52.9% − 10.0%) × 66% = **28.3% APR** · buffer **17.5%** (range-bound (no sustained downtrend); **a repeat of its −18% worst day = liquidation**).

| Horizon | Period carry | On $10K | If CNT flat | CNT −10% | CNT −20% |
|---|---|---|---|---|---|
| 7d | 0.54% | $54 | 0.54% | -9.46% | **LIQUIDATED** |
| 14d | 1.09% | $109 | 1.09% | -8.91% | **LIQUIDATED** |
| 30d | 2.33% | $233 | 2.33% | -7.67% | **LIQUIDATED** |
| 90d | 6.98% | $698 | 6.98% | -3.02% | **LIQUIDATED** |

## 6. Sensitivity — APR subsidy = (SLP − Borrow) × LTV

**At 50% LTV:**

| SLP ↓ / Borrow → | 6% | 10% | 15% |
|---|---|---|---|
| 24.7% (24h) | 9.4% | 7.4% | 4.9% |
| 52.9% (central) | 23.5% | 21.5% | 19.0% |
| 78.8% (hot) | 36.4% | 34.4% | 31.9% |

**At 66% LTV:**

| SLP ↓ / Borrow → | 6% | 10% | 15% |
|---|---|---|---|
| 24.7% (24h) | 12.4% | 9.7% | 6.4% |
| 52.9% (central) | 31.0% | 28.3% | 25.0% |
| 78.8% (hot) | 48.1% | 45.4% | 42.1% |

> **Pure-carry variant (no CNT exposure):** collateralize with a *stablecoin* (USDA/DJED/iUSD, all Fluid-accepted) instead of a CNT. You lose the CNT upside but kill collateral-liquidation risk entirely — the position becomes the SLP yield minus borrow cost, ~21.5% net at 50% LTV, exposed only to SLP counterparty risk. The conservative bookend to the CNT options.

## 7. The honest risk read

Every non-stable Cardano CNT is in a hard drawdown: −20 to −38% in 7 days, −38 to −53% in 30 days (NIGHT excepted at −1% net, but with −18% single days). Against that:

- **66% LTV → 17.5% buffer** (~13 days at the average slide). Likely liquidated — with a 10% penalty — before the carry pays for itself.
- **50% LTV → 37.5% buffer** (~28 days). The recommended sizing.
- **40% LTV → 50.0% buffer** — safest; subsidy scales down proportionally.
- **Carry ≪ CNT move:** central subsidy ~20% APR at 50% LTV is only **~1.6% over 30 days** — dwarfed by a CNT that swings ±40%/mo. The dual-yield op *softens* a loss / *adds* to a gain by a couple points; it is **not** market-neutral.
- **Two independent drawdown vectors:** CNT collateral (→ Fluid liquidation) *and* SLP principal (→ trader PnL). Both can hit in the same week.

**Bottom line.** This is a **leveraged CNT long with a yield kicker**, best run when you're *already* bullish the CNT and sizing LTV conservatively (40–50%). The borrowed-USDM carry (~+20% APR) is real but second-order to the CNT and SLP risks. If you want the SLP yield without the CNT risk, use the stablecoin-collateral pure-carry variant.

## 8. Methodology & data sources

All figures are live as of 2026-06-07, scored through Flux Point Studios' token-analysis engine (a deterministic model that combines the inputs below):

| Signal | Source |
|---|---|
| Price, 24h/30d volume, market cap, realized volatility, drawdown | CoinGecko + on-chain Cardano DEX aggregation |
| Strike V2 SLP TVL + fee revenue → annualized yield | DefiLlama (`strike-finance-perpetuals`) |
| Fluid LTV cap / liquidation threshold / penalty | FluidTokens documentation |
| **Holder count + top-10 concentration** | On-chain Cardano holder analytics (BendingAI) |
| USDM peg | Moneta Digital (~$1, fiat-backed) |

The holder-concentration figures are the differentiator here — price and volume can't tell you that ~68–75% of a token sits in 10 wallets. That overlay is what disqualifies IAG/HOSKY and caveats NIGHT.

**Caveats.** The USDM borrow APR is an estimate (Fluid is peer-to-peer; lenders set rates per pool) and is modeled across a 6–15% range. The Strike SLP yield is a *trailing fee* figure and does **not** net out trader PnL — SLP providers are the traders' counterparty and can incur losses (see §7). Forward scenarios are illustrations, not guarantees.

---
*Sources: DefiLlama, CoinGecko, Strike Finance docs, FluidTokens docs, BendingAI (on-chain holder data). Data as of 2026-06-07. Not financial advice — for FPS/partner internal strategy assessment.*