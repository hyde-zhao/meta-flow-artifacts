# CP2 人工门禁发起消息 — CR-139 Strategy Data Foundation

**Checklist 路径**：`process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md`
**人工审查稿**：`process/checkpoints/CP2-CR139-REQUIREMENTS-BASELINE-REVIEW.md`
**Context Capsule**：`process/context/CP2-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml`

## 自动预检结论
**PASS_WITH_RISK**（0 BLOCKING）。49 REQ（REQ-201..249）：45 项整改 1:1 映射（机械核验 a=6/b=7/c=12/d1=6/d2=14=45）+ 4 跨切约束；8 新增 UC（UC-51..57）、24 SCN、MVP=Wave1 P0；既有合同 15 类已 grep 核验。RISK 1/2 已解除（host-orchestrator live SGQ 已确认 D8a/D8b）；RISK 3（Wave2/3 部分项无独立 SCN）非 BLOCKING，CP5 落地。

## Context Capsule 摘要
来源 handoff v0.7 五轮评审定稿 + CR-139 parent CR 审计边界；read_profile=compact；默认读取策略为 must_read（handoff + CR-139 + USE-CASES/REQUIREMENTS 基线），未读完整 HLD/LLD；本轮无全文档读取扩展。核心：数据湖支撑多因子+ML 策略四阶段全流程可信/可复现/可审计；四组门禁为 CP3 必过；45 项整改按 Wave 分批；既有合同闭环非新建。

## 审批者摘要
- **本次确认服务的整体目标**：冻结 CR-139 parent CR 的 CP2 范围基线，把数据湖整改 45 项锁定为审计边界，确认 D8a/D8b 纳入口径，为 CP3 HLD 铺路。
- **推荐动作**：`approve`——接受 CP2 范围基线全部产物与 D8a/D8b 纳入口径。
- **approve 后会发生什么**：CP2 通过，STATE 推进到 solution-design（CP3），meta-se 刷新 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP/FEATURE-DESIGN-MATRIX 并新开 companion HLD「Strategy Data Foundation」，四组门禁集落为 CP3 门禁；CP3 通过后才拆 Story + LLD。
- **approve 不授权什么**：不授权 runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write；CP2/CP3 通过不等于授权实现或运行时。
- **不确认会阻塞什么**：CP2 未通过则不得进入 CP3 HLD、不得拆 Story、不得实现（规则 1/2/25）。

## 决策分层
- 必须用户决策：D8a、D8b（已 live 确认采纳推荐，CP2 门禁为最终 ratification）
- 高风险策略确认：SGA-139-01 范围策略（已 live 确认）、SGA-139-04 物理迁移后置（Wave1/D3 既定）
- agent 默认处理：SGA-139-02 d2 路由、SGA-139-03 deferred、D1-D7 追溯确认、产物路由、文档处理
- 仅审计记录：DQ-CR139-01 ML feature 层归属（D5 附条件）、DQ-CR139-02 DuckDB 引入时机、Deferred Ideas DEF-139-01..04

## 决策收集覆盖
已扫描来源：CR-139 §7、handoff v0.7 §6、CP2 capsule、meta-pm 交还摘要、CP2 灰区讨论日志、CLARIFICATION-LOG。候选问题数 14，纳入待决策数 2（D8a/D8b），其余为追溯确认/resolved/deferred/审计层。分类 / N/A 原因：D1-D7 已同意追溯、灰区 resolved/deferred、DQ 入审计层。

## 本轮待人工决策项：2

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| D8a | scope | (d1) 纯新建 6 项（L3/E4/T7/T8/X1/X2）是否纳入？ | 纳入，按新建能力处理 | 不纳入；部分纳入 | 推荐保证信任基础/可审计/跨源完整，T7/T8 是 Wave1 P0 前置；不纳入则前置缺失 | 范围/复杂度/CP3 门禁 | 修改口径则增量回更；不纳入移入 BACKLOG |
| D8b | scope | (d2) 既有合同闭环 14 项是否纳入？ | 纳入，不重复设计（F1-F4 版本化/其余闭环） | 不纳入；全部只接通不补版本化 | 推荐区分配置事实源版本化与执行链路闭环，路由精准；不纳入则闭环断链 | 范围/设计路由/CP3 Feature 归属 | 修改口径则增量回更；不纳入移入 BACKLOG |

> D8a/D8b 均经 host-orchestrator live SGQ 用户确认采纳推荐方案（2026-06-28T15:50:00+08:00）。完整待决策表见人工审查稿。

## 用户需决策事项
本轮需用户决策事项：D8a（(d1) 纯新建 6 项纳入）、D8b（(d2) 既有合同闭环 14 项纳入不重复设计）。回复 `approve` 表示接受 D8a/D8b 推荐方案及 CP2 范围基线全部产物。

## 不授权项
runtime / NAS / provider / catalog 运行时连接与凭据读取 / QMT/MiniQMT/gateway 运行时 / 下单撤单/模拟盘/实盘 / 真实数据湖写入 / published pointer 前移执行 / 物理分区迁移（Wave1 N1 后置到基线冻结后）/ 实盘读审计写 / broker facts 实盘写 / run evidence 上传 / Git remote write/commit/push / 任何 publish/live/production 写入。后续必要验证按 action scope 单独形成 `runtime_authorization` 决策项。

---

**如果你回复 approve**：接受待人工决策清单内全部推荐方案（D8a 纳入 / D8b 纳入不重复设计）及 CP2 范围基线，CP2 通过，推进到 CP3（solution-design，meta-se 刷新蓝图/矩阵/新开 companion HLD）。
**回复 `修改: <具体修改点>`**：调整单项决策或范围（如 `修改: D8b=部分纳入`），host-orchestrator 按规则 36 重新计算影响面、增量回更产物并重新发起确认。
**回复 `reject`**：CP2 不通过，回退到 requirement-clarification 重新澄清。

`approve` 表示接受推荐方案，**不表示授权**上述"不授权项"中的任何 runtime / 凭据 / 外部接口 / 数据写入 / publish / live / 交易类操作；这些必须独立形成 `runtime_authorization` 决策项。
