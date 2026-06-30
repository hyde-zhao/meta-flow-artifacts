---
discussion_id: "CP2-CR139-SCENARIO-DISCUSSION"
status: "live-sgq-completed-pending-cp2-human-gate"
created_at: "2026-06-28T00:00:00+08:00"
created_by: "meta-pm"
cr_id: "CR-139"
delegated_phase: "requirement-clarification"
interaction_mode: "handoff-trail-plus-live-sgq"
source_context: "process/context/CP2-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml"
live_sgq_completed_at: "2026-06-28T15:50:00+08:00"
live_sgq_completed_by: "host-orchestrator"
---

# CP2 Scenario Discussion Log — CR-139 Strategy Data Foundation

## 讨论目标

将数据湖模块从"已验证的小窗口市场数据链路"推进为**策略生产数据底座**，支撑多因子 + ML 策略从信息收集 → 回测 → 模拟盘 → 实盘全流程，达到可信、可复现、可审计。基于 handoff v0.7（五轮评审定稿）确认 CP2 范围基线，识别会影响交付的灰区，形成 D8a/D8b 纳入口径与 Deferred Ideas。

## 交互方式说明

本次委托未提供 `AskUserQuestion` 平台工具（`interaction_mode=handoff-trail-plus-return-summary`）。SGQ-* 场景确认证据来源为 handoff v0.7 五轮评审 trail（D1-D7 已同意、D8a/D8b 二轮建议批准）。meta-pm 以"建议批准"为 working basis 推进产物，灰区重点选择与 D8a/D8b 正式确认在交还摘要中呈现，由 host-orchestrator 在 CP2 人工门禁统一确认。若用户在 CP2 前纠正，meta-pm 按规则 36"决策修订再发布"增量回更。

## Scenario Gray Areas

| 灰区 ID | 问题 | 为什么重要 | 影响面 | 用户选择 | 状态 |
|---|---|---|---|---|---|
| SGA-139-01 | 范围广度 vs 落地节奏：45 项全纳入 vs 先 P0/P1 收口、P2 defer | 决定 CP2 范围是"全量基线"还是"分批基线"，影响 Wave 切分与 MVP 边界 | 范围 / 复杂度 / 验证 / 交付出口 / 后续门控 | selected | resolved-by-recommendation（建议：45 项全纳入 CP2 基线，按 Wave 分批实现；P2 入 BACKLOG deferred） |
| SGA-139-02 | 既有合同闭环边界（d2 14 项）：哪些算纯闭环、哪些需补版本化事实源 | 决定 d2 项是"只接通执行"还是"补版本化 + release 闭环"，影响设计路由与 LLD 深度 | 范围 / 复杂度 / 验证 / CP3 Feature 归属 | selected | resolved-by-recommendation（建议：F1/F2/F3/F4 补版本化事实源 + release 闭环；L1/L2/L4/L5/E3/E5/T5/T6/X3/X4 接通执行/前置/闭环） |
| SGA-139-03 | ML feature 层归属：lake `features/` 子层 vs 独立 feature store（D5 附条件） | 决定 ML feature 持久化层架构，影响 V3 设计与未来切换成本 | 架构 / 复杂度 / CP3 | deferred | open-deferred（D5 已决：lake `features/` 子层带版本；切换独立 feature store 条件写入 Deferred，后续 CR 触发） |
| SGA-139-04 | 物理分区迁移时机与存量 38 run_id 分区：Wave1 N1 后置后是否迁移、D3 不重命名影响 | 决定存量分区是"保持只读去重"还是"物理迁移收敛"，影响归因与回滚 | 范围 / 复杂度 / 验证 / 回滚 | selected | resolved-by-recommendation（建议：Wave1 N1 物理迁移后置到基线冻结之后；存量按 D3 不重命名，新数据规范化 + 旧数据下次重跑替换；读层去重优先 C2b） |

## SGQ 场景确认交互记录（规则 44：至少 1 条 SGQ-*）

### SGQ-139-01 — 范围广度与落地节奏

- **问题**：CR-139 的 45 项整改，CP2 范围基线应如何切分？
- **候选选项**：
  1. 45 项全纳入 CP2 基线，按 Wave1(P0)/Wave2(P1)/Wave3(P2) 分批实现，P2 入 BACKLOG deferred（推荐）
  2. 仅 P0(四道防线+基线) 入 CP2，P1/P2 后续 CR
  3. P0+P1 入 CP2，P2 后续 CR
