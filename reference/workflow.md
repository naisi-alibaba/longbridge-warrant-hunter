# 工作流 · 端到端严格执行序列

> 本篇管**"怎么做"——严格的步骤序列**；判据定义与理由见 `framework.md`（本篇只引用 `见 framework Lx`，不重复解释）。所有数据来自 Longbridge OpenAPI MCP（工具名以你的 MCP 配置为准，可能带前缀）。
>
> **每步七字段**：【目的】【前置条件】【调用】【判据 IF-THEN】【GATE 否决点】【输出】【失败处理】。任何一步 GATE 不过即出局（回上一层或回候选池）。
>
> **全局前置纪律**（进入步骤1前必须成立）：
> - GATE：步骤0/1（盲扫）未输出异常清单前，**禁止**调用 `stock_positions` / 读取昨日结论（防锚定）。
> - 每个判断带证伪预算；矩阵决定输出，不为凑认购/认沽对称而降标准。
> - 结论带市场环境限定（勿外推）；小样本只论机制不论收益；任何数字先答"哪个口径/分母/可否复现"（见 framework L12）。

---

## 步骤 0 · 入口路由

【目的】 决定走全市场盲扫还是单标的直入。
【前置条件】 无。
【调用】 无。
【判据 IF-THEN】
- IF 用户未指定标的 THEN 进入全市场盲扫模式 → 步骤 1。
- IF 用户已指定标的 THEN 跳过盲扫，该标的直接进 → 步骤 3（鱼身定位）。
【GATE 否决点】 无。
【输出】 模式标记 + （若有）目标标的。
【失败处理】 无。

---

## 步骤 1 · 盲扫全市场（锁异常）

【目的】 产出当日异常清单（广度、领涨/领跌板块、资金落点），圈定"主线 + 主线之外"候选池。
【前置条件】 步骤0 = 盲扫模式。
【调用】（全部可并行）
- `market_temperature`（market=HK）：取 temperature/valuation/sentiment。
- `quote`（["HSI.HK","HSTECH.HK","HSCEI.HK"]）：取三大指数现价/涨跌。
- `rank_list`（key 来自 `rank_categories` 的 second_tags，如 hot_all-hk）：取 chg/inflow/balance(成交额)/market_cap/industry。★定"活跃大中盘"最好用的单工具。
- `top_movers`（markets=HK，**sort=2 热度**，非 sort=1 波幅）：取异动股 + alert_reason。
- `anomaly`（market=HK）：取异动告警（alert_time 是 Unix 秒，需转）。
- `constituent`（HSI.HK/HSTECH.HK/HSCEI.HK）：取成分强势股 + tags。
- `screener_search`（HK，prevchg>0 / <0 各一次取 total）：粗估涨跌家数=广度。
【判据 IF-THEN】
- IF `screener_search` 返回 463/超时/空 THEN 广度改用 rank_list 涨跌统计估算，输出注明"广度降级"。
- 口径提醒：`rank_list.chg` 是小数分数（−0.0195=−1.95%），`calc_indexes.change_rate` 是百分数值本身（−1.95）——两工具口径相反，勿混。
【GATE 否决点】 无（本步只收集，不否决）。
【输出】 候选标的池 + 当日基调（温度/指数方向/领涨领跌板块）。
【失败处理】 单个发现层工具失败 → 该维标 UNAVAILABLE，不阻断；`constituent` 的 rise_num/fall_num 恒为0，不可用作广度。

---

## 步骤 2 · 收窄到可打票池

【目的】 从候选池剔除不可交易标的，只留有窝轮的流动大中盘。
【前置条件】 步骤1 输出候选池。
【调用】
- 对每个候选并行调 `warrant_list`（warrant_type=["Call"] 和 ["Put"] 各一次，sort_by=Volume，sort_order=Descending）：判有无可交易窝轮。
【判据 IF-THEN】
- IF 该标的 Call 与 Put 均无 → 无轮。
- IF 候选轮全为僵尸轮（last_done=0 或 IV=0 或 effective_leverage=null）→ 视同无可交易轮。
- 用 rank_list 的 balance(成交额)/market_cap 剔仙股/微盘。
【GATE 否决点】 **无可交易窝轮的标的 → 出局**（后续步骤都不做）。GATE：`top_movers` 波幅榜结果不作标的来源（波幅榜以仙股为主，无对应轮）。
【输出】 可打票池：{标的, 有 Call/Put, 成交额, 市值}。
【失败处理】 `warrant_list` 返空 → 该标的出局。

