# Longbridge Warrant Hunter · 港股权证盈亏比猎手

**🌐 中文（当前） · [English](README.en.md)**

> A Claude **Agent Skill** that hunts high risk-reward Hong Kong warrants (窝轮) and CBBCs (牛熊证), long & short, on top of the **Longbridge OpenAPI MCP**. Direction-first, evidence-driven — not a news summarizer.

一个基于 **Longbridge OpenAPI MCP** 的 Claude 技能：在港股权证（窝轮 + 牛熊证）里**多空双向**找盈亏比高的机会。核心是一套可证伪的判据——**先定方向，再选结构**。

> 📌 **当前版本 v3（2026-06-26）** — 新增「离场端结构再筛」+「止损按杠杆折算」。版本差异与方法论演进见 [`CHANGELOG.md`](CHANGELOG.md)。

---

## ✨ 它做什么

按固定工作流跑：

```
盲扫全市场锁异常 → 双向鱼身定位(头/中/尾) → 资金结构验证(防错杀)
   → 筛权证(IV/杠杆/到期/街货/价差两道闸) → 量化盈亏比×持仓周期
   → 盘口确认 → 出「行动盘」(标的×鱼段×工具×盈亏比×周期×触发×止损×仓位)
```

**核心思想（鱼身定位）**：只重仓"鱼中段"（确认后的趋势中段）、轻仓"鱼头"（刚起涨/起跌）、绝不碰"鱼尾"（抛物线顶/瀑布底）。**双向对称**——对一只已跌很多的票买认沽，等同于追抛物线顶买认购，是同一个错。鱼尾按**资金结构翻转 + 速度**判，**绝不按涨跌幅枪毙**。

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

- "港股有没有盈亏比好的认购/认沽权证" → 全市场盲扫双向
- "帮我在 中芯国际 上选只窝轮/牛熊证" → 单标的鱼身定位 + 选轮
- "现在做认购还是认沽，挂哪只轮，止损止盈怎么定"

技能会读 `reference/framework.md`（判据）+ `reference/workflow.md`（步骤）执行。

## 📁 示例

- [2026-06-24 · 半导体领涨 + CXO 轮动](examples/2026-06-24-semis-cxo.md) — 一次完整跑动：进场合规却仍浮亏，**第五节「离场复盘」是 v3 离场纪律的活教材**（进场用结构尺、离场误用价格尺的代价）。

## 🧭 不可违背的铁律

- 鱼尾按**结构翻转 + 抛物线速度**判，**绝不按涨跌幅**；对"看起来涨/跌多"的票必须拉资金结构再裁。**错杀 = 框架有洞。**
- **主力是参考不是裁决**：资金只确认/证伪方向，强信号也带止损。
- **两道独立闸**：股票鱼身 ∩ 权证经济性（IV 非极值 / 到期≥3月 / 街货<50% / 价差可接受）。
- **已确认即动，犹豫放离场端**；认沽止损更紧、仓位更小。
- **矩阵决定输出**，不凑认购/认沽对称。

## ⚠️ 免责声明 / Disclaimer

本技能仅为基于公开行情的**研究分析工具，不构成任何投资建议**。港股权证、牛熊证为**高杠杆、可归零、可被强制收回**的衍生品，风险极高。任何依据本技能输出做出的交易决策，盈亏与后果由使用者自负。**强烈建议先用模拟盘（paper trading）验证框架，再考虑实盘。** 作者与贡献者不对任何损失负责。

This skill is a research/analysis tool only and is **not investment advice**. HK warrants and CBBCs are high-leverage instruments that can go to zero or be mandatorily called. Use a paper account first. The authors accept no liability for any losses.

## 📄 License

MIT — see [LICENSE](LICENSE).
