---
checkpoint: CP2
checkpoint_id: CP2-CR171-STAGE3-LAUNCH-SCOPE
cr_id: CR-171
title: Stage 3 Launch / Real-Lake Entry Decision Gate — Scope and Authorization Review
status: approved
gate_profile: runtime-high-risk
opened_at: "2026-07-15T21:38:00+08:00"
context_ref: process/context/CP2-CR171.context.json
cp0_result_ref: process/checks/CP0-CR171-STAGE3-LAUNCH-REQUEST-INTAKE.result.json
cp1_result_ref: process/checks/CP1-CR171-USE-CASE-COMPLETENESS.result.json
discussion_ref: process/discussions/CP2-CR171-SCENARIO-DISCUSSION-LOG.md
route_plan_ref: process/checks/CP0-CR171.route-plan.json
result_ref: process/checks/CP2-CR171-STAGE3-LAUNCH-SCOPE.result.json
approved_at: "2026-07-15T21:38:59+08:00"
approved_by: user
---

# CP2 — CR-171 Stage 3 Launch Scope and Authorization Review

## Decision Brief

### 审批者摘要

本门禁只决定未来 Stage 3 入口的路线、verifier 处置和可冻结的 future read scope；不执行数据访问，也不授权真实 computation、runtime、write、aggregate、CR155 promotion 或 trading。

