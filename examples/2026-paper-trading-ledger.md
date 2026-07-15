# 示例 · 模拟盘收益台账（2026-06-17 → 07-15）：从「少量手动单」到「7×24 守护进程自动执行」

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘（Longbridge paper account）方法论演示，非业绩、非收益承诺、非投资建议。**
> 这不是"晒收益"，而是把一段完整的模拟盘轨迹摊开：**每一笔动作对应什么决策理由、验证或推翻了框架的哪一条、最后落到多少盈亏**。
>
> 这段轨迹跨两个时代：
> - **手动时代（06-17 → 07-02）**：少量手动单，靠人开会话触发；核心课题是**离场纪律**。已实现 **−4,145 HKD**。
> - **守护进程时代（07-03 → 07-15）**：`hk-market-daemon` 上线，快层 watcher 秒级盯盘 + 自动执行。交易频率从 2 周 6 条腿暴涨到 9 个交易日 **51 个平仓回合**。这一段先用亏损把**想平却平不掉、台账 vs 真实成交落差、常驻进程的状态一致性**这些"规模化才暴露的账"摊开，又在 **07-15 单日毛 +7,285** 的带动下**由累计负转正**。
>
> **截止 2026-07-15 收盘：**
> - 累计已实现 **+1,547.50 HKD**（手动 −4,145 ＋ 守护进程 +5,692.50）——**账户转正**。转正发生在 07-15：守护进程时代当日毛 **+7,285**，把本时代由 07-14 的 −1,592.50 拉到 **+5,692.50**。
> - 守护进程时代：已平 **51 个回合**，16 个标的中 **9 盈 / 6 亏 / 1 平**，合计 **+5,692.50 HKD**（按回合胜率 22/29 ≈ 43%）。
> - 当前仍持有 **10 个标的**，合计浮盈约 **+40 HKD**（未计入已实现；07-14 的 +1,032.50 浮盈今日大多已兑现为已实现）。
>
> **口径说明**：正文数字为 **SDK 真实成交价 × 量、不计手续费**的毛已实现口径（两时代同口径、可直接相加）。含手续费的净口径逐回合明细见 [`2026-07-15-pnl-reconciliation.md`](./2026-07-15-pnl-reconciliation.md)（07-01→07-15：净 +3,972.36、真实已实现 +4,679.07、胜率 43.1%）。单笔买入 ≤ 1 万 HKD；账户纯现金 HKD/USD/SGD 各约 100 万。本地台账逐行盈亏记的是提交限价、有重复/空行，不可直接用；本文一律以 SDK 真实成交回算。

---

## 1. 手动时代 —— 动作 × 理由 × 收益（06-17 → 07-02）

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

**手动时代小结**：6 条已平仓腿，动用并已平仓成本 37,310 HKD，已实现 **−4,145 HKD**（≈ −11.1%）。核心结论——**进场端功课只决定"不选错股"；真正吃掉/守住收益的是离场端**（详见 §3）。

---

## 2. 守护进程时代 —— 规模化之后（07-03 → 07-15）

> 07-03 起 `hk-market-daemon` 上线：快层 watcher 秒级盯持仓价 + 高频轮询大单结构，命中止损/止盈/结构翻转即自动执行。人从"每笔点确认"退到"定规则、盯异常"。**交易量级从此不是一个数量级**——9 个交易日 51 个平仓回合。**07-15 是转折点：单日毛 +7,285，把本时代从 07-14 的 −1,592.50 一举拉到 +5,692.50、整段转正。**
>
> 口径：SDK 真实成交价 × 量、不计费（一「笔」= 平仓回合 0→持仓→0）。

**单标的汇总（按盈亏降序，截止 07-15）**

