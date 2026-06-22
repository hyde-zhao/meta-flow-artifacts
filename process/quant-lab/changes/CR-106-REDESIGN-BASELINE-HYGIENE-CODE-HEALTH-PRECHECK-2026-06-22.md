---
cr_id: "CR-106"
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
rollback_to: "precheck baseline clean state before CR106"
approval_result: "cp8-approved-user-20260622-ready"
created_at: "2026-06-22T12:32:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T12:45:00+08:00"
source: "user"
linked_issue: ""
parent_cr: ""
source_checkpoint: "process/checks/PRECHECK-QL-RD-000-REDESIGN-BASELINE-2026-06-22.md"
source_decision_id: "USER-20260622-BASELINE-OK-CONTINUE-REMEDIATION"
follow_up_type: "redesign-baseline-hygiene"
risk_class: "low-process-static-readonly"
owner: "host-orchestrator"
revisit_condition: "If readonly code health precheck finds concrete failures, create a scoped remediation CR instead of broad code edits."
acceptance_criteria: "CP2 approves low-risk scope; CP6/CP7 readonly/static evidence identifies whether code-level remediation is required; no CR105/runtime/trading/NAS/provider/lake/catalog action is executed."
close_condition: "Closed by CP8 approval at 2026-06-22T13:18:00+08:00 as READY. Baseline hygiene report, code health precheck evidence, governance artifact remediation evidence and CR tracking status are written; CP7 remediation verification passed imports, identity and focused static regression."
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-106 Redesign Baseline Hygiene and Code Health Precheck

## 变更描述

用户已审阅 `precheck/ql-rd-000-redesign-baseline` baseline 清洁检查并确认“没有问题”，随后要求继续推进整改。

本 CR 将整改入口限定为低风险 baseline hygiene 与只读代码健康预检，目标是回答：

> 当前 `precheck/ql-rd-000-redesign-baseline` 是否存在必须进入代码层面整改的明确失败证据？

本 CR **不是** CR105，不进入 order-write / submit-cancel / simulation-live，不执行真实 QMT / MiniQMT / XtQuant / gateway runtime，不读取凭据 / 账户 / raw log，不访问 NAS，不执行 provider / lake / catalog publish。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | N/A |
| 来源检查点 | `process/checks/PRECHECK-QL-RD-000-REDESIGN-BASELINE-2026-06-22.md` |
| 来源决策 ID | `USER-20260622-BASELINE-OK-CONTINUE-REMEDIATION` |
| follow-up 类型 | `redesign-baseline-hygiene` |
| 风险等级 | `low-process-static-readonly` |
| owner | `host-orchestrator` |
| 重访条件 | 只读检查发现明确失败时，另起小范围代码整改 CR。 |
| 验收标准 | 形成只读健康预检证据和下一步建议，不直接修改源码。 |
| 关闭条件 | CP8 审查确认 baseline hygiene 结论和后续分流。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `process` |
| Legacy CR kind | `requirement-change` |
| 生命周期状态 | `active` |
| 就绪状态 | `ready` |
| 门禁状态 | `cp8_pending` |
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