- **推荐项**：选项 1
- **影响与 trade-off**：选项 1 保证范围基线完整、审计边界闭环、Traceability 完整，但 CP3/CP4/CP5 工作量大；选项 2/3 减少近期工作量但破坏 parent CR 审计完整性，且 P1（ML/模拟盘解锁）依赖 P0 信任基础，拆分易产生跨 CR 依赖冲突。
- **用户回答（来源 handoff v0.7 五轮评审 + capsule D8a/D8b 二轮建议批准）**：采纳选项 1 方向——45 项全纳入，按 Wave 分批；D8a(6)/D8b(14) 建议批准。
- **复述确认**：meta-pm 以"45 项全纳入 CP2 基线、按 Wave1/2/3 分批、P2 deferred、D8a/D8b 建议批准"为 working basis 推进产物；正式确认路由 CP2 人工门禁。
- **影响面**：范围 / 复杂度 / 验证 / 交付出口 / 后续门控。

### SGQ-139-02 — 既有合同闭环 vs 版本化事实源

- **问题**：d2 14 项既有合同闭环，哪些只接通执行、哪些需补版本化事实源 + release 闭环？
- **候选选项**：
  1. 配置类（F1/F2/F3/F4）补版本化事实源 + release 闭环；执行/前置类（L1/L2/L4/L5/E3/E5/T5/T6/X3/X4）接通执行/前置/闭环（推荐）
  2. 全部只接通执行，不补版本化
  3. 全部补版本化事实源
- **推荐项**：选项 1
- **影响与 trade-off**：选项 1 区分"配置事实源需版本化以支撑回测/归因复现"与"执行链路只需闭环"，设计路由精准；选项 2 导致配置类无版本化、回测不可复现；选项 3 对执行类过度设计。
- **用户回答（来源 handoff §3.10 + §3.6/3.7/3.8/3.11 修法列）**：F1-F4 修法明确"补版本化事实源 + release 闭环"；L1/L2/L4/L5/E3/E5/T5/T6/X3/X4 修法明确"接通执行/前置/闭环"。
- **复述确认**：meta-pm 按 F1-F4 版本化、其余闭环 路由写入 REQUIREMENTS 增量。
- **影响面**：范围 / 设计路由 / CP3 Feature 归属。

## Deferred Ideas

| ID | 想法 / 风险 / 扩展场景 | 来源 | 延后原因 | 重启条件 |
|---|---|---|---|---|
| DEF-139-01 | ML feature 层切换到独立 feature store | SGA-139-03 / D5 附条件 | 当前采用 lake `features/` 子层带版本；独立 store 切换成本高，无即时需求 | feature 规模 / 跨湖查询 / 在线推理延迟触发独立 store 需求时，后续 CR 评估 |
| DEF-139-02 | 物理分区迁移 / 候选压缩（N1 物理迁移部分） | SGA-139-04 / Wave1 N1 后置 | Wave1 明确后置到基线冻结之后，避免"历史数据变化"与"结构修复"混在一起无法归因 | T7 黄金值对比归因完成后，基于基线结果决定是否物理迁移 |
| DEF-139-03 | Wave3 P2 项（L5 replay / R4 延迟优化 / T1 / M3/M4 / F4） | SGA-139-01 | P2 优先级，非 MVP/P1 解锁前置 | Wave1/Wave2 完成后按需启动 |
| DEF-139-04 | 存量 run_id 一次性重命名 | D3 | D3 已决不重命名，新数据规范化 + 旧数据下次重跑替换 | 仅当重跑替换不可行且命名冲突阻塞时重新评估 |

## 讨论恢复点

`process/checks/CP2-CR139-DISCUSSION-CHECKPOINT.json`

## Live SGQ 交互记录（host-orchestrator 补充，2026-06-28T15:50:00+08:00）

> meta-pm 委托期间未提供 AskUserQuestion 工具，SGQ 证据原为 handoff v0.7 评审 trail。host-orchestrator 回收后按规则 44 补一轮 live SGQ 用户可见交互，向用户确认范围策略与 D8a/D8b 纳入口径。用户全部采纳推荐方案，无需按规则 36 决策修订再发布。

| 问题 | 推荐方案 | 用户回答 | 结果 |
|---|---|---|---|
| SGA-139-01 范围策略：45 项整改 CP2 基线如何切分？ | 45 项全纳入按 Wave1(P0)/Wave2(P1)/Wave3(P2) 分批，P2 入 BACKLOG deferred | 45 项全纳入按 Wave 分批（推荐） | 采纳，无需回更 |
| D8a：(d1) 纯新建 6 项（L3/E4/T7/T8/X1/X2）是否纳入？ | 纳入，按新建能力处理 | 纳入（推荐） | 采纳，REQ-201..245 已按此口径编写 |
| D8b：(d2) 既有合同闭环 14 项是否纳入？ | 纳入，按既有合同接通闭环/版本化扩展处理，不重复设计 | 纳入不重复设计（推荐） | 采纳，REQ-201..245 已按此口径编写 |

**结论**：3 项决策全部采纳推荐方案，0 项需决策修订再发布。SGQ live 交互闭环，RISK 2（无 live SGQ）解除。D8a/D8b 由"建议批准 working basis"升级为"用户 live 确认"，正式 CP2 人工门禁为最终 ratification。
