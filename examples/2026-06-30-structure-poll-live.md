# 示例 · 2026-06-30 结构轮询首次实战：阿里翻转离场 + 中芯回撤不误杀

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘方法论演示，非业绩、非收益承诺。**
> 承接 [`2026-06-29-meltup-structure-poll.md`](2026-06-29-meltup-structure-poll.md)：v3.1 把"离场看结构"落成了 30 分钟轮询 + 检测器。**今天是该系统的首次 live 验证——而且双向都对：该走时走、该忍时忍。**

## 1. 盘面：一日从沸点到冰点
- 昨日 melt-up（情绪83）→ 今日 **risk-off（情绪19、温度42）**：HSI −1.2%、HSCEI −1.3%，但 **HSTECH +0.9%**（科技撑、大盘杀=窄市背离，06-24 同款见顶前兆）。
- 半导体逆势强（中芯 +5.4%、智谱 +9.9%、CPO 链），是窄市里的 idiosyncratic alpha。

## 2. 结构轮询系统的三个 live 结果（核心）

**① 该走时走（v3 留的洞，今天补上）—— 阿里 09:38 预警 → 10:08 翻转确认 → 离场**
- 09:38（开盘8分钟）：大单 −1,293万派发 + capital_flow 仍升 = 单证 → **翻转预警，不动**（早盘噪声不误杀）。
- 10:08：大单 **−5,921万**派发加深 + capital_flow **自峰回吐 89%**、斜率 −540 = **双证齐 → 翻转确认**。
- 按 L5.1 结构尺一级离场：**阿里 92.7 仍在 91 价格止损上方就先走**，平在 **−8.2%**（−450 HKD）。若按老办法等价格破 91，窝轮约 −12~15%。**这正是 06-26 中芯"结构翻了却等价格、拖到崩"那个洞——今天用检测器堵上了。**

**② 该忍时忍（不误杀）—— 中芯午后回撤**
- 14:43：中芯从高点 91.55 回撤到 88.30、窝轮跌破成本，capital_flow 自峰回 −4%。看着像要翻。
- 但检测器：**大单仍 +16.5亿吸筹、散户转净出（结构更纯）、flow 没恶化** → 判 **持有**，不杀。
- 结果：中芯收回 89.40（+5.4%）、窝轮回到成本。**高位回撤 ≠ 派发，结构尺正确区分、避免了恐慌离场。**

**③ 单证不升级就不动 —— 药明连续 3 轮翻转预警**
- 11:08/11:38/11:52：大单轻微派发（−320~−608万）+散户进，但 capital_flow 始终净流入未恶化 = 单证 → 全程"翻转预警、不动仓"。检测器没在轻微噪声上误杀。（后因鱼头 10 日窗口 day5 迟迟不升鱼中，**用户手动离场**落袋。）

## 3. 另一课：L3「按幅度错杀」自纠
- 10:30 全局首跑时，我**按"中芯 +14.6%/5日 extended"把它枪毙了**——犯框架 L3 红线。
- 重做阶段2：CLV 120日 **0.833/第87分位=吸筹档** + 大单 **+6.81亿**吸筹 + 突破新高非鱼尾 → **认购健康鱼中**。距历史价位无关，"涨多"不是枪毙理由。
- 改正后建仓（19823 单手破1万护栏 → 换 23604 单手9,500≤1万）。

## 4. 成交与收益（模拟盘）
| 标的 | 动作 | 价×量 | 实现 HKD |
|---|---|---|---|
| 阿里14190 | 翻转确认·全平 | 0.101×50,000 | **−450（−8.2%）** |
| 药明22574 | 用户指示·全平剩半 | 0.070×45,000 | **+675（+27.3%）** |
| 中芯23604 | 鱼中·建仓 | 0.380×25,000 | 持有(收平成本) |

- **药明整腿收官**：06-29减半 +855 + 06-30全平 +675 = **+1,530**（成本4,950、+30.9%）。
- **累计已实现：−2,395**（中芯两腿06-26 −3,475 + 药明 +1,530 + 阿里 −450）。
- 持仓过夜：中芯23604（认购鱼中，收89.40/+5.4%、大单+17.87亿吸筹、CLV收盘0.705、窝轮平成本）。

→ **一句话：v3.1 不是纸上规则——今天它真的"该走时走（阿里，先于价格）、该忍时忍（中芯回撤）、噪声不乱动（药明预警）"。结构尺三个方向都经住了实战。**

