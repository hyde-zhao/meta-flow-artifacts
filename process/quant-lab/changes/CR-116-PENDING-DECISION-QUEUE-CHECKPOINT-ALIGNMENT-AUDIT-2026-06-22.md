---
cr_id: "CR-116"
cr_type: "process"
cr_kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "low"
workflow_mode_before: "standard"
workflow_mode_after_change: "fast-lane"
fast_lane_upgrade_reason: ""
rollback_to: "CR115 closed READY with no active formal CR"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T17:31:28+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T17:46:19+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-114"
source_checkpoint: "process/checkpoints/CP8-CR114-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR114-01"
follow_up_type: "pending-decision-queue-checkpoint-alignment-audit"
risk_class: "low-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If the audit requires bulk historical STATE rewrite, source code changes, tests changes, checker implementation changes, runtime execution, NAS access, credential read, trading write or publish, stop CR116 and create a separate scoped CR."
acceptance_criteria: "CR116 audits current pending decision queue and checkpoint DQ table alignment, classifies stale historical accepted DQ records, proposes a scoped current-gate correction path, and keeps source/runtime work out of scope."
close_condition: "CP8 approved at 2026-06-22T17:46:19+08:00; CR116 closed READY. No source code, tests, checker implementation, bulk historical STATE rewrite, runtime, NAS, credentials, trading or publish action is authorized."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-116 Pending Decision Queue / Checkpoint Alignment Audit

## 变更描述

用户明确要求：

> 启动 FU-CR114-003，继续低风险地清理 pending decision queue 与 checkpoint 对齐问题

本 CR 从 `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md#FU-CR114-003` 转入正式 CR。目标是在 no-code / no-runtime 范围内，审计并收敛当前 `human_gate_decisions.pending_human_decisions[]` 与 CP2 / CP3 / CP5 / CP8 checkpoint Decision Brief 中 DQ 表的对齐规则。

本 CR 不批量重写历史 `STATE.md`，不删除历史审计事实，不修改源码、不修改 tests、不修改 checker implementation，不启动 CR105，不恢复 CR089，不连接 QMT / MiniQMT / XtQuant / gateway runtime，不访问 NAS，不读取凭据 / 账户 / raw log，不执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write 或 catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-114` |
| 来源检查点 | `process/checkpoints/CP8-CR114-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR114-01` |
| follow-up 类型 | `pending-decision-queue-checkpoint-alignment-audit` |
| 来源候选 | `FU-CR114-003` |
| 风险等级 | `low-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若发现需要批量历史 STATE 重写、源码、checker 实现、runtime、NAS、凭据、交易写或 publish，另起独立 CR。 |
| 验收标准 | 输出 pending queue / checkpoint DQ 对齐审计、当前门禁队列规则和后续分流；不改代码、不启动 runtime。 |
| 关闭条件 | CP2 确认范围，CP8 确认对齐结论和 follow-up tracking。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
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
    write_audit_evidence: true
    update_cr_index: true
    update_state_summary: true
    bulk_historical_state_rewrite: false
  runtime:
    qmt: false
    miniqmt: false
    xtquant: false
    gateway: false
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
  trading:
    submit: false
    cancel: false
    buy: false
    sell: false
    simulation: false
    live: false
  provider_lake_catalog:
    provider_fetch: false
    lake_write: false
    catalog_publish: false
```

允许动作仅限：

- 读取 CR114 / CR115 follow-up tracking、CR-INDEX、STATE 顶层摘要、`human_gate_decisions` 当前门禁字段、CP2 / CP8 checkpoint、context capsule 和相关 process 证据。
- 生成 CR116 audit draft、context capsule、CP2 预检和人工审查稿。
- 分类 stale accepted DQ、current pending DQ、checkpoint DQ table 和 launch prompt decision IDs 的一致性缺口。
- 运行本地只读状态检查或 git status；不启动服务、不连接外部 runtime。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-AUDIT-CR116.md` | 新增 | N/A | 文件头部修订记录 | cp2-approved |
| `process/context/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-CONTEXT.yaml` | 新增 | N/A | 不适用 | cp2-approved |
| `process/checks/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-PRECHECK.md` | 新增 | N/A | 不适用 | approved |
| `process/checkpoints/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-REVIEW.md` | 新增 | N/A | 不适用 | approved |
| `process/context/CP8-CR116-DELIVERY-CONTEXT.yaml` | 新增 | N/A | 不适用 | approved |
| `process/release/RELEASE-CONTEXT-CR116.yaml` | 新增 | N/A | 不适用 | approved |
| `process/checks/CP8-CR116-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/checkpoints/CP8-CR116-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 FU-CR114-003 历史来源并标记 active | 不适用 | cp2-approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 追加 CR116 active 项，不删除历史项 | 不适用 | cp2-approved |
| `process/STATE.md` | 原文档更新 | 只刷新顶层人类摘要和当前门禁提示；不批量重写历史长表 | 不适用 | cp2-approved |
| 源码 / tests / checker implementation | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做过程治理审计。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；未来实现候选启动时再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | pending queue / checkpoint gate | true | 输出对齐规则和清理建议，不创建 Story 实现批次。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 继续禁止源码、checker implementation、runtime、NAS、凭据、交易写和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/checks` / `process/checkpoints` / `docs/design` | true | 生成 CR116 scope gate 和后续 CP8 收口证据。 |

