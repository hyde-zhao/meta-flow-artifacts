---
cr_id: "CR-108"
cr_type: "product-scope"
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
rollback_to: "CR107 closed READY with no active formal CR"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T14:01:49+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T14:13:19+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-107"
source_checkpoint: "process/checkpoints/CP8-CR107-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR107-01"
follow_up_type: "redesign-backlog-decision-gate"
risk_class: "medium-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If backlog decision selects architecture, implementation, runtime, NAS, credentials, trading or publish work, create separate scoped CRs instead of expanding CR108."
acceptance_criteria: "CR108 converts CR107 redesign intake into a 2-4 item executable backlog decision set, with priority, gate type, non-scope boundary, and next CR recommendation; no source code is modified and no runtime is started."
close_condition: "User approved CR108 CP2 and CP8 at 2026-06-22T14:13:19+08:00, selected FU-CR108-002 as the next formal CR, and closed CR108 as READY without source-code or runtime authorization."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-108 Redesign Backlog Decision Gate

## 变更描述

用户明确要求启动：

> FU-CR107-001 Redesign Backlog Decision Gate

CR107 已关闭为 `closed-current-delivery / READY`，并把 `FU-CR107-001` 标记为默认下一低风险候选。本 CR 将该候选转为正式 CR108，目标是把 CR107 的 redesign scope intake 转换为一组可执行、可排序、可关闭或延后的后续 CR 决策。

本 CR **不改源码**，不进入 HLD / LLD / 实现，不启动 runtime，不恢复 CR089，不启动 CR105，不访问 NAS，不读取凭据、账户、raw log，不执行 provider / lake / catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-107` |
| 来源检查点 | `process/checkpoints/CP8-CR107-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR107-01` |
| follow-up 类型 | `redesign-backlog-decision-gate` |
| 风险等级 | `medium-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若 backlog 决策选择架构、实现、runtime、NAS、凭据、交易或 publish 工作，必须另起独立 CR。 |
| 验收标准 | 形成 2-4 个后续候选 CR 的优先级、门禁类型、非范围边界和下一 CR 推荐。 |
| 关闭条件 | CP8 确认 backlog decision 后关闭 CR108，并恢复 active formal CR 为 none。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `product-scope` |
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

