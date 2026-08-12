# Workflow · End-to-end strict execution sequence

> **🌐 [中文](workflow.md) · English**
>
> This file is the **"how to do it" — a strict step sequence**; criteria definitions and rationale are in `framework.en.md` (this file only cites `see framework Lx`, it does not re-explain). All data comes from the Longbridge OpenAPI MCP (tool names follow your MCP config, may be prefixed).
>
> **Seven fields per step**: [Goal] [Preconditions] [Calls] [Criteria IF-THEN] [GATE veto] [Output] [Failure handling]. Any step whose GATE fails drops out (back up a level or back to the candidate pool).
>
> **Global preconditions** (must hold before Step 1):
> - GATE: before Step 0/1 (blind scan) outputs the anomaly list, **do not** call `stock_positions` / read yesterday's conclusions (anti-anchoring).
> - Attach a falsification budget to every judgment; the matrix decides output — don't lower a side's bar to force call/put symmetry.
> - Conclusions carry a regime caveat (no extrapolation); a small sample speaks to mechanism not returns; for any number first answer "which convention / denominator / reproducible?" (see framework L12).

---

## Step 0 · Entry routing

[Goal] Decide blind-scan the whole market vs. go straight to a single name.
[Preconditions] None.
[Calls] None.
[Criteria IF-THEN]
- IF the user did not name a symbol THEN enter whole-market blind-scan mode → Step 1.
- IF the user named a symbol THEN skip the scan, that name goes straight to → Step 3 (fish-positioning).
[GATE veto] None.
[Output] Mode flag + (if any) target symbol.
[Failure handling] None.

---

## Step 1 · Blind-scan the whole market (lock anomalies)

[Goal] Produce the day's anomaly list (breadth, leading/lagging sectors, where money landed); frame the "main line + off the main line" candidate pool.
[Preconditions] Step 0 = scan mode.
[Calls] (all parallel)
- `market_temperature` (market=HK): temperature/valuation/sentiment.
- `quote` (["HSI.HK","HSTECH.HK","HSCEI.HK"]): index prices/changes.
- `rank_list` (key from `rank_categories` second_tags, e.g. hot_all-hk): chg/inflow/balance(turnover)/market_cap/industry. ★ the single best tool for "active mid/large caps".
- `top_movers` (markets=HK, **sort=2 heat**, not sort=1 amplitude): movers + alert_reason.
- `anomaly` (market=HK): anomaly alerts (alert_time is Unix seconds, convert).
- `constituent` (HSI.HK/HSTECH.HK/HSCEI.HK): strong constituents + tags.
- `screener_search` (HK, prevchg>0 / <0 each once for total): rough advancer/decliner breadth.
[Criteria IF-THEN]
- IF `screener_search` returns 463/timeout/empty THEN estimate breadth from rank_list up/down stats, note "breadth degraded".
- Convention note: `rank_list.chg` is a decimal fraction (−0.0195 = −1.95%); `calc_indexes.change_rate` is the percent value itself (−1.95) — opposite conventions, don't mix.
[GATE veto] None (this step only collects).
[Output] Candidate pool + the day's tone (temperature/index direction/leading-lagging sectors).
[Failure handling] A single discovery-layer tool failing → mark that dimension UNAVAILABLE, don't block; `constituent`'s rise_num/fall_num are always 0, unusable for breadth.

---

## Step 2 · Narrow to a tradable pool

[Goal] Drop untradable names from the pool, keep only liquid mid/large caps that have warrants.
[Preconditions] Step 1 candidate pool.
[Calls]
- For each candidate, `warrant_list` in parallel (warrant_type=["Call"] and ["Put"] once each, sort_by=Volume, sort_order=Descending): does a tradable warrant exist.
[Criteria IF-THEN]
- IF the name has neither Call nor Put → no warrant.
- IF all candidate warrants are zombies (last_done=0 or IV=0 or effective_leverage=null) → treat as no tradable warrant.
- Use rank_list balance(turnover)/market_cap to drop penny/micro caps.
[GATE veto] **A name with no tradable warrant → drop** (nothing downstream). GATE: `top_movers` amplitude ranking is not a source of names (mostly penny stocks with no warrants).
[Output] Tradable pool: {name, has Call/Put, turnover, market cap}.
[Failure handling] `warrant_list` returns empty → that name drops.

---

## Step 3 · Two-way fish-positioning (judge direction + set segment)

