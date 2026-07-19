---
checkpoint: CP2
checkpoint_id: CP2-CR172-REQUIREMENTS-BASELINE
cr_id: CR-172
title: Stage 3 Real-Evidence Activation Phase A — Requirements, Scope and Authorization Review
status: approved
gate_profile: runtime-high-risk
opened_at: "2026-07-16T10:34:00+08:00"
context_ref: process/context/CP2-CR172.context.json
cp0_result_ref: process/checks/CP0-CR172-REQUEST-INTAKE.result.json
cp1_result_ref: process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json
discussion_ref: process/discussions/CP2-CR172-SCENARIO-DISCUSSION-LOG.md
route_plan_ref: process/checks/CP0-CR172.route-plan.json
result_ref: process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json
approval_result: approved
selected_path: PATH-B
approval_source: "用户明确指示：完成四项非阻塞整改后视为批准 CP2，并继续推进到下一个人工门禁"
prior_review_result: changes_requested
reviewed_by: user
reviewed_at: "2026-07-16T11:35:08+08:00"
reopened_at: "2026-07-16T11:04:35+08:00"
---

# CP2 — CR-172 Stage 3 Real-Evidence Activation Phase A

## 门禁修订记录

| 版本 | 时间 | 修订人 | 变更要点 | 门禁效果 |
|---|---|---|---|---|
| r1 | 2026-07-16 10:34 +08:00 | Host Orchestrator | 首次发起 8 项 PATH/授权/治理决策。 | pending |
| r2 | 2026-07-16 10:53 +08:00 | 用户 / Host Orchestrator / meta-pm | 用户要求 UC-58 补齐业务动机、用户痛点、使用价值与旅程第 0 步；完成 4/4 整改、SGQ 2/2 与 CP1/CP2 重检后重新发起。8 个 DQ、五字段事实、PATH 条件和授权边界均未改变。 | pending（prior=`changes_requested`） |
| r3 | 2026-07-16 11:35 +08:00 | 用户 / Host Orchestrator / meta-pm | 评估并完成四项终审整改：策略占位/未来 CP3-CP6 锚定、CR172 风险登记、当前 source refs、CR170 scoped Git 授权隔离。用户明确整改后视为批准 CP2；按 0/5+owner OPEN 条件选择 PATH-B。 | approved（PATH-B） |

## Decision Brief

### 审批者摘要

CR-172 的范围整改已经完成，CP0、CP1 与 CP2 自动预检均为 `PASS`。范围评审提出的三项改进已正式冻结：PATH-B 完成后必须恢复 activation；PATH-C 后 C2/C3 默认各自建立独立 runtime-high-risk CR；不同 owner 的 estimator merge 必须采用同 revision/hash 双 ledger 批准且权限取交集。

上一轮 CP2 评审指出 UC-58 偏工程/治理视角。该问题现已整改：业务动机、用户痛点、使用价值、业务触发 `4/4` 明示，用户可见确认 `3/3`。真实用户意图是让研究负责人和策略研究员获得可用于待评估策略真实 multiple-testing / overfit 评估的 C1 typed evidence，并为后续 mature StrategyAdmissionPackage 提供证据基础；PATH、blast radius、五字段和联合审批是实现这一价值的安全约束。`策略 X` 当前只是业务示例，PATH-B 与具体策略无关；未来 activation CP3 必须冻结 `strategy_id + strategy_name`，CP6 evidence 必须一致锚定或 fail-closed。fixture/static evidence 仍只证明契约/流程，未来 C1 evidence 本身也不构成 admission `PASS/PASS_WITH_RISK`。

当前仍缺少不可推断的授权事实：五字段具体值为 `0/5`，授权 data owner identity 为 `OPEN`。因此本门禁使用条件式推荐：