---

## 步骤 3 · 双向鱼身定位（判方向 + 定鱼段）

【目的】 对每个标的输出唯一鱼段标签：认购鱼头/认购鱼中/认沽鱼头/认沽鱼中/鱼尾(出清)。
【前置条件】 步骤2 输出可打票池（无轮标的已剔）。
【调用】（七维立体判位，禁止依赖单一指标）
- `calc_indexes`：取 ChangeRate/FiveDayChangeRate/TenDayChangeRate/HalfYearChangeRate/YtdChangeRate（多周期动量）+ VolumeRatio/Amplitude/TurnoverRate（截面补充）。
- `candlesticks`（day, ~120根, forward_adjust=true）：自算 ① 距20/60日高、距年高低 ② Donchian20 通道 ③ ATR% ④ 量分位/量价背离 ⑤ 抛物线速度三量（涨幅自我分位 + 日收益z + ROC加速度，见 framework L3）。
  注：ATR%/CLV/量分位/抛物线等**逐日时序量只能从 candlesticks 自算**（calc_indexes 只给单点截面，不给序列）；时间戳可能 +1 日偏移，用 prev_close 对齐（见 framework L13）。
- `capital_distribution`：取大单/散户净进出。
- `capital_flow`：取当日累计净流入序列（now/peak/trough/斜率）。
【判据 IF-THEN】
- IF 结构翻转（吸筹↔派发反向）AND 抛物线速度三量共振（见 framework L3）THEN 标记"鱼尾" → 出清（记为反向潜在鱼头）。
- ELSE IF 突破Donchian上轨后中段（1~3×ATR）AND 大单净进 AND 散户净出 THEN "认购鱼中"。
- ELSE IF 破Donchian下轨后中段 AND 大单净出 AND 散户净进 THEN "认沽鱼中"。
- ELSE IF 刚破通道（≤1×ATR，首日/次日启动）THEN 对应方向"鱼头"（轻仓候选）。
- ELSE 标"证据不足" → 不进候选。
【GATE 否决点】
- 鱼尾判定必须"结构翻转 ∩ 抛物线速度"双证齐备；缺一不得判鱼尾（见 framework L3）。
- **禁止仅按涨跌幅判鱼尾。**
- GATE：**CLV 禁止作进场扳机**（见 framework L4）；CLV 只在离场端作软佐证，不进本步判位。
- ATR% 为极大值时，①-④维可靠性打折（元条件）。
【输出】 候选表：{标的, 方向, 鱼段, 七维快照, 大单结构方向, capital_flow 斜率符号}。
【失败处理】
- `capital_distribution` 无当日数据/时间戳早于开盘 → 结构维记中性，不据此判鱼尾。
- `candlesticks` 时间戳偏移校验失败 → 该标的标"时间轴存疑"，不出可执行腿。

---

## 步骤 4 · 进场证据投票（方向证据够不够）

