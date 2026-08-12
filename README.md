# Longbridge Warrant Hunter · 港股窝轮盈亏比猎手

**🌐 中文（当前） · [English](README.en.md)**

> A Claude **Agent Skill** that hunts high risk-reward Hong Kong warrants (窝轮, calls/puts), long & short, on top of the **Longbridge OpenAPI MCP**. Direction-first, evidence-driven — not a news summarizer. Plain warrants only, no CBBCs.

一个基于 **Longbridge OpenAPI MCP** 的 Claude 技能：在港股**窝轮（认购/认沽）**里**多空双向**找盈亏比高的机会。核心是一套可证伪的判据——**先定方向，再选结构**。**只做窝轮、不做牛熊证（CBBC）**：强制收回触价即时归零、风险不可控。

> 📌 **当前版本 v5（2026-08-12）** — 判据从"资金结构+CLV"扩为**多维证据体系**：CLV 实证降级（弱均值回归，永不作进场扳机）、新增进场三维证据投票 / 抗操纵两维 / 对抗式独立盲审 / 结论使用纪律；工作流全面改写为**严格编号步骤**（步骤 0-9，每步七字段）。版本差异与方法论演进见 [`CHANGELOG.md`](CHANGELOG.md)。

---

## ✨ 它做什么

按严格步骤序列跑（每步含前置条件/调用/判据/否决点，详见 `reference/workflow.md`）：

```
步骤0 入口路由 → 步骤1 盲扫锁异常 → 步骤2 收窄到有轮票池
   → 步骤3 双向鱼身定位(七维立体判位) → 步骤4 进场三维证据投票(≥2维正向才进)
   → 步骤5 选券(选券三问·第二道闸) → 步骤6 盈亏比+止损折算
   → 步骤7 对抗式独立盲审 → 步骤8 出「行动盘」+盘口确认 → 步骤9 持仓监控离场
```

**核心思想（鱼身定位）**：只重仓"鱼中段"（确认后的趋势中段）、轻仓"鱼头"（刚起涨/起跌）、绝不碰"鱼尾"（抛物线顶/瀑布底）。**双向对称**——对一只已跌很多的票买认沽，等同于追抛物线顶买认购，是同一个错。鱼尾按**资金结构翻转 ∩ 抛物线速度**双证判，**绝不按涨跌幅枪毙**。

## 📦 安装

> 仓库根目录即技能本体（`SKILL.md` + `reference/`）。注意：**技能目录名 = 斜杠命令**，所以克隆时目标文件夹请命名为 `longbridge-warrant-hunter`。装好后**实时检测、无需重启**（若你机器此前从未有过 `~/.claude/skills/` 目录，首次需重启 Claude Code 一次）。

### 方式 A · 克隆到技能目录（最稳，推荐）

```bash
# 个人 / 全局（所有项目可用）
git clone https://github.com/naisi-alibaba/longbridge-warrant-hunter.git ~/.claude/skills/longbridge-warrant-hunter

# 或仅当前项目
git clone https://github.com/naisi-alibaba/longbridge-warrant-hunter.git .claude/skills/longbridge-warrant-hunter
```

### 方式 B · 插件市场一键装

在 Claude Code 里：

```text
/plugin marketplace add naisi-alibaba/longbridge-warrant-hunter
/plugin install longbridge-warrant-hunter@longbridge-warrant-hunter
/reload-plugins
```

之后以 `/longbridge-warrant-hunter` 调用（依赖本仓库的 `.claude-plugin/marketplace.json`）。

### 方式 C · 只取文件、不带 git 历史

```bash
npx degit naisi-alibaba/longbridge-warrant-hunter ~/.claude/skills/longbridge-warrant-hunter
```

装好后用 `/longbridge-warrant-hunter` 触发，或直接描述需求让 Claude 按 `description` 自动调用。

## 🔌 前置依赖

- **Claude Code / 支持 Agent Skills 的 Claude 客户端**
- **Longbridge OpenAPI MCP**（数据来源唯一，必须连接）——提供行情、市场温度、行业排名、资金流/资金分布、权证链、盘口、（可选）下单与到价提醒等工具。
  - 申请 Longbridge OpenAPI 凭证并配置 MCP：参见 Longbridge 官方文档。
  - 工具名以你的 MCP 配置为准（可能带前缀）。

## 🚀 用法

在对话里触发（或按你客户端的技能调用方式）：

- "港股有没有盈亏比好的认购/认沽窝轮" → 全市场盲扫双向
- "帮我在 中芯国际 上选只窝轮" → 单标的鱼身定位 + 选轮
- "现在做认购还是认沽，挂哪只轮，止损止盈怎么定"

技能会读 `reference/framework.md`（判据）+ `reference/workflow.md`（步骤）执行。

## 🎛️ 可调参数（自定义你的策略）

本技能的判据里带 **☆** 标记的都是**经验阈值、可自行覆写**——它们不是定论，是可执行的默认值。想更激进或更保守，改这些数字即可。用法：**在你的指令里直接说明覆写值**（例如"把盈亏比门槛提到 2.0、主动买阈值放宽到 60%"），或在本地 fork 里改 `reference/framework.md` / `reference/workflow.md` 对应行。带 **★** 的是有机制/实证支撑的硬阈值，不建议随意改。

