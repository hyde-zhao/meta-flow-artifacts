---
cr_id: "CR-115"
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
rollback_to: "CR114 closed READY"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T17:05:02+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T17:23:57+08:00"
source: "cp8-follow-up"
linked_issue: ""
parent_cr: "CR-114"
source_checkpoint: "process/checkpoints/CP8-CR114-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR114-01"
follow_up_type: "human-gate-launch-message-checkable-draft-contract"
risk_class: "low-scope-no-runtime"
owner: "host-orchestrator"
revisit_condition: "If the review requires checker implementation, source code changes, tests changes, runtime execution, NAS access, credential read, trading write or publish, stop CR115 and create a separate scoped CR."
acceptance_criteria: "CR115 defines and reviews a checkable human gate launch message draft contract, proves it can be validated against checkpoint content, and keeps implementation/runtime work out of scope."
close_condition: "CP8 approved at 2026-06-22T17:23:57+08:00; CR115 closed READY. No source code, tests, checker implementation, runtime, NAS, credentials, trading or publish action is authorized."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-115 Human Gate Launch Message Checkable Draft Contract

## 变更描述

用户明确要求：

> approve，启动 FU-CR114-001 Human Gate Launch Message Checkable Draft Contract

本 CR 从 `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md#FU-CR114-001` 转入正式 CR。目标是在 no-code/no-runtime 范围内，定义和复盘一个可检查的人工作业门禁发起消息草案契约：发起消息必须能和 checkpoint 中的 Decision Brief、Context Capsule Summary、Decision Collection Coverage、待人工决策清单、不授权项和三个 exact 回复保持一致。

