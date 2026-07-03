# 示例 · 模拟盘收益台账（2026-06-17 → 07-02）：一张表看清「动作 · 理由 · 认知迭代 · 收益」

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘（Longbridge paper account）方法论演示，非业绩、非收益承诺、非投资建议。**
> 这不是"晒收益"，而是把一段完整的模拟盘轨迹摊开：**每一笔动作对应什么决策理由、验证或推翻了框架的哪一条、最后落到多少盈亏**。截止 2026-07-02（上一交易日；07-01 港股休市），账户已实现 **−4,145 HKD**、当前空仓。
> 单笔买入 ≤ 1 万 HKD；账户纯现金 HKD/USD/SGD 各约 100 万。逐日底稿见项目内 `knowledge/paper-trading/2026-warrant-tracker.md`。

---

## 1. 总台账 —— 动作 × 理由 × 收益（一张表）

> 收益列＝该笔平仓的**已实现**盈亏（HKD）；累计列＝当日收盘的累计已实现。持有中不计已实现。

| 日期 | 动作 | 标的（方向） | 成交 价×量 | 决策理由（一句） | 触发/验证的框架 | 已实现 | 累计已实现 |
|---|---|---|---|---|---|---|---|
| 06-17 | 武装未触发 | 比亚迪27631（认沽） | — | 汽车负 alpha、破 81 顺势；价位未破 → **纪律未入场** | L1 只在扳机确认后动 | 0 | 0 |
| 06-24 | 建仓 ×3 | 中芯23604 / 中芯61345（认购） | 0.305×25k / 0.177×55k | 半导体领涨、中芯大单 **+26 亿吸筹**、突破中段、CLV 0.814 吸筹档 → 鱼中 | v1 鱼身定位（L2 四维对齐） | 持有 | 0 |
| 06-24 | 建仓 | 药明22574（认购） | 0.055×90k | CXO 轮动、价外证凸性、鱼头小试限亏 | v1 鱼头轻仓 | 持有 | 0 |
| 06-25 | 误单即撤 | 小米28168（认沽） | 0.435×12k 买→卖 | Agent 未经确认自动下单 → 按指示反向平仓 | **round-trip 持平、0 盈亏**；纪律修正 | 0 | 0 |
| 06-26 | 全平 ×2 | 中芯23604 / 中芯61345 | 0.265×25k / 0.132×55k | 结构 06-25 已翻转本该走，却拖用价格尺 78 → 崩 | **v3 触发（离场纪律）** | **−1,000 / −2,475** | −3,475 |
| 06-29 | 减半止盈 | 药明22574（认购） | 0.074×45k | 触及 35 减半纪律；CLV≈1.0 吸筹 → 留半跟 | v3.1 结构轮询上线 | **+855** | −2,620 |
| 06-29 | 建仓 | 阿里14190（认购） | 0.110×50k | 价格站稳 94 + 大单实时流入 = 扳机；CLV 0.882 软佐证 | **R7：CLV 永不作进场扳机** | 持有 | −2,620 |
| 06-30 | 翻转确认·全平 | 阿里14190 | 0.101×50k | 大单 −5,921万派发 + flow 自峰回吐 89% = 双证齐 | **v3.1 结构尺首验（先于价格离场）** | **−450** | −3,070 |
| 06-30 | 鱼中·重建仓 | 中芯23604（认购） | 0.380×25k | 自纠「按 +14.6% 涨幅错杀」→ CLV 0.833 吸筹 + 大单 +6.81 亿 + 突破新高 | **L3 反错杀（不按涨跌幅枪毙）** | 持有 | −3,070 |
| 06-30 | 全平剩半 | 药明22574 | 0.070×45k | 鱼头 day5 未升鱼中、质量走软 → 用户指示落袋 | 鱼头周期纪律 | **+675** | −2,395 |
| 07-02 | 翻转确认·全平 | 中芯23604 | 0.310×25k | 假期跳空破价格止损 → **选 A 尊重结构尺**留仓；11:09 大单 −2.4 亿 + flow 回吐 67% 双证齐才走 | **v3.1 结构尺（两头都对）** | **−1,750** | **−4,145** |
| 07-03 | 仅条件单 | 紫金2899（认购13494） | 未成交 | 黄金 melt-up、多月底部放量反转、CLV 0.924 吸筹；flow 自峰回撤 48% → 回踩限价不追 | 证伪预算 / 不追尖顶 | 0 | −4,145 |

---

## 2. 认知迭代轴 —— 每一版框架都是一笔亏损"买"来的

> 台账的真正产出不是收益数字，而是**规则**。这段 −4,145 HKD 是 v1→v3.1 的"学费"，每一版都由一次被真实数据反转的判断触发。

