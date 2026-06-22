---
cr_id: "CR-110"
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
rollback_to: "CR109 closed READY with FU-CR109-001 selected"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T14:34:43+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T14:47:29+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-109"
source_checkpoint: "process/checkpoints/CP8-CR109-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR109-01"
follow_up_type: "state-cr-tracking-contract-normalization-plan"
risk_class: "medium-governance-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If normalization requires source-code changes, checker implementation changes, runtime access, NAS, credentials, trading or publish actions, create separate scoped CRs instead of expanding CR110."
acceptance_criteria: "CR110 defines the State / CR tracking / follow-up row contract normalization scope, inventory, priority and follow-up route without modifying source code or starting runtime."
close_condition: "CP2 approved CR110 scope at 2026-06-22T14:47:29+08:00; CP8 approved at 2026-06-22T15:04:13+08:00 and closes CR110 as READY. Implementation, checker, STATE historical cleanup and runtime work remain follow-up candidates only."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-110 State / CR Tracking Contract Normalization Plan

## 变更描述

用户在 CR109 CP8 后明确要求推进候选 CR。按 CR109 follow-up tracking 的优先级，本 CR 将 `FU-CR109-001 State / CR Tracking Contract Normalization Plan` 转为正式 CR110。

本 CR 只做治理和合同归一化计划，不改源码、不修改 checker implementation、不启动 HLD / LLD / Story 实现，不启动 runtime，不恢复 CR089，不启动 CR105，不访问 NAS，不读取凭据、账户、raw log，不执行 provider / lake / catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-109` |
| 来源检查点 | `process/checkpoints/CP8-CR109-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR109-01` |
| follow-up 类型 | `state-cr-tracking-contract-normalization-plan` |
| 风险等级 | `medium-governance-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若归一化需要源码 / checker 实现修改、runtime、NAS、凭据、交易或 publish 工作，必须另起独立 CR。 |
| 验收标准 | 形成 State / CR tracking / follow-up row 合同归一化范围、优先级、非范围边界和后续路线。 |
| 关闭条件 | CP8 确认归一化计划后关闭 CR110；若选中代码层整改，另起独立实现 CR。 |

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
    update_follow_up_tracking: true
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

- 读取 CR109 关闭证据、CR-INDEX、STATE 摘要和 follow-up tracking 台账。
- 生成 CR110 contract normalization draft、context capsule、CP2 / CP8 检查证据。
- 对 State / CR tracking / follow-up row 合同项做盘点、排序、合并、延后、取消或转下一 CR 建议。
- 运行本地治理 / 台账一致性检查，如 `meta-flow check cr-tracking --project-root .` 和 `meta-flow check human-gate --checkpoint ...`。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/STATE-CR-TRACKING-CONTRACT-NORMALIZATION-CR110.md` | 新增 | N/A | 文件头部修订记录 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 索引追加 CR110 active 项，不删除历史项 | 不适用 | approved |
| `process/changes/CR-109-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | FU-CR109-001 转 active，保留 FU-CR109-002/003 candidate | 不适用 | approved |
| `process/STATE.md` | 原文档更新 | 人类摘要记录 CR110 active state，不删除历史状态 | 不适用 | approved |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前需求真相不改变 | 不适用 | approved |
| 源码 / tests / checker implementation | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做治理 contract normalization plan。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；后续实现 CR 再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | state / cr tracking cleanup candidate queue | true | 输出合同归一化候选项排序和下一步建议。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 明确不授权 CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `docs/design` / `process/checks` / `process/checkpoints` | true | 生成 CR110 contract normalization draft、CP2 scope gate 和后续 CP8 收口证据。 |

## 初始归一化候选池

