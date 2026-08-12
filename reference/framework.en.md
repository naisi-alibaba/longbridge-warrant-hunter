# Framework · Warrant Risk-Reward & Fish-Positioning (criteria)

> **🌐 [中文](framework.md) · English**
>
> Long & short. This file is the **"why" and the criteria definitions**; for the strict execution sequence see `workflow.en.md` (this file does not say "call X then do Y", it only defines the criteria). All data comes from the Longbridge OpenAPI MCP.
>
> **Version v5 (2026-08-12)** · changes in [`../CHANGELOG.md`](../CHANGELOG.md). This release expands the criteria from a single "money structure + CLV" into a **multi-evidence system**: rewrites L4 (CLV empirically demoted to weak mean-reversion, never an entry trigger), adds the parabolic-velocity quantification in L3, and adds L9 (three-dimension entry-evidence vote), L10 (two anti-manipulation dimensions + de-sanctifying the fish-body), L11 (adversarial blind review), L12 (conclusion-usage discipline).
>
> **Threshold marks**: ★ = a hard threshold with mechanism/evidence support; ☆ = an empirical value, to be calibrated, user-overridable; ⚠ = source uncertain, use with care. Every ☆ number is not a verdict — it is an executable placeholder.

---

## L0 Core
A warrant has no standalone value — **risk-reward is synthesized**. With no directional view, expected value is negative (time decay + bid-ask spread are guaranteed outflows).
Real decomposition: `ΔW/W ≈ effective_leverage × ΔS/S + vega × ΔIV − theta × Δt − spread`.
Main path: **set direction → fish-positioning → multi-evidence confirmation → pick the warrant → quantify R:R → adversarial review → execution** (direction-driven, not relative-value arbitrage).

## L1 The truth about risk-reward
- **Leverage scales both gain and loss equally — it does NOT improve the R:R ratio.**
- Asymmetry comes from three things: ① convexity (gamma — strongest ATM/OTM); ② the target-vs-stop distance ratio on the underlying; ③ a strictly honored stop.
- Every R:R number assumes: **you leave at the stop, you do not hold to zero.**
- **⚠ "Highest effective leverage" is the most expensive trap in selection.** The reported "effective leverage" is an instantaneous theoretical figure; a deep-OTM warrant's tiny price denominator inflates it into a false high — a "lottery leverage" that only pays off on a violent move. When the direction is right but the underlying moves only a little, that OTM leverage never materializes and theta + spread bleed it instead. **Never pick by "biggest leverage"** — a high leverage number must be read together with moneyness + delta (see L8).
- **⚠ Size the stop with delta-derived real leverage** (real leverage ≈ delta × underlying price ÷ warrant price ÷ conversion ratio), not the raw `effective_leverage` field — the latter is distorted for deep-OTM warrants (see L8 stop sizing).

## L2 Fish-positioning (the organizing ruler, symmetric both ways)

| | Healthy call | Healthy put | Tail (forbidden) |
|---|---|---|---|
| Position | mid-body after breakout | mid-body after breakdown | parabolic top / waterfall bottom |
| Large orders | net in (accumulation) | net out (distribution) | **structure flips** |
| Retail | net out | net in | **structure flips** |
| Distance from history | irrelevant | irrelevant | irrelevant |

- **Head** = a fresh turn · small size · loss-capped instrument (OTM warrant);
- **Mid-body** = confirmed · core size · efficient instrument (ATM / ITM warrant);
- **Tail** = an absolute veto.

**Computable segment boundaries** (removing the ambiguity of "mid-body", normalized by ATR):
- Head = from the last channel breakout/breakdown, progressed ≤ 1×ATR (☆);
- Mid-body = progressed 1~3×ATR (☆) with the trend continuing, not flipped;
- Tail = L3 criterion triggered.
> ATR multiples are ☆ structural placeholders, calibrated to the name's volatility and horizon.

> **This framework trades plain warrants (calls/puts) only — no CBBCs.** CBBCs carry a mandatory-call mechanism: touch the call price and they knock out to zero instantly and uncontrollably — a different risk nature, so they are excluded wholesale. All instrument choices below are made only among ITM/ATM/OTM plain warrants.

## L3 Tail criterion: structure + velocity, not magnitude
- "Fell too much" for a put = "rose too much" for a call = **the same most-expensive instinct**. A downtrend makes fresh relative lows by definition — that is exactly what it is supposed to do.
- **Tail = money-structure flip ∩ parabolic velocity, both required.**
- **Never kill on the % move. Without the velocity proof you may NOT rule it a tail. A false kill = a hole in the framework — fix it.**

