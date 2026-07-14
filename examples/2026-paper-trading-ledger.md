# 示例 · 模拟盘收益台账（2026-06-17 → 07-14）：从「少量手动单」到「7×24 守护进程自动执行」

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘（Longbridge paper account）方法论演示，非业绩、非收益承诺、非投资建议。**
> 这不是"晒收益"，而是把一段完整的模拟盘轨迹摊开：**每一笔动作对应什么决策理由、验证或推翻了框架的哪一条、最后落到多少盈亏**。
>
> 这段轨迹跨两个时代：
> - **手动时代（06-17 → 07-02）**：少量手动单，靠人开会话触发；核心课题是**离场纪律**。已实现（不计费口径）**−4,145 HKD**。
> - **守护进程时代（07-03 → 07-14）**：`hk-market-daemon` 上线，快层 watcher 秒级盯盘 + 自动执行。交易频率从 2 周 6 条腿暴涨到 8 个交易日 **33 个平仓回合**。这一段第一次把**手续费拖累、想平却平不掉、台账 vs 真实的落差**这些"规模化才暴露的账"摊到台面上。
>
> **截止 2026-07-14 收盘：**
> - **不计费同口径累计已实现 ≈ −5,737.50 HKD**（手动 −4,145 ＋ 守护进程毛 −1,592.50）。
> - 守护进程时代若按 **SDK 真实成交+扣费**口径：已平 33 回合 **净 −2,723.41 HKD**（毛 −1,592.50 − 手续费 1,130.91），**胜率 36.4%**。
> - 当前仍持有 **10 个标的**，合计浮盈 **+1,032.50 HKD**（未计入已实现）。
>
> 单笔买入 ≤ 1 万 HKD；账户纯现金 HKD/USD/SGD 各约 100 万。逐日底稿见项目内 `knowledge/paper-trading/2026-warrant-tracker.md`；守护进程时代盈亏以 `reconcile_pnl.js`（SDK 真实成交+费用）为准，本地台账（提交限价、不计费）偏乐观、仅作底稿。

---

## 1. 手动时代 —— 动作 × 理由 × 收益（06-17 → 07-02）

> 收益列＝该笔平仓的**已实现**盈亏（HKD，不计费）；累计列＝当日收盘的累计已实现。持有中不计已实现。

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

**手动时代小结**：6 条已平仓腿，动用并已平仓成本 37,310 HKD，已实现 **−4,145 HKD**（≈ −11.1%）。核心结论——**进场端功课只决定"不选错股"；真正吃掉/守住收益的是离场端**（详见 §3）。

---

## 2. 守护进程时代 —— 规模化之后（07-03 → 07-14）

> 07-03 起 `hk-market-daemon` 上线：快层 watcher 秒级盯持仓价 + 高频轮询大单结构，命中止损/止盈/结构翻转即自动执行。人从"每笔点确认"退到"定规则、盯异常"。**交易量级从此不是一个数量级**——8 个交易日 33 个平仓回合。
>
> 本段口径以 **SDK 真实成交价 + 真实手续费**为准（一「笔」= 平仓回合 0→持仓→0）。

**单标的汇总（按净盈亏降序，SDK 口径）**