| 标的（方向） | 回合 | 盈亏 HKD（不计费） | 均持 |
|---|---|---|---|
| 小米14225（认购） | 6 | **+3,660.00** | 1d2h |
| 阿里14461（认购） | 1 | **+3,325.00** | 6d22h |
| 中兴13599（认购） | 7 | **+2,440.00** | 13h |
| 美团29665（认购） | 3 | **+2,090.00** | 15h |
| 信达29344（认购） | 8 | +820.00 | 18h |
| 中芯23604（认购） | 1 | +700.00 | 1h51m |
| 29053（认购） | 1 | +495.00 | 3h53m |
| 洪桥15159（认购） | 3 | +450.00 | 7h12m |
| 13779（认购） | 2 | +95.00 | 1d14h |
| 快手29060（认沽） | 1 | 0.00 | 18h |
| 紫金13494（认购） | 2 | −100.00 | 4d23h |
| 药明康德28701（认购） | 5 | −140.00 | 1d |
| 国寿14673（认购） | 3 | −880.00 | 7h33m |
| 美团26790（认购） | 2 | **−1,980.00** | 3d1h |
| 药明生物22574（认购） | 2 | **−2,237.50** | 1d13h |
| 快手27786（认购） | 4 | **−3,045.00** | 1d17h |
| **合计（51 回合）** | 51 | **+5,692.50** | 均持 1d5h |

> 标的为**窝轮/牛熊证代码**，同一正股不同期可能换证（小米=1810、阿里=9988、中兴=763、美团=3690、信达=1801、中芯、洪桥=1378、药明康德=2359、药明生物=2269、国寿=2628、紫金=2899、快手=1024）。16 个标的口径：**9 盈 / 6 亏 / 1 平**；按回合 22 胜 / 29 负 ≈ 43%。

**当前仍持有（未实现，不计入已实现）**：10 个标的，合计浮盈约 **+40 HKD**（07-14 的 +1,032.50 浮盈今日大多兑现为已实现；剩余持仓多空互抵近平）。

**代表性回合**

- **07-15 的转正日** —— 单日毛 +7,285 由几笔大回合撑起：**阿里14461 +3,325**（06 日建的老仓持满 6 天今晨了结）、**美团29665 +2,530**、**信达29344 +1,361**、**中兴13599 +980**；亏损端 美团29665 开盘追高一笔 −979、国寿14673 −739。**同一天既有最大单笔盈利、也有开盘追高的亏损**——后者正是 §3 教训②"重启批量追高"的现场。
- **小米14225（本时代最大盈利标的，6 回合累计 +3,660）**——07-10 吸筹进场、结构一直站住，鱼身兑现；把整段拉回来的主力。
- **中兴13599（7 回合 +2,440）**——最稳的一档；也吃过一次 §3 教训③的"刚卖又买"抖动（14:41 翻转确认卖、14:47 又买、−360）。
- **快手27786（合计 −3,045，含单回合约 −20%、−32%）**——反复以"吸筹/普涨"进场却持续失血，本时代最大亏损源。更关键的是 **2026-07-13 15:58 HKT 结构尺已判「翻转确认」、发出平仓单（卖 140,000@0.073），却挂到收盘过期未成交**——该走没走成，仓位拖到隔日继续亏。**规模化把一个偏差信号的代价放大，执行层再没兜住**（详见 §3 ①）。
- **药明生物22574（−2,237.50）**——**2026-07-13 11:04 HKT 触发止损、发出平仓单（卖 87,500@0.088）被券商拒单**，只能在更低价补平，亏损里含这段被拒后的滑价。同属 §3 ① 的"想平却平不掉"。

---

## 3. 认知迭代轴 —— 每一版框架/每一条运维教训，都是一笔亏损"买"来的

> 台账的真正产出不是收益数字，而是**规则**。手动时代的 −4,145 是 v1→v3.1 的"学费"；守护进程时代又用亏损"买"回**三条只有规模化、自动化才暴露**的运维账（① 想平却平不掉 ② 度量要对账 ③ 常驻进程的状态一致性），并在补上第三条的同一天（07-15）由 −1,592.50 转正为 +5,692.50。

**手动时代：判据框架 v1 → v3.1**