---
<a id="english"></a>
# Example · 2026-06-30 Structure-poll goes live: BABA flip-exit + SMIC dip not false-killed

> **🌐 [中文 ↑](#示例--2026-06-30-结构轮询首次实战阿里翻转离场--中芯回撤不误杀) · English (current)**
>
> ⚠️ **A paper-account methodology walkthrough — NOT a track record.**
> Continues [`2026-06-29-meltup-structure-poll.md`](2026-06-29-meltup-structure-poll.md): v3.1 turned "exit on structure" into a 30-min poll + detector. **Today is its first live test — and it was right in both directions: cut when it should, hold when it should.**

## 1. Tape: freezing to boiling to freezing in a day
- Yesterday melt-up (sentiment 83) → today **risk-off (sentiment 19, temp 42)**: HSI −1.2%, HSCEI −1.3%, but **HSTECH +0.9%** (tech holds, broad market sells = narrow-market divergence, the same pre-top pattern as 06-24).
- Semis bucked the tape (SMIC +5.4%, AI names), the idiosyncratic alpha in a narrow market.

## 2. Three live results of the structure-poll system (core)

**① Cut when you should (the gap v3 left, now closed) — BABA: warn 09:38 → confirm 10:08 → exit**
- 09:38 (8 min in): large money −HK$12.9m distributing + capital_flow still rising = one proof → **early warning, no action** (don't false-kill opening noise).
- 10:08: large money **−HK$59.2m** distribution deepens + capital_flow **gives back 89% from peak**, slope −540 = **two proofs → confirmed flip**.
- Per L5.1 structure ruler: **exit at 92.7 while price is still above the 91 stop**, realized **−8.2%** (−HK$450). Waiting for the price break at 91 would've meant ~−12~15% on the warrant. **This is exactly the 06-26 SMIC hole ("structure flipped but waited for price, then it crashed") — closed today by the detector.**

**② Hold when you should (no false-kill) — SMIC afternoon pullback**
- 14:43: SMIC pulls back from 91.55 high to 88.30, warrant dips below cost, capital_flow −4% from peak. Looks like it might flip.
- But the detector: **large money still +HK$1.65bn accumulating, retail flips to net-out (cleaner structure), flow not deteriorating** → verdict **HOLD**, no cut.
- Result: SMIC recovers to 89.40 (+5.4%), warrant back to cost. **A high-level pullback ≠ distribution; the ruler told them apart and avoided a panic exit.**

**③ One proof, no escalation → no action — WuXi: 3 consecutive flip-warnings**
- 11:08/11:38/11:52: mild large-money distribution (−HK$3~6m) + retail in, but capital_flow stayed net-inflow / not deteriorating = one proof → "warn, don't act" throughout. No false-kill on mild noise. (Later closed **manually by the user** as the head leg failed to upgrade by day 5 of its 10-day window.)

## 3. Side lesson: self-correcting an L3 "false-kill by magnitude"
- On the first 10:30 global pass, I **dismissed SMIC by "+14.6%/5d, extended"** — an L3 red-line error.
- Redoing stage-2: CLV-120d **0.833 / 87th pct = accumulation band** + large money **+HK$6.81bn** + breakout to new highs (not a tail) → **a healthy call mid-body**. Distance from historical price is irrelevant; "up a lot" is not a kill reason.
- Re-entered after correction (19823 breaks the ≤HK$10k guardrail at 1 lot → switched to 23604 at HK$9,500/lot).

## 4. Fills & P&L (paper)
| Name | Action | Px × Qty | Realized HKD |
|---|---|---|---|
| BABA 14190 | confirmed-flip · full exit | 0.101×50,000 | **−450 (−8.2%)** |
| WuXi 22574 | user-directed · exit last half | 0.070×45,000 | **+675 (+27.3%)** |
| SMIC 23604 | mid-body · open | 0.380×25,000 | held (closed at cost) |

- **WuXi leg closed out**: 06-29 trim +855 + 06-30 exit +675 = **+1,530** (cost 4,950, +30.9%).
- **Cumulative realized: −2,395** (SMIC two legs 06-26 −3,475 + WuXi +1,530 + BABA −450).
- Held overnight: SMIC 23604 (call mid-body, stock closed 89.40/+5.4%, large money +HK$1.787bn accumulating, closing CLV 0.705, warrant flat at cost).

→ **One line: v3.1 isn't a paper rule — today it actually cut when it should (BABA, ahead of price), held when it should (SMIC pullback), and stayed put on noise (WuXi warnings). The structure ruler passed live in all three directions.**