【目的】 判定候选腿是否有足够正向证据可进（对应 framework L9）。
【前置条件】 步骤3 输出非鱼尾、非"证据不足"的候选腿。
【调用】（三源，可与步骤3 复用同批数据）
- `capital_distribution`（结构维）· `capital_flow`（资金流维）· `trades`（count≥200，逐笔主动买维）。
- trades 处理：**剔除竞价特殊单**（trade_type ∈ {U,Y,M,P}）后，主动买占比 = 主买 ÷（主买+主卖），不含中性。
【判据 IF-THEN】（认购看多；认沽取镜像。每维 +1/0/−1，见 framework L9）
- 结构维：大单净进+散户净出→+1；大单净出+散户净进→−1；否则0。
- 资金流维：净流入>0且斜率≥0→+1；<0且斜率≤0→−1；矛盾→0。
- 逐笔维：主动买≥55%(☆)→+1；≤45%(☆)→−1；中间→0。
- IF ≥2维正向 THEN 证据充分。
- ELSE IF ≥2维反向 THEN 方向证伪。
- ELSE 证据不足（含全中性）。
【GATE 否决点】
- **仅"证据充分"放行进入选券**；"证据不足"和"方向证伪"**都不进**（中性=不进，不是"没反向就进"）。
- GATE：投票只裁准入、不裁方向；方向来自步骤3 的价格通道（见 framework L9 角色边界）。
【输出】 通过腿：{标的, 方向, 鱼段, 投票明细(三维符号)}。
【失败处理】 某维数据缺失 → 该维记0（中性），按"证据不足"从严处理，不臆造。

---

## 步骤 5 · 选券（选哪只窝轮·第二道独立闸）

【目的】 为通过腿选出经济的窝轮，或判定"股票好但无经济轮"出局。
【前置条件】 步骤4 通过 + 已先定目标位（见步骤6，回本判断依赖它——可在此步内先算目标位）。
【调用】
- `warrant_list`（warrant_type 定向 Call/Put，sort_by 按需）：取 strike_price/itm_otm/balance_point/premium/delta/effective_leverage/implied_volatility/outstanding_ratio/expiry_date/conversion_ratio。
- `static_info`：取 lot_size（每手股数）。
  ⚠ warrant_list 的 premium/itm_otm/IV/delta 是**小数分数**（0.20=20%，−0.25=−25%），比较阈值前用小数或×100，别拿 0.20 当 20。
【判据 IF-THEN】（选券三问 + 经济性硬闸，见 framework L8）
- 价内外：IF 价外程度≥25%(☆) AND 非(鱼头 AND 赌大波动) THEN 淘汰；|delta|≥0.5(★) 价内/平价鱼中优先。
- 回本：`R_be=|回本点−现价|/现价`，`R_tg=|目标−现价|/现价`。IF R_tg<R_be THEN 淘汰；R_be≤R_tg<1.5×R_be(☆)→边际降级；≥1.5×R_be→通过。
- 溢价：IF premium>20%(☆) THEN 淘汰或仅方向极强短打保留。
- 经济性硬闸（AND，任一不过即淘汰）：到期≥3月(★) · 街货<50%(★) · IV非同标的候选轮极值(☆:剔最高/最低10%) · 价差可接受 · 换股比已知。
- 工具配鱼身/周期：鱼头→价外；鱼中→价内/平价；短打→价内；波段→平价/价外。
【GATE 否决点】 某标的全部候选轮被淘汰 → **该标的出局（股票再好也不做，见 framework L7 两闸独立）**。
【输出】 选定轮：{窝轮代码, 价内外, delta, 实际杠杆, 回本点, 溢价, 到期, 街货}。
【失败处理】 warrant_quote 不含 delta/杠杆/溢价/回本——这些一律回 `warrant_list` 取。

---

## 步骤 6 · 量化盈亏比 + 止损折算

【目的】 算出标的止损线与盈亏比，判是否达门槛。
【前置条件】 步骤5 选定轮 + 步骤1 的 `quote` 现价。
【调用】 复用 `quote` 现价 + warrant_list 的 delta/effective_leverage。
【判据 IF-THEN】（见 framework L8）
- 定目标位 target：取更保守者——① 前一结构位（前高/前低/通道对侧）② 现价 ±2×ATR(☆)。目标位必须先于选券回本判断确定。
- 止损折算（反向定序）：① 定可承受窝轮回撤 D_w=−10~12%(☆) → ② 标的容忍幅度 D_s = D_w ÷ 实际杠杆（实际杠杆用 delta 反推：delta×现价÷轮价÷换股比，**不直接信接口 effLev**）→ ③ 止损线 认购 `现价×(1−D_s)` / 认沽 `现价×(1+D_s)`。
- 盈亏比 RR =|目标轮回报| ÷|止损轮回报|（用实际杠杆映射，线性近似）。
【GATE 否决点】 **IF RR < 1.5(☆) THEN 出局**（盈亏比不足）。禁止直接抄正股颈线/整数关做止损（会被杠杆放大数倍）。
【输出】 可执行腿：{窝轮, 方向, 现价, 目标, 标的止损线, RR, 仓位档}。
【失败处理】 价外证 effLev/delta 折算失真 → 明确标注该止损不可靠，降低仓位或放弃。