- **v1（06-24）· 鱼身定位基线** — 先定方向、只重仓鱼中、鱼尾按结构+速度判（绝不按涨跌幅）、主力=参考非裁决、两道独立闸（股票鱼身 ∩ 权证经济性）。
- **v2（06-25）· 结构量化** — 给"大幅派发/吸筹"一把可回测的刻度：CLV 120 日自我分位（主）+ 实时大单（辅）。当日金属簇 −6~8% 全被"大单接刀=吸筹"否决，**防错杀样板**。
- **v3（06-26）· 离场纪律【最贵的一课，−3,475】** — 中芯两腿进场完全合规（+26 亿吸筹），却从 +7.64% 回吐成亏损。根因在离场：**进场用结构尺、离场却偷换成价格尺 78**。落库 R6 + L5.1 + L8。
- **v3.1（06-29）· 结构监控落地** — 把"结构翻转就走"落成**双轨轮询 + `structure_flip` 机器判据**（大单方向 + flow 斜率/回撤双证）。这一版正是**守护进程能自动执行的前提**。

**守护进程时代：三条"规模化才暴露"的运维账**

- **① 想平却平不掉【自动化独有的执行层风险】** — 本时代累计 **17 笔平仓卖单失败**（拒单 / 限价挂到收盘未成交；07-15 起还叠加了"重启自撤"，见 ③.4）：快手27786、药明生物22574、美团26790、阿里14461 都出现"结构尺喊走、单子没成交、仓位卡住"。人手点单会立刻改价重挂，自动系统却可能一路挂到收盘。**结构判对了，执行层没兜住，照样多亏。**（executor 已加失败回滚+冷却重挂，但这是本时代最该补的下一课。）

  **关键失败平仓单实录（想平没平掉）**——两大亏损标的的失血，很大一块正来自这里：

  | 时间（HKT） | 标的 | 平仓触发 | 卖单 | 券商结果 | 后果 |
  |---|---|---|---|---|---|
  | 2026-07-13 15:58 | 快手27786 | **结构翻转确认** | 卖 140,000@0.073 | 过期未成交 | 结构尺已判该走，单子挂到收盘没成交，仓位拖到隔日继续失血 |
  | 2026-07-07 09:15 | 快手27786 | 止损 | 卖 65,000@0.098 | 过期未成交 | 止损没执行成，回合亏损扩大 |
  | 2026-07-13 11:04 | 药明22574 | 止损 | 卖 87,500@0.088 | 拒单 | 被拒后才在更低价补平，−2,237.50 里含这段滑价 |

  > 快手 −3,045、药明 −2,237.50 这两个本时代最大亏损，**不是判据看错方向，而是"判对了却卖不出去"**——结构翻转/止损信号都发了，卖单却被拒或挂到收盘过期。这类账在手动时代看不见（人手会立刻改价重下），只有自动化 7×24 才暴露。

- **② 度量本身要对账【台账不可直接当业绩】** — 本地台账逐行盈亏记的是**提交限价**、还夹着重复/空行，与真实成交价回算的结果对不上。本时代因此把盈亏统一改用 **`reconcile_pnl.js` 按 SDK 真实成交回算**：**面板/台账只能当底稿，公开的收益数字必须用真实成交价重算。**

