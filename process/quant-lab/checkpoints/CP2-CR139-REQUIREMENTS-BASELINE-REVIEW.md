---
checkpoint: CP2
cr_id: CR-139
title: "CP2 需求/场景/范围基线门 — CR-139 Strategy Data Foundation 人工审查"
type: human-gate
gate_profile: standard
status: approved
created_at: 2026-06-28T00:00:00+08:00
created_by: host-orchestrator
approved_by: user
approved_at: 2026-06-28T16:05:00+08:00
approval_result: cp2-approved
auto_precheck_conclusion: PASS_WITH_RISK
auto_precheck_path: process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md
checklist_path: process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md
context_capsule: process/context/CP2-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml
discussion_log: process/discussions/CP2-CR139-SCENARIO-DISCUSSION-LOG.md
discussion_checkpoint: process/checks/CP2-CR139-DISCUSSION-CHECKPOINT.json
human_review_status: pending
pending_human_decisions_count: 2
blocking_decisions: 0
---

# CP2 人工门禁 — CR-139 Strategy Data Foundation

## Checklist 路径
`process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md`

## 自动预检结论
**PASS_WITH_RISK**（0 BLOCKING）。49 REQ（REQ-201..249）：45 项整改 1:1 映射 + 4 跨切约束；机械核验类别 a=6/b=7/c=12/d1=6/d2=14=45，与 handoff §3.12 一致。8 新增 UC（UC-51..57）、24 SCN、MVP=Wave1 P0。既有合同 15 类已 grep 核验存在。RISK 1/2 已解除（host-orchestrator live SGQ 用户确认 D8a/D8b）；RISK 3（Wave2/3 部分项无独立 SCN）非 BLOCKING，CP5 落地。

## Decision Brief

### 审批者摘要
- **本次确认服务的整体目标**：冻结 CR-139 parent CR 的 CP2 范围基线，把数据湖整改 45 项锁定为审计边界，确认 D8a/D8b 纳入口径，为 CP3 HLD 人工门禁铺路。
- **推荐动作**：`approve`——接受 CP2 范围基线（USE-CASES v1.17 / REQUIREMENTS v1.17 / SCENARIOS-CR139 / TEST-MATRIX-CR139 / STORY-MAP-CR139 / MVP-SCOPE-CR139 / RELEASE-SLICES-CR139 / BACKLOG-CR139）与 D8a/D8b 纳入口径。
- **approve 后会发生什么**：CP2 通过，STATE 推进到 solution-design（CP3），由 meta-se 刷新 BLUEPRINT/DOMAIN-MAP/DEPENDENCY-MAP/FEATURE-DESIGN-MATRIX 并新开 companion HLD「Strategy Data Foundation」，四组门禁集落为 CP3 门禁；CP3 通过后才拆 Story + LLD（CP4/CP5）。
- **approve 不授权什么**：不授权 runtime/NAS/QMT/trading/provider-lake-catalog 写入/物理分区迁移/Git remote write；CP2/CP3 通过不等于授权实现或运行时（详见"不授权项"）。
- **不确认会阻塞什么**：CP2 未通过则不得进入 CP3 HLD、不得拆 Story、不得实现（规则 1/2/25）。

### Context Capsule Summary
- capsule 路径：`process/context/CP2-CR139-STRATEGY-DATA-FOUNDATION-CONTEXT.yaml`
- read_profile：compact
- 默认读取策略：must_read = handoff v0.7 + CR-139 + USE-CASES/REQUIREMENTS v1.16 基线；read_if_needed = HLD-DATA-LAKE/docs/scenarios；do_not_read_by_default = 完整 HLD-DATA-LAKE / 全部 LLD / 全部 Story 卡片。
- 全文档读取：本轮无全文档读取扩展；meta-pm 与 host-orchestrator 均消费 capsule + must_read，未读完整 HLD/LLD。