本 CR 不修改源码、不修改 tests、不修改 checker implementation，不启动 CR105，不恢复 CR089，不连接 QMT / MiniQMT / XtQuant / gateway runtime，不访问 NAS，不读取凭据 / 账户 / raw log，不执行 submit / cancel / buy / sell、simulation / live、provider fetch、lake write 或 catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-114` |
| 来源检查点 | `process/checkpoints/CP8-CR114-DELIVERY-READINESS.md` |
| 来源决策 ID | `DQ-CP8-CR114-01` |
| follow-up 类型 | `human-gate-launch-message-checkable-draft-contract` |
| 来源候选 | `FU-CR114-001` |
| 风险等级 | `low-scope-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | 若发现需要 checker 实现、源码、tests、runtime、NAS、凭据、交易写或 publish，另起独立 CR。 |
| 验收标准 | 输出可检查的 human gate launch message draft contract、样例草案、检查项映射和后续分流；不改代码、不启动 runtime。 |
| 关闭条件 | CP2 确认范围，CP8 确认 draft contract 和 follow-up tracking。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready` |
| 门禁状态 | `closed` |
| 门禁模板 | `compact` |

## CP2 审批结果

| 字段 | 内容 |
|---|---|
| CP2 结果 | approved |
| 审批人 | user |
| 审批时间 | 2026-06-22T17:15:31+08:00 |
| 接受决策 | `DQ-CP2-CR115-01..03` 推荐方案 |
| 授权边界 | 继续 no-code/no-runtime；不授权源码、tests、checker implementation、runtime、NAS、凭据、交易写、CR105、CR089 恢复或 publish。 |
| 下一门禁 | CP8 Delivery Readiness |

## CP8 审批结果

| 字段 | 内容 |
|---|---|
| CP8 结果 | approved |
| 审批人 | user |
| 审批时间 | 2026-06-22T17:23:57+08:00 |
| 接受决策 | `DQ-CP8-CR115-01..02` 推荐方案 |
| 关闭结论 | CR115 closed-current-delivery / READY |
| 后续候选 | `FU-CR115-001` 仅登记为 candidate，不自动启动；`FU-CR114-002..004` 继续保留候选。 |
| 授权边界 | 继续 no-code/no-runtime；不授权源码、tests、checker implementation、runtime、NAS、凭据、交易写、CR105、CR089 恢复或 publish。 |

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
    write_review_evidence: true
    update_cr_index: true
    update_state_summary: true
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

- 读取 CR114 follow-up tracking、CR-INDEX、STATE 顶层摘要、CP2 / CP8 checkpoint、context capsule 和相关 process 证据。
- 生成 CR115 draft contract、context capsule、CP2 预检和人工审查稿。
- 对发现项做分流：本轮关闭、后续 CR 候选、需要 checker implementation CR、需要高风险授权 gate、不启动。
- 运行本地只读状态检查或 git status；不启动服务、不连接外部 runtime。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/design/HUMAN-GATE-LAUNCH-MESSAGE-CHECKABLE-DRAFT-CONTRACT-CR115.md` | 新增 | N/A | 文件头部修订记录 | cp2-approved |
| `process/context/CP2-CR115-HUMAN-GATE-LAUNCH-MESSAGE-CONTEXT.yaml` | 新增 | N/A | 不适用 | cp2-approved |
| `process/checks/CP2-CR115-HUMAN-GATE-LAUNCH-MESSAGE-PRECHECK.md` | 新增 | N/A | 不适用 | cp2-approved |
| `process/checkpoints/CP2-CR115-HUMAN-GATE-LAUNCH-MESSAGE-REVIEW.md` | 新增 | N/A | 不适用 | approved |
| `process/context/CP8-CR115-DELIVERY-CONTEXT.yaml` | 新增 | N/A | 不适用 | approved |
| `process/release/RELEASE-CONTEXT-CR115.yaml` | 新增 | N/A | 不适用 | approved |
| `process/checks/CP8-CR115-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/checkpoints/CP8-CR115-DELIVERY-READINESS.md` | 新增 | N/A | 不适用 | approved |
| `process/changes/CR-114-FOLLOW-UP-TRACKING-2026-06-22.md` | 原文档更新 | 保留 FU-CR114-001 历史来源并标记 closed | 不适用 | approved |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 关闭 CR115，追加 FU-CR115-001 candidate，不删除历史项 | 不适用 | approved |
| `process/STATE.md` | 原文档更新 | 只刷新顶层人类摘要，不重写历史长表 | 不适用 | approved |
| 源码 / tests / checker implementation | 不变 | 不授权修改 | 不适用 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只做人工作业门禁发起消息契约复盘。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；未来实现候选启动时再定义验证范围。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | human gate launch message backlog | true | 输出 draft contract，不创建 Story 实现批次。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime / NAS / credentials / trading / publish boundary | false | 继续禁止源码、checker implementation、runtime、NAS、凭据、交易写和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/checks` / `process/checkpoints` / `docs/design` | true | 生成 CR115 scope gate 和后续 CP8 收口证据。 |

## 冲突预检结论

| 检查维度 | 结果 | 证据 | 说明 |
|---|---|---|---|
| active formal CR | PASS | CR114 CP8 approve | CR114 已关闭，本 CR 成为唯一 active formal CR。 |
| blocked formal CR | PASS_WITH_NOTE | `CR-089` | CR089 仍 blocked；CR115 不恢复 CR089、不触碰 runtime / NAS / interface。 |
| 影响面重叠 | PASS | `FU-CR114-001` impact surface | 仅涉及 human gate launch message / decision brief contract。 |
| 高风险授权 | PASS | 本 CR authorization policy | 不授权 runtime、凭据、NAS、交易写、publish。 |
| 源码 / tests / checker implementation | PASS | 本 CR authorization policy | 不授权源码、tests 或 checker implementation 变更。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量文档 / 规则 / 排序修改 | true | 只做 draft contract 和台账同步。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不修改架构契约或权限边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改接口和文件所有权，不创建实现 Story。 |
| 需要 HLD / LLD 才能解释影响 | false | 本轮只输出治理契约草案；实现候选未来单独启动。 |
| 是否保持 fast-lane | true | 保留 CP2 和 CP8 审查证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR115 不启动代码实现。

## CP2 待确认决策

| 决策 ID | 类型 | 推荐方案 | 备选方案 | 回退条件 |
|---|---|---|---|---|
| DQ-CP2-CR115-01 | scope | 接受 CR115 只做 no-code/no-runtime human gate launch message draft contract，不改代码、不启动 runtime | A. 暂停 CR115；B. 改为 pending queue audit；C. 改为 checker implementation CR | 若用户要求源码或 runtime，另起独立 CR |
| DQ-CP2-CR115-02 | follow_up_tracking | 接受按“checkpoint 可追溯性、DQ 表一致性、不授权项显式性、三个 exact 回复、checker 可验证性”排序检查项 | A. 只输出样例消息；B. 先做 checker implementation；C. 只保留原则不写草案 | 若用户调整排序，更新 draft 并重新发起 CP2 |
| DQ-CP2-CR115-03 | runtime_authorization | 继续禁止源码修改、tests 修改、checker implementation change、CR105、CR089 恢复、runtime、交易写、NAS、凭据和 publish 被 CR115 隐式启动 | A. 另起源码实现 CR；B. 另起 checker implementation CR；C. 另起 runtime/NAS/trading authorization gate | 若用户明确要求高风险验证，停止 CR115 默认路线并创建独立 gate |

## 当前状态

| 字段 | 内容 |
|---|---|
| 当前门禁 | closed |
| 自动预检 | `process/checks/CP2-CR115-HUMAN-GATE-LAUNCH-MESSAGE-PRECHECK.md` |
| 人工审查稿 | `process/checkpoints/CP2-CR115-HUMAN-GATE-LAUNCH-MESSAGE-REVIEW.md` |
| Draft contract | `docs/design/HUMAN-GATE-LAUNCH-MESSAGE-CHECKABLE-DRAFT-CONTRACT-CR115.md` |
| 下一步 | CR115 已关闭为 READY；后续候选见 `process/changes/CR-115-FOLLOW-UP-TRACKING-2026-06-22.md`，均需用户明确选择后另起 CR。 |