| 标的（方向） | 回合 | 净盈亏 HKD | 毛盈亏 | 手续费 | 胜/负 | 胜率 | 累计收益率 | 均持 |
|---|---|---|---|---|---|---|---|---|
| 小米14225（认购） | 5 | **+3,195.19** | +3,400.00 | 204.81 | 2/3 | 40% | +6.5% | 1d3h |
| 中兴13599（认购） | 3 | **+1,482.70** | +1,560.00 | 77.30 | 2/1 | 67% | +5.2% | 1d7h |
| 中芯23604（认购） | 1 | **+662.59** | +700.00 | 37.41 | 1/0 | 100% | +12.7% | 1h51m |
| 29665（认购） | 1 | +500.79 | +520.00 | 19.21 | 1/0 | 100% | +5.2% | 22h |
| 29053（认购） | 1 | +456.60 | +495.00 | 38.40 | 1/0 | 100% | +5.0% | 3h53m |
| 15159（认购） | 1 | 0.00 | 0.00 | 0.00 | 0/1 | 0% | 0.0% | 1h58m |
| 29060（认购） | 1 | −37.40 | 0.00 | 37.40 | 0/1 | 0% | −0.7% | 18h |
| 13779（认购） | 2 | −56.33 | +95.00 | 151.33 | 1/1 | 50% | −0.2% | 1d15h |
| 紫金13494（认购） | 2 | −174.41 | −100.00 | 74.41 | 1/1 | 50% | −1.8% | 4d23h |
| 28701（认购） | 3 | −444.12 | −370.00 | 74.12 | 1/2 | 33% | −3.1% | 1d8h |
| 29344（认购） | 5 | −798.60 | −630.00 | 168.60 | 1/4 | 20% | −2.8% | 1d0h |
| 美团26790（认购） | 2 | **−2,095.03** | −1,980.00 | 115.03 | 0/2 | 0% | −7.3% | 3d1h |
| 药明22574（认购） | 2 | **−2,275.73** | −2,237.50 | 38.23 | 0/2 | 0% | −11.5% | 1d13h |
| 快手27786（认购） | 4 | **−3,139.66** | −3,045.00 | 94.66 | 1/3 | 25% | −7.0% | 1d17h |
| **合计（已平 33 回合）** | 33 | **−2,723.41** | **−1,592.50** | **1,130.91** | 12/21 | **36.4%** | | 均持 1d12h |

> 标的为**窝轮/牛熊证代码**，同一正股不同期可能换证。名称为对应正股（小米=1810、中兴=763、中芯、药明、美团、阿里=9988、紫金）。

**当前仍持有（未实现，不计入已实现）**：10 个标的，合计浮盈 **+1,032.50 HKD**（阿里14461 +600 / 15159 +390 / 快手27786 +240 / 29344 +120 …）。

**四笔代表性回合**

- **小米14225（+2,960.99 / +29.6%，持 3d4h）**——本时代最大单回合盈利。07-10 吸筹进场、结构一直站住，鱼身兑现。全期 5 回合累计 +3,195，是把整段拉回来的主力。
- **中兴13599（+1,261.12 / +13.1%，持 3d20h）**——吸筹进场、拿住到位；67% 胜率是本时代最稳的一档。
- **快手27786（−3,139 合计，含单回合 −1,993 / −20.6% 与 −1,857 / −32.5%）**——反复以"吸筹/普涨"进场却持续失血，本时代最大亏损源。**规模化把一个偏差信号的代价放大了。**
- **药明22574（−2,275 / −11.5%）**——07-13 结构翻转该走，**平仓卖单被券商拒单**（挂 0.088 未成交），没走成、拖到更深。这就是下面 §3 的"想平却平不掉"。

---

## 3. 认知迭代轴 —— 每一版框架/每一条运维教训，都是一笔亏损"买"来的

> 台账的真正产出不是收益数字，而是**规则**。手动时代的 −4,145 是 v1→v3.1 的"学费"；守护进程时代的 −2,723（净）又"买"回三条**只有规模化、自动化才暴露**的账。

**手动时代：判据框架 v1 → v3.1**

