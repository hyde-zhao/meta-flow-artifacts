---
cr_id: "CR-135"
title: "Runner Execution Artifact Bundle / Replay Workflow"
cr_type: "feature"
cr_kind: "implementation-gate"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR135 runner artifact bundle state"
approval_result: "approved-cp8-ready-with-risk"
created_at: "2026-06-23T17:24:26+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T18:07:24+08:00"
source: "user-request"
parent_cr: "CR-134"
source_decision_id: "USER-20260623-START-CR135"
follow_up_type: "runner-development"
risk_class: "offline-runner-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "Successful offline runner runs can write a run artifact bundle containing run-result.json, runner-evidence.index.json, run-spec.snapshot.json and manifest.json or equivalent stable machine-readable files; bundle replay/inspect reads the bundle without re-running the strategy package; blocked runs do not write a misleading pass bundle; all outputs keep not_authorization=true, qmt_allowed=false and forbidden external counters at 0."
close_condition: "CR135 implementation, tests and runner boundary checks pass; CP6/CP7 evidence records bundle writing, replay/inspect behavior, blocked-run behavior and deny-default authorization boundaries."
conflict_keys: ["strategy-runner-core", "runner-artifact-bundle", "runner-replay-inspect", "offline-runner-cli"]
impact_surface: ["trading/strategy_runner", "tests", "process/docs/features/strategy-runner-core", "process/checks", "process/context", "process/changes", "process/state"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: ["WARN-CR135-CR-TRACKING"]
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "closed"
---

# CR-135 Runner Execution Artifact Bundle / Replay Workflow

## 变更描述

在 CR134 已完成 successful offline run 的 lightweight evidence index 后，本 CR 为 Strategy Runner Core 增加执行产物 bundle 与 replay / inspect 工作流。

最小目标：

1. 定义 offline run artifact bundle 目录结构，至少包含 `run-result.json`、`runner-evidence.index.json`、`run-spec.snapshot.json`、`manifest.json` 或等价稳定文件。
2. 支持 CLI 或 helper 在 successful offline run 后一次性写入 bundle。
3. 支持从 bundle replay / inspect，不重新运行 strategy package，不访问外部系统。
4. 将 CR134 的 evidence index 作为 bundle 内的机器证据入口。
5. blocked run 只能写 blocked result 或诊断摘要，不写 misleading pass bundle。
6. 继续保持 offline-only；不授权 runtime、NAS、QMT、credentials、provider、lake、catalog、trading 或真实账户 / 委托 / 成交读取。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-134` |
| 来源检查点 | `CR089-CLOSURE-TO-CR135-CONTEXT-RESET-HANDOFF-2026-06-23` |
| 来源决策 ID | `USER-20260623-START-CR135` |
| follow-up 类型 | `runner-development` |
| 风险等级 | `offline-runner-only-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | CR134 evidence index 可作为 bundle 内机器证据入口。 |
| 验收标准 | 见 frontmatter `acceptance_criteria`。 |
| 关闭条件 | 见 frontmatter `close_condition`。 |

## CR 类型与门禁策略

| 字段 | 内容 |
|---|---|
| CR 类型 | `feature` |
| Legacy CR kind | `implementation-gate` |
| 生命周期状态 | `closed` |
| 就绪状态 | `ready_with_risk` |
| 门禁状态 | `closed` |
| 门禁模板 | `standard` |

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
```

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `trading/strategy_runner/artifact_bundle.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `trading/strategy_runner/run_spec.py` | 原代码更新 | CR128/CR133/CR134 RunSpec 合同保留；新增 bundle 输出或 snapshot 合同 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/runner.py` | 原代码更新 | CR128 runner API 与 CR134 evidence index 行为保留；新增 successful bundle writer 调用 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/cli.py` | 原代码更新 | CR133/CR134 CLI 保留；新增 bundle 输出和 inspect / replay 入口 | CP6/CP7 证据 | approved |
| `tests/test_cr135_runner_artifact_bundle.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `process/docs/features/strategy-runner-core/*` | 原文档更新 | v1.0-v1.2 保留修订记录；新增 v1.3 | 文档内修订记录 | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR134 evidence index output | CR135 artifact bundle evidence entry | 原文保留 | bundle 应复用 CR134 evidence index，不重新定义外部运行权限。 |
| CR133 RunSpec file / CLI | CR135 bundle output / inspect CLI | 原文保留 | 新入口必须保持 RunSpec fail-closed 与 spec-relative path 语义。 |
| CR128 offline runner core | CR135 successful bundle writer | 原文保留 | 不改变 offline-first / fake readonly / deny-default 边界。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增、删除或重定义 runner core 能力 | strategy-runner-core | true | 增加 successful run artifact bundle 与 replay / inspect 能力。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | offline runner fixture / blocked run / CLI | true | 新增 bundle 写入、manifest、snapshot、inspect/replay、blocked no-pass-bundle 场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | runner next slice | true | CR135 作为 CR134 后续 runner slice，进入 standard-code 实现与验证。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime/NAS/QMT/credentials/provider/lake/catalog/trading | false | 只写本地 offline bundle；外部权限继续 deny-default。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | code/tests/feature docs/process evidence | true | 新增 bundle 模块、测试、feature docs v1.3、CP6/CP7 证据。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`pre-CR135 runner artifact bundle state`。
- 需要重新确认的对象：Strategy Runner Core feature docs、RunSpec/runner/CLI bundle output 合同、CR135 tests。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 runner 输出与 replay/inspect 合同，走 standard-code。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不新增权限，只延续 deny-default 边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 新增 bundle 输出与 CLI / helper 入口。 |
| 需要 HLD / LLD 才能解释影响 | false | 现有 Strategy Runner Core feature authority 足够。 |
| 是否保持 fast-lane | false | 保持 `workflow_mode_after_change=standard`。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- batch_id：`CR-135-LITE-CODE-GATE`
- 批次范围来源：CR 影响分析
- 批次内 Story：N/A，本 CR 使用单一 standard-code implementation gate。
- 批次人工确认稿：N/A
- 开发启动条件：
  - [ ] CR135 正式 CR 已登记为 active。
  - [ ] runner feature authority 和 CR134 evidence index 合同可读。
  - [ ] 外部权限边界保持 deny-default。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR 并完成冲突预检 | 用户请求 / handoff / state v2 | 本 CR、CR-INDEX、CR ledger、STATE.current.json | CR 已登记 | 准备实现 |
| 2 | `meta-dev` | 实现 bundle writer / inspect / replay | CR135、runner feature docs、CR134 code | 代码、测试、实现证据 | CP6 | 交回验证 |
| 3 | `meta-qa` | 验证 runner 边界和回归 | CR135 代码、测试、evidence | CP7 验证证据 | CP7 | 交回收敛 |
| 4 | `host-orchestrator` | 收敛终验 | CP6/CP7、CR、索引 | CP8 自动预检与人工审查稿 | 等待用户确认或有效预授权 | 回填关闭 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 自动通过条件：
  - [ ] 自动预检结论为 `PASS`
  - [ ] 无 `BLOCKING`
  - [ ] 无 `REQUIRED`
  - [ ] 授权动作明确包含关闭 CR 和 / 或推进 `delivered`
- 授权原文：
- 授权时间：
- 回填要求：若生效，人工审查稿必须标注 `approval_source=user-preauthorized`

## 后续事项台账

- 是否存在后续事项：false
- 台账路径：`process/changes/CR-135-FOLLOW-UP-TRACKING-2026-06-23.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`meta-flow check cr-tracking --project-root .`

## 处理结论

- 审批结论：`approved-cp8-ready-with-risk`
- [ ] 自动批准（低风险）
- [x] 人工已确认（中风险）
- [ ] 待人工审批（高风险）

## CP8 关闭记录

| 字段 | 内容 |
|---|---|
| 关闭时间 | `2026-06-23T18:07:24+08:00` |
| 关闭结论 | `closed-current-delivery / READY_WITH_RISK` |
| 批准来源 | 用户回复“批准CR0135，然后继续推进项目”。 |
| 风险接受 | `WARN-CR135-CR-TRACKING` 作为非阻断 warning，后续候选 `FU-CR135-001`。 |
| 非授权边界 | 本批准不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write。 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Context handoff | `process/context/CR089-CLOSURE-TO-CR135-CONTEXT-RESET-HANDOFF-2026-06-23.md` | 指定 CR135 最小范围和非授权边界。 |
| Parent CR | `CR-134` | Runner evidence index integration。 |
| Feature docs | `process/docs/features/strategy-runner-core/*` | Runner core 长期 feature authority。 |
