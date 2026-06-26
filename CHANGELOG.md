# Changelog

> 语义：方法论框架的版本线（承接 `reference/framework.md` 内部既有 v1/v2）。每次「判断被真实数据反转 → 框架级修订」升一版。
> Semantics: the methodology version line (continues the v1/v2 inside `reference/framework.md`). Each "a call reversed by real data → framework-level revision" bumps a version.

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