[Goal] Output one segment label per name: call-head / call-mid / put-head / put-mid / tail (clear out).
[Preconditions] Step 2 tradable pool (no-warrant names already dropped).
[Calls] (7-dimension position read; no single indicator)
- `calc_indexes`: ChangeRate/FiveDayChangeRate/TenDayChangeRate/HalfYearChangeRate/YtdChangeRate (multi-period momentum) + VolumeRatio/Amplitude/TurnoverRate (snapshot supplement).
- `candlesticks` (day, ~120 bars, forward_adjust=true): self-compute ① distance to 20/60-day high, to year high/low ② Donchian20 channel ③ ATR% ④ volume percentile / volume-price divergence ⑤ the three parabolic-velocity quantities (return self-percentile + daily-return z + ROC acceleration, see framework L3).
  Note: ATR%/CLV/volume percentile/parabolic and other **per-day time-series can only be self-computed from candlesticks** (calc_indexes gives a single-point snapshot, not a series); timestamps may be +1 day, align with prev_close (see framework L13).
- `capital_distribution`: large/retail net in-out.
- `capital_flow`: today's cumulative net inflow series (now/peak/trough/slope).
[Criteria IF-THEN]
- IF structure flip (accumulation↔distribution reverses) AND parabolic velocity resonates on all three quantities (see framework L3) THEN mark "tail" → clear out (record as reverse potential head).
- ELSE IF above Donchian upper mid-body (1~3×ATR) AND large net-in AND retail net-out THEN "call-mid".
- ELSE IF below Donchian lower mid-body AND large net-out AND retail net-in THEN "put-mid".
- ELSE IF just broke the channel (≤1×ATR, day-1/2 start) THEN corresponding "head" (small-size candidate).
- ELSE mark "insufficient evidence" → not a candidate.
[GATE veto]
- A tail ruling requires "structure flip ∩ parabolic velocity" both present; missing either, may NOT rule a tail (see framework L3).
- **Do not rule a tail on the % move alone.**
- GATE: **CLV is forbidden as an entry trigger** (see framework L4); CLV is only an exit-side soft cue, not part of this step's position read.
- When ATR% is extreme, discount the reliability of dimensions ①-④ (a meta-condition).
[Output] Candidate table: {name, direction, segment, 7-dim snapshot, large-order structure direction, capital_flow slope sign}.
[Failure handling]
- `capital_distribution` has no same-day data / timestamp earlier than the open → structure dimension neutral, don't rule a tail on it.
- `candlesticks` timestamp cross-check fails → mark the name "time-axis uncertain", produce no executable leg.

---

## Step 4 · Entry evidence vote (is directional evidence sufficient)