- **v1（06-24）· 鱼身定位基线** — L0–L9：先定方向、只重仓鱼中、鱼尾按结构+速度判（绝不按涨跌幅）、主力=参考非裁决、两道独立闸（股票鱼身 ∩ 权证经济性）。中芯/药明首次建仓即依此。
- **v2（06-25）· 结构量化** — 给"大幅派发/吸筹"一把**可回测的刻度**：CLV 120 日自我分位（主）+ 实时大单（辅）。触发自「大单只有当日、无历史，肉眼判派发会漂」。当日金属簇 −6~8% 全被"大单接刀=吸筹"否决，**防错杀样板**。
- **v3（06-26）· 离场纪律【最贵的一课，−3,475】** — 中芯两腿进场完全合规（+26 亿吸筹），却从 +7.64% 回吐成亏损。根因不在进场、在离场：**进场用结构尺、离场却偷换成价格尺 78**。落库 R6（判据一致性）+ L5.1（离场端结构再筛）+ L8（止损按杠杆折算，不抄正股技术位）。
- **v3.1（06-29）· 结构监控落地** — v3 说"结构翻转就走"，但结构（大单/flow）长桥**无 push、无历史**，手判必迟到。本版把判断落成**双轨轮询 + `structure_flip` 机器判据**（大单方向 + flow 斜率/回撤双证）。非判断被反转 → 点版本。
- **应用与活验证（06-30 / 07-02）** — 结构尺三个方向都经住实战：
  1. **该走时走**：阿里双证翻转 → 平在 −8.2%，早于价格止损（等价格约 −12~15%）。
  2. **该忍时忍**：06-30 中芯午后回撤、07-02 假期跳空 −26%，但大单仍接刀吸筹 → 判"持有"未误杀，07-02 最终等到真派发（−2.4 亿）才走，**平在 −18.4% 而非早盘恐慌价 −26%**。
  3. **噪声不乱动**：药明单证连续 3 轮预警不升级 → 不动。

> **一句话认知**：进场端的功课（v1/v2）只决定"不选错股"；真正吃掉/守住收益的是**离场端**（v3/v3.1）。中芯两次进场都合规，第一次因离场偷换尺子亏 −3,475，第二次用结构尺把跳空 −26% 的错杀救回到 −18.4%。**同一个进场逻辑，离场纪律不同，结果天差地别。**

---

## 3. 收益汇总（截止 2026-07-02 收盘）

**按腿明细（已平仓）**

| 标的（方向） | 成本 HKD | 已实现 HKD | 单腿收益率 | 备注 |
|---|---|---|---|---|
| 中芯61345 牛证（认购） | 9,735 | **−2,475** | −25.4% | 06-26 离场偷换尺，杠杆放大 |
| 中芯23604 平价证（认购·首次） | 7,625 | **−1,000** | −13.1% | 同上 |
| 小米28168（认沽·误单） | 5,220 | **0** | 0% | 买入即撤平，round-trip |
| 药明22574（认购·整腿） | 4,950 | **+1,530** | **+30.9%** | 减半+855 / 剩半+675，价外凸性兑现 |
| 阿里14190（认购） | 5,500 | **−450** | −8.2% | 结构尺先于价格离场 |
| 中芯23604（认购·重建） | 9,500 | **−1,750** | −18.4% | 07-02 结构翻转确认离场 |

**合计**

- **累计已实现：**`−2,475 −1,000 +0 +1,530 −450 −1,750 =` **−4,145 HKD**
- **动用并已平仓成本基数**（不含小米 round-trip）：**37,310 HKD** → 实现收益率 ≈ **−11.1%**
  （口径说明：分母为已平仓各腿成本之和，非账户总额；账户为约 100 万纯现金，故对账户的影响约 −0.4%。避免用杠杆品种的腿级 % 冒充账户收益。）
- **当前持仓：**空仓（中芯 07-02 已离场）。07-03 仅紫金 1 条认购条件单（回踩限价，未成交）。

**累计曲线（收盘）**：06-24 `0` → 06-25 `0` → 06-26 `−3,475` → 06-29 `−2,620` → 06-30 `−2,395` → 07-02 `−4,145`。

---

## 4. 一句话总结

> 这张台账想证明的不是"赚了多少"，而是**框架在真金白银（模拟）的反馈里怎么进化**：−4,145 HKD 里，中芯两次进场同样合规，差别全在离场——第一次（−3,475）催生了 v3 离场纪律，第二次（−1,750）用 v3.1 结构尺把假期跳空的 −26% 错杀救成 −18.4% 的纪律离场。**亏损被转成了规则；这才是台账的产出。**