- **③ 常驻进程的状态一致性【07-15 补：四修 + 一条自造的坑】** — 让机器 7×24 跑、且盘中反复热更新，暴露出一批"进程重启 / 隔夜残留 / 判据抖动"的状态账。**今天没动任何判据**，全在执行层修补：

  1. **隔夜残单跨日成交** — Day 限价单当日没成交又没撤，挂到次日开盘一次性撮合：快手27786 今晨 09:32 三笔同价成交，实为昨日 16:01 止损 + 16:15 进场两笔隔夜残单 + 今日进场单同时成交。→ **收盘/退出即撤未成交单 + 启动清隔夜残单**（只撤本项目 remark 的单、仅模拟盘，绝不碰手动/真实单）。
  2. **进程重启"失忆"批量追高** — 去重/冷却是内存态、重启即清零；加上进场原是"价格 ≥ trigger"的**电平**判断，重启一上来把所有"现价已在 trigger 上方"的标的一次性全买（追高）。→ **进场改「trigger 一次性」**：每个 trigger 只打一次，须等慢层重新分析更新该标的 trigger 才复活；用当日成交记录恢复状态，重启不忘。改后重启只出合理离场、**零重复进场**。（07-15 开盘那笔美团29665 −979 正是修复前的追高。）
  3. **结构判据阈值抖动"刚卖又买"** — 中兴13599 因资金回撤比 dd=0.52 越过 0.5 判「翻转确认」于 14:41 卖出，14:47 dd 回落判「翻转预警」、进场闸松开又买回，同价同量、−360。→ 由 ②·2 的"trigger 一次性"一并覆盖（同 trigger 不重进）。
  4. **⚠️ 一条自己造的坑（诚实记录）** — ① 的"退出撤单"会把**未成交的止盈单也一并撤掉**；今天为上线补丁一天内重启了 **5 次**，每次都撤掉当时挂着的止盈/止损单——对账里 07-15 那 10+ 笔"已撤单"平仓单就是这个副作用，被机器误标成"券商平仓失败"（也是 ① 的 17 笔里今天新增的部分）。**教训：撤单要区分"隔夜残留"与"当日在途的合法离场单"；盘中热修代价高，非紧急改动应收盘后部署。**
  5. **手动 App 操作已可被感知同步** — 订单回报订阅收全账户单，人在 App 手动止盈止损 → 自动重拉持仓、失效盈亏缓存、补记台账、推飞书。人机同池不再脱节。

> **一句话认知（跨两个时代）**：手动时代证明了"离场纪律 > 进场功课"；守护进程时代补上后半句——**当你把纪律交给机器 7×24 执行，胜负手在"判断对不对"之外又多了两层："单子平不平得掉"、"进程重启/隔夜后状态一不一致"。** 16 个标的 9 盈 6 亏 1 平，靠小米/阿里/中兴/美团几个大回合把整段做到 **+5,692.50、并在 07-15 转正**；但转正当天也暴露了追高、卡单、自撤止盈这些执行漏洞。**规模化不改判据的对错，只放大判据之外的每一个漏洞——把漏洞逐条堵成规则，就是台账的产出。**

---

## 4. 收益汇总（截止 2026-07-15 收盘）

**两个时代（同口径：SDK 真实成交价×量、不计费）**

| 时代 | 已实现 | 关键指标 |
|---|---|---|
| 手动（06-17→07-02） | **−4,145 HKD** | 6 腿，成本基数 37,310，≈ −11.1% |
| 守护进程（07-03→07-15） | **+5,692.50 HKD** | 51 回合，16 标的 9 盈/6 亏/1 平；含 07-15 单日 +7,285 |
| **合计** | **+1,547.50 HKD（转正）** | |

- **当前持仓：** 10 个标的仍持有，合计浮盈约 **+40 HKD**（未计入已实现；07-14 的 +1,032.50 浮盈今日大多兑现为已实现）。

