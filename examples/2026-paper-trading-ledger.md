# 示例 · 模拟盘收益台账（2026-06-17 → 07-16）：从「少量手动单」到「7×24 守护进程自动执行」

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘（Longbridge paper account）方法论演示，非业绩、非收益承诺、非投资建议。**
> 这不是"晒收益"，而是把一段完整的模拟盘轨迹摊开：**每一笔动作对应什么决策理由、验证或推翻了框架的哪一条、最后落到多少盈亏**。
>
> 这段轨迹跨两个时代：
> - **手动时代（06-17 → 07-02）**：少量手动单，靠人开会话触发；核心课题是**离场纪律**。已实现 **−4,145 HKD**。
> - **守护进程时代（07-03 → 07-16）**：`hk-market-daemon` 上线，快层 watcher 秒级盯盘 + 自动执行。交易频率从 2 周 6 条腿暴涨到 10 个交易日 **65 个平仓回合**。这一段先用亏损把**想平却平不掉、台账 vs 真实成交落差、常驻进程的状态一致性、离场信号的盘外护栏**这些"规模化才暴露的账"逐条摊开；在 07-15 单日毛 +7,285 由累计负转正后，**07-16 延续修复行情、单日再毛 +9,680**——最大亏损标的快手27786 单日翻回 +5,383，守护进程时代累计做到 +15,373。
>
> **截止 2026-07-16 收盘：**
> - 累计已实现 **+11,228 HKD**（手动 −4,145 ＋ 守护进程 +15,373）。转正发生在 07-15，07-16 进一步拉开：守护进程时代当日毛 **+9,680**（07-15 收盘 +5,692.50 → 07-16 收盘 +15,373）。
> - 守护进程时代：已平 **65 个回合**，22 个标的中 **13 盈 / 7 亏 / 2 平**，合计 **+15,373 HKD**（按回合胜率 28/37 ≈ 43%）。
> - 当前仍持有 **12 个标的**，合计浮动约 **−2,689 HKD**（未计入已实现；07-16 大量浮盈已兑现为已实现，剩余多为深度价外/被套持仓）。
>
> **口径说明**：正文数字为 **SDK 真实成交价 × 量、不计手续费**的毛已实现口径（两时代同口径、可直接相加）。含手续费的净口径逐回合明细见 [`2026-07-16-pnl-reconciliation.md`](./2026-07-16-pnl-reconciliation.md)（07-01→07-16：已平回合净 +12,640.96、真实已实现 +12,606.54、手续费 2,732.04、胜率 43.1%）；07-15 快照见 [`2026-07-15-pnl-reconciliation.md`](./2026-07-15-pnl-reconciliation.md)。单笔买入 ≤ 1 万 HKD；账户纯现金 HKD/USD/SGD 各约 100 万。本地台账逐行盈亏记的是提交限价、有重复/空行，不可直接用；本文一律以 SDK 真实成交回算。

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

## 2. 守护进程时代 —— 规模化之后（07-03 → 07-16）

> 07-03 起 `hk-market-daemon` 上线：快层 watcher 秒级盯持仓价 + 高频轮询大单结构，命中止损/止盈/结构翻转即自动执行。人从"每笔点确认"退到"定规则、盯异常"。**交易量级从此不是一个数量级**——10 个交易日 65 个平仓回合。**07-15 单日毛 +7,285 由 07-14 的 −1,592.50 转正到 +5,692.50；07-16 单日再毛 +9,680，把本时代推到 +15,373。最亮的一笔是曾经最大的亏损源快手27786——单日一个回合 +5,383（+18.2%），把它从累计 −3,045 翻成 +2,415。**
>
> 口径：SDK 真实成交价 × 量、不计费（一「笔」= 平仓回合 0→持仓→0）。

**单标的汇总（按盈亏降序，截止 07-16）**

