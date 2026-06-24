---
cr_id: "CR-137"
title: "Offline Runner Run Registry"
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
rollback_to: "pre-CR137 runner run registry state"
approval_result: "approved-cp8-ready-with-risk"
created_at: "2026-06-23T19:16:05+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-24T09:18:51+08:00"
source: "user-request"
parent_cr: "CR-136"
source_decision_id: "USER-20260623-START-CR137"
follow_up_type: "runner-development"
risk_class: "offline-runner-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "Offline runner can append a local run registry entry for successful validated bundles and blocked diagnostic runs; registry entries include run_id, status, bundle path, manifest sha256, package_id, not_authorization and qmt_allowed; CLI exposes minimal append/list/inspect paths; no runtime, NAS, QMT, credentials, provider/lake/catalog or trading is authorized."
close_condition: "CR137 implementation, tests and runner boundary checks pass; CP6/CP7 evidence records pass bundle entry, blocked diagnostic entry, registry API/CLI behavior and deny-default authorization boundaries."
conflict_keys: ["strategy-runner-core", "runner-run-registry", "runner-artifact-bundle", "offline-runner-cli"]
impact_surface: ["trading/strategy_runner", "tests", "process/docs/features/strategy-runner-core", "process/checks", "process/context", "process/changes", "process/state"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: ["WARN-CR137-CR-TRACKING"]
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "closed"
---

# CR-137 Offline Runner Run Registry

## 变更描述

在 CR134/135/136 已完成单次 offline runner 的 evidence index、artifact bundle、inspect/replay 和 bundle validation 后，本 CR 增加多次本地运行的轻量 registry，用于后续 batch run、报告、对比和审计入口。

最小目标：

1. 定义本地 run registry 文件格式：`runner-runs.index.json`。
2. successful bundle 写入后可追加 registry entry。
3. registry entry 记录 `run_id`、`status`、`bundle_path`、`manifest_sha256`、`created_at`、`package_id`、`not_authorization`、`qmt_allowed`。
4. 已通过 spec 校验但执行 blocked 的运行可登记 diagnostic entry，但不得链接 pass bundle。
5. CLI 提供 registry append/list/inspect 最小入口。
6. registry 只记录本地路径与摘要，不复制完整 result、target portfolio、order intents 或 raw evidence。
7. 继续保持 offline-only；不授权 runtime、NAS、QMT、credentials、provider、lake、catalog、trading、Git write 或真实账户 / 委托 / 成交读取。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-136` |
| 来源决策 ID | `USER-20260623-START-CR137` |
| follow-up 类型 | `runner-development` |
| 风险等级 | `offline-runner-only-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | CR136 已完成 validated artifact bundle；下一步需要多次本地运行登记。 |
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
| `trading/strategy_runner/run_registry.py` | 新增 | N/A | CP6/CP7 证据 | active |
| `trading/strategy_runner/run_spec.py` | 原代码更新 | CR128/133/134/135 RunSpec 合同保留；新增 registry output path | CP6/CP7 证据 | active |
| `trading/strategy_runner/runner.py` | 原代码更新 | CR134/135/136 输出链路保留；追加 registry 写入 | CP6/CP7 证据 | active |
| `trading/strategy_runner/cli.py` | 原代码更新 | CR133/135/136 CLI 入口保留；新增 registry append/list/inspect | CP6/CP7 证据 | active |
| `trading/strategy_runner/__init__.py` | 原代码更新 | 既有 public API 保留；新增 registry export | CP6/CP7 证据 | active |
| `tests/test_cr137_runner_run_registry.py` | 新增 | N/A | CP6/CP7 证据 | active |
| `process/docs/features/strategy-runner-core/*` | 原文档更新 | v1.0-v1.4 保留修订记录；新增 v1.5 | 文档内修订记录 | active |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR135 artifact bundle | CR137 registry pass entry | 原文保留 | registry entry 引用 bundle path 和 manifest sha256，不复制 bundle 正文。 |
| CR136 bundle validation | CR137 append from bundle | 原文保留 | append bundle 前先 validate，坏 bundle fail closed。 |
| CR134/135 blocked no-pass-output | CR137 blocked diagnostic entry | 原文保留 | blocked entry 只记录诊断，不写或链接 pass bundle。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增 runner core 能力 | strategy-runner-core | true | 增加 offline run registry。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | pass bundle entry、blocked diagnostic entry、registry CLI | true | 新增 CR137 专项测试。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | runner next slice | true | CR137 作为 CR136 后续 offline runner slice，进入 standard-code 实现与验证。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime/NAS/QMT/credentials/provider/lake/catalog/trading | false | 只读写本地 registry / bundle 文件；外部权限继续 deny-default。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | code/tests/feature docs/process evidence | true | 新增 registry API/CLI、测试、feature docs v1.5、CP6/CP7 证据。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`pre-CR137 runner run registry state`。
- 需要重新确认的对象：Strategy Runner Core feature docs、run registry API / CLI 合同、CR137 tests。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 RunSpec、public API 和 CLI 行为，走 standard-code。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不新增权限，只延续 deny-default 边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 新增 public registry API 与 CLI 参数。 |
| 需要 HLD / LLD 才能解释影响 | false | 现有 Strategy Runner Core feature authority 足够。 |
| 是否保持 fast-lane | false | 保持 `workflow_mode_after_change=standard`。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- batch_id：`CR-137-LITE-CODE-GATE`
- 批次范围来源：CR 影响分析
- 批次内 Story：N/A，本 CR 使用单一 standard-code implementation gate。
- 批次人工确认稿：N/A
- 开发启动条件：
  - [x] CR137 正式 CR 已登记为 active。
  - [x] CR136 validated bundle 合同可读。
  - [x] 外部权限边界保持 deny-default。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR 并完成冲突预检 | 用户请求 / CR136 state | 本 CR、CR-INDEX、CR ledger、STATE.current.json | CR 已登记 | 准备实现 |
| 2 | `meta-dev` | 实现 registry API / CLI | CR137、CR136 code、runner feature docs | 代码、测试、实现证据 | CP6 | 交回验证 |
| 3 | `meta-qa` | 验证 registry fail-closed 行为和回归 | CR137 代码、测试、evidence | CP7 验证证据 | CP7 | 交回收敛 |
| 4 | `host-orchestrator` | 收敛终验 | CP6/CP7、CR、索引 | CP8 自动预检与人工审查稿 | 等待用户确认或有效预授权 | 回填关闭 |

## 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 适用检查点：CP8
- 授权原文：
- 授权时间：
- 回填要求：若生效，人工审查稿必须标注 `approval_source=user-preauthorized`

## 后续事项台账

- 是否存在后续事项：false
- 台账路径：`process/changes/CR-137-FOLLOW-UP-TRACKING-2026-06-23.md`
- CR 索引路径：`process/changes/CR-INDEX.yaml`
- 一致性检查：`meta-flow check cr-tracking --project-root .`

## 处理结论

- 审批结论：`approved-cp8-ready-with-risk`
- [ ] 自动批准（低风险）
- [x] 人工已确认（中风险）
- [ ] 正在实现与验证

## CP8 关闭记录

| 字段 | 内容 |
|---|---|
| 关闭时间 | `2026-06-24T09:18:51+08:00` |
| 关闭结论 | `closed-current-delivery / READY_WITH_RISK` |
| 批准来源 | 用户回复“approve”。 |
| 风险接受 | `WARN-CR137-CR-TRACKING` 作为非阻断 warning，后续候选 `FU-CR137-001`。 |
| 非授权边界 | 本批准不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading/Git write。 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-136` | Runner bundle schema / compatibility validation。 |
| Feature docs | `process/docs/features/strategy-runner-core/*` | Runner core 长期 feature authority。 |
