# Longbridge Warrant Hunter · HK Warrant Risk-Reward Hunter

**🌐 [中文](README.md) · English (current)**

> A Claude **Agent Skill** that hunts high risk-reward Hong Kong warrants (窝轮, calls/puts), long & short, on top of the **Longbridge OpenAPI MCP**. Direction-first, evidence-driven — not a news summarizer. Plain warrants only, no CBBCs.

> 📌 **Current version v5 (2026-08-12)** — criteria expanded from "money structure + CLV" into a **multi-evidence system**: CLV demoted (empirically weak mean-reversion, never an entry trigger); adds a three-dimension entry-evidence vote, two anti-manipulation dimensions, an adversarial blind review, and a conclusion-usage discipline; the workflow is fully rewritten as **strict numbered steps** (steps 0-9, seven fields each). See [`CHANGELOG.md`](CHANGELOG.md) for the version diff and methodology evolution.

---

## ✨ What it does

Runs a strict step sequence (each step has preconditions / calls / criteria / veto gate — see `reference/workflow.md`):

```
Step0 entry routing → Step1 blind-scan anomalies → Step2 narrow to names-with-warrants
   → Step3 two-way fish-positioning (7-dimension position read) → Step4 three-dim entry-evidence vote (≥2 positive to enter)
   → Step5 warrant selection (three selection questions · second gate) → Step6 R:R + leverage-sized stop
   → Step7 adversarial blind review → Step8 action board + order-book confirm → Step9 position monitoring & exit
```

**Core idea (fish-positioning)**: size up only the "mid-body" (a confirmed trend mid-way), take a small taste at the "head" (a fresh turn), and never the "tail" (parabolic top / waterfall bottom). **Symmetric both ways** — buying puts on a stock that already fell a lot is the same mistake as chasing calls at a parabolic top. Tails are judged by **money-structure flip ∩ parabolic velocity (both required), never by the size of the move.**

## 📦 Install

> The repo root *is* the skill body (`SKILL.md` + `reference/`). Note: **the skill folder name = the slash command**, so clone into a folder named `longbridge-warrant-hunter`. It's picked up automatically (live detection), **no restart** — unless `~/.claude/skills/` did not exist before, in which case restart Claude Code once.

### Option A · Clone into your skills dir (most reliable, recommended)

```bash
# Personal / global (all projects)
git clone https://github.com/naisi-alibaba/longbridge-warrant-hunter.git ~/.claude/skills/longbridge-warrant-hunter

# Or project-only
git clone https://github.com/naisi-alibaba/longbridge-warrant-hunter.git .claude/skills/longbridge-warrant-hunter
```

### Option B · One-line plugin marketplace

In Claude Code:

```text
/plugin marketplace add naisi-alibaba/longbridge-warrant-hunter
/plugin install longbridge-warrant-hunter@longbridge-warrant-hunter
/reload-plugins
```

Then invoke as `/longbridge-warrant-hunter` (relies on `.claude-plugin/marketplace.json`).

### Option C · Files only, no git history

```bash
npx degit naisi-alibaba/longbridge-warrant-hunter ~/.claude/skills/longbridge-warrant-hunter
```

## 🔌 Requirements

- **Claude Code / a Claude client that supports Agent Skills**
- **Longbridge OpenAPI MCP** (the only data source, required) — quotes, market temperature, industry ranks, capital flow / distribution, warrant chains, order book, and optionally order placement & price alerts.
  - Apply for Longbridge OpenAPI credentials and configure the MCP: see Longbridge official docs.
  - Tool names follow your MCP config (may carry a prefix).

## 🚀 Usage