- 当前直接回复 `approve`：按最安全推荐选择 `PATH-B`，只启动独立离线 estimator 前置；CR-172 activation 保持 deferred，不进入 CP3。
- 若提供五字段 `5/5` 具体值和 data owner identity：默认推荐 `PATH-C`（C1-first），再进入 CP3 设计；不能只写“可冻结”。
- `PATH-A` 只有在 5/5+owner 成立并显式接受 C1-C3 首次联合 blast radius 时才可选。

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 冻结 CR-172 的 PATH-B/C/A 路由、五字段授权前提、effective-trial 审批边界、C1 降级、C2/C3 治理、E1/OI-005 与 joint approval 契约。 |
| 用户业务锚点 | 策略 X 需要真实 multiple-testing / overfit 证据；未来 C1 real typed evidence 供研究员评估并支撑后续 mature SAP，但不直接产生 admission PASS。 |
| 当前推荐动作 | 由于字段值 `0/5` 且 owner `OPEN`，当前 `approve` 推荐 PATH-B；若补齐 5/5+owner，则推荐 PATH-C。 |
| approve 后会发生什么 | 当前证据下：CR-172 activation deferred，另行建立/推进 FU-CR164-004 离线方法学 CR；五字段可冻结后恢复 CR-172 并重开 CP2。 |
| approve 不授权什么 | 不授权真实 lake/NAS 读取、credential/env 读取、provider fetch、任何 write、C1-C3 computation/binding、runtime、aggregate、CR155、trading、publish/deploy 或 Git remote write。 |
| 不确认会阻塞什么 | CR-172 停在 CP2；不得进入 CP3、Story、LLD、实现或任何真实 evidence activation。 |

### 五字段与授权 owner 事实表

下表展示的是当前可审计事实，不是待模型补全的占位符。`OPEN` 不能由历史值、旧 CP3、目录、manifest、通配符或隐式继承替代。

| 字段 | 当前具体值 | 当前状态 | 冻结要求 |
|---|---|---|---|
| `data_release` | `OPEN / 未提供` | `0/1` | 精确 release ID；禁止 `latest/current/*`。 |
| `datasets` | `OPEN / 未提供` | `0/1` | 精确有限数据集清单；禁止通配符或隐式继承。 |
| `date_range` | `OPEN / 未提供` | `0/1` | 精确起止边界及闭开区间语义。 |
| `read_identity` | `OPEN / 未提供` | `0/1` | 已授权、可审计的读取身份；不得读取凭据原文。 |
| `output_directory` | `OPEN / 未提供` | `0/1` | 精确有限目录；该字段不构成写授权。 |
| 授权 data owner identity | `OPEN / 未提供` | 未冻结 | 必须能对同一 scope revision/hash 留下可审计批准事件。 |