| 标的（方向） | 回合 | 盈亏 HKD（不计费） | 均持 |
|---|---|---|---|
| 小米14225（认购） | 7 | **+6,900.00** | 1d2h |
| 阿里14461（认购） | 1 | **+3,325.00** | 6d22h |
| 快手27786（认购） | 5 | **+2,415.00** | 1d19h |
| 美团29665（认购） | 5 | **+2,360.00** | 14h |
| 中兴13599（认购） | 8 | **+1,540.00** | 13h |
| 信达29344（认购） | 8 | +820.00 | 18h |
| 紫金13533（认购） | 1 | +700.00 | 1d21h |
| 中芯23604（认购） | 1 | +700.00 | 1h51m |
| 中芯29053（认购） | 1 | +495.00 | 3h53m |
| 洪桥15159（认购） | 3 | +450.00 | 7h12m |
| 泡泡玛特14905（认购） | 1 | +368.00 | 3h24m |
| 平安14023（认购） | 1 | +200.00 | 4h30m |
| 13779（认购） | 2 | +95.00 | 1d15h |
| 小鹏14275（认购） | 1 | 0.00 | 3h |
| 快手29060（认沽） | 1 | 0.00 | 18h |
| 中芯21496（认沽） | 2 | −40.00 | 10h35m |
| 紫金13494（认购） | 2 | −100.00 | 4d23h |
| 药明生物22574（认购） | 3 | −535.00 | 1d16h |
| 药明康德28701（认购） | 6 | −680.00 | 1d |
| 阿里13790（认购） | 1 | −780.00 | 3h57m |
| 国寿14673（认购） | 3 | −880.00 | 7h33m |
| 美团26790（认购） | 2 | **−1,980.00** | 3d1h |
| **合计（65 回合）** | 65 | **+15,373.00** | 均持 约1d3h |

> 标的为**窝轮/牛熊证代码**，同一正股不同期/不同方向可能换证（小米=1810、阿里=9988、快手=1024、美团=3690、中兴=763、信达=1801、紫金=2899、中芯=981、洪桥=1378、泡泡玛特=9992、平安=2318、小鹏=9868、药明生物=2269、药明康德=2359、国寿=2628）。22 个标的口径：**13 盈 / 7 亏 / 2 平**；按回合 28 胜 / 37 负 ≈ 43%。

**当前仍持有（未实现，不计入已实现）**：12 个标的，合计浮动约 **−2,689 HKD**（07-16 大量浮盈已兑现为已实现；剩余多为深度价外或被套持仓，多空互抵后偏负）。

**代表性回合**

- **07-16 的续涨日** —— 单日毛 +9,680。主力是**快手27786 +5,383（+18.2%）**：本时代一度最大的亏损源（累计 −3,045），07-14 低位 0.082 建的仓在今日反弹里了结，单一回合把它翻成累计 +2,415——**"最烂标的"靠一个大回合翻身**。其余大回合：**小米14225 +3,221（+33.5%）**、**药明生物22574 +1,665（+16.7%，把它从 −2,237.50 收窄到 −535）**、**紫金13533 +681**；亏损端 中兴13599 −900、药明康德28701 −559、阿里13790 −780。
- **07-15 的转正日** —— 单日毛 +7,285：阿里14461 +3,325（老仓持满 6 天）、美团29665 +2,530、信达29344 +1,361、中兴13599 +980；亏损端 美团29665 开盘追高 −979、国寿14673 −739（后者是 §3 教训②"重启批量追高"现场）。
- **小米14225（本时代最大盈利标的，7 回合累计 +6,900）**——07-10 起吸筹进场、结构反复站住，鱼身多次兑现；把整段拉起来的主力。
- **快手27786（合计 +2,415，路径最曲折）**——本时代前段最大亏损源（含单回合 −20%、−32%，且 07-13 15:58 结构翻转平仓单挂到收盘过期未成交，仓位拖到隔日继续亏，详见 §3 ①），07-16 靠一个大回合翻回正数。**同一标的既是 §3① 的反面教材，也是最终净贡献者。**
- **药明生物22574（−535，曾 −2,237.50）**——07-13 止损单被拒（详见 §3 ①）后深亏，07-16 靠 +1,665 的回合把窟窿收窄了大半。

---

## 3. 认知迭代轴 —— 每一版框架/每一条运维教训，都是一笔亏损"买"来的

> 台账的真正产出不是收益数字，而是**规则**。手动时代的 −4,145 是 v1→v3.1 的"学费"；守护进程时代又用亏损"买"回**四条只有规模化、自动化才暴露**的运维账（① 想平却平不掉 ② 度量要对账 ③ 常驻进程的状态一致性 ④ 离场信号的盘外护栏）。转正发生在 07-15，07-16 一边延续修复行情、一边又暴露并修好了第四条。

**手动时代：判据框架 v1 → v3.1**

- **v1（06-24）· 鱼身定位基线** — 先定方向、只重仓鱼中、鱼尾按结构+速度判（绝不按涨跌幅）、主力=参考非裁决、两道独立闸（股票鱼身 ∩ 权证经济性）。
- **v2（06-25）· 结构量化** — 给"大幅派发/吸筹"一把可回测的刻度：CLV 120 日自我分位（主）+ 实时大单（辅）。当日金属簇 −6~8% 全被"大单接刀=吸筹"否决，**防错杀样板**。
- **v3（06-26）· 离场纪律【最贵的一课，−3,475】** — 中芯两腿进场完全合规（+26 亿吸筹），却从 +7.64% 回吐成亏损。根因在离场：**进场用结构尺、离场却偷换成价格尺 78**。落库 R6 + L5.1 + L8。
- **v3.1（06-29）· 结构监控落地** — 把"结构翻转就走"落成**双轨轮询 + `structure_flip` 机器判据**（大单方向 + flow 斜率/回撤双证）。这一版正是**守护进程能自动执行的前提**。