Trigger in chat (or via your client's skill-invocation):

- "Any good risk-reward call/put warrants in HK right now?" → whole-market two-way scan
- "Pick me a warrant on SMIC" → single-name fish-positioning + selection
- "Call or put now, which warrant, what stop/target?"

The skill reads `reference/framework.{md|en.md}` (criteria) + `reference/workflow.{md|en.md}` (steps).

## 🎛️ Tunable parameters (customize your strategy)

Every threshold marked **☆** in the criteria is an **empirical value you can override** — not a verdict, just an executable default. Want it more aggressive or more conservative? Change the number. How: **state the override in your prompt** (e.g. "raise the R:R gate to 2.0 and loosen the active-buy threshold to 60%"), or edit the corresponding line in `reference/framework.md` / `reference/workflow.md` in a local fork. Thresholds marked **★** are mechanism/evidence-backed hard limits — not recommended to change.

| Parameter | Default (☆) | What it controls | Raise = | Lower = | Source |
|---|---|---|---|---|---|
| **R:R gate** | ≥ 1.5 | below this, no trade | pickier, fewer trades, higher per-trade EV | looser, more trades | framework L8 / workflow Step6 |
| **Mid-body ATR band** | 1~3×ATR | how far from breakout counts as "mid-body" (core-size zone) | enter later, ride trend | only fresh breakouts, exit earlier | framework L2 |
| **Parabolic tail · return percentile** | ≥ 90th | self-percentile line for "overheated sprint" (★ percentile, not absolute move) | fewer tail-calls | earlier tail-calls | framework L3 |
| **Parabolic tail · daily-return z** | ≥ 2.0 | z-score line for abnormal acceleration | fewer tail-calls | more sensitive | framework L3 |
| **Entry vote · active-buy upper** | ≥ 55% | tick dimension scores +1 | require stronger buying | easier to reach "positive" | framework L9 |
| **Entry vote · active-buy lower** | ≤ 45% | tick dimension scores −1 | flip to negative sooner | more tolerant | framework L9 |
| **Selection · deep-OTM line** | OTM ≥ 25% | beyond this = "lottery", cull (except loss-capped head) | allow more OTM / convexity | stay nearer ITM | framework L8 |
| **Selection · ITM delta line** | \|delta\| ≥ 0.5 | high-delta line preferred for mid-body | require deep ITM | allow ATM | framework L8 |
| **Selection · breakeven margin** | target ≥ 1.5× breakeven move | how far target must clear breakeven | stricter | accept thin margin | framework L8 |
| **Selection · premium cap** | ≤ 20% | above this bleeds, cull | tolerate higher premium | only low premium | framework L8 |
| **Stop · bearable warrant drawdown D_w** | −10~12% | start point for the leverage-sized stop (÷ real leverage = underlying tolerance) | wider stop, hold longer | tighter stop, cut faster | framework L8 |
| **Put-side execution asymmetry** | tighter stop / size −1 tier | extra conservatism on the short side | — | off = fully symmetric | framework L6 |
| **Position poll frequency** | every 30 min | interval for catching a structure flip | fewer calls, less responsive | more sensitive, more calls | workflow Step9 |

> **★ Hard limits (don't change)**: expiry ≥3 months (theta cliff), street-ratio <50% (issuer control), tail by self-percentile not absolute move, CLV never an entry trigger, plain warrants only (no CBBCs). These are mechanism- or evidence-backed; changing them breaks the framework's coherence.
>
> **Read framework L12 "conclusion-usage discipline" before tuning**: these defaults were mostly derived in a specific market regime (range-bound); a trending / high-volatility regime may want different values, and a small sample only speaks to mechanism, not returns. Validate your overrides on a paper account.

## 📁 Example

- [📒 **Paper-trading ledger (2026-06-17 → 07-02)**](examples/2026-paper-trading-ledger.md) — **one table for "action · rationale · cognitive iteration · P&L"**: the full trajectory, cumulative realized −HK$4,145, showing how each loss "bought" a framework version (v1→v3.1). ⚠️ Paper-account demo, not a track record.
- [2026-06-24 · Semis lead + CXO rotation](examples/2026-06-24-semis-cxo.md) — a full run: a compliant entry that still bled into a loss; **section 5 "Exit post-mortem" is the live teaching case for the v3 exit discipline** (the cost of entering on structure but exiting on price).

## 🧭 Iron rules

- **CLV is never an entry trigger**: empirically weak mean-reversion (high CLV tends to fall back next day); treating it as an "accumulation footprint" gets the direction backwards — CLV is only an exit-side soft cue.
- **Entry needs positive evidence**: a three-dimension vote (structure / flow / tape) requires ≥2 positive to enter; **neutral = no entry** (not "no reversal = enter").
- Tails are judged by **structure flip ∩ parabolic velocity (both required), never by % move**; for any name that "looks up/down a lot" you MUST pull the money-structure before ruling it a tail. **A false kill = a hole in the framework.**
- **Don't pick a warrant by leverage alone**: a deep-OTM warrant's "high leverage" is a lottery — check moneyness / breakeven / premium first (the three selection questions).
- **Adversarial blind review**: before acting, run a separate blind pass (no access to the bullish rationale) against a manipulation red-flag list; prefer under-killing to over-reading.
- **Two independent gates**: stock fish-segment ∩ warrant economics (IV not extreme / expiry ≥3mo / street-ratio <50% / acceptable spread).
- **Smart money is a reference, not a verdict**; act when confirmed, put hesitation on the exit side; puts get tighter stops & smaller size; the matrix decides output — don't force symmetry.
- **Conclusion-usage discipline**: not-backtestable = a ceiling on confidence; conclusions hold only within the current market regime; a small sample speaks to mechanism, not returns.

## ⚠️ Disclaimer

This skill is a research/analysis tool only and is **NOT investment advice**. HK warrants are **high-leverage instruments that can go to zero**. Plain warrants only — this skill does not trade CBBCs (uncontrollable mandatory-call risk). Any trade decision based on this skill's output, and its outcome, is solely the user's responsibility. **Strongly validate the framework on a paper account before considering real money.** The authors accept no liability for any losses.

## 📄 License

MIT — see [LICENSE](LICENSE).