---
<a id="english"></a>
# Example · Paper-trading ledger (2026-06-17 → 07-02): one table for "action · rationale · cognitive iteration · P&L"

> **🌐 [中文 ↑](#示例--模拟盘收益台账2026-06-17--07-02一张表看清动作--理由--认知迭代--收益) · English (current)**
>
> ⚠️ **A paper-account (Longbridge) methodology walkthrough — NOT a track record, NOT a return promise, NOT investment advice.**
> This is not a "look at my gains" post; it lays a full paper-trading trajectory flat so you can see **which decision rationale each fill came from, which framework rule it validated or broke, and what P&L it landed at**. Through 2026-07-02 (the prior session; HK was closed 07-01), realized P&L is **−HK$4,145**, account currently flat.
> Per-trade buys ≤ HK$10k; account is all-cash, ~1M each of HKD/USD/SGD. Day-by-day source: `knowledge/paper-trading/2026-warrant-tracker.md` (project-side).

---

## 1. Master ledger — action × rationale × P&L (one table)

> "Realized" = realized P&L (HKD) of that close; "Cumulative" = running realized at that day's close. Open positions are not counted as realized.

| Date | Action | Name (dir.) | Fill Px×Qty | Rationale (one line) | Framework triggered/validated | Realized | Cumulative |
|---|---|---|---|---|---|---|---|
| 06-17 | Armed, untriggered | BYD 27631 (put) | — | Auto negative-alpha, break 81 momentum; price never broke → **discipline: no entry** | L1 act only on confirmed trigger | 0 | 0 |
| 06-24 | Open ×2 | SMIC 23604 / 61345 (call) | 0.305×25k / 0.177×55k | Semis lead, SMIC large money **+HK$2.6bn** accumulating, mid-body breakout, CLV 0.814 | v1 fish-body (L2 four-way align) | held | 0 |
| 06-24 | Open | WuXi 22574 (call) | 0.055×90k | CXO rotation, OTM convexity, head-leg small size | v1 head = light size | held | 0 |
| 06-25 | Mis-order, reversed | Xiaomi 28168 (put) | 0.435×12k buy→sell | Agent auto-fired without confirm → reversed per instruction | **round-trip flat, 0 P&L**; discipline fix | 0 | 0 |
| 06-26 | Full exit ×2 | SMIC 23604 / 61345 | 0.265×25k / 0.132×55k | Structure flipped 06-25, should've cut, but waited for price stop 78 → crashed | **v3 triggered (exit discipline)** | **−1,000 / −2,475** | −3,475 |
| 06-29 | Trim (target) | WuXi 22574 (call) | 0.074×45k | Hit 35 trim rule; CLV≈1.0 accumulating → keep half | v3.1 structure poll live | **+855** | −2,620 |
| 06-29 | Open | BABA 14190 (call) | 0.110×50k | Price holds 94 + live large-money inflow = trigger; CLV 0.882 soft support | **R7: CLV is never an entry trigger** | held | −2,620 |
| 06-30 | Confirmed flip · full exit | BABA 14190 | 0.101×50k | Large money −HK$59.2m distributing + flow gives back 89% = two proofs | **v3.1 ruler's first live test (exit ahead of price)** | **−450** | −3,070 |
| 06-30 | Mid-body · re-open | SMIC 23604 (call) | 0.380×25k | Self-corrected an "up +14.6% → kill" error → CLV 0.833 + large money +HK$6.81bn + new-high breakout | **L3 anti-false-kill (never by magnitude)** | held | −3,070 |
| 06-30 | Exit last half | WuXi 22574 | 0.070×45k | Head leg failed to upgrade by day 5, quality softening → user-directed close | head-leg horizon discipline | **+675** | −2,395 |
| 07-02 | Confirmed flip · full exit | SMIC 23604 | 0.310×25k | Holiday gap broke the price stop → **chose (A) respect the structure ruler**, held; exited only when 11:09 large money −HK$240m + flow −67% two proofs | **v3.1 ruler (right in both directions)** | **−1,750** | **−4,145** |
| 07-03 | Conditional only | Zijin 2899 (call 13494) | unfilled | Gold melt-up, multi-month base breakout on volume, CLV 0.924; flow −48% from peak → limit-on-pullback, no chase | falsification budget / no chasing tops | 0 | −4,145 |

---

## 2. Cognitive-iteration axis — every framework version was bought with a loss

> The real output of a ledger is not the P&L number, it's the **rules**. This −HK$4,145 is the "tuition" for v1→v3.1; each version was triggered by a call reversed by real data.

- **v1 (06-24) · fish-body baseline** — L0–L9: direction first, size only the mid-body, judge the tail by structure + speed (never by magnitude), smart-money = reference not verdict, two independent gates (stock fish-body ∩ warrant economics). SMIC/WuXi's first entries followed this.
- **v2 (06-25) · structure quantified** — gives "heavy distribution/accumulation" a **backtestable scale**: CLV-120d self-percentile (primary) + real-time large money (secondary). Triggered by "large money is same-day, no history — eyeballing distribution drifts." That day the metals cluster (−6~8%) was all vetoed as "large money catching the knife = accumulation" — the **anti-false-kill template**.
- **v3 (06-26) · exit discipline [the most expensive lesson, −3,475]** — SMIC two legs entered fully compliant (+HK$2.6bn accumulating) yet bled from +7.64% into a loss. Root cause was the exit, not the entry: **entered on the structure ruler, then swapped to a price ruler (78) to exit.** Codified R6 (criterion consistency) + L5.1 (exit-side structure re-screen) + L8 (size the stop by leverage, don't copy the stock level).
- **v3.1 (06-29) · structure monitoring operationalized** — v3 said "flip in structure = exit", but structure (large money / flow) has **no push and no history** on Longbridge, so hand-judgment always lags. This release turned the judgment into a **two-track poll + `structure_flip` machine criterion** (large-money direction + flow slope/drawdown, two proofs). No call reversed → a point release.
- **Application & live validation (06-30 / 07-02)** — the ruler passed live in all three directions:
  1. **Cut when you should**: BABA two-proof flip → exit at −8.2%, ahead of the price stop (waiting would've meant ~−12~15%).
  2. **Hold when you should**: SMIC's 06-30 afternoon pullback and 07-02 holiday gap (−26%) — but large money still catching the knife → verdict HOLD, no false-kill; on 07-02 it finally waited for true distribution (−HK$240m) before cutting, **exiting at −18.4% instead of the −26% panic price.**
  3. **Stay put on noise**: WuXi's three consecutive one-proof warnings never escalated → no action.

> **One-line takeaway**: the entry homework (v1/v2) only decides "don't pick the wrong stock"; what actually loses or saves the P&L is the **exit** (v3/v3.1). Both SMIC entries were compliant; the first lost −3,475 by swapping rulers at the exit, the second used the structure ruler to rescue a −26% gap-false-kill down to a −18.4% disciplined exit. **Same entry logic, different exit discipline, worlds apart.**

---

## 3. P&L summary (through 2026-07-02 close)

**By leg (closed)**

| Name (dir.) | Cost HKD | Realized HKD | Leg return | Note |
|---|---|---|---|---|
| SMIC 61345 CBBC (call) | 9,735 | **−2,475** | −25.4% | 06-26 exit swapped ruler, leverage-amplified |
| SMIC 23604 ATM (call, 1st) | 7,625 | **−1,000** | −13.1% | ditto |
| Xiaomi 28168 (put, mis-order) | 5,220 | **0** | 0% | bought then reversed, round-trip |
| WuXi 22574 (call, full leg) | 4,950 | **+1,530** | **+30.9%** | trim +855 / rest +675, OTM convexity paid |
| BABA 14190 (call) | 5,500 | **−450** | −8.2% | structure ruler, exit ahead of price |
| SMIC 23604 (call, re-open) | 9,500 | **−1,750** | −18.4% | 07-02 confirmed-flip exit |

**Totals**

- **Cumulative realized:** `−2,475 −1,000 +0 +1,530 −450 −1,750 =` **−HK$4,145**
- **Deployed-and-closed cost base** (excl. Xiaomi round-trip): **HK$37,310** → realized return ≈ **−11.1%**
  (Basis note: denominator is the sum of closed-leg costs, NOT the account. The account is ~1M all-cash, so the account impact is ~−0.4%. Leg-level % on a leveraged instrument must not be passed off as an account return.)
- **Current holdings:** flat (SMIC exited 07-02). 07-03 has only one Zijin call conditional order (limit-on-pullback, unfilled).

**Cumulative curve (close):** 06-24 `0` → 06-25 `0` → 06-26 `−3,475` → 06-29 `−2,620` → 06-30 `−2,395` → 07-02 `−4,145`.

---

## 4. One line

> What this ledger tries to prove is not "how much was made", but **how the framework evolves under real (paper) money feedback**: within −HK$4,145, both SMIC entries were equally compliant — the difference was entirely at the exit. The first (−3,475) gave birth to the v3 exit discipline; the second (−1,750) used the v3.1 structure ruler to turn a −26% holiday-gap false-kill into a −18.4% disciplined exit. **Losses were converted into rules — that is the ledger's true output.**
