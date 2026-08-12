---
name: longbridge-warrant-hunter
description: 港股窝轮（认购/认沽）盈亏比猎手，多空双向，基于 Longbridge OpenAPI MCP 的全市场行情与资金数据。用方向驱动的多维证据体系（鱼身定位 + 资金结构 + 逐笔 + 抗操纵 + 对抗复核）做窝轮择时与选券。当用户想在港股寻找盈亏比高的认购/认沽窝轮、或对某个标的做窝轮择时与选轮时使用。只做窝轮、不做牛熊证（CBBC 强制收回风险不可控）。需连接 Longbridge OpenAPI MCP；研究分析工具，非投资建议。 | EN: HK plain-warrant (calls/puts) risk-reward hunter, long & short, on the Longbridge OpenAPI MCP. Direction-driven, multi-evidence (fish-positioning + money structure + tick tape + anti-manipulation + adversarial review) warrant timing & selection. Use when the user wants high risk-reward HK call/put warrants, or warrant timing & selection on a specific name. Plain warrants only — no CBBCs (uncontrollable mandatory-call risk). Requires Longbridge OpenAPI MCP; research tool, not investment advice.
---

# 港股窝轮盈亏比猎手（HK Warrant Hunter）

多空双向，在港股**窝轮（认购/认沽）**里找**盈亏比高**的机会。核心不是播报，是按一套可证伪的判据**先定方向、再选结构**。**不做牛熊证（CBBC）**——强制收回机制触价即时归零、风险不可控，整体排除。

## 触发场景
- "港股有没有盈亏比好的认购/认沽窝轮"
- "帮我在 XXX（标的）上选只窝轮"
- "现在做认购还是认沽，挂哪只轮，止损止盈怎么定"

## 前置依赖
- 已连接 **Longbridge OpenAPI MCP**（行情、资金流、权证链、盘口、可选下单）。工具名以你的 MCP 配置为准（可能带前缀，如 `quote` / `industryRank` / `capitalDistribution` / `warrantList` / `warrantQuote` / `depth` / `submitOrder` 等）。
- 无 Longbridge MCP 则无法运行（数据来源唯一）。

## 怎么用 / How to use
1. **判据 / criteria**：读 `reference/framework.md`（中文）或 `reference/framework.en.md`（English）——判据定义 L0-L13 与"为什么"。
2. **执行序列 / steps**：严格按 `reference/workflow.md`（中文）或 `reference/workflow.en.md`（English）——步骤 0-9，每步七字段（目的/前置/调用/判据/否决点/输出/失败处理）。两版等价，按用户语言择一。实例见 `examples/`。
3. **入口路由**：IF 用户未指定标的 THEN 全市场盲扫双向找方向；IF 用户已指定标的 THEN 跳过盲扫、直接进该标的的鱼身定位与选券。
4. **可调参数**：策略里所有带 ☆ 的阈值（盈亏比门槛、鱼段边界、证据投票线、选券三问线等）都可自行覆写——集中清单见 [`README.md` 的「可调参数」](README.md)。

## 不可违背的铁律（详见 framework，均为否决点）
1. **只做窝轮认购/认沽，不做牛熊证（CBBC）**：强制收回触价即时归零，整体排除。
2. **CLV 永不作进场扳机**：实证为弱均值回归（高 CLV 次日倾向反跌），把它当"吸筹脚印"方向是反的；CLV 仅离场端软佐证（见 framework L4）。
3. **进场要正向证据，不是"没反向就进"**：三维证据投票（结构/资金流/逐笔）≥2 维正向才进，中性=不进（见 framework L9）。
4. **鱼尾按「结构翻转 ∩ 抛物线速度」双证判，绝不按涨跌幅枪毙**；缺速度证据不得判鱼尾（见 framework L3）。
5. **两道独立闸**：股票鱼身 ∩ 窝轮经济性（IV 非极值 / 到期≥3月 / 街货<50% / 价差可接受），两闸全过才出手。
6. **选券别只看杠杆**：深度价外证的"高有效杠杆"是低价分母灌出的彩票杠杆——先看价内外、回本点、溢价（选券三问，见 framework L8）。
7. **抗操纵独立盲审**：出手前另起一趟盲审（看不到看多理由）按主力红旗清单复核，宁可漏杀不过度解读（见 framework L11）。
8. **主力是参考不是裁决**；已确认即动、犹豫放离场端；认沽止损更紧、仓位下调一级；矩阵决定输出、不凑对称。
9. **结论使用纪律**：不可回测=可信度天花板；结论仅在当前市场环境内成立；小样本只论机制不论收益（见 framework L12）。

## 输出
一张**行动盘**：标的 × 鱼段 × 工具 × 盈亏比 × 持仓周期 × 触发 × 止损 × 仓位。

> ⚠️ 免责：本技能输出为基于公开行情的研究分析，**不构成任何投资建议**。窝轮为高杠杆、可归零的衍生品，盈亏自负。强烈建议先用模拟盘验证框架再考虑实盘。
