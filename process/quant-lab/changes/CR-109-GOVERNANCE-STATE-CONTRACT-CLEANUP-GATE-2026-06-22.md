---
cr_id: "CR-109"
cr_type: "process-governance"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "fast-lane"
fast_lane_upgrade_reason: ""
rollback_to: "CR108 closed READY with FU-CR108-002 selected"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T14:13:19+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T14:24:01+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-108"
source_checkpoint: "process/checkpoints/CP8-CR108-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR108-01"
follow_up_type: "governance-state-contract-cleanup-gate"
risk_class: "medium-governance-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If cleanup requires source-code changes, checker implementation changes, runtime access, NAS, credentials, trading or publish actions, create separate scoped CRs instead of expanding CR109."
acceptance_criteria: "CR109 defines a no-code/no-runtime governance and state-contract cleanup decision set, ranks cleanup candidates, identifies which items can remain ledger/doc-only, and recommends the next CR without modifying source code or starting runtime."
close_condition: "User approved CR109 CP8 at 2026-06-22T14:34:43+08:00, selected FU-CR109-001 as the next candidate CR, and closed CR109 as READY without source-code or runtime authorization."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-109 Governance / State Contract Cleanup Gate

## 变更描述

用户明确要求：

> approve，并启动FU-CR108-002 Governance / State Contract Cleanup Gate

CR108 已关闭为 `closed-current-delivery / READY`，并把 `FU-CR108-002` 选为下一低风险候选。本 CR 将该候选转为正式 CR109，目标是收敛 redesign 前的治理 / 状态契约清洁范围和优先级。

本 CR **暂不改代码**，不修改 checker 实现，不启动 HLD / LLD / Story 实现，不启动 runtime，不恢复 CR089，不启动 CR105，不访问 NAS，不读取凭据、账户、raw log，不执行 provider / lake / catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-108` |
| 来源检查点 | `process/checkpoints/CP8-CR108-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR108-01` |
| follow-up 类型 | `governance-state-contract-cleanup-gate` |
| 风险等级 | `medium-governance-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若治理清洁需要源码 / checker 实现修改、runtime、NAS、凭据、交易或 publish 工作，必须另起独立 CR。 |
| 验收标准 | 形成治理 / 状态契约清洁候选池、优先级、不授权边界和下一 CR 推荐。 |
| 关闭条件 | CP8 确认治理清洁队列后关闭 CR109；若选中代码层整改，另起独立实现 CR。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process-governance` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready` |
| 门禁状态 | `closed` |
| 门禁模板 | `compact` |

## 结构化权限策略

```yaml
authorization_policy:
  source_code:
    modify: false
    refactor: false
    test_change: false
    checker_implementation_change: false
  process_artifacts:
    read: true
    write_governance_evidence: true
    update_cr_index: true
    update_state_summary: true
  nas:
    access: false
    list: false
    read: false
    write: false
    publish: false
    delete: false
  credentials:
    env_read: false
    secret_read: false
    account_read: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
  trading:
    submit: false
    cancel: false
    simulation: false
    live: false
  provider_lake_catalog:
    provider_fetch: false
    lake_write: false
    catalog_publish: false
```

允许动作仅限：