**累计曲线（收盘）**：
手动段 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`；
守护进程段（叠加）07-02 `−4,145` → 07-14 `−5,737.50` → **07-15 `+1,547.50`（单日 +7,285，账户转正）**。

> 逐笔明细见项目内 `data/paper-trading/`：`2026-warrant-tracker.md`（逐笔底稿）、`pnl-reconciliation-*.md`（SDK 对账，含费净口径）、`weekly-pnl-*.md`（周复盘）。

---

## 5. 一句话总结

> 这张台账想证明的不是"赚了多少"，而是**框架在真金白银（模拟）的反馈里怎么进化**——而且进化分两级：
> **手动时代**用 −4,145 买回了 v1→v3.1 的判据（结论：离场纪律 > 进场功课）；
> **守护进程时代**把这套纪律交给机器 7×24 跑，买回三条运维账：**平仓单会平不掉、台账必须用真实成交回算、常驻进程重启/隔夜后状态要一致**——并在补上第三条的 07-15 由累计负转正（**+1,547.50 HKD**）。
> **规模化和自动化不改判据的对错，只把判据之外的每一个漏洞放大给你看。亏损被转成了规则——这才是台账的产出。**

---
<a id="english"></a>
# Example · Paper-trading ledger (2026-06-17 → 07-15): from "a few hand trades" to a "7×24 daemon"

> **🌐 [中文 ↑](#示例--模拟盘收益台账2026-06-17--07-15从少量手动单到724-守护进程自动执行) · English (current)**
>
> ⚠️ **A paper-account (Longbridge) methodology walkthrough — NOT a track record, NOT a return promise, NOT investment advice.**
> This is not a "look at my gains" post; it lays a full paper-trading trajectory flat so you can see **which decision rationale each fill came from, which framework rule it validated or broke, and what P&L it landed at**.
>
> The trajectory spans two eras:
> - **Hand era (06-17 → 07-02)**: a few manual trades, human-triggered per session; the core topic is **exit discipline**. Realized **−HK$4,145**.
> - **Daemon era (07-03 → 07-15)**: `hk-market-daemon` went live — a fast-layer watcher polling prices per-second + auto-execution. Trade frequency exploded from 6 legs in two weeks to **51 close-rounds in 9 sessions**. This era first laid bare the costs that only appear at scale — **want-to-exit-but-can't, the ledger-vs-real-fill gap, and long-running-process state consistency** — then, powered by a **+7,285 gross single day on 07-15**, flipped from cumulative-negative to positive.
>
> **Through 2026-07-15 close:**
> - Cumulative realized **+HK$1,547.50** (hand −4,145 + daemon +5,692.50) — **the account turned positive**, on 07-15 (that single day's +7,285 gross lifted the daemon era from −1,592.50 to +5,692.50).
> - Daemon era: 51 closed rounds; of 16 names, **9 win / 6 loss / 1 flat**; total **+HK$5,692.50** (per-round win rate 22/29 ≈ 43%).
> - Currently still holding **10 names**, total floating ~**+HK$40** (07-14's +1,032.50 float was mostly realized today).
>
> **Basis note**: throughout, this is **SDK real fill-price × qty, no fees** — gross realized (same basis across both eras, additive). For the fee-inclusive net basis per round, see [`2026-07-15-pnl-reconciliation.md`](./2026-07-15-pnl-reconciliation.md) (07-01→07-15: net +3,972.36, real realized +4,679.07, 43.1% win). Per-trade buys ≤ HK$10k; account all-cash, ~1M each HKD/USD/SGD. The local ledger's per-row P&L uses submit-limit prices (with duplicate/blank rows) and is not usable directly; all numbers here are recomputed from SDK real fills.

---

## 1. Hand era — action × rationale × P&L (06-17 → 07-02)

> "Realized" = realized P&L (HKD) of that close; "Cumulative" = running realized at that day's close. Open positions are not counted.

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

## 2. Daemon era — after going to scale (07-03 → 07-15)

> From 07-03 `hk-market-daemon` went live: a fast-layer watcher polls position prices per-second + polls large-money structure at high frequency, auto-executing on stop / target / structure-flip. The human stepped back from "click-confirm each trade" to "set rules, watch exceptions." Volume jumped an order of magnitude — 51 close-rounds in 9 sessions. **07-15 was the turning point: a +7,285 gross single day lifted the era from −1,592.50 (through 07-14) to +5,692.50, flipping the whole era positive.**
>
> Basis: SDK real fill-price × qty, no fees (a "round" = 0→position→0).

**By name (P&L desc, through 07-15)**

| Name (dir.) | Rounds | P&L HKD (no fee) | Avg hold |
|---|---|---|---|
| Xiaomi 14225 (call) | 6 | **+3,660.00** | 1d2h |
| BABA 14461 (call) | 1 | **+3,325.00** | 6d22h |
| ZTE 13599 (call) | 7 | **+2,440.00** | 13h |
| Meituan 29665 (call) | 3 | **+2,090.00** | 15h |
| Innovent 29344 (call) | 8 | +820.00 | 18h |
| SMIC 23604 (call) | 1 | +700.00 | 1h51m |
| 29053 (call) | 1 | +495.00 | 3h53m |
| Hongqiao 15159 (call) | 3 | +450.00 | 7h12m |
| 13779 (call) | 2 | +95.00 | 1d14h |
| Kuaishou 29060 (put) | 1 | 0.00 | 18h |
| Zijin 13494 (call) | 2 | −100.00 | 4d23h |
| WuXi AppTec 28701 (call) | 5 | −140.00 | 1d |
| China Life 14673 (call) | 3 | −880.00 | 7h33m |
| Meituan 26790 (call) | 2 | **−1,980.00** | 3d1h |
| WuXi Bio 22574 (call) | 2 | **−2,237.50** | 1d13h |
| Kuaishou 27786 (call) | 4 | **−3,045.00** | 1d17h |
| **Total (51 rounds)** | 51 | **+5,692.50** | avg 1d5h |

> Symbols are **warrant/CBBC codes**; the same underlying may rotate codes across expiries (Xiaomi=1810, BABA=9988, ZTE=763, Meituan=3690, Innovent=1801, SMIC, Hongqiao=1378, WuXi AppTec=2359, WuXi Bio=2269, China Life=2628, Zijin=2899, Kuaishou=1024). Across 16 names: **9 win / 6 loss / 1 flat**; by round 22 win / 29 loss ≈ 43%.

**Currently still holding (unrealized)**: 10 names, total floating ~**+HK$40** (07-14's +1,032.50 float was mostly realized today; the remaining book roughly nets flat).

**Representative rounds**

- **07-15, the turnaround day** — a +7,285 gross day carried by a few big rounds: **BABA 14461 +3,325** (an old leg opened on the 6th, held a full 6 days, closed this morning), **Meituan 29665 +2,530**, **Innovent 29344 +1,361**, **ZTE 13599 +980**; on the loss side Meituan 29665 −979 (an open chase) and China Life 14673 −739. **The same day held both the biggest single win and an open-chase loss** — the latter is exactly §3 lesson ②2 "restart mass-chase" in the flesh.
- **Xiaomi 14225 (the era's biggest winning name, 6 rounds +3,660)** — entered accumulating 07-10, structure held, fish-body paid; the main force pulling the era back.
- **ZTE 13599 (7 rounds +2,440)** — the steadiest name; also took one §3 lesson ②3 "sell-then-rebuy" whipsaw (flip-sold 14:41, re-bought 14:47, −360).
- **Kuaishou 27786 (−3,045 total, incl. single rounds ~−20% and −32%)** — repeatedly entered "accumulating/broad-rally" yet kept bleeding; the era's biggest loss source. More telling: **on 2026-07-13 15:58 HKT the structure ruler had confirmed the flip and fired the exit (sell 140,000@0.073), but it sat unfilled to the close** — the exit never happened and the position dragged into the next day (see §3 ①).
- **WuXi Bio 22574 (−2,237.50)** — **on 2026-07-13 11:04 HKT a stop-loss exit fired (sell 87,500@0.088) but the broker rejected it**, re-filled lower; part of the loss is that slippage. Also the "want-to-exit-but-can't" of §3 ①.

---

## 3. Cognitive-iteration axis — every framework version / ops lesson was bought with a loss

> The real output of a ledger is not the P&L number, it's the **rules**. The hand era's −4,145 was the tuition for v1→v3.1; the daemon era bought back **three accounts that only scale and automation expose** (① want-to-exit-but-can't ② measurement must be reconciled ③ long-running-process state consistency), and on the very day it closed the third (07-15) it flipped from −1,592.50 to +5,692.50.

**Hand era: criterion framework v1 → v3.1**

- **v1 (06-24) · fish-body baseline** — direction first, size only the mid-body, judge the tail by structure + speed (never by magnitude), smart-money = reference not verdict, two independent gates.
- **v2 (06-25) · structure quantified** — a backtestable scale for distribution/accumulation: CLV-120d self-percentile (primary) + real-time large money (secondary). The metals cluster (−6~8%) was all vetoed as "catching the knife = accumulation" — the **anti-false-kill template**.
- **v3 (06-26) · exit discipline [most expensive lesson, −3,475]** — SMIC's two legs entered fully compliant yet bled from +7.64% into a loss. Root cause was the exit: **entered on the structure ruler, then swapped to a price ruler (78) to exit.** Codified R6 + L5.1 + L8.
- **v3.1 (06-29) · structure monitoring operationalized** — turned "flip → exit" into a **two-track poll + `structure_flip` machine criterion**. This version is exactly **what let the daemon auto-execute.**

**Daemon era: three "only-at-scale" ops accounts**

- **① Want-to-exit-but-can't [an execution-layer risk unique to automation]** — **17 exit sell orders failed** this era (rejected / limit unfilled to close; from 07-15 also "cancelled-on-restart", see ③.4): Kuaishou 27786, WuXi Bio 22574, Meituan 26790, BABA 14461 all hit "the ruler says exit, the order never fills, the position stays stuck." A human would instantly re-price and re-submit; an automated system may sit on the limit to the close. **The judgment was right; the execution layer didn't catch it, and it lost anyway.** (The executor now rolls back on failure + re-submits after cooldown — but this is the era's clearest next lesson.)

  **Record of the key failed exits (wanted out, couldn't get out)** — much of the two biggest losers' bleed comes from exactly here:

  | Time (HKT) | Name | Exit trigger | Sell order | Broker result | Consequence |
  |---|---|---|---|---|---|
  | 2026-07-13 15:58 | Kuaishou 27786 | **confirmed flip** | sell 140,000@0.073 | expired unfilled | ruler had said exit; order sat to the close unfilled, position dragged into next day |
  | 2026-07-07 09:15 | Kuaishou 27786 | stop-loss | sell 65,000@0.098 | expired unfilled | stop never executed, the round's loss widened |
  | 2026-07-13 11:04 | WuXi 22574 | stop-loss | sell 87,500@0.088 | rejected | re-filled lower after the reject; the −2,237.50 includes this slippage |

  > Kuaishou −3,045 and WuXi −2,237.50, the era's two biggest losses, **weren't the criteria pointing the wrong way — they were "right call, couldn't sell."** The flip/stop signals both fired, but the sell orders were rejected or expired at the close. This account is invisible in the hand era (a human instantly re-prices and re-submits); only 7×24 automation exposes it.

- **② Measurement itself must be reconciled [the ledger isn't a track record]** — the local ledger's per-row P&L uses **submit-limit prices** (with duplicate/blank rows) and doesn't match a recompute from real fills. So this era standardized P&L on **`reconcile_pnl.js`, recomputed from SDK real fills**: **the panel/ledger is only a source; any published number must be recomputed from real fill prices.**

- **③ Long-running-process state consistency [added 07-15: four fixes + one self-inflicted bug]** — running 7×24 *and* hot-patching intraday exposed a cluster of "restart / overnight-leftover / criterion-jitter" state accounts. **No criterion changed today**; all fixes were execution-layer:

  1. **Stale day-orders fill across sessions** — unfilled, un-cancelled limit orders fill at the next open: Kuaishou 27786's three same-price fills this morning were yesterday's 16:01 stop + 16:15 entry (two overnight leftovers) + today's entry. → **cancel unfilled orders on close/shutdown + clean stale orders on startup** (only our own remark-tagged orders, paper only, never manual/live).
  2. **Restarts "forget" → mass-chase** — dedup/cooldown are in-memory, wiped on restart; entry was a *level* check (`price ≥ trigger`), so a restart bought everything already above its trigger at once. → **one-shot-per-trigger entry**: each trigger fires once, re-armed only when the slow layer updates that item's trigger; state recovered from today's fills so restarts don't forget. After the fix, a restart emits only legit exits, **zero re-entries** (07-15's open chase Meituan 29665 −979 was pre-fix).
  3. **Criterion jitter around a threshold → sell-then-rebuy** — ZTE 13599 flip-sold 14:41 (flow drawdown 0.52 > 0.5 → "confirmed"), re-bought 14:47 when it eased to "warning"; same price/qty, −360. → Covered by ②2's one-shot-per-trigger.
  4. **⚠️ A self-inflicted bug (logged honestly)** — ①'s "cancel-on-shutdown" also cancels *unfilled take-profit* orders; with 5 restarts today to ship patches, each cancelled the resting exits — the 10+ "cancelled" close-orders in today's report are this side-effect, mislabeled as "broker close failures" (and part of ①'s 17). **Lesson: distinguish stale leftovers from legitimate in-flight exits; hot-patching intraday is costly — deploy after close unless urgent.**
  5. **Manual app actions are now sensed** — the order feed captures all account orders, so manual stop/take-profit → auto re-pull positions, invalidate P&L cache, mirror to the local ledger, push to Feishu.

> **One-line takeaway (across both eras)**: the hand era proved "exit discipline > entry homework"; the daemon era added the back half — **once you hand the discipline to a machine running 7×24, the deciding factors grow by two more layers beyond "is the call right": "can you actually close the order," and "is state consistent across restarts / overnight."** Across 16 names 9 win / 6 loss / 1 flat, a few big Xiaomi/BABA/ZTE/Meituan rounds took the era to +5,692.50 and flipped it positive on 07-15 — the same day that also exposed chasing, stuck orders, and self-cancelled take-profits. **Scale doesn't change whether the criteria are right — it magnifies every hole outside the criteria; each hole plugged becomes a rule — that is the ledger's output.**

---

## 4. P&L summary (through 2026-07-15 close)

**Two eras (same basis: SDK real fill-price × qty, no fees)**

| Era | Realized | Key metrics |
|---|---|---|
| Hand (06-17→07-02) | **−HK$4,145** | 6 legs, cost base 37,310, ≈ −11.1% |
| Daemon (07-03→07-15) | **+HK$5,692.50** | 51 rounds, 16 names 9 win/6 loss/1 flat; incl. 07-15's +7,285 single day |
| **Total** | **+HK$1,547.50 (positive)** | |

- **Current holdings:** 10 names still held, total floating ~**+HK$40** (07-14's +1,032.50 was mostly realized today).

**Cumulative curve (close):**
Hand leg 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`;
Daemon leg (stacked) 07-02 `−4,145` → 07-14 `−5,737.50` → **07-15 `+1,547.50` (a +7,285 day; account turned positive)**.

> Per-fill detail in `data/paper-trading/`: `2026-warrant-tracker.md` (raw log), `pnl-reconciliation-*.md` (SDK reconcile, net basis), `weekly-pnl-*.md` (weekly review).

---

## 5. One line

> What this ledger tries to prove is not "how much was made", but **how the framework evolves under real (paper) money feedback** — and it evolved in two tiers:
> the **hand era** spent −4,145 to buy back the v1→v3.1 criteria (conclusion: exit discipline > entry homework);
> the **daemon era** handed that discipline to a machine running 7×24 and bought back three ops accounts: **exit orders can fail to fill, the ledger must be recomputed from real fills, and a long-running process must stay state-consistent across restarts/overnight** — turning cumulative positive (+1,547.50) on 07-15, the day it closed the third.
> **Scale and automation don't change whether the criteria are right — they magnify every hole outside the criteria. Losses were converted into rules — that is the ledger's true output.**