[Goal] Decide whether a candidate leg has enough positive evidence to enter (per framework L9).
[Preconditions] Step 3 output legs that are non-tail and not "insufficient evidence".
[Calls] (three sources, may reuse Step 3's batch)
- `capital_distribution` (structure dim) · `capital_flow` (flow dim) · `trades` (count≥200, active-buy dim).
- trades handling: **remove auction specials** (trade_type ∈ {U,Y,M,P}), then active-buy share = active buy ÷ (active buy + active sell), excluding neutral.
[Criteria IF-THEN] (call = bullish; put mirrors. Each dim +1/0/−1, see framework L9)
- Structure dim: large net-in + retail net-out → +1; large net-out + retail net-in → −1; else 0.
- Flow dim: net inflow>0 & slope≥0 → +1; <0 & slope≤0 → −1; conflict → 0.
- Tick dim: active-buy ≥55% (☆) → +1; ≤45% (☆) → −1; in between → 0.
- IF ≥2 dims positive THEN evidence sufficient.
- ELSE IF ≥2 dims negative THEN direction falsified.
- ELSE insufficient (incl. all-neutral).
[GATE veto]
- **Only "evidence sufficient" passes to selection**; "insufficient" and "falsified" **both do not enter** (neutral = no entry, not "no reversal = enter").
- GATE: the vote decides admission only, not direction; direction comes from Step 3's price channel (see framework L9 role boundary).
[Output] Passing legs: {name, direction, segment, vote detail (three-dim signs)}.
[Failure handling] A missing dimension → score it 0 (neutral), treat strictly as "insufficient", don't fabricate.

---

## Step 5 · Warrant selection (which warrant · second independent gate)

[Goal] Select an economic warrant for a passing leg, or rule "good stock but no economic warrant" out.
[Preconditions] Step 4 passed + the target level already set (see Step 6, breakeven depends on it — the target may be computed within this step first).
[Calls]
- `warrant_list` (warrant_type directed Call/Put, sort_by as needed): strike_price/itm_otm/balance_point/premium/delta/effective_leverage/implied_volatility/outstanding_ratio/expiry_date/conversion_ratio.
- `static_info`: lot_size.
  ⚠ warrant_list's premium/itm_otm/IV/delta are **decimal fractions** (0.20=20%, −0.25=−25%); before comparing thresholds use the decimal or ×100, don't read 0.20 as 20.
[Criteria IF-THEN] (three selection questions + economics hard gate, see framework L8)
- Moneyness: IF OTM ≥ 25% (☆) AND NOT (head AND betting a big move) THEN cull; |delta|≥0.5 (★) ITM/ATM preferred for mid-body.
- Breakeven: `R_be=|breakeven−spot|/spot`, `R_tg=|target−spot|/spot`. IF R_tg<R_be THEN cull; R_be≤R_tg<1.5×R_be (☆) → margin downgrade; ≥1.5×R_be → pass.
- Premium: IF premium>20% (☆) THEN cull, or keep only a very-strong-direction short hold.
- Economics hard gate (AND, any failure culls): expiry≥3mo (★) · street-ratio<50% (★) · IV not extreme among the name's candidates (☆: drop top/bottom 10%) · acceptable spread · conversion ratio known.
- Instrument by segment/horizon: head→OTM; mid-body→ITM/ATM; short→ITM; swing→ATM/OTM.
[GATE veto] All candidate warrants of a name culled → **that name drops (however good the stock, see framework L7 two independent gates)**.
[Output] Selected warrant: {code, moneyness, delta, real leverage, breakeven, premium, expiry, street-ratio}.
[Failure handling] warrant_quote lacks delta/leverage/premium/breakeven — always fetch these from `warrant_list`.

---

## Step 6 · Quantify R:R + leverage-sized stop

[Goal] Compute the underlying stop line and R:R, decide whether the gate is met.
[Preconditions] Step 5 selected warrant + Step 1's `quote` spot.
[Calls] Reuse `quote` spot + warrant_list delta/effective_leverage.
[Criteria IF-THEN] (see framework L8)
- Set target: the more conservative of ① a prior structure level (prior high/low / channel opposite side) ② spot ±2×ATR (☆). The target must be set before the selection breakeven check.
- Stop sizing (reverse order): ① set bearable warrant drawdown D_w=−10~12% (☆) → ② underlying tolerance D_s = D_w ÷ real leverage (real leverage delta-derived: delta×spot÷warrant price÷conversion ratio, **not the raw effLev field**) → ③ stop line call `spot×(1−D_s)` / put `spot×(1+D_s)`.
- R:R = |target warrant return| ÷ |stop warrant return| (mapped via real leverage, linear approx).
[GATE veto] **IF RR < 1.5 (☆) THEN drop** (R:R insufficient). Never copy a neckline/round number for the stop (leverage multiplies it several-fold).
[Output] Executable leg: {warrant, direction, spot, target, underlying stop, RR, size tier}.
[Failure handling] OTM warrant effLev/delta folding distorted → flag the stop as unreliable, cut size or drop.

---

## Step 7 · Adversarial independent blind review (before acting)

[Goal] Set aside the main rationale and re-check the candidate leg from the operator's angle (per framework L11).
[Preconditions] Step 6 executable leg.
[Calls] `trades` (read direction sequence) · `broker_holding_detail` (CCASS full participants, filter parti_number starting with A = clearing houses).
[Criteria IF-THEN] (blind: parameters + raw data only, no bullish/bearish rationale)
- Tape: large orders same-direction = trend material; alternating = suspected wash.
- CCASS: price rising AND the concentrated participant reducing (chg_1/5/20 negative) → distribution-into-strength material. ⚠ CCASS is T+1, don't mix with today's trades as "real-time participants".
- Long-leg flags: breakout too textbook / controlled shrinking-volume new high / distribution-into-strength; short-leg flags: panic-tail bear-lure / dig-a-pit shakeout / oversold bounce / short squeeze.
- IF multiple flags present AND evidence hard THEN "suspected manipulation".
- ELSE pass (prefer under-killing to over-reading).
[GATE veto] Suspected with hard evidence → pull the leg or downgrade to watch.
[Output] Review verdict: {suspected/pass, confidence, concern}.
[Failure handling] CCASS/trades fetch fails → review degrades to "unreviewed", flag and let a human decide whether to act.

---

## Step 8 · Action board + pre-execution live confirmation

[Goal] Finalize the action board and do a live order-book confirmation before ordering.
[Preconditions] Step 7 passed legs.
[Calls] (parallel)
- `warrant_quote` (live price/IV/expiry/street-ratio) · `depth` (spread) · `capital_flow` (live net flow) · `capital_distribution` (structure).
- `depth` levels depend on permission: asks length≥5=LV2, use 10-level spread; only 1 level=LV1, degrade to "best level only, penny-warrant spread not precisely judgeable".
[Criteria IF-THEN]
- IF spread acceptable AND money structure confirms direction live THEN confirmed → execute at the size tier.
- IF unconfirmed THEN place a conditional order, don't market-chase.
- IF it's a breakout day THEN the warrant has likely priced in the intraday move → no market order, limit only (don't chase the best ask).
- IF direction = put THEN tighter stop band, size tier down one (see framework L6).
[GATE veto]
- GATE: live-confirmation tools (warrant_quote/depth) failing → **do not order**, don't execute off a stale snapshot.
- GATE: the matrix decides output — a direction whose candidates are all filtered out → that direction "stays flat", never lower the bar to force symmetry.
[Output] Action board: {name × segment × warrant × R:R × horizon × trigger × stop × size} (head small / mid-body core).
[Failure handling] See GATE: live tool failure = defer.