---

## 步骤 7 · 对抗式独立盲审（出手前）

【目的】 抛开主判断理由，从主力视角复核候选腿是否为诱导形态（对应 framework L11）。
【前置条件】 步骤6 输出可执行腿。
【调用】 `trades`（读 direction 序列）· `broker_holding_detail`（CCASS 全席位，过滤 parti_number 以 A 开头的清算所席位）。
【判据 IF-THEN】（盲审：只看参数+原始数据，不看看多/看空理由）
- 逐笔：大单连续同向=趋势原料；买卖交替=疑似对倒。
- CCASS：价在拉升 AND 集中席位在减持（chg_1/5/20 为负）→ 拉高出货原料。⚠ CCASS 是 T+1，不与今日 trades 当"实时席位"混。
- 看多腿红旗：突破画像过标准/控盘缩量新高/拉高出货；看空腿红旗：恐慌尾端诱空/挖坑洗盘/超跌反弹/空头挤压。
- IF 多条红旗齐备 AND 证据够硬 THEN 判"疑似操纵"。
- ELSE 放行（宁可漏杀，不过度解读）。
【GATE 否决点】 疑似操纵且证据硬 → 撤下该腿或降级观察。
【输出】 审查结论：{疑似/放行, 置信度, concern}。
【失败处理】 CCASS/trades 取数失败 → 审查降级为"未复核"，标注后由人工决定是否出手。

---

## 步骤 8 · 出行动盘 + 执行前实时确认

【目的】 定稿行动盘并在下单前做实时盘口确认。
【前置条件】 步骤7 放行的腿。
【调用】（可并行）
- `warrant_quote`（live 价/IV/到期/街货）· `depth`（买卖价差）· `capital_flow`（live 净流向）· `capital_distribution`（结构）。
- `depth` 档数取决于权限：asks 长度≥5=LV2 用十档判价差；仅1档=LV1 降级为"仅最优档，penny 轮价差不可精判"。
【判据 IF-THEN】
- IF 价差可接受 AND 资金结构实时印证方向 THEN 已确认 → 按仓位档执行。
- IF 未确认 THEN 挂条件单，不市价追。
- IF 当日为突破日 THEN 权证多已计入日内涨幅 → 禁市价单，一律限价（不追最优卖价）。
- IF 方向=认沽 THEN 止损带更紧、仓位档下调一级（见 framework L6）。
【GATE 否决点】
- GATE：实时确认工具（warrant_quote/depth）失败 → **不下单**，不凭旧快照执行。
- GATE：矩阵决定输出——某方向候选经过滤后为空 → 该方向"空仓"，禁止为凑对称降标准。
【输出】 行动盘：{标的 × 鱼段 × 窝轮 × 盈亏比 × 周期 × 触发 × 止损 × 仓位}（鱼头小/鱼中核心）。
【失败处理】 见 GATE：实时工具失败即暂缓。

---

## 步骤 9 · 持仓监控与离场（可选，若持有）

