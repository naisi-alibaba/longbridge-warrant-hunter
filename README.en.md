# Longbridge Warrant Hunter · HK Warrant Risk-Reward Hunter

**🌐 [中文](README.md) · English (current)**

> A Claude **Agent Skill** that hunts high risk-reward Hong Kong warrants (窝轮) and CBBCs (牛熊证), long & short, on top of the **Longbridge OpenAPI MCP**. Direction-first, evidence-driven — not a news summarizer.

> 📌 **Current version v3 (2026-06-26)** — adds "exit-side structure re-screen" + "stop sized by leverage". See [`CHANGELOG.md`](CHANGELOG.md) for the version diff and methodology evolution.

---

## ✨ What it does

Runs a fixed workflow:

```
Blind-scan the whole market for anomalies → two-way fish-positioning (head/mid/tail)
   → money-structure verification (no false kills) → screen warrants (IV/leverage/expiry/
   street-ratio/spread, two gates) → quantify risk-reward × holding horizon
   → order-book confirmation → an "action board"
   (name × segment × instrument × R:R × horizon × trigger × stop × size)
```

**Core idea (fish-positioning)**: size up only the "mid-body" (a confirmed trend mid-way), take a small taste at the "head" (a fresh turn), and never the "tail" (parabolic top / waterfall bottom). **Symmetric both ways** — buying puts on a stock that already fell a lot is the same mistake as chasing calls at a parabolic top. Tails are judged by **money-structure flip + velocity, never by the size of the move.**

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
- "Pick me a warrant / CBBC on SMIC" → single-name fish-positioning + selection
- "Call or put now, which warrant, what stop/target?"

The skill reads `reference/framework.{md|en.md}` (criteria) + `reference/workflow.{md|en.md}` (steps).

## 📁 Example

- [2026-06-24 · Semis lead + CXO rotation](examples/2026-06-24-semis-cxo.md) — a full run: a compliant entry that still bled into a loss; **section 5 "Exit post-mortem" is the live teaching case for the v3 exit discipline** (the cost of entering on structure but exiting on price).

## 🧭 Iron rules

- Tails are judged by **structure flip + parabolic velocity, never by % move**; for any name that "looks up/down a lot" you MUST pull the money-structure before ruling it a tail. **A false kill = a hole in the framework.**
- **Smart money is a reference, not a verdict**: flow only confirms/falsifies your directional call; even a strong signal still gets a stop.
- **Two independent gates**: stock fish-segment ∩ warrant economics (IV not extreme / expiry ≥3mo / street-ratio <50% / acceptable spread).
- **Act when confirmed, put the hesitation on the exit side**; puts get tighter stops & smaller size.
- **The matrix decides the output** — don't force call/put symmetry; if one side is all tails, stay flat and wait for a trigger.

## ⚠️ Disclaimer

This skill is a research/analysis tool only and is **NOT investment advice**. HK warrants and CBBCs are **high-leverage instruments that can go to zero or be mandatorily called**. Any trade decision based on this skill's output, and its outcome, is solely the user's responsibility. **Strongly validate the framework on a paper account before considering real money.** The authors accept no liability for any losses.

## 📄 License

MIT — see [LICENSE](LICENSE).