- 读取 CR107 关闭证据、scope intake、release context 和当前 CR tracking。
- 生成 CR108 backlog decision draft、context capsule、CP2 / CP8 检查证据。
- 对候选项做治理层排序、合并、延后、取消或转下一 CR 建议。
- 运行本地治理 / 台账一致性检查，如 `meta-flow check cr-tracking --project-root .`。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/REDESIGN-BACKLOG-DECISION-CR108.md` | 新增 | N/A | 文件头部修订记录 | approved |
| `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md` | 不变 | CR107 baseline 保留 | 不适用 | approved |
| `docs/product/USE-CASES.md` | 不变 | 既有基线保留 | 不适用 | approved |
| `docs/product/REQUIREMENTS.md` | 不变 | 既有基线保留 | 不适用 | approved |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前需求真相不改变 | 不适用 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 索引关闭 CR108 并启动 CR109，不删除历史项 | 不适用 | approved |
| `process/STATE.md` | 原文档更新 | 人类摘要记录 CR108 closed 和 CR109 active state，不删除历史状态 | 不适用 | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR107 scope intake | CR108 backlog decision draft | 原文保留 | CR108 消费 CR107 的候选排序，不改写 CR107 历史结论。 |
| `FU-CR107-001` | `CR-108` | formalized | 用户明确启动该候选，转为正式 CR108。 |
| `FU-CR107-002` | CR108 candidate routing | 原文保留 | 仅作为 architecture-only HLD 备选，不由 CR108 自动启动。 |
| `CR-026` / `CR-027` / `CR-028` | CR108 deferred candidates | 原文保留 | 只做保留 / 延后 / 取消建议，不启动 Spike。 |
| `RA-CR097-001` / `CR-089` / `FU-CR101-001` | CR108 forbidden / high-risk routing | 原文保留 | 真实 runtime、CR089 恢复和 CR105 均不由 CR108 启动。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做 backlog decision。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；后续具体 CR 再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | redesign candidate queue | true | 输出 2-4 个后续候选 CR 的排序和下一步建议。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 明确不授权 CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `docs/design` / `process/checks` / `process/checkpoints` | true | 生成 CR108 backlog decision、CP2 scope gate 和后续 CP8 收口证据。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`CR107 closed READY with no active formal CR`。
- 需要重新确认的对象：
  - `process/changes/CR-108-REDESIGN-BACKLOG-DECISION-GATE-2026-06-22.md`
  - `docs/design/REDESIGN-BACKLOG-DECISION-CR108.md`
  - `process/context/CP2-CR108-REDESIGN-BACKLOG-DECISION-CONTEXT.yaml`
  - `process/checks/CP2-CR108-REDESIGN-BACKLOG-DECISION-PRECHECK.md`
  - `process/checkpoints/CP2-CR108-REDESIGN-BACKLOG-DECISION-REVIEW.md`
  - `process/changes/CR-INDEX.yaml`
  - `process/STATE.md`

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | true | 只做 backlog decision 和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不做 HLD / ADR 修改，不放大权限边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改源码或 Story 依赖。 |
| 需要 HLD / LLD 才能解释影响 | false | CR108 只决定后续 CR 队列；若选择 HLD，另起 CR。 |
| 是否保持 fast-lane | true | 可使用 fast-lane 的文档 / 治理路径，但保留 CP2 和 CP8 证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR108 不启动代码实现。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR108 和 CP2 scope gate | 用户明确启动 FU-CR107-001 | CR108、backlog decision draft、CP2 自动预检、CP2 人工审查稿 | approved | 用户已确认 backlog decision scope |
| 2 | `host-orchestrator` | 收敛 backlog decision | CP2 approved | 2-4 个后续候选 CR 的排序、保留 / 延后 / 取消建议 | approved | 用户选择 `FU-CR108-002` |
| 3 | `host-orchestrator` | 同步 CR tracking | CR108 结论 | CR-INDEX / STATE / cr-tracking check | closed | active formal CR 切换为 CR109 |

## 自动终验授权

- 是否启用：false。
- 授权范围：不适用。
- 适用检查点：CP8。
- 自动通过条件：N/A。
- 授权原文：N/A。
- 授权时间：N/A。

## 后续事项台账

- 是否存在后续事项：待 CP8 判断。
- 台账路径：`process/changes/CR-108-FOLLOW-UP-TRACKING-2026-06-22.md`。
- CR 索引路径：`process/changes/CR-INDEX.yaml`。
- 一致性检查：`meta-flow check cr-tracking --project-root .`。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR108-002 | Governance / State Contract Cleanup Gate | active | CR | 1 | `process/changes/CR-109-GOVERNANCE-STATE-CONTRACT-CLEANUP-GATE-2026-06-22.md` | active_cr=CR-109 | cp2_pending | 无 | 已按用户要求启动为 CR109。 |
| FU-CR108-001 | Architecture-only Redesign HLD Gate | candidate | CR | 2 |  | blocked_by=CR109 decision | not_started | 等待治理清洁决策 | 仅当用户选择架构路线时启动。 |
| FU-CR108-003 | Redesign Implementation Planning Gate | candidate | CR | 3 |  | blocked_by=CR109/HLD decision | not_started | 需先完成 backlog / HLD / governance 判断 | 仅在明确实现范围后启动。 |

## 处理结论

- 审批结论：`cp8-approved-user-20260622-ready`
- [ ] 自动批准（低风险）
- [x] 人工确认通过（中风险）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-107` | Redesign scope intake closed READY；默认下一候选为 FU-CR107-001。 |
| Candidate | `FU-CR107-001` | 已由用户明确启动并转为 CR108。 |
| Scope Draft | `docs/design/REDESIGN-BACKLOG-DECISION-CR108.md` | CR108 backlog decision 草案。 |
| Forbidden Candidate | `FU-CR101-001 / CR105` | order-write / simulation-live 方向，不由 CR108 启动。 |
| Blocked CR | `CR-089` | 保持 blocked，不由 CR108 恢复。 |

## CP2 / CP8 关闭结果

| 字段 | 结果 |
|---|---|
| CP2 审查 | `approved`，见 `process/checkpoints/CP2-CR108-REDESIGN-BACKLOG-DECISION-REVIEW.md` |
| CP8 收口 | `approved / READY`，见 `process/checkpoints/CP8-CR108-DELIVERY-READINESS.md` |
| 关闭时间 | 2026-06-22T14:13:19+08:00 |
| 选择的下一候选 | `FU-CR108-002 Governance / State Contract Cleanup Gate` |
| 新正式 CR | `CR-109` |
| 保留候选 | `FU-CR108-001 Architecture-only Redesign HLD Gate`、`FU-CR108-003 Redesign Implementation Planning Gate` |
| 不授权范围 | 源码修改、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户 / raw log 读取、provider/lake/catalog publish |