- 读取 CR108 关闭证据、follow-up tracking、CR-INDEX、STATE 摘要和当前治理文档。
- 生成 CR109 governance cleanup draft、context capsule、CP2 / CP8 检查证据。
- 对治理 / 状态契约候选项做排序、合并、延后、取消或转下一 CR 建议。
- 运行本地治理 / 台账一致性检查，如 `meta-flow check cr-tracking --project-root .` 和 `meta-flow check human-gate --checkpoint ...`。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/GOVERNANCE-STATE-CONTRACT-CLEANUP-CR109.md` | 新增 | N/A | 文件头部修订记录 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 索引记录 CR109 CP2 approved / CP8 pending，不删除历史项 | 不适用 | approved |
| `process/STATE.md` | 原文档更新 | 人类摘要记录 CR109 CP2 approved / CP8 pending，不删除历史状态 | 不适用 | approved |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前需求真相不改变 | 不适用 | approved |
| 源码 / tests / checker implementation | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做治理 cleanup gate。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；后续具体实现 CR 再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | governance cleanup candidate queue | true | 输出治理清洁候选项排序和下一步建议。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 明确不授权 CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `docs/design` / `process/checks` / `process/checkpoints` | true | 生成 CR109 governance cleanup draft、CP2 scope gate 和后续 CP8 收口证据。 |

## 初始治理清洁候选池

| 候选 | 默认动作 | 说明 |
|---|---|---|
| State / active_change contract normalization | 推荐候选 | 收敛 `process/STATE.md` 人类摘要与 `CR-INDEX` / active formal CR 的一致性口径。 |
| CR-INDEX item schema normalization | 推荐候选 | 收敛 active / recent_items / items 重复字段、状态枚举和 follow-up row 映射。 |
| Follow-up tracking row consistency | 推荐候选 | 统一 formal CR、candidate、blocked/deferred 的生命周期字段。 |
| Context capsule naming and read-profile consistency | 候选 | 统一 CP2 / CP8 capsule 字段、read expansion log 和 must_read/read_if_needed 口径。 |
| Human gate launch / checkpoint consistency | 候选 | 确认人工门禁文件、发起消息、approved 回填语义一致。 |
| CP result JSON / ledger gap analysis | 延后 | 如果需要补机器 result / ledger，另行决定是否进入更大治理 CR。 |

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR109-01 | scope | 接受 CR109 只做 no-code/no-runtime governance / state contract cleanup gate | A. 暂停治理清洁；B. 改为 architecture-only HLD gate | 若用户要求实现或 runtime，另起独立 CR |
| DQ-CP2-CR109-02 | follow_up_tracking | 接受初始候选池，并默认优先整理 State / CR tracking / follow-up row 契约 | A. 优先 context capsule；B. 优先 human gate；C. 只做盘点不排序 | 若用户调整排序，更新本文件并重新发起 CP2 |
| DQ-CP2-CR109-03 | runtime_authorization | 继续禁止源码修改、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 被 CR109 隐式启动 | A. 另起源码实现 CR；B. 另起 runtime/NAS authorization gate | 若用户明确要求代码或 runtime，停止 CR109 默认路线并创建独立 gate |

## 处理结论

- 审批结论：`cp2-approved-user-20260622`
- [ ] 自动批准（低风险）
- [x] 人工确认通过（中风险）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-108` | Redesign backlog decision closed READY；选择 FU-CR108-002。 |
| Candidate | `FU-CR108-002` | 已由用户明确启动并转为 CR109。 |
| Scope Draft | `docs/design/GOVERNANCE-STATE-CONTRACT-CLEANUP-CR109.md` | CR109 governance cleanup 草案。 |
| Forbidden Candidate | `FU-CR101-001 / CR105` | order-write / simulation-live 方向，不由 CR109 启动。 |
| Blocked CR | `CR-089` | 保持 blocked，不由 CR109 恢复。 |

## CP2 审查结果

| 字段 | 结果 |
|---|---|
| CP2 审查 | `approved`，见 `process/checkpoints/CP2-CR109-GOVERNANCE-STATE-CONTRACT-CLEANUP-REVIEW.md` |
| 审查时间 | 2026-06-22T14:24:01+08:00 |
| 接受的决策项 | `DQ-CP2-CR109-01`、`DQ-CP2-CR109-02`、`DQ-CP2-CR109-03` |
| 当前门禁 | `cp8_pending` |
| 下一步 | 准备 CR109 CP8 收口材料，确认治理清洁队列和下一候选是否仅保留为 candidate。 |
| 不授权范围 | 源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取、provider/lake/catalog publish |

## CP8 收口准备

| 字段 | 结果 |
|---|---|
| Release Context | `process/release/RELEASE-CONTEXT-CR109.yaml` |
| CP8 自动预检 | `process/checks/CP8-CR109-DELIVERY-READINESS.md` |
| CP8 人工审查稿 | `process/checkpoints/CP8-CR109-DELIVERY-READINESS.md` |
| Follow-up Tracking | `process/changes/CR-109-FOLLOW-UP-TRACKING-2026-06-22.md` |
| release_decision | `READY` |
| release_artifact_profile | `minimal` |
| 后续候选策略 | `FU-CR109-001` 已按用户要求转为 CR110；`FU-CR109-002`、`FU-CR109-003` 保持 candidate。 |
| 当前状态 | CR109 已关闭为 READY。 |

## CP8 关闭结果

| 字段 | 结果 |
|---|---|
| CP8 审查 | `approved`，见 `process/checkpoints/CP8-CR109-DELIVERY-READINESS.md` |
| 审查时间 | 2026-06-22T14:34:43+08:00 |
| 关闭结论 | `closed-current-delivery / READY` |
| 选择的下一候选 | `FU-CR109-001 State / CR Tracking Contract Normalization Plan` |
| 新正式 CR | `CR-110` |
| 保留候选 | `FU-CR109-002 Context Capsule / Human Gate Consistency Review`、`FU-CR109-003 CP Result JSON / Ledger Gap Analysis` |
| 不授权范围 | 源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取、provider/lake/catalog publish |