**Parabolic-velocity quantification** (a "parabolic sprint" requires all three):
1. **Cumulative move sits at the stock's own historical percentile ≥ 90th (★ self-percentile, not absolute move** — absolute magnitude false-kills strong names);
2. **The latest 1-day return has a z-score ≥ 2.0 (☆)** vs the last 20 days (abnormal acceleration); the z window's mean/std **excludes the current bar** (so the judged quantity doesn't pollute its own normalization baseline);
3. **The rate-of-change of the move (ROC acceleration) > 0 and at its own high percentile** (the sprint is accelerating).
> The 90th percentile / z≥2 are ☆ empirical values. Any one missing = velocity proof fails → not a tail.

## L4 Money structure & CLV: smart money is a reference, not a verdict
- Structure: accumulation = large in / retail out; distribution = large out / retail in; tail = the structure flips.
- **But smart money can be wrong, can bull-/bear-trap, and "large orders" may just be issuer hedging.** Flow's job is to "confirm or falsify the directional call," **not to make the call**; even a huge one-day print is not a guarantee. When flow contradicts price/position it is a *warning*, not a *reverse signal*.
- **⚠ CLV (close location = (close−low)/(high−low)) is a weak mean-reversion signal, not a trend footprint.** Empirically: with each stock's own percentile as the ruler, a high CLV (close near the day's high) tends to **fall back** next day rather than continue up (t+1 information coefficient ≈ −0.03, uncradeable after cost). Therefore:
  - **CLV is never an entry trigger** — don't enter because "it closed high today = accumulation"; that's backwards;
  - CLV's only sound role is on the **exit side** (while holding, a close breaking below its own low percentile = a soft cue of weakening structure, not a standalone exit);
  - Entry direction uses real-time money structure + tape (see L9), not CLV.

## L5 Entry/exit discipline
- **Wait only when unconfirmed; act once confirmed** (price breakout/breakdown + L9 evidence sufficient = enough to enter, no second confirmation needed).
- **Put the hesitation on the exit side, not the entry side.** "Don't chase" = enter now · right size · add on proof · use a limit, don't sweep the best ask · don't go all-in on a one-day move; it does **not** mean "wait for a cheaper price".
- **Following smart money means following it IN and OUT**; conviction is rented from the flow → the moment structure flips, leave.
- **Exit-signal priority**: ① a reverse structure flip (call: large money net-in → net-out + retail net-out → net-in; puts mirror) = a **first-class signal, cut/close that day**; ② price stop = second-class backstop. When the first-class signal fires, you may **not** downgrade it to "watch and hold" on the grounds that price hasn't hit the stop. Execute **whichever of this or the L8 leverage-sized stop triggers first**.

## L6 Execution asymmetry between the two sides
Direction is symmetric, execution is not: **puts get tighter stops and their size dropped one tier** (downside bounces and short squeezes are more violent than upmoves).
> Note: whether to keep this asymmetry is calibrated against live short-side performance — if short legs are not materially riskier, it may revert to fully symmetric. Kept for now.

## L7 Two independent gates
**"Stock fish-segment" and "warrant economics" are judged separately and must not be merged.**
- A stock can be mid-body (accumulated), yet its warrant uneconomic (extreme IV / breakeven too far / leverage crushed) → still no trade.
- Conversely, however cheap the warrant, if the stock is a tail → no trade. **Both gates must pass.**

## L8 Warrant economics criteria (the second gate)
IV (compare across warrants on the same underlying; avoid sector extremes) · effective leverage · premium/breakeven · moneyness + delta (ITM = higher hit-rate / OTM = higher convexity) · **expiry ≥3 months (★ avoid the theta cliff)** · **street-ratio <50% (★ avoid issuer control)** · issuer spread · conversion ratio.