**CP8 的成功只代表可审计的决策与状态收敛。** 即使 CP8 成功，CR-171 也可合法地以 `insufficient_for_current_entry` 或 `incompatible_rework_required` verdict 关闭，并路由 follow-up；它不自动表明 `stage3_entry_ready=true`。

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结可审计的 Stage 3 entry route、verifier disposition 和 future read-contract 边界，同时把历史 Stage 3 evidence 维持为 legacy / require-revalidation。 |
| 推荐动作 | `approve` 下方三项推荐选择：C1-C4 real-producer、event-bounded waiver、scoped research-data-lake read-only。 |
| approve 后会发生什么 | 回填 CP2 结果后仅进入 CP3 架构/授权/历史 revalidation contract 设计；不会执行任何数据读取或 computation。 |
| approve 不授权什么 | 不授权 credentials、provider、lake/NAS write、catalog/current-pointer、real computation、aggregate、runtime、trading、repair/backfill/rerun 或 CR155 promotion。 |
| 不确认会阻塞什么 | CR-171 停在 CP2；不得进入 CP3，也不产生任何 Stage 3 read/computation/runtime 行为。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR171.context.json` |
| capsule 状态 | ready / context check PASS |
| read_profile | compact；CP2 默认只消费 capsule、CP0/CP1 result、CP2 discussion 与本 checkpoint。 |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时扩展全文档。 |
| 全文档读取扩展 | CP1 的 9 个基线对象均已逐项记录；CP0 的 2 个 formal tracking/scope 审计也已记录。 |
| 已知全局 debt | `READ-EXPANSION-LEDGER` 的历史 legacy lines 使全局检查 FAIL；CR-171 新增 lines 合规，该遗留不在本 CR 修复范围。 |
| 缺失 / waived 理由 | 无；waiver=0。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP0 conflict precheck | `process/checks/CR171-STAGE3-LAUNCH-CONFLICT-PRECHECK.json` | scanned | 6 debt/risk groups | 0 | 逐项披露；非本 CR 修复项。 |
| product baseline | UC/REQ/scenario/matrix/scope/backlog | scanned | 5 requirements / 6 QAC / 4 scenarios | 3 | route、verifier、read scope 进入 CP2。 |
| meta-pm discussion | `process/discussions/CP2-CR171-SCENARIO-DISCUSSION-LOG.md` | scanned | 3 | 3 | 无剩余澄清阻塞。 |
| authorization policy | CR-171 hard fences / authorization policy | scanned | 6 deny classes | 0 | 固定 deny-default，不是 CP2 可隐含放行项。 |
| downstream artifacts | HLD / Story / LLD / implementation | N/A | 0 | 0 | CP2 前尚未生成，按 route 正常。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 3 | 下方 route、verifier、read-scope 三项。 |
| 高风险策略确认 | 2 | waiver 的两项机械失效事件；read/computation 的严格分离。 |
| agent 默认处理 | 2 | historical revalidation 只判定不修复；CP4–CP6 当前 N/A，若 CP3 提出可执行工具则重新计算 route。 |
| 仅审计记录 | 6 | CR-032/010/018/031/033/168 debt 与 CR-170 risk alias join。 |

### 待人工决策清单

下方 Checklist 的前三行是本门禁的 **3/3** 待人工决策；选择必须逐项明确，不能只以“继续”代替。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `CP2-CR171-DQ-ROUTE` | architecture | Stage 3 采用哪条 evidence route？ | `c1_c4_real_producer` | `current_runner` | 推荐让 CR-170 canonical hardening 有真实 consumer，并把 computation 放在独立 activation CR；备选更少 CR 但 runner boundary 必须在 CP3 一次性完成。 | 决定入口 CR 数（2 或 1）、producer binding 的 future owner 和 `R-CR170-RUNNER-GAP` 去向。 | 若 CP3 feasibility verdict 不成立，回到 CP2 或新建 follow-up；不得就地改为 computation。 |
| `CP2-CR171-DQ-VERIFIER` | risk_acceptance | FU-006 是否先完成？ | `event_bounded_waiver` | `fu006_first` | 推荐不阻断入口决策，但有 2 个机械到期事件；备选最强独立性但入口总数增至 3。 | 未在期限前完成 FU-006 时，admission PASS/PASS_WITH_RISK 与 exit gate 均不可发生。 | 在任一到期事件前可转为 FU-006 first；CR-170 历史 waiver 不可继承。 |
| `CP2-CR171-DQ-READ-SCOPE` | runtime_authorization | 是否为后续受控研究冻结 read contract？ | `scoped_research_data_lake_read_only` | `no_read_scope_approved` | 推荐只定义五元组和六类 deny，不读取数据；备选最保守，Stage 3 保持未启动。 | 影响未来 CP3 可冻结的 allow fields；绝不影响 computation authorization。 | 任一 release/schema/PIT/lineage gap 需 follow-up CR，不能在本 CR 就地扩范围。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 3 |
| 必须用户决策 | `CP2-CR171-DQ-ROUTE`、`CP2-CR171-DQ-VERIFIER`、`CP2-CR171-DQ-READ-SCOPE`。 |
| 推荐回复 | `approve` 三项推荐，或逐项给出替代值。 |
| 如果你回复 approve | 只进入 CP3 solution-design；CP3 仍不得读取数据湖或授权 real computation。 |
| 修改 | 回复 `修改: <DQ-ID>=<value>`；Host 回写产品基线并重跑 CP1/CP2。 |
| reject | CR-171 保持/退回 CP2，Stage 3 保持未启动。 |

## Entry Criteria

- [x] CR-171 已正式创建，`runtime-high-risk` route plan 已生成。
- [x] CP0 fresh conflict precheck PASS；CR-032/010/018/031/033/168 debt 已逐项披露。
- [x] CP0 已实际解析 `R-CR170-STAGE3-OVERCLAIM → R-CR170-RUNNER-GAP` alias。
- [x] CP1 产品基线、场景、需求、矩阵、规划与 historical narrative 标注已增量完成且 PASS。
- [x] 真实数据湖/NAS、凭据、provider、write、real computation、runtime、trading 均未发生且尚未授权。

## Checklist

| ID | 需审查项 | 推荐 | 备选 | 选择后的可观察后果 |
|---|---|---|---|---|
| `CP2-CR171-DQ-ROUTE` | Stage 3 evidence route | **`c1_c4_real_producer`** | `current_runner` | 推荐：入口共 2 个 CR；后续 Real-Evidence Activation CR 才可处理 real-data mapping、PIT/lineage binding、computation authorization 和 run identity。备选：入口 1 个 CR，但 CP3 必须一次性冻结 current runner 的完整 read-only execution boundary。 |
| `CP2-CR171-DQ-VERIFIER` | FU-006 处置 | **`event_bounded_waiver`** | `fu006_first` | 推荐：入口共 2 个 CR；waiver 在任一 real-evidence admission 可为 PASS/PASS_WITH_RISK 前，或 Stage 3 exit gate 可启动前失效。备选：入口共 3 个 CR，先完成 FU-006 independent verifier lane。CR-170 waiver 不继承。 |
| `CP2-CR171-DQ-READ-SCOPE` | future scoped research-lake read contract | **`scoped_research_data_lake_read_only`** | `no_read_scope_approved` | 推荐：只批准一个待 CP3 冻结的五元组 `data_release / datasets / date_range / read_identity / output_directory`；无条件 deny credentials/environment、provider fetch、lake/NAS write、catalog/current-pointer mutation、runtime、trading。备选：任何 read 保持未授权，Stage 3 未启动。 |

### 不可被本次批准推导的事项

- C1-C4 real computation、producer binding、aggregate orchestration、final StrategyAdmissionPackage 或 CR155 promotion。
- provider fetch、lake/NAS write、catalog/current-pointer mutation、credentials/env/account read。
- QMT、broker、simulation、paper、live、trading、publish 或 deployment。
- 历史运行的 repair、backfill、rerun、manifest rewrite 或 defect remediation。
- CR-010/018/032 的关闭、修复或台账清理。

## Exit Criteria

- [ ] 三项 DQ 每项均有明确选择；未选择不得进入 CP3。
- [ ] 若选择 waiver，两个机械失效事件和不继承规则被明确接受。
- [ ] 若选择 read scope，其五个 allow 字段和六类 deny-default 都被接受；本门禁文本本身仍不执行 read。
- [ ] 认可以下 claim ceiling：`stage3_started=false`、`stage3_entry_ready=false`、`real_computation_authorized=false`，直至后续独立证据和授权成立。

## Deliverables

- 已生成：[CR-171](../../changes/CR-171.md)、[CP0 result](../../checks/CP0-CR171-STAGE3-LAUNCH-REQUEST-INTAKE.result.json)、[CP1 result](../../checks/CP1-CR171-USE-CASE-COMPLETENESS.result.json)、[CP2 context](../../context/CP2-CR171.context.json)、[CP2 discussion log](../../discussions/CP2-CR171-SCENARIO-DISCUSSION-LOG.md)。
- 待本门禁批准后生成：CP2 decision/result、CP3 architecture/authz/revalidation contract。

## 人工审查结果

状态：**approved**

用户于 `2026-07-15T21:38:59+08:00` 批准继续推进，按本 Decision Brief 的推荐值回填以下三项决定：

| 决策 ID | 已批准值 | 审批效果 |
|---|---|---|
| `CP2-CR171-DQ-ROUTE` | `c1_c4_real_producer` | 入口采取两 CR 路径；real-data-to-producer binding 与 real computation authorization 保留给独立 activation CR。 |
| `CP2-CR171-DQ-VERIFIER` | `event_bounded_waiver` | 仅采用事件界定的临时 waiver；在首个 real-evidence admission PASS/PASS_WITH_RISK 决策前或 Stage 3 exit gate 启动前，FU-006 必须完成。 |
| `CP2-CR171-DQ-READ-SCOPE` | `scoped_research_data_lake_read_only` | 只解锁 CP3 冻结未来五元组 read contract；本身不执行读取，更不授权 computation。 |

本批准只解锁 CP3 solution-design。CP3 才会准备实际 `data_release / datasets / date_range / read_identity / output_directory` 五元组的人工冻结材料；在 CP3 前不会读取数据湖，也没有 real computation、runtime、write、aggregate、CR155 promotion 或 trading 授权。
