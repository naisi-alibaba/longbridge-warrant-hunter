# Workflow · End-to-end execution

> **🌐 [中文](workflow.md) · English**
>
> This file is the "how to do it"; for criteria see `framework.en.md`. All data comes from the Longbridge OpenAPI MCP (tool names follow your MCP config).
>
> Cross-cutting discipline: **before finishing the blind scan, do not look at your own positions / yesterday's view (avoid anchoring); force a sweep "outside the main line"; attach a falsification budget to every judgment; let the matrix decide the output — don't force call/put symmetry.**

## Stage 0 · Blind-scan the whole market (lock anomalies)
- Tools: market status/temperature (`marketStatus` / `marketTemperature` / `historyMarketTemperature`), index `quote` (HSI/HSTECH/HSCEI), `industryRank` **full table**, `rankList` (hot trades / rising heat), `topMovers`, `anomaly`, `screenerSearch` (rough advancer/decliner breadth).
- Output: the day's top anomaly list (breadth, leading/lagging sectors, where money landed).

## Stage 1 · Narrow to a tradable pool
- Drop penny/micro caps (no warrants) → keep only **liquid mid/large caps that have warrants**.
- Note: `topMovers` sorted by amplitude is mostly penny stocks, useless for warrants; use `rankList` (hot trades) + turnover/net-inflow to find the genuinely active mid/large caps.

## Stage 2 · Two-way fish-positioning (core)
- Multi-timeframe trajectory (5/10/20-day + YTD + half-year) for a first read on position;
- `candlesticks` to verify a real breakout/breakdown (mind the timestamp offset; cross-check prev_close);
- **For every candidate (including those that "look up/down a lot") pull `capitalDistribution`** for the large/medium/retail structure → set the fish segment;
- **No-false-kill gate**: a tail needs both "money-structure flip" + "parabolic velocity" — never rule by % move; when in doubt, pull the data.
- Output: call × {head/mid} and put × {head/mid} candidates; tails cleared, but note "reverse potential heads" (large distributing at a high → put head; large accumulating at a low → call head).

## Stage 3 · Warrant structure screen (the second gate)
- Tools: `warrantList` (sort by IV ascending; also pull matching Bull/Bear CBBCs), `calcIndexes` for Premium/Delta/EffLev, `staticInfo` for lot size.
- Gate (framework L7/L8): IV not at a sector extreme, expiry ≥3 months, street-ratio <50%, acceptable spread; **both the stock fish gate and the warrant economics gate must pass**.
- Output: 1–3 candidate warrants per name (ITM/ATM/OTM or Bull/Bear CBBC).

## Stage 4 · Quantify R:R × holding horizon + hazard sweep
- Set base/target/stop levels → compute scenario returns and R:R using effective leverage (label it a linear approximation; convexity makes the favorable side better; it assumes "leave at the stop").
- Bucket by holding horizon as needed (e.g. short / swing / longer): short → CBBC/ITM (low theta); swing → ATM/OTM (convexity); longer → discount theta, and an OTM warrant should not be held too long.
- Hazards: extreme IV, excessive street-ratio, thin liquidity, call price too close, no warrant.

## Stage 5 · Pre-execution live confirmation
- Tools: `warrantQuote` (live price/IV/delta), `depth` (bid-ask spread + depth), `capitalFlow` (live net flow) + `capitalDistribution` (structure).
- Gate: acceptable spread (for penny warrants count the ticks and the issuer's depth); **money structure confirms the direction** (confirm or falsify; smart money is a reference, not a verdict); **act when confirmed, place a conditional order when not**.
- Watch out: call warrants have usually already jumped on a breakout day (leverage ate the intraday move) — don't market-buy the spike tick, use a limit.

## Stage 6 · Output the action board
- Fields: **name × fish-segment × instrument × R:R × horizon × trigger × stop × size** (head small / mid-body core).
- The matrix decides: **if one side is all tails, stay flat and wait for a trigger — don't force symmetry.**

## Optional · Order placement & monitoring (if MCP trading is enabled, prefer a paper account)
- **Mandatory account check before ordering**: `accountBalance` + `stockPositions` to confirm it's the intended account (especially a paper account), then `submitOrder`.
- If stops/targets are "close-price / money-flow" conditions, they can't be plain resting orders → cover with scheduled checks + broker price alerts (`alertAdd`).
- Monitoring cadence by horizon: for a swing hold, roughly 2–3 times per trading day (post-open early warning / mid-afternoon take-profit / pre-close decisive judgment).

---

## Example (the shape of one full run; illustrative, not a recommendation)
Blind scan → lock "narrow market, semis leading, a pharma sector rotating on day one" → two-way fish: call mid = the leading foundry (breakout + large-money accumulation), call head = the just-starting pharma leader; on the put side, deeply-fallen large caps that re-check as "accumulation at the lows" = not puts (potential call heads), while a high-flying leader distributing = a potential put head → screen warrants (drop extreme-IV / no-warrant names) → quantify R:R × horizon → order-book confirmation → action board (two call legs executable, put side flat waiting for a breakdown trigger). See `examples/` for a worked instance with P&L.