| 参数 | 默认值(☆) | 含义 / 作用 | 调大 = | 调小 = | 出处 |
|---|---|---|---|---|---|
| **盈亏比门槛 RR** | ≥ 1.5 | 低于此不出手 | 更挑剔、机会更少、单笔期望更高 | 更宽松、机会更多、胜率要求更高 | framework L8 / workflow 步骤6 |
| **鱼中段 ATR 边界** | 1~3×ATR | 从突破起行进多少算"鱼中"（核心仓区） | 允许更晚进场、追趋势 | 只做刚突破段、更早离场 | framework L2 |
| **抛物线鱼尾·涨幅分位** | ≥ 90 分位 | 判"冲刺过热"的自我分位线（★用分位不用绝对幅度） | 更少判鱼尾、容忍更热 | 更早判鱼尾、更保守 | framework L3 |
| **抛物线鱼尾·日收益 z** | ≥ 2.0 | 异常加速的 z-score 线 | 更少判鱼尾 | 更敏感 | framework L3 |
| **进场投票·主动买上线** | ≥ 55% | 逐笔维记 +1 的主买占比 | 要求更强买盘才算正向 | 更易凑正向证据 | framework L9 |
| **进场投票·主动买下线** | ≤ 45% | 逐笔维记 −1 的主买占比 | 更易判反向 | 更宽容 | framework L9 |
| **选券·深度价外线** | 价外 ≥ 25% | 超过此价外程度视为"彩票"淘汰（鱼头限亏除外） | 允许更价外、更博凸性 | 只做更贴价内、更稳 | framework L8 |
| **选券·价内 delta 线** | \|delta\| ≥ 0.5 | 鱼中优先的高 delta 界 | 只要极价内 | 放宽到平价 | framework L8 |
| **选券·回本安全边际** | 目标 ≥ 1.5×回本幅度 | 目标须超回本多少才算够安全 | 更严、要求更大空间 | 更宽、接受薄边际 | framework L8 |
| **选券·溢价上限** | ≤ 20% | 超过此溢价漏血严重、淘汰 | 容忍更高溢价 | 只做低溢价 | framework L8 |
| **止损·可承受窝轮回撤 D_w** | −10~12% | 折算标的止损的起点（÷实际杠杆得标的容忍幅度） | 止损更宽、扛更久 | 止损更紧、更快认错 | framework L8 |
| **认沽执行不对称** | 更紧止损 / 仓位降一级 | 做空侧的额外保守 | — | 关掉=完全对称 | framework L6 |
| **持仓结构轮询频率** | 每 30 分钟 | 抓结构翻转的轮询间隔 | 更省调用、更迟钝 | 更灵敏、更多调用 | workflow 步骤9 |

> **★ 硬阈值（不建议改）**：到期 ≥3 月（避 theta 悬崖）、街货 <50%（避发行商控盘）、鱼尾用自我分位而非绝对幅度、CLV 不作进场扳机、只做窝轮不做牛熊证。这些有机制或实证支撑，改动会破坏框架自洽。
>
> **改参数前先读 framework L12「结论使用纪律」**：这些默认值多在特定市场环境（震荡市）下经验得来，**趋势市/高波动环境未必最优**；且小样本只能论机制不能论收益。请用模拟盘验证你的覆写值。

## 📁 示例

- [📒 **模拟盘收益台账（2026-06-17 → 07-02）**](examples/2026-paper-trading-ledger.md) — **一张表看清「动作 · 决策理由 · 认知迭代 · 收益」**：完整轨迹、累计已实现 −4,145 HKD，讲清每笔亏损如何"买"来一版框架（v1→v3.1）。⚠️ 模拟盘演示，非业绩承诺。
- [2026-06-24 · 半导体领涨 + CXO 轮动](examples/2026-06-24-semis-cxo.md) — 一次完整跑动：进场合规却仍浮亏，**第五节「离场复盘」是 v3 离场纪律的活教材**（进场用结构尺、离场误用价格尺的代价）。

## 🧭 不可违背的铁律

- **CLV 永不作进场扳机**：实证为弱均值回归（高 CLV 次日倾向反跌），仅离场端软佐证。
- **进场要正向证据**：三维证据投票（结构/资金流/逐笔）≥2 维正向才进，**中性=不进**。
- 鱼尾按**结构翻转 ∩ 抛物线速度**双证判，**绝不按涨跌幅**；对"看起来涨/跌多"的票必须拉资金结构再裁。**错杀 = 框架有洞。**
- **选券别只看杠杆**：深度价外的"高杠杆"是彩票杠杆——先看价内外/回本/溢价（选券三问）。
- **抗操纵独立盲审**：出手前另起一趟盲审（看不到看多理由）按主力红旗清单复核，宁可漏杀不过度解读。
- **两道独立闸**：股票鱼身 ∩ 窝轮经济性（IV 非极值 / 到期≥3月 / 街货<50% / 价差可接受）。
- **主力是参考不是裁决**；已确认即动、犹豫放离场端；认沽止损更紧、仓位更小；矩阵决定输出、不凑对称。
- **结论使用纪律**：不可回测=可信度天花板；结论仅在当前市场环境内成立；小样本只论机制不论收益。

## ⚠️ 免责声明 / Disclaimer

本技能仅为基于公开行情的**研究分析工具，不构成任何投资建议**。港股窝轮为**高杠杆、可归零**的衍生品，风险极高（本技能不做牛熊证/CBBC——其强制收回风险不可控）。任何依据本技能输出做出的交易决策，盈亏与后果由使用者自负。**强烈建议先用模拟盘（paper trading）验证框架，再考虑实盘。** 作者与贡献者不对任何损失负责。

This skill is a research/analysis tool only and is **not investment advice**. HK warrants are high-leverage instruments that can go to zero. Plain warrants only — this skill does not trade CBBCs (uncontrollable mandatory-call risk). Use a paper account first. The authors accept no liability for any losses.

## 📄 License

MIT — see [LICENSE](LICENSE).