## 冲突预检结论

| 检查维度 | 结果 | 证据 | 说明 |
|---|---|---|---|
| active formal CR | PASS | `meta-flow check cr-tracking --project-root .` | 启动前 active formal CR 为 none。 |
| blocked formal CR | PASS_WITH_NOTE | `CR-089` | CR089 仍 blocked；CR116 不恢复 CR089、不触碰 runtime / NAS / interface。 |
| 影响面重叠 | PASS | `FU-CR114-003` impact surface | 仅涉及 pending decision queue、checkpoint DQ table 和 state summary。 |
| 高风险授权 | PASS | 本 CR authorization policy | 不授权 runtime、凭据、NAS、交易写、publish。 |
| 源码 / tests / checker implementation | PASS | 本 CR authorization policy | 不授权源码、tests 或 checker implementation 变更。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档 / 规则 / 排序修改 | true | 只做 audit、规则说明和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不修改架构契约或权限边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改接口和文件所有权，不创建实现 Story。 |
| 需要 HLD / LLD 才能解释影响 | false | 本轮只输出治理审计；实现候选未来单独启动。 |
| 是否保持 fast-lane | true | 保留 CP2 和 CP8 审查证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR116 不启动代码实现。

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR116-01 | scope | 接受 CR116 只做 no-code/no-runtime pending queue 与 checkpoint DQ 对齐审计，不改代码、不启动 runtime | A. 暂停 CR116；B. 改为 CR115 checker implementation；C. 改为批量历史 STATE rewrite 设计 CR | 若用户要求源码、runtime 或批量历史重写，另起独立 CR |
| DQ-CP2-CR116-02 | implementation | 接受当前原则：checkpoint DQ 表是人工门禁真相源；STATE `pending_human_decisions[]` 只承载当前待确认项，历史 accepted DQ 不应继续被解释为 pending | A. 保持现状只记录风险；B. 把历史 accepted DQ 全量迁出 STATE；C. 改 checker 实现容忍历史混入 | 推荐方案可低风险收敛语义；B/C 需要更大授权 |
| DQ-CP2-CR116-03 | runtime_authorization | 继续禁止源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 被 CR116 隐式启动 | A. 另起源码实现 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading authorization gate | 若用户明确要求高风险验证，停止 CR116 默认路线并创建独立 gate |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T17:39:56+08:00 |
| 接受决策 | `DQ-CP2-CR116-01`、`DQ-CP2-CR116-02`、`DQ-CP2-CR116-03` |
| 下一门禁 | CP8 delivery readiness |
| 不授权项 | 源码修改、tests 修改、checker implementation change、批量历史 STATE rewrite、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## CP8 审批结果

| 字段 | 内容 |
|---|---|
| 审批结论 | `approved` |
| 审批人 | user |
| 审批时间 | 2026-06-22T17:46:19+08:00 |
| 接受决策 | `DQ-CP8-CR116-01`、`DQ-CP8-CR116-02`、`DQ-CP8-CR116-03` |
| 关闭状态 | `closed-current-delivery / READY` |
| 后续候选 | `FU-CR115-001`、`FU-CR114-004`、`FU-CR114-002` 仅保留为 candidate，不自动启动。 |
| 不授权项 | 源码修改、tests 修改、checker implementation change、批量历史 STATE rewrite、CR105、CR089 恢复、runtime、交易写、NAS、凭据、provider/lake/catalog publish |

## 当前状态

| 字段 | 内容 |
|---|---|
| 当前门禁 | closed |
| 自动预检 | `process/checks/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-PRECHECK.md` |
| 人工审查稿 | `process/checkpoints/CP2-CR116-PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-REVIEW.md` |
| Audit draft | `docs/design/PENDING-DECISION-QUEUE-CHECKPOINT-ALIGNMENT-AUDIT-CR116.md` |
| 下一步 | CR116 已关闭为 READY；后续候选仅保留在 follow-up tracking 中，需用户明确选择并另起正式 CR 才能启动。 |