**守护进程时代：四条"规模化才暴露"的运维账**

- **① 想平却平不掉【自动化独有的执行层风险】** — 对账累计标记 **44 笔平仓卖单未成交**（拒单 / 限价挂到收盘未成交 / 重启自撤 / 盘外触发；后两类是下面 ③.4、④ 的自造问题，真正的券商拒单+挂到收盘过期是其中一部分）：快手27786、药明生物22574、美团26790、阿里14461 都出现"结构尺喊走、单子没成交、仓位卡住"。人手点单会立刻改价重挂，自动系统却可能一路挂到收盘。**结构判对了，执行层没兜住，照样多亏。**（executor 已加失败回滚+冷却重挂，但这是本时代最该补的下一课。）

  **关键失败平仓单实录（想平没平掉）**——两大亏损标的的失血，很大一块正来自这里：

  | 时间（HKT） | 标的 | 平仓触发 | 卖单 | 券商结果 | 后果 |
  |---|---|---|---|---|---|
  | 2026-07-13 15:58 | 快手27786 | **结构翻转确认** | 卖 140,000@0.073 | 过期未成交 | 结构尺已判该走，单子挂到收盘没成交，仓位拖到隔日继续失血 |
  | 2026-07-07 09:15 | 快手27786 | 止损 | 卖 65,000@0.098 | 过期未成交 | 止损没执行成，回合亏损扩大 |
  | 2026-07-13 11:04 | 药明22574 | 止损 | 卖 87,500@0.088 | 拒单 | 被拒后才在更低价补平，−2,237.50 里含这段滑价 |

  > 快手、药明这两个本时代前段最大亏损，**不是判据看错方向，而是"判对了却卖不出去"**——结构翻转/止损信号都发了，卖单却被拒或挂到收盘过期。这类账在手动时代看不见（人手会立刻改价重下），只有自动化 7×24 才暴露。（两者 07-16 均已靠后续回合大幅收窄甚至翻正——见 §2。）

- **② 度量本身要对账【台账不可直接当业绩】** — 本地台账逐行盈亏记的是**提交限价**、还夹着重复/空行，与真实成交价回算的结果对不上。本时代因此把盈亏统一改用 **`reconcile_pnl.js` 按 SDK 真实成交回算**：**面板/台账只能当底稿，公开的收益数字必须用真实成交价重算。**

- **③ 常驻进程的状态一致性【07-15 补：四修 + 一条自造的坑】** — 让机器 7×24 跑、且盘中反复热更新，暴露出一批"进程重启 / 隔夜残留 / 判据抖动"的状态账。**当天没动任何判据**，全在执行层修补：

  1. **隔夜残单跨日成交** — Day 限价单当日没成交又没撤，挂到次日开盘一次性撮合：快手27786 07-15 09:32 三笔同价成交，实为昨日 16:01 止损 + 16:15 进场两笔隔夜残单 + 当日进场单同时成交。→ **收盘/退出即撤未成交单 + 启动清隔夜残单**（只撤本项目 remark 的单、仅模拟盘，绝不碰手动/真实单）。
  2. **进程重启"失忆"批量追高** — 去重/冷却是内存态、重启即清零；加上进场原是"价格 ≥ trigger"的**电平**判断，重启一上来把所有"现价已在 trigger 上方"的标的一次性全买（追高）。→ **进场改「trigger 一次性」**：每个 trigger 只打一次，须等慢层重新分析更新该标的 trigger 才复活；用当日成交记录恢复状态，重启不忘。改后重启只出合理离场、**零重复进场**。（07-15 开盘那笔美团29665 −979 正是修复前的追高。）
  3. **结构判据阈值抖动"刚卖又买"** — 中兴13599 因资金回撤比 dd=0.52 越过 0.5 判「翻转确认」于 14:41 卖出，14:47 dd 回落判「翻转预警」、进场闸松开又买回，同价同量、−360。→ 由 ②·2 的"trigger 一次性"一并覆盖（同 trigger 不重进）。
  4. **⚠️ 一条自己造的坑（诚实记录）** — ① 的"退出撤单"会把**未成交的止盈单也一并撤掉**；07-15 为上线补丁一天内重启了 **5 次**，每次都撤掉当时挂着的止盈/止损单——对账里那 10+ 笔"已撤单"平仓单就是这个副作用，被机器误标成"券商平仓失败"（也是 ① 的 44 笔里的一部分）。**教训：撤单要区分"隔夜残留"与"当日在途的合法离场单"；盘中热修代价高，非紧急改动应收盘后部署。**
  5. **手动 App 操作已可被感知同步** — 订单回报订阅收全账户单，人在 App 手动止盈止损 → 自动重拉持仓、失效盈亏缓存、补记台账、推飞书。人机同池不再脱节。