---

## Step 9 · Position monitoring & exit (optional, if holding)

[Goal] While holding, catch a structure flip and exit ahead of price.
[Preconditions] A held leg.
[Calls] Poll `capital_distribution` + `capital_flow` every 30 min (structure has no push, active poll only, see framework L13); CLV computed at close from `candlesticks`.
[Criteria IF-THEN] (exit-signal priority, see framework L5)
- IF a reverse structure flip is confirmed (large-order direction flip ∩ flow slope deterioration) THEN first-class signal → cut/close that day (**do not** downgrade to watch on the grounds price hasn't hit the stop).
- ELSE IF only one proof THEN warning; if it doesn't escalate over several rounds, hold.
- IF the underlying hits the Step 6 stop line THEN second-class backstop close.
- CLV closing below its own low percentile → soft cue of weakening structure (not standalone, pairs with the first-class signal).
[GATE veto] First-class signal and price stop — execute whichever triggers first.
[Output] Exit instruction / hold.
[Failure handling] Missing poll data → mark that round "structure unknown", fall back to the price stop backstop.

---

## Optional · Ordering & account (if MCP trading is enabled, prefer a paper account)
- GATE: `submit_order` **must be preceded** by `account_balance` + `stock_positions` confirming it's the intended account (especially a paper account).
- Penny warrants: LO limit orders only, never MO.
- "Close-price / money-flow" conditional stops can't be plain resting orders → cover with scheduled polling + `alert_add` (condition=price_rise/fall/percent_*, frequency=once/daily/every) price alerts.

---

## Appendix · All dimensions → MCP tool cheat-sheet

| Dimension | MCP tool | Key fields / usage | Convention pitfall |
|---|---|---|---|
| Multi-period momentum | `calc_indexes` | ChangeRate/FiveDay.../HalfYear.../Ytd... | **change_rate is the percent value itself** (−1.95=−1.95%), not a fraction |
| Volume ratio/amplitude/turnover | `calc_indexes` | VolumeRatio/Amplitude/TurnoverRate | — |
| Dist-to-high/low·ATR·vol pct·CLV·parabolic·Donchian | `candlesticks` (day,~120) | all self-computed from bars | only bars give a series; timestamp +1 day, prev_close check |
| Large/mid/retail structure | `capital_distribution` | large/retail net in-out | same-day only · no history · not backtestable; empty pre-open → neutral |
| Cumulative net inflow | `capital_flow` | per-minute cumulative series (now/peak/trough/slope) | a series not a point; slope is the only pre-price signal |
| Tick active direction | `trades` | per-tick direction (Up=buy/Down=sell/Neutral) | remove trade_type∈{U,Y,M,P}; denominator excludes neutral |
| CCASS full participants | `broker_holding_detail` | list[].shares.value + chg_1/5/20 + strong | T+1 not real-time; filter parti_number starting with A |
| Candidate warrants (selection) | `warrant_list` | strike/itmOtm/balancePoint/premium/delta/effLev/IV/street/expiry/conv | **premium/itmOtm/IV/delta are decimal fractions**; Call/Put only |
| Warrant live quote | `warrant_quote` | price/IV/expiry/street/strike/conv | **lacks delta/leverage/premium/breakeven/moneyness** — fetch from warrant_list |
| Order-book spread | `depth` | best bid/ask levels | LV1 only 1 level, levels 2-10 are LV2 |
| Corporate actions / earnings | `corp_action` | future ReportDate/DividendExDate | filter date≥today; no new entry near earnings |
| Lot size | `static_info` | lot_size | R:R by lots |
| Blind-scan discovery layer | `market_temperature`/`anomaly`/`top_movers`/`rank_list`/`constituent`/`screener_search` | temperature/anomaly/ranks/constituents/breadth | some subsystems intermittently 463 → degrade; constituent's rise/fall_num always 0 |

> **Permission in one line**: on LV1 (e.g. HK_L1_OpenAPI) every tool above is available except `depth` levels 2-10 and the `brokers` real-time queue; the two anti-manipulation dimensions (trades + broker_holding_detail) are not tier-limited. Probe the tier at runtime with one `depth` call (asks length ≥5 = LV2). Any call failing / returning empty → score that evidence dimension neutral and flag the gap explicitly, never fill with a guess; a live entry-trigger tool failing = defer the order.