| 候选 | 默认动作 | 说明 |
|---|---|---|
| active formal CR source of truth | 推荐候选 | 明确 `CR-INDEX.active_crs[]`、formal CR frontmatter、`STATE.md.active_change` 的优先级和同步规则。 |
| gate_status enum contract | 推荐候选 | 统一 `cp2_pending`、`cp8_pending`、`closed`、`not_started` 等机器枚举，避免自然语言状态污染。 |
| follow-up row lifecycle contract | 推荐候选 | 统一 `candidate`、`active`、`closed`、`blocked`、`deferred` 在台账与索引中的字段。 |
| state summary staleness policy | 候选 | 明确 `process/STATE.md` 作为人类摘要时的陈旧字段处理方式。 |
| CR tracking checker expectation notes | 候选 | 将 checker 当前能识别的字段和常见失败模式固化成治理说明。 |

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR110-01 | scope | 接受 CR110 只做 no-code/no-runtime State / CR tracking contract normalization plan | A. 暂停 CR110；B. 改为 context capsule / human gate consistency review | 若用户要求实现或 runtime，另起独立 CR |
| DQ-CP2-CR110-02 | follow_up_tracking | 接受初始候选池，并默认优先整理 active formal CR source of truth、gate_status enum 和 follow-up row lifecycle | A. 只整理 CR-INDEX；B. 只整理 STATE 摘要；C. 只输出盘点不排序 | 若用户调整排序，更新本文件并重新发起 CP2 |
| DQ-CP2-CR110-03 | runtime_authorization | 继续禁止源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 被 CR110 隐式启动 | A. 另起源码实现 CR；B. 另起 runtime/NAS authorization gate | 若用户明确要求代码或 runtime，停止 CR110 默认路线并创建独立 gate |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T14:47:29+08:00 |
| 接受决策 | `DQ-CP2-CR110-01`、`DQ-CP2-CR110-02`、`DQ-CP2-CR110-03` |
| 下一门禁 | CP8 delivery readiness |
| 不授权项 | 源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## 归一化计划结论

| 排名 | 合同项 | CP2 后处理 | CP8 收口口径 |
|---:|---|---|---|
| 1 | active formal CR source of truth | 接受 | `CR-INDEX.active_crs[]` 是活动正式 CR 汇总入口；formal CR frontmatter 和 follow-up tracking 必须与其同步；`STATE.md.active_change` 是人类摘要且必须跟随索引刷新。 |
| 2 | gate_status enum contract | 接受 | 当前允许枚举为 `not_started`、`cp2_pending`、`cp3_pending`、`cp5_pending`、`cp8_pending`、`closed`、`blocked`；不得把 `closed_current_delivery` 等自然语言状态写入 `gate_status`。 |
| 3 | follow-up row lifecycle contract | 接受 | 候选行不占执行锁；转正式 CR 后，台账行和正式 CR 行必须同步为 `active`；正式 CR 关闭后，候选别名也必须同步为 `closed` 或保留为明确 candidate。 |
| 4 | state summary staleness policy | 后续候选 | 如需整理 STATE 历史长表或陈旧摘要，另起候选，不在 CR110 中重写历史。 |
| 5 | CR tracking checker expectation notes | 后续候选 | 如需修改 checker 或补机器 result/ledger，另起独立实现 / 工具治理 CR。 |

## 处理结论

- 审批结论：`cp8-approved-ready`
- [ ] 自动批准（低风险）
- [x] 人工确认已通过（中风险）
- [ ] 待人工审批（高风险）

## CP8 关闭结果

| 字段 | 内容 |
|---|---|
| 关闭结论 | `closed-current-delivery / READY` |
| 审批人 | user |
| 审批时间 | 2026-06-22T15:04:13+08:00 |
| 关闭范围 | active formal CR source of truth、gate_status enum、follow-up row lifecycle contract 计划 |
| 后续候选 | `FU-CR110-001 STATE Summary Staleness Cleanup Candidate`、`FU-CR110-002 CR Tracking Checker Expectation Notes Candidate` |
| 不授权项 | 源码修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-109` | Governance cleanup gate closed READY；选择 FU-CR109-001。 |
| Candidate | `FU-CR109-001` | 已由用户明确启动并转为 CR110。 |
| Scope Draft | `docs/design/STATE-CR-TRACKING-CONTRACT-NORMALIZATION-CR110.md` | CR110 contract normalization 草案。 |
| Retained Candidate | `FU-CR109-002` | Context Capsule / Human Gate Consistency Review，保持 candidate。 |
| Retained Candidate | `FU-CR109-003` | CP Result JSON / Ledger Gap Analysis，保持 candidate。 |
| Forbidden Candidate | `FU-CR101-001 / CR105` | order-write / simulation-live 方向，不由 CR110 启动。 |
| Blocked CR | `CR-089` | 保持 blocked，不由 CR110 恢复。 |