【目的】 持仓期间抓结构翻转，先于价格离场。
【前置条件】 已持有腿。
【调用】 每 30 分钟轮询 `capital_distribution` + `capital_flow`（结构无 push，只能主动轮询，见 framework L11）；CLV 用 `candlesticks` 收盘算。
【判据 IF-THEN】（离场信号优先级，见 framework L5）
- IF 结构反向翻转确认（大单方向翻转 ∩ flow 斜率恶化）THEN 一级信号 → 当日减/平（**禁止**因未达价格止损降级为观察持有）。
- ELSE IF 仅单证 THEN 预警，连续多轮不升级则维持。
- IF 标的触及步骤6 止损线 THEN 二级兜底平仓。
- CLV 收盘跌破自身低分位 → 结构走弱软佐证（不单独触发，配合一级信号）。
【GATE 否决点】 一级信号与价格止损**取先到者执行**。
【输出】 离场指令 / 维持。
【失败处理】 轮询数据缺失 → 该轮标"结构未知"，回退到价格止损兜底。

---

## 可选 · 下单与账户（若 MCP 开通交易，建议模拟盘）
- GATE：`submit_order` **前置**必先 `account_balance` + `stock_positions` 确认是预期账户（尤其模拟盘）。
- penny 轮一律 LO 限价单，不用 MO。
- "收盘价/资金流"条件止损挂不成普通委托 → 用定时轮询 + `alert_add`（condition=price_rise/fall/percent_*，frequency=once/daily/every）到价提醒兜底。

---

## 附录 · 全维度 → MCP 服务接口速查

| 数据维度 | MCP 工具 | 关键字段 / 用法 | 口径坑 |
|---|---|---|---|
| 多周期动量 | `calc_indexes` | ChangeRate/FiveDay.../HalfYear.../Ytd... | **change_rate 是百分数值本身**（−1.95=−1.95%），非分数 |
| 量比/振幅/换手 | `calc_indexes` | VolumeRatio/Amplitude/TurnoverRate | — |
| 距高低·ATR·量分位·CLV·抛物线速度·Donchian | `candlesticks`（day,~120根） | 全部从 K 线自算 | 只有 K 线给时序；时间戳 +1 日偏移，prev_close 校验 |
| 大/中/散单结构 | `capital_distribution` | 大单/散户净进出 | 仅当天·无历史·不可回测；盘前返空→记中性 |
| 当日累计净流入 | `capital_flow` | 逐分钟累计序列(now/peak/trough/斜率) | 累计序列非单点；斜率唯一先于价格翻 |
| 逐笔主动方向 | `trades` | 每笔 direction(Up主买/Down主卖/Neutral) | 剔 trade_type∈{U,Y,M,P}；分母不含中性 |
| CCASS 全席位 | `broker_holding_detail` | list[].shares.value + chg_1/5/20 + strong | T+1 非实时；过滤 parti_number 以 A 开头 |
| 候选窝轮(选券) | `warrant_list` | strike/itmOtm/balancePoint/premium/delta/effLev/IV/街货/到期/换股比 | **premium/itmOtm/IV/delta 是小数分数**；只取 Call/Put |
| 窝轮实时报价 | `warrant_quote` | 现价/IV/到期/街货/行权价/换股比 | **不含 delta/杠杆/溢价/回本/价内外**——回 warrant_list 取 |
| 盘口价差 | `depth` | 最优买卖档 | LV1 只1档，2-10档属 LV2 |
| 公司行动/财报 | `corp_action` | 未来 ReportDate/DividendExDate | 按 date≥今日过滤；临近财报不开新仓 |
| 每手股数 | `static_info` | lot_size | 盈亏比按手换算 |
| 盲扫发现层 | `market_temperature`/`anomaly`/`top_movers`/`rank_list`/`constituent`/`screener_search` | 温度/异动/榜单/成分/广度 | 部分子系统间歇463→降级；constituent 的 rise/fall_num 恒0 |

> **权限一句话**：LV1（如 HK_L1_OpenAPI）下上表工具除 `depth` 2-10 档与 `brokers` 实时队列外全部可用；抗操纵两维（trades + broker_holding_detail）不受档次限制。运行时用一次 `depth` 探测档次（asks 长度≥5=LV2）。任一调用失败/返空 → 该证据维记中性并显式标注缺失，绝不用推测值填补；涉及进场扳机的实时工具失败即暂缓下单。