- **The three selection questions (how to choose among candidates on the same name — don't just look at leverage)**:
  1. **How far ITM/OTM?** Deep OTM (OTM ≥ 25%, ☆) = needs a large favorable move to be worth anything; direction right but only a nudge = you bleed for nothing — unless explicitly a loss-capped head betting on a big move, don't pick it. ITM/ATM (|delta| ≥ 0.5, ★) track the underlying closely, the mid-body default.
  2. **How far is breakeven from spot?** Convert both "breakeven" and "target" to an **underlying move**: `R_be = |breakeven − spot| / spot`; `R_tg = |target − spot| / spot`. **IF R_tg < R_be → cull** (target can't reach breakeven, EV negative); R_be ≤ R_tg < 1.5×R_be (☆) → thin margin, downgrade/flag; R_tg ≥ 1.5×R_be → pass.
  3. **How much does premium eat?** Premium > 20% (☆) means most of the price is time value / water level — flat-to-small-favorable action bleeds → cull, or keep only for a very-strong-direction short hold. A low-premium ITM warrant leaves the directional payoff to you.
- **Instrument by fish-segment**: head → OTM loss-capped (betting on a big move, loss capped); mid-body → ATM / ITM warrant (high delta, low premium, converts direction efficiently).
- **Instrument by horizon**: short → ITM (low theta, high delta); swing → ATM/OTM (convexity); longer → discount theta, OTM not to be held too long.
- **Size the stop by leverage, NOT by copying the stock's technical level.** Reverse the order: first set the **bearable warrant drawdown D_w** (e.g. −10~12%, ☆) → ÷ real leverage = underlying tolerance D_s → add to entry for the underlying stop (call `spot×(1−D_s)`, put `spot×(1+D_s)`). Copying a neckline/platform/round number lets leverage multiply the stop several-fold (e.g. underlying −8% × 3x ≈ warrant −24%, breaking L1). **Use delta-derived real leverage (L1), not the raw effLev field; where the OTM folding is unreliable, flag it explicitly.**
- **R:R gate**: `RR = |target warrant return| / |stop warrant return|`. **RR < 1.5 (☆) → no trade.** Map the underlying's three scenarios (base/target/stop) to warrant returns via real leverage (linear approximation; convexity favors the aligned side; assumes "leave at the stop").

## L9 Entry evidence vote: require positive evidence, not "no reversal = enter"
The biggest source of losses is not "trading a reversal" but "letting neutral / no-positive-evidence through" — price broke out, but money didn't speak, and you carry it on price alone. **Upgrade entry from "enter if there's no reverse signal" to "enter only with sufficient positive evidence."**

A three-dimension equal-weight vote (call = bullish, put mirrors), each dimension scores +1/0/−1:
- **Structure dimension** (source `capital_distribution`): large net-in + retail net-out = accumulation (+1); large net-out + retail net-in = distribution (−1); else 0.
- **Flow dimension** (source `capital_flow`): cumulative net inflow > 0 with slope ≥ 0 = inflow (+1); < 0 with slope ≤ 0 = outflow (−1); direction/slope conflict = 0.
- **Active-buy dimension** (source `trades`, auction specials removed): active-buy share of active volume (denominator = active buy + active sell, excluding neutral) ≥ 55% (+1) / ≤ 45% (−1) / in between = 0.

Verdict (for a call; a put reads "positive" as bearish-direction evidence):
- **≥2 dimensions positive → evidence sufficient, may enter**;
- ≥2 dimensions negative → direction already falsified, no entry (if holding, consider reversing);
- otherwise (including all-neutral) = **evidence insufficient, no entry** (gather evidence or switch names).

> **Role boundary of the vote**: it only decides "enter or not" (an admission gate / veto), **not the direction itself** — direction still comes from the price channel (see workflow Step3); the money vote is an admission gate, not a source of direction (consistent with L4 "money is a reference, not a verdict").
> 55/45% are ☆ empirical values, to be calibrated, overridable.

## L10 Anti-manipulation: price/volume/net-flow is the script the operator writes; identifying the operator means going down to the hardest-to-fake layer
**De-sanctifying the fish-body**: the framework treats a "standard fish-body" (breakout holds + rising volume + steady advance + room above) as the safest entry picture — but this **is exactly the textbook definition of a bull-trap picture**. Price, volume, net inflow are all the cheapest things for an operator to construct for you; the standard fish-body is the highest-information-entropy segment, most in need of external corroboration, not the safest one. On a controlled name, "new high on shrinking volume = top divergence" can even be a clean contra-indicator.

**Two hardest-to-fake anti-manipulation ingredients** (present the objective facts only, no verdict — you judge wash/lure per the framework):
- **Tick-by-tick active-direction sequence** (source `trades`): large orders **sweeping same-direction** vs **alternating buy/sell** wash trades are two entirely different natures.
- **CCASS full-participant concentration / changes** (source `broker_holding_detail`, T+1, not real-time): top-participant concentration, yesterday's top add/reduce participants. Cross with today's tape — e.g. "price rising while the concentrated participant is reducing" = distribution-into-strength raw material.

> ⚠ Data-tier constraint: the real-time broker queue (`brokers`) / 10-level depth (`depth` levels 2-10) are LV2 permissions; `trades` and `broker_holding_detail` (exchange disclosure) are **not limited by quote tier** and are fully available on LV1 — so the two anti-manipulation dimensions anchor on these two tools.

## L11 Adversarial independent blind review (a decision-hygiene structure)
Don't fold anti-manipulation into the main judgment (it pollutes every read) — make it an **independent second "devil's advocate" pass**:
- **Blind**: the reviewer sees only the candidate leg's parameters + raw multi-dimension / microstructure data, and **not the bullish/bearish rationale** — independence is the core, otherwise it's just self-endorsement.
- **A direction-grouped red-flag list**: long legs check "breakout picture too textbook = bull-trap, controlled new-high on shrinking volume, distribution-into-strength"; short legs check "panic-tail bear-lure, dig-a-pit shakeout, oversold-bounce trap, short squeeze".
- **Restraint**: default to pass; only rule "suspected manipulation" when multiple red flags are present and the evidence is hard — **prefer under-killing to over-reading**.
- **Output**: an independent review verdict (suspected/pass + confidence + specific concern), to compare against before acting. Technical reading runs the main line, anti-manipulation runs this independent line — the two are decoupled.

## L12 Conclusion-usage discipline (how to treat your own criteria conclusions)
Criteria can deceive; more deceptive still is overconfidence in the conclusions. A few meta-rules:
- **Adversarial re-check ≠ independent evidence**: re-computing on the same data in the same regime is only same-source recompute; true independent validation needs a different regime / out-of-sample.
- **Not-backtestable = a ceiling on confidence**: large-order structure / flow is same-day, no history, not backtestable — usable for live judgment but **cannot be "backtest-validated"**, so confidence has a ceiling. Backtestable (e.g. CLV) and non-backtestable are complementary; don't treat the non-backtestable as ironclad.
- **Every "settled conclusion" implies "only within the current market regime"**: what a range-bound market validates may not hold in a trending / high-volatility regime — don't extrapolate.
- **A small sample speaks to mechanism, not returns**: a dozen trades can only clarify "is the mechanism right", not claim "this makes money".
- **Metric discipline**: for any number entering a conclusion, ask three questions first — which convention, what's the denominator, can it be reproduced in one command. The same batch of trades can show a "win/loss ratio" that differs by more than 2× (by money / by percent / by profit factor), steering you toward completely different fixes.
- **A normalization denominator must not come from the judged quantity itself**: otherwise you normalize away the very magnitude information you're trying to judge.

## L13 Data realities (Longbridge OpenAPI MCP)
- No direct warrant theta (estimate from IV + expiry); `delta`/`effective_leverage`/`premium`/`itmOtm`/`balance_point` are **only in `warrant_list`**; `warrant_quote` lacks them (it has IV/expiry/street-ratio/strike/conversion only).
- **`calc_indexes` DOES return the underlying's volume fields** (`volume`/`turnover`/`capital_flow`/`volume_ratio`/`amplitude`/`turnover_rate`/multi-period change rates — broker-convention snapshot values); what returns null are the **warrant/option-only fields** (delta/gamma/theta/vega/rho/IV/premium/itmOtm/strike/balance_point/effLev — a stock has no such concepts). ⚠ But `calc_indexes` gives only a **single-point snapshot, not a time series**: ATR% / CLV / volume percentile / Donchian / parabolic velocity and other **per-day time-series derivatives must be self-computed from `candlesticks`**.
- **Change-rate convention differs across three tools** (disambiguate before converting): `calc_indexes.change_rate` = "−1.95" (the percent value itself); `rank_list.chg` and `warrant_list.*` fields = "−0.0195" (a decimal fraction, needs ×100).
- Candlestick timestamps may be offset by +1 day (to read a day's close, take the bar stamped a calendar day earlier; cross-check with `prev_close`).
- Warrants only exist on liquid mid/large caps (penny-stock anomalies are irrelevant); **some names have no warrants → not executable = dropped.**
- **Price has a native push alert; structure (`capital_distribution` large/retail + `capital_flow`) has none, and large-money is same-day with no history** → position structure monitoring can **only** be polled actively (e.g. every 30 min); the `capital_flow` slope is the only real-time quantity that flips **before price**.
- `capital_distribution` may return empty or an abnormal timestamp pre-open / in no-trade windows → treat as "structure not ready", score that dimension neutral, don't fabricate accumulation/distribution.
- **Permission tier**: the core chain depends only on tools fully available on LV1 (quote / calc_indexes except warrant-only fields / candlesticks / capital_distribution / capital_flow / trades / broker_holding_detail / warrant_list incl. delta·effLev·premium·IV / warrant_quote / corp_action / static_info); **only `depth` levels 2-10 and the `brokers` real-time queue are LV2**. Probe the tier at runtime with one `depth` call (asks length ≥5 = LV2).