- **④ 离场信号的盘外护栏【07-16 补，已修】** — 离场信号（止损/止盈/结构翻转）原本**没有交易时段护栏**（只有进场有）。07-16 16:01 连续竞价已收盘（收盘竞价 CAS 时段），正股收盘竞价参考价跌破止损，快手27786、洪桥15159 被自动挂出止损卖单；手动停机撤单后，保活脚本 16:05 又把进程拉起，对**仍破位的止损重新开卖**，被交易所直接**拒单**。模拟盘全部未成交、无损失，但属逻辑漏洞——收盘竞价/盘后普通限价单本就会被拒、隔夜也无法成交。→ **修复：executor 统一「盘外自动离场只告警不下单」时段闸（09:30–12:00 / 13:00–16:00 之外命中止损/止盈/翻转只推飞书、不下单，下一交易日重评；手动平仓不受限）+ 自停时刻对齐 16:00（boot 已过则退出码 3 下线，保活脚本不再重启，杜绝 16:00–16:15 裸奔窗口与重启重下）**，新增 `test/executor_offhours` 回归。**这是 §3① 的收盘侧镜像：① 是"想平却平不掉"，④ 是"收盘后不该平却还在挂"——同一执行层，两个方向的漏洞。**

> **一句话认知（跨两个时代）**：手动时代证明了"离场纪律 > 进场功课"；守护进程时代补上后半句——**当你把纪律交给机器 7×24 执行，胜负手在"判断对不对"之外又多了两层："单子平不平得掉"、"进程重启/隔夜/盘外后状态与动作一不一致"。** 22 个标的 13 盈 7 亏 2 平，靠小米/阿里/快手/美团几个大回合把整段做到 **+15,373、07-15 转正、07-16 拉开**；但同期也逐条暴露了追高、卡单、自撤止盈、盘外下单这些执行漏洞。**规模化不改判据的对错，只放大判据之外的每一个漏洞——把漏洞逐条堵成规则，就是台账的产出。**

---

## 4. 收益汇总（截止 2026-07-16 收盘）

**两个时代（同口径：SDK 真实成交价×量、不计费）**

| 时代 | 已实现 | 关键指标 |
|---|---|---|
| 手动（06-17→07-02） | **−4,145 HKD** | 6 腿，成本基数 37,310，≈ −11.1% |
| 守护进程（07-03→07-16） | **+15,373 HKD** | 65 回合，22 标的 13 盈/7 亏/2 平；含 07-15 单日 +7,285、07-16 单日 +9,680 |
| **合计** | **+11,228 HKD（转正）** | |

- **当前持仓：** 12 个标的仍持有，合计浮动约 **−2,689 HKD**（未计入已实现；07-16 大量浮盈已兑现为已实现，剩余多为深度价外/被套持仓）。

