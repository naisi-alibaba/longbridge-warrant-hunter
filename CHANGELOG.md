# Changelog

> 语义：方法论框架的版本线（承接 `reference/framework.md` 内部既有 v1/v2）。每次「判断被真实数据反转 → 框架级修订」升一版。
> Semantics: the methodology version line (continues the v1/v2 inside `reference/framework.md`). Each "a call reversed by real data → framework-level revision" bumps a version.

---

## v3.1 — 2026-06-29 · 结构监控落地（Structure monitoring operationalized）

**触发 / Trigger**：不是判断被反转，而是 v3 留下的执行洞——v3 说「结构翻转就离场」，但**价格能用原生预警 push，结构（大单/capital_flow）长桥无 push、大单无历史**，手判会漂移、会迟到（06-26 中芯即如此）。本版把 v3 的判断**落成可执行的轮询 + 机器判据**。
Not a reversed call — an execution gap left by v3. v3 said "flip in structure = exit", but price has a native push alert while structure has none on Longbridge (large-money also has no history); hand-judgment drifts and lags (the 06-26 SMIC case). This release turns v3's judgment into an executable poll + machine criterion.

**方法论更新 / What changed**

1. **双轨监控 / two-track monitoring**：① 持仓结构轮询（每 30 分钟，只看持仓，逐腿跑翻转检测器 + 价格 vs 止损止盈）；② 全局找新标的（每日 3 档，盲扫双向 + 临近收盘 CLV 定案）。结构监控**只能主动轮询**——这是数据现实，不是选择。
   Two tracks: (1) position structure poll every 30 min (holdings only, run the flip detector + price-vs-stop/target per leg); (2) global new-target scan 3× daily. Structure can **only** be polled actively — a data reality, not a choice.

2. **结构翻转检测器 / structure-flip detector**（机器判据，配 CLV 自我分位检测器）：大单净额方向 + `capital_flow` 斜率/回撤双证。`capital_flow` 斜率是唯一能**先于价格**翻的实时量（累计净流入见顶回落 = flow 恶化）。**翻转确认 = 大单方向翻转 ∩ flow 恶化 → 当日减/平**；单证 = 预警。大单/散户**仅当天、无历史、不可回测** → live 用；CLV（OHLCV 派生）可回测——一快一慢、互补。
   A machine criterion (pairs with the CLV self-percentile detector): large-money direction + `capital_flow` slope/drawdown, two proofs. The slope is the only real-time quantity that flips **before price**. Confirmed flip = large-money direction flip ∩ flow deterioration → cut that day; one proof = warning. Large/retail is same-day, no history, not backtestable (live only); CLV is backtestable — fast vs slow, complementary.

3. **进场扳机澄清 / entry-trigger clarification**：**CLV 收盘定案永不作进场扳机**（呼应 L5/R7）；进场扳机 = 价格 + 当日大单/capital_flow 实时方向，CLV 盘中只软佐证、不阻塞。把「价格+实时资金已确认」误降级成「CLV 类·收盘定案·盘中不开仓」= 过度保守。
   A CLV close-mark is **never** an entry trigger (per L5/R7); the trigger is price + same-day large-money/capital_flow, intraday CLV only soft support. Downgrading a confirmed price+money entry into "CLV-class, settle at close, no intraday open" = over-conservative.

**与 v3 的关系 / Relation to v3**：v3 是扳机（该卖时卖），v3.1 是探照灯（及时看见该卖的信号）。无判断被反转 → **点版本（.1），非升大版**。
v3 is the trigger (sell when you should); v3.1 is the searchlight (see the signal in time). No call was reversed → a **point release (.1)**, not a major bump.

**实战 / Worked example**：[`examples/2026-06-29-meltup-structure-poll.md`](examples/2026-06-29-meltup-structure-poll.md) — melt-up 全日，结构轮询 ~12 次零翻转，药明减半 +855 / 累计实现 −2,620。

