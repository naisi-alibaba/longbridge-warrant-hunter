# 示例 · 2026-06-29 全面 risk-on melt-up + 结构轮询落地

> **🌐 中文（当前） · [English ↓](#english)**
>
> ⚠️ **模拟盘方法论演示，不是业绩、不构成收益承诺。仓位结果按每日收盘更新（见第 3 节）。**
> 承接 [`2026-06-24-semis-cxo.md`](2026-06-24-semis-cxo.md)：中芯两腿已于 06-26 按结构尺离场（实现 −3,475）；本篇是新一轮 + 把 v3「离场看结构」落成可执行的**轮询 + 检测器**。

## 1. 盘面（盲扫结论）
- 全面 **risk-on melt-up**：HSI +1.57%、HSTECH +3.23%、温度 72 / 情绪 **83**（上周五情绪还 26，两天从冰点冲沸点）、成交 3,153 亿。
- 领涨：半导体（中芯 +6%、兆易 +14.6%、澜起 +6.6%）+ 互联网（阿里/腾讯/美团）+ 创新药 CXO（药明系、晶泰 +14%）。
- **主线之外**：PCB / 覆铜板 / 光纤逆势成片杀跌（建滔积层板 −2.3%、建滔集团 −5%、长飞 −4.5%）——去年大牛高位品种。

## 2. 双向矩阵裁定（数据先行）
| 标的 | 鱼段 | CLV 自我分位 / 大单结构 | 裁定 |
|---|---|---|---|
| 阿里 9988 | 认购**鱼头**（超跌反弹） | CLV 0.882（吸筹档）+ 大单净流入榜首、收盘 +5.9 亿 | ✅ 鱼头小试 |
| 药明 2269 | 认购鱼头→**升鱼中** | CLV 收盘 0.857（吸筹档）+ 大单 +1.83 亿碾压散户 | ✅ 续持（06-24 建仓） |
| 华虹 1347 | 上升趋势新高 | 冲 205→191 回落、盘中 CLV 0.28 中性 | ❌ 今日结构未确认 → 条件观察 |
| 澜起 6809 | — | 流通盘极小 + 窝轮 IV>144% 极值 | ❌ 经济性一票否决 |
| 建滔 148 / 长飞 6869 | 价格崩塌 | **大单整日不派发**（建滔大单净进、长飞大单净进散户割肉=吸筹向） | ❌ 背离否决，空仓等真派发 |

> **防错杀复刻**：长飞 −5.9%、建滔 −5.7% 看似完美认沽，但 `capital_distribution` 全天显示大单**不派发**（派发需大出散进，二者均无）→ 这是散户割肉/大单接刀，不是机构派发。跌幅再大、大单不派发就不做空（同 06-25 金属簇课）。

## 3. 行动盘 → 模拟成交 + 持仓跟踪（每笔 ≤1 万 HKD）

**本轮成交（模拟盘）**
| 时点 | 腿 | 动作 | 成交 × 量 | HKD | 备注 |
|---|---|---|---|---|---|
| 06-29 早 | 22574 药明价外证 | **减半止盈** | 0.074 × 45,000 | +3,330 | 触及 35 减半纪律；成本 2,475 → 实现 **+855 / +34.5%** |
| 06-29 早 | 14190 阿里认购 | 鱼头小试·建仓 | 0.110 × 50,000 | −5,500 | 限价不扫尖顶（gap 日窝轮已 +19%）；价格站稳 93 + 大单实时流入 = 进场扳机，CLV 0.882 仅软佐证 |

**收盘持仓（持有过夜）**
| 腿 | 量 | 成本 | 收盘 | 浮动 | 止损 / 止盈 |
|---|---|---|---|---|---|
| 22574 药明 | 45,000 | 0.055 | ~0.076 | **+945** | 收 <31 或大单转派发 / 38 全平 |
| 14190 阿里 | 50,000 | 0.110 | ~0.105 | −250 | 阿里收 <91 / 100 减半·105 全平 |

**收益对账**：今日实现 **+855**（药明减半）；累计实现 **−2,620**（含中芯两腿 −3,475 + 药明 +855）。结构轮询全天 ~12 次，**零翻转 / 零触发 / 零额外操作**，两腿全程 standing、尾盘转强。

## 4. 本日真正的方法论产出：把「离场看结构」落成可执行（v3 → v3.1）

v3 的结论是「结构翻转就离场」，但留了一个洞：**价格能用原生预警 push（秒级），结构（大单 / capital_flow）长桥无 push、大单无历史——你怎么及时看见翻转？** 06-26 中芯就是死在「结构 06-25 已翻、但没机器判据、退回手判 → 拖到 06-26 价格破位才走、牛证多亏到 −25%」。

**两条独立轨道**（落地）：
- **轨 1 · 持仓结构轮询（每 30 分钟，只看持仓）**：逐腿拉 `capital_flow` + `capital_distribution` → 跑结构翻转检测器 → 翻转确认 / 触及止损止盈即动。
- **轨 2 · 全局找新标的（每日 3 档）**：盲扫双向找新机会，临近收盘做 CLV 收盘定案。

**结构翻转检测器**（机器判据，消手判漂移；与 CLV 自我分位检测器互补）：
- 大单净额 = large_in − large_out；散户净额 = small_in − small_out。结构：吸筹=大单+ & 散户− / 派发=大单− & 散户+。
- `capital_flow` 斜率 = 唯一能**先于价格**翻的实时量：累计净流入**见顶回落**（回撤比 > 阈值且斜率转负）或转净流出 = flow 恶化。
- **翻转确认 = 大单方向翻转 ∩ flow 恶化（双证）→ 当日减/平**；单证 = 翻转预警、收紧复核；都不中 = 持有。
- 数据真相：大单/散户**仅当天、无历史、`quant_run` 不可用** → 不可回测，只能 live；CLV（OHLCV 派生）可回测，二者一快一慢、一不可回测一可回测。

**执行纪律澄清（本日自纠）**：**CLV 收盘定案永不作进场扳机**；进场扳机 = 价格（破位/突破/止损止盈，盘中实时）+ 当日大单 / capital_flow 实时方向，CLV 盘中只是**软佐证、不阻塞执行**。把「价格 + 实时资金已确认」的进场误降级成「CLV 类、收盘定案、盘中不开仓」= 过度保守、违背 L5/R7。阿里腿正是按价格 + 资金实时扳机盘中建的仓。

→ **沉淀为 v3.1**（详见 [`CHANGELOG.md`](../CHANGELOG.md)）。**一句话：v3 让你「该卖时卖」，v3.1 让你「及时看见该卖的信号」——结构没有 push，就主动轮询 + 机器判据。**

---
<a id="english"></a>
# Example · 2026-06-29 Broad risk-on melt-up + structure polling operationalized

> **🌐 [中文 ↑](#示例--2026-06-29-全面-risk-on-melt-up--结构轮询落地) · English (current)**
>
> ⚠️ **A paper-account methodology walkthrough — NOT a track record or return promise. Position results are updated after each daily close (section 3).**
> Continues [`2026-06-24-semis-cxo.md`](2026-06-24-semis-cxo.md): the SMIC legs were exited on 06-26 on the structure ruler (realized −3,475). This is a new cycle + turning v3's "exit on structure" into an executable **poll + detector**.

## 1. Tape (blind-scan conclusion)
- Broad **risk-on melt-up**: HSI +1.57%, HSTECH +3.23%, temperature 72 / sentiment **83** (was 26 last Friday — two days from freezing to boiling), turnover HK$315bn.
- Leadership: semis (SMIC +6%, GigaDevice +14.6%, Montage +6.6%) + internet (BABA/Tencent/Meituan) + CXO/pharma (WuXi names, XtalPi +14%).
- **Outside the main line**: PCB / copper-clad-laminate / optical-fiber sold off as a block against the tape (Kingboard Laminates −2.3%, Kingboard 148 −5%, YOFC −4.5%) — last year's high-flyers.

## 2. Two-way matrix verdict (data first)
| Name | Fish segment | CLV self-pct / money structure | Verdict |
|---|---|---|---|
| BABA 9988 | Call **head** (oversold bounce) | CLV 0.882 (accum.) + large money #1 inflow, +HK$5.9bn at close | ✅ Small taste |
| WuXi Bio 2269 | Call head→**mid** | Closing CLV 0.857 (accum.) + large +HK$1.83bn dwarfing retail | ✅ Hold (entered 06-24) |
| Hua Hong 1347 | Uptrend new high | Spiked 205→191, intraday CLV 0.28 neutral | ❌ Today unconfirmed → conditional watch |
| Montage 6809 | — | Tiny float + warrant IV >144% extreme | ❌ Economics veto |
| Kingboard 148 / YOFC 6869 | Price collapse | **Large money never distributes all day** (large net-in, retail capitulating = accumulation-leaning) | ❌ Divergence veto, stay flat, wait for real distribution |

> **No-false-kill replay**: YOFC −5.9%, Kingboard −5.7% look like textbook puts, but `capital_distribution` shows large money **not distributing** all day (distribution needs large-out & retail-in — neither present) → this is retail capitulation / large absorbing, not institutional distribution. However large the drop, don't short while large money isn't distributing (same as the 06-25 metals lesson).

## 3. Action board → paper fills + position tracking (≤HK$10k each)

**Fills (paper)**
| When | Leg | Action | Fill × Qty | HKD | Note |
|---|---|---|---|---|---|
| 06-29 a.m. | 22574 WuXi OTM call | **Trim half (target)** | 0.074 × 45,000 | +3,330 | hit the 35 trim rule; cost 2,475 → realized **+855 / +34.5%** |
| 06-29 a.m. | 14190 BABA call | Head small taste — open | 0.110 × 50,000 | −5,500 | limit, no sweeping the tip (warrant already +19% on a gap day); BABA holding 93 + real-time large inflow = entry trigger, CLV 0.882 only soft support |

**Close positions (held overnight)**
| Leg | Qty | Cost | Close | Unreal. | Stop / Target |
|---|---|---|---|---|---|
| 22574 WuXi | 45,000 | 0.055 | ~0.076 | **+945** | close <31 or large flips to distribute / 38 all-out |
| 14190 BABA | 50,000 | 0.110 | ~0.105 | −250 | BABA close <91 / trim 100·all-out 105 |

**P&L reconciliation**: realized today **+855** (WuXi trim); cumulative realized **−2,620** (SMIC two legs −3,475 + WuXi +855). Structure poll ran ~12× all day — **zero flips / zero triggers / zero extra actions**; both legs stood all day and strengthened into the close.

## 4. The day's real methodology output: making "exit on structure" executable (v3 → v3.1)

v3 concluded "flip in structure = exit", but left a hole: **price has a native push alert (sub-second); structure (large money / capital_flow) has no push on Longbridge and large-money has no history — how do you see the flip in time?** SMIC died on exactly this on 06-26: structure had flipped on 06-25, but with no machine criterion it fell back to hand-judgment → held until price broke on 06-26, the CBBC down −25%.

**Two independent tracks** (now live):
- **Track 1 · Position structure poll (every 30 min, holdings only)**: per leg pull `capital_flow` + `capital_distribution` → run the flip detector → act on a confirmed flip / stop / target.
- **Track 2 · Global new-target scan (3× daily)**: blind-scan both ways for new setups; near the close, settle CLV close-marks.

**Structure-flip detector** (machine criterion, kills hand-judgment drift; complements the CLV self-percentile detector):
- large-net = large_in − large_out; retail-net = small_in − small_out. Structure: accumulation = large+ & retail− / distribution = large− & retail+.
- `capital_flow` slope = the only real-time quantity that flips **before price**: cumulative net inflow **peaking and rolling over** (drawdown ratio > threshold with negative slope) or turning net-out = flow deteriorating.
- **Confirmed flip = large-money direction flip ∩ flow deterioration (two proofs) → trim/cut that day**; one proof = early warning, tighten & re-check; neither = hold.
- Data reality: large/retail structure is **same-day only, no history, `quant_run` unavailable** → not backtestable, live only; CLV (OHLCV-derived) is backtestable — one fast/one slow, one un-backtestable/one backtestable.

**Execution-discipline clarification (self-correction today)**: **a CLV close-mark is never an entry trigger**; the entry trigger = price (break/breakout/stop/target, real-time) + same-day large-money / capital_flow direction; intraday CLV is only **soft support, never a block**. Downgrading a "price + real-time money already confirmed" entry into "a CLV-class signal, settle at close, don't open intraday" = over-conservative, against L5/R7. The BABA leg was opened intraday precisely on the price + money real-time trigger.

→ **Codified as v3.1** (see [`CHANGELOG.md`](../CHANGELOG.md)). **One line: v3 makes you "sell when you should"; v3.1 makes you "see the sell signal in time" — structure has no push, so poll it actively with a machine criterion.**
