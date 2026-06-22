---
cr_id: "CR-107"
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
rollback_to: "CR106 closed READY baseline with no active formal CR"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T13:38:58+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T13:50:24+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-106"
source_checkpoint: "process/checkpoints/CP8-CR106-DELIVERY-READINESS.md"
source_decision_id: "USER-20260622-START-CR107-REDESIGN-SCOPE-INTAKE"
follow_up_type: "redesign-scope-intake"
risk_class: "medium-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If the user explicitly requests order-write, submit/cancel, simulation/live, runtime, NAS, credentials or provider/lake/catalog actions, stop CR107 and start a separate high-risk gate."
acceptance_criteria: "CR107 produces a redesign scope intake, candidate priority table, non-scope boundary, and next CR recommendation; no source code is modified and no high-risk runtime is started."
close_condition: "Closed by CP8 approval at 2026-06-22T13:53:47+08:00 as READY. Scope intake and candidate prioritization are written; default next low-risk candidate is FU-CR107-001."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-107 Redesign Scope Intake and Candidate Prioritization

## 变更描述

用户明确要求：

> 下一步启动 CR107，做 redesign 范围收口和候选项排序；暂不改代码，暂不启动高风险 runtime。

本 CR 承接 CR106 的 READY 结论。CR106 已证明当前 baseline hygiene 与代码健康预检无阻断，当前证据不要求源码整改。因此 CR107 的目标不是实现或修复代码，而是把 redesign 后续工作拆成可决策、可排队、可追溯的候选项。

本 CR **不是** CR105，不进入 order-write / submit-cancel / simulation-live，不执行 QMT / MiniQMT / XtQuant / gateway runtime，不访问 NAS，不读取凭据 / 账户 / raw log，不执行 provider / lake / catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-106` |
| 来源检查点 | `process/checkpoints/CP8-CR106-DELIVERY-READINESS.md` |
| 来源决策 ID | `USER-20260622-START-CR107-REDESIGN-SCOPE-INTAKE` |
| follow-up 类型 | `redesign-scope-intake` |
| 风险等级 | `medium-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 用户明确要求高风险 runtime、交易写、NAS、凭据或 publish 时，停止 CR107 并另起独立门禁。 |
| 验收标准 | 形成 scope intake、候选项排序、非范围边界和下一 CR 建议；不修改源码，不启动 runtime。 |
| 关闭条件 | CP2 确认范围基线，CP8 确认候选路由和台账同步。 |

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

- 读取仓库状态、CR 台账、CR106 关闭证据、baseline 预检和现有设计治理文档。
- 生成 CR107 scope intake、候选项排序、CP2 / CP8 检查点和 context capsule。
- 对候选 CR 做治理层分类：推荐、延后、转 Spike、保持 blocked、需要高风险门禁。
- 运行本地治理 / 台账一致性检查，如 `meta-flow check cr-tracking --project-root .`。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md` | 新增 | N/A | 文件头部修订记录 | pending |
| `docs/product/USE-CASES.md` | 不变 | 既有基线保留 | 不适用 | pending |
| `docs/product/REQUIREMENTS.md` | 不变 | 既有基线保留 | 不适用 | pending |
| `docs/design/MODULE-BOUNDARIES.yaml` | 不变 | CR106 已补齐治理事实源 | 不适用 | pending |
| `docs/design/PACKAGE-IDENTITY.yaml` | 不变 | CR106 已补齐治理事实源 | 不适用 | pending |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前需求真相不改变 | 不适用 | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 索引追加 CR107 active 项，不删除历史项 | 不适用 | pending |
| `process/STATE.md` | 原文档更新 | 人类摘要记录 CR107 active state，不删除历史状态 | 不适用 | pending |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR106 READY baseline | CR107 redesign scope intake | 原文保留 | CR107 消费 CR106 的“无需源码整改”结论，进入范围收口。 |
| `FU-CR101-001 / CR105` | CR107 forbidden scope | 原文保留 | order-write / simulation-live 继续保留为高风险候选，不由 CR107 启动。 |
| CR089 blocked readiness | CR107 candidate routing | 原文保留 | CR107 只分类 CR089，不恢复 CR089。 |
| CR-026 / CR-027 / CR-028 / RA-CR097-001 | CR107 candidate priority table | 原文保留 | CR107 只给出推荐路由，不自动启动候选项。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只形成 redesign scope intake。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；候选项未来启动时再独立定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR backlog / candidate queue | true | 输出候选项排序和下一 CR 建议，不创建 Story 实现批次。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 明确禁止 CR105、runtime、交易写、NAS、凭据、账户原文和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/checks` / `process/checkpoints` / `docs/design` | true | 生成 CR107 scope intake、CP2 scope gate 和后续 CR routing evidence。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`CR106 closed READY baseline with no active formal CR`。
- 需要重新确认的对象：
  - `process/changes/CR-107-REDESIGN-SCOPE-INTAKE-AND-CANDIDATE-PRIORITIZATION-2026-06-22.md`
  - `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md`
  - `process/context/CP2-CR107-REDESIGN-SCOPE-INTAKE-CONTEXT.yaml`
  - `process/checks/CP2-CR107-REDESIGN-SCOPE-INTAKE-PRECHECK.md`
  - `process/checkpoints/CP2-CR107-REDESIGN-SCOPE-INTAKE-REVIEW.md`
  - `process/changes/CR-INDEX.yaml`
  - `process/STATE.md`

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | true | 只做 scope intake、候选项排序和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不改架构契约，不放大权限边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改接口和文件所有权，不创建实现 Story。 |
| 需要 HLD / LLD 才能解释影响 | false | 只形成下一步排序；未来候选项启动后再进入 CP3 / CP5。 |
| 是否保持 fast-lane | true | 可使用 fast-lane 的文档 / 治理路径，但保留 CP2 和 CP8 审查证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR107 不启动代码实现。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR107 和 CP2 scope gate | 用户明确启动 CR107 | CR107、scope intake draft、CP2 自动预检、CP2 人工审查稿 | CP2 pending | 等待用户确认 scope routing |
| 2 | `host-orchestrator` | 收敛 redesign 候选项排序 | CP2 approved | candidate priority table / next CR recommendation | CP8 pending | 用户确认关闭或启动下一 CR |
| 3 | `host-orchestrator` | 同步 CR tracking | CR107 结论 | CR-INDEX / STATE / cr-tracking check | CP8 | active formal CR 恢复为 none |