**首次 live 验证（双向都对）/ first live validation (both directions)**：[`examples/2026-06-30-structure-poll-live.md`](examples/2026-06-30-structure-poll-live.md) — 阿里双证翻转确认 → 先于价格止损离场(−8.2% vs 等价格约−12~15%)；中芯午后回撤大单仍吸筹 → 判"持有"未误杀、收回成本；药明单证连续3轮不升级 → 不动。该走时走、该忍时忍、噪声不乱动。
First live validation (both directions): BABA two-proof confirmed flip → exit ahead of the price stop (−8.2% vs ~−12~15% waiting for price); SMIC afternoon pullback with large money still accumulating → "HOLD", no false-kill, recovered to cost; WuXi one-proof warnings ×3 didn't escalate → no action.

---

## v3 — 2026-06-26 · 离场纪律（Exit discipline）

**触发 / Trigger**：模拟盘中芯三腿。进场完全合规（06-24 大单 +26 亿吸筹、突破中段），却仍从 +7.64% 回吐成浮亏——根因不在进场、在离场。详见 [`examples/2026-06-24-semis-cxo.md`](examples/2026-06-24-semis-cxo.md) 第五节。
The SMIC three-leg paper trade: a fully compliant entry (06-24, large money +HK$2.6bn accumulating, mid-body breakout) still bled from +7.64% into a loss — the root cause was the exit, not the entry. See section 5 of the example.

**方法论更新 / What changed**

1. **L5 新增「离场端结构再筛（与进场同尺）」/ new "exit-side structure re-screen (same ruler as entry)"**
   - 进场用资金结构尺，离场也必须用同一把尺：结构反向翻转（认购：大单净进→净出 + 散户净出→净进）= 一级离场信号，当日减/平，**不得**因「价格没破远端止损」降级为「观察持有」。价格止损只是兜底。
   - Enter on structure, exit on structure: a reverse structure flip is a first-class exit signal — cut that day; do NOT downgrade it to "watch and hold" just because price hasn't hit a far stop. The price stop is only a backstop.

2. **L8 补丁「止损位按杠杆折算，不抄正股技术位」/ patch "size the stop by leverage, don't copy the stock level"**
   - 止损次序反过来：先定可承受的权证回撤（−10~12%）÷ 有效杠杆 = 标的容忍幅度 → 标的止损位。直接抄正股颈线/平台会被杠杆放大数倍（标的 −8% × 3x ≈ 权证 −24%）。
   - Reverse the order: bearable warrant drawdown ÷ effective leverage = the underlying's tolerance → the stop. Copying the stock's technical level lets leverage multiply it several-fold.

**与 v2 的差异 / Diff vs v2**：v2 解决「鱼尾/派发**有没有**发生」（CLV 自我分位量化）；v3 解决「派发**发生后怎么离场**」——v2 把信号看准了，v3 才把信号**执行成动作**。两者互补：v2 是探测器，v3 是扳机。
v2 answered "*did* a tail/distribution happen" (CLV self-percentile); v3 answers "*how to exit once it has*" — v2 reads the signal, v3 pulls the trigger. Complementary: v2 the detector, v3 the trigger.

**归因 / Attribution**：(c) 框架错误——进场判据（结构）与离场判据（价格 78）不一致。对应分析规则集新增 **R6 判据一致性**（仅本地 EOD 项目侧）。
(c) framework error — entry criterion (structure) and exit criterion (price 78) were inconsistent.

---

## v2 — 2026-06-25 · 结构量化（基线 / baseline）

新增 L4+「CLV 自我分位（主）+ 实时大单（辅）」，给「大单**大幅**派发/吸筹」一把可自校准、有历史的刻度（大单数据仅当日、无历史 → 用 OHLCV 派生的 CLV 自我分位回测）。
Added L4+ "CLV self-percentile (primary) + real-time large orders (secondary)", giving "*heavy* distribution/accumulation" a self-calibrating, historied scale.

## v1 — 2026-06-24 · 首个公开版 / first public release

L0–L9 完整框架：内核 → 鱼身定位 → 鱼尾结构判据 → 资金=参考非裁决 → 进出场纪律 → 双向不对称 → 两道独立闸（鱼身 ∩ 经济性）→ 数据现实。含「华虹错杀」「主力=参考」两处实战修正。
The full L0–L9 framework, with the "Hua Hong false-kill" and "smart-money-is-a-reference" field corrections baked in.