### 决策分层
| 层级 | 事项 |
|---|---|
| 必须用户决策 | D8a（d1 纯新建 6 项纳入）、D8b（d2 既有合同 14 项纳入不重复设计）——已 live 确认采纳推荐，CP2 门禁为最终 ratification |
| 高风险策略确认 | SGA-139-01 范围策略（45 项全纳入按 Wave 分批）——已 live 确认；SGA-139-04 物理迁移后置——按 Wave1/D3 既定 |
| agent 默认处理 | SGA-139-02（d2 闭环 vs 版本化路由）、SGA-139-03 deferred、D1-D7 追溯确认、产物路由、文档处理 |
| 仅审计记录 | DQ-CR139-01（ML feature 层归属，D5 已决附条件）、DQ-CR139-02（DuckDB 引入时机，CP3/CP5）、Deferred Ideas DEF-139-01..04 |

### Decision Collection Coverage
| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---|---|---|
| CR-139 §7 决策表（D1-D8） | 已扫描 | 8 | 2（D8a/D8b） | D1-D7 已同意（追溯确认，非待决策） |
| handoff v0.7 §6 | 已扫描 | 8 | 0 | 与 CR-139 §7 同源，已计入上 |
| CP2 context capsule | 已扫描 | 2（D8a/D8b）+ 4 灰区 + 2 DQ | 2（D8a/D8b） | 灰区 resolved/deferred，DQ 入审计层 |
| meta-pm 交还摘要 | 已扫描 | 2（D8a/D8b）+ 2 DQ | 2（D8a/D8b） | DQ 入审计层 |
| CP2 灰区讨论日志 | 已扫描 | 4 灰区 | 0 | SGA-01/02/04 resolved，SGA-03 deferred |
| CLARIFICATION-LOG | 已扫描 | D8a/D8b + D1-D7 | 2（D8a/D8b） | D1-D7 追溯 |

合计候选 14，纳入待决策 2（D8a/D8b），其余为追溯确认 / resolved / deferred / 审计层。

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| D8a | scope | (d1) 纯新建 6 项（L3 读审计 / E4 离线在线一致性 / T7 回归基线 / T8 自动清册 / X1 复权 PIT / X2 跨源一致性）是否纳入 CR-139 范围？ | 纳入，按新建能力处理（需完整设计+LLD） | 不纳入（后续 CR 处理）；部分纳入（仅 T7/T8 等 Wave1 前置项） | 推荐项保证信任基础/可审计/跨源一致性完整，T7/T8 是 Wave1 P0 前置、L3/E4/X1/X2 是门禁组必要项；不纳入则 Wave1 P0 前置缺失、削弱可审计性；部分纳入留下门禁组缺口 | 范围 / 复杂度 / CP3 门禁；CP3/CP4/CP5 工作量增加 | CP2 修改口径则 meta-pm 按规则 36 增量回更 REQUIREMENTS/SCENARIOS；切换为不纳入时把对应项移入 BACKLOG-CR139 deferred |
| D8b | scope | (d2) 既有合同闭环 14 项（L1/L2/L4/L5/E3/E5/T5/T6/F1/F2/F3/F4/X3/X4）是否纳入？15 类既有合同已 grep 核验存在，整改为闭环/版本化扩展非新建。 | 纳入，按既有合同接通闭环/版本化扩展处理，不重复设计（F1-F4 补版本化事实源+release 闭环；L1/L2/L4/L5/E3/E5/T5/T6/X3/X4 接通执行/前置/闭环） | 不纳入（后续 CR 处理）；全部只接通执行不补版本化 | 推荐项区分配置事实源（需版本化以支撑回测/归因复现）与执行链路（只需闭环），设计路由精准；不纳入则既有合同闭环断链、run-id 贯通/成本前置/读前门禁缺失；全部只接通则配置类无版本化、回测不可复现 | 范围 / 设计路由 / CP3 Feature 归属；既有合同闭环断链风险 | CP2 修改口径则增量回更；切换为不纳入时把对应项移入 BACKLOG-CR139；切换为全部只接通时回退 F1-F4 版本化设计 |