## 自动终验授权

- 是否启用：false。
- 授权范围：不适用。
- 适用检查点：CP8。
- 自动通过条件：N/A。
- 授权原文：N/A。
- 授权时间：N/A。

## 后续事项台账

- 是否存在后续事项：待 CP8 判断。
- 台账路径：`process/changes/CR-107-FOLLOW-UP-TRACKING-2026-06-22.md`。
- CR 索引路径：`process/changes/CR-INDEX.yaml`。
- 一致性检查：`meta-flow check cr-tracking --project-root .`。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR107-001 | Redesign Backlog Decision Gate | candidate | CR | 1 |  | blocked_by=CR107-CP8 | 未启动 | 等待 CR107 确认候选排序 | 若 CP8 确认，可启动低风险 backlog decision gate。 |
| FU-CR107-002 | Architecture-only Redesign HLD Gate | candidate | CR | 2 |  | blocked_by=CR107-CP8 | 未启动 | 需要先确认 redesign scope | 若用户选择进入架构设计，另起 CP3/HLD CR。 |

## 处理结论

- 审批结论：`cp8-approved-user-20260622-ready`
- [ ] 自动批准（低风险）
- [x] 待人工确认（中风险）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-106` | Baseline hygiene closed READY；当前证据不要求源码整改。 |
| Branch | `precheck/ql-rd-000-redesign-baseline` | 当前 redesign baseline 分支。 |
| Scope Draft | `docs/design/REDESIGN-SCOPE-INTAKE-CR107.md` | CR107 范围收口和候选排序草案。 |
| Forbidden Candidate | `FU-CR101-001 / CR105` | order-write / simulation-live 方向，不由 CR107 启动。 |
| Blocked CR | `CR-089` | 保持 blocked，不由 CR107 恢复。 |

## CP8 关闭结果

| 字段 | 内容 |
|---|---|
| 关闭时间 | 2026-06-22T13:53:47+08:00 |
| 关闭状态 | `closed-current-delivery / READY` |
| 用户确认 | `approve` |
| 默认下一候选 | `FU-CR107-001 Redesign Backlog Decision Gate` |
| 备选候选 | `FU-CR107-002 Architecture-only Redesign HLD Gate` |
| 延后候选 | `CR-026`、`CR-027`、`CR-028` |
| 不启动候选 | `RA-CR097-001`、`CR-089`、`FU-CR101-001 / CR105` |
| 不授权范围 | 源码修改、CR105、CR089 恢复、runtime、交易写、NAS、凭据、账户原文、provider/lake/catalog publish |