- **v1（06-24）· 鱼身定位基线** — 先定方向、只重仓鱼中、鱼尾按结构+速度判（绝不按涨跌幅）、主力=参考非裁决、两道独立闸（股票鱼身 ∩ 权证经济性）。
- **v2（06-25）· 结构量化** — 给"大幅派发/吸筹"一把可回测的刻度：CLV 120 日自我分位（主）+ 实时大单（辅）。当日金属簇 −6~8% 全被"大单接刀=吸筹"否决，**防错杀样板**。
- **v3（06-26）· 离场纪律【最贵的一课，−3,475】** — 中芯两腿进场完全合规（+26 亿吸筹），却从 +7.64% 回吐成亏损。根因在离场：**进场用结构尺、离场却偷换成价格尺 78**。落库 R6 + L5.1 + L8。
- **v3.1（06-29）· 结构监控落地** — 把"结构翻转就走"落成**双轨轮询 + `structure_flip` 机器判据**（大单方向 + flow 斜率/回撤双证）。这一版正是**守护进程能自动执行的前提**。

**守护进程时代：三条"规模化才暴露"的运维账**

- **① 手续费拖累【本时代新增的一级成本】** — 手动时代 2 周才 6 条腿，费用几乎不可见；自动化 8 天跑 33 个回合，**手续费 1,130.91 HKD**把毛 −1,592.50 直接砸成净 **−2,723.41**。高频下，**费用是必须计进判据的一级线目**，不能再当"忽略不计"。
- **② 想平却平不掉【自动化独有的执行层风险】** — 本时代 **10 笔平仓卖单失败**（拒单 / 限价挂到收盘未成交）：快手27786、药明22574、美团26790、阿里14461 都出现"结构尺喊走、单子没成交、仓位卡住"。人手点单会立刻改价重挂，自动系统却可能一路挂到收盘。结构判对了，**执行层没兜住，照样多亏**。（executor 已加失败回滚+冷却重挂，但这是本时代最该补的下一课。）
- **③ 台账 vs 真实的落差【度量本身要对账】** — 本地台账（提交限价、不计费）显示毛 **+806.15**，SDK 真实净额只有 **+88.38**（含未平已卖部分）/ 已平回合 **−2,723.41**。"看着赚、实际亏"正是本时代催生 `reconcile_pnl.js` 的原因：**面板/台账不能直接当业绩，必须用 SDK 成交回算。**

> **一句话认知（跨两个时代）**：手动时代证明了"离场纪律 > 进场功课"；守护进程时代补上后半句——**当你把纪律交给机器 7×24 执行，胜负手从"判断对不对"又多了两层："费用扛不扛得住"和"单子平不平得掉"。** 36.4% 胜率能靠小米/中兴几个大回合把毛亏压到 −1,592，却被手续费和卡单拖成净 −2,723。**规模化不改判据的对错，只放大判据之外的每一个漏洞。**

---

## 4. 收益汇总（截止 2026-07-14 收盘）

**两个时代 · 两个口径**

| 时代 | 口径 | 已实现 | 关键指标 |
|---|---|---|---|
| 手动（06-17→07-02） | 不计费·成交价×量 | **−4,145 HKD** | 6 腿，成本基数 37,310，≈ −11.1% |
| 守护进程（07-03→07-14） | 不计费·毛（同上口径可加） | **−1,592.50 HKD** | 33 回合 |
| 守护进程（07-03→07-14） | **SDK 真实·扣费净额** | **−2,723.41 HKD** | 胜率 36.4%，手续费 1,130.91 |

- **不计费同口径累计（两段可加）：** `−4,145 + (−1,592.50) =` **−5,737.50 HKD**
- **含费真实口径（守护进程段扣费）：** 手动段费用未被工具捕捉（早于对账脚本），故真实全口径**略差于 −5,737.50**；仅守护进程段单看即 **净 −2,723.41 HKD**。
- **当前持仓：** 10 个标的仍持有，合计浮盈 **+1,032.50 HKD**（未计入已实现；其中含 4 个"平仓失败卡住"的仓位）。