### 用户需决策事项
本轮需用户决策事项：D8a（(d1) 纯新建 6 项纳入）、D8b（(d2) 既有合同闭环 14 项纳入不重复设计）。两项均经 host-orchestrator live SGQ 用户确认采纳推荐方案（2026-06-28T15:50:00+08:00），CP2 人工门禁为最终 ratification。回复 `approve` 表示接受 D8a/D8b 推荐方案及 CP2 范围基线全部产物。

## 灰区结论
- SGA-139-01（范围策略）：45 项全纳入按 Wave 分批，P2 入 BACKLOG deferred。✅ live 确认。
- SGA-139-02（d2 闭环 vs 版本化）：F1-F4 补版本化事实源+release 闭环；其余接通执行/前置/闭环。
- SGA-139-03（ML feature 层归属）：deferred（D5 已决 lake `features/` 子层，切换独立 store 入 DEF-139-01）。
- SGA-139-04（物理迁移时机）：Wave1 N1 后置到基线冻结后；D3 不重命名存量；读层去重优先 C2b。
- Deferred Ideas：DEF-139-01..04（feature store 切换 / 物理迁移执行 / Wave3 P2 / 存量重命名）。

## 不授权项（non-authorized scope）
CP2/CP3 通过**不授权**：runtime / NAS / 外部存储 / provider / catalog 运行时连接与凭据读取 / QMT/MiniQMT/gateway 运行时 / 下单撤单/模拟盘/实盘 / 真实数据湖写入 / published pointer 前移执行 / 物理分区迁移（Wave1 N1 明确后置到基线冻结之后）/ 实盘读审计写 / broker facts 实盘写 / run evidence 上传 / Git remote write/commit/push/分支治理 / 任何 publish/live/production 写入。后续必要验证按 action scope、运行窗口、脱敏、回滚、审计范围单独形成 `runtime_authorization` 决策项。

## Deliverables（CP2 产物）
| 产物 | 路径 |
|---|---|
| USE-CASES.md v1.17 | process/USE-CASES.md |
| REQUIREMENTS.md v1.17 | process/REQUIREMENTS.md |
| SCENARIOS-CR139.yaml | process/SCENARIOS-CR139.yaml |
| TEST-MATRIX-CR139.md | process/TEST-MATRIX-CR139.md |
| STORY-MAP-CR139.md | process/STORY-MAP-CR139.md |
| MVP-SCOPE-CR139.md | process/MVP-SCOPE-CR139.md |
| RELEASE-SLICES-CR139.md | process/RELEASE-SLICES-CR139.md |
| BACKLOG-CR139.md | process/BACKLOG-CR139.md |
| 灰区讨论日志 | process/discussions/CP2-CR139-SCENARIO-DISCUSSION-LOG.md |
| 讨论 checkpoint | process/checks/CP2-CR139-DISCUSSION-CHECKPOINT.json |
| CLARIFICATION-LOG | process/discussions/CLARIFICATION-LOG.md |
| CP2 自动预检 | process/checks/CP2-CR139-REQUIREMENTS-BASELINE-PRECHECK.md |

## 人工审查结果
> 用户于 2026-06-28T16:05:00+08:00 回复批准 CP2。

- 审查人：user
- 审查时间：2026-06-28T16:05:00+08:00
- 审查结论：approved（CP2 通过）
- 修改意见：批准 D8a 6 项和 D8b 14 项全部纳入 CR-139；d1 按新建能力进入完整设计 + LLD；d2 按既有合同闭环/版本化扩展处理，不重复设计；**追加约束：Wave1 结构性变更必须在 T7/T8 基线与 C2a 画像之后执行**（已写入 CR-139 §5 Wave1 与 REQ-247 约束，handoff §5 重排口径强化）
- 采纳决策：D8a=A1（纳入，新建能力处理）、D8b=B1（纳入，不重复设计，F1-F4 版本化/其余闭环）、SGA-139-02=C1、SGA-139-04=D1