**累计曲线（收盘）**：
手动段 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`；
守护进程段（叠加）07-02 `−4,145` → 07-14 `−5,737.50` → 07-15 `+1,547.50`（单日 +7,285，账户转正）→ **07-16 `+11,228`（单日 +9,680）**。

> 逐笔明细见项目内 `data/paper-trading/`：`2026-warrant-tracker.md`（逐笔底稿）、`pnl-reconciliation-*.md`（SDK 对账，含费净口径）、`weekly-pnl-*.md`（周复盘）。

---

## 5. 一句话总结

> 这张台账想证明的不是"赚了多少"，而是**框架在真金白银（模拟）的反馈里怎么进化**——而且进化分两级：
> **手动时代**用 −4,145 买回了 v1→v3.1 的判据（结论：离场纪律 > 进场功课）；
> **守护进程时代**把这套纪律交给机器 7×24 跑，买回四条运维账：**平仓单会平不掉、台账必须用真实成交回算、常驻进程重启/隔夜后状态要一致、离场信号要有盘外护栏**——并在 07-15 由累计负转正、07-16 拉开到 **+11,228 HKD**。
> **规模化和自动化不改判据的对错，只把判据之外的每一个漏洞放大给你看。亏损被转成了规则——这才是台账的产出。**

---
<a id="english"></a>
# Example · Paper-trading ledger (2026-06-17 → 07-16): from "a few hand trades" to a "7×24 daemon"

> **🌐 [中文 ↑](#示例--模拟盘收益台账2026-06-17--07-16从少量手动单到724-守护进程自动执行) · English (current)**
>
> ⚠️ **A paper-account (Longbridge) methodology walkthrough — NOT a track record, NOT a return promise, NOT investment advice.**
> This is not a "look at my gains" post; it lays a full paper-trading trajectory flat so you can see **which decision rationale each fill came from, which framework rule it validated or broke, and what P&L it landed at**.
>
> The trajectory spans two eras:
> - **Hand era (06-17 → 07-02)**: a few manual trades, human-triggered per session; the core topic is **exit discipline**. Realized **−HK$4,145**.
> - **Daemon era (07-03 → 07-16)**: `hk-market-daemon` went live — a fast-layer watcher polling prices per-second + auto-execution. Trade frequency exploded from 6 legs in two weeks to **65 close-rounds in 10 sessions**. This era laid bare, one by one, the costs that only appear at scale — **want-to-exit-but-can't, the ledger-vs-real-fill gap, long-running-process state consistency, and the missing off-hours gate on exits** — flipped from cumulative-negative to positive on a +7,285 gross day (07-15), then **added another +9,680 gross on 07-16** as the recovery continued (the era's former biggest loser, Kuaishou 27786, swung +5,383 in a single round), taking the daemon era to +15,373.
>
> **Through 2026-07-16 close:**
> - Cumulative realized **+HK$11,228** (hand −4,145 + daemon +15,373). The account turned positive on 07-15 and widened on 07-16 (daemon single day **+9,680 gross**; era close 07-15 +5,692.50 → 07-16 +15,373).
> - Daemon era: 65 closed rounds; of 22 names, **13 win / 7 loss / 2 flat**; total **+HK$15,373** (per-round win rate 28/37 ≈ 43%).
> - Currently still holding **12 names**, total floating ~**−HK$2,689** (most of the float was realized on 07-16; the remaining book is mostly deep-OTM / stuck positions and nets slightly negative).
>
> **Basis note**: throughout, this is **SDK real fill-price × qty, no fees** — gross realized (same basis across both eras, additive). For the fee-inclusive net basis per round, see [`2026-07-16-pnl-reconciliation.md`](./2026-07-16-pnl-reconciliation.md) (07-01→07-16: closed-round net +12,640.96, real realized +12,606.54, fees 2,732.04, 43.1% win); the 07-15 snapshot is [`2026-07-15-pnl-reconciliation.md`](./2026-07-15-pnl-reconciliation.md). Per-trade buys ≤ HK$10k; account all-cash, ~1M each HKD/USD/SGD. The local ledger's per-row P&L uses submit-limit prices (with duplicate/blank rows) and is not usable directly; all numbers here are recomputed from SDK real fills.

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

## 2. Daemon era — after going to scale (07-03 → 07-16)

> From 07-03 `hk-market-daemon` went live: a fast-layer watcher polls position prices per-second + polls large-money structure at high frequency, auto-executing on stop / target / structure-flip. The human stepped back from "click-confirm each trade" to "set rules, watch exceptions." Volume jumped an order of magnitude — 65 close-rounds in 10 sessions. **07-15's +7,285 gross day flipped the era from −1,592.50 (through 07-14) to +5,692.50; 07-16 added another +9,680 gross, taking it to +15,373. The brightest round: the era's former biggest loss source, Kuaishou 27786 — a single round +5,383 (+18.2%) that swung it from a cumulative −3,045 to +2,415.**
>
> Basis: SDK real fill-price × qty, no fees (a "round" = 0→position→0).

**By name (P&L desc, through 07-16)**

| Name (dir.) | Rounds | P&L HKD (no fee) | Avg hold |
|---|---|---|---|
| Xiaomi 14225 (call) | 7 | **+6,900.00** | 1d2h |
| BABA 14461 (call) | 1 | **+3,325.00** | 6d22h |
| Kuaishou 27786 (call) | 5 | **+2,415.00** | 1d19h |
| Meituan 29665 (call) | 5 | **+2,360.00** | 14h |
| ZTE 13599 (call) | 8 | **+1,540.00** | 13h |
| Innovent 29344 (call) | 8 | +820.00 | 18h |
| Zijin 13533 (call) | 1 | +700.00 | 1d21h |
| SMIC 23604 (call) | 1 | +700.00 | 1h51m |
| SMIC 29053 (call) | 1 | +495.00 | 3h53m |
| Hongqiao 15159 (call) | 3 | +450.00 | 7h12m |
| Pop Mart 14905 (call) | 1 | +368.00 | 3h24m |
| Ping An 14023 (call) | 1 | +200.00 | 4h30m |
| 13779 (call) | 2 | +95.00 | 1d15h |
| XPeng 14275 (call) | 1 | 0.00 | 3h |
| Kuaishou 29060 (put) | 1 | 0.00 | 18h |
| SMIC 21496 (put) | 2 | −40.00 | 10h35m |
| Zijin 13494 (call) | 2 | −100.00 | 4d23h |
| WuXi Bio 22574 (call) | 3 | −535.00 | 1d16h |
| WuXi AppTec 28701 (call) | 6 | −680.00 | 1d |
| BABA 13790 (call) | 1 | −780.00 | 3h57m |
| China Life 14673 (call) | 3 | −880.00 | 7h33m |
| Meituan 26790 (call) | 2 | **−1,980.00** | 3d1h |
| **Total (65 rounds)** | 65 | **+15,373.00** | avg ~1d3h |

> Symbols are **warrant/CBBC codes**; the same underlying may rotate codes across expiries/directions (Xiaomi=1810, BABA=9988, Kuaishou=1024, Meituan=3690, ZTE=763, Innovent=1801, Zijin=2899, SMIC=981, Hongqiao=1378, Pop Mart=9992, Ping An=2318, XPeng=9868, WuXi Bio=2269, WuXi AppTec=2359, China Life=2628). Across 22 names: **13 win / 7 loss / 2 flat**; by round 28 win / 37 loss ≈ 43%.

**Currently still holding (unrealized)**: 12 names, total floating ~**−HK$2,689** (most of the float was realized on 07-16; the remaining book is mostly deep-OTM / stuck positions and nets slightly negative).

**Representative rounds**

- **07-16, the continuation day** — +9,680 gross, led by **Kuaishou 27786 +5,383 (+18.2%)**: the era's former biggest loss source (cumulative −3,045), the 07-14 low-entry at 0.082 closed into today's bounce, and a single round flipped it to a cumulative +2,415 — **the "worst name" redeemed by one big round.** Other big rounds: **Xiaomi 14225 +3,221 (+33.5%)**, **WuXi Bio 22574 +1,665 (+16.7%, narrowing it from −2,237.50 to −535)**, **Zijin 13533 +681**; on the loss side ZTE 13599 −900, WuXi AppTec 28701 −559, BABA 13790 −780.
- **07-15, the turnaround day** — +7,285 gross: BABA 14461 +3,325 (an old leg held 6 days), Meituan 29665 +2,530, Innovent 29344 +1,361, ZTE 13599 +980; loss side Meituan 29665 −979 (an open chase) and China Life 14673 −739 (the former is §3 lesson ②2 "restart mass-chase" in the flesh).
- **Xiaomi 14225 (the era's biggest winning name, 7 rounds +6,900)** — entered accumulating from 07-10, structure held repeatedly, fish-body paid multiple times; the main force pulling the era up.
- **Kuaishou 27786 (+2,415 total, the most winding path)** — the era's biggest loss source early on (incl. single rounds −20%, −32%, and the 07-13 15:58 flip-exit that expired unfilled to the close, dragging into the next day — see §3 ①), yet flipped positive on a single big 07-16 round. **The same name is both §3①'s cautionary tale and the final net contributor.**
- **WuXi Bio 22574 (−535, was −2,237.50)** — deep loss after the 07-13 rejected stop (see §3 ①), then a +1,665 round on 07-16 closed most of the hole.

---

## 3. Cognitive-iteration axis — every framework version / ops lesson was bought with a loss

> The real output of a ledger is not the P&L number, it's the **rules**. The hand era's −4,145 was the tuition for v1→v3.1; the daemon era bought back **four accounts that only scale and automation expose** (① want-to-exit-but-can't ② measurement must be reconciled ③ long-running-process state consistency ④ the off-hours gate on exits). It flipped positive on 07-15 and, on 07-16, both continued the recovery and exposed-and-fixed the fourth account.

**Hand era: criterion framework v1 → v3.1**

- **v1 (06-24) · fish-body baseline** — direction first, size only the mid-body, judge the tail by structure + speed (never by magnitude), smart-money = reference not verdict, two independent gates.
- **v2 (06-25) · structure quantified** — a backtestable scale for distribution/accumulation: CLV-120d self-percentile (primary) + real-time large money (secondary). The metals cluster (−6~8%) was all vetoed as "catching the knife = accumulation" — the **anti-false-kill template**.
- **v3 (06-26) · exit discipline [most expensive lesson, −3,475]** — SMIC's two legs entered fully compliant yet bled from +7.64% into a loss. Root cause was the exit: **entered on the structure ruler, then swapped to a price ruler (78) to exit.** Codified R6 + L5.1 + L8.
- **v3.1 (06-29) · structure monitoring operationalized** — turned "flip → exit" into a **two-track poll + `structure_flip` machine criterion**. This version is exactly **what let the daemon auto-execute.**

**Daemon era: four "only-at-scale" ops accounts**

- **① Want-to-exit-but-can't [an execution-layer risk unique to automation]** — the reconcile flags **44 exit sell orders unfilled** cumulatively (rejected / limit unfilled to close / cancelled-on-restart / fired off-hours; the last two are the self-inflicted ③.4 and ④ below — genuine broker rejects + close-expiries are a subset): Kuaishou 27786, WuXi Bio 22574, Meituan 26790, BABA 14461 all hit "the ruler says exit, the order never fills, the position stays stuck." A human would instantly re-price and re-submit; an automated system may sit on the limit to the close. **The judgment was right; the execution layer didn't catch it, and it lost anyway.** (The executor now rolls back on failure + re-submits after cooldown.)

  **Record of the key failed exits (wanted out, couldn't get out)** — much of the two biggest early losers' bleed comes from exactly here:

  | Time (HKT) | Name | Exit trigger | Sell order | Broker result | Consequence |
  |---|---|---|---|---|---|
  | 2026-07-13 15:58 | Kuaishou 27786 | **confirmed flip** | sell 140,000@0.073 | expired unfilled | ruler had said exit; order sat to the close unfilled, position dragged into next day |
  | 2026-07-07 09:15 | Kuaishou 27786 | stop-loss | sell 65,000@0.098 | expired unfilled | stop never executed, the round's loss widened |
  | 2026-07-13 11:04 | WuXi 22574 | stop-loss | sell 87,500@0.088 | rejected | re-filled lower after the reject; the −2,237.50 includes this slippage |

  > Kuaishou and WuXi, the era's two biggest early losses, **weren't the criteria pointing the wrong way — they were "right call, couldn't sell."** The flip/stop signals both fired, but the sell orders were rejected or expired at the close. Invisible in the hand era (a human instantly re-prices); only 7×24 automation exposes it. (Both were sharply narrowed or flipped positive by later rounds on 07-16 — see §2.)

- **② Measurement itself must be reconciled [the ledger isn't a track record]** — the local ledger's per-row P&L uses **submit-limit prices** (with duplicate/blank rows) and doesn't match a recompute from real fills. So this era standardized P&L on **`reconcile_pnl.js`, recomputed from SDK real fills**: **the panel/ledger is only a source; any published number must be recomputed from real fill prices.**

- **③ Long-running-process state consistency [added 07-15: four fixes + one self-inflicted bug]** — running 7×24 *and* hot-patching intraday exposed a cluster of "restart / overnight-leftover / criterion-jitter" state accounts. **No criterion changed that day**; all fixes were execution-layer:

  1. **Stale day-orders fill across sessions** — unfilled, un-cancelled limit orders fill at the next open: Kuaishou 27786's three same-price fills on 07-15 09:32 were yesterday's 16:01 stop + 16:15 entry (two overnight leftovers) + that day's entry. → **cancel unfilled orders on close/shutdown + clean stale orders on startup** (only our own remark-tagged orders, paper only, never manual/live).
  2. **Restarts "forget" → mass-chase** — dedup/cooldown are in-memory, wiped on restart; entry was a *level* check (`price ≥ trigger`), so a restart bought everything already above its trigger at once. → **one-shot-per-trigger entry**: each trigger fires once, re-armed only when the slow layer updates that item's trigger; state recovered from today's fills so restarts don't forget. After the fix, a restart emits only legit exits, **zero re-entries** (07-15's open chase Meituan 29665 −979 was pre-fix).
  3. **Criterion jitter around a threshold → sell-then-rebuy** — ZTE 13599 flip-sold 14:41 (flow drawdown 0.52 > 0.5 → "confirmed"), re-bought 14:47 when it eased to "warning"; same price/qty, −360. → Covered by ②2's one-shot-per-trigger.
  4. **⚠️ A self-inflicted bug (logged honestly)** — ①'s "cancel-on-shutdown" also cancels *unfilled take-profit* orders; with 5 restarts on 07-15 to ship patches, each cancelled the resting exits — the 10+ "cancelled" close-orders in that report are this side-effect, mislabeled as "broker close failures" (part of ①'s 44). **Lesson: distinguish stale leftovers from legitimate in-flight exits; hot-patching intraday is costly — deploy after close unless urgent.**
  5. **Manual app actions are now sensed** — the order feed captures all account orders, so manual stop/take-profit → auto re-pull positions, invalidate P&L cache, mirror to the local ledger, push to Feishu.

- **④ The off-hours gate on exits [added 07-16, fixed]** — exit signals (stop / take-profit / structure-flip) originally had **no trading-hours gate** (only entry did). On 07-16 16:01, with continuous trading already closed (the CAS window), the underlying's closing-auction reference price broke the stop, so Kuaishou 27786 and Hongqiao 15159 got auto-submitted stop sells; after a manual stop cancelled them, the keep-alive relaunched the process at 16:05 and **re-fired the still-breached stops**, which the exchange **rejected outright**. In paper none filled — no loss — but it's a genuine logic hole: post-close/CAS plain limit orders get rejected and can't fill overnight anyway. → **Fix: a unified "off-hours auto-exit = alert only, no order" gate in the executor** (outside 09:30–12:00 / 13:00–16:00, a stop/take-profit/flip hit only pushes to Feishu and re-evaluates next session; manual liquidation is exempt) **+ auto-stop aligned to 16:00** (a boot past the stop time exits with code 3 so the keep-alive won't relaunch, killing the 16:00–16:15 free-run window and the restart-re-fire). Added `test/executor_offhours` regression. **This is §3①'s close-side mirror: ① is "want-to-exit-but-can't," ④ is "shouldn't-exit-after-close-but-still-placing" — same execution layer, two directions of the same hole.**

> **One-line takeaway (across both eras)**: the hand era proved "exit discipline > entry homework"; the daemon era added the back half — **once you hand the discipline to a machine running 7×24, the deciding factors grow by two more layers beyond "is the call right": "can you actually close the order," and "is state/action consistent across restarts / overnight / off-hours."** Across 22 names 13 win / 7 loss / 2 flat, a few big Xiaomi/BABA/Kuaishou/Meituan rounds took the era to +15,373 — positive on 07-15, widened on 07-16 — while the same span exposed, one by one, chasing, stuck orders, self-cancelled take-profits, and off-hours ordering. **Scale doesn't change whether the criteria are right — it magnifies every hole outside the criteria; each hole plugged becomes a rule — that is the ledger's output.**

---

## 4. P&L summary (through 2026-07-16 close)

**Two eras (same basis: SDK real fill-price × qty, no fees)**

| Era | Realized | Key metrics |
|---|---|---|
| Hand (06-17→07-02) | **−HK$4,145** | 6 legs, cost base 37,310, ≈ −11.1% |
| Daemon (07-03→07-16) | **+HK$15,373** | 65 rounds, 22 names 13 win/7 loss/2 flat; incl. 07-15's +7,285 and 07-16's +9,680 single days |
| **Total** | **+HK$11,228 (positive)** | |

- **Current holdings:** 12 names still held, total floating ~**−HK$2,689** (most of the float was realized on 07-16; the remaining book is mostly deep-OTM / stuck positions).

**Cumulative curve (close):**
Hand leg 06-24 `0` → 06-26 `−3,475` → 06-30 `−2,395` → 07-02 `−4,145`;
Daemon leg (stacked) 07-02 `−4,145` → 07-14 `−5,737.50` → 07-15 `+1,547.50` (a +7,285 day; account turned positive) → **07-16 `+11,228` (a +9,680 day)**.

> Per-fill detail in `data/paper-trading/`: `2026-warrant-tracker.md` (raw log), `pnl-reconciliation-*.md` (SDK reconcile, net basis), `weekly-pnl-*.md` (weekly review).

---

## 5. One line

> What this ledger tries to prove is not "how much was made", but **how the framework evolves under real (paper) money feedback** — and it evolved in two tiers:
> the **hand era** spent −4,145 to buy back the v1→v3.1 criteria (conclusion: exit discipline > entry homework);
> the **daemon era** handed that discipline to a machine running 7×24 and bought back four ops accounts: **exit orders can fail to fill, the ledger must be recomputed from real fills, a long-running process must stay state-consistent across restarts/overnight, and exits need an off-hours gate** — turning cumulative positive on 07-15 and widening to **+HK$11,228** on 07-16.
> **Scale and automation don't change whether the criteria are right — they magnify every hole outside the criteria. Losses were converted into rules — that is the ledger's true output.**
