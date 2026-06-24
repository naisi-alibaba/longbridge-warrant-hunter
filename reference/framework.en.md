# Framework · Warrant Risk-Reward & Fish-Positioning (criteria)

> **🌐 [中文](framework.md) · English**
>
> Long & short. This file is the "how to think"; for the steps see `workflow.en.md`. All data comes from the Longbridge OpenAPI MCP.

## L0 Core
A warrant has no standalone value — **risk-reward is synthesized**. With no directional view, expected value is negative (time decay + bid-ask spread are guaranteed outflows).
Real decomposition: `ΔW/W ≈ effective_leverage × ΔS/S + vega × ΔIV − theta × Δt − spread`.
Main path: **set direction → fish-positioning → money confirmation → pick the warrant → quantify R:R → order-book execution** (direction-driven, not relative-value arbitrage).

## L1 The truth about risk-reward
- **Leverage scales both gain and loss equally — it does NOT improve the R:R ratio.**
- Asymmetry comes from three things: ① convexity (gamma — strongest ATM/OTM); ② the target-vs-stop distance ratio on the underlying; ③ a strictly honored stop.
- Every R:R number assumes: **you leave at the stop, you do not hold to zero.**

## L2 Fish-positioning (the organizing ruler, symmetric both ways)

| | Healthy call | Healthy put | Tail (forbidden) |
|---|---|---|---|
| Position | mid-body after breakout | mid-body after breakdown | parabolic top / waterfall bottom |
| Large orders | net in (accumulation) | net out (distribution) | **structure flips** |
| Retail | net out | net in | **structure flips** |
| Distance from history | irrelevant | irrelevant | irrelevant |

- **Head** = a fresh turn · small size · loss-capped instrument (OTM warrant);
- **Mid-body** = confirmed · core size · efficient instrument (ATM warrant / CBBC);
- **Tail** = an absolute veto.

## L3 Tail criterion: structure + velocity, not magnitude
- "Fell too much" for a put = "rose too much" for a call = **the same most-expensive instinct**. A downtrend makes fresh relative lows by definition — that is exactly what it is supposed to do.
- Tail = **money-structure flip (distribution/accumulation reverses) + parabolic velocity**, both required.
- **Never kill on the % move. A false kill = a hole in the framework — fix it.**

## L4 Money structure: smart money is a reference, not a verdict
- Structure: accumulation = large in / retail out; distribution = large out / retail in; tail = the structure flips.
- **But smart money can be wrong, can bull-/bear-trap, and "large orders" may just be issuer hedging.** Flow's job is to "confirm or falsify my directional call," **not to make the call for me**; even a huge one-day print is not a guarantee.
- Usage: directional judgment first, flow verification second; when flow contradicts price/position it is a *warning*, not a *reverse signal*.

## L5 Entry/exit discipline
- **Wait only when unconfirmed; act once confirmed** (breakout/breakdown + aligned same-day money structure = enough to enter, no second confirmation needed).
- **Put the hesitation on the exit side, not the entry side.** "Don't chase" ≠ "wait for a cheaper price" — it = enter now · right size · add on proof · use a limit, don't sweep the spike · don't go all-in on a one-day move.
- **Following smart money means following it IN and OUT**; conviction is rented from the flow → the moment structure flips, leave.

## L6 Execution asymmetry between the two sides
Direction is symmetric, execution is not: **puts get tighter stops and smaller size** (downside bounces and short squeezes are more violent than upmoves).

## L7 Two independent gates
**"Stock fish-segment" and "warrant economics" are judged separately and must not be merged.**
- A stock can be mid-body (accumulated), yet its warrant uneconomic (extreme IV / breakeven too far / leverage crushed) → still no trade.
- Conversely, however cheap the warrant, if the stock is a tail → no trade. **Both gates must pass.**

## L8 Warrant economics criteria (the second gate)
IV (compare across warrants on the same underlying; avoid sector extremes) · effective leverage · premium/breakeven · moneyness + delta (ITM = higher hit-rate / OTM = higher convexity) · **expiry ≥3 months** (avoid the theta cliff) · street-ratio <50% · issuer spread · conversion ratio.
- CBBCs: no theta/IV but a mandatory call — the **call price must sit beyond your stop**.
- Instrument by fish-segment: head → OTM loss-capped; mid-body → ATM warrant / CBBC.
- Instrument by horizon: short hold → CBBC / ITM (low theta); swing → ATM/OTM (convexity); longer → discount theta, slow direction → CBBC.

## L9 Data realities (Longbridge MCP)
- No direct warrant theta (estimate from IV + expiry);
- Candlestick timestamps may be offset by +1 day (to read a day's close, take the bar stamped the previous calendar day; cross-check with prev_close);
- Warrants only exist on liquid mid/large caps (penny-stock anomalies are irrelevant);
- **Some names have no warrants → not executable = dropped.**