结论：当前为 `0/5 + owner OPEN`；因此 PATH-C/A 的 CP2 批准条件不成立。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR172.context.json` |
| capsule 状态 | `ready`；待 `meta-flow context check` 验证。 |
| read_profile | `compact`；默认只消费 capsule、CP0/CP1/CP2 result、CR summary、checkpoint 与 meta-pm return summary。 |
| 默认读取策略 | capsule-first；必须读取项以机器状态、CR summary、CP result、checkpoint 和 return summary 为限。 |
| 全文档读取 | 仅在 capsule 缺失、字段冲突、人工审计、深度评审或 schema 失败时展开，并记录 read-expansion event。 |
| 已知历史 debt | 全局 CR tracking 仍有 CR-010/018/031 legacy lifecycle 枚举债务；与 CR-172 决策无直接冲突，本 CR 不越界修复。 |
| 缺失 / waived | 五字段 5 项及 owner identity 缺失；不是 waiver，也不得静默补值。 |
| 业务视角整改 | `4/4 PASS`：业务动机、用户痛点、使用价值、业务触发；`SGQ=2/2`。 |

### Decision Collection Coverage

| 来源 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---:|---:|---:|---|
| CR-172 范围整改与正式 CR | scanned | 8 | 8 | 全部进入本门禁。 |
| CP0 conflict precheck | scanned | 2 groups | 0 | FU 状态冲突已修；legacy CR debt 仅审计披露。 |
| CP1 产品基线与 meta-pm discussion | scanned | 8 + 2 SGQ | 8 | 2 个 SGQ 已确认用户意图与 PATH-C 倾向；8 个正式 DQ 继续 OPEN，无遗漏。 |
| 授权与 claim ceiling | scanned | 9 deny classes | 0 | 固定 deny-default，不是可隐含放行项。 |
| HLD / Story / LLD / implementation | N/A | 0 | 0 | CP2 前依法尚未生成。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 8 | 下方 DQ-001～008；每项必须有明确值。 |
| 高风险策略确认 | 4 | 路径/blast radius、五字段、C2/C3 治理、joint approval。 |
| agent 默认处理 | 0 | 不允许 Agent 代填字段或 owner。 |
| 仅审计记录 | 3 | CR-171 关闭基线、FU-CR161-010 状态同步、legacy lifecycle debt。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| `DQ-CR172-001` | runtime_authorization | 五字段具体值与授权 data owner 是否可冻结？ | `not_freezable_currently`：接受当前 `0/5 + owner OPEN` 事实，进入 PATH-B | 提供 5/5 具体值 + owner identity | 推荐不伪造授权，代价是 activation 延后；备选可解锁 C1-first，但必须给出真实、有限、可审计值。 | 任一字段缺失/模糊仍批准 PATH-C 会造成授权逃逸。 | 补齐 5/5+owner 后恢复 CR-172、重开 CP2；任何 revision 变化需重新批准。 |
| `DQ-CR172-002` | scope | FU-CR164-004 是否独立？ | `separate_methodology_cr` | 满足完整 merge 条件时 `joint_merge` | 独立 CR 审批/回滚最清晰；合并可减少 CR，但混合方法学与数据授权域。 | 审批人或风险边界不同会使联合范围失真。 | 任一 merge condition 或双 ledger 事件缺失即强制拆分。 |
| `DQ-CR172-003` | implementation | C1 是否支持 effective count 不可用降级？ | `typed_unavailable_required` | `block_c1_until_estimator` | 推荐可先验证 C1 通道且 truthful fail-closed；备选耦合 estimator、进度慢。 | 禁止把 `raw_trial_count` 重标为 effective count；否则 evidence 不可信。 | 若设计无法保持 `C1 computable=false`，退回 requirement-clarification。 |
| `DQ-CR172-004` | risk_acceptance | 最终采用 PATH-B/C/A？ | 当前 `PATH-B`；在 DQ-001=5/5+owner 时条件推荐 `PATH-C` | `PATH-A` 或 `reject` | PATH-B 最安全但不完成 activation；PATH-C blast radius 最小；PATH-A CR 少但首次归因/回滚更难。 | PATH-C/A 与五字段条件不一致时门禁无效。 | 五字段不可冻→PATH-B；5/5+owner→PATH-C；PATH-A 需显式三 producer 风险接受。 |
| `DQ-CR172-005` | follow_up_tracking | E1 何时触发？ | `before_later_admission_pass_or_pass_with_risk` | 提前到 CR-172 CP8 | 推荐与 CR-171 定义一致，不让 design/activation CP8 无谓阻塞；提前更保守但语义错误。 | 延后越过 admission 决策会违反 verifier fence。 | E1 定义变化时重审 FU-006 waiver；CR-172 CP8 不触发 E1。 |
| `DQ-CR172-006` | scope | OI-CR171-005 是否排除？ | `independent_revalidation_audit_lane` | 扩入 CR-172 | 推荐保持 audit 与 activation 的审批/证据边界独立。 | 扩入会混合 classification、历史审计和新数据授权。 | 如需 classification，另行建立 revalidation/audit CR。 |
| `DQ-CR172-007` | follow_up_tracking | PATH-C 后 C2/C3 如何治理？ | `two_independent_runtime_high_risk_crs`，连同 CR-172 总 activation CR=`3` | `same_parent_sequential_slices` | 独立 CR 的 owner、回滚、证据和风险接受最清晰；slice 数少但更耦合。 | 模糊“后续 CR 或 slice”会使总 CR 数与审批边界不可估。 | slice 仅在同五字段 revision、同审批/风险/回滚、C1 CP7 无高风险、独立证据/CP6/CP7、无新增权限/冲突时可切换。 |
| `DQ-CR172-008` | risk_acceptance | 不同 owner 合并时如何记录联合审批？ | `dual_owner_same_revision_hash_ledger_with_intersection` | estimator 保持独立，不启用联合审批 | 推荐可机械验证，且权限取更严格交集；独立方案最清晰但增加 CR。 | partial approval、hash mismatch 或 revision 变化会导致授权被宽松解释。 | 两位 owner 必须对同一 approval_group/revision/hash 分别批准；任一缺失或变更使旧批准失效并拆分。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | `8/8`，均为 `OPEN`。 |
| 推荐回复 | 当前证据下回复 `approve`，接受 PATH-B 与其余七项推荐；或用 `修改:` 补齐 5/5+owner 并选择 PATH-C。 |
| 如果回复 approve | 不进入 CR-172 CP3；转为 FU-CR164-004 独立离线方法学前置，CR-172 activation deferred，之后仍需恢复并重开 CP2。 |
| 选择 PATH-C 的最低输入 | `DQ-CR172-001` 必须含 `data_release`、`datasets`、`date_range`、`read_identity`、`output_directory`、`data_owner`；同时 `DQ-CR172-004=PATH-C`。 |
| reject | CR-172 保持 CP2 pending；Stage 3 activation 继续不启动。 |

## Entry Criteria

- [x] CR-171 已完整关闭，CR-172 已正式创建并采用 `runtime-high-risk` route。
- [x] FU-CR161-010 stale 状态已同步为 `closed-by-CR171`。
- [x] CP0 fresh conflict precheck `PASS`；无 CR-172 直接冲突。
- [x] CP1 产品基线增量 `PASS`：1/1 use case、8/8 P0、8/8 scenarios、8/8 matrix。
- [x] 三项评审改进已写入 CR、需求、MVP、场景与 backlog。
- [x] UC-58 业务视角整改 `4/4 PASS`，SGQ `2/2`；CP1 与 CP2 自动预检重跑均 `PASS`，blocker=`0`。
- [x] 真实 lake/NAS、credential、provider、write、computation、runtime、trading 操作计数均为 `0`。
- [x] 五字段与 owner 缺失已显式披露，不被伪装为已冻结。

## Checklist

| ID | 需审查项 | 推荐选择 | 选择后的可观察后果 |
|---|---|---|---|
| `DQ-CR172-001` | 五字段与 owner | 当前 `not_freezable_currently` | 选择 PATH-B；不得进入 activation 设计。 |
| `DQ-CR172-002` | effective-trial 所有权 | `separate_methodology_cr` | 方法学风险与数据授权风险分离。 |
| `DQ-CR172-003` | C1 降级 | `typed_unavailable_required` | estimator 未完成时 C1 通道可存在，但 `C1 computable=false`。 |
| `DQ-CR172-004` | PATH | 当前 `PATH-B`；5/5+owner 后 `PATH-C` | PATH-B 完成不等于 activation；PATH-C 只激活 C1。 |
| `DQ-CR172-005` | E1 | `before_later_admission_pass_or_pass_with_risk` | CR-172 CP8 不触发 E1。 |
| `DQ-CR172-006` | OI-005 | `independent_revalidation_audit_lane` | CR-172 不关闭 OI-005。 |
| `DQ-CR172-007` | C2/C3 | `two_independent_runtime_high_risk_crs` | 默认 activation CR 总数为 3。 |
| `DQ-CR172-008` | joint approval | `dual_owner_same_revision_hash_ledger_with_intersection` | partial/mismatch/revision change 均 fail-closed。 |

### 不可被本次批准推导的事项

- Stage 3 已启动、entry-ready、mature admission PASS、aggregate 完成或 CR155 promotion。
- 任何真实 lake/NAS/provider/credential/env read、write、catalog/current-pointer mutation。
- C1/C2/C3/C4 computation、producer binding 执行、runtime、simulation、paper/live trading。
- OI-005 historical classification/revalidation、repair、backfill、rerun 或 manifest rewrite。
- FU-006、FU-009、aggregate、alpha-decay、publish/deploy 或 Git remote write。

## Exit Criteria

- [x] 8 个 DQ 均由用户明确批准、修改或拒绝。
- [x] 若选择 PATH-C/A，五字段必须达到 `5/5` 且 data owner identity 已提供；本次选择 PATH-B，故该分支为 N/A，且 activation 继续 blocked。
- [x] 若选择 PATH-B，明确接受“只完成 estimator 前置，五字段可冻结后恢复 CR-172 并重开 CP2”。
- [x] 若保留 joint approval 选项，同 revision/hash、双 owner ledger、权限交集与失效规则被接受；当前选择独立 estimator CR，不启用 joint merge。
- [x] 保持 claim ceiling：`stage3_started=false`、`stage3_entry_ready=false`、`real_computation_authorized=false`。
- [x] CP2 approval 只决定路由和治理合同，不构成真实数据或运行时授权。

## Deliverables

- 已生成：`process/changes/CR-172.md`、`process/checks/CP0-CR172-REQUEST-INTAKE.result.json`、`process/checks/CP1-CR172-USE-CASE-COMPLETENESS.result.json`。
- 已生成：`process/checks/CP2-CR172-REQUIREMENTS-BASELINE.result.json`、`process/context/CP2-CR172.context.json`、本人工 checkpoint。
- 已生成：8 个产品基线增量文件、CP2 discussion log/checkpoint、meta-pm return summary。
- 已生成：`process/handoffs/CR172-CP2-META-PM-UC-REMEDIATION-RETURN-SUMMARY.md`；记录 4/4 业务锚点、2/2 SGQ、8/8/8 覆盖复核与零授权扩张。
- 待本门禁批准后：PATH-B 转独立 estimator CR；或 5/5+owner 下 PATH-C 转 CP3 solution-design。

## 人工审查结果

状态：**approved（selected path=`PATH-B`；prior=`changes_requested`）**

用户于 `2026-07-16T11:35:08+08:00` 明确指示：评估四项非阻塞问题，完成必要修改后即视为批准 CP2，并继续推进到下一个人工门禁。四项处理结果如下：

1. “策略 X”已定义为 CP2 业务示例占位；PATH-B strategy-agnostic=`1/1`；未来 PATH-C/A 的 CP3 Entry 冻结 `strategy_id + strategy_name`，CP6 C1 evidence 同一身份锚定，缺失或 mismatch fail-closed。
2. `STATE.current.json` 已登记 `R-CR172-*` 五项风险，覆盖 scope authorization、shared PIT/lineage、raw/effective alias、producer-local overclaim 与 joint approval ambiguity。
3. `source_refs` 采用累计审计策略，保留历史 CR161 refs，同时追加 CR172 的 CR、CP0/CP1/CP2、context、checkpoint、gate launch 与本轮 handoff 关键引用。
4. `authz_policy_refs` 已删除两项 CR170 scoped Git remote-write 授权，并显式加入 `NO_GIT_REMOTE_WRITE`；CR170 授权不跨 CR 继承。

依据当前不可推断事实 `five fields=0/5`、`data owner=OPEN`，本次 `approve` 精确解析为以下 8 项决定：

| 决策 ID | 已批准值 |
|---|---|
| `DQ-CR172-001` | `not_freezable_currently` |
| `DQ-CR172-002` | `separate_methodology_cr` |
| `DQ-CR172-003` | `typed_unavailable_required` |
| `DQ-CR172-004` | `PATH-B` |
| `DQ-CR172-005` | `before_later_admission_pass_or_pass_with_risk` |
| `DQ-CR172-006` | `independent_revalidation_audit_lane` |
| `DQ-CR172-007` | `two_independent_runtime_high_risk_crs` |
| `DQ-CR172-008` | `dual_owner_same_revision_hash_ledger_with_intersection`（仅未来拟 joint merge 时适用；当前 estimator 独立） |

批准效果仅为：CR-172 activation 进入 `blocked/deferred`，独立启动 FU-CR164-004 的离线 estimator 方法 CR；其完成不等于 activation 完成，五字段可冻结后仍需恢复 CR-172 并重开 CP2。批准后仍保持：

- `ready_for_design=false`（仅对 CR-172 activation）
- `stage3_started=false`
- `stage3_entry_ready=false`
- `real_lake_read=false`
- `real_computation_or_runtime=false`
- `write=false`
- `git_remote_write=false`