- 读取仓库状态、CR 台账、process 路由和 baseline 报告。
- 运行本地只读 / 静态 / fixture / dry-run 健康检查。
- 生成 `process/checks/*`、`process/checkpoints/*`、`process/context/*` 和 CR 台账证据。
- 仅在检查发现明确失败后，提出后续小范围代码整改 CR 建议。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `docs/product/USE-CASES.md` | 不变 | 既有基线保留 | 不适用 | pending |
| `docs/product/REQUIREMENTS.md` | 不变 | 既有基线保留 | 不适用 | pending |
| `process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml` | 不变 | 当前需求真相不改变 | 不适用 | pending |
| `process/checks/PRECHECK-QL-RD-000-REDESIGN-BASELINE-2026-06-22.md` | 不变 | 已作为 CR106 来源证据保留 | 不适用 | pending |
| `process/changes/CR-INDEX.yaml` | 原文档更新 | 索引追加 CR106 active 项，不删除历史项 | 不适用 | pending |
| `process/STATE.md` | 原文档更新 | 人类摘要追加 CR106 当前动作，不删除历史状态 | 不适用 | pending |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| `PRECHECK-QL-RD-000-REDESIGN-BASELINE` | CR106 baseline hygiene scope | 原文保留 | baseline 预检已 PASS，CR106 仅承接后续低风险整改入口。 |
| CR102 / CR103 / CR104 closed state | CR106 non-authorization boundary | 原文保留 | CR106 不重新打开已关闭 CR，不扩大其 runtime / NAS / 交易边界。 |
| `FU-CR101-001` / proposed CR105 | CR106 forbidden scope | 原文保留 | order-write / simulation-live 继续保留为高风险候选，不由 CR106 启动。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | `REQUIREMENTS.md` / current baseline | false | 不改需求基线；只确认 baseline hygiene。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | `SCENARIOS.yaml` / `TEST-MATRIX.md` | false | 不改测试矩阵；后续只读检查作为 health evidence。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | `process/DEVELOPMENT-PLAN.yaml` | false | 不创建 Story 实现批次；若发现代码失败再另起修复 CR。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | 权限 / runtime / 交易边界 | false | 明确禁止 CR105、runtime、NAS、凭据、交易写和 publish。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | `process/checks` / `process/checkpoints` | true | 生成 CR106 scope precheck、manual gate 和后续只读健康检查报告。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`precheck baseline clean state before CR106`。
- 需要重新确认的对象：
  - `process/changes/CR-106-REDESIGN-BASELINE-HYGIENE-CODE-HEALTH-PRECHECK-2026-06-22.md`
  - `process/changes/CR-INDEX.yaml`
  - `process/STATE.md`
  - `process/checks/CP2-CR106-REDESIGN-BASELINE-HYGIENE-SCOPE-PRECHECK.md`
  - `process/checkpoints/CP2-CR106-REDESIGN-BASELINE-HYGIENE-REVIEW.md`

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | true | 只做 process / ledger / static / readonly health precheck。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不改变架构和权限边界，只强化不授权边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | false | 不修改源码或 Story 依赖。 |
| 需要 HLD / LLD 才能解释影响 | false | 范围是 baseline hygiene 和代码健康预检。 |
| 是否保持 fast-lane | true | 可走低风险 fast-lane，但仍保留 CP2 / CP6 / CP7 / CP8 证据。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false。
- batch_id：N/A。
- 批次范围来源：N/A。
- 批次内 Story：N/A。
- 批次人工确认稿：N/A。
- 开发启动条件：
  - [ ] 不适用；CR106 不启动代码实现。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR106 和 CP2 scope gate | 用户确认 baseline 没问题并要求继续整改 | CR106、CP2 自动预检、CP2 人工审查稿 | CP2 pending | 等待用户确认 scope |
| 2 | `host-orchestrator` | 执行只读代码健康预检 | CP2 approved | static/import/test/dry-run 证据 | CP6/CP7 evidence | 判断是否需要代码整改 CR |
| 3 | `host-orchestrator` | 形成整改建议 | 健康预检结果 | CP8 readiness / follow-up candidates | CP8 pending | 用户确认关闭或启动修复 CR |

## 自动终验授权

- 是否启用：false。
- 授权范围：不适用。
- 适用检查点：CP8。
- 自动通过条件：N/A。
- 授权原文：N/A。
- 授权时间：N/A。

## 后续事项台账

- 是否存在后续事项：待 CP7 / CP8 判断。
- 台账路径：`process/changes/CR-106-FOLLOW-UP-TRACKING-2026-06-22.md`。
- CR 索引路径：`process/changes/CR-INDEX.yaml`。
- 一致性检查：`meta-flow check cr-tracking --project-root .`。

| 候选编号 | 标题 | 状态 | 类型 | 优先级 | 正式 CR 路径 | 相关 active CR / blocked_by / superseded_by | 当前门控 | 阻塞原因 | 下一步 |
|---|---|---|---|---:|---|---|---|---|---|
| FU-CR106-001 | Code-level remediation from health precheck findings | candidate | CR | 1 |  | blocked_by=CR106-CP7-results | 未启动 | 仅当 CR106 发现明确代码失败时才启动 | 等待 CR106 只读预检结果 |

## 处理结论

- 审批结论：`cp2-approved-user-20260622`
- [x] 自动批准（低风险）
- [x] 待人工确认（中风险）
- [ ] 待人工审批（高风险）

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Baseline Precheck | `process/checks/PRECHECK-QL-RD-000-REDESIGN-BASELINE-2026-06-22.md` | CR106 来源证据。 |
| Branch | `precheck/ql-rd-000-redesign-baseline` | 当前整改基线分支。 |
| Forbidden Candidate | `FU-CR101-001 / CR105` | order-write / simulation-live 方向，不由 CR106 启动。 |