**累计曲线（收盘，不计费）**：
手动段 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`；
守护进程段（毛，叠加）07-02 `−4,145` → 07-14 `−5,737.50`。

> 逐笔明细见项目内 `data/paper-trading/`：`2026-warrant-tracker.md`（逐笔底稿）、`pnl-reconciliation-*.md`（SDK 对账）、`weekly-pnl-*.md`（周复盘）。

---

## 5. 一句话总结

> 这张台账想证明的不是"赚了多少"，而是**框架在真金白银（模拟）的反馈里怎么进化**——而且进化分两级：
> **手动时代**用 −4,145 买回了 v1→v3.1 的判据（结论：离场纪律 > 进场功课）；
> **守护进程时代**把这套纪律交给机器 7×24 跑，又用 −2,723（净）买回三条运维账：**手续费是一级成本、平仓单会平不掉、台账必须对账**。
> **规模化和自动化不改判据的对错，只把判据之外的每一个漏洞放大给你看。亏损被转成了规则——这才是台账的产出。**

---
<a id="english"></a>
# Example · Paper-trading ledger (2026-06-17 → 07-14): from "a few hand trades" to a "7×24 daemon"

> **🌐 [中文 ↑](#示例--模拟盘收益台账2026-06-17--07-14从少量手动单到724-守护进程自动执行) · English (current)**
>
> ⚠️ **A paper-account (Longbridge) methodology walkthrough — NOT a track record, NOT a return promise, NOT investment advice.**
> This is not a "look at my gains" post; it lays a full paper-trading trajectory flat so you can see **which decision rationale each fill came from, which framework rule it validated or broke, and what P&L it landed at**.
>
> The trajectory spans two eras:
> - **Hand era (06-17 → 07-02)**: a few manual trades, human-triggered per session; the core topic is **exit discipline**. Realized (no-fee basis) **−HK$4,145**.
> - **Daemon era (07-03 → 07-14)**: `hk-market-daemon` went live — a fast-layer watcher polling prices per-second + auto-execution. Trade frequency exploded from 6 legs in two weeks to **33 close-rounds in 8 sessions**. This era is the first to put **fee drag, "want-to-exit-but-can't", and the ledger-vs-reality gap** — the costs that only appear at scale — on the table.
>
> **Through 2026-07-14 close:**
> - **Same-basis (no-fee) cumulative realized ≈ −HK$5,737.50** (hand −4,145 + daemon gross −1,592.50).
> - Daemon era on the **SDK real-fill + fees** basis: 33 closed rounds, **net −HK$2,723.41** (gross −1,592.50 − fees 1,130.91), **win rate 36.4%**.
> - Currently still holding **10 names**, total floating **+HK$1,032.50** (not counted as realized).
>
> Per-trade buys ≤ HK$10k; account is all-cash, ~1M each of HKD/USD/SGD. Day-by-day source: `knowledge/paper-trading/2026-warrant-tracker.md`. Daemon-era P&L is per `reconcile_pnl.js` (SDK real fills + fees); the local ledger (submit-limit price, no fees) is optimistic and used only as source.

---

## 1. Hand era — action × rationale × P&L (06-17 → 07-02)

> "Realized" = realized P&L (HKD, no fees) of that close; "Cumulative" = running realized at that day's close. Open positions are not counted.

| Date | Action | Name (dir.) | Fill Px×Qty | Rationale (one line) | Framework | Realized | Cumulative |
|---|---|---|---|---|---|---|---|
| 06-17 | Armed, untriggered | BYD 27631 (put) | — | Auto negative-alpha, break 81; price never broke → **no entry** | L1 act only on confirmed trigger | 0 | 0 |
| 06-24 | Open ×2 | SMIC 23604 / 61345 (call) | 0.305×25k / 0.177×55k | Semis lead, SMIC **+HK$2.6bn** accumulating, mid-body breakout, CLV 0.814 | v1 fish-body (L2 align) | held | 0 |
| 06-24 | Open | WuXi 22574 (call) | 0.055×90k | CXO rotation, OTM convexity, head-leg light size | v1 head = light size | held | 0 |
| 06-25 | Mis-order, reversed | Xiaomi 28168 (put) | 0.435×12k buy→sell | Agent auto-fired without confirm → reversed | **round-trip flat, 0**; discipline fix | 0 | 0 |
| 06-26 | Full exit ×2 | SMIC 23604 / 61345 | 0.265×25k / 0.132×55k | Structure flipped 06-25, should've cut, waited for price stop 78 → crashed | **v3 (exit discipline)** | **−1,000 / −2,475** | −3,475 |
| 06-29 | Trim (target) | WuXi 22574 (call) | 0.074×45k | Hit 35 trim; CLV≈1.0 accumulating → keep half | v3.1 structure poll live | **+855** | −2,620 |
| 06-29 | Open | BABA 14190 (call) | 0.110×50k | Price holds 94 + live inflow = trigger; CLV 0.882 soft | **R7: CLV never an entry trigger** | held | −2,620 |
| 06-30 | Confirmed flip · exit | BABA 14190 | 0.101×50k | Large money −HK$59.2m distributing + flow gives back 89% | **v3.1 ruler's first live test** | **−450** | −3,070 |
| 06-30 | Mid-body · re-open | SMIC 23604 (call) | 0.380×25k | Self-corrected an "up +14.6% → kill" error → CLV 0.833 + +HK$6.81bn + new high | **L3 anti-false-kill** | held | −3,070 |
| 06-30 | Exit last half | WuXi 22574 | 0.070×45k | Head leg failed to upgrade by day 5 → user-directed close | head-leg horizon | **+675** | −2,395 |
| 07-02 | Confirmed flip · exit | SMIC 23604 | 0.310×25k | Holiday gap broke price stop → **chose (A) respect structure ruler**; exited only on 11:09 −HK$240m + flow −67% | **v3.1 ruler (right both ways)** | **−1,750** | **−4,145** |

**Hand era summary**: 6 closed legs, deployed-and-closed cost HK$37,310, realized **−HK$4,145** (≈ −11.1%). Core takeaway — **entry homework only decides "don't pick the wrong stock"; what actually loses/saves P&L is the exit** (see §3).

---

## 2. Daemon era — after going to scale (07-03 → 07-14)

> From 07-03 `hk-market-daemon` went live: a fast-layer watcher polls position prices per-second + polls large-money structure at high frequency, auto-executing on stop / target / structure-flip. The human stepped back from "click-confirm each trade" to "set rules, watch exceptions." Volume jumped an order of magnitude — 33 close-rounds in 8 sessions.
>
> This era is on the **SDK real-fill + real-fee** basis (a "round" = 0→position→0).

**By name (net P&L desc, SDK basis)**

| Name (dir.) | Rounds | Net HKD | Gross | Fees | W/L | Win% | Cum. return | Avg hold |
|---|---|---|---|---|---|---|---|---|
| Xiaomi 14225 (call) | 5 | **+3,195.19** | +3,400.00 | 204.81 | 2/3 | 40% | +6.5% | 1d3h |
| ZTE 13599 (call) | 3 | **+1,482.70** | +1,560.00 | 77.30 | 2/1 | 67% | +5.2% | 1d7h |
| SMIC 23604 (call) | 1 | **+662.59** | +700.00 | 37.41 | 1/0 | 100% | +12.7% | 1h51m |
| 29665 (call) | 1 | +500.79 | +520.00 | 19.21 | 1/0 | 100% | +5.2% | 22h |
| 29053 (call) | 1 | +456.60 | +495.00 | 38.40 | 1/0 | 100% | +5.0% | 3h53m |
| 15159 (call) | 1 | 0.00 | 0.00 | 0.00 | 0/1 | 0% | 0.0% | 1h58m |
| 29060 (call) | 1 | −37.40 | 0.00 | 37.40 | 0/1 | 0% | −0.7% | 18h |
| 13779 (call) | 2 | −56.33 | +95.00 | 151.33 | 1/1 | 50% | −0.2% | 1d15h |
| Zijin 13494 (call) | 2 | −174.41 | −100.00 | 74.41 | 1/1 | 50% | −1.8% | 4d23h |
| 28701 (call) | 3 | −444.12 | −370.00 | 74.12 | 1/2 | 33% | −3.1% | 1d8h |
| 29344 (call) | 5 | −798.60 | −630.00 | 168.60 | 1/4 | 20% | −2.8% | 1d0h |
| Meituan 26790 (call) | 2 | **−2,095.03** | −1,980.00 | 115.03 | 0/2 | 0% | −7.3% | 3d1h |
| WuXi 22574 (call) | 2 | **−2,275.73** | −2,237.50 | 38.23 | 0/2 | 0% | −11.5% | 1d13h |
| Kuaishou 27786 (call) | 4 | **−3,139.66** | −3,045.00 | 94.66 | 1/3 | 25% | −7.0% | 1d17h |
| **Total (33 closed rounds)** | 33 | **−2,723.41** | **−1,592.50** | **1,130.91** | 12/21 | **36.4%** | | avg 1d12h |

> Symbols are **warrant/CBBC codes**; the same underlying may rotate codes across expiries. Names are the underlyings (Xiaomi=1810, ZTE=763, SMIC, WuXi, Meituan, BABA=9988, Zijin).

**Currently still holding (unrealized)**: 10 names, total floating **+HK$1,032.50** (BABA 14461 +600 / 15159 +390 / Kuaishou 27786 +240 / 29344 +120 …).

**Four representative rounds**

- **Xiaomi 14225 (+2,960.99 / +29.6%, held 3d4h)** — the era's biggest single winning round. Entered accumulating 07-10, structure held, fish-body paid. Its 5 rounds total +3,195 — the main force pulling the era back.
- **ZTE 13599 (+1,261.12 / +13.1%, held 3d20h)** — accumulating entry, held to target; 67% is the era's steadiest name.
- **Kuaishou 27786 (−3,139 total, incl. single rounds −1,993 / −20.6% and −1,857 / −32.5%)** — repeatedly entered "accumulating/broad-rally" yet kept bleeding; the era's biggest loss source. **Scale amplified the cost of one biased signal.**
- **WuXi 22574 (−2,275 / −11.5%)** — 07-13 structure flipped and it should've been cut, but **the exit sell order was rejected** by the broker (0.088 unfilled), so it stayed on and bled deeper. This is the "want-to-exit-but-can't" of §3.

---

## 3. Cognitive-iteration axis — every framework version / ops lesson was bought with a loss

> The real output of a ledger is not the P&L number, it's the **rules**. The hand era's −4,145 was the tuition for v1→v3.1; the daemon era's −2,723 (net) bought back three accounts that **only scale and automation expose**.

**Hand era: criterion framework v1 → v3.1**

- **v1 (06-24) · fish-body baseline** — direction first, size only the mid-body, judge the tail by structure + speed (never by magnitude), smart-money = reference not verdict, two independent gates.
- **v2 (06-25) · structure quantified** — a backtestable scale for distribution/accumulation: CLV-120d self-percentile (primary) + real-time large money (secondary). The metals cluster (−6~8%) was all vetoed as "catching the knife = accumulation" — the **anti-false-kill template**.
- **v3 (06-26) · exit discipline [most expensive lesson, −3,475]** — SMIC's two legs entered fully compliant yet bled from +7.64% into a loss. Root cause was the exit: **entered on the structure ruler, then swapped to a price ruler (78) to exit.** Codified R6 + L5.1 + L8.
- **v3.1 (06-29) · structure monitoring operationalized** — turned "flip → exit" into a **two-track poll + `structure_flip` machine criterion**. This version is exactly **what let the daemon auto-execute.**

**Daemon era: three "only-at-scale" ops accounts**

- **① Fee drag [a new first-order cost this era]** — the hand era barely traded, so fees were invisible; 8 days of automation ran 33 rounds and **HK$1,130.91 in fees** turned a −1,592.50 gross straight into a **−2,723.41 net**. At high frequency, **fees are a first-order line item** that must enter the criteria, not be waved off.
- **② Want-to-exit-but-can't [an execution-layer risk unique to automation]** — **10 exit sell orders failed** this era (rejected / limit unfilled to close): Kuaishou 27786, WuXi 22574, Meituan 26790, BABA 14461 all hit "the ruler says exit, the order never fills, the position stays stuck." A human would instantly re-price and re-submit; an automated system may sit on the limit to the close. **The judgment was right; the execution layer didn't catch it, and it lost anyway.** (The executor now rolls back on failure + re-submits after cooldown — but this is the era's clearest next lesson.)
- **③ Ledger-vs-reality gap [measurement itself needs reconciling]** — the local ledger (submit-limit, no fees) showed gross **+806.15**; the SDK real net was only **+88.38** (incl. partial sells of open rounds) / closed rounds **−2,723.41**. "Looks like a gain, actually a loss" is exactly why `reconcile_pnl.js` was born this era: **the panel/ledger can't be taken as a track record; it must be recomputed from SDK fills.**

> **One-line takeaway (across both eras)**: the hand era proved "exit discipline > entry homework"; the daemon era added the back half — **once you hand the discipline to a machine running 7×24, the deciding factors grow by two more layers beyond "is the call right": "can you eat the fees" and "can you actually close the order."** A 36.4% win rate held gross losses to −1,592 thanks to a few big Xiaomi/ZTE rounds, but fees and stuck orders dragged it to −2,723 net. **Scale doesn't change whether the criteria are right — it magnifies every hole outside the criteria.**

---

## 4. P&L summary (through 2026-07-14 close)

**Two eras · two bases**

| Era | Basis | Realized | Key metrics |
|---|---|---|---|
| Hand (06-17→07-02) | no-fee · fill×qty | **−HK$4,145** | 6 legs, cost base 37,310, ≈ −11.1% |
| Daemon (07-03→07-14) | no-fee · gross (same, additive) | **−HK$1,592.50** | 33 rounds |
| Daemon (07-03→07-14) | **SDK real · net of fees** | **−HK$2,723.41** | win 36.4%, fees 1,130.91 |

- **Same-basis (no-fee) cumulative (additive):** `−4,145 + (−1,592.50) =` **−HK$5,737.50**
- **Fee-inclusive real basis (daemon leg):** the hand era's fees predate the reconcile tool and aren't captured, so the true all-in is **slightly worse than −5,737.50**; the daemon leg alone is **net −HK$2,723.41**.
- **Current holdings:** 10 names still held, total floating **+HK$1,032.50** (not realized; 4 of them are "stuck after a failed exit").

**Cumulative curve (close, no-fee):**
Hand leg 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`;
Daemon leg (gross, stacked) 07-02 `−4,145` → 07-14 `−5,737.50`.

> Per-fill detail in `data/paper-trading/`: `2026-warrant-tracker.md` (raw log), `pnl-reconciliation-*.md` (SDK reconcile), `weekly-pnl-*.md` (weekly review).

---

## 5. One line

> What this ledger tries to prove is not "how much was made", but **how the framework evolves under real (paper) money feedback** — and it evolved in two tiers:
> the **hand era** spent −4,145 to buy back the v1→v3.1 criteria (conclusion: exit discipline > entry homework);
> the **daemon era** handed that discipline to a machine running 7×24 and spent −2,723 (net) to buy back three ops accounts: **fees are a first-order cost, exit orders can fail to fill, and the ledger must be reconciled.**
> **Scale and automation don't change whether the criteria are right — they magnify every hole outside the criteria. Losses were converted into rules — that is the ledger's true output.**
